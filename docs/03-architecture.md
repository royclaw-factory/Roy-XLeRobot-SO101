# 03 — Architecture（Runtime / 執行邊界）

> Anvil/X0 的 runtime 與執行邊界圖：**真正跑 LeRobot 的是 Pi 5（Tailscale `pi5`），本 repo 只放 evidence、不執行任何機器人**。涵蓋資料流、控制板→port→馬達拓樸、leLab 與 plain LeRobot CLI 的關係、以及已驗證的 de-facto run commands。硬體聲明一律引用 `docs/04-run-log/` 檔名；未驗證者標 **TBD / 待驗證**。

---

## 0. 一句話心智模型

**兩個世界，一條界線。** Pi 5 是唯一會驅動馬達、跑 `lerobot-*` CLI、存校正快取的地方；本 child repo（GitHub `roy4222/Roy-XLeRobot-SO101`、本機 `AnvilBot`）是**純證據庫**——只收 run-log / failure-log / metrics / decisions / media-index，不含執行環境、不跑任何指令。看到 repo 裡的指令，都是「在 Pi 上實際跑過、抄回來存證」的，不是這裡執行的。

```
┌─────────────────────────────┐        ┌──────────────────────────────────────┐
│  本 repo (AnvilBot, 證據庫)  │        │  Pi 5 @ Tailscale pi5 (真實 runtime) │
│  - docs/04-run-log/*         │  抄回  │  ~/lerobot/.venv (py3.10.20)         │
│  - 05-failure / 06-metrics   │ ◀───── │  lerobot 0.4.4 (Seeed-Projects fork) │
│  - 07-decisions              │  存證  │  feetech/scservo_sdk                  │
│  - 08-media-index (只索引)   │        │  ~/.cache/huggingface/lerobot/...     │
│  ✗ 不執行、不驅動馬達        │        │  ✗ 不在 git 內，Tailscale SSH 進去   │
└─────────────────────────────┘        └──────────────────────────────────────┘
                                                       │ USB /dev/ttyACM0
                                                       ▼
                                          控制板 → 6-motor TTL daisy-chain → 臂
```

---

## 1. Runtime map（真實執行環境，全部在 Pi 5）

下表所有值出自 `docs/04-run-log/2026-06-26-follower-motor-id-setup.md` L5–6（除另註）；本 repo 內**不存在**等價環境。

| 元件 | 位置 / 值 | 說明 | Evidence |
|---|---|---|---|
| Host | Raspberry Pi 5（aarch64, Debian 13 trixie） | 唯一執行機 | `2026-06-26-follower-motor-id-setup.md` L5 |
| 網路 | Tailscale `pi5` / `100.104.92.26` | 開發機經 Tailscale SSH 進去操作 | L5 |
| Python | `~/lerobot/.venv`，**Python 3.10.20** | venv，非系統 Python | L6 |
| LeRobot | **0.4.4（Seeed-Projects fork）** | 非 upstream HF main；CLI `lerobot-*` 由此提供 | L6 |
| 馬達 SDK | `feetech` / `scservo_sdk`（pip，OK） | Feetech STS3215 半雙工 TTL | L6 |
| 校正快取 | `~/.cache/huggingface/lerobot/calibration/robots/so_follower/` | `roy_follower.json` / `roy_follower2.json` | 兩份 follower run-log |
| 自寫 demo | `~/demo_follower_v2.py`、`~/reach_follower.py` | 用 `SO101Follower.send_action()`，**非 lerobot CLI** | `2026-06-26-follower-motor-id-setup.md` L56–60 |

**界線重申**：以上沒有任何一項在本 repo 內。本 repo 只在 `docs/` 留文字證據；大檔（影片 / dataset / bag）依硬規則只在 `docs/08-media-index.md` 留索引，不 commit。

---

## 2. 硬體拓樸（board → port → 6-motor daisy-chain）

每隻臂 = 一張 Feetech 控制板（USB 端在 Pi 列舉為 `/dev/ttyACM0`，**一次只接一隻臂**）→ 一條 6 顆 STS3215 的半雙工 TTL daisy-chain（ID 1 底座 … ID 6 握把/夾爪）。

```
Pi5 USB ── /dev/ttyACM0 ── [控制板] ──TTL菊鏈── M1─M2─M3─M4─M5─M6
                                          pan lift elbow wFlex wRoll grip
```

四手↔控制板↔狀態對應（單一真相見 `docs/hardware-state.md`；此處為 runtime 視角摘要）：

| 臂 | 控制板 | bus 健康（scan_port） | 校正 id | runtime 備註 / Evidence |
|---|---|---|---|---|
| Follower #1 | `...3925`（good） | ✅ `{1000000:[1..6]}` 連掃 3 次穩 | `roy_follower` | **F3(elbow/id3) 接頭 marginal**：靜態 scan 穩、寫入間歇掉封包，`num_retry=5` 可繞但不可長扛 → teleop/錄資料 blocker（`2026-06-26-follower-motor-id-setup.md` L44,46,62–67） |
| Follower #2 | `...0335`（good） | ✅ `{1000000:[1..6]}` 連掃 5 次全綠 | `roy_follower2` | 全程無 status packet error；最乾淨的 bus（`2026-06-27-follower2-bringup.md` L51,55） |
| Leader #1 | `...0371` | ✅ `{1000000:[1..6]}`（2026-06-29 全綠、零掉封包） | `roy_leader` | 6 顆混齒比、**7.4V→配 5V**；齒比↔關節對應待 teleop 施力驗（`2026-06-29-leader1-calibration.md`） |
| Leader #2 | TBD | ❌ 未開始 | ❌ | 未開始 |
| 控制板 `...4639` | —（暫停） | **NO**：USB 可列舉但 `scan_port {}`、`RX_BYTES 0` | — | **SUSPECT、未複驗、不在關鍵路徑**（`05-failure-log.md` 2026-06-26） |

> **證據紀律（硬規則）：USB 能列舉 ≠ servo bus 健康。** `...4639` 即反例——`/dev/serial/by-id/` 看得到、port 開得起來，但 servo 端 0 回應。**沒有 `scan_port` 綠燈 + 校正 + teleop 證據，不得稱任何 bus「healthy」。** Leader #1 已於 2026-06-29 補驗（scan `[1..6]` 全綠 + 校正 `roy_leader`）；唯齒比↔關節對應待 teleop 功能性驗。
>
> Leader 逐關節齒比一律以 `docs/04-run-log/2026-06-27-leader-bringup.md`（L9–25）為準，**禁用**被組裝 runbook §4 C2 明文推翻的舊對應；完整齒比表見 `docs/hardware-state.md`。

---

## 3. 資料流：leader → follower teleop（設計路徑，W2 待驗證）

目標控制迴圈：**leader 臂手動擺動 → 讀關節角 → 鏡像送到 follower 臂**，由 plain LeRobot CLI `lerobot-teleoperate` 一條指令承載（無 policy、無負載）。

```
[人手] → Leader 臂(讀 6 軸位置) ──/dev/ttyACM?──▶ Pi5 / lerobot-teleoperate ──/dev/ttyACM?──▶ Follower 臂(send_action 鏡像)
                                                   (兩條 USB serial 同時開)
```

**現況：此迴圈尚未跑過 → 整條 data flow 標 TBD / 待驗證。** 阻塞點 = Leader #1 還沒組裝/校正/驗 bus（見第 2 節）。

**W2 配對決策（去風險）**：第一條 teleop 配 **Leader #1 → Follower #2（最乾淨 bus）**，**不配** Follower #1（避開 F3 marginal）；F3 修復排在正式錄資料前。決策記錄見 `docs/07-decisions.md`。teleop 之後的 record / train / replay / eval **全部 NOT-this-week**，W2 不寫一行對應 code。

---

## 4. leLab vs plain LeRobot CLI（為何 W2 不碰 leLab）

| 維度 | plain LeRobot CLI（W2 用這個） | leLab（FastAPI + React 包裝，W3 才看） |
|---|---|---|
| 形態 | `lerobot-setup-motors / -calibrate / -teleoperate` 等指令 | 瀏覽器導引的 calibrate→teleop→record→train→rollout 全流程 UI |
| 與 LeRobot 關係 | 直接就是 LeRobot 0.4.4（Seeed fork）的 CLI | **wrap，不 fork**：底層仍 shell-out 到 upstream `lerobot_train/record/rollout` |
| Python | Pi 上已驗證 **3.10.20** | **硬性要求 3.12+**，且把 lerobot pin 在特定 commit |
| W2 結論 | ✅ Roy 手上已驗證、teleop 直接夠用 | ❌ 與 Pi 的 3.10.20 / lerobot 0.4.4 衝突，**W2 不引入** |

**定位**：leLab 是 W3「record→train→job 編排」的 **copy-idea 參考**（其 `teleoperate.py` 鏡像迴圈、`record.py` episode/reset 相位機、`train.py`/`jobs.py` CLI 包裝值得抄想法，但 SO-101 硬編需改）。W2 為了一個 UI 引進 FastAPI+React + 版本衝突，不划算。詳細分級見 `docs/reference-map.md`。leLab 延後決策記於 `docs/07-decisions.md`。

> 注意：`lerobot-*` CLI 來自 **LeRobot 本體**（Pi 的 venv），不是 leLab。run-log 用到的 CLI 與 leLab 無依賴關係。

---

## 5. De-facto run commands（已在 Pi 上實跑、抄回存證）

以下指令出自 run-log，**皆在 Pi `~/lerobot/.venv` 內執行**；本 repo 不執行它們。

```bash
# 1) 寫馬達 ID（follower / leader 各一次；leader 用 --teleop.*）
lerobot-setup-motors --robot.type=so101_follower  --robot.port=/dev/ttyACM0      # F1/F2 ✅
lerobot-setup-motors --teleop.type=so101_leader   --teleop.port=/dev/ttyACM0     # L1 ✅（僅到此）

# 2) 校正（產出 ~/.cache/.../so_follower/<id>.json）
lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=roy_follower    # ✅
lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=roy_follower2   # ✅
# lerobot-calibrate --teleop.type=so101_leader ...   # ← Leader #1 待做（assembly 後），TBD

# 3) bus 健檢（宣稱「healthy」的唯一依據）
python -c "from lerobot.motors.feetech import FeetechMotorsBus; print(FeetechMotorsBus.scan_port('/dev/ttyACM0'))"
#   good 板 → {1000000: [1,2,3,4,5,6]}   ；  SUSPECT 板(...4639) → {}

# 4) 首次動作 demo（自寫，非 CLI；只 Follower #1 跑過）
python ~/demo_follower_v2.py   # pan±25 / gripper 0~100 / 點頭
python ~/reach_follower.py      # 前後伸 ×2

# 5) teleop（W2 里程碑，尚未跑 → TBD）
# lerobot-teleoperate ...   leader → follower 鏡像，配 L1→F2
```

| 步驟 | F1 | F2 | L1 | L2 |
|---|---|---|---|---|
| setup-motors | ✅ | ✅ | ✅ | ❌ |
| 組裝 | ✅ | ✅ | ⏳ 下一步 | ❌ |
| scan bus | ✅ 3× | ✅ 5× | 🔄 TBD | ❌ |
| calibrate | ✅ | ✅ | ❌ 待做 | ❌ |
| 首次動作 | ✅ | 🔄 TBD | ❌ blocked | ❌ |
| teleop / record / train / eval | ❌ 未開始 | ❌ 未開始 | ❌ | ❌ |

（步驟矩陣摘自三份 run-log + `docs/06-metrics.md`；完整單一真相見 `docs/hardware-state.md`。）

---

## 6. 安全與電源邊界（runtime 前置條件）

- **5V/12V 不可誤插**：Leader 為 7.4V 馬達 **必配 5V**，**12V 直插 leader = 永久燒馬達**；Follower 為 12V（STS3215-C018）。電源須實體貼標 `5V→LEADER` / `12V→FOLLOWER`（`docs/research/2026-06-26-so-arm101-assembly-runbook.md` §0；`2026-06-27-leader-bringup.md` L7）。
- **一次只接一隻臂**：四隻臂目前都列舉成 `/dev/ttyACM0`；多臂同時上線的 port 區分尚未處理（W2 teleop 需同開 leader+follower 兩條 serial，port 對應屬待驗證項，追蹤於 `docs/open-questions.md`）。

---

## 7. 定位（避免新人踩坑）

- **四層命名＝同一台機器人**（見 `CONTEXT.md`）：品牌 **Anvil x XLeRobot** = 短稱 **Anvil/X0** = GitHub repo id `roy4222/Roy-XLeRobot-SO101` = 本機目錄 `src/AnvilBot`（GitHub UI 看不出這層對應）。
- **Anvil 是與 Go2 / PawAI 平行的獨立機器人主線，不是其手臂子模組。**
- repo 根的 `README.md` / `README_cn.md` / `LICENSE` 是 **upstream MakerMods** 內容（MIT 2025 MakerMods），**不代表 Roy 的進度**；Roy 的真實進度只在 `docs/04-run-log/` 等 evidence 檔。
- 本檔為 runtime/邊界圖；**W2 不是 VLA / VR / world-model / mobile base 實作週**，那些只在 `docs/learning-roadmap.md` / `docs/reference-map.md` 當未來菜單。

---

## 8. Evidence 連結

- `docs/04-run-log/2026-06-26-follower-motor-id-setup.md` — Pi 環境、F1 ID/組裝/bus 排障/校正/首次動作、F3 marginal
- `docs/04-run-log/2026-06-27-follower2-bringup.md` — F2 ID、板 `...0335` 5× 全綠、校正 range
- `docs/04-run-log/2026-06-27-leader-bringup.md` — L1 6 顆混齒比、5V、ID 表（bus 健康仍 TBD）
- `docs/05-failure-log.md` — F3 接頭 marginal、板 `...4639` SUSPECT
- `docs/06-metrics.md` — follower 校正 encoder range
- `docs/07-decisions.md` — 2F+2L 傾向、SUSPECT 不蓋棺、（待附加）L1→F2 teleop 配對、leLab 延 W3
- `docs/hardware-state.md` — 四手狀態 + leader 逐關節齒比的單一真相（W2 P0 交付）

> 本檔只描述 runtime 與邊界，不重述硬體明細與齒比；具體值一律以上列 evidence 檔與 `docs/hardware-state.md` 為準。未列入 evidence 的成果一律視為 **TBD / 待驗證**。
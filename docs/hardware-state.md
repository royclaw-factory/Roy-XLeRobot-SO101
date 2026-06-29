# Hardware State — 四臂即時狀態表

> 四臂（2 follower + 2 leader）的單一真相：馬達型號／電壓／控制板／port／馬達 ID／齒比／校正／組裝／已知問題。所有值的 ground-truth 來自 `docs/04-run-log/`，未經 run-log/failure-log 佐證者一律標「TBD／待驗證」。本檔為彙整視圖，遇衝突以原始 run-log 為準。

**證據基準**：`04-run-log/2026-06-26-follower-motor-id-setup.md`、`04-run-log/2026-06-27-follower2-bringup.md`、`04-run-log/2026-06-27-leader-bringup.md`、`05-failure-log.md`、`06-metrics.md`、`07-decisions.md`。
**最後更新**：2026-06-29（依上述 run-log 收斂）。

> 注意：本 repo 只存證據，不執行。真正的 lerobot 跑在 Pi 5（Tailscale `pi5` / 100.104.92.26，`~/lerobot/.venv`）。GitHub README 為 MakerMods upstream，**不代表 Roy 進度**。

---

## 電壓鐵則（先讀；違反會永久燒馬達）

| 臂型 | 馬達 | 電源供應器 | 後果 |
|---|---|---|---|
| **Leader**（7.4V 馬達） | STS3215 混齒比 | **5V** | 接 12V → **直接燒馬達（不可逆）** |
| **Follower**（12V 馬達） | STS3215-C018 等 | **12V** | — |

- 電源須**實體貼標**：`5V→LEADER` / `12V→FOLLOWER`。
- 依據：`2026-06-27-leader-bringup.md` L7（「**切勿接 12V，會燒**」）、`2026-06-26-follower-motor-id-setup.md` L8（C018 為 12V）。

## 板況警示（blocker，務必先看）

- **F3 接頭 marginal（Follower #1）**：靜態 `scan_port` 穩，但**需回應的寫入間歇掉封包**，`id3`（F3／肘）最常重現；`num_retry=5` 只能繞過，**不可長期硬扛**，teleop／錄資料會不穩。待辦＝徹底重壓／更換 F3 接頭（或該段線）。依據：`2026-06-26-follower-motor-id-setup.md` L62-67、`07-decisions.md` L21。
- **控制板 `...4639` = SUSPECT、暫停、未複驗**：USB 可列舉，但 `scan_port` 回 `{}`、`RX_BYTES 0`。不蓋棺、不在關鍵路徑，待「好馬達+好線」單獨複驗。依據：`05-failure-log.md` 2026-06-26 第 2 列、`07-decisions.md` L16-21。
- **Leader #1 servo bus 健康仍 TBD**：目前只寫了 6 顆 ID，組裝＋整串 `scan_port`＋校正全部**待做**。在驗證前**不得宣稱 leader ready／bus healthy**。依據：`2026-06-27-leader-bringup.md` L32-36。

---

## 四臂狀態總表

| 臂 | 馬達 / 電壓 | 控制板 | port | 馬達 ID | 組裝 | bus（scan_port） | 校正（id / json） | 已知問題 / blocker |
|---|---|---|---|---|---|---|---|---|
| **Follower #1** | 6× STS3215-C018 / 12V / 1:345 | `...3925`（板#2，good） | `/dev/ttyACM0` | ✅ 1-6 | ✅ | ✅ `[1..6]`（連掃 3 次） | ✅ `roy_follower` | **F3 接頭 marginal 掉封包**（待修） |
| **Follower #2** | 6× STS3215（model 777）/ 12V | `...0335`（板#3，二手，good） | `/dev/ttyACM0` | ✅ 1-6 | ✅ | ✅ `[1..6]`（連掃 5 次全綠） | ✅ `roy_follower2` | 無；**尚未做首次動作 demo** |
| **Leader #1** | 6× STS3215 混齒比 / **7.4V→配 5V** | `...0371`（板#4，`5AAF220371`） | `/dev/ttyACM0` | ✅ 1-6 | **待做** | **TBD**（首掃 `{}` 已解，整串未驗） | **待做** `roy_leader` | bus 健康未驗證；組裝/校正 pending |
| **Leader #2** | TBD（推測同 L1，6× STS3215 混齒比 / 7.4V） | TBD | TBD | ❌ 未開始 | ❌ | ❌ | ❌ | 尚未開始 |

> port 標 `/dev/ttyACM0` 為「單接該板時」的列舉值，並非可同時佔用。校正 json 一律落在 `~/.cache/huggingface/lerobot/calibration/robots/so_follower/<id>.json`。
> 證據：Follower #1 = `2026-06-26-follower-motor-id-setup.md` L4-52；Follower #2 = `2026-06-27-follower2-bringup.md` L4-57；Leader #1 = `2026-06-27-leader-bringup.md` L4-37；Leader #2 = 未開工。

---

## Leader #1 逐關節齒比對應（run-log 權威）

依 `2026-06-27-leader-bringup.md` L17-26（工具逐顆確認）。**裝錯關節＝力矩不匹配**，務必照此表。

| 提示名稱 | ID | 標籤 | 齒比 / 型號 |
|---|---|---|---|
| shoulder_pan | 1 | L1 / 底座 | **1/191 C044** |
| shoulder_lift | 2 | L2 / 肩抬 | **1/345 C001** |
| elbow_flex | 3 | L3 / 肘 | **1/191 C044** |
| wrist_flex | 4 | L4 / 腕俯仰 | **1/147 C046** |
| wrist_roll | 5 | L5 / 腕滾 | **1/147 C046** |
| gripper | 6 | L6 / 握把扳機（**handle，非夾爪**） | **1/147 C046** |

分布＝1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。

> ⚠️ **拒用被推翻的舊對應**：組裝 runbook `2026-06-26-so-arm101-assembly-runbook.md` §4 C2 明文標示「ID1:1/345, ID2-3:1/147, ID4-5:1/191, ID6:1/345」**經查證為錯誤**（把肘/腕齒比反置），照它裝會把馬達放到錯誤關節。一律以上表 run-log 值為準。

---

## 控制板狀態（USB 列舉 ≠ servo bus 健康）

| 控制板 | 指派 | USB 列舉 | servo bus | 狀態 | 證據 |
|---|---|---|---|---|---|
| `...4639`（板#1） | 原試 Follower #1 | ✅ | ❌ `scan_port {}`、`RX_BYTES 0` | **SUSPECT、暫停、未複驗** | `05-failure-log.md` 2026-06-26 第 2 列 |
| `...3925`（板#2） | **Follower #1（現役）** | ✅ | ✅ `{1000000:[1..6]}`（穩定 3 次） | **GOOD** | `2026-06-26-follower-motor-id-setup.md` L44-46 |
| `...0335`（板#3，二手） | **Follower #2（現役）** | ✅ | ✅ `{1000000:[1..6]}`（連掃 5 次全綠、零掉封包） | **GOOD** | `2026-06-27-follower2-bringup.md` L51, L55 |
| `...0371`（板#4） | **Leader #1（現役）** | ✅ | **TBD**（已寫 ID；整串 scan 待組裝後驗） | **USB OK／單顆曾回 `{3:777}`；整串 `[1..6]` bus 未驗** | `2026-06-27-leader-bringup.md` L6, L30 |

**判準（證據紀律）**：`ls /dev/serial/by-id/` 能看到、`screen` 能開 ≠ bus 健康。`...4639` 即反例（USB 通、servo 端零回應）。**宣稱 bus healthy 前一律先 `FeetechMotorsBus.scan_port("/dev/ttyACM0")` 取得 `[1..6]`**（唯讀、全鮑率；`from lerobot.motors.feetech import FeetechMotorsBus`）。

---

## 校正範圍（counts，供對照；權威見 06-metrics 與各 run-log）

| 軸 | Follower #1（`roy_follower`） | Follower #2（`roy_follower2`） |
|---|---|---|
| shoulder_pan | 955–3140 | 983–3174 |
| shoulder_lift | 837–3208 | 847–3205 |
| elbow_flex | 1019–3228 | 854–3092 |
| wrist_flex | 937–3208 | 934–3244 |
| wrist_roll | 0–4095（整圈，記錄階段排除屬正常） | 整圈自動記錄（手掃排除） |
| gripper | 936–2358 | 949–2479 |

> 證據：`2026-06-26-follower-motor-id-setup.md` L49、`2026-06-27-follower2-bringup.md` L53。Leader #1 校正未做（TBD）。

## 執行環境（runtime，非本 repo）

- 主機：Raspberry Pi 5（Debian 13 trixie / aarch64），Tailscale `pi5` / 100.104.92.26。
- 環境：`~/lerobot/.venv`（**Python 3.10.20、lerobot 0.4.4 Seeed-Projects fork**、feetech/scservo_sdk OK）。
- 校正快取：`~/.cache/huggingface/lerobot/calibration/robots/so_follower/`。
- 自寫首次動作腳本（Pi 上，非 lerobot CLI）：`~/demo_follower_v2.py`、`~/reach_follower.py`（`SO101Follower.send_action()`，`max_relative_target=15`）。
- 證據：`2026-06-26-follower-motor-id-setup.md` L5-6, L56-60。

---

## 已知問題與待驗證（TBD 清單）

| 項目 | 狀態 | 後續 / 緩解 |
|---|---|---|
| F3 接頭 marginal（Follower #1） | **未修**，teleop blocker | W2 teleop 改配 Follower #2；正式錄資料前重壓/更換 F3 |
| 控制板 `...4639` SUSPECT | **未複驗** | 好馬達+好線單獨重測；不在關鍵路徑 |
| Leader #1 組裝 + bus + 校正 | **TBD** | 依 ID1→ID6 組裝（5V、混齒比放對關節）→ `scan_port [1..6]` → `lerobot-calibrate ... --teleop.id=roy_leader` |
| Leader #2 整條 pipeline | **未開始** | 同 Leader #1 流程 |
| Follower #2 首次動作 demo | **未做** | 仿 `~/demo_follower_v2.py` 改 port/id 目視確認 |
| teleop / record / train / eval | **全未開始** | leader→follower 首測建議配 **Leader #1 → Follower #2**（乾淨 bus），用 `lerobot-teleoperate`，無負載、無 policy，錄影片+log 當證據 |
| 二手馬達來源混雜 | 記錄即可，非 blocker | 工具覆寫舊 ID 無問題 |
| 馬達方向「預設反轉」是否需驗 LeRobot `SO101*Config` | **TBD** | 於 `open-questions.md` 追蹤 |

> 提醒：W2 不是 VLA / VR / world-model / mobile base 實作週。Anvil／X0 是與 Go2／PawAI 平行的獨立機器人，**不是其手臂子模組**。
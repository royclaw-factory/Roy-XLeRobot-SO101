# 00 — Overview

> **Anvil x XLeRobot**（短稱 Anvil/X0；GitHub `roy4222/Roy-XLeRobot-SO101`，本機 `src/AnvilBot`）專案總覽：定位、與 Go2/PawAi 的平行關係、北極星、暑假第一條 learning loop 與 touch-marker、目前真實現況。命名 glossary 見 `CONTEXT.md`；主 repo 的角色摘要見 `RoyBot-Lab/robots/xlerobot-so101.md`。此檔放本 child repo 自身視角。

## 四個名字 = 同一台機器人（新人先看這段）

| 層級 | 名稱 |
|---|---|
| 正式品牌名 / README 標題 | **Anvil x XLeRobot** |
| 平台代號 / 短稱 | **Anvil/X0** |
| GitHub repo legal id（`origin`） | **`roy4222/Roy-XLeRobot-SO101`** |
| 本機工作目錄 | **`src/AnvilBot`**（舊路徑 `src/Roy-XLeRobot-SO101` 已不存在）|

這層對應在 GitHub UI **看不出來**（repo 名不含 Anvil），是最容易誤判的坑。四者指同一條 robot line；命名 glossary 與決策見 `CONTEXT.md` 與 `docs/07-decisions.md`（2026-06-29 命名 ADR）。

## 這是什麼

**Anvil x XLeRobot** 是 Roy 的 **SO-ARM101 / XLeRobot 機械手臂線**——低成本「輪式雙臂 Physical-AI 平台」、manipulation learning 的學習平台。硬體採 SO-101 Pro 版（follower 12V C018、leader 7.4V 混齒比）；行動底盤（LeKiwi / dual-wheel）為長期 stretch，非現階段。軟體跑 HuggingFace LeRobot（Seeed fork）在 Pi 5 上。本 repo 放該機器人線的**技術細節與證據**（run log / failure log / metrics / 媒體索引），**不執行機器人**。

## 這不是什麼（定位釐清）

- **Anvil 是與 Go2 / PawAi（Furnace OS）平行的獨立主線**，不是它們的手臂子模組。各線 git 歷史 / license / 更新節奏 / 公開程度不同，刻意不合併。
- **不是 code monorepo**：本 repo 是「fork-overlay + 證據庫」，真正的執行環境在 Pi 5（見現況），這裡只存紀錄。
- **upstream MakerMods README 不是 Roy 的進度**：`README.md` / `README_cn.md` / `LICENSE`（MIT 2025 **MakerMods**）/ `3mf/` / `assets/` 是硬體設計上游疊加層，描述的是預組裝外觀升級件，**不可當作 Roy 的實機狀態引用**。Roy 的真相在 `docs/04-run-log/`、`05-failure-log.md`、`06-metrics.md`、`07-decisions.md`。

## 北極星

一台可重現、低成本、會移動、能用雙臂操作、能收 teleop 資料、能訓 policy、並逐步長出感知 / 語音 / world-model 的類人形平台。

## 狀態模型（Stage 能力 gate + Week 排程，2026-06-29 LOCKED）

兩層，不要混為一套（決策見 `docs/07-decisions.md`）：

- **Stage 0–3 = 權威能力成熟度 gate**（沿用父 `robots/xlerobot-so101.md` LOCKED 模型）：Stage 0 follower bring-up + 首次動作 → Stage 0.5 leader station → **Stage 0 pass = 單臂 teleop 連續 5 分鐘無失控 + 影片/log** → Stage 1（teleop 收資料 + ACT，entry task = `touch marker`）→ Stage 2/3（接觸任務 / 評估，含推動式物品分類）。
- **W1–W4 = 行事曆排程**，疊在 Stage 上，**不是**另一套里程碑。
- **退役（historical）**：Phase 1–8 / Button Press 舊模型不再使用。
- ⚠️ **W2「讓 teleop 動起來」≠ Stage 0 pass**（W2 ⊂ Stage 0；5 分鐘穩定 teleop 可能落在 W2 末或 W3）。

## 目標（暑假第一條 learning loop）

跑通**第一條 SO-101 manipulation learning pipeline**：

```
硬體 bring-up → 校正 → leader→follower teleop → dataset → ACT baseline → 真機推論 → eval → 失敗分析 → roy422.dev 草稿
```

首個任務刻意保守：**touch marker（手臂去碰一個標記點）= Stage 1 的 entry task**。它驗的是「資料 → 訓練 → 推論 → eval」這條**管線本身**能不能跑通，不是任務難度。**推動式物品分類**屬 Stage 1 後段 / Stage 2 的 eval，不是第一個 gate。真正瓶頸是乾淨 teleop demo 的資料品質，不是模型容量。分階 AI 路線（ACT → HIL-SERL / Diffusion → SmolVLA / X-VLA → world-model）屬未來菜單，收進 `docs/learning-roadmap.md`。

## W2（2026-06-30 ~ 07-06，排程；推進 Stage 0.5 → 朝 Stage 0 pass）範圍

W2 是**硬體收斂排程，不是學習週**，目標是推進 **Stage 0.5（leader station）** 並朝 **Stage 0 pass** 邁進——但 **W2 本身 ≠ Stage 0 pass**。只做：補完 leader、跑出第一條 leader→follower 空中動作 teleop、收斂四手真相、給老師採購清單。

**W2 要做（scope）**
- 四臂貼標（Leader A/B、Follower A/B）。
- 四手狀態總表（ID / 齒比 / 控制板 / port / 校正 / 組裝 / blocker）→ 將建 `docs/hardware-state.md`。
- 至少一條 **Leader #1 → Follower #2（乾淨 bus）** 空中動作 teleop（無負載、無 policy）。
- v0 採購清單（給老師）→ 填 `docs/02-bom.md`。
- 證據：四臂貼標照 / teleop 影片 / terminal log（大檔只放索引，見 `docs/08-media-index.md`）。

**W2 NOT-this-week（明確排除）**
- 50-ep dataset、ACT 訓練、SmolVLA / pi0 / GR00T 等 VLA。
- mobile base（LeKiwi / dual-wheel）、VR bimanual（Quest / Phosphobot）、world-model（Cosmos3）。
- Go2 / PawAi 整合、自動成功判定 / reward model。
- 不為上述任何一項在 W2 寫一行 code；它們只進 reference-map / learning-roadmap 當未來菜單。

## 非目標 / 範圍邊界（長期）

- 不把單台機器人的大量技術細節寫進主 repo（主 repo 只放 manifest / 索引）。
- 不做「四手霸王」（4 follower、24-DoF 自訂系統）——列為 phase-2 stretch，非現階段（見 `07-decisions.md`）。
- Quest 2 VR teleop 列為 phase-2 spike，待雙 follower 穩定後再試。

## 現況（2026-06-29，Stage 0 ✅ / Stage 0.5 🔄 / Stage 0 pass ❌）

> **Stage 狀態**：Stage 0（follower bring-up + 首次動作）✅；Stage 0.5（leader station）🔄 進行中（Leader #1 僅寫 ID）；**Stage 0 pass（單臂 teleop 5 分鐘無失控 + evidence）尚未達成**。
>
> 證據紀律：「USB 能列舉」≠「servo bus 健康」。沒有 scan / calibration / teleop evidence 不可稱 bus healthy。Leader #1 目前**只寫了 ID**，bus 健康仍 **TBD / 待驗證**。

| 臂 | 馬達 / 電壓 | 控制板 | ID | 組裝 | bus | 校正 | blocker / 備註 |
|---|---|---|---|---|---|---|---|
| Follower #1 | 6× STS3215-C018 / 12V / 1:345 | `...3925`(good) | ✅ | ✅ | ✅(scan 3×穩) | ✅ `roy_follower` | **F3（肘）接頭 marginal、寫入間歇掉封包**；teleop/錄資料前須重壓/更換 |
| Follower #2 | 6× STS3215(777) / 12V | `...0335`(good) | ✅ | ✅ | ✅(scan 5×全綠) | ✅ `roy_follower2` | 乾淨，無已知問題；首次動作 demo 尚未做 |
| Leader #1 | 6× STS3215 混齒比 / **7.4V → 配 5V** | `...0371` | ✅ | **待做** | **TBD / 待驗證** | **待做** | bus 健康未驗證，組裝＋scan＋校正皆 TBD |
| Leader #2 | TBD（同 L1） | TBD | ❌ | ❌ | ❌ | ❌ | 未開始 |

控制板 `...4639` = **SUSPECT、暫停、未複驗**（USB 可列舉但 `scan_port` 回 `{}`、`RX_BYTES 0`），不在關鍵路徑。

**進度來源 run-log（硬體聲明一律引用）**
- Follower #1：`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`（ID / 組裝 / bus 排障 / 校正 / 首次自主動作，已目視確認；F3 待辦）。
- Follower #2：`docs/04-run-log/2026-06-27-follower2-bringup.md`（ID / 板 `...0335` / scan 5×全綠 / 校正 range）。
- Leader #1：`docs/04-run-log/2026-06-27-leader-bringup.md`（6 顆混齒比 / 5V / ID 表；bus 健康 TBD）。
- 失敗 / 風險：`docs/05-failure-log.md`（F3 marginal、板 `...4639` SUSPECT）。
- 量測：`docs/06-metrics.md`（follower 校正 encoder range）。
- 決策：`docs/07-decisions.md`（2F+2L 傾向、四手霸王 / VR 推 phase-2、suspect 不蓋棺）。

## Leader 逐關節齒比（以 run-log 為準）

> 一律用 `docs/04-run-log/2026-06-27-leader-bringup.md` 的值；**拒用**組裝 runbook §4 C2 已標錯並推翻的舊對應（該舊對應把肘 / 腕齒比反置，照它裝會把馬達裝到錯誤關節）。混齒比裝錯關節 = 力矩不匹配。

| 關節 (ID) | 功能 | 齒比 / 型號 |
|---|---|---|
| L1 | shoulder_pan 底座 | **1/191 C044** |
| L2 | shoulder_lift 肩抬 | **1/345 C001** |
| L3 | elbow_flex 肘 | **1/191 C044** |
| L4 | wrist_flex 腕俯仰 | **1/147 C046** |
| L5 | wrist_roll 腕滾 | **1/147 C046** |
| L6 | gripper 握把扳機 | **1/147 C046** |

分布 = 1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。

## 安全紅線（兩條必守）

- **5V / 12V 誤插 = 燒 leader 馬達**：leader 7.4V 馬達必配 5V，12V 直燒（`docs/04-run-log/2026-06-27-leader-bringup.md`）。電源實體貼標 `5V→LEADER` / `12V→FOLLOWER`。
- **Leader 混齒比裝錯關節 = 力矩不匹配**：務必照上表 run-log 齒比裝配。

## 接下來 3 個行動（對齊 W2）

1. **補完 Leader #1（關鍵路徑）**：ID1 底座 → ID6 握把組裝（混齒比放對關節、**5V 電源**）→ 整串 `FeetechMotorsBus.scan_port` 驗 `[1..6]` 全綠 → `lerobot-calibrate --teleop.type=so101_leader`。產出新 run-log + scan 截圖；在此之前 Leader bus 健康一律寫 TBD。
2. **拿下 W2 里程碑：第一條 leader→follower 空中動作 teleop**——配 **Leader #1 → Follower #2（乾淨 bus）**、用 `lerobot-teleoperate`、無負載無 policy，錄影片 + terminal log。避開 F3；F3 修復排在正式錄資料前。
3. **收斂 W2 P0 文件**：四臂貼標拍照 → 建 `docs/hardware-state.md`（含齒比 / 板 / port / 校正 / blocker）→ 填 `docs/02-bom.md`（v0 採購清單給老師）→ 填 `docs/08-media-index.md` 索引照片 / 影片 / log。
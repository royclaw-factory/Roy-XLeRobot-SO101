# 02 — BOM（物料清單 / v0 採購清單）

> Anvil x XLeRobot 的硬體 / 零件 / 印件清單。**現有**以 run-log / `docs/hardware-state.md` 為準；**規格 / 採購**以上游 `reference/SO-ARM100` README 為源（金額採購前複查）。給老師的 v0 採購清單以「B. 待購」為主。未確認者一律 TBD。

## A. 現有硬體（已驗證，來源 run-log）

| 項目 | 規格 | 數量 | 狀態 | 出處 |
|---|---|---|---|---|
| Follower 臂馬達 | 6× STS3215-C018 / 12V / 1:345（每臂） | 2 臂＝12 顆 | ✅ 已 ID/組裝/校正（F1/F2） | `04-run-log/2026-06-26-*`、`2026-06-27-follower2-*` |
| Leader 臂馬達 | 6× STS3215 混齒比 / 7.4V（1×1/345 + 2×1/191 + 3×1/147，每臂） | 2 臂＝12 顆 | 🔄 L1 已 ID；L2 未開始 | `04-run-log/2026-06-27-leader-bringup.md` |
| 控制板（Feetech / Seeed XIAO，晶片 1a86） | USB-serial servo bus driver | 4 塊 | `...3925` good / `...0335` good / `...0371` leader / **`...4639` SUSPECT** | `hardware-state.md`、`05-failure-log.md` |
| 電源供應器 | 12V（follower）、5V（leader） | 各 ≥1 | ✅ | `04-run-log/2026-06-27-leader-bringup.md` |
| 樹莓派 | Pi 5（Debian 13 / aarch64） | 1 | ✅（Tailscale `pi5`） | run-log |

> ⚠️ 電壓鐵則：leader 7.4V 馬達**必配 5V**，12V 直插燒馬達。電源實體貼標 `5V→LEADER` / `12V→FOLLOWER`。

## B. v0 待購清單（給老師；規格源 SO-ARM100，金額複查）

| 項目 | 規格 / 建議 | 數量 | 用途 / 階段 | 備註 |
|---|---|---|---|---|
| 固定外部 RGB camera | UVC webcam（型號 **TBD**，如穩定 720p+ webcam） | 1 | Stage 1 `touch marker` dataset（W3） | 先一顆穩定 UVC 過 pipeline；深度相機（D405）留長期（`open-questions.md` Q6） |
| F3 接頭 / 5264 線材 | STS3215 3-pin / 5264 連接器 | 數條 | 修 Follower #1 F3 marginal | 重壓 or 換線（Q4） |
| 備用控制板 | 同現用 Feetech 板 | 0–1 | 視 `...4639` 複驗結果 | 確認壞才補（Q3） |
| Leader #2 物料 | 同 Leader #1（混齒比 6 顆 + 板 + 5V） | 1 套 | 視 2F+2L 是否 LOCK（Q1/Q5） | **未鎖定前不採購** |
| 3D 列印耗材 | PLA+（結構）/ TPU（夾爪） | — | 補印件 | 規格見 SO-ARM100 `3DPRINT.md` |

> 完整 BOM / 採購連結 / 列印規格的權威在 `reference/SO-ARM100/README.md` 與 `3DPRINT.md`（read-only）。金額時效會變，採購前複查；歷史量級見 `docs/reference-map.md` §2。mobile base（LeKiwi）/ Jetson / 電池為長期 stretch，**非 W2 採購**。

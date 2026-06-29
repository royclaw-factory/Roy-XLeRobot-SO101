# Anvil x XLeRobot

Roy's low-cost wheeled dual-arm Physical AI platform, built on the **XLeRobot**, **SO-ARM101**, **LeRobot**, and **LeKiwi** references.

| | |
|---|---|
| **Project brand** | **Anvil x XLeRobot**（platform shorthand: `Anvil/X0`） |
| **Repository identity** | `roy4222/Roy-XLeRobot-SO101`（GitHub origin — repo 名刻意保留，未改名） |
| **Local workspace** | `src/AnvilBot`（本機目錄；舊路徑 `src/Roy-XLeRobot-SO101` 已不存在） |
| **Upstream hardware** | `makermods-robotics/MakerMods-XLeRobot`（見 [README-makermods-hardware.md](./README-makermods-hardware.md)） |

> 這四個名字＝**同一台機器人**。GitHub UI 看不出這層對應，是新人最大誤判點。完整命名 glossary 見 [`CONTEXT.md`](./CONTEXT.md)。

---

## 這是什麼

**Anvil x XLeRobot** 是低成本「輪式雙臂 Physical AI 平台」、manipulation learning 的學習平台。硬體採 SO-101 Pro 版（follower 12V、leader 7.4V 混齒比）+ 未來 LeKiwi 行動底盤；控制軟體跑 HuggingFace **LeRobot**（Seeed fork）在 Raspberry Pi 5 上。

**本 repo 是「fork 疊加層 + 證據庫」，不是 code repo、也不執行機器人。** 它放這條線的技術細節與證據（run-log / failure-log / metrics / 決策 / 媒體索引）；真正驅動馬達的 `lerobot-*` 跑在 Pi 5（見 [`docs/03-architecture.md`](./docs/03-architecture.md)）。

## 這不是什麼

- **不是 Go2 / PawAI（Furnace OS）的手臂子模組**——Anvil 是與其平行的獨立機器人主線。
- **不是 MakerMods 專案**——`README-makermods-hardware.md`、`README_cn.md`、`LICENSE`、`3mf/`、`assets/` 是 upstream MakerMods 的硬體設計疊加層，**不代表 Roy 的進度**。Roy 的真相在 `docs/04-run-log/`、`05-failure-log.md`、`06-metrics.md`、`07-decisions.md`。

## 北極星

一台可重現、低成本、會移動、能用雙臂操作、能收 teleop 資料、能訓 policy、並逐步長出感知 / 語音 / world-model 的類人形平台。第一條 learning loop：`bring-up → 校正 → leader→follower teleop → dataset → ACT → 真機推論 → eval → 失敗分析`。

## 現況（2026-06-29）

狀態用 **Stage 0–3 能力 gate**（權威）+ **W1–W4 排程**（疊在 Stage 上）兩層表達（見 [`docs/07-decisions.md`](./docs/07-decisions.md)）。

- **Stage 0（follower bring-up）**：Follower #1 / #2 已 ID + 組裝 + bus + 校正；Follower #1 首次自主動作已目視確認。✅（細節 [`docs/hardware-state.md`](./docs/hardware-state.md)）
- **Stage 0.5（leader station）**：Leader #1 已寫 ID，組裝 / bus 健康 / 校正 **TBD**；Leader #2 未開始。🔄
- **Stage 0 pass（單臂 teleop 連續 5 分鐘無失控 + evidence）**：**尚未達成**。
- **目前排程 W2**：補完 Leader #1 + 跑出第一條 leader→follower 空中動作 teleop（**W2 ≠ Stage 0 pass**）。
- **Stage 1 entry task** = `touch marker`（尚未開始）。

> ⚠️ 證據紀律：「USB 能列舉」≠「servo bus 健康」；沒有 scan/calibration/teleop evidence 不可稱 ready。已知 blocker：Follower #1 的 F3 接頭 marginal、控制板 `...4639` SUSPECT 未複驗。

## 文件導覽

| 檔案 | 內容 |
|---|---|
| [`CONTEXT.md`](./CONTEXT.md) | 領域語言 / 命名 / 狀態語彙 glossary |
| [`AGENTS.md`](./AGENTS.md) | 硬規則（不編造成果、evidence-first） |
| [`docs/00-overview.md`](./docs/00-overview.md) | 專案總覽 / 定位 / 北極星 / W2 scope |
| [`docs/repo-map.md`](./docs/repo-map.md) | repo / reference / upstream 拓樸 + 真相歸屬 |
| [`docs/hardware-state.md`](./docs/hardware-state.md) | 四臂狀態總表（馬達/板/port/ID/齒比/校正/blocker） |
| [`docs/03-architecture.md`](./docs/03-architecture.md) | runtime / 執行邊界（Pi 5 vs 本 repo） |
| [`docs/01-setup.md`](./docs/01-setup.md) | Pi 環境 + lerobot 安裝 + 已驗證指令 |
| [`docs/02-bom.md`](./docs/02-bom.md) | BOM / v0 採購清單 |
| [`docs/learning-roadmap.md`](./docs/learning-roadmap.md) | teleop→dataset→ACT→eval 學習路線 + 安全門 |
| [`docs/reference-map.md`](./docs/reference-map.md) | leLab / SO-ARM100 / XLeRobot / 外部 repo 寶藏表 |
| [`docs/07-decisions.md`](./docs/07-decisions.md) | 輕量 ADR（命名 / 狀態模型 / 2F+2L / …） |
| [`docs/open-questions.md`](./docs/open-questions.md) | 待 Roy/PM 拍板清單 |
| [`docs/04-run-log/`](./docs/04-run-log/) · [`05`](./docs/05-failure-log.md) · [`06`](./docs/06-metrics.md) · [`08`](./docs/08-media-index.md) · [`09`](./docs/09-public-notes.md) | run-log / failure / metrics / 媒體索引 / 公開素材 |

## 上游 / 致謝（built on）

Anvil x XLeRobot 站在這些開源專案之上（皆為 read-only 參考或上游，不代表本專案身分）：

- **[MakerMods-XLeRobot](https://github.com/makermods-robotics/MakerMods-XLeRobot)** — 硬體升級設計（3D 列印件 / 機構 / BOM）。原始 README 保留於 [`README-makermods-hardware.md`](./README-makermods-hardware.md) / [`README_cn.md`](./README_cn.md)。
- **[XLeRobot](https://github.com/Vector-Wangel/XLeRobot)**（Vector Wang）— 雙臂 + 行動底盤平台原型。
- **[SO-ARM100/101](https://github.com/TheRobotStudio/SO-ARM100)**（TheRobotStudio）— 手臂規格 / CAD / BOM。
- **[LeRobot](https://github.com/huggingface/lerobot)**（HuggingFace）— 控制 / 校正 / dataset / policy 框架。
- **[LeKiwi](https://github.com/SIGRobotics-UIUC/LeKiwi)**（SIGRobotics-UIUC）— 行動底盤 + leLab web 流程。

## License

本 repo 的硬體疊加層沿用上游 [MIT License](./LICENSE)（© 2025 MakerMods）。Roy 自寫的 `docs/` 內容為本專案的紀錄與證據。

# Repo Map — Roy-XLeRobot-SO101 / AnvilBot

> 本 child repo 自身的 repo / reference / upstream topology 與「真相歸屬」分類表。給一上車的 agent 或工程師：先看懂哪些檔是 Roy 的真進度、哪些是 upstream 疊加層、哪些是 read-only 參考，避免把 MakerMods 的 README 當成 Roy 的成果。與父 repo `RoyBot-Lab/docs/repo-map.md` 互補（父表管跨機器人線，本表管這一台內部）。

---

## 0. 一句話定位

`roy4222/Roy-XLeRobot-SO101`（本機目錄 `AnvilBot`）是 RoyBot-Lab 的 child repo，性質是 **「fork 疊加層 + 證據庫」，不是 code repo**。它**不執行機器人**——真正跑 `lerobot` 的是 Pi 5；本 repo 只存 run-log / failure-log / metrics / decisions / 媒體索引。Anvil 與 Go2 / PawAi 是**平行主線**，不是其手臂子模組。

---

## 1. 三個名字 = 同一台機器人（最容易踩的坑）

| 場景 | 名稱 | 來源 |
|---|---|---|
| Obsidian / 策略筆記 | **Anvil / X0** | 父 repo 的 vault |
| GitHub 正規名（`origin`） | **`roy4222/Roy-XLeRobot-SO101`** | `git remote -v` |
| 本機目錄 | **`AnvilBot`** | `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot` |

**這層對應在 GitHub UI 看不出來**——repo 名不含 Anvil / X0。新人只看 GitHub 頁面不會知道三者是同一台，必須先讀 `CLAUDE.md`。權威編碼在 `CLAUDE.md`。

---

## 2. Remote / Repo 角色表（topology）

| 角色 | 名稱 / URL | 性質 | Roy 改不改 | 說明 |
|---|---|---|---|---|
| `origin` | `roy4222/Roy-XLeRobot-SO101` | Roy 主 fork | ✅ push 文件/證據 | Anvil/X0 的正規 GitHub 名 |
| `upstream` | `makermods-robotics/MakerMods-XLeRobot` | 硬體設計上游 | ❌ 不改（直接用 `.3mf`） | 3D 列印件 / 組裝 / 外觀升級 |
| `ref-xlerobot` | `Vector-Wangel/XLeRobot` | 平台原型參考 | ❌ read-only | 雙臂 + 行動底盤 + web control 原始平台 |
| `ref-lekiwi` | `SIGRobotics-UIUC/LeKiwi`（內含 leLab） | 軟體棧 + 行動底盤參考 | ❌ read-only | leLab = lerobot 的 web 流程；行動底盤 CAD |

> 注意：`ref-soarm100`（`TheRobotStudio/SO-ARM100`，手臂規格源頭）在 `references.repos` 由父 repo 的 vcstool 拉進 `reference/`，是 **first-class 硬體真相源**，但目前未在本 repo 的 `git remote` 設定中。具體 remote 設定以實機 `git remote -v` 為準；本表只記角色。

所有 git URL 走 HTTPS（此環境 SSH 到 GitHub 不通）。

---

## 3. 真相歸屬分類表（Truth ownership）— 本檔核心

四類，依「可信度 / 是否 Roy 進度」分。**讀任何狀態前先確認它落在哪一類。**

### 3.1 Roy 自寫真相（有實質內容，可引用為狀態）

| 路徑 | 角色 |
|---|---|
| `AGENTS.md` | 硬規則：不編造成果、不 commit secrets、evidence-first |
| `CLAUDE.md` | 專案定位 / git 紀律 / 三名字對應 |
| `docs/00-overview.md` | 目標與範圍（現況段停在 2026-06-26，待 sharpen） |
| `docs/04-run-log/2026-06-26-follower-motor-id-setup.md` | Follower #1：ID / 組裝 / bus 排障 / 校正 / 首次動作 |
| `docs/04-run-log/2026-06-27-follower2-bringup.md` | Follower #2：ID / 板 `...0335` / 校正 range |
| `docs/04-run-log/2026-06-27-leader-bringup.md` | Leader #1：6 顆混齒比 / 5V / ID 表 |
| `docs/05-failure-log.md` | F3 接頭 marginal、板 `...4639` SUSPECT |
| `docs/06-metrics.md` | follower 校正 encoder range |
| `docs/07-decisions.md` | 2F+2L 傾向、suspect 不蓋棺（2 條皆有實質內容，**禁止覆蓋**） |
| `docs/research/2026-06-26-so-arm101-assembly-runbook.md` | 對抗式查證過的組裝指南 |

### 3.2 TBD 骨架（**別當成已完成**）

| 路徑 | 狀態 |
|---|---|
| `docs/01-setup.md` | TBD placeholder |
| `docs/02-bom.md` | 表格骨架，全 TBD |
| `docs/03-architecture.md` | 標題骨架 |
| `docs/08-media-index.md` | 僅標題，索引未填 |
| `docs/09-public-notes.md` | TBD placeholder |

### 3.3 Upstream 疊加層（**非 Roy 進度，勿引用為狀態**）

| 路徑 | 內容 |
|---|---|
| `README.md` / `README_cn.md` | **MakerMods 硬體升級說明**（非 Roy 的進度／成果） |
| `LICENSE` | MIT 2025 **MakerMods**（非 Roy） |
| `3mf/` | Bambu Studio 列印專案（MakerMods） |
| `assets/`（gifs / images / mechanical） | MakerMods demo 影片 / 外觀照 / 機構照 |

> **頭號誤判風險**：打開 GitHub 看到的門面是 MakerMods upstream README，不是 Roy 的狀態。Roy 真進度全在 §3.1 的 run-log / failure-log，不在 README。

### 3.4 Read-only 參考（永不修改，`reference/` 被 gitignore）

| 路徑 | 來源 |
|---|---|
| `reference/XLeRobot` | `Vector-Wangel/XLeRobot` |
| `reference/SO-ARM100` | `TheRobotStudio/SO-ARM100` |
| `reference/leLab` | `SIGRobotics-UIUC/LeKiwi` fork |

`.gitignore` 僅一行 `reference/`——三個參考 repo 由父 repo 的 vcstool 拉取，本 repo 不 commit 它們。

---

## 4. 責任邊界（AnvilBot / XLeRobot / SO-ARM100 / leLab / LeKiwi）

| 元件 | Owner | 邊界 / 介面 |
|---|---|---|
| **AnvilBot repo（本 repo）** | Roy | 證據庫：記 bring-up / 硬體狀態 / 決策 / 失敗 / metrics。**不執行機器人**。 |
| **`roy4222/Roy-XLeRobot-SO101`（origin）** | Roy | push 本 repo 文件更新；追蹤這一台的進度。 |
| **MakerMods-XLeRobot（upstream）** | MakerMods | 硬體設計（3D CAD / 組裝 / 外觀）。Roy 直接用 `.3mf` 不改。 |
| **SO-ARM100（`reference/`）** | TheRobotStudio | 手臂**規格真相源**：CAD / STL / URDF / BOM / 馬達型號 / 電壓 / pinout。Roy 讀規格、不改。**first-class**。 |
| **leLab / LeRobot（Pi 上）** | SIGRobotics-UIUC / HuggingFace | Pi 上的控制軟體：`lerobot-setup-motors` / `lerobot-calibrate` / `lerobot-teleoperate`。Roy 呼叫 CLI，**不 fork 進本 repo**。 |
| **XLeRobot（`reference/`）** | Vector-Wangel | 平台原型（雙臂 + 行動底盤 + web control）。Roy 的控制走 LeRobot 而非 XLeRobot 原生棧 → **glance 級**。 |
| **LeKiwi（`reference/`，含 leLab）** | SIGRobotics-UIUC | 行動底盤 CAD + holonomic 運動學。mobile base 升級時才用 → **長期**。 |
| **Pi 5（`pi5` / 100.104.92.26）** | Roy | 跑 lerobot venv、存校正快取、產 log。不在本 repo，走 Tailscale SSH。 |
| **Leader / Follower 硬體** | Roy | 實體資產，用序號 / 板 ID 在 run-log 追蹤（`...3925` / `...0335` / `...0371` / `...4639`）。 |

**參考分級小結**：SO-ARM100 = first-class（硬體規格）；leLab = first-class（軟體棧，但 W3 才引入，**W2 不引入**——leLab 需 Python 3.12+，與 Pi 的 3.10.20 / lerobot 0.4.4 Seeed fork 衝突）；XLeRobot / LeKiwi = glance / 長期。

---

## 5. 執行環境（不在本 repo）

| 項目 | 值 | 出處 |
|---|---|---|
| 主機 | Pi 5 @ Tailscale `pi5` / 100.104.92.26 | run-log |
| venv | `~/lerobot/.venv` | run-log |
| Python / lerobot | 3.10.20 / lerobot 0.4.4（Seeed fork） | `2026-06-26-follower-motor-id-setup.md` |
| 控制路徑 | feetech SDK → board → 6 馬達 daisy-chain | run-log |

本 repo 是純證據庫，**不在這裡執行任何 lerobot 指令**。

---

## 6. 證據紀律（引用狀態前務必守住）

1. **「USB 能列舉」≠「servo bus 健康」**。板 `...4639` 能在 `/dev/serial/by-id/` 列舉，但 `scan_port` 回 `{}`、`RX_BYTES 0`（`05-failure-log.md`）——這是 bus 不健康的反例。**沒有 scan / calibration / teleop evidence，不可稱任何 bus「healthy」。**
2. **Leader #1 bus 健康仍 TBD**：`2026-06-27-leader-bringup.md` 目前只到「逐顆寫 ID」，組裝 + 整串 `scan_port [1..6]` + 校正皆**待做 / 待驗證**（同檔 L32–36）。掃描期間只見單顆回應（`Motors found {3: 777}`），不等於整串健康。不可宣稱 leader ready。
3. **Leader 逐關節齒比一律用 run-log 值**（下表，已對 `2026-06-27-leader-bringup.md` L17–26 逐行核對），**不抄被組裝 runbook 明文推翻的舊對應**（runbook §4 已標示舊「ID1:1/345…」為錯誤）。

| 關節 (ID) | 功能 | 齒比 / 型號 |
|---|---|---|
| L1 | shoulder_pan 底座 | **1/191 C044** |
| L2 | shoulder_lift 肩抬 | **1/345 C001** |
| L3 | elbow_flex 肘 | **1/191 C044** |
| L4 | wrist_flex 腕俯仰 | **1/147 C046** |
| L5 | wrist_roll 腕滾 | **1/147 C046** |
| L6 | gripper 握把扳機 | **1/147 C046** |

分布 = 1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。逐關節狀態 / 四手總表見 `docs/hardware-state.md`（W2 P0 交付，建立中）。

4. **電源安全**：leader 7.4V 馬達**必配 5V**，誤接 12V 直接燒馬達（`2026-06-27-leader-bringup.md` L7）。

---

## 7. 與父 repo 的關係

- 父 repo `RoyBot-Lab/docs/repo-map.md` = 跨五條機器人線的權威總表（repo 名 / 分支 / 可見度 / upstream）。具體值衝突時以**父 repo-map 為準**。
- 本檔只管 `Roy-XLeRobot-SO101` 一台內部的 remote / reference / 真相歸屬，是父表的下鑽補充。
- 父 repo 的角色摘要另見 `RoyBot-Lab/robots/xlerobot-so101.md`。

---

## 8. 已知缺口（TBD / 待驗證）

- §3.2 五份骨架（01-setup / 02-bom / 03-architecture / 08-media-index / 09-public-notes）尚未填實質內容。
- `ref-soarm100` 是否已加入實機 `git remote`：以 `git remote -v` 為準（本表記為 first-class 角色，remote 連線 TBD）。
- 控制板 `...4639` SUSPECT 未複驗；Follower #1 的 F3 接頭 marginal 待修；Leader #1 組裝 / 整串 scan / 校正待做；Leader #2 未開始。以上逐項狀態以 run-log / failure-log 為準，勿在本檔擴寫成已完成。
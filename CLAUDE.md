# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> 一律用繁體中文回應。

## 這個 repo 是什麼（先讀這段）

這是 **Anvil / X0**（低成本輪式雙臂 Physical AI 平台）的 Child Repo。三個名字指同一台機器人：平台在 Obsidian 叫 **Anvil / X0**、GitHub 正規名是 **`roy4222/Roy-XLeRobot-SO101`**（`origin`）、本機目錄叫 **`AnvilBot`**。它落在 `RoyBot-Lab/src/AnvilBot`，**被主 repo gitignore、由 vcstool 拉進來**，本身是獨立 git repo。

本 repo 是 **fork-overlay + 證據 repo，不是 code repo**：

- `README.md` / `README_cn.md` / `3mf/` / `assets/` / `LICENSE` 來自上游 **MakerMods-XLeRobot**（硬體升級設計）——描述的是 MakerMods 的機構件與 BOM，**不是 Roy 寫的程式**，也不是 Roy 的進度真相。
- Roy 自己加的只有兩層：**`AGENTS.md`（硬規則）** 與 **`docs/`（證據 + 決策骨架）**。要找 Roy 的狀態/決策一律看 `docs/`。
- `reference/`（XLeRobot / SO-ARM100 / leLab）是 **gitignored 的 read-only 上游**，只供查閱程式與組裝文件，**永遠不改、不是真相來源**。

因此本 repo **沒有 build / test / lint**。編輯動作幾乎都是改 Markdown。真正驅動機器人的程式 **不在這裡**——它是 HuggingFace **LeRobot（Seeed-Projects fork, 0.4.4）** 跑在 **Raspberry Pi 5**（Tailscale `pi5` / 100.104.92.26，venv `~/lerobot/.venv`）上。**本 repo 只記錄發生過的事（evidence），不執行機器人。**

## 進場閱讀順序與真相歸屬

- **Roy 的目標 / 範圍 / 現況** → `docs/00-overview.md`
- **路線與選型決策（輕量 ADR）** → `docs/07-decisions.md`
- **實機做了什麼、結果、異常** → `docs/04-run-log/<date>-*.md`（含 motor-id / bring-up 逐筆）
- **硬體組裝 how-to / 安全細節** → `docs/research/2026-06-26-so-arm101-assembly-runbook.md`（已做對抗式查證，被推翻的說法不採信）
- **量測數字** → `docs/06-metrics.md`；**失敗/卡關** → `docs/05-failure-log.md`；**大檔索引** → `docs/08-media-index.md`；**可公開文章素材** → `docs/09-public-notes.md`
- 多數編號 doc（`01-setup` / `02-bom` / `03-architecture`）目前是 **TBD 骨架**，尚未填。

主 repo（RoyBot-Lab）只放 manifest / 索引；**單台機器人的技術細節寫在這裡，不要往主 repo 塞**。北極星 / roadmap 那類策略敘事留在 Obsidian MOC，**不要搬進本 repo**（repo 只放技術細節 + 證據）。

## 操作機器人的實際指令（跑在 Pi 上，不在本 repo）

de-facto 的「build/run」就是 Pi 上的 lerobot CLI（標準流程，port/id 換成自己的）：

```bash
lerobot-find-port                                                          # 認 /dev/ttyACM* 埠
lerobot-setup-motors  --robot.type=so101_follower --robot.port=/dev/ttyACM0   # follower：逐顆寫 ID（夾爪 6→1）
lerobot-setup-motors  --teleop.type=so101_leader  --teleop.port=/dev/ttyACM0  # leader：注意是 --teleop.* 前綴
lerobot-calibrate     --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=roy_follower
lerobot-teleoperate   ...                                                   # leader → follower 遙操作（待雙臂就緒）
```

bus 健檢（唯讀、不會動馬達）：`FeetechMotorsBus.scan_port("/dev/ttyACM0")`（`from lerobot.motors.feetech import FeetechMotorsBus`）。認板用 `ls /dev/serial/by-id/`（**勿信 dmesg 歷史序號**）。Pi 上互動式 CLI 用 `tmux` 跑、`capture-pane -p` 讀畫面。

## 硬體真相（會反覆用到，搞錯會燒設備）

- **🔴 電壓鐵則**：本套件是 **SO-101 Pro**——**Leader = 7.4V 馬達 → 接 5V 電源；Follower = 12V 馬達 → 接 12V 電源**。兩顆 DC 接頭外觀近似，**12V 誤插到 Leader 會直接燒馬達**。通電前先確認電源標籤。
- **馬達 ID 必須一次一顆**：出廠 ID 相同，先串聯再批量設定會 ID 衝突。逐顆接板寫 ID 1~6（夾爪優先 6→1），全設完才 daisy-chain。
- **Leader 用混齒比**（分布：1× 1/345 + 2× 1/191 + 3× 1/147），裝錯關節會力矩不匹配；逐關節對應以 `docs/04-run-log/2026-06-27-leader-bringup.md` 為準。**Follower 全 6 顆 1/345**，無裝錯問題。
- **系統組態（傾向，未 LOCKED）**：主線 **2 follower + 2 leader** 標準 leader→follower；**不做「四手霸王」**（4-follower / 24-DoF 自訂）；**Quest 2 VR teleop 列 phase-2**。細節見 `docs/07-decisions.md`。
- **已知硬體問題**：第一隻 follower 的 F3 接頭 marginal、寫入時偶發掉封包（`Incorrect/no status packet`），靠 `num_retry` 繞過但待徹底重壓/換接頭。控制板 `...4639` 標 SUSPECT 待複驗。

## 硬規則（`AGENTS.md`，違反即停手）

1. **不准編造實機成功**：沒有 run-log evidence 的動作，不寫成已完成；不杜撰 metrics / 照片 / 影片。未知具體值寫 **TBD / 待驗證**。
2. **不 commit secrets**：`.env` / API key / token / 帳密 / 校內競賽私密資料。
3. **大檔只寫索引**：影片 / ROS bag / dataset 只在 `docs/08-media-index.md` 留 index，不直接 commit。
4. raw evidence 進 `docs/04-run-log/`、`docs/05-failure-log.md`、`docs/06-metrics.md`；可公開素材進 `docs/09-public-notes.md`（須能對回實際 evidence）。
5. **不靜默刪檔、不自動部署、不把 Proposal 當定論**。

## Git

- **Remotes**：`origin` = `roy4222/Roy-XLeRobot-SO101`（Roy 會 push）；`upstream` = `makermods-robotics/MakerMods-XLeRobot`；`ref-xlerobot`（Vector-Wangel）、`ref-lekiwi`（SIGRobotics）= 純參考上游。
- **同步上游只用 merge**：`git fetch upstream && git merge upstream/main`。**不 rebase、不 `--force`、不要按 GitHub 網頁的「Sync fork」按鈕**（會用 rebase/force 破壞本地分叉）。
- **不准** `git reset --hard` / `git clean` / `git push --force`。
- 所有 git URL 走 **HTTPS**（此環境 SSH 到 GitHub 不通）。本機在 WSL，從 Windows 端看可能出現 filemode 假 dirty——git 操作盡量都在 WSL 內做。
- `reference/` 已被 `.gitignore` 擋掉，不會進版控。

# 01 — Setup

> Anvil x XLeRobot 的環境、相依、安裝與已驗證 run command。**真正的執行環境在 Pi 5，不在本 repo**（本 repo 只存證據）。架構/邊界見 `docs/03-architecture.md`、四臂硬體狀態見 `docs/hardware-state.md`。硬體聲明一律引用 `docs/04-run-log/` 檔名；未驗證者標 TBD。

## 環境需求

| 項目 | 值 | 出處 |
|---|---|---|
| 執行主機 | Raspberry Pi 5（Debian 13 trixie / aarch64） | run-log |
| 網路 | Tailscale `pi5` / `100.104.92.26`（本機 WSL 同 tailnet） | run-log |
| venv | `~/lerobot/.venv`（**Python 3.10.20**） | `04-run-log/2026-06-26-follower-motor-id-setup.md` |
| LeRobot | **0.4.4（Seeed-Projects fork）**，非 upstream HF main | 同上 |
| 馬達 SDK | `feetech` / `scservo_sdk`（pip，OK） | 同上 |
| 權限 | 使用者在 `dialout` 群組，免每次 `sudo chmod` | — |
| 電源 | follower 12V、leader **5V**（7.4V 馬達；12V 直插 leader 燒馬達） | `04-run-log/2026-06-27-leader-bringup.md` |

> **WSL → Pi 工作流**：本機（WSL）只編輯 repo / 寫腳本；**不要直接在 Pi 上寫長期腳本**——先在 WSL repo 寫，再同步到 Pi（rsync / scp）。

## 連線 Pi

```bash
ssh roy422@100.104.92.26        # 或 Tailscale 名 pi5（首次 host key 可能要確認）
```

## 安裝（Pi 上，已完成；此處為紀錄）

```bash
source ~/lerobot/.venv/bin/activate      # venv 已建（Python 3.10.20）
# lerobot 0.4.4 Seeed-Projects fork + feetech extra 已安裝；使用者已在 dialout 群組
```

> 從零重建的完整步驟尚未整理 = **TBD**。目前環境已可用；重裝前先備份校正快取 `~/.cache/huggingface/lerobot/calibration/`。

## Build / Run（已驗證的 de-facto 指令；皆在 Pi venv 內跑）

本 repo 無 build / test / lint。操作機器人的指令跑在 Pi：

```bash
lerobot-find-port                                                              # 認 /dev/ttyACM* 埠
lerobot-setup-motors --robot.type=so101_follower --robot.port=/dev/ttyACM0     # follower 逐顆寫 ID（夾爪 6→1）
lerobot-setup-motors --teleop.type=so101_leader  --teleop.port=/dev/ttyACM0    # leader（--teleop.* 前綴、5V 電源）
lerobot-calibrate    --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=roy_follower
# bus 健檢（宣稱 healthy 的唯一依據；唯讀、全鮑率）
python -c "from lerobot.motors.feetech import FeetechMotorsBus; print(FeetechMotorsBus.scan_port('/dev/ttyACM0'))"
# lerobot-teleoperate ...   # leader→follower（W2 里程碑，尚未跑 → TBD）
```

- 認板：`ls /dev/serial/by-id/`（**勿信 dmesg 歷史**）。
- 互動式 CLI：Pi 上用 `tmux` 跑、`tmux send-keys -t <s> Enter` 送鍵、`capture-pane -p` 讀畫面（換馬達/擺位需人手）。
- 校正快取：`~/.cache/huggingface/lerobot/calibration/robots/so_follower/<id>.json`。

> 證據紀律：「USB 能列舉」≠「bus 健康」；沒有 `scan_port` 全綠 + 校正 + teleop evidence 不可稱 ready。完整指令矩陣與四臂狀態見 `docs/03-architecture.md` §5 與 `docs/hardware-state.md`。

# Run Log — 2026-06-27 第二隻 Follower Bring-up

**階段**：Stage 0 Bring-up（2F+2L 基線，第二隻 follower）
**裝置**：SO-101 Follower #2（6× STS3215，model 777）
**主機**：Raspberry Pi 5（Tailscale `pi5` / 100.104.92.26）
**環境**：`~/lerobot/.venv`（lerobot 0.4.4 Seeed fork）
**控制板**：**二手板 `5AAF220335`（結尾 `...0335`）= 第三塊板**，非昨天的好板 `...3925`，也非 suspect `...4639`。USB `/dev/ttyACM0`，bus 端正常。
**電源**：12V（follower）。

## 馬達 ID 設定（✅ 完成，工具逐顆確認）

執行官方互動式工具，一次一顆、夾爪優先（6→1）：

```
lerobot-setup-motors --robot.type=so101_follower --robot.port=/dev/ttyACM0
```

| 提示名稱 | 設定 ID | 標籤 | 工具回報 |
|---|---|---|---|
| gripper | 6 | F6 / 夾爪 | `'gripper' motor id set to 6` |
| wrist_roll | 5 | F5 / 腕滾 | `'wrist_roll' motor id set to 5` |
| wrist_flex | 4 | F4 / 腕俯仰 | `'wrist_flex' motor id set to 4` |
| elbow_flex | 3 | F3 / 肘 | `'elbow_flex' motor id set to 3` |
| shoulder_lift | 2 | F2 / 肩抬 | `'shoulder_lift' motor id set to 2` |
| shoulder_pan | 1 | F1 / 底座 | `'shoulder_pan' motor id set to 1` |

指令正常結束，6 顆 ID 全部寫入成功。這批是**二手馬達**（寫入前已有舊 ID，例如第一顆讀到 ID 3），工具直接覆寫，無問題。

## 異常與處理：首掃 Input voltage error（已解）

接好第一顆、上電後首次唯讀 `scan_port` 回報：

```
Some motors found returned an error status:
{3: '[RxPacketError] Input voltage error!'}
SCAN {}
```

馬達回報輸入電壓超出規格。**使用者更換電源後復掃乾淨**：

```
Motors found for baudrate=1000000: {3: 777}
SCAN {1000000: [3]}
```

> 具體根因（原電源電壓不符／接觸不良／供電不足）**未深究，TBD**。教訓：上電後先 `scan_port`，看到 voltage error 一律先斷電查電源，別硬寫。

## 組裝 → 掃描 → 校正（✅ 完成）

1. **機構組裝完成**（ID1 底座 → ID6 夾爪），F3 佈線無歪針問題。
2. **整串 `scan_port` 連掃 5 次全綠**（無 voltage error、無掉封包）：`SCAN#1~5 {1000000: [1,2,3,4,5,6]}`（6 顆 model 777）。
3. **校正完成**：`lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=roy_follower2`
   - 各軸 range（counts）：pan 983–3174、lift 847–3205、elbow 854–3092、wrist_flex 934–3244、gripper 949–2479；`wrist_roll` 整圈自動記錄（手掃時排除）。
   - 存檔：`~/.cache/huggingface/lerobot/calibration/robots/so_follower/roy_follower2.json`
   - **全程無 `Incorrect/no status packet`** —— 二手板 `...0335` 寫入路徑乾淨，沒有第一隻 F3 那種 marginal 接頭掉封包問題。

**Follower #2 bring-up 狀態**：馬達 ID ✅ / 組裝 ✅ / bus ✅（5/5）/ 校正 ✅。尚未做首次動作 demo 與 teleop。

## 下一步

- （可選）首次動作 demo（仿第一隻 `~/demo_follower_v2.py` / `~/reach_follower.py`，改 port/id）目視確認會動、方向正確。
- 2 隻 leader：設 ID（`--teleop.type=so101_leader`，**5V 電源**，混齒比放對關節）→ 組裝 → 校正。
- 之後 `lerobot-teleoperate` 跑 leader→follower。

## 辨識小抄

- 認板：`ls /dev/serial/by-id/`（勿信 dmesg 歷史）。本台 follower #2 = `...0335`。
- bus 健檢：`FeetechMotorsBus.scan_port("/dev/ttyACM0")`（`from lerobot.motors.feetech import FeetechMotorsBus`），唯讀、全鮑率。
- 互動式 CLI：Pi 上 `tmux` 跑、`tmux send-keys -t <s> Enter` 送鍵、`capture-pane -p` 讀畫面。

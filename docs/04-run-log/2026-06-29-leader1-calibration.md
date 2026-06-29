# Run Log — 2026-06-29 Leader #1 Bus 健檢 + 校正

**階段**：Stage 0.5（leader station）—— 接續 `2026-06-27-leader-bringup.md`（當時只寫了 ID，組裝/bus/校正皆 TBD）
**裝置**：SO-101 Leader #1（6× STS3215 混齒比，7.4V 馬達）
**主機**：Pi 5（Tailscale `pi5`），venv `~/lerobot/.venv`（lerobot 0.4.4 Seeed fork）
**控制板**：板#4 `5AAF220371`（`...0371`），USB `/dev/ttyACM0`（`/dev/serial/by-id/usb-1a86_USB_Single_Serial_5AAF220371-if00`）
**電源**：**5V**（leader 7.4V 馬達配 5V；切勿接 12V）
**操作方式**：WSL 端 SSH 進 pi5，於 tmux `setupL` session 跑 lerobot CLI，`capture-pane` 讀畫面、`send-keys` 送 ENTER（使用者實體操作手臂）。

## 1. Bus 健檢 `scan_port`（✅ 全綠，唯讀）

使用者回報「已接好第一隻 leader」（機構組裝 + daisy-chain + 插板 + 5V）後，宣稱 ready 前先唯讀掃描：

```python
from lerobot.motors.feetech import FeetechMotorsBus
FeetechMotorsBus.scan_port("/dev/ttyACM0")
```

輸出：

```
Motors found for baudrate=1000000: {1: 777, 2: 777, 3: 777, 4: 777, 5: 777, 6: 777}
SCAN_RESULT: {1000000: [1, 2, 3, 4, 5, 6]}
```

→ 6 顆 ID 1–6 全在 bus 上回應（baudrate 1000000）。`model 777` = STS3215 通用韌體型號（齒比是實體屬性、不在此 register，屬正常）。**這把 `2026-06-27-leader-bringup.md` L32-36 與 `hardware-state.md` 的「Leader #1 bus 健康 TBD」正式翻為 verified。**

## 2. 校正 `lerobot-calibrate`（✅ 完成，json 已存）

```bash
~/lerobot/.venv/bin/lerobot-calibrate \
  --teleop.type=so101_leader --teleop.port=/dev/ttyACM0 --teleop.id=roy_leader
```

流程：`SOLeader connected` → 擺中段設 homing offset（送 ENTER）→ 手動掃各軸極限（`wrist_roll` 除外，自動記整圈）→ 送 ENTER 停止 → 存檔 → 乾淨 disconnect。

末端為 **handle 扳機（非夾爪）**，`gripper` 軸即扳機行程，故幅度較其他軸短，屬正常。

### 最終校正值（`roy_leader.json`，含 homing offset）

存檔路徑：`~/.cache/huggingface/lerobot/calibration/teleoperators/so_leader/roy_leader.json`
（注意：leader/teleop 落在 `…/calibration/**teleoperators/so_leader**/`，與 follower 的 `…/robots/so_follower/` 不同目錄。）

| 軸 | ID | homing_offset | range_min | range_max | 幅度 |
|---|---:|---:|---:|---:|---:|
| shoulder_pan | 1 | 620 | 982 | 3147 | 2165 |
| shoulder_lift | 2 | -1977 | 806 | 3117 | 2311 |
| elbow_flex | 3 | 1396 | 858 | 3119 | 2261 |
| wrist_flex | 4 | 812 | 846 | 3111 | 2265 |
| wrist_roll | 5 | -1947 | 0 | 4095 | 4095（整圈自動記錄）|
| gripper（handle 扳機）| 6 | -585 | 871 | 2084 | 1213 |

全部 `drive_mode = 0`。各軸 range 與兩隻 follower 同量級（合理）：follower #1 shoulder_pan 955–3140 / Leader #1 982–3147 等。

## 3. 待驗證 / 下一步

- ✅ 本次驗到：Leader #1 bus `[1..6]`、校正 `roy_leader` 存檔成功（json 上列）。
- ⚠️ **齒比↔關節對應正確性「尚未」於此步驗證**：校正只證明各馬達在 bus 上、各關節有真實行程；混齒比是否裝在正確關節（裝錯＝力矩不匹配）要到 **teleop 實際施力**才看得出來。
- **下一步＝第一條 leader→follower 空中動作 teleop**：建議 **Leader #1 → Follower #2（乾淨 bus；避開 Follower #1 的 F3 marginal 接頭）**，`lerobot-teleoperate`，無負載、無 policy，錄影片 + log 當證據。
- 之後：**Leader #2** 整條 pipeline（同流程）；朝 **Stage 0 pass**（單臂 teleop 連續 5 分鐘無失控 + 影片/log）。

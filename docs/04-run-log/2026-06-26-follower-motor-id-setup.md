# Run Log — 2026-06-26 Follower 馬達 ID 設定

**階段**：Stage 0 Bring-up（組裝前置）
**裝置**：SO-101 Pro Follower（6× STS3215-C018, 12V, 1:345）
**主機**：Raspberry Pi 5（Debian 13 trixie, aarch64, Tailscale `pi5` / 100.104.92.26）
**環境**：`~/lerobot/.venv`（Python 3.10.20, lerobot 0.4.4, Seeed-Projects fork, feetech/scservo_sdk OK）
**控制板**：Seeed XIAO 伺服控制板，USB serial `/dev/ttyACM0`（晶片 1a86）
**電源**：12V 電源供應器（C018 為 12V 馬達）

## 做了什麼

執行官方互動式工具，**一次一顆**逐顆寫入 ID（工具固定夾爪優先、ID6→ID1）：

```
lerobot-setup-motors --robot.type=so101_follower --robot.port=/dev/ttyACM0
```

## 結果（工具實際輸出，逐顆確認）

| 提示名稱 | 設定 ID | 標籤 | 工具回報 |
|---|---|---|---|
| gripper | 6 | F6 / 夾爪 | `'gripper' motor id set to 6` |
| wrist_roll | 5 | F5 / 腕滾 | `'wrist_roll' motor id set to 5` |
| wrist_flex | 4 | F4 / 腕俯仰 | `'wrist_flex' motor id set to 4` |
| elbow_flex | 3 | F3 / 肘 | `'elbow_flex' motor id set to 3` |
| shoulder_lift | 2 | F2 / 肩抬 | `'shoulder_lift' motor id set to 2` |
| shoulder_pan | 1 | F1 / 底座 | `'shoulder_pan' motor id set to 1` |

指令正常結束，6 顆 ID 全部寫入成功，馬達已逐顆貼標籤。

## 失敗 / 異常

無（NONE）。本次無觸發任何失敗分類。

## 待驗證 / 下一步

- 馬達 ID **寫入成功已驗證**（工具回報）；但「ID 正確對應到組裝後的關節」需待機構組裝完成後，於整臂校正（`lerobot-calibrate`）時驗證。
- 下一步：依官方圖示組裝 follower 機構（ID1→底座 … ID6→夾爪），完成後做 `lerobot-calibrate --robot.type=so101_follower`。
- Leader（7.4V，5V 電源，混合齒比）尚未設定。

## 後續（同日 20:xx）：組裝 → bus 故障排除 → 校正完成

1. **機構組裝完成**（follower）。
2. **bus 故障排除**（詳見 `05-failure-log.md` 2026-06-26 兩筆）：組裝後整支掃不到馬達，逐段隔離後確認＝**F3 接頭歪針間歇拉死匯流排** + 一條疑似反接的延長線；第一張控制板 `...4639` 列為 suspect 待複驗，改用第二張板 `...3925`。
   - 診斷工具：`FeetechMotorsBus.scan_port()` 當回饋迴路；`ls /dev/serial/by-id/` 即時辨識板子（勿用 dmesg 歷史）。
   - 逐顆單獨認證 6 顆馬達**全部 good**；喬正 F3 針腳後整串 `scan_port → {1000000: [1,2,3,4,5,6]}` 穩定（連掃 3 次一致）。
3. **校正完成**：`lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=roy_follower`
   - 首次跑在寫 Lock 到 id6 時報 `Incorrect status packet`（偶發雜訊，無重試）；重跑即過。
   - 各關節 range（counts）：pan 955–3140、lift 837–3208、elbow 1019–3228、wrist_flex 937–3208、wrist_roll 0–4095（整圈，記錄階段排除屬正常）、gripper 936–2358。
   - 存檔：`~/.cache/huggingface/lerobot/calibration/robots/so_follower/roy_follower.json`

**Follower bring-up 狀態**：馬達 ID ✅ / 組裝 ✅ / bus ✅ / 校正 ✅ / **首次自主動作 ✅（已目視確認）**。尚未做 teleop（需 leader）。

## 首次自主動作（已目視確認）

用 `SO101Follower.send_action()` 自寫慢速腳本（normalized 目標、`max_relative_target=15`、逐步插值、結束強制關扭力），**無 leader、無 policy**，直接命令 follower 動作：

- **demo**：回正 → 揮手（shoulder_pan ±25）→ 夾爪開合（0~100）→ 點頭（wrist_flex ±20）→ 回 home。位置遙測 START（shoulder_lift −94.6 / elbow 100，扭力關時垂下）→ END（≈0）。**Roy 目視確認手臂確實如此動作。**
- **reach 來回 ×2**：home → 伸出（shoulder_lift −40 / elbow −35 / wrist +20）→ 收回 home，重複 2 次，END≈home。
- 腳本：`~/demo_follower_v2.py`、`~/reach_follower.py`（在 Pi）。

### ⚠️ 已知問題：bus 偶發掉封包（待修，HARDWARE）
- `lerobot-calibrate` 與 `robot.connect()` 的 torque 寫入曾分別在 id6 / id1 / **id3** 報 `Incorrect status packet` / `no status packet`；**id3（F3，先前歪針那顆）為最常重現點**。
- 靜態 `scan_port` 穩定（連掃 3 次 [1..6]），但**需要回應的寫入會間歇失敗**；`num_retry=5` 或 connect 自動重試即可繞過 → 典型**接頭邊緣接觸**。
- **待辦：徹底重壓/更換 F3 接頭（或該段線）**，否則 teleop/錄資料會不穩；不可長期靠重試硬扛。

## 待辦
- 複驗第一張板 `...4639` 是否真壞（好馬達+好線單獨測）。
- Leader（7.4V / 5V 電源 / 混合齒比）設 ID + 組裝 + 校正。
- 之後才能 `lerobot-teleoperate` 做 leader→follower。

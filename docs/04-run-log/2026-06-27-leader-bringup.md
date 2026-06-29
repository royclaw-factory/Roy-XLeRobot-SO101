# Run Log — 2026-06-27 第一隻 Leader Bring-up

**階段**：Stage 0 Bring-up（2F+2L 基線，第一隻 leader）
**裝置**：SO-101 Leader #1（6× STS3215 混齒比，7.4V 馬達）
**主機**：Pi 5（Tailscale `pi5` / 100.104.92.26）
**控制板**：**第四塊板 `5AAF220371`（`...0371`）= leader #1 用**，USB `/dev/ttyACM0`，bus 端正常。
**電源**：**5V**（leader 7.4V 馬達配 5V；**切勿接 12V，會燒**）。

## 馬達 ID + 齒比對應（✅ 完成，工具逐顆確認）

```
lerobot-setup-motors --teleop.type=so101_leader --teleop.port=/dev/ttyACM0
```

夾爪優先 6→1，**各 ID 須對應正確齒比**（裝錯關節會力矩不匹配）：

| 提示名稱 | ID | 標籤 | 齒比 / 型號 | 工具回報 |
|---|---|---|---|---|
| gripper | 6 | L6 / 握把扳機 | 1/147 C046 | `'gripper' motor id set to 6` |
| wrist_roll | 5 | L5 / 腕滾 | 1/147 C046 | `'wrist_roll' motor id set to 5` |
| wrist_flex | 4 | L4 / 腕俯仰 | 1/147 C046 | `'wrist_flex' motor id set to 4` |
| elbow_flex | 3 | L3 / 肘 | 1/191 C044 | `'elbow_flex' motor id set to 3` |
| shoulder_lift | 2 | L2 / 肩抬 | 1/345 C001 | `'shoulder_lift' motor id set to 2` |
| shoulder_pan | 1 | L1 / 底座 | 1/191 C044 | `'shoulder_pan' motor id set to 1` |

齒比分布 = 1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。這批亦為**二手馬達**（首顆讀到舊 ID 3），工具直接覆寫，無問題。

## 異常與處理：首掃 SCAN {}（已解）

接好板 `...0371`、馬達後首次唯讀掃描回 `SCAN {}`（一顆都沒回應、無 voltage error）＝典型**沒供電／馬達沒接好**。使用者確認 5V 電源 + 5264 接頭後復掃乾淨：`Motors found {3: 777}` / `SCAN {1000000: [3]}`。

## 待驗證 / 下一步

- 機構組裝（ID1 底座 → ID6；**末端為握把 handle，非夾爪**）—— **待做**。
- 組裝＋ daisy-chain ＋ 5V 後整串 `scan_port` 應回 `[1,2,3,4,5,6]` —— **待驗證**。
- `lerobot-calibrate --teleop.type=so101_leader --teleop.port=/dev/ttyACM0 --teleop.id=roy_leader` —— **待做**。
- **第二隻 leader** —— 待做（同流程）。
- 之後 `lerobot-teleoperate` 跑 leader→follower。

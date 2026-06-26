# 07 — Decisions

> 設計 / 選型 / 路線決策紀錄（輕量 ADR）。

## 2026-06-26 — 系統組態：主線採「2 follower + 2 leader」（傾向，待 Roy 最終拍板）

- 背景：手上有 2 組手臂＝2 follower（C018 12V，1/345）+ 2 leader（7.4V，混齒比 1/147・1/191・1/345）。Roy 提案三選一：(a) 2F+2L 標準 leader→follower 遙操作、(b) 四手霸王（4 隻都當 follower、24-DoF、Quest 2 VR 控）、(c) Quest 2 VR 直控雙 follower。
- 決定（傾向）：**主線 2F+2L**；**Quest 2 VR teleop 列 phase-2 spike**；**不做四手霸王**。
- 理由：
  - leader 馬達是低扭力、好手推、混齒比，**為當 leader 而設計**；當 follower 會變「2 強 2 弱」、齒比不一致。
  - XLeRobot 官方硬體路線就是 2 follower；leader 用於雙臂 leader-follower 操作。
  - 四手霸王＝自訂機構/供電/碰撞/action space/校正/資料格式全要自做，與「先跑通 manipulation learning pipeline」目標不對齊。
  - 真機 VR teleop 官方 code 仍 coming soon（需自行 port），變因多；leader→follower 是最穩、最好 debug、最適合收資料的路。
- 狀態：傾向方向，未鎖定；Roy 確認後改為 LOCKED。

## 2026-06-26 — Follower bring-up 用好板，疑似壞板列 suspect 不蓋棺

- 背景：組裝後 bus 全掃不到，逐段隔離；控制板有 2 塊。
- 決定：用好板 `...3925` 完成 follower bring-up；第一張板 `...4639` 貼 **SUSPECT**、暫停使用，待「單顆馬達 + 好線 + scan」複驗才結案。
- 理由：USB 能列舉 ≠ servo bus 端健康；證據顯示 `...4639` 在好馬達/好線下仍讀不到，但可能曾受 F3 歪針短路波及，未用乾淨組合複驗前不判死刑。
- 待辦：徹底重壓/更換 **F3 接頭**（仍是 bus 偶發掉封包主嫌）；複驗 `...4639`。

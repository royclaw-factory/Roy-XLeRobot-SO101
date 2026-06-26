# 00 — Overview

> Roy-XLeRobot-SO101 專案總覽。主 repo 的角色摘要見 `RoyBot-Lab/robots/xlerobot-so101.md`，此檔放本 child repo 自身視角。

## 這是什麼

Roy 的 **SO-ARM101 / XLeRobot 機械手臂線** child repo——低成本 Physical AI / manipulation learning 的學習平台。硬體採 SO-101 Pro 版（follower 12V C018、leader 7.4V 混齒比）+ LeKiwi 行動底盤；軟體跑 HuggingFace LeRobot（Seeed fork）。本 repo 放該機器人線的**技術細節與證據**（run log / failure log / metrics / 媒體索引）。

## 目標

跑通**第一條 SO-101 manipulation learning pipeline**：teleop / leader→follower 收資料 → 訓練 ACT baseline → 真機推動式物品分類 eval → 失敗分析 → roy422.dev 草稿。

## 非目標 / 範圍邊界

- 不把單台機器人的大量技術細節寫進主 repo（主 repo 只放 manifest / 索引）。
- 不做「四手霸王」（4 follower、24-DoF 自訂系統）——列為 phase-2 stretch，非現階段（見 `07-decisions.md`）。
- Quest 2 VR teleop 列為 phase-2 spike，待雙 follower 穩定後再試。

## 現況（2026-06-26）

第一隻 **follower bring-up 已過第一關**：6 顆馬達 ID ✅ / 機構組裝 ✅ / bus ✅ / 校正 ✅ / 首次自主動作 ✅（已目視確認）。詳見 `docs/04-run-log/2026-06-26-follower-motor-id-setup.md`。尚未做 teleop（需 leader）。

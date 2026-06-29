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

## 2026-06-29 — W2 範圍護欄：本週只做「收斂真相 + 補完 leader + 第一條 teleop」（傾向/待 LOCKED）

- 背景：硬體進度真實（2 follower 已 bring-up、leader #1 已寫 ID），但 repo 可讀真相落後實機，且周邊「可用菜單」（VR、mobile base、VLA、world-model、Go2 整合）很多，容易範圍蔓延。W2 必須先把力氣收斂在最穩、能直接拿里程碑的那條路。
- 決定：W2 **IN-scope**＝補完 Leader #1（組裝→`scan_port` 驗 bus→校正）、跑出第一條 leader→follower 空中動作 teleop、收斂 W2 P0 文件（四手狀態總表、v0 BOM、媒體索引、sharpen overview）。以下一律 **NOT-this-week**：

  | 排除項 | 性質 | 為何不進 W2 | 歸屬 |
  |---|---|---|---|
  | 四手霸王（4×follower 24-DoF） | 自訂機構/控制 | 與「先跑通 pipeline」不對齊（見 2026-06-26 條目） | phase-2，未排程 |
  | VR 真機 teleop（Phosphobot / SO-101-VR-Control / Quest） | 資料採集替代路 | 官方真機 code 仍需自 port、變因多；leader→follower 更穩好 debug | phase-2 spike |
  | mobile base（LeKiwi / XLeRobot dual-wheel） | 行動底盤 | Anvil W2 為固定雙臂；底盤是後續升級，現在裝會擴大供電/運動學/校正面 | 長期（reference-map） |
  | Go2 / Furnace OS 整合 | 跨機器人整合 | Anvil 與 Go2/PawAI 是平行主線、各自獨立；W2 不接線、不共用 stack | 平行線，非 Anvil 任務 |
  | ACT / SmolVLA / pi0 / GR00T 訓練 | policy learning | 夏季第一迴圈目標，非 W2；真正瓶頸是乾淨 teleop 資料 | learning-roadmap |
  | world-model（Cosmos 3） | 環境模擬器 | 見下方獨立條目 | phase-2 |
  | AmazingHand 末端手 | 末端執行器升級 | 見下方獨立條目 | 副線 / phase-2 |

- 理由：W2 的中心交付是「真相收斂 + 一條會動的 teleop」，不是擴張能力面。上列每一項都「可用」但都不是 W2 里程碑，全部進 `reference-map.md` / `learning-roadmap.md` 當未來菜單，W2 一行 code 都不為它們寫。
- 狀態：傾向方向，未鎖定；Roy 確認後改為 LOCKED。

## 2026-06-29 — W2 首次 teleop 配對：Leader #1 → Follower #2（避開 F3，去風險）

- 背景：W2 里程碑＝至少一條 leader→follower 空中動作 teleop。兩隻 follower 健康度不同。
- 決定：W2 首次 teleop **配 Leader #1 → Follower #2**，不配 Follower #1；F3 接頭修復排到「正式錄資料前」再做，不讓 P0 卡在硬體維修。
- 理由（evidence）：
  - Follower #2（板 `...0335`）整串 `scan_port` 連掃 5 次全綠、校正全程無 `Incorrect/no status packet`（`docs/04-run-log/2026-06-27-follower2-bringup.md`）。
  - Follower #1 有 **F3（id3/elbow）接頭 marginal**：靜態 scan 穩，但需回應的寫入會間歇掉封包，`num_retry=5` 只能繞過、不可長期硬扛（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`；`docs/05-failure-log.md`；本檔 2026-06-26 條目待辦）。teleop/錄資料下會不穩。
- 前置守則：在 Leader #1「組裝 + 整串 `FeetechMotorsBus.scan_port` 驗 `[1..6]` 全綠 + 校正」完成前，**leader bus 健康一律寫 TBD**（`docs/04-run-log/2026-06-27-leader-bringup.md` 僅到「寫了 ID」）。leader 逐關節齒比一律以該 run-log 值為準（1×C001 1/345 + 2×C044 1/191 + 3×C046 1/147），不抄被組裝 runbook 明文推翻的舊對應。
- 狀態：傾向方向，待 leader bus 驗證後執行。

## 2026-06-29 — leLab web 流程延到 W3（Python 版本衝突）

- 背景：leLab（lerobot 的 web 化 record/train/job 流程）很香，考慮是否在 W2 引入做 teleop UI。
- 決定：**W2 不引入 leLab**，teleop 直接用已驗證的 `lerobot-teleoperate` CLI；leLab 列為 **W3 的 record/train/job-orchestration copy-idea 參考**。
- 理由：leLab 硬性要求 **Python 3.12+** 且把 lerobot pin 在特定 commit；Roy 的 Pi 跑 **Python 3.10.20 + lerobot 0.4.4 Seeed fork**（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`）。為了 UI 在 W2 引入 FastAPI+React 與一整套版本衝突，風險與收益不成比例。
- 狀態：傾向方向，未鎖定。

## 2026-06-29 — Anvil/X0 為 Go2/PawAI 的平行主線，非其手臂子模組（定位釐清）

- 背景：Anvil 與 PawAI（Go2 + SO-ARM + Furnace OS）都是 RoyBot-Lab 的機器人線，外部容易誤把 Anvil 當成 Go2 的手臂模組。
- 決定：**Anvil/X0 是與 Go2/PawAI 平行的獨立機器人線（獨立 Child Repo）**，不是 Go2 的子模組、不共用其 stack、不在 W2 與其整合。
- 理由：三層架構下每條機器人線是各自的 `Roy-*` Child Repo，git 歷史/可見度/節奏獨立；Anvil 的北極星是「低成本輪式雙臂 Physical-AI 平台」自身的 manipulation learning 迴圈。三名字＝同一台機器人（Obsidian 叫 **Anvil / X0**、GitHub `roy4222/Roy-XLeRobot-SO101`、本機目錄 `AnvilBot`），但此關係在 GitHub UI 看不出來，需於 `00-overview.md` 明寫。
- 狀態：定位釐清，視為 LOCKED（除非父 repo repo-map 另有更新）。

## 2026-06-29 — world-model（Cosmos 3）角色界定：環境模擬器，非控制器（phase-2）

- 背景：roadmap 素材含 NVIDIA Cosmos 3（action-conditioned world model）。需先界定它在 Anvil 的角色，避免被當成可直接上車的 policy。
- 決定：Cosmos（world-model）定位為 **phase-2 工具**，**W2 完全不碰**。
- 理由：Cosmos 是「學到的環境模擬器 / action-conditioned 影片生成器」，**不是 RL 控制器**。其用途為 (a) latent space 軌跡規劃、(b) 合成資料擴增（real + Cosmos rollout 餵 ACT 再訓）、(c) 想像中示範——全部應在「真機 baseline 穩了之後」才接。Anvil 的瓶頸是乾淨 teleop 資料量，不是世界模型容量。詳見 `learning-roadmap.md`（待建）。
- 狀態：角色界定，視為 LOCKED。

## 2026-06-29 — AmazingHand（仿人靈巧手）為副線，非 W2 主線

- 背景：AmazingHand 是 RoyBot-Lab 另一條機器人線；也可被視為 Anvil 末端執行器（夾爪→多指手）的升級方向。
- 決定：對 Anvil 而言 **AmazingHand 為副線 / phase-2 選配**，W2 不整合；W2 末端一律用 SO-101 原生夾爪（gripper, L6）。
- 理由：W2 要驗的是「資料→訓練→推論→eval」這條 pipeline 本身（首個任務 touch marker 刻意保守），換多指手會引入額外的機構/控制/校正/action space 面，與 W2 目標不對齊。多指手的 BOM / 介面 / 是否升級留待第一迴圈跑通後評估（TBD）。
- 狀態：傾向方向，未鎖定。

## 2026-06-29 — 命名層級鎖定：品牌「Anvil x XLeRobot」，repo id 保留 `Roy-XLeRobot-SO101`（不改名）

- 背景：同一條線有四個名字（Obsidian 品牌「Anvil/X0」、GitHub `Roy-XLeRobot-SO101`、本機目錄 `AnvilBot`、上游 XLeRobot/MakerMods），GitHub UI 看不出對應；先前 agent 跳神、舊 memory 還停在舊路徑 `src/Roy-XLeRobot-SO101`（目錄已改名）。
- 決定（**LOCKED**）：四層命名鎖定——
  - 正式品牌名 / README 標題＝**Anvil x XLeRobot**；
  - 平台代號 / 短稱＝**Anvil/X0**；
  - GitHub repo legal id（origin）＝**`roy4222/Roy-XLeRobot-SO101`**（**不改名**，走品牌別名路線，不做 repo rename migration）；
  - 本機工作目錄＝**`src/AnvilBot`**（docs 一律明寫它對應 `Roy-XLeRobot-SO101`；舊路徑 `src/Roy-XLeRobot-SO101` 已不存在）；
  - 上游技術來源＝XLeRobot / SO-ARM101 / LeRobot / LeKiwi / leLab（是依賴/參考，**不是** Anvil 本身的身分）。
- 理由：改 GitHub repo 名成本高（斷連結、要同步父 manifest/repo-map/CONTEXT 與多處引用），收益僅門面好看；品牌用正式別名即可拿到「專案有自己身分」的好處，不必現在做 repo rename migration。
- 同步狀態（2026-06-29 **已完成**）：① README 已改 Anvil x XLeRobot 門面＋上游 attribution（原 MakerMods README 保留為 `README-makermods-hardware.md`）；② 本 repo docs 已套用四層命名；③ 父 repo `CONTEXT.md` / `docs/repo-map.md`（路徑已改 `src/AnvilBot`）/ `robots/xlerobot-so101.md` 已同步新命名與新路徑。glossary 見 `CONTEXT.md`。
- 狀態：命名層級 **LOCKED**；GitHub repo rename 為日後可選 migration（非現在）。

## 2026-06-29 — 狀態/里程碑模型：Stage 為權威能力 gate、Week 為排程、退役 Phase/Button-Press

- 背景：repo 內三套狀態語言並存——父 `robots/xlerobot-so101.md` 的 Stage 0–3（LOCKED）、同檔自標不一致的 Phase 1–8 + Button Press、本 repo 新 docs 的 W1–W4 週框架；第一任務名亦有 touch marker / 推動式物品分類 / Button Press 三種，agent 易自相矛盾。
- 決定（**LOCKED**）：分層，不三選一互殺——
  - **Stage 0–3（+0.5 leader station）= 權威能力成熟度 gate**（沿用父 repo LOCKED 模型）。
  - **W1–W4 = 行事曆排程**，疊在 Stage 上，不是另一套能力里程碑。
  - **Phase 1–8 / Button Press = 退役**，標 historical，不再當里程碑。
  - **Stage 0 pass = 單臂 teleop 連續 5 分鐘無失控 + 影片/log evidence**；**W2「讓 leader→follower air-motion teleop 動起來」≠ Stage 0 pass**（W2 ⊂ Stage 0）。
  - **Stage 1 entry task = touch marker**（保守 pipeline 驗證）；**推動式物品分類 = Stage 1 後段 / Stage 2 eval**，不當第一個 gate。
- 理由：三套並存會讓文件與 agent 互相矛盾；Stage 是能力長弧（父已 LOCKED），Week 是時間排程，兩者正交、該分層；任務名分層可止住「第一個 demo 到底做什麼」反覆漂移。
- 同步狀態（2026-06-29 **已完成**）：本 repo `00-overview` / `learning-roadmap` / `open-questions` 與父 `robots/xlerobot-so101.md` 已同步此分層、退役 Phase/Button-Press。
- 狀態：**LOCKED**。
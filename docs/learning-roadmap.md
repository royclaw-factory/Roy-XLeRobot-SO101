# Learning Roadmap — teleop → dataset → ACT → eval → failure

> **Anvil x XLeRobot**（短稱 Anvil/X0；GitHub `roy4222/Roy-XLeRobot-SO101`，本機 `src/AnvilBot`）的學習路線：把「遙操作收資料 → 訓練 policy → 真機推論 → eval → 失敗分析」這條管線分成 now / next / long-term，並界定 touch-marker 任務階梯、第一版 dataset 策略、L0–L3 learned-policy 安全門、以及 AI 技術的引入順序。本檔是**菜單與排序**，不是進度宣稱；硬體狀態以 run-log 為準（見文末證據表）。

---

## 0. 先讀這段（讀者守則）

- **W2 不是學習週。** 本週（W2）NOT-this-week 明確排除：50-ep dataset、ACT 訓練、SmolVLA / pi0 / GR00T、mobile base、VR bimanual、Go2 整合、自動成功判定。W2 唯一的學習相關里程碑是「一條 leader→follower 空中動作 teleop」（控制迴圈驗證，**無 policy、無物件**），其餘全在 next / long-term。這份 roadmap 的存在不代表可以提早跳關。
- **狀態用 Stage 能力 gate（權威）+ Week 排程兩層**（見 `docs/07-decisions.md`）。本檔 NOW = **Stage 1**（teleop→dataset→ACT，entry task `touch marker`）；NEXT ≈ Stage 2/3；long-term = phase-2。**Stage 0 pass = 單臂 teleop 連續 5 分鐘無失控 + evidence；W2 的「一條會動的 teleop」是 Stage 0 內、朝 Stage 0 pass，≠ Stage 0 pass 本身。** Phase 1–8 / Button Press 舊模型已退役。
- **瓶頸是「乾淨 teleop 資料」，不是模型容量。** 因此起點固定是 ACT（輕量、已是 LeRobot 參考基線），不是 pi0 / GR00T（巨型、閉源資料、Pi 跑不動）。資料品質 > 資料數量 > 模型大小。
- **Anvil 是與 Go2 / PawAi 平行的獨立機器人**，不是任何機器人的手臂子模組。本檔只談 Anvil 自身的 SO-101 學習線。
- **Evidence 紀律**：本檔任何「已完成」字樣都必須回指 run-log；尚無證據者一律寫 **TBD / 待驗證**。**目前 Anvil 沒有任何 teleop / dataset / 訓練 / eval 紀錄**——leader #1 仍待組裝與 bus 驗證（`docs/04-run-log/2026-06-27-leader-bringup.md` L32–36），所以本檔 NOW 之後的每一階都是計畫，不是成果。
- **不要把 upstream README 當進度。** repo 根的 `README.md` / `README_cn.md` / `LICENSE` 是 MakerMods 硬體上游內容，與 Roy 的學習進度無關。
- **外部成熟度數字皆待驗證。** 下面 next / long-term 表格中的「成熟度 / 訓練時數 / GPU 需求」來自 2026-06 外部網路調查，**未在 Anvil 實際 stack（Pi 5、lerobot 0.4.4 Seeed fork）上驗證**，一律視為待驗證。Pi 5 本身不具訓練這些模型的算力，**訓練主機 TBD**。

---

## 1. 三階總覽（now / next / long-term）

| 階段 | 對應里程碑 | 核心技術 | 前置（gating） | 證據狀態 |
|---|---|---|---|---|
| **NOW**（夏季第一迴圈，**非 W2**） | leader→follower teleop → 第一版 dataset → **ACT baseline** → 真機推論 → touch-marker eval → 失敗分析 | ACT（動作分塊行為克隆）+ leader→follower CLI 收資料 | Leader #1 組裝 + bus 健康 + 校正完成；Follower bus 乾淨 | **全部 TBD**（leader 未組裝；尚無任何資料/訓練） |
| **NEXT** | 在單一任務跑通 ACT 後再開 | reward classifier、HIL-SERL（人在環 RL）、Diffusion Policy、SARM / Robometer eval、SmolVLA | ACT 在 touch-marker 收斂 + 有乾淨示範 + 訓練主機（GPU，TBD） | 未開始；外部成熟度待驗證 |
| **LONG-TERM / phase-2** | 規模化、跨形體、世界模型、行動底盤、VR | X-VLA、VQ-BeT、pi0 / GR00T（對照用）、Isaac Lab sim2real、Cosmos3 world-model、Quest VR teleop、LeKiwi / XLeRobot mobile base | 500+ demo、多任務、GPU 叢集、機構升級——皆遠期 | 全部 study-only；W2 一行 code 都不為它們寫 |

> W2 真正會動到的「外部資源」只有 SO-ARM100 的 BOM / 電壓核對（給採購清單）。leLab / Phosphobot / VR / 大模型全部進 reference / roadmap 當未來菜單，不進 W2 程式。leLab 因要求 Python 3.12+ 與 Pi 的 Python 3.10.20 / lerobot 0.4.4 衝突（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md` L6），延到 W3 當 copy-idea，W2 teleop 直接用已驗證的 `lerobot-teleoperate` CLI。

---

## 2. Touch-marker 任務階梯

「touch marker」是**刻意保守的管線驗證任務**：它驗的是「資料 → 訓練 → 推論 → eval」這條管線本身，不是任務難度。階梯由易到難，**每一階都先收乾淨示範、再訓練、再 eval**；不跳階。

| 階 | 任務 | 驗證目的 | 變因 | 成功判定（第一版） | 何時 |
|---|---|---|---|---|---|
| **T0** | leader→follower 空中鏡像動作（**無物件、無 policy**） | 控制迴圈通、延遲可接受、bus 在動態負載下不掉封包 | 無 | 人工目視 + terminal log 無 `Incorrect/no status packet` | **W2 排程**（Stage 0 內、朝 Stage 0 pass；配 Leader #1 → Follower #2 乾淨 bus）|
| **T1** | follower 觸碰**單一固定位置** marker | 第一個會錄資料的 manipulation；end-to-end pipeline 跑通 | 無（位置固定） | 人工目視「指尖/夾爪觸到 marker」 | **Stage 1 entry task**（NOW；ACT baseline 第一個對象）|
| **T2** | 觸碰**workspace 內隨機位置** marker | 測 ACT 對位置的泛化（擾動式資料才有用的第一個點） | marker 位置擾動、起手姿勢擾動 | 目視觸碰率（門檻 TBD） | NOW → NEXT |
| **T3** | **多 marker / 顏色選擇**觸碰指定目標 | 引入 semantic / multimodal 決策 | 顏色、數量、語言指令（若加 VLA） | 選對 + 觸到（門檻 TBD） | NEXT |
| **T4** | 觸碰 → **推動式物品分類**（push-to-bin） | 接到 `docs/00-overview.md` 的 eval 目標 | 物品種類、起始位置 | 分類正確率（門檻 TBD） | **Stage 1 後段 / Stage 2**（NEXT；非第一個 gate；接 reward classifier 後可半自動判定）|

- **成功判定**：T0–T2 一律**人工目視 + 影片/log**，不做自動成功判定（自動判定屬 NEXT 的 reward classifier / Robometer）。
- **資料紀律**：每階的示範影片、teleop 影片、terminal log 為**大檔，不 commit**，只在 `docs/08-media-index.md` 留索引（硬規則 5）。

---

## 3. 第一版 dataset 策略

**前提（必須先成立，否則整段是 TBD）**：Leader #1 組裝完成、整串 `scan_port` 回 `[1..6]` 全綠、`lerobot-calibrate` 完成（目前皆未做，`docs/04-run-log/2026-06-27-leader-bringup.md` L32–36）。

| 面向 | 第一版策略 | 依據 / 備註 |
|---|---|---|
| **收集方式** | leader→follower 鏡像 teleop（`lerobot-teleoperate`） | 最穩、最好 debug 的路（`docs/07-decisions.md` 2026-06-26） |
| **配對** | **Leader #1 → Follower #2（板 `...0335`，乾淨 bus）** | Follower #2 連掃 5 次全綠、校正零掉封包（`docs/04-run-log/2026-06-27-follower2-bringup.md`）；Follower #1 的 F3 接頭 marginal、teleop 下會間歇掉封包（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md` L62–67），**F3 修復前不拿來錄資料** |
| **品質 > 數量** | 乾淨示範為瓶頸；寧可少而乾淨，不要多而髒 | roadmap 的核心原則 |
| **擾動式 > 靜態重複** | 用「位置 / 光照 / 起手姿勢擾動 + 少量負面（miss）示範」取代大量靜態重複 | 外部 hackathon 證據（anjalidhabaria：40 變化 demo 的泛化 > 100 靜態 demo）；**copy-idea，待 Anvil 自行驗證** |
| **episode 規模** | 第一版以小批量（數十 ep 量級）跑通 pipeline 為先，數字 **TBD**（W3 定） | 不在 W2 做 50-ep dataset |
| **多視角 encoder** | 規劃**分離式多視角 encoder**（非共享） | 外部證據指分離 encoder 優於共享；**待驗證**，且會增加參數量 |
| **相機** | 型號 **TBD**（參考 SO-ARM100 的 D405 / UVC 相機座），列入 `docs/02-bom.md` 採購 | 尚無相機 evidence |
| **資料格式** | `LeRobotDataset`（lerobot 0.4.4 Seeed fork） | 與現有 stack 一致（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md` L6） |
| **大檔處理** | dataset / 影片 / log 不 commit，只留索引 | 硬規則 5；`docs/08-media-index.md` |

---

## 4. L0–L3 learned-policy 安全門

讓「學到的 policy」驅動真機前，必須逐級通過安全門；**每升一級都要有 evidence**，沒有就停在當級。這套門與既有硬體安全事實綁定。

| 門 | 准許範圍 | 進入條件（gating） | 綁定的硬體安全事實 |
|---|---|---|---|
| **L0** | 純離線：dataset 視覺化、replay dry-run、**無馬達輸出** | 永遠允許 | 無風險 |
| **L1** | 真機推論但**限幅**：低速、`max_relative_target` 限制、**無負載**、人手在 E-stop | bus healthy（`scan_port [1..6]` 全綠，有截圖）+ 校正完成 | `max_relative_target=15` 已用於 follower 首次動作（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`）；**USB 能列舉 ≠ bus healthy**（板 `...4639` 反例，`docs/05-failure-log.md`） |
| **L2** | 真機推論 + **接觸任務**（touch marker，低力矩） | L1 通過 + 該任務有 eval（人工目視或 reward classifier）+ **F3 已修復** | Follower #1 的 F3 marginal 不可硬扛（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md` L67；`docs/07-decisions.md` L21）；接觸任務務必確認用對 follower |
| **L3** | 連續 / 長序列 / 多技能串接 / 半自主 run | L2 通過 + `docs/05-failure-log.md` 有失敗分析 + eval 通過率達門檻（**TBD**） | 多技能串接誤差會累積（外部 hackathon 證據：鏈式 ACT 無錯誤恢復機制）→ 須先有 failure log |

**跨所有門的硬安全條件（違反即停手）**：

- **5V / 12V 電源誤插 = 燒 leader 馬達**：leader 7.4V 馬達**必配 5V**，12V 直燒（`docs/research/2026-06-26-so-arm101-assembly-runbook.md` §0；`docs/04-run-log/2026-06-27-leader-bringup.md` L7）。電源須實體貼標 `5V→LEADER` / `12V→FOLLOWER`。
- **Leader 混齒比裝錯關節 = 力矩不匹配**：務必照 run-log 齒比表（下節），**拒用**組裝 runbook 已標錯的舊對應。
- **宣稱 bus healthy 前必跑 `FeetechMotorsBus.scan_port`**，不可只憑 `/dev/serial/by-id/` 列舉或 `screen` 開得起來。

### Leader 逐關節齒比（一律用 run-log 值）

來源 `docs/04-run-log/2026-06-27-leader-bringup.md` L17–26（工具逐顆確認）。**不可採用組裝 runbook §4 C2 已明文標錯的舊對應**（`...assembly-runbook.md` 標示「ID1:1/345, ID2-3:1/147, ID4-5:1/191, ID6:1/345」為錯誤）。

| 關節 (ID) | 功能 | 齒比 / 型號 |
|---|---|---|
| L1 | shoulder_pan 底座 | 1/191 C044 |
| L2 | shoulder_lift 肩抬 | 1/345 C001 |
| L3 | elbow_flex 肘 | 1/191 C044 |
| L4 | wrist_flex 腕俯仰 | 1/147 C046 |
| L5 | wrist_roll 腕滾 | 1/147 C046 |
| L6 | gripper 握把扳機 | 1/147 C046 |

分布 = 1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。

---

## 5. AI 技術何時碰（引入排序）

排序原則：**模仿（ACT）→ 強化（HIL-SERL）→ 規模化（SmolVLA / X-VLA）→ 研究（Cosmos / VQ-BeT）**，每一階都建立在上一階的乾淨資料上。下表「成熟度 / 資源」欄來自 2026-06 外部調查，**未在 Anvil stack 驗證，皆待驗證**；Pi 5 不負責訓練，訓練主機 TBD。

| 技術 | 是什麼 | 何時碰 | 為何這個順序 | 證據狀態 |
|---|---|---|---|---|
| **ACT** | Transformer 行為克隆，預測動作分塊 | **NOW**（第一個 baseline） | 輕量、已是 LeRobot 參考基線；Anvil 缺的是資料不是模型容量 | 待 Anvil 跑通；外部稱 production-ready |
| **Reward classifier** | 視覺成功/失敗分類器，給自動 reward | NEXT（收到 ~20–30 乾淨 demo 後） | 銜接模仿與 RL，免人工標 reward | 未開始 |
| **HIL-SERL（+SAC）** | 人在環 RL，少量 demo + reward + 人為接管精修 | NEXT（ACT 收斂後） | 解 ACT 在接觸任務的瓶頸 | 未開始；需 GPU learner（TBD） |
| **Diffusion Policy** | 去噪擴散動作預測，擅長多模態 / 接觸密集 | NEXT（ACT + 一個任務驗成後 A/B） | 出現「左或右皆可」等多模態才需要 | 未開始 |
| **SARM / Robometer** | 影片式 reward / 進度模型（eval 用） | NEXT（長序列 / 多階段任務時） | 取代脆弱的 frame-index reward 標註 | 未開始；需 GPU 推論 |
| **SmolVLA（~450M）** | 緊湊 VLA，單 GPU 可微調 | NEXT（ACT 穩固後、想做低成本邊緣部署） | 契合「便宜小手臂」哲學；非研究必需 | 未開始 |
| **X-VLA** | 軟提示跨形體 VLA | LONG-TERM（累積 500+ demo、要跨多台 SO-101） | 規模化到車隊才划算 | study-only |
| **VQ-BeT** | 動作 token 化，長序列 / 組合性 | LONG-TERM（1000+ 多任務 demo 才考慮） | Anvil 瓶頸是資料量不是序列長度 | study-only（LeRobot 尚未整合） |
| **pi0 / pi0.5 / GR00T** | 巨型通用 / 人形基礎模型 | LONG-TERM（僅作論文對照基線） | 閉源資料、算力門檻高，不適合本實驗 | study-only；**不訓練** |
| **Isaac Lab sim2real** | SO-101 數位孿生 + domain randomization | LONG-TERM（ACT baseline 後做資料增強 / 安全測試） | 是工具不是 policy；真機資料優先 | study-only；需 NVIDIA GPU |
| **Cosmos3 world-model** | 動作條件影片世界模型 | LONG-TERM / phase-2（規劃 / 合成資料） | 是環境模擬器不是控制器；真機 baseline 穩了再用 | study-only |
| **VR teleop（Quest）** | Phosphobot / WebXR 遙操作 | LONG-TERM / phase-2 spike | 官方真機 code 變因多；leader→follower 才是最穩收資料路 | study-only（`docs/00-overview.md` 已列 phase-2） |
| **Mobile base（LeKiwi / XLeRobot）** | 行動底盤 + holonomic 運動學 | LONG-TERM（W2 明確排除） | 先把固定雙臂操作學會 | study-only |

**明確避免**：在初期實驗用 pi0 / GR00T（overkill、閉源）；無 1000+ 多樣 demo 就上 VQ-BeT；以 Isaac Lab 當主要訓練來源（sim gap 會咬人，真機資料優先）。

---

## 6. 證據來源與待解問題

**本檔引用的 run-log / 真相檔**（硬體聲明一律回指這些）：

| 檔案 | 提供的證據 |
|---|---|
| `docs/04-run-log/2026-06-26-follower-motor-id-setup.md` | Follower #1：ID / 組裝 / bus 排障 / 校正 / 首次動作；F3 接頭 marginal；Pi5 + venv + lerobot 0.4.4 stack |
| `docs/04-run-log/2026-06-27-follower2-bringup.md` | Follower #2：板 `...0335`、scan 5 次全綠、校正零掉封包 |
| `docs/04-run-log/2026-06-27-leader-bringup.md` | Leader #1：6 顆混齒比 ID 表、5V 電源、**組裝/bus/校正皆 TBD** |
| `docs/05-failure-log.md` | 板 `...4639` SUSPECT（USB 列舉但 `scan_port {}` / `RX_BYTES 0`） |
| `docs/06-metrics.md` | follower 校正 encoder range |
| `docs/07-decisions.md` | 2F+2L 傾向、suspect 不蓋棺、F3 待修 |
| `docs/research/2026-06-26-so-arm101-assembly-runbook.md` | 電壓安全（§0）、被推翻的錯誤齒比對應（§4 C2） |

**滾動式待解問題（TBD / 待驗證）**：

1. Leader #1 組裝 + 整串 `scan_port [1..6]` + 校正——全未做（roadmap NOW 的真正起點）。
2. 第一版 dataset 的 episode 數、相機型號、成功率門檻——W3 定。
3. 各任務階梯（T2–T4）與安全門（L3）的**通過率門檻**——尚無數據可定。
4. 馬達方向「預設反轉」是否需驗證 LeRobot `SO101*Config`——TBD。
5. 訓練主機（GPU）——next 之後所有模型訓練的前置，**TBD**。
6. Follower #1 F3 接頭修復——L2 接觸任務 / 正式錄資料前必修。
7. leLab Python 3.12 vs Pi 3.10.20 / lerobot 0.4.4 衝突——W2 不引入。

> 本檔僅為計畫與排序。任何一階要標成「已完成」，必須先在 `docs/04-run-log/` 留下對應 run-log 與媒體索引；在那之前一律寫 TBD / 待驗證。
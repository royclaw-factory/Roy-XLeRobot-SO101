# Open Questions — 待決策清單

> 用途：彙整 Anvil / X0 目前所有「待 Roy / PM 拍板」或「待補證據才能定論」的開放問題。這是一份**滾動式清單**：每題標明現況證據、可選方案、影響面、待誰決策與優先序。**未驗證者一律寫 TBD / 待驗證**，硬體聲明一律引用 `docs/04-run-log/` 檔名。解決後不刪除，改標「已決議」並指向對應 ADR（`docs/07-decisions.md`）或 run-log。

> 範圍提醒：本週（W2，2026-06-30 ~ 07-06）是排程上的**硬體收斂週**（推進 Stage 0.5、朝 Stage 0 pass，**W2 ≠ Stage 0 pass**），**不是** VLA / VR / world-model / mobile base / ACT 訓練週。本清單中標 P2 / 長期者，W2 一行 code 都不為它寫，只記錄待決，避免範圍蔓延。
>
> **已決議（不再 open，見 `docs/07-decisions.md` 2026-06-29）**：① 命名層級 LOCKED（品牌 **Anvil x XLeRobot** / repo id `Roy-XLeRobot-SO101` / 本機 `src/AnvilBot`）；② 狀態模型 LOCKED（**Stage 能力 gate + Week 排程**，退役 Phase/Button-Press）；③ GitHub repo **不改名**（rename 為日後可選 migration）。下表只列仍 open 的項。

---

## 0. 總表（依優先序）

| ID | 開放問題 | 類別 | 待誰決策 | 優先序 | 狀態 |
|---|---|---|---|---|---|
| Q1 | 系統組態是否正式 LOCK 2F+2L | 組態/里程碑 | Roy / PM | P0 | 傾向已記，未 LOCK |
| Q2 | W2 首條 teleop 用哪組 leader→follower 配對 | 組態/里程碑 | Roy（工程建議已備） | P0 | 待 Roy 確認 |
| Q3 | 控制板 `...4639`（SUSPECT）何時複驗、誰測 | 硬體複驗 | Roy | P1 | 未複驗 |
| Q4 | Follower #1 的 F3 接頭何時修、用重壓還是換線 | 硬體維修 | Roy | P1 | 待辦 |
| Q5 | Leader #2 何時啟動、用哪批馬達/板 | 硬體擴充 | Roy / PM | P1 | 未開始 |
| Q6 | 固定外部 RGB camera 是否到位、型號/座/視角 | 感測/任務 | Roy / PM | P1 | TBD |
| Q7 | touch-marker 任務的 marker / 桌面 / home pose 是否物理固定 | 任務環境 | Roy / PM | P1 | TBD |
| Q8 | 馬達「預設反轉」是否需對 LeRobot `SO101*Config` 實證 | 軟體正確性 | Roy | P2 | 待驗證 |
| Q9 | leLab 的 Python 3.12 floor vs Pi 3.10.20 衝突如何長期解 | 軟體棧 | Roy | P2 | 待決（W2 先避） |
| Q10 | 第一版 ACT baseline 用哪台 GPU、何時開訓 | 學習/算力 | Roy / PM | P2（非 W2） | TBD |
| Q11 | 哪些 reference 要從 glance 升級為 first-class | Reference 分級 | Roy | P2 | 傾向已備 |
| Q12 | 二手馬達來源混雜，是否需建批次/相容性追蹤 | 物料來源 | Roy | P2 | 記錄即可 |

---

## 1. 系統組態與里程碑

### Q1 — 系統組態是否正式 LOCK「2 Follower + 2 Leader」

| 欄位 | 內容 |
|---|---|
| 問題 | 目前 2F+2L 是「傾向」還是「鎖定」？是否允許之後改 4-hand 霸王 / 單臂等其他組態？ |
| 現況與證據 | `docs/07-decisions.md` 記為**傾向（tendence），非 LOCK**；4-hand despot 與 VR teleop 已明確推到 phase-2。實機現況：Follower #1 / #2 已 bring-up（`2026-06-26-follower-motor-id-setup.md`、`2026-06-27-follower2-bringup.md`），Leader #1 bus + 校正已驗（`2026-06-29-leader1-calibration.md`：scan `[1..6]`、`roy_leader`），Leader #2 未開始。 |
| 選項 | (a) 正式 LOCK 2F+2L 作為夏季第一迴圈定組態；(b) 維持「傾向」、保留彈性到 Leader #1 teleop 驗證後再鎖。 |
| 影響 | 影響 BOM v0 採購數量（`docs/02-bom.md`）、Leader #2 是否該本週啟動、文件中「定組態」敘述能否定稿。 |
| 待誰決策 | Roy / PM。 |
| 優先序 | P0（牽動 W2 P0 採購清單與里程碑文案）。 |
| 建議 | 工程上 2F+2L 站得住（leader 為低扭混齒比、專為當 leader 設計，當 follower 會「2 強 2 弱」）。建議**等 Q2 首條 teleop 跑通後再 LOCK**：leader bus 雖已驗（2026-06-29），但「2 強 2 弱」手感與齒比↔關節對應要 teleop 施力才驗得出，定死前先跑通。 |

### Q2 — W2 首條 leader→follower 空中動作 teleop 用哪組配對

| 欄位 | 內容 |
|---|---|
| 問題 | W2 P0 里程碑「至少一條 leader→follower 空中動作 teleop」要配哪一隻 follower？ |
| 現況與證據 | Follower #2（板 `...0335`）整串 `scan_port` **連掃 5 次全綠、校正全程無掉封包**（`2026-06-27-follower2-bringup.md`）。Follower #1 有 **F3 接頭 marginal、寫入間歇掉封包**（`2026-06-26-follower-motor-id-setup.md`；見 Q4），靜態 scan 穩但動態寫入會失敗。Leader #1 bus + 校正已驗（`2026-06-29-leader1-calibration.md`：scan `[1..6]` 全綠、`roy_leader` 存檔），可起首測。 |
| 選項 | (a) **Leader #1 → Follower #2（乾淨 bus）**；(b) Leader #1 → Follower #1（需先修 F3）。 |
| 影響 | 決定 P0 是否被硬體維修卡住；決定第一支 teleop 影片/log 的證據能不能在 W2 內拿到。 |
| 待誰決策 | Roy（工程建議已備，等確認）。 |
| 優先序 | P0。 |
| 建議 | **配 (a) Leader #1 → Follower #2** 去風險，F3 修復（Q4）留到正式錄資料前再做，不要讓里程碑卡在維修上。前提：Leader #1 須先完成組裝＋`scan_port [1..6]` 全綠＋校正（在此之前 leader bus 健康一律寫 TBD）。 |

---

## 2. 硬體複驗 / 維修 / 擴充

### Q3 — 控制板 `...4639`（SUSPECT）何時複驗、由誰測

| 欄位 | 內容 |
|---|---|
| 問題 | `...4639` 是真的壞、還是當時的馬達/線材問題？要不要花時間複驗，何時測？ |
| 現況與證據 | `docs/05-failure-log.md`：該板 **USB 可列舉、port 開得起來，但 `scan_port` 回 `{}`、`RX_BYTES 0`**——即「USB 能列舉 ≠ servo bus 健康」的反例。`docs/07-decisions.md` 記為 **SUSPECT、暫停、未蓋棺**，待用「好馬達 + 好線」單獨複驗。 |
| 選項 | (a) W2 內安排一次乾淨複驗（好馬達＋好線單測）；(b) 暫不複驗、維持 off 關鍵路徑，待 Leader #2 需要板時再測。 |
| 影響 | 若可救活，可省一張板的採購（影響 BOM v0）；若確認壞，BOM 要列補購。 |
| 待誰決策 | Roy。 |
| 優先序 | P1（不在 W2 P0 關鍵路徑，但牽動 BOM）。 |

### Q4 — Follower #1 的 F3 接頭何時修、重壓 vs 換線

| 欄位 | 內容 |
|---|---|
| 問題 | F3（elbow / id3）接頭 marginal 要用重壓解決還是更換接頭/線段？排在什麼時間點？ |
| 現況與證據 | `2026-06-26-follower-motor-id-setup.md`：**id3 為最常重現點**，靜態 scan 穩、需回應的寫入間歇失敗，`num_retry=5` 只能繞過。`docs/07-decisions.md` 待辦：**徹底重壓/更換 F3，不可長期硬扛**。對 teleop / 錄資料是 blocker。 |
| 選項 | (a) 先重壓觀察；(b) 直接換接頭/該段線；先後順序：W2 用 Follower #2 繞過（見 Q2），F3 修復排在**正式錄資料前**。 |
| 影響 | 決定 Follower #1 能否進正式 dataset 錄製；不修則 Follower #1 不可宣稱可用於錄資料。 |
| 待誰決策 | Roy。 |
| 優先序 | P1（W2 可延後，但錄資料前必解）。 |

### Q5 — Leader #2 何時啟動、用哪批馬達 / 哪張板

| 欄位 | 內容 |
|---|---|
| 問題 | Leader #2 本週要不要開？用哪批混齒比馬達、哪張控制板？ |
| 現況與證據 | **完全未開始**（`2026-06-27-leader-bringup.md` 僅記 Leader #1）。Leader 需 1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147) 的混齒比配置，並配 **5V**（7.4V 馬達，12V 直接燒，見安全條目）。 |
| 選項 | (a) W2 先只完成 Leader #1 + 首條 teleop，Leader #2 延到 W2 末/W3；(b) 平行啟動 Leader #2。 |
| 影響 | 牽動 Q1（2F+2L 是否 LOCK）與 BOM v0（第二批 leader 馬達/板/5V 電源是否採購）。 |
| 待誰決策 | Roy / PM。 |
| 優先序 | P1。 |
| 建議 | W2 聚焦 Leader #1 + 首條 teleop；Leader #2 待 Q1 LOCK 後再排，避免同時開兩條未驗證路線。 |

---

## 3. 感測與任務環境（touch-marker pipeline 驗證任務）

### Q6 — 固定外部 RGB camera 是否到位

| 欄位 | 內容 |
|---|---|
| 問題 | 用於錄 demo / 之後 ACT 觀測的**固定外部 RGB camera** 是否已備？型號、相機座、視角、解析度/幀率怎麼定？ |
| 現況與證據 | repo 內 **無任何 camera 到位證據**（`docs/08-media-index.md` 仍為骨架；run-log 未記相機）。SO-ARM100 參考提供相機座選項（D405 / UVC），屬可抄規格但**尚未落到 Anvil 實機** → 一律 TBD。 |
| 選項 | (a) 先用一支固定 UVC webcam 過 W2 pipeline 驗證；(b) 等 D405 等深度/高品質相機到貨再錄。 |
| 影響 | 沒有固定相機＝無法錄可重訓的 demo，會卡住「夏季第一迴圈」的 dataset 階段（非 W2，但 W2 末的 dataset readiness 檢查需要它）。 |
| 待誰決策 | Roy / PM。 |
| 優先序 | P1（W2 P1 stretch「camera visibility check」直接相關）。 |

### Q7 — touch-marker 的 marker / 桌面 / home pose 是否物理固定

| 欄位 | 內容 |
|---|---|
| 問題 | 首個任務「touch marker」（刻意保守的 pipeline 驗證任務）需要的 **marker 位置、桌面、機械臂 home pose** 是否物理固定、可重現？ |
| 現況與證據 | 目前 repo **無任務環境定義**（無 fixture、無 home pose 規格、無桌面座標）。touch-marker 驗的是「資料→訓練→推論→eval」管線本身，可重現的固定環境是其前提；現況一律 TBD。 |
| 選項 | (a) 制定固定 fixture（marker 貼點、桌面標記、leader/follower 安裝座、home pose）並拍照存證；(b) 暫以鬆散擺放跑通 teleop，固定化留到錄正式 dataset 前。 |
| 影響 | 環境不固定＝demo 不可重現＝policy 評估無意義；影響 dataset 與 eval 階段的有效性。 |
| 待誰決策 | Roy / PM。 |
| 優先序 | P1（W2 teleop 可先鬆散，dataset 前必須固定並寫進 `docs/03-architecture.md` / run-log）。 |

---

## 4. 軟體正確性 / 軟體棧

### Q8 — 馬達「預設反轉」是否需對 LeRobot `SO101*Config` 實證

| 欄位 | 內容 |
|---|---|
| 問題 | SO-101 馬達方向「預設反轉」狀態，是否需要對照 LeRobot 原始碼（`SO101LeaderConfig` / `SO101FollowerConfig`）逐顆驗證，避免 teleop 鏡像方向錯誤？ |
| 現況與證據 | `CLAUDE.md` 註記方向由 LeRobot config 決定，但**可用文件未明列預設反轉對應 → 待驗證**。follower 校正 encoder range 已記（`docs/06-metrics.md`），但「方向正負號」未對程式碼核對。 |
| 選項 | (a) 在首條 teleop 前 grep LeRobot config 對齊每顆 `drive_mode`/正負；(b) 以 teleop 實測鏡像方向是否正確，反推有無需要改 config。 |
| 影響 | 方向錯會讓 leader→follower 鏡像反向，teleop 與後續 dataset 全部失真。 |
| 待誰決策 | Roy（工程任務）。 |
| 優先序 | P2（首條 teleop 時順手驗證即可解）。 |

### Q9 — leLab 的 Python 3.12 floor vs Pi 3.10.20 衝突如何長期解

| 欄位 | 內容 |
|---|---|
| 問題 | leLab 的 web 流程（record/train/job 編排）硬性要求 **Python 3.12+** 且把 lerobot pin 在特定 commit；Pi 跑的是 **Python 3.10.20 + lerobot 0.4.4 Seeed fork**（`2026-06-26-follower-motor-id-setup.md`）。長期要不要、以及怎麼引入 leLab？ |
| 現況與證據 | W2 的 teleop 已可用 Roy 手上驗證過的 `lerobot-teleoperate` CLI 完成，不需 leLab。leLab 列為 **W3 的 record/train/job 編排 copy-idea 參考**。 |
| 選項 | (a) W3 在另一台/另一 venv（py3.12）跑 leLab，與 Pi 控制環境分離；(b) 不引入 leLab，自寫薄 CLI 包裝沿用既有流程；(c) 等 leLab 放寬版本需求。 |
| 影響 | 決定 W3 是否要維護雙 Python 環境；避免在 W2 為 UI 引入版本衝突與整套 FastAPI+React。 |
| 待誰決策 | Roy。 |
| 優先序 | P2（W2 明確不引入）。 |

---

## 5. 學習 / 算力（非 W2）

### Q10 — 第一版 ACT baseline 用哪台 GPU、何時開訓

| 欄位 | 內容 |
|---|---|
| 問題 | 「夏季第一迴圈」的 ACT baseline 要用哪台 GPU（本地 RTX 4090 等 / 雲端）？何時開訓？ |
| 現況與證據 | **ACT 不是 W2**（W2 非學習週）。目前 repo **尚無任何 teleop / dataset**，真正瓶頸是「乾淨 teleop demo 的資料品質」而非模型或算力。ACT 一般需 ~50 起的乾淨 demo、單張 24GB 級 GPU、數小時訓練（屬 `docs/learning-roadmap.md` 規劃，未實作）。 |
| 選項 | (a) 本地 GPU；(b) 雲端租用；(c) 先不決，等 dataset 量到再選。 |
| 影響 | 影響長期算力預算與採購；但**不阻擋 W2**。 |
| 待誰決策 | Roy / PM。 |
| 優先序 | P2 / 長期（W2 不處理，僅登記）。 |

---

## 6. Reference 分級

### Q11 — 哪些 reference 要從 glance 升級為 first-class

| 欄位 | 內容 |
|---|---|
| 問題 | 目前 first-class = SO-ARM100（硬體源頭）、leLab（軟體棧）；glance = XLeRobot。哪些要升級、哪些 fork-later 對象要轉正？ |
| 現況與證據 | `reference/` 三 repo 全 gitignored（`.gitignore` 僅一行 `reference/`），由父 repo vcstool 拉取。SO-ARM100 在 W2 即被 BOM/電壓引用（first-class 合理）；leLab 為 W3 copy-idea（first-class 軟體合理）；XLeRobot 的 mobile base / IK / web 屬長期，W2 排除。 |
| 選項 | (a) 維持現分級；(b) 將 LeKiwi（mobile base CAD）在進入 mobile 階段時升 first-class；(c) 視 W3 需求把某 hackathon repo 的資料策略升為 copy-code 對象。 |
| 影響 | 影響 `docs/reference-map.md` 的分級與 copy-idea/copy-code 標註，避免過早把長期項目拉進關鍵路徑。 |
| 待誰決策 | Roy。 |
| 優先序 | P2。 |
| 備註 | VR（Phosphobot / SO-101-VR-Control）、世界模型（Cosmos3 / pi0 / GR00T）一律 study-only / 長期，**W2 全部排除**，不升級。 |

---

## 7. 物料來源

### Q12 — 二手馬達來源混雜，是否需建批次 / 相容性追蹤

| 欄位 | 內容 |
|---|---|
| 問題 | 近期 run-log 出現二手馬達（如 Follower #2 板 `...0335` 為二手、馬達型號標 777），是否需要建立「新品 vs 二手 / 批次 / 相容性」追蹤？ |
| 現況與證據 | `2026-06-27-follower2-bringup.md` 記板 `...0335` 為二手且驗證乾淨（5/5 scan 全綠），目前**非 blocker**；但全機馬達型號/齒比/來源散落各 run-log，未集中。 |
| 選項 | (a) 在 `docs/hardware-state.md` 增一欄「來源/批次」；(b) 維持現狀只在 run-log 逐筆記。 |
| 影響 | 影響日後排障可追溯性與 BOM 補購判斷；目前僅記錄，非關鍵路徑。 |
| 待誰決策 | Roy。 |
| 優先序 | P2（記錄即可）。 |

---

## 附錄：跨題硬約束（決策時務必遵守）

- **電源安全**：Leader 7.4V 馬達**必配 5V**，12V 直接燒馬達（`2026-06-27-leader-bringup.md`；組裝 runbook §0）。電源須實體貼標 `5V→LEADER` / `12V→FOLLOWER`。任何 leader 相關決策不得違反。
- **Leader 混齒比對應**：一律用 run-log 值——L1/L3 = 1/191(C044)、L2 = 1/345(C001)、L4/L5/L6 = 1/147(C046)（`2026-06-27-leader-bringup.md`）。**拒用**組裝 runbook §4 已標錯的舊對應（`ID1:1/345, ID2-3:1/147, ID4-5:1/191, ID6:1/345`）。
- **bus 健康定義**：「USB 能列舉」≠「servo bus 健康」。任何「板 OK / leader ready」聲明須有 `FeetechMotorsBus.scan_port` 全綠證據，否則寫 TBD（反例：板 `...4639`，`docs/05-failure-log.md`）。
- **定位**：Anvil / X0 是與 Go2 / PawAI 平行的獨立機器人主線，**不是其手臂子模組**。任何組態/採購決策依此定位。
# 參考寶藏地圖（Reference Map）

> 用途：把 `reference/`（leLab / SO-ARM100 / XLeRobot / LeKiwi）與外部 hackathon repos、Binh notes 收成一張「寶藏表」——每個來源**可用什麼、是 copy-idea 還是 copy-code、何時用（W2 / W3 / 長期）、有什麼風險與前置條件**。給未來一上車的 agent / 工程師快速判斷「哪些能抄、哪些只看、哪些 W2 一行都別碰」。

---

## 0. 先讀這段（三條紀律）

1. **這張表是「未來菜單」，不是 W2 工單。** W2 **不是** VLA / VR / world-model / mobile base 實作週。W2 明確 NOT-this-week：50-ep dataset、ACT 訓練、SmolVLA/pi0/GR00T、mobile base、VR bimanual、Go2 整合、自動成功判定。下面標 `長期` / `W3` 的東西，W2 一行 code 都不要為它寫。
2. **W2 唯一真的會「動到」的參考是 SO-ARM100 的 BOM / 電壓表**（拿去填 `docs/02-bom.md` v0 採購清單、核對 5V/12V）。其餘 leLab / XLeRobot / LeKiwi / hackathon / VR / 大模型全部只進這張表與 `docs/learning-roadmap.md` 當參考。
3. **硬體狀態以 run-log 為準，不以 upstream README 為準。** `reference/` 各 repo 的 README（含 MakerMods upstream）描述的是**它們的**能力，不是 Anvil 的進度。Anvil 的實機真相只在 `docs/hardware-state.md` 與 `docs/04-run-log/*`。任何「我們已經能 X」的聲明若沒有 run-log / failure-log 佐證，一律寫 **TBD / 待驗證**。

**Anvil 定位提醒**：Anvil / X0（GitHub `roy4222/Roy-XLeRobot-SO101`，本機 `AnvilBot`）是與 Go2 / PawAi 平行的**獨立機器人主線**，不是它們的手臂子模組。

---

## 1. 寶藏總表（一眼版）

| 來源 | 性質 | 主要可用點 | copy-idea / copy-code / study | 何時用 | 第一風險 |
|---|---|---|---|---|---|
| **SO-ARM100**（`reference/SO-ARM100`） | first-class 硬體源頭 | BOM 採購連結、電壓/扭力規格、STL/STEP/URDF、相機座 | copy-code（CAD/STL/URDF）+ study（規格） | **W2**（BOM/電壓）→ 長期（sim URDF、相機座） | 12V 誤插 leader 馬達 = 燒毀 |
| **leLab**（`reference/leLab`） | first-class 軟體棧 | teleop/calibrate/record/train/jobs 的流程與 CLI 包裝、多機 config + port 偵測 | copy-idea（流程/架構）；少量 copy-code 須去 SO101 硬編 | **W3**（record→train→rollout web 流程） | **Python 3.12+ floor** 撞 Pi 的 3.10.20 / lerobot 0.4.4 |
| **XLeRobot**（`reference/XLeRobot`） | glance 平台原型 | dual-wheel 底盤、`SO101Kinematics` IK、ManiSkill 雙臂 sim、FastAPI+React web | copy-idea（架構）；底盤/IK 可選 copy-code | **長期**（mobile base，W2 排除）；IK/web 可 W3 | 多處 sim/VR/RL 標「official coming soon」= 半成品 |
| **LeKiwi**（`SIGRobotics-UIUC/LeKiwi`，含 leLab 出處） | 行動底盤 + 軟體棧參考 | 堆疊底板 CAD、holonomic 底盤運動學 | copy-code（CAD）+ study（運動學） | **長期**（mobile base 升級時） | holonomic 比差速複雜；需 Fusion 360 匯出 |
| **Binh's LeRobot notes**（`lerobot.binhph.am`） | study-only 參考站 | policy 比較矩陣、推論取捨、reward model 全景 | study-only | **W3**（選 ACT vs diffusion 的決策） | 站內容 React 動態載入，無靜態摘要 |
| **Hackathon repos**（多個 GitHub） | copy-idea 為主 | 擾動式資料策略、技能串接、VLM 狀態機、Solo CLI wrapper | copy-idea；少量 copy-code（pexpect wrapper） | **W3**（dataset→ACT 資料策略 / 編排） | 多為 demo 級、無錯誤恢復；任務特定不一定泛化 |
| **VR / 大模型**（Phosphobot、SO-101-VR-Control、pi0/GR00T/Cosmos3） | study-only / phase-2 | VR teleop、VLA、world-model | study-only | **長期 / phase-2** | **W2 全部排除**；商業平台授權/延遲未知 |

> 路徑說明：`reference/` 三個 repo（XLeRobot / SO-ARM100 / leLab）全部 **gitignored**（本 repo `.gitignore` 僅一行 `reference/`），由父 repo 的 vcstool 拉取，本 repo 不 commit 它們；**永不修改**。LeKiwi / hackathon / VR repos 為外部 GitHub，僅 study/clone 對照，不納入本 repo。

---

## 2. SO-ARM100 — first-class 硬體源頭（W2 會真的用）

**權威範圍**：BOM 採購連結、3D 列印規格、STL/STEP/URDF/MJCF、馬達電壓與扭力、相機座設計。
**非權威**：軟體 setup / 校正 code / gripper 控制映射 —— 那些以 LeRobot library 為準。

| 資源 | 可用點 | 模式 | 何時 | 前置 / 風險 |
|---|---|---|---|---|
| `README.md` BOM（雙臂 / 單 follower） | 直接抄成 `docs/02-bom.md` v0 採購清單（含 Alibaba/Amazon/EU/CN 連結；金額**以 SO-ARM100 README 為準、採購前複查**，README 當時量級雙臂約 $230、單 follower 約 $122） | copy-code（清單）+ study | **W2 P0**（給老師的採購清單） | 連結與價格時效會變，採購前複查 |
| `README.md` 馬達規格 | **7.4V STS3215 ≈ 16.5kg·cm（leader 永遠 7.4V）；12V 版 ≈ 30kg·cm（follower 可升 12V，需約 12V 5A PSU）** | study（安全核對） | **W2**（核對四手電壓） | **12V 誤插 7.4V leader 馬達＝直接燒**；電源須實體貼標 `5V→LEADER` / `12V→FOLLOWER`；5A 為保守值，以實機線材標示為準 |
| `3DPRINT.md` / `README.md` 列印規格 | PLA+、0.2mm 層高、15% infill、Z-up 少支撐；列印前用 gauge 驗準度 | study | 長期（重印件時） | 不同床尺寸用不同合併 STL |
| `Simulation/SO101/*` URDF / MJCF | `so101_new_calib.urdf`（預設，virtual zero 在中段）、MuJoCo MJCF、joints_properties | copy-code（URDF for sim） | 長期（sim、IK） | gripper 在 LeRobot 為線性關節（0=閉 100=開），URDF/MJCF 尚未反映 |
| `Optional/Wrist_Cam_Mount_*`、`Overhead_Cam_Mount_*` | D405 / 32×32 UVC / overhead webcam 相機座（M2/M3 規格、解析度建議） | copy-code（STL）+ study | 長期（加相機收 dataset 時） | UVC 模組最低 720p/30fps 為建議非硬限 |
| `STEP/SO101`、`Optional/Compliant_Gripper` | 源 CAD（可改）、TPU 軟夾爪（drop-in） | copy-code | 長期 | TPU 95A 列印 |

**對照實機**：四手目前用的馬達/電壓/控制板/校正狀態，**一律以 `docs/hardware-state.md` 與 run-log 為準**——
- Follower #1：6× STS3215-C018 / 12V / 1:345，板 `...3925`，已校正 `roy_follower`（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`）。
- Follower #2：6× STS3215 / 12V，板 `...0335`，已校正 `roy_follower2`（`docs/04-run-log/2026-06-27-follower2-bringup.md`）。
- Leader #1：6× STS3215 **混齒比 / 7.4V→配 5V**，已寫 ID，組裝與 bus 健康 **TBD**（`docs/04-run-log/2026-06-27-leader-bringup.md`）。

**Leader 逐關節齒比（一律用 run-log 值，禁抄被組裝 runbook 推翻的舊對應）**：

| 關節 (ID) | 功能 | 齒比 / 型號 |
|---|---|---|
| L1 | shoulder_pan 底座 | 1/191 C044 |
| L2 | shoulder_lift 肩抬 | 1/345 C001 |
| L3 | elbow_flex 肘 | 1/191 C044 |
| L4 | wrist_flex 腕俯仰 | 1/147 C046 |
| L5 | wrist_roll 腕滾 | 1/147 C046 |
| L6 | gripper 握把扳機 | 1/147 C046 |

> 分布 = 1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。出處：`docs/04-run-log/2026-06-27-leader-bringup.md` L17–26。混齒比裝錯關節 = 力矩不匹配，務必照此表。

---

## 3. leLab — first-class 軟體棧（W3 抄流程，W2 不引入）

**一句話**：leLab 是 HuggingFace 官方為 LeRobot 做的 **FastAPI + React web UI**，把 calibrate→teleoperate→record→train→rollout 全包成瀏覽器流程。它**不 fork lerobot**，所有 train/record/rollout 都 shell out 到上游 CLI。**SO-101 硬編**、**Python 3.12+ 硬底線**、lerobot pin 在特定 commit。

**為什麼 W2 不引入**：Roy 的 Pi 跑 **Python 3.10.20 + lerobot 0.4.4 Seeed fork**（`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`）。leLab 要 3.12+ 且 pin commit，硬碰會炸版本。W2 的 teleop 用 Roy 手上**已驗證的 `lerobot-teleoperate` CLI** 就夠，不為了 UI 引入一整套 FastAPI+React + 版本衝突。leLab 延到 W3 當 **copy-idea**（決策已記 `docs/07-decisions.md`）。

| leLab 模組 | 可用點 | 模式 | 何時 | 風險 / 前置 |
|---|---|---|---|---|
| `teleoperate.py`（340 行） | leader→follower 鏡像迴圈、20FPS WebSocket 廣播、同步連線握手、雙裝置安全斷線 | copy-idea（流程）；copy-code 須換掉 `SO101*Config` 與 `_SO101_URDF_CORRECTIONS` | **W3** | 校正後 leader/follower 都在開放 serial port |
| `calibrate.py`（537 行） | 逐關節範圍記錄、位置輪詢、`threading.Event` UI 同步、編碼器 wrap 警告 | copy-idea | W3（也可 study 校正流程） | SO-101 硬編路徑 |
| `record.py`（871 行） | episode/reset 相位機、相機平台後端 pin（Linux V4L2）、resume、HF Hub 上傳 | copy-idea；patch 硬體後 copy-code | **W3 dataset 核心** | 相機後端平台特定；中斷會壞 dataset（用 atomic write） |
| `train.py`（188 行） | `TrainingRequest` schema → `build_training_command()` 組 `lerobot_train` CLI；無硬體硬編 | copy-code（純 CLI builder，最乾淨） | **W3** | accelerate / W&B 可選 |
| `jobs.py`（1272 行） | JobRegistry、local subprocess 生命週期（survive uvicorn reload）、stdout metrics 解析、checkpoint 列舉 | copy-idea（W3 backbone） | W3 | HF Cloud 需 API key |
| `utils/config.py`（450 行） | 多機 robot record（JSON per robot）、port 偵測（拔→插 15s poll）、校正檔雙位置複製 | copy-code（最少摩擦） | W2/W3 共用（W3 才真用） | 校正快取在 `~/.cache/huggingface/lerobot/` |
| `server.py`（1268 行）/ 前端 5 頁 | FastAPI 40+ endpoint、ConnectionManager 廣播；React calibrate/record/train/inference 頁、URDF 3D viewer、相機 modal | copy-idea / study | W3 UX | 前端 SO-101 URDF 路徑硬編 |
| `pyproject.toml` | 依賴矩陣參考（FastAPI / websockets / lerobot pin commit） | reference | — | **Python 3.12 硬底線**；lerobot commit 會 drift，更新時搜 `SO101LeaderConfig`/`SO101FollowerConfig` |

**W3 重用路線（粗）**：copy `train.py` + `jobs.py` + `datasets.py`（無硬體硬編）→ 接 record 流程 → 接既有 lerobot CLI。teleop 端若要 web 化，再 copy `teleoperate.py` 並把 `SO101*` 全換成 Anvil config。

---

## 4. XLeRobot — glance 平台原型（長期 mobile base）

**定位**：雙臂 + 行動底盤 + web control 的原始平台。**W2 排除 mobile base**，這裡只當未來菜單。注意官方多處標「official code coming soon」= 半成品，別當穩定品依賴。

| 資源 | 可用點 | 模式 | 何時 | 風險 / 前置 |
|---|---|---|---|---|
| `SO101Kinematics`（`SO101Robot.py`） | 雙臂解析 IK（EE 位置控制，無奇異點 workaround） | copy-idea；可選 copy-code | W3（若要 EE 控制） | 已在 ManiSkill + 真機測過 |
| Dual-wheel 差速底盤 v0.4.0（servo） | 官方建議的穩定底盤、差速運動學、STL | study；長期 copy-code | **長期**（W2 排除） | servo 設定同 Feetech |
| 3-omni v0.3.0 / Mecanum 4-wheel | 舊 omni（legacy）/ holonomic（實驗） | study | 長期 stretch | omni 慢、Mecanum 4 馬達複雜 |
| MuJoCo / ManiSkill 雙臂 sim | 鍵盤/Xbox/VR teleop 範例、ReplicaCAD 場景、dataset 錄製範例 | copy-idea；sim 可 USE-AS-IS | 長期（sim 驗證、RL） | ManiSkill 渲染建議 NVIDIA GPU |
| FastAPI + React web control（server/client + ZeroMQ bridge） | web teleop 後端骨架、命令限流、robot/sim 後端可換 | copy-idea | W3（web 化時） | 與 leLab 功能重疊，二選一 |
| SmolVLA / ACT pipeline（`VLA_smol.md` / `VLA_ACT.md`） | 三相機 ~20 episode → drawer/pick/zipper 的 record→train→eval recipe | copy-idea | **W3+（學習週，非 W2）** | VLA 教學標「temporary / official coming soon」，勿當 production |
| 硬體 STL（`XLeRobot_0_4_0_extra.stl` 等）、ODrive | 底盤/夾爪/相機 gimbal 列印件；brushless 驅動參考 | copy-code（STL）/ study | 長期 | ODrive 未整進主 codebase |

---

## 5. LeKiwi — 行動底盤 + 軟體棧出處（長期）

| 資源 | 可用點 | 模式 | 何時 | 風險 / 前置 |
|---|---|---|---|---|
| 堆疊底板 CAD（20mm 間距標準板） | 機構設計直接重用 | copy-code（CAD） | **長期**（mobile base 升級） | 需 Fusion 360 匯出 |
| holonomic（3-wheel Kiwi）底盤運動學 | 全向移動運動學參考 | study | 長期 | holonomic 比差速複雜，但工作空間更全向 |
| 平台架構（SO-ARM101 + Pi 5 + 雙 RGB） | 與 Anvil 同型，架構決策對照 | study-only | 長期 | LeKiwi 未被 XLeRobot 直接整合，只借機構原則 |

---

## 6. Binh's LeRobot notes — 決策參考站（W3 選 policy）

| 資源 | 可用點 | 模式 | 何時 | 風險 |
|---|---|---|---|---|
| `lerobot.binhph.am` | 所有 visuomotor policy 比較矩陣、推論取捨、reward model 全景 | study-only | **W3**（選 ACT vs diffusion vs VLA 的決策參考） | 站內容 React 動態載入，無靜態摘要，需互動瀏覽 |

---

## 7. Hackathon repos — 資料策略 / 編排（W3 copy-idea）

W2 不碰。這些是 W3「dataset→ACT」時的**資料策略與多技能編排**靈感來源；多為 demo 級、無錯誤恢復、任務特定，泛化性 TBD。

| Repo | 可用點 | 模式 | 何時 | 風險 / 前置 |
|---|---|---|---|---|
| **anjalidhabaria/physical_ai_hack_eta**（Coffee 倒咖啡） | **擾動式資料策略：40 變化 demo > 100 靜態 demo**；分離式 multi-view encoder 勝共享；30-chunk ACT 較 32 平滑 | copy-idea | **W3 dataset** | 擾動需領域知識；分離 encoder 參數翻倍；結果倒飲料特定 |
| **Neil7281/Physical-AI-hack-2026**（Coffee Pouring） | 三段 ACT 串接（Grasp→Traverse→Putback）、pexpect 包 Solo CLI | copy-idea（串接）+ copy-code（pexpect wrapper） | W3（多 policy pipeline） | 各 policy 獨立訓練，誤差跨鏈累積、無錯誤恢復 |
| **alisoncossette/ladybugs-robotics**（讀書機器人） | 感知驅動狀態機（`assess_scene` loop）、三執行緒 TTS、Claude Vision 當任務分類器 | copy-idea（狀態機/threading）+ study（VLM-as-selector） | W2 teleop 編排概念 / W3 多技能 | 緊耦合 Claude Vision + ElevenLabs；VLM 呼叫 +100–200ms 延遲 |
| **poorvirhebbar/insertion_physical_ai**（插入） | Solo CLI record-train-inference 整合、迭代資料收集 | copy-code（Solo CLI 模式）+ study | W2–W3 workflow 加速 | 需 Solo CLI 生態；資料策略新意有限 |
| **MuammerBay/isaac_so_arm101**（Isaac Lab RL） | SO-ARM101 的 Isaac Lab RL/IK、Sim2Real bridge、reaching task | study；選擇性 copy-code（task config） | **長期 W4+**（若走 RL） | 需 Isaac Sim license（$4.5k+）；僅走 RL 才值得 |
| **praveenVnktsh/robohacks26**（藥局） | 語意地圖（video→點雲→2D occupancy + 標物）、VLM 物件增強 | study-only | 長期（需導航時） | overkill；需 LiDAR + 腕相機 + STT 依賴 |
| **dragonkhoi/RoboCafe** | （LeRobot 重打包，無新貢獻） | SKIP | — | 與 LeRobot 本身重複 |

---

## 8. VR / 大模型 — phase-2（W2 全部排除）

| 來源 | 可用點 | 模式 | 何時 | 風險 |
|---|---|---|---|---|
| **phospho-app/phosphobot** | 統一 web UI（VR Quest / gamepad / leader / 鍵盤）、record-train-deploy、ACT/SmolVLA/π0.5/GR00T、HF 匯出 | study-only（架構）；謹慎 copy-code | phase-2 | YC W24 商業平台，授權/定價未知；與 LeRobot 訓練重疊只多 VR 層 |
| **kabilankb/SO-101-VR-Control** | 瀏覽器 WebXR（Quest）teleop、雙控制器→單臂、PyBullet IK、馬達校正安全邊界（5° buffer） | study-only；可 copy-code（WebXR+IK 架構） | phase-2 | Quest only；20Hz WebSocket 延遲；IK 非即時 |
| **pi0 / GR00T / Cosmos3 等** | VLA、world-model | study-only | phase-2 / 長期 | 學習週素材，W2 一行不寫 |

> 提醒：findings 裡這些被標「可用」是指**它們本身能跑**，不是 Anvil W2 該做。W2 明確 NOT：VR bimanual、VLA、world-model、mobile base、Go2 整合。

---

## 9. 風險 / 前置條件總覽

| 類別 | 條目 | 緩解 / 出處 |
|---|---|---|
| **安全（最高）** | 5V/12V 誤插燒 leader（7.4V）馬達 | 電源實體貼標 `5V→LEADER` / `12V→FOLLOWER`；`docs/04-run-log/2026-06-27-leader-bringup.md` |
| **安全** | leader 混齒比裝錯關節 = 力矩不匹配 | 照 §2 run-log 齒比表，拒用舊 runbook 對應 |
| **版本衝突** | leLab Python 3.12+ vs Pi 3.10.20 / lerobot 0.4.4 | W2 不引入 leLab，用 `lerobot-teleoperate` CLI；leLab 延 W3（`docs/07-decisions.md`） |
| **硬體 blocker** | Follower #1 F3 接頭 marginal 掉封包 | W2 teleop 改配 **Leader #1 → Follower #2（乾淨 bus）**；F3 修復排在正式錄資料前；`docs/05-failure-log.md` |
| **未驗證** | Leader #1 bus 健康（只寫 ID，未組裝/scan/校正） | 一律寫 TBD；補完前不稱 leader ready；`docs/04-run-log/2026-06-27-leader-bringup.md` |
| **未驗證** | 控制板 `...4639` SUSPECT 未複驗（USB 能列舉 ≠ bus 健康） | 不蓋棺、不在關鍵路徑；`docs/05-failure-log.md` |
| **參考品質** | XLeRobot VR/RL/VLA 多標「official coming soon」 | 當 prototype/study，勿當穩定依賴 |
| **參考品質** | hackathon demo 無錯誤恢復、任務特定 | 只抄資料策略/編排概念，泛化性自行驗證 |
| **採購** | SO-ARM100 BOM 連結時效、二手馬達來源混雜 | 採購前複查連結；來源混雜記錄即可（非 blocker） |
| **授權** | Phosphobot 商業平台定價未知 | study-only，不依賴 |

---

## 10. 何時用什麼（時程速查）

- **W2（現在）**：只動 **SO-ARM100 BOM/電壓**（→ `docs/02-bom.md`、核對四手電壓）；teleop 用既有 `lerobot-teleoperate` CLI（Leader #1 → Follower #2）。其餘全部只看不抄。
- **W3（dataset → ACT，學習週）**：leLab（record/train/jobs copy-idea）、hackathon 資料策略（擾動式 demo、分離 encoder、技能串接）、Binh notes（選 policy）、XLeRobot SmolVLA/ACT recipe。
- **長期 / phase-2**：XLeRobot dual-wheel 底盤 + IK + web control、LeKiwi CAD/運動學、Isaac Lab RL、VR（Phosphobot / SO-101-VR-Control）、world-model（Cosmos3）、語意地圖（robohacks）。

---

## 11. 相關文件

- `docs/hardware-state.md` — 四手實機狀態單一真相（W2 P0；本表硬體聲明的來源）。
- `docs/learning-roadmap.md` — 分階 AI 路線（ACT baseline → HIL-SERL/Diffusion → SmolVLA/X-VLA → world-model；抬頭：W2 非學習週）。
- `docs/02-bom.md` — v0 採購清單（源自 §2 SO-ARM100 BOM）。
- `docs/04-run-log/*`、`docs/05-failure-log.md`、`docs/06-metrics.md`、`docs/07-decisions.md` — Evidence 與決策。
- `docs/open-questions.md` — 滾動 TBD（板 `...4639` 複驗、F3 修復、Leader #2、leLab 版本衝突等）。

> 規則重申：本表只負責「指路」。任何要宣稱已完成的事，先回 `hardware-state.md` / run-log 找 Evidence；找不到就寫 **TBD / 待驗證**。
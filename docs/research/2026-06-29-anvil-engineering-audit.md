# Anvil / X0 工程稽核報告（2026-06-29）

> 稽核對象：`roy4222/Roy-XLeRobot-SO101`（本機目錄 `AnvilBot`），RoyBot-Lab 的 child repo。
> 證據基準：`docs/04-run-log/2026-06-26-follower-motor-id-setup.md`、`2026-06-27-follower2-bringup.md`、`2026-06-27-leader-bringup.md`、`docs/05-failure-log.md`、`docs/06-metrics.md`、`docs/07-decisions.md`。所有未經 run-log / failure-log 佐證者一律標 **TBD / 未驗證**。

---

## 1. 執行摘要（目前 repo 最大問題與建議方向）

**一句話結論**：硬體進度真實且乾淨（2 隻 follower 已 bring-up、leader #1 已寫 ID），但 **repo 的「可讀真相面」嚴重落後於實機進度**——一個新接手的人打開 GitHub 看到的是 upstream MakerMods 的 README，而不是 Roy 的實際狀態；真相全埋在三份 run-log 裡，沒有一張「四手狀態總表」把它收斂起來。

**三個最大問題（按優先序）**：

1. **沒有單一真相的四手狀態表**。Leader/Follower × ID/齒比/控制板/port/校正/組裝/blocker 這些值散在 3 份 run-log + failure-log + decisions，彼此要交叉比對才拼得出來。這正是 W2 P0 的核心交付，但目前不存在 → 應建立 `docs/hardware-state.md`。
2. **門面誤導**：`README.md` / `README_cn.md` / `LICENSE` 是 MakerMods upstream 內容（MIT 2025 MakerMods，非 Roy），`docs/03-architecture.md`、`01-setup.md`、`02-bom.md` 還是 TBD 骨架。下一個 agent 很容易把 upstream README 當成 Roy 的進度。`00-overview.md` 雖有實質內容但現況停在 2026-06-26（只記第一隻 follower）。
3. **關鍵阻礙未在顯眼處標示**：Follower #1 的 **F3 接頭 marginal 掉封包**（teleop/錄資料會不穩）、控制板 `...4639` **SUSPECT 未複驗**、Leader #1 **servo bus 健康未驗證**（只寫了 ID、尚未組裝/scan）。這些是 blocker，卻只在 run-log 內文，沒有彙整。

**建議方向**：W2 不要碰 VLA / VR / world-model / mobile base / Go2，把力氣全部放在「**收斂真相 + 補完 leader #1 + 跑出第一個 leader→follower 空中動作 teleop**」。文件面只做能直接餵 W2 P0 的幾份（hardware-state、02-bom 採購清單、sharpen 00-overview、08-media-index 收證據），其餘 reference/roadmap 類文件排在 P1，避免開太多坑。

---

## 2. 目標對齊（Goal alignment）

**我理解的 Anvil / X0 定位**：低成本「輪式雙臂 Physical-AI 平台」，是 Go2 / PawAi（Furnace OS）的**平行主線**，不是它的子模組。北極星＝一台可重現、低成本、會移動、能用雙臂操作、能收 teleop 資料、能訓 policy、並逐步長出感知 / 語音 / world-model 的類人形機器人。夏季第一迴圈：硬體 → 校正 → leader-follower teleop → dataset → ACT → 真機推論 → eval → 失敗分析；首個任務「touch marker」是刻意保守的 pipeline 驗證任務。

**資深工程師表態：整體同意 PM/Roy 的定位與排序，但提出三點 sharpening 與一點 pushback。**

- **同意（強）**：「先用 leader→follower CLI 跑通最穩的那條路、把 VR 真機與四手霸王推到 phase-2」是正確的工程判斷（`07-decisions.md` 已記）。leader 馬達是低扭力混齒比、為當 leader 而設計，拿去當 follower 會「2 強 2 弱」，這個決策站得住。touch marker 當 pipeline 驗證任務也對——它驗的是「資料→訓練→推論→eval」這條管線本身，而不是任務難度。

- **Sharpening 1（teleop 配對去風險）**：W2 的「至少一條 leader→follower 空中動作 teleop」**應該配 Leader #1 → Follower #2**，不要配 Follower #1。理由：Follower #2（板 `...0335`）整串 scan 連 5 次全綠、校正全程無掉封包（`2026-06-27-follower2-bringup.md` L51, L55）；Follower #1 有 F3 marginal 接頭，teleop 下會間歇掉封包（`2026-06-26-follower-motor-id-setup.md` L62-67）。用乾淨的那隻去拿 W2 里程碑，F3 修復留到正式錄資料前再做，不要讓 P0 卡在硬體維修上。

- **Sharpening 2（不要把 leLab 拉進 W2）**：leLab 的 web 流程很香，但它**硬性要求 Python 3.12+** 且把 lerobot pin 在特定 commit；Roy 的 Pi 跑的是 **Python 3.10.20 + lerobot 0.4.4 Seeed fork**（`2026-06-26-follower-motor-id-setup.md` L6）。W2 的 teleop 直接用 Roy 手上已驗證的 `lerobot-teleoperate` CLI 就夠，把 leLab 列為 W3 的 record/train/job-orchestration **copy-idea 參考**，不要在 W2 為了 UI 引入版本衝突與一整套 FastAPI+React。

- **Sharpening 3（明確化 ACT 不是 W2）**：ai_roadmap / Cosmos3 的素材很完整，但 ACT baseline 是「夏季第一迴圈」的目標，**不是 W2**。建議把它收進 `docs/learning-roadmap.md` 並在抬頭明寫「W2 非學習週」，避免 roadmap 的存在誘導提早跳關。資料品質（乾淨 teleop demo）才是真正瓶頸，這點 roadmap 抓對了。

- **Pushback（輕，針對範圍蔓延風險）**：findings 裡 VR（Phosphobot / SO-101-VR-Control）、mobile base（LeKiwi / XLeRobot dual-wheel）、world-model（Cosmos3）都被標成「可用」。它們確實可用，但**全部不屬於 W2**（W2 明確 NOT：50-ep dataset、ACT 訓練、SmolVLA/pi0/GR00T、mobile base、VR bimanual、Go2 整合、自動成功判定）。我的立場是：這些只進 reference-map / learning-roadmap 當「未來菜單」，W2 一行 code 都不要為它們寫。

---

## 3. Repo 地圖（Repo map）

| 角色 | 名稱 / 路徑 | 性質 | 說明 |
|---|---|---|---|
| origin | `roy4222/Roy-XLeRobot-SO101` | Roy 主 fork | Anvil/X0 的正規 GitHub 名；Roy push 文件/證據到這裡 |
| upstream | `makermods-robotics/MakerMods-XLeRobot` | 硬體設計上游 | 3D 列印件 / 組裝 / 外觀；Roy 直接用 `.3mf` 不改 |
| ref-xlerobot | `Vector-Wangel/XLeRobot` | 平台原型參考（read-only） | 雙臂+行動底盤+web control 的原始平台；**glance 級** |
| ref-soarm100 | `TheRobotStudio/SO-ARM100` | 手臂規格源頭（read-only） | SO-100/101 的 CAD/STL/URDF/BOM/馬達規格；**first-class** |
| ref-lekiwi | `SIGRobotics-UIUC/LeKiwi`（內含 leLab） | 軟體棧 + 行動底盤參考（read-only） | leLab = lerobot 的 web 流程；行動底盤 CAD；**first-class（軟體）** |
| 執行環境 | Pi 5 @ Tailscale `pi5` / 100.104.92.26，`~/lerobot/.venv` | 不在本 repo | 真正跑 lerobot 的地方；本 repo 只存證據 |

**三個名字＝同一台機器人**：Obsidian 叫 **Anvil / X0** = GitHub `roy4222/Roy-XLeRobot-SO101` = 本機目錄 `AnvilBot`（CLAUDE.md 已記）。**這層關係在 GitHub UI 看不出來**（repo 名不含 Anvil/X0），是新人最容易踩的坑，應在 `00-overview.md` 開頭明寫。

**關係定位**：Anvil 與 Go2/PawAi 是**平行主線**，不是子模組。`reference/` 三個 repo 全部 gitignored（`.gitignore` 僅一行 `reference/`），由父 repo 的 vcstool 拉取，本 repo 不 commit 它們——這是正確的。

---

## 4. 真相來源（Truth sources）

**Roy 自寫真相（有實質內容，可信）**：
- `AGENTS.md`（硬規則）、`CLAUDE.md`（定位/git紀律/三名字對應）
- `docs/00-overview.md`（目標；但現況停在 2026-06-26，需更新）
- `docs/04-run-log/2026-06-26-follower-motor-id-setup.md`（Follower #1：ID/組裝/bus 排障/校正/首次動作）
- `docs/04-run-log/2026-06-27-follower2-bringup.md`（Follower #2：ID/板 `...0335`/校正 range）
- `docs/04-run-log/2026-06-27-leader-bringup.md`（Leader #1：6 顆混齒比/5V/ID 表）
- `docs/05-failure-log.md`（F3 接頭 marginal、板 `...4639` SUSPECT）
- `docs/06-metrics.md`（follower 校正 encoder range）
- `docs/07-decisions.md`（2F+2L 傾向、suspect 不蓋棺；**2 條都有實質內容，禁止覆蓋**）
- `docs/research/2026-06-26-so-arm101-assembly-runbook.md`（已對抗式查證的組裝指南）

**只是骨架 / TBD（別當成已完成）**：`docs/01-setup.md`、`docs/02-bom.md`、`docs/03-architecture.md`、`docs/08-media-index.md`（僅標題）、`docs/09-public-notes.md`。

**Upstream 疊加層（非 Roy 進度，勿引用為狀態）**：`README.md`、`README_cn.md`（MakerMods 硬體升級說明）、`LICENSE`（MIT 2025 **MakerMods**，非 Roy）、`3mf/`、`assets/`。

**Read-only 參考（永不修改）**：`reference/XLeRobot`、`reference/SO-ARM100`、`reference/leLab`。

**兩個必須守住的證據紀律**：
1. **「USB 能列舉」≠「servo bus 健康」**。板 `...4639` 能在 `/dev/serial/by-id/` 列舉，但 `scan_port` 回 `{}`、`RX_BYTES 0`（`05-failure-log.md`）——這是「bus 不健康」的反例。Leader #1 目前只到「寫了 ID」，**bus 健康仍 TBD**（`2026-06-27-leader-bringup.md` L32-36）。
2. **Leader 逐關節齒比一律用 run-log 值**（下表，已對 `2026-06-27-leader-bringup.md` L17-26 逐行核對），**不抄被組裝 runbook 明文推翻的舊對應**（runbook §4 C2 標示「ID1:1/345, ID2-3:1/147, ID4-5:1/191, ID6:1/345」為錯誤）。

| 關節 (ID) | 功能 | 齒比 / 型號 |
|---|---|---|
| L1 | shoulder_pan 底座 | **1/191 C044** |
| L2 | shoulder_lift 肩抬 | **1/345 C001** |
| L3 | elbow_flex 肘 | **1/191 C044** |
| L4 | wrist_flex 腕俯仰 | **1/147 C046** |
| L5 | wrist_roll 腕滾 | **1/147 C046** |
| L6 | gripper 握把扳機 | **1/147 C046** |

分布＝1×C001(1/345) + 2×C044(1/191) + 3×C046(1/147)。

**四手實況快照（W2 起點）**：

| 臂 | 馬達/電壓 | 控制板 | ID | 組裝 | bus | 校正 | blocker |
|---|---|---|---|---|---|---|---|
| Follower #1 | 6× STS3215-C018 / 12V / 1:345 | `...3925`(good) | ✅ | ✅ | ✅(3×) | ✅ `roy_follower` | **F3 接頭 marginal 掉封包** |
| Follower #2 | 6× STS3215(777) / 12V | `...0335`(good) | ✅ | ✅ | ✅(5×) | ✅ `roy_follower2` | 無；尚未做首次動作 demo |
| Leader #1 | 6× STS3215 混齒比 / **7.4V→配 5V** | `...0371` | ✅ | **待做** | **TBD** | **待做** | bus 健康未驗證 |
| Leader #2 | TBD（同 L1） | TBD | ❌ | ❌ | ❌ | ❌ | 未開始 |

控制板 `...4639` = **SUSPECT、暫停、未複驗**（不在關鍵路徑）。

---

## 5. 參考寶藏（Reference treasures）

| 來源 | 可用點 | copy-idea / copy-code | 何時 |
|---|---|---|---|
| **SO-ARM100**（first-class 硬體源頭） | BOM 與採購連結（v0 採購清單直接抄）、3D 列印規格、STL/STEP/URDF、馬達規格與電壓（7.4V vs 12V，**12V 誤插 leader 燒馬達**）、相機座 D405/UVC | **copy-code**（CAD/STL/URDF）+ **study**（規格） | **W2**（採購清單、電壓核對）→ 長期（URDF for sim、相機座） |
| **leLab**（first-class 軟體棧） | `teleoperate.py` 鏡像迴圈、`calibrate.py` 範圍記錄、`record.py` episode/reset 相位機、`train.py`/`jobs.py` CLI 包裝與 job 生命週期、`utils/config.py` 多機設定 + port 偵測 | **copy-idea**（流程/架構）；少量 copy-code 須改 SO101 硬編 | **W3**（record→train→rollout 的 web 流程）。**W2 不引入**：Python 3.12+ 與 Pi 的 3.10.20/lerobot 0.4.4 衝突；W2 用 `lerobot-teleoperate` CLI 即可 |
| **XLeRobot**（glance） | dual-wheel v0.4.0 底盤、`SO101Kinematics` IK、ManiSkill 雙臂 sim、FastAPI+React web control | **copy-idea**（架構）；底盤/IK 可選 copy-code | **長期**（mobile base，W2 明確排除）；IK/web 可 W3 |
| **LeKiwi** | 堆疊底板 CAD、holonomic 底盤運動學 | **copy-code**（CAD）+ **study**（運動學） | **長期**（mobile base 升級時） |
| **Binh's LeRobot notes** | policy 比較矩陣、推論取捨、reward model 全景 | **study-only** | **W3**（選 ACT vs diffusion 時的決策參考） |
| **Hackathon repos** | 擾動式資料策略（anjalidhabaria：40 變化 demo > 100 靜態）、分離式 encoder、技能串接（Neil7281）、VLM 狀態機（Ladybugs） | **copy-idea**（資料策略/編排）；pexpect Solo CLI wrapper 可 copy-code | **W3**（dataset→ACT 的資料策略與多技能編排） |
| **VR / 大模型**（Phosphobot、SO-101-VR-Control、pi0/GR00T/Cosmos3） | VR teleop、VLA、world-model | study-only | **長期/phase-2**，**W2 全部排除** |

**一句話導讀**：W2 只動到 **SO-ARM100 的 BOM/電壓**；leLab/XLeRobot/LeKiwi/hackathon/VR/大模型全部進 `reference-map.md` 與 `learning-roadmap.md` 當未來菜單，不進 W2 程式。

---

## 6. 建議文件變更（Docs changes proposed）

**硬約束遵守**：現有 00-09 scaffold 編號**不 renumber、不覆蓋有實質內容的檔**（`04-run-log/*`、`05-failure-log`、`06-metrics`、`07-decisions` 一律保留）；新主題用具名檔；07 用**附加**；03 填進既有 skeleton；00 做 sharpen。

**新增（CREATE，具名檔）**：

| 檔案 | 角色 | 優先序 |
|---|---|---|
| `docs/hardware-state.md` | **四手狀態總表**——臂×ID/齒比/控制板/port/校正/組裝/blocker 的單一真相。內容從 3 份 run-log 收斂（含上方第 4 節快照表 + leader 齒比表）。**這是 W2 P0 中心交付** | **W2 P0** |
| `docs/reference-map.md` | reference/ 各 repo 的 first-class vs glance 分級 + copy-idea/copy-code + W2/W3/長期時程（收第 5 節綜整） | P1（W2 末） |
| `docs/learning-roadmap.md` | 分階 AI 路線：ACT baseline（夏季第一迴圈）→ HIL-SERL/Diffusion → SmolVLA/X-VLA → Cosmos/world-model。**抬頭明寫「W2 非學習週」**，資料品質為瓶頸 | P1 |
| `docs/repo-map.md` | 本 child repo 自身的 remote/reference 角色表（origin/upstream/ref-*、gitignore、Pi 執行環境）；與父 repo 的 repo-map 互補 | P2 |
| `docs/open-questions.md` | 滾動式 TBD/未驗證清單：板 `...4639` 複驗、F3 修復、Leader #2、馬達方向預設反轉是否需驗 LeRobot config、leLab Python 3.12 vs Pi 3.10 衝突、二手馬達來源混雜 | P2 |

**更新（UPDATE，既有檔）**：

| 檔案 | 動作 | 內容 |
|---|---|---|
| `docs/00-overview.md` | **sharpen**（保留現有目標段） | 開頭加「三名字＝同一台機器人」與「Anvil 為 Go2/PawAi 平行主線、非子模組」；現況更新到 2026-06-29（2 follower bring-up ✅、Leader #1 僅 ID）；明列 **W2 scope 與 NOT-this-week**（無 50-ep dataset/ACT/VLA/mobile/VR bimanual/Go2/自動成功判定） |
| `docs/03-architecture.md` | **填 TBD skeleton** | runtime map：Pi 5 @ 100.104.92.26、`~/lerobot/.venv`（py3.10.20 / lerobot 0.4.4 Seeed fork）、feetech sdk、校正快取路徑、board→port→6 馬達 daisy-chain 拓樸、leader→follower 資料流（`lerobot-teleoperate`）、本 repo＝純證據庫不執行 |
| `docs/07-decisions.md` | **附加新 ADR（保留既有 2 條）** | (a) W2 首次 teleop 配 Leader#1→Follower#2（乾淨 bus）去風險、F3 修復留待錄資料前；(b) leLab 因 Python 3.12 vs Pi 3.10/lerobot 0.4.4 衝突，延到 W3 當 copy-idea；(c) 重申 Anvil 為平行主線非 Go2 子模組 |
| `docs/02-bom.md` | **填 TBD skeleton** | **v0 採購清單**（W2 P0 交付，給老師）：以 SO-ARM100 BOM 為源、列現有 vs 待購、控制板/電源/線材/相機 |
| `docs/01-setup.md` | **填 TBD skeleton** | Pi 環境 + venv + lerobot 安裝 + 已驗證 run command（`lerobot-setup-motors` / `lerobot-calibrate` / `lerobot-teleoperate` / `FeetechMotorsBus.scan_port` 健檢）。把散在 run-log 的指令收斂 |
| `docs/08-media-index.md` | **填索引** | W2 證據索引：四臂貼標照、teleop 影片、terminal log（大檔只放索引不 commit，遵守硬規則） |

**本輪不動**：`docs/09-public-notes.md`（第一迴圈尾端再寫）、`docs/05-failure-log.md` / `06-metrics.md`（有內容，只在新事件時 append）、既有 `04-run-log/*`（只新增當日新檔，不改舊檔）。

---

## 7. 風險 / 阻礙（Risks / blockers）

**硬體**
- **F3 接頭 marginal 掉封包（Follower #1）**：靜態 scan 穩，但需回應的寫入間歇失敗，`num_retry=5` 只能繞過、**不可長期硬扛**（`2026-06-26-follower-motor-id-setup.md` L62-67；`07-decisions.md` L21）。對 teleop/錄資料是 blocker。緩解：W2 teleop 改用 Follower #2；錄資料前徹底重壓/更換 F3。
- **Leader #1 bus 健康未驗證**：只寫了 ID，組裝＋整串 `scan_port [1..6]`＋校正全 **TBD**（`2026-06-27-leader-bringup.md` L32-36）。不可宣稱 leader ready。
- **控制板 `...4639` SUSPECT 未複驗**：好馬達+好線單獨測尚未做（`07-decisions.md`、`05-failure-log.md`），不蓋棺、不在關鍵路徑。

**安全**
- **5V/12V 電源誤插＝燒 leader 馬達**：leader 7.4V 馬達必配 5V，12V 直燒（assembly-runbook §0；`2026-06-27-leader-bringup.md` L7）。電源須實體貼標 `5V→LEADER` / `12V→FOLLOWER`。
- **Leader 混齒比裝錯關節＝力矩不匹配**：務必照第 4 節 run-log 齒比表，**拒用** runbook 已標錯的舊對應。

**repo 整理**
- 門面誤導：README 為 MakerMods upstream、03/01/02 仍 TBD → 新人易誤判進度。對策見第 6 節。
- 未追蹤檔（`CLAUDE.md` 與 2 份 leader/follower2 run-log）尚未 commit；建議待 Roy 指示再 commit（遵守 git 紀律，不擅自 push）。

**run command**
- **leLab Python 3.12 floor vs Pi Python 3.10.20 / lerobot 0.4.4 衝突**：W2 勿引入 leLab，teleop 用既有 `lerobot-teleoperate` CLI。

**資料缺口**
- 尚無任何 teleop / dataset；Follower #2 首次動作 demo 未做；二手馬達來源混雜（記錄即可，非 blocker）。
- 馬達方向「預設反轉」是否需驗證 LeRobot `SO101*Config` 仍 TBD（`open-questions.md` 追蹤）。

---

## 8. 接下來 3 個行動（Next 3 actions，對齊 W2）

1. **補完 Leader #1（關鍵路徑）**：依 ID1 底座→ID6 握把組裝（混齒比放對關節、**5V 電源**）→ 整串 `FeetechMotorsBus.scan_port` 驗 `[1..6]` 全綠 → `lerobot-calibrate --teleop.type=so101_leader --teleop.port=/dev/ttyACM0 --teleop.id=roy_leader`。產出：新 run-log + scan 截圖。（在此之前 Leader bus 健康一律寫 TBD。）

2. **跑出 W2 里程碑：第一條 leader→follower 空中動作 teleop**——配 **Leader #1 → Follower #2（乾淨 bus）**，用 `lerobot-teleoperate`，無負載、無 policy，錄影片 + terminal log 當證據。避開 F3，先拿下 P0；F3 修復排在正式錄資料前。

3. **收斂 W2 P0 文件交付**：四臂貼標拍照 → 建 `docs/hardware-state.md` 四手狀態總表（含齒比表/板/port/校正/blocker）→ `docs/02-bom.md` 填 v0 採購清單給老師 → `docs/08-media-index.md` 索引照片/影片/log。同時 sharpen `00-overview.md`（三名字、平行主線、W2 scope/NOT）。

> reference-map / learning-roadmap / repo-map / open-questions / 01-setup / 03-architecture 列 P1-P2，W2 行有餘力再補，避免開太多坑。
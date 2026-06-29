# SO-ARM101 雙臂 + LeKiwi 今日組裝與首次 Teleop 實戰手冊

> 本手冊只根據提供的研究資料（本地 repo 擷取 + web 查證 + 對抗式裁決）綜合而成，未經查證或互相衝突處一律標 **待驗證/TBD**。涉及「會燒馬達」的安全步驟都附行內出處，並已套用對抗式裁決：凡被判 `refuted`（推翻）或 `uncertain`（不確定）的說法，**不以事實呈現**。

---

## 0. ✅ 已確認的實際出貨清單（2026-06-26，由實體 BOM 確定，取代第 1/5 章的「電壓版本 TBD」）

Roy 實際收到的出貨清單對應 **SO-101 Pro 版：1 支 Leader + 1 支 Follower 配對**。原手冊第 1、5 章的「電壓版本＝待驗證」**已解除，確定為 Pro 版（Leader 5V / Follower 12V）**。

**Leader 臂（6 顆，全 7.4V）→ 接 5V 電源供應器**
| 關節 | 馬達型號 | 齒比 |
|---|---|---|
| L1 底座旋轉 | STS3215-C044 | 1:191 |
| L2 肩抬 | STS3215-C001 | 1:345 |
| L3 肘 | STS3215-C044 | 1:191 |
| L4 腕俯仰 | STS3215-C046 | 1:147 |
| L5 腕滾 | STS3215-C046 | 1:147 |
| L6 夾爪 | STS3215-C046 | 1:147 |

**Follower 臂（6 顆 STS3215-C018，全 12V、全 1:345）→ 接 12V 電源供應器**
- F1~F6 同型號，無裝錯位置問題；仍須一次一顆設 ID 1~6。

**其餘**：伺服控制板（Seeed XIAO 版）×2、5V 電源 ×1、12V 電源 ×1、USB-C ×2、直流電源線 ×2。**本箱不含 LeKiwi 輪馬達/底盤件**（另箱，今日不處理）。

> 🔴 **最致命坑（Pro 版專屬）**：5V 與 12V 兩顆電源供應器 DC 接頭都是 5.5×2.1mm、外觀近似。**12V 電源誤插到 Leader（7.4V 馬達）會直接燒馬達。** 立即在電源上貼標籤：`5V→LEADER` / `12V→FOLLOWER`，分開放，每次接線前先確認。〔來源: wiki.seeedstudio lerobot_so100m_new — Pro 版電源規則；本手冊第 1、5 章〕

---

## 1. 速覽

### 今天的目標
- 完成 **2 支 SO-ARM101 的實體組裝**（3D 件已印好）。
- 設好 **馬達 ID（一次一顆）**、接好線與電源、跑完 **find port → calibrate → 首次 teleop / 首次動作**。
- LeKiwi 底盤與雙臂整合屬「相依但可延後」，今天若只組手臂可跳過第 7 章（但先讀其相依標註）。

### 需要的東西（最小集）
- 2× SO-ARM101（每支 6 顆 STS3215）、1~2 個馬達控制板、USB-C 資料線、對應電源供應器、#0/#1 十字螺絲起子、桌邊夾具、記號筆/標籤。〔本地: SO-ARM100/README.md | Sourcing Parts〕
- 電腦（Linux 假設 port 為 `/dev/ttyACM0`、`/dev/ttyACM1`）。
- 軟體：LeRobot（官方 CLI）或 leLab（網頁引導式）。

### 預估時間
- 單支手臂機構組裝：依官方流程約數小時級；XLeRobot 全機（雙臂+底盤+Pi）官方稱「約 4 小時內」。〔web: github.com/Vector-Wangel/XLeRobot, confidence high〕
- 今天只到「雙臂 + 首次 teleop」是合理範圍；LeKiwi 整合可另開一天。

### 最危險的 3 個坑（務必先讀）

1. **電壓接錯 → 永久燒毀馬達（最致命）。** STS3215 有 **7.4V 版**（用 ~5V/6V 供電，堵轉扭矩約 16.5 kg·cm）與 **12V 版**（堵轉約 30 kg·cm）兩種；**接錯電壓會損壞伺服**。〔web: github.com/.../SO100.md, confidence high〕**哪支臂用幾 V 取決於你的套件版本**，存在版本衝突（見下框與第 5 章），出手前務必先確認你手上的版本。
   - 同時：**電源線一律最後接；要插拔其他線（USB）前先斷電源**，否則會傷控制板。〔本地驗證: assemble.md §Wiring 重要標記, verdict=confirmed〕

2. **馬達 ID 設定必須「一次一顆」。** STS3215 出廠 ID 預設都相同（通常為 1），若先串聯（daisy-chain）再批量設定會 **ID 衝突、無法通訊**。必須單顆接板、逐顆寫 ID 1~6，全部設定完才串聯。〔web 驗證: huggingface.co/docs/lerobot/lekiwi + wiki.seeedstudio, verdict=confirmed〕

3. **Leader 臂的混合齒比（若你要組 leader+follower 配對）。** Leader 各關節用三種不同齒比，**裝錯關節會力矩不匹配、無法正常運作**，務必標記區分（L1–L6 / F1–F6）。注意：本地擷取資料給的「逐關節齒比對應」經查證為**錯誤**，正確對應見第 2 章/第 4 章。〔對抗裁決: 警告3 verdict=refuted〕

> **電壓版本衝突（必讀，今日 TBD 待你確認套件）**
> 來源彼此不一致，因為**取決於你買的是哪個版本**：
> - **SO101 Standard 版**：leader 與 follower **都用 5V**。〔web: wiki.seeedstudio lerobot_so100m_new, high〕
> - **SO101 Pro 版**：**Leader 5V、Follower 12V**，兩者不可混用，建議貼標/顏色區分。〔web: wiki.seeedstudio, high〕
> - **SO-ARM100/101 README 觀點**：leader 一律用 **7.4V 馬達**；follower 可選 7.4V（5V 供電）或 12V（12V 供電）。〔web: github SO-ARM100 README, high〕
> - **XLeRobot 雙 follower 整機**：採 **12V 版**（兩支臂都是 follower，皆 12V，需高功率電源）。〔本地: hardware_intro/index.md〕
>
> **行動指引**：先看你套件盒上的標示/電源供應器輸出（5V？12V？）與馬達標示（7.4V？12V？），**讓供電電壓符合馬達規格**。在未確認前，**不要通電**。你具體屬於哪一版 = **待驗證**。

---

## 2. 階段 A — 組裝前準備（BOM 對齊、工具、3D 件檢查）

### A1. BOM 對齊
- 每支 SO-ARM101 = **6 顆 STS3215**；follower 全部 **1/345 齒比**。〔web: huggingface so101, verdict=confirmed〕
- Leader（若要組）= **三種齒比混搭**：共 1 顆 1/345、2 顆 1/191、3 顆 1/147。〔web: huggingface so101 + github README, confirmed〕
- 其他：馬達控制板、USB-C 資料線、電源供應器（電壓見第 5 章）、#0 與 #1 十字起子、夾具。〔本地: SO-ARM100/README.md | Sourcing Parts〕
- 完整 BOM/採購連結/3D 列印說明的權威來源是 TheRobotStudio 的 SO-ARM100 repo README。〔web: github.com/TheRobotStudio/SO-ARM100, high〕

### A2. 工具
- #0 與 #1 Phillips 十字起子、尖嘴鉗（標記馬達、切割推車金屬網時用）、記號筆/標籤。

### A3. 3D 件檢查
- 依列印機尺寸選 STL：Ender(220×220) 用 `Ender_Follower_SO101.stl` / `Ender_Leader_SO101.stl`；Prusa/UP(205×250) 用 `Prusa_*_SO101.stl`。〔本地: SO-ARM100/README.md | Printing the Parts〕
- 建議：PLA+、層高 0.2mm、填充 15%，列印後移除支撐。〔本地: 同上〕
- 逐件檢查孔位/支撐殘料；**3D 件材料（PLA/PETG/TPU）對組裝強度的影響無測試數據 = 待驗證**。〔本地 open_question〕

> **SO-ARM101 vs SO-ARM100（組裝差異，影響你今天的流程）**
> 101 相對 100 的三大改動：①改良佈線（解決舊版 joint 3 易斷線、且不再限制關節活動範圍）②**組裝免拆/換齒輪**（更簡單）③leader 改用新齒比馬達。〔web: github README + openelab + wiki.seeedstudio, high〕101 為推薦版本，100 已棄用。

---

## 3. 階段 B — 馬達 ID 設定與方向（一次一顆）

> **鐵則：一次只接一顆馬達到控制板，寫好 ID 再接下一顆；全部設定完才串聯。** 出廠 ID 都相同，批量會衝突。〔web 驗證: huggingface lekiwi + wiki.seeedstudio + waveshare, verdict=confirmed〕官方建議在「機械組裝之前」就把 ID 設好，因為裝完再拆排線很麻煩。〔web: waveshare Set Servo ID, high〕

### B1. 你有兩條工具路線（擇一）

**路線 1：官方 LeRobot CLI（推薦給「兩支都當手臂用」的標準 SO-101 流程）**
先找 port：
```
lerobot-find-port
```
〔web 驗證: huggingface.co/docs/lerobot/so101, verdict=confirmed〕

Follower 設定 ID 與 baudrate（會逐顆引導；port 換成自己的）：
```
lerobot-setup-motors --robot.type=so101_follower --robot.port=/dev/tty.usbmodem585A0076841
```
〔web 驗證: huggingface so101, verdict=confirmed〕

Leader 設定（注意改用 `--teleop.*` 前綴）：
```
lerobot-setup-motors --teleop.type=so101_leader --teleop.port=/dev/tty.usbmodem575E0031751
```
〔web 驗證: huggingface so101, verdict=confirmed〕

**路線 2：Bambot 網頁工具（XLeRobot 文件採用；尤其底盤輪馬達官方 lerobot 目前不支援手臂以外馬達設定時）**
- 開 https://bambot.org/feetech.js ，逐顆連接→掃描→重新命名 ID。〔本地: assemble.md §Configure Motors〕
- Linux 權限不足先開放 port：`sudo chmod 666 /dev/ttyACM0`（重插 USB 後權限會重置；永久解法是把使用者加入 `dialout` 群組後重新登入）。〔web 驗證: nvidia troubleshooting, high〕
- **Bambot 在 Linux 是否需特殊驅動/權限 = 待驗證**（文件主要提 Windows/Mac）。〔本地 open_question〕

### B2. ID 分配
- **每支 follower**：ID **1~6**（由下到上對應關節，夾爪為 6）。〔本地: assemble.md §Configure Motors；web: waveshare〕
- **Leader**（若組）：同樣 ID 1~6，但齒比不同（見第 4 章）。
- **標記**：在每顆馬達用記號筆/標籤寫清楚用途，例如 **F1–F6（follower）/ L1–L6（leader）**，避免混用。〔對抗裁決: 警告3, 警告4, confirmed〕

### B3. 馬達方向 / 反轉
- 方向由 LeRobot 軟體設定（follower/leader 的預設 config 內定義），**SO-ARM101 各馬達的預設反轉狀態，可用文件未明列 = 待驗證**，需查 LeRobot 原始碼的 `SO101LeaderConfig` / `SO101FollowerConfig`。〔本地 open_question 1 + leLab/calibrate.py〕**不要在沒看到實際 config 前自行假設方向。**

---

## 4. 階段 C — 手臂機構組裝（leader 與 follower 差異；joint 順序；夾爪）

### C1. 共通組裝
- SO-ARM101 **免拆/換齒輪**，按 HuggingFace LeRobot 官方圖示由關節 1（底座旋轉）往上組到夾爪。〔web: github README; 本地: assemble.md §SO101 Arms〕
- Follower 與 Leader 的組裝步驟**大致相同**，主要差異在 **Step 12 之後的末端**：follower 裝**夾爪 gripper**，leader 裝**握把 handle**。〔web: wiki.seeedstudio lerobot_so100m_new, high〕

### C2. Leader 齒比逐關節對應（**已用查證後的正確值；本地擷取的對應是錯的**）
> 對抗裁決把本地資料的 leader 齒比對應判為 **refuted**。請用以下**官方對應**：〔web: huggingface so101 + wiki.seeedstudio, verdict=refuted→改用官方值〕

| 關節 | 功能 | 正確齒比 |
|---|---|---|
| L1 | Base / Shoulder Pan（底座旋轉） | **1/191** |
| L2 | Shoulder Lift（肩抬/Pitch） | **1/345** |
| L3 | Elbow Flex（肘彎） | **1/191** |
| L4 | Wrist Flex（腕俯仰） | **1/147** |
| L5 | Wrist Roll（腕翻滾） | **1/147** |
| L6 | Gripper（夾爪） | **1/147** |

- 分布 = 1×1/345 + 2×1/191 + 3×1/147。**Follower 則全部 6 顆 1/345。**〔web: huggingface so101, confirmed〕
- **不要採用本地擷取裡「ID1:1/345, ID2-3:1/147, ID4-5:1/191, ID6:1/345」這組對應 —— 經查證為錯誤，照它裝會把馬達裝到錯誤關節。**〔對抗裁決: 警告3, refuted〕

> **若你走 XLeRobot 路線（雙 follower）**：兩支都是 follower、**全 1/345**，沒有 leader 混合齒比問題，上面 C2 可略過。〔本地: assemble.md §SO101 Arms；web: xlerobot assemble〕

### C3. 關節限制（參考，非今日必設）
- URDF 限制（弧度）：Joint2 約 −0.1~3.45、Joint3 約 −0.2~π；夾爪線性關節 0=全閉、100=全開。〔本地: Simulation/SO101 README + SO101Robot.py〕
- **這些限制是否在校準時自動套用、或需手動設定 = 待驗證**。〔本地 open_question 2〕

---

## 5. 階段 D — 接線與供電（哪支臂 5V / 哪支臂 12V；USB/serial）

### D1. 接線順序（安全關鍵）
1. 先把 6 顆舵機用 5264 連接器**串聯**到控制板（紅=電源、黑=地、白=信號）。〔本地: assemble.md §Wiring〕
2. **自行延長 5264 線時注意極性，反向連接會導致錯誤。**〔本地驗證: assemble.md §Wiring, verdict=confirmed〕
3. USB-C→USB-A 資料線接電腦。
4. **電源線「最後」才接；之後要插拔任何其他線，務必先斷電源**，否則損傷馬達控制板。〔本地驗證: assemble.md §Wiring 重要標記, verdict=confirmed〕

### D2. 哪支臂幾伏特（依版本，務必先對照第 1 章電壓框）
- **讓供電電壓符合馬達規格**：7.4V 馬達配 ~5V 供電、12V 馬達配 12V 供電；**接錯會永久損壞**。〔web: github SO100.md, high；wiki.seeedstudio, confirmed〕
- 版本對照（再次強調，今日先確認）：
  - Standard：兩臂 5V。
  - Pro：Leader 5V、Follower 12V（不可混用）。
  - XLeRobot 整機：12V 版，**需高功率電源（12V、5A+ 等級；精確 5A 為保守值非官方逐字）**，不可用低功率供應器。〔對抗裁決: 警告2, verdict=confirmed（5A 為保守近似）〕
- **你的實際版本 = 待驗證**，未確認前不要通電。

### D3. 找不到馬達的快速判讀（接線自檢）
- **全部馬達都無回應** → 通常是**沒接電源**（USB 只負責通訊，馬達要另外供電）。〔web 驗證: nvidia troubleshooting, high〕
- **只缺單一 ID（如偵測到 2-6 缺 1）** → daisy-chain 某處接觸不良，會影響下游；重插電源、檢查排線。〔web: nvidia troubleshooting, high〕

---

## 6. 階段 E — 軟體環境與校正（安裝、find port、calibrate、首次 teleop）

### E1. 安裝
- LeRobot 之外，SO-101 需額外安裝 Feetech SDK：
```
pip install -e ".[feetech]"
```
〔web 驗證: huggingface so101, high〕
- 若用 **leLab**（網頁引導式）：
```
pip install -e .
```
**leLab 需 Python ≥ 3.12，舊版會安裝失敗。**〔對抗裁決: leLab Python≥3.12, verdict=confirmed〕leLab **硬編碼只支援 SO-101 leader/follower**，不支援其他機型；換硬體要改所有特徵模組的 config。〔對抗裁決: leLab hardcoded SO-101, verdict=confirmed〕
- **SO-ARM101 是否與 leLab 的「SO-101」為同一硬體別稱 = 待驗證**（leLab 文件只提 SO-101）。〔本地 open_question〕

### E2. 找 port
```
lerobot-find-port
```
〔web 驗證: confirmed〕Linux 權限不足：`sudo chmod 666 /dev/ttyACM0`（與 `/dev/ttyACM1`）。〔web: huggingface so101 + nvidia, high〕
- 插入順序可能決定埠號（第一個 = ttyACM0 = Follower，第二個 = ttyACM1 = Leader），但此為 **medium 信度，請以 `lerobot-find-port` 實測為準**。〔web: wiki.seeedstudio lerobot_so100m, confidence medium〕

### E3. 校正（calibrate）

**安全：校正會關閉馬達扭矩，手放開手臂會直接掉落 —— 全程用手扶穩。**〔web 驗證: docs.phospho.ai, high〕

**做法（軟體層問題，非硬體危險）：**
- 校正流程要求**先把每個關節移到行程中段（中位）再掃過全行程的 min/max**。把關節置中是為了避開 12-bit 編碼器（0–4095，中位 2048）的換行邊界，避免記到錯誤範圍。〔web 驗證: huggingface so101, verdict=uncertain（核心做法為真）〕
- **澄清（已被對抗裁決推翻的說法，不要當真）**：本地資料提到的具名例外 `CalibrationDiscontinuityError` **查無實證**，且編碼器換行**不是不可逆硬體損壞**，只是校準資料/範圍的軟體問題，重新校準即可解決。〔對抗裁決: 警告1 verdict=refuted；encoder-wrap verdict=uncertain〕
- 推關節時要推到**真正的機械止點**，不要被卡住的線材/障礙物擋住而記到錯誤 min/max。〔web 驗證: nvidia troubleshooting, high〕
- **重新校正前，先完整刪除舊校準檔**（`~/.cache/huggingface/lerobot/calibration/robots` 或 `.../teleoperators`），否則會報錯。〔web 驗證: waveshare, high〕

**官方 CLI 指令（port 與 id 換成自己的）：**
```
lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/tty.usbmodem58760431551 --robot.id=my_awesome_follower_arm
```
```
lerobot-calibrate --teleop.type=so101_leader --teleop.port=/dev/tty.usbmodem58760431551 --teleop.id=my_awesome_leader_arm
```
〔web 驗證: huggingface so101, 兩者皆 verdict=confirmed〕

> **指令版本注意**：本地擷取出現的 `lerobot calibrate so_follower --port /dev/ttyACM0` 屬**較舊/不同介面**；查證顯示官方現行新版 CLI 進入點是上面 `lerobot-calibrate --robot.type=...` 形式。**以官方新版為準**，舊式僅供對照。〔web 驗證: huggingface so101, confirmed；本地: leLab README〕

**或用 leLab 網頁引導式**：`lelab`（uvicorn:8000，自動開瀏覽器）或 `lelab --dev`（另起 Vite:8080）；在 http://localhost:8000 選裝置類型/port/config ID，依提示完成原點與範圍錄製。校準檔存於 `~/.cache/huggingface/lerobot/calibration/`。〔本地: leLab CLAUDE.md + README〕

### E4. 首次 teleop / 首次動作
- **標準 leader→follower**：移動 leader，follower 鏡像跟隨；leLab 提供即時關節串流。〔本地: leLab README/teleoperate.py〕
- 你有 2 支臂，可組成 **1 leader + 1 follower** 做最快的首次 teleop 驗證。
- **XLeRobot 變體（雙 follower）**：要用 LeKiwi 程式測單臂版時，需把「不與底盤共用同一塊控制板」的那支 SO101 拆下、夾在桌上接 PC 當 leader。〔web: xlerobot software/index, high〕

---

## 7. LeKiwi 底盤與雙臂整合（概覽；今日可跳過，但標出相依）

> 今天若只組手臂可跳過本章。**相依**：底盤輪馬達要佔用 ID 7/8/9，與手臂的 1–6 同一條 bus；整合前手臂 ID 必須已正確設好（第 3 章）。

- **馬達 ID**：手臂 1–6、底盤三輪 **7/8/9**（ID 8=後輪、7 與 9=左前/右前輪）。〔web: wiki.seeedstudio lerobot_lekiwi, high〕全部串在同一 bus、單一控制板連 Raspberry Pi。〔web: github SIGRobotics LeKiwi, high〕
- **XLeRobot 雙板配置**：兩塊控制板、兩支 follower（各原編 1–6）；一塊管 head（ID 7,8）、一塊管 wheel base（ID 7,8,9），建議標記 L1–L8 / R1–R9 區分。〔web: xlerobot assemble, high；本地: material.md〕
- **底盤輪馬達 ID 設定**：同樣**逐顆單獨接板**，不可先串聯。〔web: huggingface lekiwi, high〕
- **連接器高度**：把底盤+連接器放推車下試壓，確認**推車四輪仍能著地**；不行就在切片軟體**微調連接器 Z 軸比例（XY 不變）重印**。〔本地驗證: assemble.md §Wheel Base, verdict=confirmed〕
- **電源**：XLeRobot 用 Anker SOLIX C300（300W 輸出、288Wh），兩塊控制板各一條 USB-C→DC(12V)、Pi 一條 USB-C→USB-C；**Type-C→DC 線務必支援 12V 規格**，全機約 ~180W。〔本地: material.md / assemble.md〕注意：「300W 上限 + 12V 線規格」這條的**引用章節對應有瑕疵、判 uncertain**，數值方向正確但請以實際線材標示為準。〔對抗裁決: 300W/12V verdict=uncertain〕
- **host/client 架構**：Pi 跑 `python -m lerobot.robots.lekiwi.lekiwi_host --robot.id=...`，PC 端當 client 透過網路（remote_ip/port）連線 teleop。〔web: huggingface lekiwi, high〕
- **組裝完成後嚴禁推著走**：會損傷馬達齒輪；要移動請**抬起**（~12kg）。〔本地驗證: assemble.md §Final Assembly, verdict=confirmed〕

---

## 8. 常見坑與對策

| 症狀 | 對策 | 出處/裁決 |
|---|---|---|
| 全部馬達無回應 | 多半沒接電源（USB 只通訊，馬達需另外供電） | nvidia troubleshooting, high |
| 只缺單一 ID（下游受影響） | daisy-chain 接觸不良；重插電源、檢查排線 | nvidia troubleshooting, high |
| ID 衝突/無法通訊 | 設 ID 必須一次一顆，全設完才串聯 | confirmed（警告4） |
| Linux Permission denied | `sudo chmod 666 /dev/ttyACM0`；永久解：加入 `dialout` 群組後重登 | nvidia, high |
| 校準記到錯誤 min/max | 推到真正機械止點，避開卡住的線材/障礙 | nvidia, high |
| 校準後中位偏移 | 可做中位校準：關扭矩手動調到位、設目前位置為 2048 再驗證 | wiki.seeedstudio, medium |
| 重新校準報錯 | 先刪光舊校準檔（robots/ 或 teleoperators/） | waveshare, high |
| 校準時手臂掉落 | 校準會關扭矩，全程用手扶穩 | docs.phospho.ai, high |
| `lerobot-setup-motors` 設 baudrate 報 `termios.error (22, 'Invalid argument')` | 已知 GitHub issue #150，截至擷取時仍 open、無官方修復；換 port/設 666 皆未解 = **待官方修復** | github SO-ARM100 issue #150, high |
| 接電/插拔後傷板 | 電源最後接、插拔前先斷電源 | confirmed（警告2） |
| 推車輪不著地 | 連接器太高/低，調 Z 軸比例重印 | confirmed |

---

## 9. 仍待釐清（open questions / 對 SO-ARM101 不確定處）

1. **你的套件電壓版本（Standard 5V / Pro 5V+12V / XLeRobot 12V）= 待驗證** —— 通電前必須先確認，這是最大風險。
2. **SO-ARM101 各馬達預設反轉狀態（motor_reversals）未在可用文件明列** —— 需查 LeRobot 原始碼 `SO101LeaderConfig` / `SO101FollowerConfig`。〔本地 open_question 1〕
3. **「SO-ARM101」與工具鏈中的「SO-101」是否完全同一硬體（別稱）** —— leLab/部分文件只寫 SO-101，未明確提 SO-ARM101。〔本地/leLab open_question〕
4. **URDF 關節限制是否於校準自動套用，或需手動設定 = 待驗證。**〔本地 open_question 2〕
5. **校準「最小移動範圍」閾值（程式碼 <100 step 警告）對應的實際關節角度與成功影響 = 待驗證。**〔本地 open_question 3〕
6. **Bambot 在 Linux 是否需特殊驅動/權限**，以及逐顆設定的詳細步驟 = 待驗證。〔本地 open_question 4〕
7. **校準 JSON 的精確結構/單位（homing_offset、range_min/max）未完整描述 = 待驗證。**〔本地 open_question 5〕
8. **本地擷取提到的 `CalibrationDiscontinuityError` 例外與「編碼器換行=不可逆硬體損壞」說法已被推翻** —— 不要據此做決策；換行是可重新校準的軟體問題。〔對抗裁決: 警告1 refuted、encoder-wrap uncertain〕
9. **本地擷取的 leader 逐關節齒比對應已被推翻** —— 一律用第 4 章 C2 的官方對應。〔對抗裁決: 警告3 refuted〕
10. `lerobot-setup-motors` 的 `termios.error (22)` issue 是否已修復 = 待官方更新。〔github #150〕

---

## 10. 來源

### 本地檔案（repo 路徑）
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/SO-ARM100/README.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/SO-ARM100/SO100.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/SO-ARM100/3DPRINT.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/SO-ARM100/Simulation/SO101/README.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/XLeRobot/docs/en/source/hardware/getting_started/assemble.md`（含 §Wiring / §Wheel Base / §Final Assembly 多項已驗證警告）
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/XLeRobot/docs/en/source/hardware/getting_started/material.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/XLeRobot/docs/en/source/hardware/getting_started/3d.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/XLeRobot/docs/en/source/hardware/hardware_intro/index.md`
- `/home/roy422/RoyBot/RoyBot-Lab/src/AnvilBot/reference/leLab/README.md`、`CLAUDE.md`、`pyproject.toml`、`lelab/calibrate.py`、`lelab/teleoperate.py`、`lelab/record.py`、`lelab/server.py`、`lelab/scripts/lelab.py`

### Web（URL）
- https://huggingface.co/docs/lerobot/so101 （find-port / setup-motors / calibrate 指令、follower 1/345 與 leader 三種齒比、feetech 安裝、chmod 666；多項 confirmed）
- https://huggingface.co/docs/lerobot/lekiwi （逐顆設定、host/client；confirmed）
- https://wiki.seeedstudio.com/lerobot_so100m_new/ （Standard/Pro 電壓、follower/leader 末端差異、中位校準）
- https://wiki.seeedstudio.com/lerobot_so100m/ （leader 前關節齒比區分、插入順序埠號 medium、佈線改良）
- https://wiki.seeedstudio.com/lerobot_lekiwi/ （底盤輪 ID 7/8/9）
- https://www.waveshare.com/wiki/SO-ARM100/101_Set_Servo_ID
- https://www.waveshare.com/wiki/SO-ARM100/101_Robotic_Arm_Calibration_and_Remote_Control （刪舊校準檔）
- https://github.com/TheRobotStudio/SO-ARM100 （BOM/3D 列印權威、101 vs 100 差異、電壓/扭矩）
- https://github.com/TheRobotStudio/SO-ARM100/blob/main/SO100.md （接錯電壓損壞伺服）
- https://github.com/TheRobotStudio/SO-ARM100/issues/150 （setup-motors termios.error，open）
- https://docs.phospho.ai/so-101/quickstart （校準關扭矩、手扶穩、leader 齒比區分）
- https://docs.nvidia.com/learning/physical-ai/sim-to-real-so-101/latest/troubleshooting.html （電源未接、缺單一 ID、權限）
- https://openelab.io/blogs/getting-started/difference-between-seeed-studio-so-arm100-and-so-arm101
- https://github.com/SIGRobotics-UIUC/LeKiwi 、 https://github.com/SIGRobotics-UIUC/LeKiwi/blob/main/Assembly.md
- https://github.com/Vector-Wangel/XLeRobot 、 https://xlerobot.readthedocs.io/en/latest/hardware/getting_started/assemble.html 、 https://xlerobot.readthedocs.io/en/latest/software/index.html 、 https://xlerobot.readthedocs.io/en/latest/software/getting_started/raspberry_pi_setup.html
- https://huggingface-lerobot.mintlify.app/api/scripts/setup-motors （STS3215 baudrate 1000000，medium）
- https://us.amazon.com/SO-ARM101-Debugging-Reduction-345-Compatible/dp/B0FLW54PJB （leader 調試馬達 7.4V/1-345，medium）
- https://www.cnx-software.com/2025/05/02/so-arm101-open-source-dual-robotic-arm-kit-works-with-hugging-faces-lerobot/

> 安全聲明：本手冊未包含任何「已驗證成功」的實機結果；所有步驟在你的具體硬體上的成效屬 **待驗證**。通電前請務必先確認你的套件電壓版本（第 1、5 章）。
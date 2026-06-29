# Anvil x XLeRobot — Context（領域語言）

Anvil x XLeRobot（短稱 Anvil/X0）是 Roy 的低成本輪式雙臂 Physical AI 平台。本檔只放**本 child repo 專屬**的詞彙與命名；跨機器人工作區的權威 glossary 在父 repo `RoyBot-Lab/CONTEXT.md`、repo/路徑權威在 `RoyBot-Lab/docs/repo-map.md`，與本檔衝突時以父 repo 為準（註：父 repo 目前的命名/路徑較舊，待同步）。

## Language

**Anvil x XLeRobot**：
這條機器人線的正式專案品牌名 / README 標題。對外、文件標題一律用這個。
_Avoid_: 「XLeRobot fork」「MakerMods 專案」（那是上游，不是本專案身分）。

**Anvil/X0**：
平台代號 / 短稱，用於內文、MOC、口語。等同 Anvil x XLeRobot，**不是**另一個專案。

**Roy-XLeRobot-SO101**：
GitHub repository legal id（`roy4222/Roy-XLeRobot-SO101`，origin）。manifest / repo-map / git 操作一律用這個。品牌改名不代表 repo 改名——repo 身分刻意保留。
_Avoid_: 當成「另一個 repo」或「舊專案」。

**AnvilBot**：
本機工作目錄名（`src/AnvilBot`，2026-06-29 起）。對應 GitHub 的 `Roy-XLeRobot-SO101`。**舊路徑 `src/Roy-XLeRobot-SO101` 已不存在。**

**上游技術來源（是依賴/參考，不是本專案身分）**：
`XLeRobot`（Vector-Wangel 平台原型）、`SO-ARM101`（TheRobotStudio 手臂規格）、`LeRobot`（HuggingFace 控制框架）、`LeKiwi`（行動底盤）、`leLab`（LeRobot web 流程）、`MakerMods-XLeRobot`（upstream 硬體設計，repo 門面現況）。

### 狀態 / 里程碑語言（分層，2026-06-29 LOCKED）

**Stage 0–3**：
**權威**能力成熟度 gate（父 repo `robots/xlerobot-so101.md` LOCKED）。Stage 0＝follower bring-up + 首次動作；Stage 0.5＝leader station；Stage 1＝teleop 收資料 + ACT；Stage 2/3＝接觸任務 / 評估。Stage pass 一律要 evidence。
_Avoid_: 用 Phase 1–8 當里程碑。

**Stage 0 pass**：
單臂 teleop 連續 5 分鐘無失控 + 影片/log evidence。**W2 只是「讓 teleop 動起來」，不等於 Stage 0 pass**（W2 ⊂ Stage 0）。

**Week（W1–W4）**：
行事曆排程，疊在 Stage 上（「這幾週推進到哪個 Stage」），**不是**另一套能力里程碑。

**touch marker**：
**Stage 1 的 entry task**——刻意保守的 pipeline 驗證任務（home → 碰固定 marker → 回 safe pose），驗的是「資料→訓練→推論→eval」管線本身。
_Avoid_: 跟「推動式物品分類」混為一談。

**推動式物品分類**：
Stage 1 後段 / Stage 2 的 eval 任務（push-to-sort），**不是第一個 gate**。

**Phase 1–8 / Button Press**：
**退役（historical）** 的舊狀態模型，不再當里程碑；保留為歷史。一律改用 Stage 0–3。

## Flagged ambiguities

- **「Anvil / X0 / Roy-XLeRobot-SO101 / AnvilBot 是四個不同專案嗎？」** 否——同一條機器人線的四個層級：品牌名 / 短稱 / repo id / 本機目錄。GitHub UI 看不出這層對應，是新人最大誤判點（決策見 `docs/07-decisions.md` 2026-06-29 命名 ADR）。
- **「狀態語言用 W1–W4 還是 Stage 0–3？」** ✅ 已解（2026-06-29 ADR）：**Stage 0–3 = 權威能力 gate**，**W1–W4 = 排程**疊在 Stage 上，**Phase 1–8 / Button Press 退役**。W2 ≠ Stage 0 pass。

## 範例對話

> 工程師：「Anvil 的 repo 在哪？跟 XLeRobot 什麼關係？」
> Roy：「GitHub 是 `roy4222/Roy-XLeRobot-SO101`，本機在 `src/AnvilBot`，對外叫 **Anvil x XLeRobot**（短稱 Anvil/X0）——都是同一台。XLeRobot / SO-ARM101 / LeRobot / LeKiwi 是它建構所依賴的上游，不是專案本身。」

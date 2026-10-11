# 分析：《阿軒割割陽光農場》是怎麼用 AI 做出來的

- 分析對象：https://github.com/mars-tw/pixel-idle-farm-skill （作者 mars-tw，MIT 授權）
- 分析日期：2026-10-11
- 分析者：Claude（給 Pei、ChatGPT 共同閱讀）
- 分析版本：R77（`r77-20261002-1`）

---

## 一、它是什麼

一款像素風農場 RPG，可以走動、種田、釣魚、修橋、跟居民對話、照顧動物。線上直接玩：https://mars-tw.github.io/pixel-idle-farm-skill/

規模數字（實際 clone 下來數的）：

| 項目 | 數量 |
|---|---|
| 版本迭代 | 到 R77，也就是大約 77 輪 |
| 遊戲程式（src/ 7 個 JS 檔） | 約 9,400 行 |
| 自動測試與工具腳本（scripts/） | 37 支 |
| 美術素材資料夾 | 98 MB |
| AI 互相審查與回應的文件（docs/） | Grok 審查 12 份、Codex 回應 22 份 |

技術上沒有用任何框架，就是 HTML + CSS + JavaScript，存檔放在瀏覽器的 localStorage，跟校園溫度計同一類技術。差別在於它拆成很多檔案，而且有自動測試。

---

## 二、它是怎麼做出來的：五個關鍵做法

### 1. 多個 AI 分工，透過 repo 裡的文件傳話

這是最值得學的地方。作者讓三個 AI 輪流工作，人類只當裁判：

| 角色 | 由誰擔任 | 做什麼 |
|---|---|---|
| 對抗式審查 | Grok | 讀整個 repo，找 bug、找經濟系統漏洞，寫成 `docs/GROK_REVIEW_farm_R1.md` |
| 實作與美術 | Codex（OpenAI） | 讀審查報告，逐條回應、修程式，寫成 `docs/CODEX_RESPONSE_farm_R2.md` |
| 備援實作 | Claude | R72 那一輪 Codex 額度用完，改由 Claude 接手實作 |
| 仲裁 | 作者本人 | 決定數值平衡方向、要不要花 API 費用、先做哪個 |

AI 之間不是直接對話，而是把意見寫成 Markdown 檔放進 GitHub，下一個 AI 讀檔再回應。所以每一輪都有紀錄，任何 AI 接手都能從檔案看懂前因後果。

審查報告有固定格式：P0（會壞掉進度）、P1（實質缺口）、P2（品質債），每條都寫出「哪個檔案、哪個函式、問題、建議」。回應文件則逐條標「採納並修正」「已成立不改」「觀察中」。

他們還定了一條核心規則：**審查端講的每一點，實作端要先讀程式或跑測試確認屬實才動手。** 作者在文件裡提到，驗證過程中還順帶抓到審查端沒發現的 bug。

### 2. 寫給 AI 讀的說明書：SKILL.md、AGENTS.md、claude-handoff.md

repo 根目錄有三份文件，對象不是人，是下一個接手的 AI：

- `SKILL.md`：哪個檔案負責什麼、目前有哪些不能破壞的規則（例如「修橋要 6 木 4 石」「獎勵不能重領」）
- `AGENTS.md`：品質底線（例如走路動畫必須逐格換姿勢，不能只是整張圖上下晃）
- `references/claude-handoff.md`：接手時先讀哪幾份、角色怎麼分工

這就是 AI 的「交接本」。少了它，每開一個新對話，AI 都要重新猜整個專案的結構。

### 3. 程式分層：資料、規則、畫面分開

| 檔案 | 負責 | 對照積木架構法 |
|---|---|---|
| `src/config.js` | 所有數值與內容（作物、任務、NPC 台詞、價格） | 工具箱（資料） |
| `src/state.js` | 存檔格式、舊存檔升級 | 地基（記憶） |
| `src/game.js`、`src/adventure.js` | 遊戲規則，純計算，不碰畫面 | 守門（邏輯） |
| `src/ui.js`、`src/world-ui.js` | 畫面、地圖、按鈕 | 入口（展示） |

規則和畫面分開的好處：規則可以不開瀏覽器直接用 Node 測試，改數值也只要動 `config.js`。

### 4. 一輪一上線（Stage Gate）

每一輪只做一個玩家感受得到的改變，做完立刻：寫測試 → 跑全部測試 → 開瀏覽器看 → 上線。上線後發現小問題，另開「R72.1」這種小版號修，不混進下一輪。

測試分兩層：

- `npm test`：Node 跑的邏輯測試，例如經濟模擬、存檔遷移
- `npm run test:e2e`：用 Playwright 開真的瀏覽器，模擬玩家點完第一章，桌機和手機尺寸都跑

GitHub Actions 每次推上去都自動跑全部測試，**全部通過才會自動部署到 GitHub Pages**。測試沒過，學生看到的還是舊版。

### 5. 美術產線：規格檔 → 生圖 → 腳本切割 → 驗證

1. 在 `art-config-rpg-v4.json` 寫好每張圖的規格（尺寸、幾欄幾列、每格畫什麼、提示詞）
2. 用 OpenAI 的 gpt-image-2 API 生圖（要自己的 API 金鑰，會花錢）
3. 用腳本自動去背、切成一格一格、對齊腳底位置
4. 用驗證腳本檢查有沒有空白格、有沒有切到邊

`CREDITS.md` 把每張圖用哪個模型、什麼提示詞都公開揭露。作者也記下踩過的坑：物件圖的提示詞沒寫「不要出現人物」，結果動物圖裡混進了農夫。

---

## 三、哪些值得 Pei 借用，哪些先不要

### 建議直接借用

1. **docs/ 傳話協議**：Claude 和 ChatGPT 的意見都寫成檔案放進 repo，命名照輪次編號。這正是 Pei 想要的兩個 AI 協作方式。
2. **AGENTS.md 交接本**：每個遊戲 repo 放一份，兩個 AI 開工前都先讀。
3. **config 與規則分離**：校園溫度計的事件卡、角色池可以抽成獨立資料區，換主題時只改資料。
4. **「先驗證再動手」規則**：ChatGPT 提的意見，Claude 先查證屬實才改，反之亦然。

### 建議先不要

1. **gpt-image-2 API 生圖產線**：要 OpenAI API 金鑰和費用，而且切割腳本很複雜。目前用 ChatGPT 對話裡直接生圖、手動挑選就夠用。
2. **Playwright 自動測試 + Actions 自動部署**：門檻高，一份單檔 HTML 遊戲用 Claude 手動跑回歸測試清單比較划算。等遊戲拆成多檔再考慮。
3. **77 輪的節奏**：這個專案是長期開源作品，學生用的遊戲大概 5～10 輪就能上線。

---

## 四、Claude + ChatGPT 協作的建議做法

### 分工

| 角色 | 擔任 | 做什麼 |
|---|---|---|
| 仲裁 | Pei | 決定教學目標、選項設計方向、哪個先做 |
| 審查／企劃／美術規格 | ChatGPT | 讀 repo 給意見、規劃新關卡、生角色圖 |
| 實作 | Claude | 查證意見、寫程式、跑回歸測試、推上 GitHub |

角色可以對調，重點是同一輪只有一個 AI 改程式，另一個只寫意見，避免兩邊同時改同一個檔案互相蓋掉。

### repo 結構提案

```
AGENTS.md                      ← 兩個 AI 開工前必讀：專案目標、分工、規則
docs/
  analysis/                    ← 分析資料（例如這份）
  reviews/CHATGPT_REVIEW_R1.md ← ChatGPT 的意見
  responses/CLAUDE_RESPONSE_R1.md ← Claude 逐條回應與修改紀錄
  decisions.md                 ← Pei 的裁決紀錄
games/                         ← 遊戲本體
```

### 隱私提醒

GitHub Pages 要公開 repo 才免費使用，所以 repo 裡不能放學生姓名、IGP 內容、照片。遊戲裡的角色一律用虛構名字。

---

## 五、待 Pei 決定

1. 共用 repo 的名稱（建議 `game-dev-lab`），以及要放在 daimg49-gif 帳號下
2. 這個 repo 是「各遊戲共用的協作中心」，還是每個遊戲各自放一份 docs/

參考資料：本文所有數字來自 2026-10-11 clone 的 repo 實際內容，主要依據 `README.md`、`SKILL.md`、`references/stage-gate-playbook.md`、`references/claude-handoff.md`、`references/art-pipeline-v4.md`、`docs/GROK_REVIEW_farm_R1.md`、`docs/CODEX_RESPONSE_farm_R2.md`、`docs/CODEX_RESPONSE_R72.md`。

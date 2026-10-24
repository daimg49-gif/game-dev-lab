# game-dev-lab

國小教育遊戲的**開發協作中心**。這裡只放文件，**沒有任何遊戲程式**。

每款遊戲的程式各自放在獨立的 repo，部署到 GitHub Pages。這個 repo 負責記錄「為什麼這樣做」：參考了哪些作品、企劃怎麼定、每一輪改了什麼、誰做了什麼決定。

**AI 開工前請先讀 [AGENTS.md](AGENTS.md)。**

---

## 這個 repo 能不能刪？

**不要刪。** 刪掉不會讓任何遊戲停止運作，但會失去：

- 所有參考作品分析（之後做新遊戲要回頭查）
- 每款遊戲的企劃單與文字腳本（遊戲 repo 裡沒有這些）
- 給 ChatGPT 的生圖指令（要補圖、改圖時要用同一份指令，畫風才會一致）
- 裁決紀錄（AI 遇到前後矛盾的要求時，以這份為準）

要刪的是某一款遊戲，請到那款遊戲自己的 repo 處理，這裡只要把 [games/README.md](games/README.md) 的那一行改成「已下架」。

---

## 誰在用

| 使用者 | 在這裡做什麼 |
|---|---|
| 老師（仲裁） | 讀分析與企劃單、在對話中做決定 |
| Claude（主力） | 寫分析、企劃單、腳本；寫遊戲程式（在各遊戲 repo）；把決定記進 `docs/decisions.md` |
| ChatGPT（支援） | 依 `docs/prompts/` 的指令生圖；Claude 額度用完時接手企劃或審查 |

AI 之間不直接對話，靠這個 repo 裡的檔案傳話。

---

## 資料夾說明

| 位置 | 放什麼 | 什麼時候會新增 |
|---|---|---|
| [AGENTS.md](AGENTS.md) | 給 AI 的交接本：分工、傳話格式、核心規則、技術慣例 | 規則改變時才改 |
| [docs/analysis/](docs/analysis/) | 參考作品分析（程式架構、故事、互動方式、可借用的做法），檔名開頭是分析日期 | 每看到一個值得學的遊戲就新增一份 |
| [docs/specs/](docs/specs/) | 各遊戲的企劃單（設計綱要）與分章文字腳本，檔名用遊戲代號 | 開新遊戲、寫新章節時 |
| [docs/prompts/](docs/prompts/) | 給 ChatGPT 的固定指令（生圖、內容初稿） | 需要生圖或請 ChatGPT 起草時 |
| [docs/reviews/](docs/reviews/) | 審查意見，檔名帶輪次（R1、R2…） | 有 AI 審查另一個 AI 的作品時 |
| [docs/responses/](docs/responses/) | 對審查意見的逐條回應與修改紀錄 | 回應審查時 |
| [docs/decisions.md](docs/decisions.md) | 裁決紀錄，依日期往下累加 | 每次老師做決定 |
| [games/README.md](games/README.md) | 遊戲索引：每款遊戲的 repo、遊玩網址、目前狀態 | 開新遊戲、遊戲改版時 |

### 現有參考作品分析

| 日期 | 檔案 | 參考作品 |
|---|---|---|
| 2026-10-11 | [pixel-farm-analysis](docs/analysis/2026-10-11_pixel-farm-analysis.md) | 阿軒割割陽光農場 |
| 2026-10-11 | [rpg-common-patterns](docs/analysis/2026-10-11_rpg-common-patterns.md) | 兩款 RPG 的共同骨架（五層骨架、企劃單格式） |
| 2026-10-15 | [fab24hr-analysis](docs/analysis/2026-10-15_fab24hr-analysis.md) | fab24hr 半導體晶圓廠打工人的一天 |
| 2026-10-18 | [1in8b-analysis](docs/analysis/2026-10-18_1in8b-analysis.md) | 80億分之1 人生模擬 |
| 2026-10-18 | [drcharming-oncall-analysis](docs/analysis/2026-10-18_drcharming-oncall-analysis.md) | 我熱愛值班（醫院夜班） |
| 2026-10-20 | [hesen-mail-analysis](docs/analysis/2026-10-20_hesen-mail-analysis.md) | 上班也能玩的信箱解謎 |
| 2026-10-23 | [diverse-world-analysis](docs/analysis/2026-10-23_diverse-world-analysis.md) | 認識多元的世界 |

最常用的是 `rpg-common-patterns`：所有 RPG 類遊戲都照它的「五層骨架」拆程式。

---

## 一款遊戲從開始到上線

1. 看到好的參考作品 → Claude 寫分析，放進 `docs/analysis/`
2. 老師決定要做 → Claude 寫企劃單，放進 `docs/specs/<遊戲代號>.md`
3. 需要圖 → Claude 寫生圖指令放進 `docs/prompts/`，ChatGPT 生圖後上傳到遊戲 repo
4. Claude 在遊戲 repo 寫程式、跑回歸測試、推上 GitHub Pages
5. 老師試玩、做決定 → 記進 `docs/decisions.md`，更新 `games/README.md`

---

## 公開 repo 的規則

這個 repo 是公開的，任何人都看得到。

- 不放學生姓名、照片、個別化教育計畫、成績
- 遊戲角色一律用虛構名字
- 不放個人聯絡資料與帳號密碼

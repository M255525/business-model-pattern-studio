# 商業模式組合工作室（business-model-pattern-studio）

輸入產品／服務 → 從 63 個書中模式＋15 個延伸模式算適配度 → 自動組合（策略／營運／獲利／情境各 N 個）→ 九種評價（商業模式圖、六構面雷達、綜效與衝突、SWOT、五力、財務健檢、驗證實驗、個案對照、模式適配排名）→ PDF 報告。純前端靜態網站，本機 port **8835**（`python -m http.server 8835`，已登記在工作區 `.claude/launch.json`）。

## 檔案
- `index.html`：全部邏輯（規則式引擎、AI、設定、歷史、PDF、跑馬燈、PWA）。
- `patterns.js`：`window.BMP` = 模式資料庫 `PATTERNS`、產業設定 `INDUSTRIES`（關鍵字、屬性、BMC 範本、機會威脅、五力基準、財務預設值）、綜效表 `SYNERGY`、範例 `SAMPLES`。
- `manual.html`、`manifest.json`、`service-worker.js`（network-first，同 business-model-canvas-generator）、`icons/`（Python PIL 產生的四色方塊圖示）、`watermark.png`（沿用 new-product-strategy-studio 的新版馬克老師圖示）。

## 智財權（重要）
模式**名稱與編號／四大類**參考《商業模式大全》（根來龍之等著，商業周刊 2021），但每個模式的定義、應用說明、案例、風險、驗證實驗**全部自行撰寫，不得改成書中原文**，也不放書封或書中圖表。**2026-10-26 使用者要求：網站、說明頁、PDF、manifest 都不顯示書名／作者出處，也不寫「63 個」這個數字**（介面只稱「經典商業模式＋延伸模式」；模式卡上的 #1–#63 編號保留）。不要再自行加回出處聲明。

## 規則式引擎（確定性）
- `attrsOf()`：產業屬性 + 表單選項（B2B/B2C、型態、價格帶）+ 關鍵字（`KW_ATTR`）→ 14 維屬性向量。
- `rankPatterns()`：`dot(tags, attrs) / |tags|`（**一定要正規化**，否則標籤多的模式天生高分，早期版本寵物照護會選到刮鬍刀＋多層次傳銷），主要標籤（權重最高者，同分取最符合者）屬性 < .4 扣 .6；產業吻合 +.7、通用模式 +.25；`CAUTION` 表壓低爭議模式（多層次傳銷 S21 等）；同分用 `hashString` 決定，不用亂數。
- `autoSelect()`：每類取前 N 名，跳過與已選模式衝突值 ≤ -2 的組合。
- 六構面分數由所選模式 `ev` 平均 + 綜效 + 五力 + 財務 LTV/CAC 推得；**總分由頁面 `overallOf()` 加權計算，不採用 AI 自報總分**。

## AI（BYOK）
`callLLM`／`extractJsonObject`／模型清單沿用 `social-post-grader`。一次呼叫回傳 selected＋summary＋bmc＋dims＋swot＋forces＋experiments；`applyAi()` 逐欄位驗證（代碼須存在、分數 1–5、等級 1–3），不合格的退回規則式，卡片標「規則式」。AI 選的代碼少於 3 個 → 組合改用規則式。各分頁有「✨ AI 重新產生此區塊」（`regenSection`）。使用者在排名分頁手動改組合 → 整份改回規則式並顯示 `#staleNote`。

## ⚙ 常用設定
localStorage 全部用 `bmPatternStudio.` 前綴（settings／history／draft／marquee），「清除本機資料」刪除此前綴的所有 key。設定：AI 服務商／模型／金鑰／測試連線、字級、每類挑選數、延伸模式、要產出的評價、自動存歷史、跑馬燈、PDF 署名／浮水印／附錄。「我的常用預設值」`settings.formDefaults`（產業、客戶類型、型態、價格帶＋6 個財務欄位）：無草稿的新表單與「🧹 清空表單」會套用；財務欄位 placeholder 由 `updateFinHints()` 依（自動判斷的）產業顯示常用值，「📥 帶入產業常用預設值」實際填入。

## PDF
`buildPrintReport()` 產生 `#printReport`，`@media print` 只顯示它；浮水印是 `<img>`，不是 CSS 背景圖。**A4 寬度會觸發 ≤820px 手機斷點**，所以 print CSS 用 `#printReport .c-xx{...!important}` 強制回到九宮格版面。驗證時用 Playwright `emulateMedia('print')`＋`page.pdf()`（先呼叫 `window.__bmp.buildPrintReport()`，因為 `page.pdf` 不會觸發 beforeprint），**不要真的點列印鈕**。

## 測試掛鉤
`window.__bmp`：`computeRule(input, selected?)`、`getState()`、`applyAi()`、`buildPrintReport()`、`overallOf()`。

## 部署
GitHub Pages（`.github/workflows/deploy-pages.yml`，分支 master）。**使用者確認本機版沒問題後才建 repo 並推送。**

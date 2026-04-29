# Human Touch 最佳案例研討系統

> SKODA 內部訓練 · 11 個真實案例 + 深度題目 · 雲端成績管理

## 系統概覽

學員透過手機掃 QR Code 進入系統 → 選擇據點、填寫姓名 → 系統從 11 個真實案例中隨機抽 10 個 → 每個案例配一題深度題目 → 完成後分數、用時、據點等資料即時上傳 Firebase 雲端 → 管理員透過後台跨裝置查閱、列印、匯出 CSV。

- **案例**:11 個(2023–2025 年銷售與服務獲獎案例 + 1 個總結思辨案例)
- **每次測驗**:抽選 10 個案例,每案例 1 題,共 10 題
- **題目深度**:情境判斷、思辨、應用層級(非單純記憶)
- **及格分數**:80 分
- **據點清單**:36 個經銷據點(依 outlet_list 完整匯入)
- **成績儲存**:Firebase Firestore(主)+ localStorage(離線備援)
- **管理員密碼**:`skoda2025`(可在 `index.html` 中搜尋此字串自行修改)

---

## 快速部署(GitHub Pages)

### 1. 推上 GitHub
將本資料夾的 `index.html` 推到 repo:`https://github.com/marschentaiwan/CaseStudy.git`

```bash
git clone https://github.com/marschentaiwan/CaseStudy.git
cd CaseStudy
# 把 index.html 複製進來
git add index.html
git commit -m "Add Human Touch case study system"
git push origin main
```

### 2. 啟用 GitHub Pages
進入 repo → Settings → Pages → Source 選 `main` branch → Save。
約 1 分鐘後可在以下網址訪問:
```
https://marschentaiwan.github.io/CaseStudy/
```

### 3. 設定 Firebase Firestore 安全規則(**重要**)
此步驟若略過,系統會因規則不允許寫入而存不了成績。

1. 進入 [Firebase Console](https://console.firebase.google.com/) → 選擇專案 `casestudy-2c515`
2. 左側選單 → Build → Firestore Database
3. 若尚未建立資料庫,點「建立資料庫」 → 選「**生產模式**」 → 選擇地區(建議 `asia-east1` 台灣)
4. 切到上方「**規則**(Rules)」分頁
5. 將本資料夾 `firestore.rules` 的內容**整段複製貼上** → 點「**發布**」

### 4. 設定授權網域
1. Firebase Console → Authentication → Settings → Authorized domains
2. 新增:`marschentaiwan.github.io`(GitHub Pages 域名)
3. 若有自訂網域也一併新增

### 5. 產生 QR Code
取得部署網址後,使用 [qr-code-generator.com](https://www.qr-code-generator.com/) 或任何 QR 工具產生條碼,印在訓練海報或培訓教材上。

---

## 使用流程

### 學員端
1. 掃 QR Code 進入系統
2. 從下拉選單選擇據點(36 個選項)
3. 輸入姓名與職務
4. 開始作答(10 題,每題搭配真實案例)
5. 完成後立即看到分數、答題檢視、解析
6. 可選擇列印成績單

### 管理端
1. 在系統右下角點擊「ADMIN」連結
2. 輸入密碼 `skoda2025`
3. 進入後台查看:
   - 總參加人數、通過人數、平均成績、參與據點數
   - 完整記錄表格(支援姓名/據點搜尋、據點篩選、狀態篩選)
   - **重新整理** — 拉取雲端最新資料
   - **匯出 CSV** — 含 BOM,Excel 可直接打開不亂碼
   - **列印報告** — 列印整個後台頁面

---

## 自訂調整

### 修改管理員密碼
在 `index.html` 中搜尋 `skoda2025`,替換成自己的密碼。

### 調整及格分數
搜尋 `PASS_SCORE = 80`,改為其他數字。

### 增減據點
搜尋 `const OUTLETS = [`,在陣列中增減據點即可。

### 新增案例
搜尋 `const CASES = [`,依現有格式新增物件:
```javascript
{
  id: 'unique_id',
  year: 2026, type: '銷售',
  branch: '某據點', hero: '某員工',
  title: '案例標題',
  body: `內文段落 1\n\n段落 2`
}
```
然後在 `QUESTIONS_BY_CASE` 物件中,以該 `id` 為 key 新增至少 1 題:
```javascript
'unique_id': [
  { text: '題目?', options: ['A','B','C','D'], answer: 1, explain: '解析' }
]
```

### 修改每次抽題數
搜尋 `shuffled.slice(0, 10)`,把 `10` 改成想要的題數(不要超過案例總數)。

---

## 資料結構(Firestore)

每筆成績記錄存於 `records` collection,欄位如下:

| 欄位 | 型別 | 說明 |
|---|---|---|
| `name` | string | 學員姓名 |
| `dealer` | string | 經銷據點 |
| `role` | string | 職務(可空) |
| `score` | number | 0–100 分 |
| `correct` | number | 答對題數 |
| `total` | number | 總題數 |
| `pass` | boolean | 是否及格 |
| `elapsed` | number | 作答秒數 |
| `answers` | array | 各題作答選項 index |
| `detailedAnswers` | array | 完整題幹 + 選項 + 對錯記錄 |
| `completedAt` | string | ISO 完成時間 |
| `createdAt` | timestamp | Firestore server timestamp |

---

## 安全考量

- **無需登入即可寫入成績** — 是為了學員端體驗順暢。Firestore 規則已限制資料格式與長度,避免被當作免費資料庫。
- **管理員密碼僅在前端驗證** — 若需要更嚴格的權限管控,建議改用 Firebase Authentication + Custom Claims,並把規則改為只有特定 UID 可讀取 records。
- **成績不可竄改** — 規則設定 `update: false, delete: false`,寫入後無法修改,確保訓練成績可信。

---

## 疑難排解

### 學員端右上角顯示「OFFLINE MODE」
代表 Firebase 初始化失敗。常見原因:
1. Firebase 專案的 API Key 已停用 → 檢查 Firebase Console
2. GitHub Pages 域名未加入授權網域 → 見上方步驟 4
3. Firestore 規則未發布 → 見上方步驟 3

### 後台沒有資料
1. 確認規則已發布為本資料夾的 `firestore.rules` 內容
2. 在後台點「重新整理」按鈕
3. 開瀏覽器開發者工具(F12)看 Console 是否有錯誤訊息

### 學員作答完顯示「雲端儲存失敗」
資料已暫存本機 localStorage,但雲端寫入失敗。請依 Console 錯誤訊息排查 Firestore 規則。

---

## 版本記錄

- **v2.0** — 新增 Firebase 雲端儲存、案例擴增至 11 個、每次抽 10 題、深度思辨題目、36 個完整據點清單
- **v1.0** — 初版,6 案例 12 題,localStorage 本機儲存

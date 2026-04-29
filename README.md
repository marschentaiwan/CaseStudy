# Human Touch 最佳案例研討系統

> SKODA 內部訓練 · 11 個真實案例 · 深度題目 · 雲端成績管理

## 系統概覽

學員透過手機掃 QR Code 進入系統 → 選擇據點(依**北/中/南**地理分組)、填寫姓名 → 系統從 11 個真實案例中**隨機抽 10 個**(每案 1 題)→ 完成後分數、用時、據點等資料即時上傳 Firebase 雲端 → 管理員透過後台跨裝置查閱、列印、匯出 CSV。

- **案例**:11 個(2023–2025 年銷售與服務獲獎案例 + 1 個總結思辨案例)
- **每次測驗**:從 11 案隨機抽 10 案,每案 1 題,共 10 題
- **題目深度**:情境判斷、思辨、應用層級(非單純記憶)
- **及格分數**:80 分
- **據點清單**:36 個經銷據點,依**北部 17 / 中部 12 / 南部 7** 分組
- **成績儲存**:Firebase Firestore(主)+ localStorage(離線備援)
- **管理員密碼**:`skoda2025`(可在 `index.html` 中搜尋此字串自行修改)

---

## 快速部署(GitHub Pages)

### 1. 推上 GitHub
```bash
git clone https://github.com/marschentaiwan/CaseStudy.git
cd CaseStudy
# 把本資料夾的 index.html 複製進來
git add index.html
git commit -m "Update Human Touch case study system"
git push origin main
```

### 2. 啟用 GitHub Pages
進入 repo → Settings → Pages → Source 選 `main` branch → Save。
約 1 分鐘後可在 `https://marschentaiwan.github.io/CaseStudy/` 訪問。

### 3. 設定 Firebase Firestore(**最重要,沒做的話雲端會失敗**)

#### 3-1. 建立 Firestore 資料庫
1. 進入 [Firebase Console](https://console.firebase.google.com/) → 選擇專案 `casestudy-2c515`
2. 左側選單 → **Build** → **Firestore Database**
3. 若尚未建立,點「**建立資料庫**」 → 選「**生產模式**」 → 地區建議 `asia-east1`(台灣)

#### 3-2. 發布安全規則
1. 切到上方「**規則**(Rules)」分頁
2. 將本資料夾 `firestore.rules` 的內容**整段複製貼上** → 點「**發布**」
3. 確認規則狀態顯示為「**已發布**」

#### 3-3. 加入授權網域
1. Firebase Console → **Authentication** → **Settings** → **Authorized domains**
2. 點「**新增網域**」 → 輸入 `marschentaiwan.github.io`
3. 若有自訂網域也一併加入

### 4. 產生 QR Code
取得部署網址後,使用 [qr-code-generator.com](https://www.qr-code-generator.com/) 或任何 QR 工具產生條碼,印在訓練海報或培訓教材上。

---

## 雲端儲存失敗排查 ⚠️

完成測驗後若顯示「雲端儲存失敗」,系統會直接顯示**具體錯誤代號**,對照下表處理:

| 顯示訊息 | 真正原因 | 解決方法 |
|---|---|---|
| **【權限錯誤】** Firestore 規則尚未發布 | Firestore Rules 還是預設的「生產模式拒絕全部」 | 依上方步驟 3-2,把 `firestore.rules` 內容貼到規則並發布 |
| **【連線錯誤】** 無法連到 Firebase | 網路問題或 GitHub Pages 域名未授權 | 1. 檢查網路 2. 依步驟 3-3 加入授權網域 |
| **【設定錯誤】** Firebase 專案可能未啟用 Firestore | Firestore 資料庫尚未建立 | 依步驟 3-1 建立 Firestore Database |
| **【初始化失敗】** Firebase 未能正確初始化 | API Key 失效或被防火牆阻擋 | 1. 檢查 Firebase Console 看 API Key 是否仍啟用 2. 換個網路試試 |
| **【未連線】** Firebase 未載入 | 載入 Firebase SDK 的 CDN 被擋 | 1. 確認可訪問 `gstatic.com` 2. 從 file:// 直接開檔會出現此訊息,屬正常,請部署到網頁伺服器 |

### 學員端右上角顯示「OFFLINE MODE」?
代表 Firebase 初始化就失敗了。常見原因依優先順序:
1. 從 file:// 直接開啟 HTML(必須透過 https:// 部署才會連得上)
2. GitHub Pages 域名未加入 Firebase 授權網域
3. Firebase 專案的 API Key 已停用

### 不確定問題在哪?
按 **F12** 打開瀏覽器開發者工具 → 切到「Console」分頁 → 重新提交一次,Firebase 會在 Console 顯示完整錯誤碼與堆疊。常見的明確錯誤訊息:

- `FirebaseError: Missing or insufficient permissions` → 規則沒發布
- `400 Bad Request` → Firestore 資料庫不存在
- `Failed to fetch` / `net::ERR_FAILED` → 網路或網域授權問題

---

## 使用流程

### 學員端
1. 掃 QR Code 進入系統
2. 從下拉選單選擇據點(依**北/中/南**分組,3 區共 36 個)
3. 輸入姓名與職務(可選)
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
在 `index.html` 搜尋 `skoda2025`,替換為自己的密碼。

### 調整及格分數
搜尋 `PASS_SCORE = 80`,改為其他數字。

### 調整每次抽案數
搜尋 `shuffled.slice(0, 10)`,把 `10` 改成想要的題數(不要超過案例總數 11)。

### 增減據點 / 調整分組
搜尋 `const OUTLET_GROUPS = [`,在對應地區的 `items` 陣列增減。新增地區直接加新物件即可,選單會自動更新。

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
然後在 `QUESTIONS_PER_CASE` 物件中,以該 `id` 為 key 新增 1 題:
```javascript
'unique_id': {
  text: '深度題目?',
  options: ['A','B','C','D'],
  answer: 1,
  explain: '解析說明'
}
```

### 修改題目
搜尋 `const QUESTIONS_PER_CASE = {`,直接編輯對應 case id 的題目內容。每案僅 1 題。

---

## 資料結構(Firestore)

每筆成績記錄存於 `records` collection:

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

- **無需登入即可寫入成績** — 為了學員端體驗順暢。Firestore 規則已限制資料格式與長度,避免被當免費資料庫。
- **管理員密碼僅前端驗證** — 若需更嚴格權限管控,建議改用 Firebase Authentication + Custom Claims。
- **成績不可竄改** — 規則設定 `update: false, delete: false`,寫入後無法修改,確保訓練成績可信。

---

## 版本記錄

- **v2.1** — 加入 Logo 視覺、字體加大、據點北中南分組、雲端錯誤訊息明確化、每案固定 1 題
- **v2.0** — 新增 Firebase 雲端儲存、案例擴增至 11 個、每次抽 10 題、深度思辨題目、36 個完整據點清單
- **v1.0** — 初版,6 案例 12 題,localStorage 本機儲存

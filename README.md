# 通訊錄 QR Code 產生器

將個人聯絡資訊（姓名、手機、Email、公司）轉換成符合 vCard 3.0 格式的 QR Code，掃描後可直接存入手機通訊錄。

**線上使用：**
- Cloudflare Pages：https://vcard-qr-generator.pages.dev
- GitHub Pages：https://go38.github.io/vcard-qr-generator/

---

## 功能

- 輸入姓氏、名字、手機號碼、電子郵件（選填）、公司名稱（選填）
- 即時產生 vCard 3.0 格式的 QR Code
- 支援中文姓名（UTF-8 編碼）
- 下載高解析度 PNG 圖片（1024×1024px），適合印刷及名片設計
- 完全離線運作，不需網路連線

---

## 檔案結構

```
QR-Code/
└── index.html    # 完整應用程式（HTML + CSS + JS 單一檔案）
```

整個專案只有一個檔案。qrcode.js 函式庫已直接內嵌其中，無任何外部相依。

---

## 設計決策

### 為何選擇單一 HTML 檔案，而非 Python 程式？

最初評估了兩個方案：

| 方案 | 優點 | 缺點 |
|------|------|------|
| Python + tkinter | 標準程式結構 | 需安裝 Python、pip 套件；tkinter 在 Windows 有 DPI 問題 |
| 單一 HTML 檔案 | 雙擊即開，無需安裝；離線可用；跨平台 | — |

選擇 HTML 方案的核心理由是**零安裝門檻**。目標使用者不一定具備技術背景，雙擊 HTML 檔案是最直覺的操作方式。

### 為何將 qrcode.js 內嵌，而非使用 CDN？

初版使用 CDN 載入 qrcode.js（`cdn.jsdelivr.net/npm/qrcode@1.5.3`），但在實際測試中發現：

1. 部分瀏覽器在開啟本機 `file://` 頁面時會封鎖外部請求
2. CDN 版本路徑失效（404），導致按鈕完全無反應且無任何錯誤提示

因此改為將 qrcode.js（~30KB minified）直接內嵌進 `<script>` 標籤，確保在任何環境下都能正常運作。

### 姓名欄位為何拆分成姓氏與名字？

vCard 3.0 的 `N` 欄位規格為：

```
N:姓氏;名字;中間名;稱謂前綴;稱謂後綴
```

若將完整姓名塞進第一個欄位（`N:王大明;;;;`），雖然多數手機能正常顯示（因為優先讀取 `FN` 欄位），但部分通訊錄 App 依賴 `N` 欄位拆分姓名，可能出現「姓氏＝王大明、名字為空」的錯誤。

拆分後產生的 vCard 格式符合規範：

```
N:王;大明;;;
FN:王大明
```

### 下載為何用獨立的高解析度渲染？

畫面上的預覽 QR Code 以 280×280px 渲染，足夠掃描使用，但不適合印刷。下載時另外呼叫 `QRCode.toDataURL()` 以 **1024×1024px** 重新渲染，兩者互不影響。使用者得到的 PNG 檔案解析度足以用於名片、海報等印刷用途。

---

## vCard 格式說明

產生的 QR Code 內容格式如下（以王大明為例）：

```
BEGIN:VCARD
VERSION:3.0
FN:王大明
N:王;大明;;;
TEL;TYPE=CELL:0912345678
EMAIL;TYPE=INTERNET:example@email.com
ORG:某某科技股份有限公司
END:VCARD
```

| 欄位 | 說明 |
|------|------|
| `FN` | 完整顯示姓名，手機通訊錄優先讀取此欄 |
| `N` | 結構化姓名，格式為 `姓氏;名字;;;` |
| `TEL;TYPE=CELL` | 手機號碼，iOS / Android 均識別為行動電話 |
| `EMAIL;TYPE=INTERNET` | 電子郵件（選填） |
| `ORG` | 公司名稱（選填） |

行尾使用 CRLF（`\r\n`），符合 vCard RFC 2426 規範。手機號碼輸入時的空格、連字號、括號會自動清除後再寫入。

---

## 錯誤更正等級

使用 `M`（Medium，15% 回復率）。

選擇 `M` 而非 `H`（High，30%）的原因：vCard 內容包含中文字元，每個中文字在 UTF-8 編碼下佔 3 bytes，使用 `H` 會使 QR Code 格數大幅增加，方塊密度過高導致掃描困難。`M` 在資料容量與掃描便利性之間取得平衡。

---

## 部署

### Cloudflare Pages

```bash
npx wrangler pages deploy . --project-name=vcard-qr-generator --branch=main
```

### GitHub Pages

透過 git push 自動觸發部署：

```bash
git add index.html
git commit -m "update"
git push
```

---

## 使用方式

1. 開啟網頁（或雙擊本機 `index.html`）
2. 填入姓氏、名字、手機號碼（必填），以及電子郵件、公司名稱（選填）
3. 點擊「產生 QR Code」
4. 用手機相機對準 QR Code 掃描
   - iOS：開啟相機 App 直接掃描
   - Android：開啟相機或內建 QR Code 掃描功能
5. 點擊彈出的「加入聯絡人」提示即可儲存

如需留存圖片，點擊「下載圖片」取得 1024×1024px PNG 檔案。

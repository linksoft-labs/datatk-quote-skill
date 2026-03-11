# datatk-quote-skill

基於 [dataTrack](https://www.datatk.com) QuoteNode REST API 的行情查詢技能，支援股票、期貨、外匯等多市場即時與歷史資料查詢。

---

## 目錄

- [前置條件](#前置條件)
- [設定](#設定)
- [快速開始](#快速開始)
- [介面一覽](#介面一覽)
- [命令列使用範例](#命令列使用範例)
- [市場代碼](#市場代碼)
- [錯誤處理](#錯誤處理)
- [參考文件](#參考文件)

---

## 前置條件

1. 安裝 **Node.js 18+**（腳本使用原生 `fetch`，無需額外依賴）。
2. 前往 [dataTrack 服務頁面](https://www.datatk.com/service) 取得你的 **Endpoint** 與 **API Key**。

---

## 設定

編輯專案根目錄下的 `env.json`，填入真實的接入位址與金鑰：

```json
{
  "endpoint": "https://你的接入位址",
  "apiKey":   "你的API Key"
}
```

> `endpoint` 末尾的斜線會被自動去除，無需手動處理。

---

## 快速開始

所有請求使用統一的腳本 `scripts/request.mjs`，透過 `--path` 指定介面路徑，透過 `--body` 傳入 JSON 請求體：

```bash
node scripts/request.mjs --path /Api/V1/Quotation/Detail --body '{"instrument":"US|AAPL","lang":"tc"}'
```

成功時，腳本會將原始 JSON 回應列印至標準輸出。  
請求失敗時（HTTP 狀態碼非 `200`），腳本會列印狀態碼與錯誤訊息後退出。

---

## 介面一覽

| 功能 | 路徑 |
| --- | --- |
| 證券詳情 | `/Api/V1/Quotation/Detail` |
| 證券清單 | `/Api/V1/Basic/BasicInfo` |
| 逐筆成交 | `/Api/V1/Quotation/Tick` |
| Level-2 買賣盤 | `/Api/V1/Quotation/DepthQuoteL2` |
| 分時圖 | `/Api/V1/History/TimeLine` |
| K 線 | `/Api/V1/History/KLine` |
| 券商清單 | `/Api/V1/Quotation/Brokers` |
| 券商佇列 | `/Api/V1/Quotation/Broker` |
| 交易日曆 | `/Api/V1/Basic/Holiday` |

---

## 命令列使用範例

### 查詢美股即時行情（蘋果）

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/Detail \
  --body '{"instrument":"US|AAPL","lang":"tc"}'
```

### 查詢港股清單（第 1 頁，每頁 50 筆）

```bash
node scripts/request.mjs \
  --path /Api/V1/Basic/BasicInfo \
  --body '{"instrument":"HKEX|1","page":1,"pageSize":50,"lang":"tc"}'
```

### 查詢騰訊日 K 線（前復權）

```bash
node scripts/request.mjs \
  --path /Api/V1/History/KLine \
  --body '{"instrument":"HKEX|00700","period":"day","right":1}'
```

### 查詢港股 Level-2 買賣盤（5 檔）

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/DepthQuoteL2 \
  --body '{"instrument":"HKEX|00700|R","depth":5}'
```

### 查詢港股交易日曆

```bash
node scripts/request.mjs \
  --path /Api/V1/Basic/Holiday \
  --body '{"market":"HKEX","startDate":"2025-01-01","endDate":"2025-12-31","lang":"tc"}'
```

### 查詢逐筆成交

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/Tick \
  --body '{"instrument":"HKEX|00700|R","page":1,"pageSize":20}'
```

---

## 市場代碼

### 股票市場

| 代碼 | 說明 |
| --- | --- |
| `SSE` | 上海證券交易所 |
| `SZSE` | 深圳證券交易所 |
| `HKEX` | 香港交易所 |
| `US` | 美國股市 |

### 期貨市場（部分）

| 代碼 | 說明 |
| --- | --- |
| `SHFE` | 上海期貨交易所 |
| `CFFEX` | 中國金融期貨交易所 |
| `COMEX` | 紐約商品交易所 |
| `CME` | 芝加哥商業交易所 |

### 外匯

| 代碼 | 說明 |
| --- | --- |
| `FOREX` | 外匯市場 |

> 完整的市場代碼、證券類型、K 線週期、復權類型等列舉值，見 `references/reference.md`。

---

## instrument 欄位格式

`instrument` 參數的格式為 `{市場}|{代碼}` 或 `{市場}|{代碼}|{資料標誌}`。

| 資料標誌 | 說明 |
| --- | --- |
| `R` | 即時行情 |
| `D` | 延遲行情 |

**範例：**

- `US|AAPL` — 蘋果美股
- `HKEX|00700` — 騰訊港股
- `HKEX|00700|R` — 騰訊港股即時行情
- `COMEX|GC2604` — COMEX 黃金期貨合約

---

## 錯誤處理

HTTP 狀態碼非 `200` 時，回應體格式如下：

```json
{
  "code": 429,
  "reason": "OPENAPI_QUOTA_EXCEEDED",
  "message": "Rate limit exceeded"
}
```

常見狀態碼：

| 狀態碼 | 說明 |
| --- | --- |
| `400` | 請求參數有誤 |
| `401` | 認證失敗，請確認 API Key |
| `403` | 無權限存取該介面或市場 |
| `429` | 超出請求頻率限制 |
| `500` | 服務內部錯誤 |

業務錯誤碼（回應體內 `code` 欄位）：

| 業務碼 | 說明 |
| --- | --- |
| `0` | 成功 |
| `1001` | 參數驗證失敗 |
| `1002` | 找不到該證券 |
| `2001` | 無效的 API Key |
| `2003` | 無介面權限 |
| `2004` | 無市場權限 |

---

## 參考文件

| 文件 | 內容 |
| --- | --- |
| [references/openapi.md](references/openapi.md) | 所有介面路徑、請求參數說明與範例 |
| [references/reference.md](references/reference.md) | 列舉值、市場代碼、錯誤碼 |
| [references/response.md](references/response.md) | 各介面回應欄位含義 |
| [references/architecture.md](references/architecture.md) | REST 層架構說明 |

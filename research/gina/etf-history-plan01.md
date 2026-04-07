# ETF-History 計劃

**建立者：** Gina
**日期：** 2026-04-07
**狀態：** 規劃中

---

## 📊 專案目標

建立 ETF 歷史價格追蹤系統，自動從 Yahoo Finance 獲取歷史資料並同步到 Google Sheet。

---

## ✅ 可用的 Yahoo Finance API

### 基礎 URL

```
https://query1.finance.yahoo.com/v7/finance/download/{TICKER}
```

### 查詢參數

| 參數 | 說明 | 選項 |
|------|------|------|
| `range` | 時間範圍 | `1d`、`5d`、`1mo`、`3mo`、`6mo`、`1y`、`2y`、`5y`、`10y`、`ytd`、`max` |
| `interval` | 資料間隔 | `1d`（日）、`1wk`（週）、`1mo`（月） |
| `start` | 開始日期 | Unix timestamp |
| `end` | 結束日期 | Unix timestamp |
| `events` | 事件類型 | `history`（歷史價格）、`div`（股利）、`split`（分割） |

### 範例 URL

```bash
# 最近一年每日資料
https://query1.finance.yahoo.com/v7/finance/download/VOO?interval=1d&range=1y

# 自訂日期範圍（使用 Unix timestamp）
https://query1.finance.yahoo.com/v7/finance/download/VTI?period1=1704067200&period2=1712496000&interval=1d
```

---

## 📋 回傳資料格式 (CSV)

```csv
Date,Open,High,Low,Close,Adj Close,Volume
2026-01-02,465.23,472.15,465.23,470.18,470.18,2185600
2026-01-03,470.50,475.68,468.90,472.45,472.45,1954300
...
```

| 欄位 | 說明 |
|------|------|
| Date | 日期 |
| Open | 開盤價 |
| High | 最高價 |
| Low | 最低價 |
| Close | 收盤價 |
| Adj Close | 調整後收盤價（考慮股利、分割） |
| Volume | 成交量 |

---

## 🏗️ 實作架構選項

### 選項 A：Shell Script + gog CLI

**優點：**
- 簡單直接
- 適合一次性任務
- 無需額外安裝套件

**缺點：**
- 錯誤處理較弱
- CSV 解析需手動處理

**參考指令：**
```bash
# 下載並解析 CSV
curl -s "https://query1.finance.yahoo.com/v7/finance/download/VOO?interval=1d&range=30d" \
  | tail -n +2 \
  | while IFS=',' read -r date open high low close adj_close volume; do
      gog sheets append <SHEET_ID> "ETF-History" \
        --values-json '[["'"$date"'","VOO",'"$open"',...]]'
    done
```

---

### 選項 B：Python + yfinance + gog

**優點：**
- 功能強大
- `yfinance` 套件處理 Yahoo Finance 限制更穩定
- 自動錯誤處理、重試機制
- 資料處理彈性高

**缺點：**
- 需要安裝 Python 和套件

**參考程式碼：**
```python
import yfinance as yf
from datetime import datetime, timedelta

# 下載歷史資料
ticker = yf.Ticker("VOO")
hist = ticker.history(period="1y", interval="1d")

# 轉換為 Google Sheets 格式
for date, row in hist.iterrows():
    # 上傳到 Google Sheets
    pass
```

---

### 選項 C：定時 Cron Job

**優點：**
- 自動化，無需手動執行
- 可設定任務排程

**實作方式：**
- 使用 OpenClaw `cron` 工具
- 或系統 cron + shell script

---

## 📝 Google Sheet 結構設計

### Sheet 名稱：`ETF-History`

| 欄位 (A-G) | 說明 | 範例 |
|-----------|------|------|
| A | 日期 | 2026-01-02 |
| B | 代號 | VOO |
| C | 開盤價 | 465.23 |
| D | 最高價 | 472.15 |
| E | 最低價 | 465.23 |
| F | 收盤價 | 470.18 |
| G | 成交量 | 2185600 |

### 標題列 (Row 1)

```
日期,代號,開盤價,最高價,最低價,收盤價,成交量
```

---

## 🎯 實施步驟

### Phase 1：手動測試
1. 建立新的 Google Sheet 標籤 `ETF-History`
2. 手動抓取一個 ETF 的歷史資料（VOO，30 天）
3. 解析 CSV 並寫入 Google Sheet
4. 驗證資料格式和完整性

### Phase 2：編寫自動化 Script
1. 選擇實作架構（選項 A 或 B）
2. 支援多個 ETF 批次處理
3. 增加錯誤處理和日誌記錄

### Phase 3：定時任務設定
1. 設定每日或每週自動更新
2. 整合到 OpenClaw cron 系統
3. 監控執行狀態

---

## 📌 注意事項

### API 限制
- Yahoo Finance 有請求頻率限制
- 過度請求可能被封鎖
- 建議批次處理，並在請求間加入延遲

### 資料來源
- Yahoo Finance 為免費公開資料
- 資料可能有延遲或不準確
- 建議交叉驗證重要資料

### Google Sheets 限制
- 單次寫入有最大行數限制（建議分段處理）
- 寫入頻率過高可能觸發 API 限流

---

## 🔄 後續優化方向

1. **支援多時間期間隔**：日、週、月
2. **加入技術指標**：MA、RSI、布林帶
3. **視覺化圖表**：自動生成走势圖
4. **警報通知**：價格突破、成交量異常
5. **多市場支援**：台灣、港股、ETF

---

## 📚 參考資源

- [Yahoo Finance API Guide](https://algotrading101.com/learn/yahoo-finance-api-guide/)
- [yfinance Python Documentation](https://pypi.org/project/yfinance/)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [gog CLI Documentation](/app/skills/gog/SKILL.md)

---

_最後更新：2026-04-07_
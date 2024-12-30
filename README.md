# 股票分析與 MACD 指標檢測

## 1. 專案簡介
本專案提供一套基於 Python 的股票分析工具，通過 MACD 指標檢測股價可能的跌點，並提供視覺化圖表展示分析結果。適合用於技術分析和市場走勢判斷。

---

## 2. 功能概述
1. **資料獲取** - 使用 Yahoo Finance 提取指定股票的歷史數據。
2. **指標計算** - 計算 MACD 和信號線，分析市場趨勢。
3. **跌點檢測** - 判定潛在的跌點並驗證其精準性。
4. **視覺化分析** - 使用折線圖和散點圖展示股價與 MACD 指標走勢。

---

## 3. 環境需求
- Python 3.x
- 安裝必要套件：
  ```bash
  pip install pandas yfinance matplotlib
  ```

---

## 4. 程式碼結構
### 4.1 引入模組
```python
import pandas as pd
import yfinance as yf
import matplotlib.pyplot as plt
```
**說明：**
- `pandas`：處理時序數據。
- `yfinance`：下載股票歷史資料。
- `matplotlib.pyplot`：生成圖表。

### 4.2 獲取股票數據
```python
def fetch_stock_data(ticker, period="1y"):
    stock_data = yf.download(ticker, period=period)
    return stock_data['Close']
```
**功能：**
- 獲取指定股票的歷史收盤價。
- 支援時間範圍選擇（預設為 1 年）。

### 4.3 計算 MACD 指標
```python
def calculate_macd(prices, short_window=12, long_window=26, signal_window=9):
    short_ema = prices.ewm(span=short_window, adjust=False).mean()
    long_ema = prices.ewm(span=long_window, adjust=False).mean()
    macd = short_ema - long_ema
    signal_line = macd.ewm(span=signal_window, adjust=False).mean()
    return macd, signal_line
```
**功能：**
- 計算 MACD 和信號線，用於分析市場動能。
- 支援短期、長期和信號線窗口參數調整。

### 4.4 找出可能的跌點
```python
def find_drop_points(prices, macd, signal_line):
    osc = macd - signal_line
    death_cross = (macd < signal_line) & (macd.shift(1) > signal_line.shift(1)) & (osc < 0)
    osc_decline = (
        (osc < osc.shift(1)) &
        (osc.shift(1) < osc.shift(2)) &
        ((osc.shift(1) - osc) > 0.01 * prices)
    )
    price_highs = (prices > prices.shift(1)) & (prices > prices.shift(-1))
    divergence = death_cross & price_highs
    combined_conditions = death_cross | osc_decline | divergence
    filtered_conditions = (
        combined_conditions &
        ((death_cross.astype(int) + osc_decline.astype(int) + divergence.astype(int)) >= 2)
    )
    drop_points = prices[filtered_conditions].index
    return drop_points
```
**功能：**
- 定義跌點條件：
  1. 死亡交叉：MACD 下穿信號線。
  2. 柱狀圖連續下降。
  3. 價格高點後出現交叉。
- 綜合條件過濾以減少誤判。

### 4.5 驗證跌點
```python
def validate_drop_points(prices, drop_points, threshold=0.02):
    precise_drops = []
    inaccurate_drops = []
    for date in drop_points:
        if date in prices.index:
            future_prices = prices.loc[date:].iloc[:5]
            if len(future_prices) > 1:
                min_price = future_prices.min()
                price_change = (min_price - prices.loc[date]) / prices.loc[date]
                price_change = float(price_change)
                if price_change <= -threshold:
                    precise_drops.append(date)
                else:
                    inaccurate_drops.append(date)
    return precise_drops, inaccurate_drops
```
**功能：**
- 驗證跌點後 5 天內是否達到預設跌幅（如 -2%）。
- 標記精準與非精準跌點。

### 4.6 繪圖
```python
def plot_macd(prices, macd, signal_line, precise_drops, inaccurate_drops):
    plt.figure(figsize=(16, 10))
    plt.subplot(2, 1, 1)
    plt.plot(prices, label="Close Price", color='blue')
    plt.scatter(precise_drops, prices.loc[precise_drops], color='green', label="Precise Drops (Green)")
    plt.scatter(inaccurate_drops, prices.loc[inaccurate_drops], color='red', label="Inaccurate Drops (Red)")
    plt.title("Stock Price and Drop Points")
    plt.legend()
    plt.subplot(2, 1, 2)
    plt.plot(macd.index, macd, label="MACD Line", color='green')
    plt.plot(signal_line.index, signal_line, label="Signal Line", color='red')
    plt.axhline(y=0, color='black', linestyle='--', label="Zero Line")
    plt.title("MACD Indicator")
    plt.legend()
    plt.tight_layout()
    plt.show()
```
**功能：**
- 可視化收盤價及標示精準與非精準跌點。
- 繪製 MACD 指標和信號線。

---

## 5. 主程式執行流程
```python
if __name__ == "__main__":
    stock_ticker = "AAPL"
    prices = fetch_stock_data(stock_ticker)
    macd, signal_line = calculate_macd(prices)
    drop_points = find_drop_points(prices, macd, signal_line)
    precise_drops, inaccurate_drops = validate_drop_points(prices, drop_points)
    plot_macd(prices, macd, signal_line, precise_drops, inaccurate_drops)
```
**說明：**
- 使用蘋果公司（AAPL）股票作為範例。
- 分析股價與 MACD 指標，顯示跌點與驗證結果。

---

## 6. 結論
該工具適合技術分析愛好者與投資者用於市場趨勢判斷，特別適合短線操作與跌點預測分析。後續可擴展更多技術指標與風險管理功能。


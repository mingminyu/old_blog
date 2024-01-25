# Akshare 教程

<show-structure depth="2"/>

Akshare 是一个基于 Python的 金融数据获取和分析工具，它提供了广泛的金融市场数据，包括股票、期货、外汇、基金等各种类型的数据。同时，Akshare 的简单易用性和丰富的功能使得它成为金融数据分析和量化交易的理想选择。

## 1. 安装

Akshare 的安装非常简单，可以直接用 `pip` 进行安装：

```Bash
pip install akshare
```

## 2. 获取数据

### 2.1 获取股票日线行情数据

使用 `stock_zh_a_daily` 可以获取指定股票代码的日行情数据。

```Python
import akshare as ak

# 获取股票日线行情数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )
print(stock_data)
```

### 2.2 获取基金数据

使用 `fund_em_open_fund_info` 可以获取指定股票代码的日行情数据。

```Python
import akshare as ak

# 获取基金净值数据
fund_data = ak.fund_em_open_fund_info(
    fund="000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )
print(fund_data)
```

### 2.3 获取期货数据

使用 `futures_zh_daily` 获取期货日线行情数据。

```Python
import akshare as ak

# 获取期货日线行情数据
futures_data = ak.futures_zh_daily(
    symbol="cu2103", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )
print(futures_data)
```

### 2.4 获取外汇数据

使用 `forex_ak_m1`  获取外汇汇率数据。

```Python
import akshare as ak

# 获取外汇汇率数据
exchange_rate_data = ak.forex_ak_m1()
print(exchange_rate_data)
```

## 3. 数据分析和可视化

除了获取数据，Akshare 还提供了数据分析和可视化的功能。

### 3.1 数据可视化

结合 Matplotlib，我们可以基于获取的数据绘制股票价格走势图。

```Python
import akshare as ak
import matplotlib.pyplot as plt

# 获取股票日线行情数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )

# 绘制股票价格走势图
plt.plot(stock_data["date"], stock_data["close"])
plt.xlabel("Date")
plt.ylabel("Close Price")
plt.title("Stock Price Trend")
plt.show()
```

### 3.2 数据分析

因为 Akshare 返回的结果是 DataFrame，我们很容易做一些衍生计算，比如计算股票收益率。

```Python
import akshare as ak

# 获取股票日线行情数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )

# 计算股票收益率
stock_data["return"] = stock_data["close"].pct_change()
print(stock_data["return"].describe())
```

## 4. 应用场景

### 4.1 量化交易策略开发

使用 Akshare 获取股票数据并开发简单的均线策略

```Python

import akshare as ak
import pandas as pd

# 获取股票日线行情数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )

# 计算5日和20日均线
stock_data["ma5"] = stock_data["close"].rolling(window=5).mean()
stock_data["ma20"] = stock_data["close"].rolling(window=20).mean()

# 生成买入信号（ma5向上穿越ma20）
stock_data["buy_signal"] = (stock_data["ma5"] > stock_data["ma20"]) & (stock_data["ma5"].shift(1) <= stock_data["ma20"].shift(1))

# 生成卖出信号（ma5向下穿越ma20）
stock_data["sell_signal"] = (stock_data["ma5"] < stock_data["ma20"]) & (stock_data["ma5"].shift(1) >= stock_data["ma20"].shift(1))

# 打印交易信号
signals = stock_data[["date", "buy_signal", "sell_signal"]]
print(signals)
```
{collapsible="true" default-state="expanded"}

> Pandas Series 的 `rolling` 方法可以滚动求值，比如使用 `rolling(window=5).mean()` 计算当前数据前 5 条的均值。
{style="note"}

### 4.2 金融市场研究

使用 Akshare 获取不同市场的历史数据并进行比较分析

```Python
import akshare as ak
import matplotlib.pyplot as plt

# 获取A股指数数据
a_share_data = ak.stock_zh_index_daily(
    index="000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31")

# 获取美股指数数据
us_stock_data = ak.stock_us_daily(symbol="DJI", start_date="2022-01-01", end_date="2022-12-31")

# 绘制A股和美股指数比较图
plt.figure(figsize=(12, 6))
plt.plot(
    a_share_data["date"], 
    a_share_data["close"], 
    label="A股指数"
    )
plt.plot(
    us_stock_data["date"], 
    us_stock_data["close"], 
    label="美股指数"
    )
plt.xlabel("Date")
plt.ylabel("Close Price")
plt.title("A股和美股指数比较")
plt.legend()
plt.show()
```
{collapsible="true" default-state="expanded"}

### 4.3 投资组合管理

使用 Akshare 跟踪不同资产的表现并生成投资组合报告。

```Python
import akshare as ak

# 获取股票、基金和期货数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )
fund_data = ak.fund_em_open_fund_info(
    fund="000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )
futures_data = ak.futures_zh_daily(
    symbol="cu2103", 
    start_date="2022-01-01", 
    end_date="2022-12-31"
    )

# 计算不同资产的年度收益率
stock_return = (stock_data["close"].iloc[-1] / stock_data["close"].iloc[0]) - 1
fund_return = (fund_data["unit_nav"].iloc[-1] / fund_data["unit_nav"].iloc[0]) - 1
futures_return = (futures_data["close"].iloc[-1] / futures_data["close"].iloc[0]) - 1

# 打印投资组合报告
print("股票收益率：", stock_return)
print("基金收益率：", fund_return)
print("期货收益率：", futures_return)
```
{collapsible="true" default-state="expanded"}

### 4.4 金融数据报告

使用 Akshare 生成股票收益率报告和可视化图表。

```Python

import akshare as ak
import pandas as pd
import matplotlib.pyplot as plt

# 获取股票日线行情数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2022-01-01", 
    end_date="2022-12-31")

# 计算股票收益率
stock_data["return"] = stock_data["close"].pct_change()

# 生成收益率报告
return_report = stock_data["return"].describe()

# 绘制收益率分布图
plt.figure(figsize=(10, 6))
plt.hist(stock_data["return"].dropna(), bins=30, alpha=0.7)
plt.xlabel("Return")
plt.ylabel("Frequency")
plt.title("Stock Return Distribution")
plt.show()

# 打印收益率报告
print(return_report)
```
{collapsible="true" default-state="expanded"}


### 4.5 数据挖掘和预测

```Python
import akshare as ak
import pandas as pd
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

# 获取股票日线行情数据
stock_data = ak.stock_zh_a_daily(
    symbol="sz000001", 
    start_date="2020-01-01", 
    end_date="2021-12-31"
    )

# 选择用于预测的特征（以日期为基础进行编码）
stock_data["date"] = pd.to_datetime(stock_data["date"])
stock_data["day_of_year"] = stock_data["date"].dt.dayofyear
X = stock_data[["day_of_year"]].values
y = stock_data["close"].values

# 训练线性回归模型
model = LinearRegression()
model.fit(X, y)

# 预测未来30天的股价
future_dates = pd.date_range(start="2023-01-01", periods=30, closed="right")
future_day_of_year = future_dates.dayofyear.values.reshape(-1, 1)
future_predictions = model.predict(future_day_of_year)

# 绘制股价预测图
plt.figure(figsize=(12, 6))
plt.plot(stock_data["date"], stock_data["close"], label="历史股价")
plt.plot(future_dates, future_predictions, label="预测股价", linestyle = "--")
plt.xlabel("Date")
plt.ylabel("Close Price")
plt.title("Stock Price Prediction")
plt.legend()
plt.show()
```
{collapsible="true" default-state="expanded"}


## 5. 总结

Akshare 更多是一个数据获取工具，但由于它返回的是 DataFrame 数据格式，结合 Pandas 进行数据分析，或者结合 Matplotlib 进行可视化都非常方便。

<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/d3TYcwmFqENxNHEDP7ihNA">Akshare 简介</a>
    <a href="https://akshare.akfamily.xyz/index.html">Akshare 文档</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
    <a href="https://github.com/akfamily/akshare">Akshare Github</a>
</category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>
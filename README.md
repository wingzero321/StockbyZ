# StockTradebyZ - 股票交易策略实现

> 基于Z哥少妇战法、补票战法、TePu战法的Python实现

![Python Version](https://img.shields.io/badge/python-3.10%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Last Updated](https://img.shields.io/badge/last%20updated-2025--06--23-brightgreen)

## 项目简介

StockTradebyZ是一个基于Python的股票交易策略实现工具，集成了数据获取、技术指标计算和选股策略执行的完整流程。项目实现了三种经典的选股策略：少妇战法、补票战法和TePu战法，可以帮助投资者进行量化选股。

| 核心组件                  | 功能简介                                                                                                             |
| --------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **`fetch_kline.py`**  | *按市值筛选* A 股股票，并抓取其**历史 K 线**保存为 CSV。支持 **AkShare / Tushare / Mootdx** 三大数据源，自动增量更新、多线程下载。*本版本不再保存市值快照*，每次运行实时拉取。 |
| **`select_stock.py`** | 读取本地 CSV 行情，依据 `configs.json` 中的 **Selector** 定义批量选股，结果输出到 `select_results.log` 与控制台。                            |
| **`Selector.py`**     | 实现了三种选股策略：**BBIKDJSelector**（少妇战法）、**BBIShortLongSelector**（补票战法）、**BreakoutVolumeKDJSelector**（TePu 战法）。        |

## 功能特点

- **多数据源支持**：可从AkShare、Tushare或Mootdx获取A股历史K线数据
- **市值筛选**：根据市值范围筛选股票，过滤掉不符合条件的标的
- **增量更新**：自动识别已下载数据，仅更新最新交易日数据
- **多线程下载**：支持并发下载，提高数据获取效率
- **多策略选股**：内置三种经典选股策略，可通过配置文件自定义参数
- **灵活配置**：通过JSON配置文件调整策略参数，无需修改代码

## 快速上手

### 安装依赖

```bash
# 建议 Python 3.10+，并使用虚拟环境
pip install -r requirements.txt
```

> 主要依赖：`akshare`、`tushare`、`mootdx`、`pandas`、`tqdm` 等。

### Tushare Token（可选）

若选择 **Tushare** 作为数据源，请按以下步骤操作：

1. **注册账号**
   点击专属注册链接 [https://tushare.pro/register?reg=820660](https://tushare.pro/register?reg=820660) 完成注册。*通过该链接注册，我将获得 50 积分 – 感谢支持！*
2. **开通基础权限**
   登录后进入「**平台介绍 → 社区捐助**」，按提示捐赠 **200 元/年** 可解锁 Tushare 基础接口。
3. **获取 Token**
   打开个人主页，点击 **「接口 Token」**，复制生成的 Token。
4. **填入代码**
   在 `fetch_kline.py` 约 **第 302 行**：

   ```python
   ts_token = "***"  # ← 替换为你的 Token 
   ```

### Mootdx 运行前置步骤

使用 **Mootdx** 数据源前，需先探测最快行情服务器一次：

```bash
python -m mootdx bestip -vv
```

脚本将保存最佳 IP，后续抓取更稳定。

### 下载历史行情

```bash
python fetch_kline.py \
  --datasource mootdx      # mootdx / akshare / tushare
  --frequency 4            # K 线频率编码（4 = 日线）
  --exclude-gem            # 排除创业板 / 科创板 / 北交所
  --min-mktcap 5e9         # 最小总市值（元）
  --max-mktcap +inf        # 最大总市值（元）
  --start 20200101         # 起始日期（YYYYMMDD 或 today）
  --end today              # 结束日期
  --out ./data             # 输出目录
  --workers 10             # 并发线程数
```

*首次运行* 将下载完整历史数据；之后脚本会 **增量更新**，只下载新的交易日数据。

### 运行选股

```bash
python select_stock.py \
  --data-dir ./data        # CSV 行情目录
  --config ./configs.json  # Selector 配置
  --date 2025-06-21        # 交易日（缺省 = 最新）
```

示例输出：

```
============== 选股结果 [TePu 战法] ===============
交易日: 2025-06-21
符合条件股票数: 1
600690
```

## 策略说明

### 1. BBIKDJSelector（少妇战法）

少妇战法是一种结合BBI指标和KDJ指标的选股策略，主要关注BBI指标上升趋势和KDJ指标的超卖区域。

**核心逻辑**：
- BBI指标连续上升一定天数，表明股票处于上升趋势
- J值低于阈值，表明股票可能处于超卖区域
- DIF值大于0，表明MACD指标处于多头区域

**适用场景**：适合在调整后寻找反弹机会，或在上升趋势中寻找调整后的买入时机。

### 2. BBIShortLongSelector（补票战法）

补票战法是一种基于BBI指标和短长期RSV对比的选股策略，主要关注短期超跌后的反弹机会。

**核心逻辑**：
- BBI指标上升，表明股票处于上升趋势
- 短期RSV低于长期RSV，表明短期超跌
- DIF值大于0，表明MACD指标处于多头区域

**适用场景**：适合在上升趋势中寻找短期调整后的买入机会。

### 3. BreakoutVolumeKDJSelector（TePu战法）

TePu战法是一种结合放量突破和KDJ指标的选股策略，主要关注放量突破后的买入机会。

**核心逻辑**：
- 股价在一定周期内出现放量上涨
- 随后出现缩量调整
- J值低于阈值，表明股票可能处于超卖区域
- DIF值大于0，表明MACD指标处于多头区域

**适用场景**：适合在放量突破后寻找缩量调整的买入机会。

## 参数说明

### `fetch_kline.py`

| 参数                  | 默认值      | 说明                                   |
| ------------------- | -------- | ------------------------------------ |
| `--datasource`      | `mootdx` | 数据源：`tushare` / `akshare` / `mootdx` |
| `--frequency`       | `4`      | K 线频率编码（下表）                          |
| `--exclude-gem`     | flag     | 排除创业板/科创板/北交所                        |
| `--min-mktcap`      | `5e9`    | 最小总市值（元）                             |
| `--max-mktcap`      | `+inf`   | 最大总市值（元）                             |
| `--start` / `--end` | `today`  | 日期范围，`YYYYMMDD` 或 `today`            |
| `--out`             | `./data` | 输出目录                                 |
| `--workers`         | `10`     | 并发线程数                                |

#### K 线频率编码

|  编码 |  周期  | Mootdx 关键字 | 用途   |
| :-: | :--: | :--------: | ---- |
|  0  |  5 分 |    `5m`    | 高频   |
|  1  | 15 分 |    `15m`   | 高频   |
|  2  | 30 分 |    `30m`   | 高频   |
|  3  | 60 分 |    `1h`    | 波段   |
|  4  |  日线  |    `day`   | ★ 常用 |
|  5  |  周线  |   `week`   | 中长线  |
|  6  |  月线  |    `mon`   | 中长线  |
|  7  |  1 分 |    `1m`    | Tick |
|  8  |  1 分 |    `1m`    | Tick |
|  9  |  日线  |    `day`   | 备用   |
|  10 |  季线  |   `3mon`   | 长周期  |
|  11 |  年线  |   `year`   | 长周期  |

### `select_stock.py`

| 参数           | 默认值              | 说明            |
| ------------ | ---------------- | ------------- |
| `--data-dir` | `./data`         | CSV 行情目录      |
| `--config`   | `./configs.json` | Selector 配置文件 |
| `--date`     | 最新交易日            | 选股日期          |
| `--tickers`  | `all`            | 股票池（逗号分隔列表）   |

其他参数请执行 `python select_stock.py --help` 查看。

### 内置策略参数

> 参数位于 `configs.json`，以下仅列常用项。

#### 1. BBIKDJSelector（少妇战法）

| 参数                | 默认    | 说明         |
| ----------------- | ----- | ---------- |
| `threshold`       | `-6`  | 当日 J 值上限   |
| `bbi_min_window`  | `17`  | BBI 上升最短天数 |
| `bbi_offset_n`    | `2`   | 锚点偏移       |
| `max_window`      | `60`  | 最大窗口       |
| `price_range_pct` | `100` | 价格波动过滤 (%) |

#### 2. BBIShortLongSelector（补票战法）

| 参数               | 默认   | 说明     |
| ---------------- | ---- | ------ |
| `n_short`        | `3`  | 短 RSV  |
| `n_long`         | `21` | 长 RSV  |
| `m`              | `3`  | 判别窗口   |
| `bbi_min_window` | `5`  | BBI 窗口 |
| `bbi_offset_n`   | `0`  | 锚点偏移   |
| `max_window`     | `60` | 最大窗口   |

#### 3. BreakoutVolumeKDJSelector（TePu 战法）

| 参数                 | 默认       | 说明       |
| ------------------ | -------- | -------- |
| `j_threshold`      | `1`      | 当日 J 值上限 |
| `up_threshold`     | `3.0`    | 放量涨幅 (%) |
| `volume_threshold` | `0.6667` | 缩量比例     |
| `offset`           | `15`     | 放量窗口 (日) |
| `max_window`       | `60`     | 最大窗口     |
| `price_range_pct`  | `100`    | 价格波动 (%) |

## 项目结构

```
.
├── appendix.json            # 附加股票池
├── configs.json             # Selector 配置
├── fetch_kline.py           # 行情抓取脚本
├── select_stock.py          # 批量选股脚本
├── Selector.py              # 策略实现
├── data/                    # CSV 数据输出目录
├── fetch.log                # 抓取日志
└── select_results.log       # 选股日志
```

## 核心代码解析

### 数据获取 (fetch_kline.py)

`fetch_kline.py` 是数据获取的核心模块，支持从多个数据源获取股票历史K线数据。主要功能包括：

1. 获取A股股票列表并按市值筛选
2. 从选定的数据源（tushare、akshare、mootdx）获取历史K线数据
3. 数据校验和增量更新
4. 多线程并
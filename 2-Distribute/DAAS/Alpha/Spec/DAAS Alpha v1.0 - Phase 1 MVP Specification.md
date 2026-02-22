
#### 1. 项目概述 (Project Overview)

项目名称： Alpha-MVP-Phase1

目标： 构建一个基于 Python 的 A 股量化辅助系统。该系统包含两个核心模块：

1. **基本面选股 (The Anchor)：** 基于价值和质量因子，每日自动筛选出具有“高安全垫”的白名单股票。
2. **公告异动监控 (The Shield)：** 针对白名单内的股票，监控其近期公告，利用关键词匹配捕捉潜在利好或利空。

#### 2. 技术栈 (Tech Stack)

- **语言：** Python 3.10+
- **数据源：** Tushare Pro (需配置 Token)
- **数据处理：** Pandas, Numpy
- **配置管理：** `.env` (用于存放敏感 Token) / `config.yaml` (用于存放策略参数)
- **输出：** CSV (股票池), Markdown (每日分析报告)

#### 3. 目录结构 (Directory Structure)

请按照以下结构生成项目文件：
Plaintext
```
DAAS-Alpha/
├── config/
│   ├── settings.yaml      # 存放策略阈值（如 PE < 20, ROE > 10）
│   └── keywords.yaml      # 存放公告监控的关键词（正面/负面）
├── data/
│   ├── raw/               # 存放下载的原始数据
│   └── output/            # 存放生成的 CSV 和报告
├── src/
│   ├── __init__.py
│   ├── data_loader.py     # 封装 Tushare API，负责获取数据
│   ├── strategy.py        # 核心选股逻辑（多因子筛选）
│   ├── monitor.py         # 公告关键词匹配逻辑
│   └── reporter.py        # 生成 Markdown 报告
├── .env                   # 存放 TUSHARE_TOKEN
├── main.py                # 程序入口
└── requirements.txt       # 依赖库
```

#### 4. 功能模块详细说明 (Functional Requirements)

**4.1 基础配置与数据层 (`data_loader.py`)**

- **功能：** 初始化 Tushare Pro 接口。
- **输入：** 从 `.env` 读取 Token。
- **方法：**
    
    - `get_stock_basics()`: 获取全市场股票列表（包含上市状态、ST状态）。
        
    - `get_daily_indicators(trade_date)`: 获取每日基本面指标（PE_TTM, PB, 股息率, 总市值）。
        
    - `get_financial_indicators(trade_date)`: 获取财务指标（ROE, 净利润增长率）。
        
    - `get_notices(stock_list, start_date)`: 获取指定股票池的公告信息。
        

**4.2 策略模块 (`strategy.py`) - "价值守门员"**

- **逻辑：** 必须同时满足以下所有条件才入选“白名单” (Anchor Pool)。
    
- **筛选规则 (参数需从 settings.yaml 读取)：**
    
    1. **排除垃圾股：** 排除名字包含 "ST", "*ST" 的股票；排除上市时间 < 365 天的新股。
        
    2. **估值安全 (Value)：** `0 < PE_TTM < 30` 且 `PB < 5`。
        
    3. **盈利能力 (Quality)：** `ROE > 8%` (最新财报数据)。
        
    4. **分红回报 (Yield)：** `股息率 (Dividend Yield) > 1.5%`。
        
- **输出：** 返回一个 DataFrame，包含股票代码、名称、所属行业、当前价格及相关因子数据。
    

**4.3 监控模块 (`monitor.py`) - "事件狙击手"**

- **逻辑：** 仅对 `strategy.py` 筛选出的股票进行扫描。
    
- **规则：** 获取这些股票过去 3 天的公告标题，进行关键词匹配。
    
- **关键词 (需从 keywords.yaml 读取)：**
    
    - **Positive (利好):** ["增持", "回购", "预增", "中标", "分红", "股权激励"]
        
    - **Negative (利空/警示):** ["减持", "立案", "调查", "亏损", "违规", "警示函"]
        
- **输出：** 返回一个列表，包含：股票代码、公告日期、公告标题、命中关键词、情感标签 (Positive/Negative)。
    

**4.4 报告模块 (`reporter.py`)**

- **功能：** 将选股结果和公告监控结果汇总生成一份 `YYYY-MM-DD_Analysis_Report.md`。
    
- **格式：**
    
    - Title: 日期 + 选股数量。
        
    - Section 1: **今日高安全垫白名单** (表格形式 display top 20 by ROE)。
        
    - Section 2: **重要异动公告** (列出命中关键词的公告，高亮显示)。
        

#### 5. 执行流程 (Execution Flow in `main.py`)

1. 加载配置。
    
2. 确定日期（默认为今天，如果是周末则自动前推至上一个交易日）。
    
3. 调用 `data_loader` 获取全市场数据。
    
4. 调用 `strategy` 执行选股，生成 `anchor_pool`。
    
5. 保存 `anchor_pool` 到 CSV。
    
6. 调用 `data_loader` 获取 `anchor_pool` 内股票的近期公告。
    
7. 调用 `monitor` 分析公告。
    
8. 调用 `reporter` 生成 Markdown 报告。
    
9. Print "任务完成，报告已生成：[项目]/reports/"。
    

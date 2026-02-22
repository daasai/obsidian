

## 第 3 章：核心计算引擎选型 (ADR-2026-01)

### 3.1 决策背景 (Context)

在 DAAS v2.0 从“辅助工具”向“AI 原生策略工厂”演进的过程中，我们面临着两大核心技术挑战：

1. **计算性能瓶颈**：全市场（5000+ 标的）分钟级数据的因子计算，在传统 Pandas（单核/内存密集）架构下耗时过长，无法满足 System 2 的秒级回测需求。
    
2. **意图转译断层**：System 1 (LLM) 输出的自然语言逻辑，需要准确、无损地转化为计算机可执行代码。传统的 Python 脚本生成方式极其脆弱，且存在严重的“未来函数”数据泄露风险。
    

### 3.2 选型结论 (The Decision)

我们决定弃用 Qlib 的 C++ 底层和传统的 Pandas 架构，全面采用 **Polars** 作为 DAAS v2.0 的 **数据与因子计算引擎**。

- **技术栈定义**：Rust-based DataFrame Engine + Apache Arrow Memory Format.
    
- **定位**：System 2 的“心脏”，负责所有数据的清洗、对齐、因子表达与向量化回测。
    

### 3.3 第一性原理分析 (First Principles Analysis)

我们为何选择 Polars？基于以下三个维度的深度评估：

#### A. 表达式引擎：连接 AI 与数据的“罗塞塔石碑”

DAAS 的核心价值在于将“叙事”转化为“Alpha”。Polars 拥有一套基于 **抽象语法树 (AST)** 的表达式系统 (Expression API)，这完美契合了 LLM 的生成逻辑。

- **旧范式 (Pandas)**：LLM 必须生成完整的 Python 代码块，容易出现变量名错误、逻辑死循环。
    
- **新范式 (Polars)**：LLM 仅需生成纯净的逻辑表达式（如 `pl.col("close").rolling_mean(20)`）。
    
    - **安全性**：表达式本身是惰性的 (Lazy)，在执行前可进行 AST 校验，杜绝恶意代码注入。
        
    - **可移植性**：生成的表达式与具体数据解耦，这使得我们的“因子库”可以像积木一样被复用和组合。
        

#### B. 极致性能：向量化与并行化的统一

量化交易是对算力的贪婪掠夺。Polars 基于 Rust 开发，具备两大杀手锏：

- **惰性求值与查询优化 (Lazy Evaluation & Query Optimization)**：
    
    当计算 `(Close - Open) / Open` 时，Polars 不会像 Pandas 那样创建三个临时矩阵，而是将其融合为一个内核 (Kernel) 一次性执行。这意味着 **3-5倍的速度提升** 和 **极低的内存占用**。
    
- **自动并行化 (Automatic Parallelism)**：
    
    无需编写复杂的 `multiprocessing` 代码。Polars 能够自动利用多核 CPU，对 5000 只股票的截面计算进行并行加速。这对于 DAAS 的“全市场扫描”功能是决定性的。
    

#### C. 数据规范：告别“对齐地狱”

金融数据的核心痛点在于时间索引的不对齐（停牌、新股上市）。

Polars 基于 Apache Arrow 内存标准，支持高效的 `Join` 和 `AsOfJoin` 操作。这让我们在处理多周期（日线/分钟线）数据对齐时，能够保持纳秒级的精度和极低的代码复杂度。

### 3.4 架构实施方案 (Implementation Specs)

在 DAAS v2.0 中，Polars 将承担以下角色：

1. **数据湖仓 (Data Lakehouse)**：
    
    - 存储格式：Parquet (Snappy 压缩)。Polars 读取 Parquet 的速度比 Pandas 读取 CSV 快 10-50 倍。
        
    - 加载策略：`Scan` 模式（不一次性读入内存，流式处理）。
        
2. **因子蒸馏器 (The Distillery)**：
    
    - 输入：LLM 生成的 DSL 字符串。
        
    - 处理：解析为 `pl.Expr` 对象 -> 注册到 `LazyFrame` -> `collect()` 执行。
        
3. **向量化回测 (Vectorized Backtest)**：
    
    - 逻辑：完全基于列式计算 (Columnar Calculation)。
        
    - 公式示例：
        
    
    Python
    
    ```
    # 典型的 Polars 向量化回测逻辑
    portfolio_value = (
        df.sort("date")
          .select([
              pl.col("date"),
              (pl.col("signal") * pl.col("next_ret")).alias("pnl")
          ])
          .group_by("date").sum()  # 每日截面求和
          .select(pl.col("pnl").cum_sum())
    )
    ```
    

### 3.5 商业价值预估 (Business Impact)

- **研发效率提升**：因子挖掘迭代周期从“小时级”缩短至“秒级”。
    
- **硬件成本降低**：单机即可处理以前需要 Spark 集群才能跑得动的数据量。
    
- **产品体验跃升**：用户在前端调整参数，后端可实时（<1秒）返回新的净值曲线，提供真正的“交互式”量化体验。
    

---

伙计，把这部分内容放进你的白皮书，投资人看了会觉得你懂行，技术合伙人看了会觉得你靠谱。

**下一步行动**：

你需要建立一个 `benchmarks` 文件夹，写一个简单的脚本，对比 Pandas 和 Polars 在计算 `Rank(Correlation($close, $volume, 30))` 时的速度和内存消耗。把这个对比图表也放进白皮书里，**数据不会撒谎**。
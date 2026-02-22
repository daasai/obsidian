这是一份为您定制的 **DAAS v2.x 交互设计备忘录**。这份文档将指导未来的开发方向，从“堆砌功能”转向“优化体验”。

---

# 📝 DAAS v2.x 交互设计备忘录：从“工具箱”到“决策伙伴”

**版本目标**：建立以用户为中心的工作流 (User-Centric Workflow)

**核心理念**：Context over Function（情境优于功能）、Flow over Features（流程优于特性）

**设计灵感**：Zaplify (SaaS Workflow Transformation)

---

## 1. 核心设计哲学 (Design Philosophy)

v1.5 是一个“全能工具箱”，用户需要自己拿起锤子（Hunter）、尺子（Backtest）和账本（Portfolio）来工作。

**v2.x 应当是一条“自动化流水线”**，系统主动将经过加工的信息推送给用户，用户只需进行关键决策。

### 三大原则：

1. **Don't make me calculate (别让我计算)**：仓位、止损价、风险敞口，系统算好直接给建议。
    
2. **Don't make me jump (别让我跳转)**：验证一个信号时，回测数据应该直接显示在信号旁边，而不是去回测页面重输代码。
    
3. **Focus on "Now" (关注当下)**：首页只展示**今天**需要处理的事项，而非所有历史数据。
    

---

## 2. 用户旅程重构 (User Journey Map)

我们将原本割裂的模块重组为完整的**投资生命周期 (Investment Pipeline)**：

|**阶段**|**传统 v1.5 路径**|**v2.x 新路径 (The Flow)**|
|---|---|---|
|**1. 发现**|打开 Hunter -> 运行扫描 -> 面对一堆代码发呆|**Dashboard 推送**："今日发现 3 个高胜率机会，且大盘环境安全。"|
|**2. 验证**|复制代码 -> 打开 Backtest -> 填参数 -> 跑回测 -> 分析|**卡片内嵌验证**：机会卡片上直接显示该策略在该股的历史胜率和回撤。|
|**3. 决策**|打开计算器 -> 算仓位 -> 打开 Portfolio -> 填表单|**一键下单**：点击 "Buy" -> 弹出抽屉 -> 确认系统建议的仓位和止损。|
|**4. 监控**|每天刷 Portfolio -> 肉眼判断是否破位|**智能警报**：系统推送 "平安银行触发止损，建议卖出"。|
|**5. 复盘**|看着盈亏数字，不知道是凭运气还是凭实力|**归因分析**："本月 RSRS 策略贡献 80% 利润，手动操作亏损 10%。"|

---

## 3. 关键界面设计 (Key UI Components)

### A. 首页：指挥塔 (The Command Center)

**功能定义**：Action-Oriented Dashboard。

**关键元素**：

- **Action Stream (行动流)**：一个按优先级排序的待办列表。
    
    - 🔴 [急] 卖出警报：`000001.SZ` 跌破止损线 9.80。
        
    - 🟢 [优] 买入建议：`600519.SH` RSRS 金叉 (胜率 85%)。
        
    - 🔵 [阅] 市场状态：大盘 RSRS 值 1.2 (强势区)。
        
- **Asset Health (资产健康)**：不再只显示总资产，而是显示**风险敞口**。
    
    - "当前仓位 65% (建议 70%)"
        
    - "最大单日风险: -1.5%"
        

### B. 猎场：情境化探索 (Contextual Hunter)

**功能定义**：Decision Support Interface。

**关键交互**：

- **Opportunity Card (机会卡片)**：取代 Excel 表格。
    
    - **左侧**：K线缩略图 + 代码 + 现价。
        
    - **中间**：AI 推荐理由 ("RSRS 突破，且位于 20日均线上方")。
        
    - **右侧 (核心)**：**Evidence (证据)**。直接展示该策略在过去 1 年的**迷你净值曲线**。
        
- **Quick Test (极速验证)**：卡片上有一个 "Run Sim" 按钮，点击后不跳转，直接在卡片下方展开详细回测数据。
    

### C. 交易：无摩擦执行 (Frictionless Execution)

**功能定义**：Smart Order Entry。

**关键交互**：

- **Smart Drawer (智能抽屉)**：在任何页面点击 "Buy"，右侧滑出下单面板。
    
- **Auto-Fill (自动填充)**：
    
    - 系统根据 v1.5 的 `Risk Manager` 逻辑，自动填入：
        
        - `Volume`: 2000 股 (基于 "单笔亏损不超过总资金 1%" 原则)。
            
        - `Stop Loss`: 9.80 元 (基于 ATR 或策略逻辑)。
            
- **Route Switch (路由开关)**：
    
    - `[模拟盘 (999)]` vs `[实盘记录 (001)]`。一键切换发送目标。
        

### D. 持仓：教练模式 (The Coach Portfolio)

**功能定义**：Feedback Loop。

**关键交互**：

- **Tag System (标签系统)**：每笔持仓自动打标。
    
    - `#RSRS策略`, `#低吸`, `#追涨`, `#AI自动单`。
        
- **Performance by Tag (分策略统计)**：
    
    - "你的 `#RSRS策略` 胜率 60%，但 `#追涨` 胜率只有 30%。"
        

---

## 4. 交互细节与微交互 (Micro-interactions)

- **Progressive Disclosure (渐进式披露)**：
    
    - 默认只显示最关键的决策信息（代码、方向、胜率）。
        
    - 鼠标悬停或点击 "Details" 才显示复杂的参数和技术指标。
        
- **Confidence Score (信心指数)**：
    
    - 给每个 Hunter 结果打分 (0-100)。
        
    - 分数 < 60 的直接折叠或置灰，减少噪音。
        
- **LUI Integration (AI 侧边栏)**：
    
    - 左侧常驻 AI 助手。当你在看某只股票时，可以直接问助手："这只股最近有解禁吗？" 助手基于当前上下文回答。
        

---

## 5. 技术准备 (Technical Implications for v1.5)

为了实现 v2.x，v1.5 的开发中需要预埋以下接口：

1. **Linkage (关联性)**：`StrategyResult` 表中需要存储更多元数据（如回测胜率），以便前端直接展示。
    
2. **Calculator (计算器)**：后端需要一个 `TradeCalculatorService`，API 输入股票和价格，输出建议仓位和止损价。
    
3. **Attribution (归因)**：`PortfolioPosition` 表需要增加 `strategy_id` 和 `source` 字段，记录这只股是“谁”让买的。
    

---

### 📝 结语

v1.5 解决了 **"能用" (Functionality)** 的问题。

v2.x 将解决 **"好用" (Usability)** 和 **"爱用" (Desirability)** 的问题。

这份备忘录不仅是 UI 设计指南，也是我们从**“量化开发者”**向**“产品经理”**思维转变的里程碑。
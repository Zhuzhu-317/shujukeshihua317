# 04 统计图形分析

本目录按老师仓库的单一README结构整理，保留此前RAWGraphs研究内容。以下各节汇集原有文档，原链接已转到对应章节；原有待核实和待运行事项继续保留。

## 阅读导航

- [00-研究总览与阅读导航](#raw-00)
- [01-仓库架构与源码调用链](#raw-01)
- [02-数据导入解析与类型推断](#raw-02)
- [03-32种图表的机制与风格图谱](#raw-03)
- [04-字段映射与数据视觉编码](#raw-04)
- [05-视觉风格配色与定制系统](#raw-05)
- [06-渲染导出项目保存与安全边界](#raw-06)
- [07-与01数据逻辑及02案例库的关联](#raw-07)
- [08-示例数据复现实验与风格规则](#raw-08)
- [09-源码证据索引版本与边界](#raw-09)
- [10-信息可视化风格设计卡与交付规范](#raw-10)

<a id="raw-00"></a>

# 研究总览与阅读导航

本研究服务独立的信息图形课程，依据 RAWGraphs 源码研究数据结构、统计计算与可视化风格，并连接课程已有的数据逻辑和案例类型。资料核对日期为 2026-11-24。

## 阅读导航

- [00-研究总览与阅读导航](README.md#raw-00)
- [01-仓库架构与源码调用链](README.md#raw-01)
- [02-数据导入解析与类型推断](README.md#raw-02)
- [03-32种图表的机制与风格图谱](README.md#raw-03)
- [04-字段映射与数据视觉编码](README.md#raw-04)
- [05-视觉风格配色与定制系统](README.md#raw-05)
- [06-渲染导出项目保存与安全边界](README.md#raw-06)
- [07-与01数据逻辑及02案例库的关联](README.md#raw-07)
- [08-示例数据复现实验与风格规则](README.md#raw-08)
- [09-源码证据索引版本与边界](README.md#raw-09)
- [10-信息可视化风格设计卡与交付规范](README.md#raw-10)

## 核心发现

1. 应用的五步流程为加载数据、选择图形、映射字段、定制、导出。数据变量与视觉变量通过映射建立对应。
2. 应用登记 32 种内置图形，31 种来自主图表包，日历热力图来自独立包。图形以本目录第 03 篇中的条目集中分析。
3. 同名图形的具体实现需要追踪字段、聚合、布局及绘制代码。条形图缺少数值映射时可计数，小提琴图的读取实现按分箱计数绘制。
4. 统计图形进入课程作品时，还需要编辑标题、图例、注释、阅读层次与视觉叙事。
5. 配色、排序、画布尺寸及随机种子等参数会影响外观与解读，应随图记录。

## 版本与核实状态

应用源码基准为 `48c3b2e`，主图表库源码基准为 `cade3b6`，核心库源码基准为 `8f03257`。应用锁文件中的依赖发布版本与这些独立仓库提交的对应关系尚待逐文件比对。本研究把应用版本、锁定依赖和补充源码基准分别记录。

已核对应用流程和主图表库的 31 组字段、映射、渲染与选项源码。日历热力图只完成应用注册与依赖核对。第 08 篇完成数值基线复核，浏览器成图、实际导出与项目往返尚待执行。

[应用入口](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/App.js) · [图形注册表](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js) · [版本与边界](README.md#raw-09)

[返回课程主页](../README.md)

---

<a id="raw-01"></a>

# 仓库架构与源码调用链

## 分层职责

| 层面 | 主要入口 | 作用 |
|---|---|---|
| 应用 | App、DataLoader、ChartSelector | 管理数据、图形、映射、视觉选项与项目状态 |
| 数据准备 | useDataLoader、parser、stack | 解析文本、选择 JSON 数组、转换长宽表、调用类型解析 |
| 字段映射 | DataMapping、ChartPreview | 为图形维度选择数据列，并检查必需项与类型 |
| 计算 | worker、rawgraphs-core、图形 mapData | 从输入记录生成布局与绘图所用数据 |
| 图形 | rawgraphs-charts 的 render | 创建比例尺、图形元素、图例与标签 |
| 输出 | Exporter、项目序列化 | 保存 SVG、位图或可恢复项目 |

## 数据调用链

用户输入 → parseAndCheckData → userData → parseDataset → dataset 与 dataTypes → 图形 dimensions 与 mapping → mapData → mappedData → renderToDOM → SVG 节点 → Exporter。

应用在后台工作线程中调用核心库的 chart 工厂，并取得映射数据。预览组件把映射数据与视觉选项交给渲染过程。工作线程承担计算组织，图形预览涉及 DOM。

## 统一图形契约

图形模块通常组合 metadata、dimensions、mapData、render、visualOptions 与 styles。metadata 描述图形身份，dimensions 描述输入角色，mapData 改变数据组织，render 决定具体绘制。

阅读某个图形时，先看 dimensions 和 mapping，再看 render 及 visualOptions。统计计算也可能出现在 render，例如箱线图的四分位数和小提琴图的分箱。

## 状态更新与依赖

App 将 currentChart、mapping、visualOptions 与 rawViz 存入界面状态。切换图形时会清理映射和重设选项，列结构改变也可能清理映射。复现时需要保存数据列名、字段类型与配置。

## 课程含义

架构可以用于解释“数据—计算—编码—风格”的生成顺序。研究源码时，以具体变量沿调用链追踪，可避免仅根据最终外观推断统计方法。

[应用入口](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/App.js) · [工作线程](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/worker/worker.js) · [预览组件](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartPreview/ChartPreview.js) · [箱线图模块](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/boxplot.js)

[研究导航](README.md#raw-00)

---

<a id="raw-02"></a>

# 数据导入解析与类型推断

## 输入与解析阶段

数据加载组件组织粘贴、文件、URL、SPARQL 和项目等入口。parser 依次尝试 SPARQL 标记、JSON 与分隔文本解析。数据源入口、格式解析与字段类型推断属于连续的不同阶段。

文本解析可采用指定分隔符，也可遍历候选分隔符并按解析列数选择候选。列数检查只是启发式，需要用实际表头和记录结构确认。

## JSON 规范化

JSON 路径选择后，数组元素经 normalizeJsonArray 形成行对象。标量可变为 value 字段，数组内标量可能拼接成字符串，嵌套对象中的标量可转换成带点号的列名。应保留原始 JSON，以便追踪扁平化后的信息对应。

## 字段类型

useDataLoader 把表格及 locale、decimal、group 等配置交给核心库 parseDataset。用户能够覆盖字段类型。数字外观可能对应编号，年份可能用作分类或时间，因此列的统计语义需要人工确定。

| 现象 | 检查内容 |
|---|---|
| 0012 等编号 | 保留前导零，明确其类别身份 |
| 1,234 或 1.234 | 确认千分位、小数分隔和地区约定 |
| 2020 等年份 | 根据任务明确时间或类别用途 |
| 空串、0、NA | 记录各自含义和处理规则 |
| 日期解析错误 | 确认格式、地区设置、时间粒度与时区 |

## 长宽表变换

stackData 保留选定标识列，把其他列变为 column 和 value 两列。例如，地区、2024、2025 可转换为地区、column、value 三列。这一步改变记录单位，之后要重新核对重复项、数值类型及聚合。

## 数据替换

替换模式检查解析错误、缺列和类型转换结果，并可保留已有类型覆盖。使用新数据复现旧图时，核对字段名和记录范围，再检查已有映射。

## 课程记录要求

记录来源、获取日期、每行含义、字段字典、清洗规则、变换前后行数与缺失值处理。RAWGraphs 的类型识别是技术准备，研究问题、单位和分母仍由课程分析确定。

[加载状态与类型调用](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoader.js) · [格式解析](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoaderUtils/parser.js) · [长宽表变换](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoaderUtils/stack.js)

[研究导航](README.md#raw-00)

---

<a id="raw-03"></a>

# 32种图表的机制与风格图谱

应用注册顺序决定本篇条目顺序。每条给出数据条件、计算机制、视觉表达和设计检查。字段契约来自主图表库固定提交，应用锁定依赖的对应性见第 09 篇。叙事用途与误读分析为课程研究归纳。

## 任务检索

| 主要任务 | 可优先查阅 |
|---|---|
| 数量与构成 | 条形、多集合条形、堆叠条形、饼图、矩形树图 |
| 时间与变化 | 折线、甘特、排名变化、坡度、地平线、流图、日历热力 |
| 分布与聚集 | 蜂群、箱线、小提琴、密度等值线、六边形分箱、凸包 |
| 连接与流向 | 冲积、桑基、弦图、弧线 |
| 层级与路径 | 树状、径向树状、圆形打包、旭日、沃罗诺伊树图 |
| 多变量与邻近 | 气泡、矩阵、平行坐标、雷达、沃罗诺伊图 |

任务存在交叉，选择图形需要结合记录单位和阅读目的。

## 01 冲积图（alluvialdiagram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| steps | number, date, string | 必需 | 多列，至少 2 列 |
| size | number | 可选或未声明必需 | 单列 |

### 统计与布局机制

将相邻步骤字段两两分组，汇总各连接的权重，缺少权重时按记录数计数，映射结果保留正值。

### 视觉表达与阅读

步骤位置、流带宽度与类别色彩组织阶段转移。

### 风格与误读检查

步骤列顺序决定阅读方向，相同名称在不同步骤中有独立身份。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/alluvialdiagram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/alluvialdiagram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/alluvialdiagram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/alluvialdiagram/visualOptions.js)

## 02 弧线图（arcdiagram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| source | number, date, string | 必需 | 单列 |
| target | number, date, string | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |

### 统计与布局机制

将源与目标组合为连接数据，再布局节点和弧线。

### 视觉表达与阅读

节点位置与弧线建立连接，线宽可承载量值。

### 风格与误读检查

节点排序改变交叉与聚集印象，弧高需要依据布局规则解释。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/arcdiagram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/arcdiagram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/arcdiagram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/arcdiagram/visualOptions.js)

## 03 条形图（barchart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| bars | number, string, date | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, string, date | 可选或未声明必需 | 单列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

按类别和可选系列分组，映射 Size 时聚合数值，未映射时使用记录数。

### 视觉表达与阅读

共同基线上的条长支持比较，方向和排序影响扫读。

### 风格与误读检查

记录分组键、聚合方式和零基线，平均值与总量具有不同意义。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/visualOptions.js)

## 04 多集合条形图（barchartmultiset）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| groups | number, string, date | 必需 | 单列 |
| bars | number | 必需 | 多列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

Sets 与多个数值列形成比较结构，数值列声明默认求和。

### 视觉表达与阅读

多个条形集合呈现相同对象的不同指标。

### 风格与误读检查

比较前核对单位和量纲，多个数值列在当前字段契约中直接映射。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartmultiset/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartmultiset/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartmultiset/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartmultiset/visualOptions.js)

## 05 堆叠条形图（barchartstacked）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| stacks | number, string, date | 必需 | 单列 |
| bars | number | 必需 | 多列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

按 stacks 及系列整理多列数值，聚合后形成堆叠区间。

### 视觉表达与阅读

总高度表达总量，色块和局部厚度表达组成。

### 风格与误读检查

中间色块缺少共同基线，百分比解释需要先确认是否归一化。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartstacked/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartstacked/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartstacked/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchartstacked/visualOptions.js)

## 06 蜂群图（beeswarm）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| xValue | number, date | 必需 | 单列 |
| size | number, date | 可选或未声明必需 | 单列 |
| color | number, string, date | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

保留观测值，利用 forceX、forceY、forceCollide 排列点并减少重叠。

### 视觉表达与阅读

主轴位置表达变量，避碰方向帮助显示局部聚集。

### 风格与误读检查

避碰产生的位移属于布局，点大小变化也会改变挤压效果。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/beeswarm/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/beeswarm/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/beeswarm/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/beeswarm/visualOptions.js)

## 07 箱线图（boxplot）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| value | number | 必需 | 单列 |
| group | number, date, string | 可选或未声明必需 | 单列 |
| color | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

按组排序值并计算四分位数，依据 IQR 和倍数选项计算须线及离群点。

### 视觉表达与阅读

箱体展示 Q1—Q3，中线展示中位数，须线与离群点补充范围。

### 风格与误读检查

当前读取实现使用截在样本极值内的理论界限作须线端点，需记录倍数与样本量。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/visualOptions.js)

## 08 气泡图（bubblechart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number, date | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| connectedBy | number, date | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |
| series | number, date, string | 可选或未声明必需 | 单列 |

### 统计与布局机制

按映射读取 x、y、大小和类别，位置使用数值或时间比例尺。

### 视觉表达与阅读

二维位置表达变量，大小与颜色可增加维度，标签辨识对象。

### 风格与误读检查

核实圆面积的数值映射，遮挡及大小跨度会影响可见性。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bubblechart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bubblechart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bubblechart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bubblechart/visualOptions.js)

## 09 排名变化图（bumpchart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |
| rank | number | 可选或未声明必需 | 单列 |
| streams | number, string, date | 可选或未声明必需 | 单列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

以 x 组织横向顺序，并结合 Ranking、Size、Streams 等字段生成序列布局。

### 视觉表达与阅读

纵向位置、曲线、宽度和颜色帮助追踪对象。

### 风格与误读检查

数据契约仅把 x 标为必需，实际比较应明确排名来源、并列规则与值的含义。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bumpchart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bumpchart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bumpchart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/bumpchart/visualOptions.js)

## 10 日历热力图（calendarHeatmap）

### 数据与核实状态

应用从独立日历热力图包登记该图形。该包锁定版本已核实，独立源码与实际聚合规则待复核。

### 视觉表达与阅读

一般以日历位置和颜色组织日期数值，此句为设计原理归纳。

### 设计检查

具体字段标识、周起点、重复日期聚合与缺日处理仍需独立包源码或运行验证。

[应用登记](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js) · [锁定依赖](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/package-lock.json)

## 11 弦图（chorddiagram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| source | number, date, string | 必需 | 单列 |
| target | number, date, string | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |

### 统计与布局机制

将源、目标和连接权重映射为关系数据，绘制环形节点与带状连接。

### 视觉表达与阅读

弧段、连接带和颜色表达关系结构，带方向需看具体选项。

### 风格与误读检查

方向、节点排序、带宽与边总量须分别解释，避免交叉遮挡。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/chorddiagram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/chorddiagram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/chorddiagram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/chorddiagram/visualOptions.js)

## 12 圆形打包图（circlepacking）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| hierarchy | number, date, string | 必需 | 多列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

由有序层级字段建立分组与包含结构，叶节点数值支持圆形布局。

### 视觉表达与阅读

圆的包含关系表达层级，大小表达权重，颜色与标签辅助识别。

### 风格与误读检查

父圆含空隙，面积读取需结合叶节点与布局规则。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circlepacking/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circlepacking/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circlepacking/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circlepacking/visualOptions.js)

## 13 径向树状图（circularDendrogram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| hierarchy | number, date, string | 必需 | 多列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

用层级字段建立树结构，渲染提供 tree 与 cluster 布局。

### 视觉表达与阅读

径向层次、分支和标签体现从属。

### 风格与误读检查

几何距离与枝长是否表示度量需要图形定义与选项支持。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circularDendrogram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circularDendrogram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circularDendrogram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/circularDendrogram/visualOptions.js)

## 14 密度等值线图（contourPlot）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number, date | 必需 | 单列 |

### 统计与布局机制

读取二维点，经密度估计与等值线布局形成区域，带宽是影响轮廓的参数。

### 视觉表达与阅读

嵌套轮廓及色阶表达相对密度。

### 风格与误读检查

带宽和阈值影响聚集外观，图形密度与概率解释需明确尺度。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/contourPlot/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/contourPlot/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/contourPlot/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/contourPlot/visualOptions.js)

## 15 凸包图（convexHull）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number, date | 必需 | 单列 |
| group | string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

对二维点按组组织，使用 polygonHull 形成包含组内点的边界。

### 视觉表达与阅读

位置表达变量，多边形轮廓及颜色标识组。

### 风格与误读检查

边界对极端点敏感，区域面积不能直接解释为样本数。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/convexHull/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/convexHull/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/convexHull/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/convexHull/visualOptions.js)

## 16 树状图（dendrogram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| hierarchy | number, date, string | 必需 | 多列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

有序层级字段生成树，渲染提供 tree 与 cluster 等布局。

### 视觉表达与阅读

分支、深度及标签表达类别路径。

### 风格与误读检查

当前输入是显式层级字段，聚类统计产生的树需要另列计算依据。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/dendrogram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/dendrogram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/dendrogram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/dendrogram/visualOptions.js)

## 17 甘特图（ganttChart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| startDate | number, date | 必需 | 单列 |
| endDate | number, date | 必需 | 单列 |
| group | string | 可选或未声明必需 | 单列 |
| color | string | 可选或未声明必需 | 单列 |

### 统计与布局机制

映射开始和结束字段，整理组并绘制时间或数值区间。

### 视觉表达与阅读

位置表达起止，条长表达持续范围，分组与颜色帮助追踪。

### 风格与误读检查

检查开始小于等于结束，区间重叠、时区和粒度需说明。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/ganttChart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/ganttChart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/ganttChart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/ganttChart/visualOptions.js)

## 18 六边形分箱图（hexagonalBinning）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number, date | 必需 | 单列 |

### 统计与布局机制

先将 x、y 映射到绘图区，再按六边形网格分箱计数。

### 视觉表达与阅读

色彩表达箱内点数，可选大小权重和原始点展示。

### 风格与误读检查

网格直径与画布尺寸会影响分箱，应固定尺寸比较。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/hexagonalBinning/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/hexagonalBinning/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/hexagonalBinning/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/hexagonalBinning/visualOptions.js)

## 19 地平线图（horizongraph）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number | 可选或未声明必需 | 单列 |
| group | number, date, string | 可选或未声明必需 | 单列 |

### 统计与布局机制

按 x 与组整理序列，使用分层与折叠的面积表达压缩纵向空间。

### 视觉表达与阅读

色阶、带层与正负方向表达幅度。

### 风格与误读检查

记录基准和带数，颜色级别与实际量值的关系需要图例。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/horizongraph/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/horizongraph/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/horizongraph/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/horizongraph/visualOptions.js)

## 20 折线图（linechart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number, date | 必需 | 单列 |
| lines | number, string, date | 可选或未声明必需 | 单列 |
| color | number, string, date | 可选或未声明必需 | 单列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

按 x、Lines 及可选系列分组聚合，按横轴顺序生成线条。

### 视觉表达与阅读

横纵位置表达变量，线与色彩帮助追踪系列。

### 风格与误读检查

同一横轴值的聚合会影响曲线，缺失与插值规则要记录。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/linechart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/linechart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/linechart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/linechart/visualOptions.js)

## 21 矩阵图（matrixplot）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date, string | 必需 | 单列 |
| y | number, date, string | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

对 x 与 y 的组合分组，计算可选的大小、颜色与标签。

### 视觉表达与阅读

行列位置表达类别配对，单元大小与颜色表达量值。

### 风格与误读检查

区分缺失组合、零值和未显示项，固定行列排序后比较。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/matrixplot/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/matrixplot/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/matrixplot/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/matrixplot/visualOptions.js)

## 22 平行坐标图（parallelcoordinates）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| dimensions | number, date | 必需 | 多列 |
| color | number, date, string | 可选或未声明必需 | 单列 |

### 统计与布局机制

读取多个数值或日期维度，各轴建立比例尺，同一记录以折线连接。

### 视觉表达与阅读

每条折线代表记录，跨轴走势支持多变量观察。

### 风格与误读检查

轴顺序、独立量程和过绘影响判断，记录标准化方式。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/parallelcoordinates/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/parallelcoordinates/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/parallelcoordinates/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/parallelcoordinates/visualOptions.js)

## 23 饼图（piechart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| arcs | number | 必需 | 多列 |
| series | number, date, string | 可选或未声明必需 | 单列 |

### 统计与布局机制

将多个数值列映射到 Arcs，聚合并计算饼图扇区。

### 视觉表达与阅读

角度与扇区面积表达组成，颜色区分类别。

### 风格与误读检查

确认共同整体与分母，负值、类别过多及相近份额需谨慎处理。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/piechart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/piechart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/piechart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/piechart/visualOptions.js)

## 24 雷达图（radarchart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| axes | number | 必需 | 多列，至少 3 列 |
| color | number, string, date | 可选或未声明必需 | 单列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

Spokes 要求至少三个数值列，结合系列和聚合组织多轴比较。

### 视觉表达与阅读

径向位置形成轮廓，多边形与颜色表现对象。

### 风格与误读检查

轴顺序与量程会改变形状，不能直接把多边形面积视为综合评分。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/radarchart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/radarchart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/radarchart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/radarchart/visualOptions.js)

## 25 桑基图（sankeydiagram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| source | number, date, string | 必需 | 单列 |
| target | number, date, string | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |

### 统计与布局机制

按源与目标汇总权重，读取实现未映射 Size 时每个连接组赋值 1。

### 视觉表达与阅读

节点与带宽组织流量，布局连接呈现转移。

### 风格与误读检查

确认省略权重的含义、节点身份与方向，数据守恒需独立检查。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sankeydiagram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sankeydiagram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sankeydiagram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sankeydiagram/visualOptions.js)

## 26 坡度图（slopechart）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| source | number, date | 必需 | 单列 |
| target | number, date | 必需 | 单列 |
| name | number, date, string | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

将每条记录的 Source 与 Target 映射为两端位置，连线连接同一对象。

### 视觉表达与阅读

端点高低、方向和斜率帮助比较变化，标签支持对象辨识。

### 风格与误读检查

统一两端量程，区分绝对变化、比例变化与名次变化。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/slopechart/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/slopechart/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/slopechart/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/slopechart/visualOptions.js)

## 27 流图（streamgraph）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| size | number | 可选或未声明必需 | 单列 |
| streams | number, string, date | 可选或未声明必需 | 单列 |
| series | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

按 x、Streams 与系列组织并聚合数值，生成随横轴变化的堆叠带。

### 视觉表达与阅读

流带厚度、颜色与整体形态呈现组成变化。

### 风格与误读检查

记录基线偏移与排序，带所在高度和带自身厚度分别读取。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/streamgraph/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/streamgraph/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/streamgraph/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/streamgraph/visualOptions.js)

## 28 旭日图（sunburst）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| hierarchy | number, date, string | 必需 | 多列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

用层级字段建立 hierarchy 并汇总叶值，partition 将角度和半径划分为层次。

### 视觉表达与阅读

环层表达深度，角跨度体现权重分配，色彩标识类别。

### 风格与误读检查

不同半径层的面积具有几何差异，份额比较需说明层级及父级。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sunburst/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sunburst/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sunburst/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sunburst/visualOptions.js)

## 29 矩形树图（treemap）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| hierarchy | number, date, string | 必需 | 多列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

hierarchy 汇总叶值，treemap 根据 tiling、padding 等选项划分矩形。

### 视觉表达与阅读

嵌套表达包含，面积表达权重，标签和颜色标识对象。

### 风格与误读检查

布局与留白会改变外观，小矩形的标签裁切需要核验。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/treemap/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/treemap/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/treemap/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/treemap/visualOptions.js)

## 30 小提琴图（violinplot）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| value | number | 必需 | 单列 |
| group | number, date, string | 可选或未声明必需 | 单列 |
| color | number, string, date | 可选或未声明必需 | 单列 |

### 统计与布局机制

当前实现使用 d3.bin 按 binsNumber 分箱，以计数生成左右对称轮廓并应用插值。

### 视觉表达与阅读

纵向位置表达值，横向宽度表达箱内计数，可叠加观测点。

### 风格与误读检查

本实现需要记录分箱数及插值，跨组样本量不同会影响宽度。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/violinplot/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/violinplot/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/violinplot/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/violinplot/visualOptions.js)

## 31 沃罗诺伊图（voronoidiagram）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| x | number, date | 必需 | 单列 |
| y | number, date | 必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

将二维点映射到画面后用 Delaunay 与 Voronoi 划分最近点区域。

### 视觉表达与阅读

位置表达数据，单元边界体现画面中的邻近结构。

### 风格与误读检查

独立缩放横纵轴会改变距离关系，单元面积需要结合点分布解释。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoidiagram/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoidiagram/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoidiagram/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoidiagram/visualOptions.js)

## 32 沃罗诺伊树图（voronoitreemap）

### 数据与字段

| 角色 ID | 接受类型 | 契约要求 | 列数 |
|---|---|---|---|
| hierarchy | number, date, string | 必需 | 多列 |
| size | number | 可选或未声明必需 | 单列 |
| color | number, date, string | 可选或未声明必需 | 单列 |
| label | number, date, string | 可选或未声明必需 | 多列 |

### 统计与布局机制

从层级叶值建立权重结构，用加权空间分割生成多边形，代码提供 seed 和裁剪形状。

### 视觉表达与阅读

多边形面积、嵌套与颜色表现权重和层级。

### 风格与误读检查

固定种子和画布以支持复现，非规则形状提高面积比较难度。

### 源码依据

[字段契约](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoitreemap/dimensions.js) · [映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoitreemap/mapping.js) · [渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoitreemap/render.js) · [视觉选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoitreemap/visualOptions.js)

## 比较阅读规则

同一数据可选择多种图形，比较时固定记录范围、单位、聚合和颜色含义，再观察外观变化。先判断位置、长度、面积、角度或流带宽度承担什么变量，再讨论风格。

[研究导航](README.md#raw-00) · [示例与实验](README.md#raw-08)

---

<a id="raw-04"></a>

# 字段映射与数据视觉编码

## 三种角色

数据字段是原表中的列，如类别、集数、日期。图形维度是 dimensions 声明的输入角色，如 Bars、Size、Hierarchy。视觉变量是最终画面的位置、长度、面积、颜色及线宽。映射把前两者连接，绘制过程决定第三者。

## 映射检查

预览组件依次检查必需维度、多个字段的最低数量和类型匹配。核心库还检查多列要求与类型。完成技术校验后，仍应核对量纲、分母、重复记录与统计语义。

## 聚合与记录单位

条形图的 Bars 对应分组键，Size 对应数量。以节目清单为例，Size 留空得到分组内节目记录数，Size 映射 Episodes 并求和得到集数合计。求平均则需要明确平均的对象与权重。

冲积图的 Steps 至少映射两列，相邻步骤分别形成流量汇总。桑基图使用 Source node、Target node 与可选 Size，读取的当前实现省略 Size 时每个连接组取 1。复用同一张表时需记录具体默认。

## 编码检查表

| 视觉变量 | 适合表达 | 要记录的条件 |
|---|---|---|
| 位置 | 数值、时间、类别顺序 | 比例尺、范围、轴方向、排序 |
| 长度 | 数量和持续区间 | 共同基线、零点、单位 |
| 面积 | 量值或构成 | 面积映射、最小尺寸、遮挡 |
| 角度 | 份额 | 共同整体、层级和分母 |
| 色彩 | 类别、顺序量、偏离基准 | 色阶类型、域、图例、反向设置 |
| 线与带 | 关系、过程、流量 | 连线语义、方向、宽度尺度 |
| 符号和文字 | 对象身份、注释 | 符号字典、标签与数据对应 |

## 层级字段

Hierarchy 支持多列时，列顺序用于构建层级路径。应记录根、分支和叶的分类规则，核对叶值是否重复汇总。树状布局的枝长与统计距离需分别确认。

## 课程应用

A 部分记录实体、字段与关系，B 部分记录聚合与布局，C 部分记录视觉变量，D 部分记录图例、标签和来源，E 部分记录阅读入口与解释顺序。

[预览校验](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartPreview/ChartPreview.js) · [核心映射](https://github.com/rawgraphs/rawgraphs-core/blob/8f03257fa7abe7d6f87e6e9d0a818612fb421eae/src/mapping.js) · [条形映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/mapping.js) · [桑基映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sankeydiagram/mapping.js)

[研究导航](README.md#raw-00)

---

<a id="raw-05"></a>

# 视觉风格配色与定制系统

## 参数如何形成风格

visualOptions 定义每种图形可编辑的参数，应用根据配置生成输入控件。共同参数涉及画布、边距、颜色、标签与图例，具体图形另有专属参数。最终风格还受到数据范围、对象数量与长宽比影响。

## 配色机制

ColorScaleWrapper 根据是否存在颜色映射数据切换默认颜色和数据色阶配置。颜色界面提供域重置、反转与锁定等操作。需要保存域和范围，保证多图之间同一颜色含义一致。

| 变量意义 | 设计处理 | 检查点 |
|---|---|---|
| 无序类别 | 区分色，必要时辅以标签或形状 | 同类颜色一致、类别数量适中 |
| 连续量 | 连续色阶 | 顺序清楚、图例单位完整 |
| 相对基准的偏离 | 以明确基准组织色彩 | 基准有数据意义 |
| 关注对象 | 降低背景强调、突出目标 | 重点由研究问题支持 |

以上为课程设计原则。某张图是否提供相应尺度，需看它的具体配置。

## 三组可复用的风格控制

1. 比较型版面。统一基线、量程和分类色，控制标签数量，提供关键值与单位。
2. 分布型版面。显示样本范围和分组，记录分箱、IQR 倍数或密度带宽，适当保留观测点。
3. 层级与关系版面。明确起点、顺序和类别，控制节点间距、连线交叉、标签遮挡与颜色语义。

## 专属参数案例

箱线图的 iqrMultiplier 影响须线和离群点，小提琴图的 binsNumber 与 interpolation 影响轮廓。矩形树图的 tiling 与 padding 改变分割形态，沃罗诺伊树图的 seed 与裁剪形状影响布局。参数调整应保留可复现记录。

## 图例与标签

标题、单位、刻度、图例和注释帮助解释数据。缩小画布、导出位图或进入矢量编辑器后，应重新检查标签裁切、文字对比和图例是否完整。

## 风格对照记录

每次只改变一组条件，例如固定数据与映射后比较颜色域、标签密度或图形间距。记录改变前后的配置，并说明变化影响了可读性、视觉强调还是统计含义。

[颜色包装组件](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartOptions/ChartOptionTypes/ChartOptionColorScaleWrapper.js) · [域操作](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartOptions/ChartOptionTypes/ColorScaleUtils.js) · [箱线选项](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/visualOptions.js) · [沃罗诺伊树图渲染](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoitreemap/render.js)

[研究导航](README.md#raw-00)

---

<a id="raw-06"></a>

# 渲染导出项目保存与安全边界

## 从映射到画面

worker 调用图形与核心库计算映射数据，预览组件通过 renderToDOM 生成可导出的 SVG。视觉选项更新使用 200 毫秒防抖，必需字段不足或类型不匹配时清空预览并提供提示。

## 输出格式

| 格式 | 代码路径与用途 | 交付时检查 |
|---|---|---|
| SVG | XMLSerializer 序列化图形节点 | 字体、裁切、图例、描边与编辑器兼容 |
| PNG | SVG 经 Image 和 Canvas 转为位图 | 像素尺寸、透明度与细线清晰度 |
| JPG | 同样经 Canvas 导出 JPEG | 背景颜色、文字与压缩影响 |
| rawgraphs | exportProject 后 JSON 序列化下载 | 数据、映射、视觉选项与依赖来源 |

位图 Canvas 尺寸由 SVG 的 clientWidth、clientHeight 设置。界面尺寸和导出分辨率应实际核对，源码并未在该处提供固定高倍分辨率策略。

## 项目保存与恢复

App 的当前路径从 rawgraphs-core 导入 serializeProject、deserializeProject，并把用户输入、数据、解析配置、图形、mapping、visualOptions 及自定义图形交给保存过程。项目恢复重新设置加载状态、图形和配置。

仓库还保留 import_export_v1.js 与 import_export_v1.1.js。当前入口的实际导入决定调用路径，研究旧文件时应注明历史用途。

## 数据与网络边界

README 表述数据由浏览器处理。源码另外存在 URL 和 SPARQL 数据入口、远程项目读取、自定义图形加载以及浏览统计事件。处理位置与网络活动应分别说明。

gaEvents 中可见图形渲染和导出事件，参数涉及图形 ID 与格式。该阅读范围没有显示它们携带原始表格值，仍需结合部署脚本和网络检查判断完整行为。

## 自定义图形

useSafeCustomCharts 为 URL、npm、文件与项目内自定义图形设置用户确认过程。当前 URL 白名单包含 localhost，npm 默认仍需确认。确认界面属于交互边界，不能据此认定加载代码已经过隔离或安全审计。

## 验证状态

本篇完成静态源码核对。SVG 导入设计软件、PNG/JPG 尺寸、字体表现和项目往返恢复需用实际成图检查，结果补入第 08 篇。

[工作线程](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/worker/worker.js) · [预览](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartPreview/ChartPreview.js) · [导出](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/Exporter/Exporter.js) · [当前入口](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/App.js) · [自定义加载确认](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useSafeCustomCharts.js) · [统计事件](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/gaEvents.js)

[研究导航](README.md#raw-00)

---

<a id="raw-07"></a>

# 与01数据逻辑及02案例库的关联

本篇为课程研究归纳。图形是否适用取决于数据结构、统计目的和作品中的阅读任务。

## 与五种数据逻辑的对应

| 数据逻辑 | 相关图形 | 需要核对 |
|---|---|---|
| [编年型](../01-信息图形数据逻辑/01-编年型信息图形概念定义.md) | 折线、甘特、坡度、排名变化、日历热力 | 时间粒度、先后、区间与缺失 |
| [层级型](../01-信息图形数据逻辑/02-层级型信息图形概念定义.md) | 树状、径向树状、旭日、圆形打包、矩形树、沃罗诺伊树 | 层级列顺序、叶值与汇总 |
| [编目型](../01-信息图形数据逻辑/03-编目型信息图形概念定义.md) | 条形、矩阵、气泡、平行坐标 | 条目范围、属性、排序与单位 |
| [字母表型](../01-信息图形数据逻辑/04-字母表型信息图形概念定义.md) | 图形内的命名、符号与索引辅助 | 课程该类型的定义仍待进一步校订 |
| [地理图形型](../01-信息图形数据逻辑/05-地理图形信息图形概念定义.md) | 二维点布局可辅助空间任务 | 坐标、投影与距离语义，当前注册表未列独立地理地图图形 |

## 与17类案例的对应

| 案例类型 | 可采用的图形或方法 | 在作品中的作用 |
|---|---|---|
| 混合型 | 条形、饼图、矩阵、关系图等组合 | 多指标证据 |
| 文字型／列表型 | 小条形、矩阵、数值标签 | 辅助查阅和事实比较 |
| 时间轴型 | 甘特、折线、排名变化 | 时间证据与变化 |
| 示意图型 | 图表作为数量补充 | 配料、尺寸或步骤统计 |
| 流程图型 | 冲积与桑基可用于实际流量 | 明确连接及权重，操作流程另需节点语义 |
| 比较型 | 条形、坡度、箱线、气泡 | 对象间差异 |
| 地标型 | 点、连接及统计图与底图配合 | 位置和路线语境 |
| 图片式 | 导出图形与图片排版 | 照片提供情境，图形提供度量 |
| 层级型 | 树、旭日、矩形树 | 从属与组成 |
| 单一图表型 | 依据主任务选择一个主图 | 集中回答问题 |
| 文字可视化 | 标签、圆形与关系 | 名称及对象识别 |
| 解剖剖析型 | 统计图补充部件数值 | 与结构图相互指引 |
| 可视化简历型 | 时间区间、条形等 | 经历和能力资料 |
| 手稿型 | 先固定数据编码再进行风格加工 | 保持度量含义 |
| 编年与编目混合型 | 时间图与对象图共享分类 | 双入口阅读 |
| 3D | 可导出的二维图形提供基础 | 后续空间表现需保持数量含义 |
| 词云图 | 当前32种内置注册表未登记 | 另行核对词频工具与文字布局 |

[17类案例索引](../02-信息图形类型案例/00-案例索引/README.md) · [应用图形注册](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js)

## A—E 回写方式

A 写记录单位和关系，B 写统计计算，C 写视觉变量，D 写概览、分组、图例和来源，E 写读者进入、比较、解释与结论顺序。完成案例后将可复用方法回写课程原文，并链接到案例与本研究具体章节。

[研究导航](README.md#raw-00)

---

<a id="raw-08"></a>

# 示例数据复现实验与风格规则

## 验证层级

本篇已完成固定源码样例的独立数值汇总，以及箱线统计规则的 Node.js 数值复核。RAWGraphs 浏览器成图、SVG/位图导出和项目往返尚待执行。下列界面步骤是可复现方案，结果状态逐项标明。

## 实验1　Netflix类型计数与集数总和

### 输入

采用应用固定提交中的 Bar chart - Netflix Original Series.tsv。样例共 109 行数据，字段包含 Genere、Title、Subgenre、Premiere_Year、Status、Seasons、Episodes、IMDB_Rating。Genere 沿用源文件拼写。样例是历史教学材料。

### 已执行的数值基线

以 Genere 分组，分别统计行数与 Episodes 总和。

| Genere | 记录数 | Episodes合计 |
|---|---|---|
| Animation | 1 | 13 |
| Comedy | 25 | 416 |
| Docu-Series | 14 | 100 |
| Drama | 18 | 362 |
| Family Animation | 32 | 726 |
| Family Live Action | 3 | 49 |
| Foreign Language | 8 | 94 |
| Marvel | 4 | 65 |
| Reality | 2 | 20 |
| Talk Show | 2 | 103 |

样例有 3 条 IMDB_Rating 为 0。评分分析前需要核实 0 的原始意义，并记录处理决定。

### RAWGraphs复现步骤

1. 导入样例，确认制表符和字段类型。
2. 选条形图，Bars 映射 Genere，Size 留空。读取当前 mapping 实现，预期按类别计数。
3. 将 Size 改为 Episodes，并选择 sum。预期得到表中第三列。
4. 固定画布、颜色含义和排序，比较“节目记录数”和“集数合计”的不同任务。
5. 保存项目与 SVG，重新打开项目并核对映射。

预期值已数值复核，界面输出待执行。

[样例数据](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/public/sample-datasets/Bar%20chart%20-%20Netflix%20Original%20Series.tsv) · [条形映射](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/mapping.js)

## 实验2　箱线须线端点

使用自拟教学数据 10、20、30、40、100，采用线性插值分位数及倍数 k=1.5。Node.js 独立复核得到 Q1=20、中位数=30、Q3=40、IQR=20，须线端点为 10 与 70，100 超出上界。

读取的图表实现将理论上界截在样本最大值内，因此此例上端点为 70。课程展示时要说明它与观测值端点的区别。实际 RAWGraphs 图像待运行确认。

[箱线实现](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/render.js)

## 实验3　小提琴轮廓对照

固定输入记录与 y 轴范围，分别改变 binsNumber、interpolation 和 showDots。记录分箱区间、每箱计数、宽度和离群观测表现。当前代码使用分箱计数生成轮廓，需观察样本量差异的影响。

状态为源码规则已核对、成图实验待执行。

[小提琴实现](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/violinplot/render.js)

## 实验4　随机布局的可复现性

选择沃罗诺伊树图，固定数据、层级顺序、尺寸、seed 与裁剪形状。连续生成两次，再只改变 seed，对照位置、面积和标签。保存所有配置并记录误差。

状态为 seed 参数与伪随机生成器源码已核对、成图实验待执行。

[随机布局](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/voronoitreemap/render.js)

## 风格规则归纳

1. 同一研究问题的风格对照，固定数据、聚合与量程。
2. 改变分箱、带宽、堆叠或基线可能改变统计外观，需与单纯换色分别记录。
3. 多图比较采用相同颜色语义和尺度域。
4. 保存数据版本、映射、视觉参数与导出尺寸，才能追溯结果。

[研究导航](README.md#raw-00)

---

<a id="raw-09"></a>

# 源码证据索引版本与边界

核对日期为 2026-11-24。本目录使用固定提交链接，方便以后查证与更新。

## 源码基准

| 材料 | 固定提交或版本 | 用途 |
|---|---|---|
| rawgraphs-app | 48c3b2ee9ff412721a2c99c7bed38e734b3fa534 | 应用工作流、注册、导入、界面与导出 |
| rawgraphs-charts | cade3b6e1b4d35d365440b1365d51b0dd48c9bce | 31种主图形的字段、映射、绘制与选项 |
| rawgraphs-core | 8f03257fa7abe7d6f87e6e9d0a818612fb421eae | 共享映射与校验分析 |
| app package.json | 标注 2.1.0 | 应用包标识 |
| app package-lock.json | charts 1.1.0、core 1.0.0、calendar-heatmap 1.0.0-beta.8 | 发布依赖锁定记录 |

独立仓库提交与锁文件发布包是否逐文件一致尚待核对。图表算法结论指向上表对应源码，应用实际安装行为应再核对锁定包。日历热力图独立源码尚未取得，因此其具体字段和聚合保持待核实。

## 应用证据索引

| 内容 | 证据 |
|---|---|
| 五步流程、状态、当前项目序列化调用 | [App](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/App.js) |
| 32种内置图形 | [charts](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js) |
| 依赖声明与锁定 | [package](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/package.json)、[lock](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/package-lock.json) |
| 数据加载与类型调用 | [useDataLoader](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoader.js) |
| 分隔文本与JSON规范化 | [parser](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoaderUtils/parser.js) |
| 长宽表变换 | [stack](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoaderUtils/stack.js) |
| 后台计算 | [worker](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/worker/worker.js) |
| 预览字段校验 | [ChartPreview](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartPreview/ChartPreview.js) |
| 导出格式与Canvas尺寸 | [Exporter](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/Exporter/Exporter.js) |
| 颜色映射与域操作 | [Wrapper](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartOptions/ChartOptionTypes/ChartOptionColorScaleWrapper.js)、[Utils](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartOptions/ChartOptionTypes/ColorScaleUtils.js) |
| 自定义代码确认流程 | [useSafeCustomCharts](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useSafeCustomCharts.js) |
| 图形浏览统计 | [gaEvents](https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/gaEvents.js) |

## 图形与核心证据

主图表库每种图形采用 src/图形ID/dimensions.js、mapping.js、render.js、visualOptions.js。第 03 篇逐条链接到四份文件，合计核对 31 组、124 份文件。数据计算可能跨 mapping 与 render，读取结论需包含两者。

[共享映射与校验](https://github.com/rawgraphs/rawgraphs-core/blob/8f03257fa7abe7d6f87e6e9d0a818612fb421eae/src/mapping.js) · [条形默认计数](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/barchart/mapping.js) · [桑基默认连接权重](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/sankeydiagram/mapping.js) · [箱线须线](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/boxplot/render.js) · [小提琴分箱](https://github.com/rawgraphs/rawgraphs-charts/blob/cade3b6e1b4d35d365440b1365d51b0dd48c9bce/src/violinplot/render.js)

## 状态与边界

- 源码事实。可在固定路径确认的字段、调用、分组、参数或公式。
- 课程归纳。任务分类、误读风险、风格取舍及与01、02的对应。
- 数值复核。第08篇对历史样例的独立汇总及箱线公式基线。
- 待运行。浏览器成图、字体、导出、项目恢复、跨设备表现。
- 待版本对照。固定独立仓库提交与应用锁定发布包的差异。

## 引用与更新

介绍功能时使用固定文件链接，引用样例时注明其历史范围与字段。更换来源提交后重新核对受影响图形、字段与默认参数，并在本文件记录版本变化。代码或素材复用需要查阅对应材料的许可证，文字分析可保持课程自身表达。

[研究导航](README.md#raw-00)

---

<a id="raw-10"></a>

# 信息可视化风格设计卡与交付规范

设计卡将任务、数据、计算和风格组织为可检查的决策。以下为本课程研究归纳，可在项目中直接填写。

## 设计卡通用字段

| 字段 | 需要记录 |
|---|---|
| 研究问题与读者 | 谁需要理解什么 |
| 数据范围 | 来源、时间、单位、记录粒度和清洗 |
| 图形选择 | 对应比较、趋势、分布、关系或层级任务 |
| 字段映射 | 每个图形角色对应哪列 |
| 统计计算 | 分组、聚合、排序、归一化与布局 |
| 编码 | 位置、长度、面积、色彩、符号与标签 |
| 风格 | 画布、颜色域、字体、间距、线宽与图例 |
| 叙事 | 总览、重点、细节、证据和限制 |
| 复现 | 来源版本、数据、映射、参数与输出 |
| 验证 | 数值校对、成图检查、导出和项目恢复 |

## 设计卡1　数量比较

适合条形图。记录类别与度量，明确 count、sum 或 mean。采用共同基线、可解释排序和统一色彩。标题注明单位和时间，局部强调由研究问题决定。

交付前核对类别总数、聚合总量、轴起点、长标签和单位。与第08篇 Netflix 数值表交叉检查。

## 设计卡2　分布比较

适合箱线、小提琴与蜂群。保留每条观测值与分组，记录样本量、四分位规则、IQR 倍数、分箱或避碰条件。颜色用于稳定分组，关键异常值可配注释。

交付前核对离群点、轮廓参数、各组样本量，以及独立量程是否影响比较。

## 设计卡3　流量与连接

适合冲积、桑基、弦图。先定义节点身份、步骤顺序、源目标和权重。记录同名对象是否需要唯一ID，核查重复连接的聚合与缺失权重。

颜色编码类别，带宽表达指定权重，减少交叉并给关键节点明确标签。交付前核对方向、节点名称和必要的流量守恒条件。

## 设计卡4　层级与组成

适合矩形树图、旭日和圆形打包。保存层级列顺序、叶值、汇总规则和标签，固定排序、配色与布局参数。随机布局保存 seed。

交付前核对根与叶的数量、重复汇总、面积或角跨度语义、标签裁切和小类别可见性。

## 交付组成

1. 最终图形。SVG 供编辑，按展示需求提供位图，并写明尺寸。
2. 项目。保存 rawgraphs 项目，用于恢复数据、映射与参数。
3. 数据。提供脱离软件也能解释的字段字典、来源和清洗说明。
4. 设计说明。记录图形选择、计算和视觉取舍，注明限制。
5. 验证记录。只勾选实际完成的数值、成图、导出与恢复检查。

## 课程整合

在02对应案例原文件中记录最终作品的 A—E 分析，并链接本目录相应机制。具象图标、照片、手绘或三维处理加入后，重新核查数据尺度和标签可读性。

[研究导航](README.md#raw-00) · [机制图谱](README.md#raw-03) · [实验记录](README.md#raw-08)

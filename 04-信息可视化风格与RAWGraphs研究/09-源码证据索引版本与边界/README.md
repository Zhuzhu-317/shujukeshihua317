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

[研究导航](../00-研究总览与阅读导航/README.md)

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

[研究导航](../00-研究总览与阅读导航/README.md)

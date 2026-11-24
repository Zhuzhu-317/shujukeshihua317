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

[研究导航](../00-研究总览与阅读导航/README.md)

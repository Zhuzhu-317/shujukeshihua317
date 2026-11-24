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

[研究导航](../00-研究总览与阅读导航/README.md)

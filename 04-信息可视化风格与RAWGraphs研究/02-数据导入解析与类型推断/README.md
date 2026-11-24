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

[研究导航](../00-研究总览与阅读导航/README.md)

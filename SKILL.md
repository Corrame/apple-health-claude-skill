---
name: apple-health-analyzer
version: 1.0.0
description: Use this skill when the user wants to analyze their Apple Health export data (export.xml), investigate health trends, find anomalies in physiological data, or act as a personal health detective using wearable data. Triggers on: "analyze my Apple Health data", "look at my health export", "check my heart rate trends", "investigate my sleep data", "health data analysis", or any request to dig into Apple Watch / iPhone health metrics from an export file. IMPORTANT: This skill requires Claude Code or equivalent (file system access + script execution). Do not attempt without these capabilities.
metadata:
  openclaw:
    requires:
      anyBins: [python3, python]
    homepage: https://github.com/HomuraTokido/apple-health-analyzer
    os: [macos, windows, linux]
---

# Apple Health Data Analysis

## 前提：确认运行环境

在开始之前，确认你具备：
- 能读取本地文件系统
- 能写Python脚本并执行
- 能在同一对话里反复迭代

如果不具备，直接告诉用户："这个分析需要在Claude Code中运行，普通聊天模式无法访问本地文件。"

用户需要准备：
- `export.xml`（iPhone -> 健康 -> 右上角头像 -> 导出所有健康数据）
- Python环境（pandas、numpy、xml.etree.ElementTree）

---

## 你的角色

你是数据侦探，兼任私人医生。**严格按顺序工作：先侦探，后医生。**

侦探阶段：只发现事实，不给建议。
医生阶段：结合用户背景，给出能落地的建议。

---

## 第一步：读项目README（如果存在）

项目目录里如果有README.md，先读它——里面有数据路径、已有结论、数据质量警告、用户背景。
不要重复已做过的分析。

---

## 第二步：扫描数据结构

写脚本扫描XML，列出所有Record类型和数量。**跳过HeartRate**（数量巨大会淹没其他类型），先看其他指标的全貌。

```python
import xml.etree.ElementTree as ET, sys
sys.stdout.reconfigure(encoding='utf-8')
types = {}
for event, elem in ET.iterparse('export.xml', events=['start']):
    if elem.tag == 'Record':
        t = elem.attrib.get('type','')
        if 'HeartRate' not in t:
            types[t] = types.get(t, 0) + 1
    elif elem.tag in ('Workout','ActivitySummary'):
        types[elem.tag] = types.get(elem.tag, 0) + 1
    elem.clear()
for k,v in sorted(types.items(), key=lambda x:-x[1]):
    print(f'{v:8d}  {k}')
```

**在分析任何数字之前，先搞清楚数据从哪来。**

---

## 第三步：设备溯源（不能跳过）

Apple Health把所有设备数据混在一起，不同设备的同一指标算法不同，直接比较会产生虚假趋势。

问用户：
- 用过哪些设备（手机、Apple Watch型号、手环、第三方APP）？
- 大概什么时候换的？

常见陷阱：
- 手环的VO2Max与Apple Watch算法不可比
- watchOS 9之前的Watch不支持睡眠分期
- 第三方APP写入数据质量参差不齐
- 家人数据可能混入（体重等）
- 设备切换时间点与指标跳变重合 ≠ 真实生理变化

---

## 第四步：逐类指标分析

按优先级写脚本分析，每个指标都要标注数据来源：

**心肺类（最重要）**
- RHR（静息心率）：年度趋势，分设备对比
- HRV（心率变异性）：自主神经健康指标
- VO2Max：心肺功能，注意设备算法差异

**睡眠类**
- 入睡时间分布：昼夜节律，注意轮班用户
- 睡眠时长：双峰分布可能是班次驱动
- 深睡/REM：只信任支持分期的设备数据

**活动类**
- 步数：区分"职业性步行"和"主动锻炼"（步数高不等于有训练效果）
- Workout记录：类型、频率、持续性
- PhysicalEffort：MET强度（3.5以下属轻度活动）

**其他**
- 呼吸频率、血氧、体重（通常稀少且有噪声）
- 步态指标：步速/步幅/不对称（参考值）

---

## 第五步：交叉验证用户（不能省）

每个重要结论都要问用户：
1. 你感受到这个变化吗？
2. 这个时间点你生活里发生了什么？
3. 这个数据来自哪个设备，你觉得可信吗？

**用户本人是ground truth，数据是参考。**

---

## 第六步：综合画像

按以下结构输出，不要只罗列数字：

```
一、生活节律（作息、睡眠结构，解释原因）
二、心肺功能（最核心的健康指标趋势）
三、睡眠质量
四、体力活动
五、其他指标
六、综合结论
   - 明确变差的（数据可信）
   - 稳定/正常的
   - 已解释的（有明确原因）
   - 数据天花板（Apple Watch测不到什么）
```

每条结论注明：可信度、数据来源、是否经用户确认。

---

## 第七步：医生模式（侦探完成后才切换）

给建议之前必须了解：
- 职业和生活结构（轮班？久坐？）
- 主观感受（累吗？睡得好吗？）
- 心理状态（抑郁/焦虑的迹象？）
- 过去尝试过什么，为什么停了

建议原则：
- 符合用户实际条件（钱、时间、身体限制）
- 给"能做到的最小一步"，不给"应该"
- 短期和长期分开
- 数据解决不了的问题承认边界，不乱给建议

---

## 数据质量红线

以下情况必须降级结论，不能直接报数字：

- 跨设备对比同一指标（不同设备基准不同）
- 年度样本量 < 10
- 手环VO2Max（算法低估）
- watchOS 9之前的睡眠分期（不支持）
- 插值体重数据（容易被异常值污染）

---

## 硬性规则

1. 先计算，后结论。不带假设去找数据。
2. 不按星期几分析——用户可能是轮班制，没有固定周末。
3. 不套"普通人"框架——先问清楚用户的生活结构。
4. 说清楚数据的天花板：压力、饮食、情绪、慢性炎症，Apple Watch测不到。
5. 侦探阶段不给建议。医生阶段不重新分析数据。
6. 步数高不等于在锻炼——区分职业步行和有训练效果的运动。

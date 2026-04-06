---
name: image-analysis
description: 识别与判断图片相关信息的通用技能。用于从图像中提取结构化事实、进行内容判断，并输出带依据与置信度的结论；同时覆盖印刷色彩管理与浏览器端大图处理工程实践。
metadata:
  author: You
  version: "2026.04.06"
---

> 该技能用于图片相关信息判断：先抽取可观察事实，再做推断，最后输出结论、置信度与依据。

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| 图片信息判断工作流 | 标准化流程：目标定义 → 事实提取 → 假设评估 → 任务适配 → 最终决策 | [core-image-information-judgment](references/core-image-information-judgment.md) |

## Advanced References

### 印刷与色彩管理

| Topic | Description | Reference |
|-------|-------------|-----------|
| RGB→CMYK 判断与转换策略 | 面向印刷场景的风险判断与参数决策（ICC、渲染意图、黑场补偿、质量门禁） | [advanced-print-color-management](references/advanced-print-color-management.md) |

### 浏览器端工程化（大图处理）

| Topic | Description | Reference |
|-------|-------------|-----------|
| WASM + Worker 图像转换流水线 | 浏览器端高保真图像转换工程实践：初始化、缓存、零拷贝传输、稳定性与观测性 | [advanced-wasm-worker-image-pipeline](references/advanced-wasm-worker-image-pipeline.md) |

## Output Principles

- 先描述**可见事实**，再给出**推断结论**，避免把猜测当事实。
- 每条关键结论都附**证据点**（位置、文字、区域、对象特征）。
- 结论需带**置信度**（高/中/低或数值）与**限制条件**（如“图片模糊导致不确定”）。
- 对于高风险场景，优先输出“需要人工复核”的明确建议。
- 涉及印刷输出时，必须记录**可追溯参数**：source ICC、target ICC、rendering intent、BPC。
- 涉及大图前端处理时，优先使用**非阻塞执行模型**（Worker）并控制内存峰值（Transferable/缓存/限并发）。
---
name: core-tiff-metadata-parsing
description: 解析TIFF文件元数据的实用技能，包括DPI检测、CMYK判断、通道计数、透明通道检测和分辨率单位判断。
---

# TIFF Metadata Parsing for Image Validation

本参考用于**图片校验**场景中的TIFF文件元数据解析，基于`utif`库实现浏览器端TIFF解析能力。适用于印刷前检查、图像质量验证和自动化处理流程。

## 适用场景

- 印刷前检查TIFF文件的DPI是否符合要求
- 判断图像是否为CMYK模式（印刷专用）
- 检测图像通道数量（颜色通道+透明通道）
- 验证分辨率单位是否为PPI（像素/英寸）
- 批量处理多个TIFF文件的元数据提取

## 核心功能概述

### 1. DPI（分辨率）提取
- 从TIFF标签中提取X方向分辨率
- 支持多种分辨率值格式（直接数值、分数数组）
- 处理缺失DPI的情况，返回null

### 2. CMYK检测
- 通过光度解释（PhotometricInterpretation）和采样数（SamplesPerPixel）判断
- 支持标准CMYK（光度解释=5）和非标准4通道图像
- 兼容多种TIFF标签表示方式

### 3. 通道计数
- 计算颜色通道数量（排除透明通道）
- 通过ExtraSamples标签检测透明通道
- 返回有效的颜色通道数（1-4）

### 4. 透明通道检测
- 检测Alpha通道（预乘或非预乘）
- 基于ExtraSamples标签值判断（1=相关alpha，2=无关alpha）

### 5. 分辨率单位判断
- 判断是否为PPI（像素/英寸）单位
- 基于ResolutionUnit标签（2=英寸）

## 技术实现要点

### TIFF标签常量定义
```typescript
const TIFF_TAGS = {
  X_RESOLUTION: 282,        // X方向分辨率
  Y_RESOLUTION: 283,        // Y方向分辨率  
  RESOLUTION_UNIT: 296,     // 分辨率单位
  PHOTOMETRIC_INTERPRETATION: 262, // 光度解释
  SAMPLES_PER_PIXEL: 277,   // 每像素采样数
  EXTRA_SAMPLES: 338        // 额外采样（透明通道）
} as const
```

### 元数据接口
```typescript
interface TiffMetadata {
  name: string              // 文件名
  dpi: number | null        // DPI值
  isCMYK: boolean           // 是否为CMYK
  channelCount: number | null // 颜色通道数
  hasAlpha: boolean         // 是否有透明通道
  rawTags: any              // 原始标签
  error?: string           // 错误信息
  isPPI: boolean           // 是否为PPI单位
}
```

## 核心解析函数

### 1. 主解析函数
```typescript
export async function parseTiffMetadata(files: File[] | File): Promise<TiffMetadata[]>
```
- 支持单个文件或文件数组
- 使用Promise.allSettled确保所有文件都被处理
- 保持输入文件顺序
- 包含错误处理和日志记录

### 2. DPI提取逻辑
```typescript
function extractDpi(tags: any, fileName: string): number | null
```
- 尝试多种标签键名获取X分辨率
- 处理数组格式的分辨率值（分数表示）
- 返回四舍五入的整数值

### 3. CMYK检测逻辑
```typescript
function detectCMYK(tags: any): boolean
```
- 标准CMYK：光度解释=5且采样数=4
- 非标准CMYK：光度解释缺失但采样数=4
- 扩展判断：光度解释=5且采样数≥4

### 4. 通道计数逻辑
```typescript
function extractChannelCount(tags: any): number | null
```
- 总通道数 = SamplesPerPixel
- 透明通道数 = ExtraSamples中值为1或2的数量
- 颜色通道数 = 总通道数 - 透明通道数

### 5. 透明通道检测
```typescript
function detectAlphaChannel(tags: any): boolean
```
- ExtraSamples=1：相关alpha（预乘）
- ExtraSamples=2：无关alpha（非预乘）
- 其他值：无透明通道

## 使用示例

### 基本使用
```typescript
import { parseTiffMetadata } from './exif'

// 单个文件
const file = document.querySelector('input[type="file"]').files[0]
const metadata = await parseTiffMetadata(file)

// 多个文件
const files = document.querySelector('input[type="file"]').files
const results = await parseTiffMetadata(Array.from(files))
```

### 结果处理
```typescript
results.forEach(result => {
  console.log(`文件: ${result.name}`)
  console.log(`DPI: ${result.dpi}`)
  console.log(`CMYK: ${result.isCMYK}`)
  console.log(`通道数: ${result.channelCount}`)
  console.log(`透明通道: ${result.hasAlpha}`)
  console.log(`PPI单位: ${result.isPPI}`)
  
  if (result.error) {
    console.error(`错误: ${result.error}`)
  }
})
```

## 印刷质量检查规则

### DPI要求
- 普通印刷：≥300 DPI
- 高质量印刷：≥600 DPI  
- 大幅面印刷：≥150 DPI（视观看距离）

### CMYK检查
- 印刷前必须确认是否为CMYK模式
- RGB图像需要转换，存在色差风险
- 4通道图像可能是CMYK或RGBA

### 通道验证
- CMYK：4个颜色通道
- CMYK+Alpha：5个通道
- RGB：3个颜色通道
- RGB+Alpha：4个通道
- 灰度：1个颜色通道
- 灰度+Alpha：2个通道

## 错误处理策略

### 文件类型错误
```typescript
if (!isTiffFile(file.name)) {
  return createErrorMetadata(file.name, '非TIFF文件')
}
```

### 解析失败
```typescript
try {
  const metadata = await parseFileMetadata(file)
  return metadata
} catch (error) {
  return createErrorMetadata(file.name, error.message)
}
```

### 批量处理容错
```typescript
const settledResults = await Promise.allSettled(
  fileArray.map(async (file, index) => {
    // 每个文件的处理逻辑
  })
)
```

## 性能优化建议

### 1. 批量处理
- 使用Promise.allSettled并行处理多个文件
- 保持结果顺序与输入顺序一致
- 限制并发数量避免内存溢出

### 2. 内存管理
- 及时释放ArrayBuffer引用
- 避免在循环中创建大型临时对象
- 使用流式处理大文件

### 3. 缓存策略
- 缓存已解析的元数据
- 基于文件哈希避免重复解析
- 实现增量更新机制

## 集成到图片校验工作流

### 校验步骤
1. **文件类型验证**：确认是否为TIFF文件
2. **元数据提取**：调用parseTiffMetadata
3. **质量检查**：验证DPI、色彩模式、通道数
4. **结果输出**：结构化报告和修复建议

### 输出格式
```json
{
  "file": "example.tif",
  "validation": {
    "dpi": {
      "value": 300,
      "status": "pass",
      "requirement": "≥300"
    },
    "colorMode": {
      "value": "CMYK",
      "status": "pass"
    },
    "channels": {
      "color": 4,
      "alpha": 0,
      "status": "pass"
    },
    "overall": "pass"
  },
  "recommendations": []
}
```

## 常见问题排查

### 1. DPI提取失败
- 检查TIFF标签结构
- 验证XResolution标签是否存在
- 确认分辨率值格式

### 2. CMYK判断不准确
- 检查PhotometricInterpretation标签
- 验证SamplesPerPixel值
- 考虑非标准CMYK格式

### 3. 通道计数错误
- 确认ExtraSamples标签解析
- 区分透明通道和额外数据通道
- 处理数组格式的ExtraSamples值

### 4. 性能问题
- 大文件处理时间过长
- 内存占用过高
- 并发处理限制

## 最佳实践

### 1. 渐进式处理
- 先验证文件类型，再解析元数据
- 分步骤处理，及时反馈进度
- 实现取消机制

### 2. 日志记录
- 记录解析过程中的关键信息
- 保存错误上下文便于调试
- 实现可配置的日志级别

### 3. 可扩展性
- 支持新的TIFF标签扩展
- 插件化校验规则
- 自定义质量阈值

## 相关工具和库

- **UTIF**：浏览器端TIFF解析库
- **ExifReader**：EXIF元数据读取
- **ImageMagick**：服务器端图像处理
- **Sharp**：Node.js图像处理库

<!--
Source references:
- exif.ts 文件实现
- TIFF 文件格式规范
- 印刷行业图像质量要求
-->

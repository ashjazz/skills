---
name: pptx
description: "Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions "deck", "slides", "presentation", or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill."
license: Proprietary. LICENSE.txt has complete terms
---

# PPTX技能

## 快速参考

| 任务 | 指南 |
|------|-------|
| 读取/分析内容 | `python -m markitdown presentation.pptx` |
| 编辑或从模板创建 | 阅读 [editing.md](editing.md) |
| 从零创建 | 阅读 [pptxgenjs.md](pptxgenjs.md) |

---

## 读取内容

```bash
# 文本提取
python -m markitdown presentation.pptx

# 视觉概览
python scripts/thumbnail.py presentation.pptx

# 原始XML
python scripts/office/unpack.py presentation.pptx unpacked/
```

---

## 编辑工作流程

**阅读 [editing.md](editing.md) 了解完整详情。**

1. 使用`thumbnail.py`分析模板
2. 解包 → 操作幻灯片 → 编辑内容 → 清理 → 打包

---

## 从零创建

**阅读 [pptxgenjs.md](pptxgenjs.md) 了解完整详情。**

当没有模板或参考演示文稿时使用。

---

## 设计理念

**不要创建无聊的幻灯片。** 白色背景上的纯项目符号不会给任何人留下深刻印象。考虑此列表中的想法用于每张幻灯片。

### 开始之前

- **选择大胆、内容知情的调色板**：调色板应该感觉为这个主题设计。如果将你的颜色换成完全不同的演示文稿仍然"有效"，你没有做出足够具体的选择。
- **主导胜于平等**：一种颜色应该主导（60-70%视觉权重），有1-2个辅助色调和一个锐利强调色。永远不要给所有颜色相等的权重。
- **深浅对比**：深色背景用于标题+结论幻灯片，浅色用于内容（"三明治"结构）。或者在整个过程中承诺深色以获得高级感。
- **承诺视觉主题**：选择一个独特的元素并重复它——圆角图像框架、彩色圆圈中的图标、粗单边边框。将其贯穿每张幻灯片。

### 调色板

选择匹配你主题的颜色——不要默认使用通用蓝色。使用这些调色板作为灵感：

| 主题 | 主要 | 次要 | 强调 |
|-------|---------|-----------|--------|
| **午夜行政** | `1E2761`（海军蓝） | `CADCFC`（冰蓝） | `FFFFFF`（白色） |
| **森林与苔藓** | `2C5F2D`（森林） | `97BC62`（苔藓） | `F5F5F5`（奶油色） |
| **珊瑚活力** | `F96167`（珊瑚） | `F9E795`（金色） | `2F3C7E`（海军蓝） |
| **温暖赤陶** | `B85042`（赤陶） | `E7E8D1`（沙子） | `A7BEAE`（鼠尾草） |
| **海洋渐变** | `065A82`（深蓝） | `1C7293`（青色） | `21295C`（午夜） |
| **炭灰极简** | `36454F`（炭灰） | `F2F2F2`（灰白色） | `212121`（黑色） |
| **青色信任** | `028090`（青色） | `00A896`（海沫） | `02C39A`（薄荷） |
| **浆果与奶油** | `6D2E46`（浆果） | `A26769`（尘埃玫瑰） | `ECE2D0`（奶油色） |
| **鼠尾草平静** | `84B59F`（鼠尾草） | `69A297`（桉树） | `50808E`（板岩） |
| **樱桃大胆** | `990011`（樱桃） | `FCF6F5`（灰白色） | `2F3C7E`（海军蓝） |

### 每张幻灯片

**每张幻灯片都需要视觉元素**——图像、图表、图标或形状。纯文本幻灯片是容易被遗忘的。

**布局选项**：
- 两列（文本在左，插图在右）
- 图标+文本行（彩色圆圈中的图标、粗体标题、下方描述）
- 2x2或2x3网格（一侧图像，另一侧内容块网格）
- 半出血图像（完整左或右侧）带内容叠加

**数据显示**：
- 大型数据调用（60-72pt的大数字，下面有小型标签）
- 比较列（之前/之后、利弊、并排选项）
- 时间线或流程（编号步骤、箭头）

**视觉抛光**：
- 部分标题旁边彩色小圆圈中的图标
- 关键数据或标语的斜体强调文本

### 排版

**选择有趣的字体配对**——不要默认使用Arial。选择有个性的标题字体并与干净的正文字体配对。

| 标题字体 | 正文字体 |
|-------------|-----------|
| Georgia | Calibri |
| Arial Black | Arial |
| Calibri | Calibri Light |
| Cambria | Calibri |
| Trebuchet MS | Calibri |
| Impact | Arial |
| Palatino | Garamond |
| Consolas | Calibri |

| 元素 | 大小 |
|---------|------|
| 幻灯片标题 | 36-44pt粗体 |
| 部分标题 | 20-24pt粗体 |
| 正文文本 | 14-16pt |
| 标签 | 10-12pt柔和 |

### 间距

- 0.5"最小边距
- 0.3-0.5"内容块之间
- 留出呼吸空间——不要填满每一寸

### 避免（常见错误）

- **不要重复相同的布局**——在幻灯片之间变化列、卡片和调用
- **不要居中正文文本**——左对齐段落和列表；只居中标题
- **不要节省大小对比**——标题需要36pt+才能从14-16pt正文中脱颖而出
- **不要默认使用蓝色**——选择反映特定主题的颜色
- **不要随机混合间距**——选择0.3"或0.5"间隙并一致使用
- **不要设计一张幻灯片而让其余保持简单**——完全承诺或在整个过程中保持简单
- **不要创建纯文本幻灯片**——添加图像、图标、图表或视觉元素；避免纯标题+项目符号
- **不要忘记文本框填充**——当对齐线条或形状与文本边缘时，在文本框上设置`margin: 0`，或偏移形状以考虑填充
- **不要使用低对比度元素**——图标和文本都需要与背景形成强烈对比；避免浅色背景上的浅色文本或深色背景上的深色文本
- **永远不要在标题下使用强调线**——这些是AI生成幻灯片的标志；使用空白或背景颜色代替

---

## QA（必需）

**假设存在问题。你的工作是找到它们。**

你的第一个渲染几乎从不正确。将QA视为bug hunt，而不是确认步骤。如果你在第一次检查时没有发现任何问题，你没有足够努力地看。

### 内容QA

```bash
python -m markitdown output.pptx
```

检查缺失内容、拼写错误、错误顺序。

**使用模板时，检查剩余的占位符文本**：

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|this.*(page|slide).*layout"
```

如果grep返回结果，在声明成功之前修复它们。

### 视觉QA

**⚠️ 使用子代理**——即使对于2-3张幻灯片。你一直在看代码，会看到你期望的，而不是实际存在的。子代理有新鲜的眼光。

将幻灯片转换为图像（参见[转换为图像](#converting-to-images)），然后使用此提示：

```
视觉上检查这些幻灯片。假设存在问题——找到它们。

寻找：
- 重叠元素（文本穿过形状、线条穿过词语、堆叠元素）
- 文本溢出或在边缘/框边界处被切断
- 为单行文本定位的装饰线但标题换行到两行
- 来源引用或页脚与上方内容碰撞
- 元素太接近（< 0.5"间隙）或卡片/部分几乎触摸
- 不均匀间隙（一个地方大空白，另一个地方拥挤）
- 与幻灯片边缘的边距不足（< 0.5"）
- 列或类似元素不一致对齐
- 低对比度文本（例如，奶油色背景上的浅灰色文本）
- 低对比度图标（例如，深色背景上的深色图标没有对比圆圈）
- 文本框太窄导致过度换行
- 剩余占位符内容

对于每张幻灯片，列出问题或关注区域，即使很小。

阅读并分析这些图像：
1. /path/to/slide-01.jpg (预期：[简要描述])
2. /path/to/slide-02.jpg (预期：[简要描述])

报告发现的所有问题，包括小的。
```

### 验证循环

1. 生成幻灯片 → 转换为图像 → 检查
2. **列出发现的问题**（如果没有发现，更批判性地再看）
3. 修复问题
4. **重新验证受影响的幻灯片**——一个修复通常会创建另一个问题
5. 重复直到完整通过没有发现新问题

**在完成至少一个修复和验证循环之前不要声明成功。**

---

## 转换为图像

将演示文稿转换为单独的幻灯片图像以进行视觉检查：

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

这创建了`slide-01.jpg`、`slide-02.jpg`等。

要在修复后重新渲染特定幻灯片：

```bash
pdftoppm -jpeg -r 150 -f N -l N output.pdf slide-fixed
```

---

## 依赖项

- `pip install "markitdown[pptx]"` - 文本提取
- `pip install Pillow` - 缩略图网格
- `npm install -g pptxgenjs` - 从零创建
- LibreOffice (`soffice`) - PDF转换（通过`scripts/office/soffice.py`为沙盒环境自动配置）
- Poppler (`pdftoppm`) - PDF转图像

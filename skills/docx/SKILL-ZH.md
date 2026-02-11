---
name: docx
description: "Use this skill whenever the user wants to create, read, edit, or manipulate Word documents (.docx files). Triggers include: any mention of "Word doc", "word document", ".docx", or requests to produce professional documents with formatting like tables of contents, headings, page numbers, or letterheads. Also use when extracting or reorganizing content from .docx files, inserting or replacing images in documents, performing find-and-replace in Word files, working with tracked changes or comments, or converting content into a polished Word document. If the user asks for a "report", "memo", "letter", "template", or similar deliverable as a Word or .docx file, use this skill. Do NOT use for PDFs, spreadsheets, Google Docs, or general coding tasks unrelated to document generation."
license: Proprietary. LICENSE.txt has complete terms
---

# DOCX创建、编辑和分析

## 概述

.docx文件是包含XML文件的ZIP存档。

## 快速参考

| 任务 | 方法 |
|------|------|
| 读取/分析内容 | `pandoc` 或解包以获取原始XML |
| 创建新文档 | 使用 `docx-js` —— 参见下面的创建新文档 |
| 编辑现有文档 | 解包 → 编辑XML → 重新打包 —— 参见下面的编辑现有文档 |

### 将.doc转换为.docx

必须先转换传统的`.doc`文件才能编辑：

```bash
python scripts/office/soffice.py --headless --convert-to docx document.doc
```

### 读取内容

```bash
# 提取带修订追踪的文本
pandoc --track-changes=all document.docx -o output.md

# 原始XML访问
python scripts/office/unpack.py document.docx unpacked/
```

### 转换为图像

```bash
python scripts/office/soffice.py --headless --convert-to pdf document.docx
pdftoppm -jpeg -r 150 document.pdf page
```

### 接受修订追踪

要生成所有修订追踪都被接受的干净文档（需要LibreOffice）：

```bash
python scripts/accept_changes.py input.docx output.docx
```

---

## 创建新文档

使用JavaScript生成.docx文件，然后验证。安装：`npm install -g docx`

### 设置
```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, ImageRun,
        Header, Footer, AlignmentType, PageOrientation, LevelFormat, ExternalHyperlink,
        TableOfContents, HeadingLevel, BorderStyle, WidthType, ShadingType,
        VerticalAlign, PageNumber, PageBreak } = require('docx');

const doc = new Document({ sections: [{ children: [/* content */] }] });
Packer.toBuffer(doc).then(buffer => fs.writeFileSync("doc.docx", buffer));
```

### 验证
创建文件后验证它。如果验证失败，解包，修复XML，然后重新打包。
```bash
python scripts/office/validate.py doc.docx
```

### 页面尺寸

```javascript
// 关键：docx-js默认为A4，不是US Letter
// 始终明确设置页面尺寸以获得一致的结果
sections: [{
  properties: {
    page: {
      size: {
        width: 12240,   // 8.5英寸 DXA
        height: 15840   // 11英寸 DXA
      },
      margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } // 1英寸边距
    }
  },
  children: [/* content */]
}]
```

**常见页面尺寸（DXA单位，1440 DXA = 1英寸）**：

| 纸张 | 宽度 | 高度 | 内容宽度（1"边距） |
|-------|-------|--------|---------------------------|
| US Letter | 12,240 | 15,840 | 9,360 |
| A4（默认） | 11,906 | 16,838 | 9,026 |

**横向方向**：docx-js在内部交换宽度/高度，所以传递纵向尺寸并让它处理交换：
```javascript
size: {
  width: 12240,   // 传递短边作为宽度
  height: 15840,  // 传递长边作为高度
  orientation: PageOrientation.LANDSCAPE  // docx-js在XML中交换它们
},
// 内容宽度 = 15840 - 左边距 - 右边距（使用长边）
```

### 样式（覆盖内置标题）

使用Arial作为默认字体（普遍支持）。保持标题黑色以提高可读性。

```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 12pt默认
    paragraphStyles: [
      // 重要：使用确切ID覆盖内置样式
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 32, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } }, // TOC需要outlineLevel
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 } },
    ]
  },
  sections: [{
    children: [
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("Title")] }),
    ]
  }]
});
```

### 列表（永远不要使用unicode项目符号）

```javascript
// ❌ 错误——永远不要手动插入项目符号字符
new Paragraph({ children: [new TextRun("• Item")] })  // 不好
new Paragraph({ children: [new TextRun("\u2022 Item")] })  // 不好

// ✅ 正确——使用带LevelFormat.BULLET的编号配置
const doc = new Document({
  numbering: {
    config: [
      { reference: "bullets",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "numbers",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
    ]
  },
  sections: [{
    children: [
      new Paragraph({ numbering: { reference: "bullets", level: 0 },
        children: [new TextRun("Bullet item")] }),
      new Paragraph({ numbering: { reference: "numbers", level: 0 },
        children: [new TextRun("Numbered item")] }),
    ]
  }]
});

// ⚠️ 每个引用创建独立编号
// 相同引用 = 继续 (1,2,3 然后 4,5,6)
// 不同引用 = 重新开始 (1,2,3 然后 1,2,3)
```

### 表格

**关键：表格需要双重宽度**——在表格上设置`columnWidths`并在每个单元格上设置`width`。没有两者，一些平台上的表格会错误渲染。

```javascript
// 关键：始终设置表格宽度以获得一致渲染
// 关键：使用ShadingType.CLEAR（不是SOLID）以防止黑色背景
const border = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const borders = { top: border, bottom: border, left: border, right: border };

new Table({
  width: { size: 9360, type: WidthType.DXA }, // 始终使用DXA（百分比在Google Docs中破坏）
  columnWidths: [4680, 4680], // 必须总和为表格宽度（DXA: 1440 = 1英寸）
  rows: [
    new TableRow({
      children: [
        new TableCell({
          borders,
          width: { size: 4680, type: WidthType.DXA }, // 也在每个单元格上设置
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR }, // CLEAR不是SOLID
          margins: { top: 80, bottom: 80, left: 120, right: 120 }, // 单元格填充（内部，不添加到宽度）
          children: [new Paragraph({ children: [new TextRun("Cell")] })]
        })
      ]
    })
  ]
})
```

**表格宽度计算**：

始终使用 `WidthType.DXA` —— `WidthType.PERCENTAGE` 在Google Docs中破坏。

```javascript
// 表格宽度 = columnWidths总和 = 内容宽度
// US Letter带1"边距：12240 - 2880 = 9360 DXA
width: { size: 9360, type: WidthType.DXA },
columnWidths: [7000, 2360]  // 必须总和为表格宽度
```

**宽度规则**：
- **始终使用 `WidthType.DXA`** —— 永远不要使用 `WidthType.PERCENTAGE`（与Google Docs不兼容）
- 表格宽度必须等于 `columnWidths` 的总和
- 单元格 `width` 必须匹配相应的 `columnWidth`
- 单元格 `margins` 是内部填充——它们减少内容区域，而不是添加到单元格宽度
- 对于全宽表格：使用内容宽度（页面宽度减去左右边距）

### 图像

```javascript
// 关键：type参数是必需的
new Paragraph({
  children: [new ImageRun({
    type: "png", // 必需：png, jpg, jpeg, gif, bmp, svg
    data: fs.readFileSync("image.png"),
    transformation: { width: 200, height: 150 },
    altText: { title: "Title", description: "Desc", name: "Name" } // 所有三个必需
  })]
})
```

### 分页符

```javascript
// 关键：PageBreak必须在Paragraph内部
new Paragraph({ children: [new PageBreak()] })

// 或者使用pageBreakBefore
new Paragraph({ pageBreakBefore: true, children: [new TextRun("New page")] })
```

### 目录

```javascript
// 关键：标题必须只使用HeadingLevel——不使用自定义样式
new TableOfContents("Table of Contents", { hyperlink: true, headingStyleRange: "1-3" })
```

### 页眉/页脚

```javascript
sections: [{
  properties: {
    page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } // 1440 = 1英寸
  },
  headers: {
    default: new Header({ children: [new Paragraph({ children: [new TextRun("Header")] })] })
  },
  footers: {
    default: new Footer({ children: [new Paragraph({
      children: [new TextRun("Page "), new TextRun({ children: [PageNumber.CURRENT] })]
    })] })
  },
  children: [/* content */]
}]
```

### docx-js的关键规则

- **明确设置页面尺寸** - docx-js默认为A4；对美国文档使用US Letter（12240 x 15840 DXA）
- **横向：传递纵向尺寸** - docx-js在内部交换宽度/高度；传递短边作为`width`，长边作为`height`，并设置`orientation: PageOrientation.LANDSCAPE`
- **永远不要使用 `\n`** - 使用单独的Paragraph元素
- **永远不要使用unicode项目符号** - 使用带`LevelFormat.BULLET`的编号配置
- **PageBreak必须在Paragraph中** - 独立创建会产生无效XML
- **ImageRun需要`type`** - 始终指定png/jpg/等
- **始终使用DXA设置表格`width`** - 永远不要使用`WidthType.PERCENTAGE`（在Google Docs中破坏）
- **表格需要双重宽度** - `columnWidths`数组和单元格`width`，两者必须匹配
- **表格宽度 = columnWidths总和** - 对于DXA，确保它们正好相加
- **始终添加单元格边距** - 使用`margins: { top: 80, bottom: 80, left: 120, right: 120 }`以获得可读填充
- **使用 `ShadingType.CLEAR`** - 永远不要为表格底纹使用SOLID
- **TOC只要求HeadingLevel** - 标题段落不使用自定义样式
- **覆盖内置样式** - 使用确切ID："Heading1"、"Heading2"等
- **包括 `outlineLevel`** - TOC需要（0代表H1，1代表H2等）

---

## 编辑现有文档

**按顺序遵循所有3个步骤。**

### 步骤1：解包
```bash
python scripts/office/unpack.py document.docx unpacked/
```

提取XML，美化打印，合并相邻运行，并将智能引号转换为XML实体（`&#x201C;`等）以便它们在编辑中幸存。使用`--merge-runs false`跳过运行合并。

### 步骤2：编辑XML

编辑`unpacked/word/`中的文件。参见下面的XML参考以获取模式。

**使用"Claude"作为修订追踪和注释的作者**，除非用户明确要求使用不同名称。

**直接使用编辑工具进行字符串替换。不要编写Python脚本。** 脚本引入了不必要的复杂性。编辑工具确切显示正在替换什么。

**关键：对新内容使用智能引号。** 添加带撇号或引号的文本时，使用XML实体以产生智能引号：
```xml
<!-- 对专业排版使用这些实体 -->
<w:t>Here&#x2019;s a quote: &#x201C;Hello&#x201D;</w:t>
```
| 实体 | 字符 |
|--------|-----------|
| `&#x2018;` | ' (左单引号) |
| `&#x2019;` | ' (右单引号/撇号) |
| `&#x201C;` | " (左双引号) |
| `&#x201D;` | " (右双引号) |

**添加注释**：使用`comment.py`处理跨多个XML文件的样板（文本必须是预转义的XML）：
```bash
python scripts/comment.py unpacked/ 0 "Comment text with &amp; and &#x2019;"
python scripts/comment.py unpacked/ 1 "Reply text" --parent 0  # 回复注释0
python scripts/comment.py unpacked/ 0 "Text" --author "Custom Author"  # 自定义作者名
```

然后在document.xml中添加标记（参见下面的注释中的XML参考）。

### 步骤3：打包
```bash
python scripts/office/pack.py unpacked/ output.docx --original document.docx
```

使用自动修复进行验证，压缩XML，并创建DOCX。使用`--validate false`跳过。

**自动修复将修复**：
- `durableId` >= 0x7FFFFFFF（重新生成有效ID）
- `<w:t>`上缺少`xml:space="preserve"`带空白

**自动修复不会修复**：
- 格式错误的XML、无效元素嵌套、缺少关系、模式违规

### 常见陷阱

- **替换整个`<w:r>`元素**：添加修订追踪时，将整个`<w:r>...</w:r>`块替换为`<w:del>...<w:ins>...`作为兄弟。不要在运行中注入修订追踪标签。
- **保留`<w:rPr>`格式**：将原始运行的`<w:rPr>`块复制到修订追踪运行中以保持粗体、字体大小等。

---

## XML参考

### 模式合规性

- **`<w:pPr>`中的元素顺序**：`<w:pStyle>`、`<w:numPr>`、`<w:spacing>`、`<w:ind>`、`<w:jc>`、`<w:rPr>`最后
- **空白**：在带前导/尾随空白的`<w:t>`上添加`xml:space="preserve"`
- **RSID**：必须是8位十六进制（例如`00AB1234`）

### 修订追踪

**插入**：
```xml
<w:ins w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:t>inserted text</w:t></w:r>
</w:ins>
```

**删除**：
```xml
<w:del w:id="2" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:delText>deleted text</w:delText></w:r>
</w:del>
```

**在`<w:del>`内部**：使用`<w:delText>`而不是`<w:t>`，以及`<w:delInstrText>`而不是`<w:instrText>`。

**最小编辑**——只标记更改的内容：
```xml
<!-- 将"30 days"改为"60 days" -->
<w:r><w:t>The term is </w:t></w:r>
<w:del w:id="1" w:author="Claude" w:date="...">
  <w:r><w:delText>30</w:delText></w:r>
</w:del>
<w:ins w:id="2" w:author="Claude" w:date="...">
  <w:r><w:t>60</w:t></w:r>
</w:ins>
<w:r><w:t> days.</w:t></w:r>
```

**删除整个段落/列表项**——当删除段落的所有内容时，也将段落标记标记为删除，以便它与下一个段落合并。在`<w:pPr><w:rPr>`中添加`<w:del/>`：
```xml
<w:p>
  <w:pPr>
    <w:numPr>...</w:numPr>  // 如果存在列表编号
    <w:rPr>
      <w:del w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z"/>
    </w:rPr>
  </w:pPr>
  <w:del w:id="2" w:author="Claude" w:date="2025-01-01T00:00:00Z">
    <w:r><w:delText>Entire paragraph content being deleted...</w:delText></w:r>
  </w:del>
</w:p>
```
如果没有`<w:del/>`在`<w:pPr><w:rPr>`中，接受更改会留下空段落/列表项。

**拒绝其他作者的插入**——在他们的插入中嵌套删除：
```xml
<w:ins w:author="Jane" w:id="5">
  <w:del w:author="Claude" w:id="10">
    <w:r><w:delText>their inserted text</w:delText></w:r>
  </w:del>
</w:ins>
```

**恢复其他作者的删除**——在之后添加插入（不要修改他们的删除）：
```xml
<w:del w:author="Jane" w:id="5">
  <w:r><w:delText>deleted text</w:delText></w:r>
</w:del>
<w:ins w:author="Claude" w:id="10">
  <w:r><w:t>deleted text</w:t></w:r>
</w:ins>
```

### 注释

运行`comment.py`后（参见步骤2），在document.xml中添加标记。对于回复，使用`--parent`标志并将标记嵌套在父级内部。

**关键：`<w:commentRangeStart>`和`<w:commentRangeEnd>`是`<w:p>`的兄弟，永远不要在`<w:r>`内部。**

```xml
<!-- 注释标记是w:p的直接子级，永远不要在w:r内部 -->
<w:commentRangeStart w:id="0"/>
<w:del w:id="1" w:author="Claude" w:date="2025-01-01T00:00:00Z">
  <w:r><w:delText>deleted</w:delText></w:r>
</w:del>
<w:r><w:t> more text</w:t></w:r>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="0"/></w:r>

<!-- 回复1嵌套在注释0内部的注释0 -->
<w:commentRangeStart w:id="0"/>
  <w:commentRangeStart w:id="1"/>
  <w:r><w:t>text</w:t></w:r>
  <w:commentRangeEnd w:id="1"/>
<w:commentRangeEnd w:id="0"/>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="0"/></w:r>
<w:r><w:rPr><w:rStyle w:val="CommentReference"/></w:rPr><w:commentReference w:id="1"/></w:r>
```

### 图像

1. 将图像文件添加到`word/media/`
2. 将关系添加到`word/_rels/document.xml.rels`：
```xml
<Relationship Id="rId5" Type=".../image" Target="media/image1.png"/>
```
3. 将内容类型添加到`[Content_Types].xml`：
```xml
<Default Extension="png" ContentType="image/png"/>
```
4. 在document.xml中引用：
```xml
<w:drawing>
  <wp:inline>
    <wp:extent cx="914400" cy="914400"/>  <!-- EMU：914400 = 1英寸 -->
    <a:graphic>
      <a:graphicData uri=".../picture">
        <pic:pic>
          <pic:blipFill><a:blip r:embed="rId5"/></pic:blipFill>
        </pic:pic>
      </a:graphicData>
    </a:graphic>
  </wp:inline>
</w:drawing>
```

---

## 依赖项

- **pandoc**：文本提取
- **docx**：`npm install -g docx`（新文档）
- **LibreOffice**：PDF转换（通过`scripts/office/soffice.py`为沙盒环境自动配置）
- **Poppler**：用于图像的`pdftoppm`

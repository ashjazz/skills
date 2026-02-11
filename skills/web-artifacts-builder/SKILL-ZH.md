---
name: web-artifacts-builder
description: Suite of tools for creating elaborate, multi-component claude.ai HTML artifacts using modern frontend web technologies (React, Tailwind CSS, shadcn/ui). Use for complex artifacts requiring state management, routing, or shadcn/ui components - not for simple single-file HTML/JSX artifacts.
license: Complete terms in LICENSE.txt
---

# Web人工制品构建者

要构建强大的前端 claude.ai 人工制品，遵循以下步骤：
1. 使用 `scripts/init-artifact.sh` 初始化前端仓库
2. 通过编辑生成的代码开发你的工件
3. 使用 `scripts/bundle-artifact.sh` 将所有代码捆绑成单个HTML文件
4. 向用户展示工件
5. （可选）测试工件

**堆栈**：React 18 + TypeScript + Vite + Parcel（捆绑）+ Tailwind CSS + shadcn/ui

## 设计和样式指南

**非常重要**：为了避免通常被称为"AI垃圾"的东西，避免使用过多的居中布局、紫色渐变、统一圆角和Inter字体。

## 快速开始

### 步骤1：初始化项目

运行初始化脚本以创建新的React项目：
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

这会创建一个完全配置的项目包括：
- ✅ React + TypeScript（通过Vite）
- ✅ Tailwind CSS 3.4.1带shadcn/ui主题系统
- ✅ 配置路径别名（`@/`）
- ✅ 40+个shadcn/ui组件预安装
- ✅ 所有Radix UI依赖项包括
- ✅ 使用路径别名支持配置Parcel（通过.parcelrc）
- ✅ Node 18+兼容性（自动检测并固定Vite版本）

### 步骤2：开发你的工件

要构建工件，编辑生成的文件。参见下面的**常见开发任务**以获取指导。

### 步骤3：捆绑成单个HTML文件

要将React应用捆绑成单个HTML工件：
```bash
bash scripts/bundle-artifact.sh
```

这创建了`bundle.html`——一个包含所有JavaScript、CSS和依赖项内联的自我包含工件。这个文件可以直接在Claude对话中作为工件共享。

**要求**：你的项目必须在根目录中有`index.html`。

**脚本做什么**：
- 安装捆绑依赖项（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 创建带路径别名支持的`.parcelrc`配置
- 使用Parcel构建（没有源映射）
- 使用html-inline将所有资产内联到单个HTML中

### 步骤4：与用户分享工件

最后，与用户分享捆绑的HTML文件，以便他们可以在对话中将其查看为工件。

### 步骤5：测试/可视化工件（可选）

注意：这完全是可选的步骤。仅在必要或请求时执行。

要测试/可视化工件，使用可用工具（包括其他技能或内置工具如Playwright或Puppeteer）。通常，避免预先测试工件，因为它在请求和看到完成工件之间增加了延迟。之后，在呈现工件后，如果请求或出现问题，进行测试。

## 参考

- **shadcn/ui组件**：https://ui.shadcn.com/docs/components

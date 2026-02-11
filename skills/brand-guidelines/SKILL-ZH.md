---
name: brand-guidelines
description: Applies Anthropic's official brand colors and typography to any sort of artifact that may benefit from having Anthropic's look-and-feel. Use it when brand colors or style guidelines, visual formatting, or company design standards apply.
license: Complete terms in LICENSE.txt
---

# Anthropic 品牌样式

## 概述

要访问Anthropic的官方品牌身份和样式资源，请使用此技能。

**关键词**：品牌、企业身份、视觉身份、后处理、样式、品牌颜色、字体排版、Anthropic品牌、视觉格式化、视觉设计

## 品牌指南

### 颜色

**主要颜色**：

- 深色：`#141413` - 主要文本和深色背景
- 浅色：`#faf9f5` - 浅色背景和深色上的文本
- 中灰色：`#b0aea5` - 次要元素
- 浅灰色：`#e8e6dc` - 微妙的背景

**强调色**：

- 橙色：`#d97757` - 主要强调色
- 蓝色：`#6a9bcc` - 次要强调色
- 绿色：`#788c5d` - 第三强调色

### 字体排版

- **标题**：Poppins（备用Arial）
- **正文**：Lora（备用Georgia）
- **注意**：字体应在你的环境中预安装以获得最佳效果

## 功能

### 智能字体应用

- 将Poppins字体应用于标题（24pt及以上）
- 将Lora字体应用于正文
- 如果自定义字体不可用，自动回退到Arial/Georgia
- 保持所有系统的可读性

### 文本样式

- 标题（24pt+）：Poppins字体
- 正文文本：Lora字体
- 基于背景的智能颜色选择
- 保持文本层次和格式

### 形状和强调色

- 非文本形状使用强调色
- 循环使用橙色、蓝色和绿色
- 保持视觉趣味同时保持品牌风格

## 技术详情

### 字体管理

- 当可用时使用系统安装的Poppins和Lora字体
- 提供自动回退到Arial（标题）和Georgia（正文）
- 不需要字体安装——与现有系统字体一起工作
- 为了获得最佳效果，在你的环境中预安装Poppins和Lora字体

### 颜色应用

- 使用RGB颜色值进行精确的品牌匹配
- 通过python-pptx的RGBColor类应用
- 跨不同系统保持颜色保真度

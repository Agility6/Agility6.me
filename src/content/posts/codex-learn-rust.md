---
title: Codex Learn Rust — WIP
pubDate: 2026-04-13
categories: ['Tech']
description: ''
---

## 背景

本篇文章打算使用Codex来学习一下Rust语言，本人是Gopher对Rust只能说是略有耳闻，一直听说Rust是一门学习路线陡峭的、入坑到放弃只需要 `Hello World` 。在AI时代下，我不打算采用阅读官方的“圣经”进行学习，直接通过迁移一个项目来进行学习。

使用该开源项目作为引子来学习Rust，[HuffmanImageCompression](https://github.com/xujj25/HuffmanImageCompression) 使用Huffman编码对图像进行无损压缩和解压。 

> 可参考Huffman相关的文章： https://agility6.me/huffman-trees-experiment/

先来对Rust做一次体感过山车吧

Rust的定位是什么
- 接近 C/C++ 的性能
- 同时提供内存安全（无 GC）
- 现代语言抽象能力（类似 Go + 一部分函数式特性）
- 关键词：Ownership（所有权）、Borrowing（借用）、Zero-cost abstraction（零成本抽象）、Fearless concurrency（无畏并发）

Go vs Rust

- 没有 GC 的内存安全：所有权 (Ownership) 与借用 (Borrowing)
- 错误处理：告别 `if err != nil`
- 并发模型：无畏并发 (Fearless Concurrency)
- Go 的哲学是 **“简单、直接、高并发”**，那么 Rust 的哲学就是 **“零成本抽象、内存安全、无畏并发”**。


Agent驱动学习也算是探索一种新的学习模式，使用类似Agent Team的模式进行学习。大致的方法就是一共有4个Agent，各司其职

1. Agent 1 执行全局拆解与排期
2. Agent 2 制定教学规范并出码 (落盘到 specs)
3. 唤醒 Agent 3（排错专家）
4. 唤醒 Agent 4（QA）

```md
# Huffman-Rust Agent Team 全局指南 (文档驱动版)

## 核心铁律：万物皆落盘
任何讨论、架构决策、代码设计都必须写入 `docs/` 或 `specs/` 目录下的 Markdown 文件中，Agent 之间通过读取这些文件进行上下文同步。

## 角色定义
1. **Agent 1 (Architect & PM)**：分析整个 C++ 项目。产出系统架构文档 `docs/architecture.md` 和分步开发路线图 `docs/roadmap.md`。
2. **Agent 2 (Rust Mentor)**：根据路线图的当前任务，产出具体的模块实现规范和保姆级注释代码，保存至 `specs/<当前模块>_guide.md`。
3. **Agent 3 (Reviewer)**：在人类抄写代码出错时，解释编译错误并引导修复。
4. **Agent 4 (QA)**：为跑通的代码编写测试，并在通过后，提示人类更新 `docs/roadmap.md` 的任务状态。
   
## 核心铁律
所有模块的开发必须经过：Agent 1 拆解 -> Agent 2 教学与出码 -> 人类抄写 -> Agent 3 查漏 -> Agent 4 测试。Agent 之间不得越权。
```

### 具体步骤

**Agent 1 执行全局拆解与排期**

```text
“请阅读 `AGENT_TEAM_GUIDE.md`。现在你严格扮演 **Agent 1 (Architect & PM)**。 我提供给你了完整的 HuffmanImageCompression C++ 项目源码。请你不要写任何 Rust 代码，完成以下两项任务：

**任务 1：输出系统架构** 请分析 C++ 项目的核心流程、数据结构和模块划分。请输出 Markdown 格式，准备让我全盘复制保存为 `docs/architecture.md`。

**任务 2：制定拆解路线图** 结合 Rust 的特性，将这个项目拆解为循序渐进的开发 Milestone（里程碑），比如 M1 (基础数据结构)、M2 (文件IO)、M3 (核心算法)。每个里程碑下划分子任务。 请输出带有复选框（`- [ ]`）的 Markdown 格式，明确指出我应该‘先写哪个，再写哪个’。准备让我全盘复制保存为 `docs/roadmap.md`。
```

人类动作：将 Agent 1 生成的两个文档分别新建并保存到 `docs/architecture.md` 和 `docs/roadmap.md` 中。**以后的开发，你每天第一件事就是打开 `roadmap.md` 看今天要推进哪个复选框。

**单模块开发循环（文档驱动执行）**

```text
“请阅读 `AGENT_TEAM_GUIDE.md` 和 `docs/roadmap.md`。现在你严格扮演 **Agent 2 (Rust Mentor)**。 我现在的进度是执行任务：**[M1.1: 实现哈夫曼树节点定义]**。 请你：

1. 查阅 C++ 源码中对应的部分，讲解在 Rust 中实现该逻辑需要掌握的 1-2 个核心知识点。
    
2. 按照符合 Rust 的习惯，编写出完整的实现代码。
    
3. 代码必须带有极度详细的中文注释（解释‘为什么这样写’），方便我临摹。 请将以上内容组织为一篇完整的 Markdown 教学文档，准备让我保存为 `specs/m1_1_tree_node_guide.md`。”
```

**人类动作：** 将 Agent 2 生成的文档保存到 `specs/` 目录下。然后，打开你的 `.rs` 源码文件，**照着 spec 文档把代码敲进去**。

**Agent 3 结对查漏补缺 (不落盘，作为实时辅助)**

在实现的过程中遇到了编译器报错

```text
“请扮演 **Agent 3 (Reviewer)**。我在实现 `specs/m1_1_tree_node_guide.md` 时报错了。 报错信息：[贴报错] 当前代码：[贴代码] 请解释原因并引导我修复，不要直接给完整代码。”
```

**Agent 4 编写测试并推动进度 (更新落盘文件)**

代码跑通后，你需要通过测试来闭环这个任务。

```text
“请扮演 **Agent 4 (QA)**。我已完成了 M1.1 任务。这是我的代码：[贴代码]。 请你：

1. 提供针对该模块的严谨的单元测试代码 (`#[cfg(test)]`)，带有注释。
    
2. 提醒我：‘测试通过后，请去 `docs/roadmap.md` 中将 M1.1 任务勾选完成（`- [x]`）’。”
```


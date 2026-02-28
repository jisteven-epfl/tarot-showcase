<div align="center">

# 🃏 法式塔罗牌: 全栈 Scala Web 游戏

<p align="center">
  <a href="README.md">English</a> | <b>简体中文</b>
</p>

> **项目展示专用：** 为遵守 EPFL CS-214 课程严格的学术诚信与版权协议，本项目的源代码不可公开或私下分享。但我非常乐意在面试的屏幕共享环节，为您实时展示并深度讲解核心逻辑与技术细节。如有交流意向，请联系：**<steven.ji@epfl.com>**。

![Scala](https://img.shields.io/badge/Scala-3-red.svg?style=flat-square)
![Scala.js](https://img.shields.io/badge/Scala.js-Frontend-blue.svg?style=flat-square)
![Architecture](https://img.shields.io/badge/Architecture-Centralized__State__Machine-brightgreen.svg?style=flat-square)
![Testing](https://img.shields.io/badge/Testing-ScalaCheck-orange.svg?style=flat-square)
![EPFL](https://img.shields.io/badge/School-EPFL-red.svg?style=flat-square)

*一个功能完整、完全使用 Scala 3 构建的经典法式塔罗牌多人在线 Web 游戏。*

</div>

## 📖 概览

本项目是一个中心化的 Web 应用程序，支持 3 至 5 名玩家在线体验传统的法式塔罗牌 (French Tarot) 游戏。通过高度健壮且纯函数式的领域模型，系统严格执行了塔罗牌的复杂规则（包括叫牌机制、底牌处理、吃牌限制以及精确的分数计算）。

该应用采用统一的 Scala 代码库，通过 Scala.js 将核心游戏逻辑无缝共享于 JVM 后端和浏览器前端，确保了整个技术栈 100% 的类型安全。

### 📊 项目数据统计

- **核心代码量**：统一代码库中约 2,100 行纯 Scala 3 代码（跨越 `shared`、`jvm` 和 `js` 模块）。
- **工程质量**：网络传输层 (Wire) 序列化实现了 **100% 的分支测试覆盖率**，核心游戏逻辑达到 **90%+** 的单元测试覆盖率。
- **业务复杂度**：在纯函数式领域模型中定义了超过 **80 个核心业务函数/拓展方法**，并手动为庞杂的代数数据类型 (ADTs) 编写了超过 40 余个 JSON 模式匹配编解码分支，未依赖任何自动派生。

### 🏗️ 架构设计与函数式工程实践

本项目的构建严格遵循现代软件构建（EPFL CS-214 课程）原则，是高级函数式编程与 C/S（客户端-服务器）架构的集中体现：

- **中心化状态机 (Centralized State Machine)**：服务器作为唯一的真相源 (Source of Truth)。游戏逻辑被表达为一个确定性的状态机，在明确定义的各个状态（`Bidding` 叫牌、`Playing` 对局、`End` 结算）之间安全转换。
- **纯函数式领域模型 (Purely Functional Domain Model)**：与通常修改状态的 OOP 方法不同，整个塔罗牌规则集通过操作不可变数据结构的纯函数来实现。状态转换通过 case class 的 `copy` 返回全新副本；大量**拓展方法 (Extension Methods)** 将复杂规则（如 `isValidPlay`、`roundWinner`）从核心数据类型中解耦出来。
- **JVM + JS 共享代码**：核心 ADT（`Card`, `Suit`, `Rank`, `Hand`）及游戏规则在 `shared` 模块中定义，同时编译服务于 JVM 后端和 JavaScript 前端，确保全栈 100% 类型安全。
- **严格的信息隐藏机制 (Views)**：从根本上杜绝作弊，服务器坚决不会向客户端发送完整的游戏全局状态。它将全局 `State` 投影为特定于每位玩家的局部 `View`，确保客户端只能接收到合法可见的数据。
- **同构 Scala.js 前端**：UI 完全采用 `Scalatags` 在 Scala 中构建，无需编写原生 HTML 或 JavaScript，以声明式方式将严格类型的 `View` 直接映射为 DOM 元素。

### 📐 项目边界：课程框架 vs. 团队的真实工作量

本项目基于 EPFL CS-214 课程提供的一个最小化的运行时骨架进行开发。为了准确展示我们的工程能力，有必要明确区分哪些是“给定的”，哪些是“我们从零搭建的”。

**课程框架提供的部分：**

- 底层的 WebSocket 通信引擎和服务器启动脚手架。
- 空的 trait/接口签名（如 `StateMachine`, `WebClientAppInstance`），仅规定了*需要实现哪些方法*，而非*如何实现*。

**由本团队（三人合作）设计并完整实现的部分：**

- **完整的领域模型 (Domain Model)：** 78 张塔罗牌牌组定义、含将牌在内的五种花色、全部计分规则（Oudlers、人头牌、半点制），以及“Petit Sec 重新洗牌”等特殊机制。
- **完整的后端状态机大脑：** 整个 `State` sealed 层级结构（`Bidding`, `Playing`, `End`）、全部 `Event` 事件类型、全部 `View` 视图投影，以及 `Logic.scala` 中完整的 `transition` + `project` 逻辑。
- **手工实现的 Wire 序列化层：** 针对所有深层嵌套的 ADT（卡牌、手牌、阶段视图等）手写完整的 JSON 编码/解码逻辑——未使用任何自动派生。
- **从零搭建的同构 Scala.js 前端：** 完整的交互式 UI (`UI.scala`)，基于 `Scalatags` 构建，包含卡牌渲染、CSS 样式设计、客户端本地合法性预验证及响应式布局。
- **全面的测试套件：** 40 余个覆盖各阶段状态转移的单元测试，外加基于属性的 Wire 正确性测试。

## 🌟 个人具体贡献

作为三人团队中的核心开发者，我主要负责后端响应式状态机的控制流搭建、领域逻辑的模块化架构设计，以及高覆盖率的自动化测试体系建设。

#### 1. 状态机流转与安全视图投影

我实现了核心的 `transition` 和 `project` 方法——整个应用程序跳动的心脏。`transition` 编排了从发牌、叫牌、吃牌到最终算分的完整游戏生命周期，确保每一次状态变更都是确定性的且无冲突的。`project` 通过将全局 `State` 投影为特定于每位玩家的 `View`，执行严格的信息隐藏，从根本上保证了“战争迷雾”——客户端永远无法接收到不该看到的数据。

#### 2. 模块化架构：Builder 模式与拓展方法

我充分利用 Scala 3 的**拓展方法 (Extension Methods)** 并结合**建造者模式 (Builder Pattern)**（通过不可变 `copy` 链式调用），将庞大的游戏逻辑从底层的 ADT 数据定义中完全剥离。像 `chooseBid → toNextPhase → organizeCard → toNextActor` 这样的复杂操作被组合为干净的、可链式调用的状态变换。这种分离使得每个函数只关注单一职责，极大提升了 670 余行领域模型代码的可维护性与可测试性。

#### 3. 防卫式编程 (Defensive Programming)：穷举模式匹配与卫语句

我在 `transition` 的所有多分支事件处理器中系统性地推行了**穷举模式匹配 (Exhaustive Pattern Matching)**——由于每个游戏子阶段（`BiddingPhase.Choosing`、`PlayingPhase.Placing` 等）都是独立的编译期类型，Scala 编译器会直接拒绝任何遗漏的状态分支，从源头消除逻辑遗漏。与此同时，我在每个状态操作函数中贯彻了严格的**卫语句**（`require` 前置条件与 `ensuring` 后置条件），这些契约充当运行时"哨兵"，在非法输入（如不该行动的玩家、非法出牌）侵入内部状态之前将其拦截，充分体现了**防卫式编程 (Defensive Programming)** 的工程纪律。

#### 4. 基于属性的测试与单元测试

我独立设计并实现了整套**基于属性的测试 (Property-Based Testing)** 基建，使用 `ScalaCheck` 框架。通过为塔罗牌的完整领域（卡牌、花色、点数、事件、视图、阶段视图）构建自定义的 `Gen` 生成器，我使用大量随机生成的负载对 Wire 序列化层进行了地毯式的压力测试，最终实现了 JSON 编码/解码 **100%** 的分支覆盖。在 PBT 之外，我与一位队友合作开发了确定性的**单元测试套件**，覆盖了完整的游戏生命周期：初始状态验证、叫牌阶段转换、吃牌规则（`isValidPlay`、`roundWinner`、`roundScore`、`countOudlers`）、回合结算以及新局重启机制——使核心游戏逻辑的代码覆盖率达到 **90%+**。

## ⚖️ 项目署名

本项目为三人合作项目，由 **Steven Ji**，**Elsa Sánchez Fernández** 和 **Nicolas Raymond Karmolinski**，于 2025 年春季学期历时 3 周完成，属于洛桑联邦理工学院 (EPFL) 的 [软件构建 (Software Construction, CS-214)](https://edu.epfl.ch/coursebook/en/software-construction-CS-214) 课程项目（8 学分）。

### 知识产权与合规性说明
- **课程教材：** 所有基础框架、实验课题及底层代码的版权均属 © 2023–2025 EPFL 所有。本仓库严格遵守课程规定，不发布任何原始课程资料或源代码。
- **原创贡献：** 项目中的实现逻辑、优化的系统架构以及特定的功能扩展均为作者的原创知识成果。
- **第三方资产：** 牌面图像引用自 [FreeSVG](https://freesvg.org/deck-of-french-tarot-playing-cards)，并遵循其开放内容许可条款。
- **项目用途：** 本仓库仅作为个人作品集使用，旨在展示项目成果、架构设计及性能指标。

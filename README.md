<div align="center">

# 🃏 French Tarot: Full-Stack Scala Web Game

<p align="center">
  <b>English</b> | <a href="README_zh.md">简体中文</a>
</p>

> **Demonstration Only:** This repository serves as a showcase for an academic software construction project. If you are interested in discussing the implementation or technical details, please contact **<steven.ji@epfl.com>**.

![Scala](https://img.shields.io/badge/Scala-3-red.svg?style=flat-square)
![Scala.js](https://img.shields.io/badge/Scala.js-Frontend-blue.svg?style=flat-square)
![Architecture](https://img.shields.io/badge/Architecture-Centralized__State__Machine-brightgreen.svg?style=flat-square)
![Testing](https://img.shields.io/badge/Testing-ScalaCheck-orange.svg?style=flat-square)
![EPFL](https://img.shields.io/badge/School-EPFL-red.svg?style=flat-square)

*A fully-featured, multiplayer web implementation of the classic French Tarot card game, built entirely in Scala 3.*

</div>

## 📖 Overview

This project is a centralized web application designed to let 3 to 5 players play the traditional French Tarot card game online. It strictly enforces the complex rules of Tarot (including bidding, handling the dog, trick-taking constraints, and precise scoring) through a highly robust, purely functional domain model.

The application uses a unified Scala codebase, seamlessly sharing core game logic between the JVM backend and the browser frontend via Scala.js, ensuring 100% type safety across the entire stack.

### 📊 Project Statistics

- **Core Implementation**: ~2,100 lines of pure Scala 3 code across a unified codebase (`shared`, `jvm`, and `js` modules).
- **Engineering Quality**: **100% test coverage** on the custom network wire serialization and **90%+ unit test coverage** on all core deterministic game logic.
- **Domain Complexity**: The pure functional domain model consists of over **80 core business functions/extension methods**, with more than 40 exhaustive pattern-matching branches manually written for JSON encode/decode without relying on auto-derivation.

### 🏗️ Engineering & Functional Architecture

This project was built adhering to modern Software Construction (CS-214) principles, acting as a showcase for advanced functional programming and client-server architecture:

- **Centralized State Machine**: The server acts as the single source of truth. Game logic is expressed as a deterministic state machine transitioning between well-defined states (`Bidding`, `Playing`, `End`).
- **Purely Functional Domain Model**: Unlike OOP approaches that mutate state, the entire Tarot ruleset is implemented as pure functions on immutable data structures. State transitions (bidding, playing a card) return entirely new `State` copies via case class `copy`, and rich **Extension Methods** modularize complex rules (e.g., `isValidPlay`, `roundWinner`) away from the core data types.
- **Shared Code (JVM + JS)**: Core ADTs (`Card`, `Suit`, `Rank`, `Hand`) and game rules are defined in a `shared` module compiled for both JVM and JavaScript, ensuring 100% type safety across the full stack.
- **Strict Information Hiding (Views)**: To fundamentally prevent cheating, the server never sends the complete game state to clients. It projects the global `State` into player-specific `View`s so each client only receives data they are legally allowed to see.
- **Isomorphic Scala.js Frontend**: The UI is built entirely in Scala via `Scalatags` — no raw HTML or JavaScript written. It declaratively maps the strictly-typed `View` into DOM elements.

### 📐 Project Scope: Framework vs. Our Implementation

This project was developed on top of a minimal runtime skeleton provided by the EPFL CS-214 course. It is important to distinguish between what was given and what was engineered from scratch by the team.

**Provided by the course framework:**

- The underlying WebSocket communication engine and server bootstrap.
- Empty trait/interface signatures for `StateMachine` and `WebClientAppInstance`, defining *what* methods to implement but not *how*.

**Designed and implemented entirely by our team (3 members):**

- **Complete Domain Model:** All 78-card Tarot deck definitions, the five suits including Trumps, all scoring rules (Oudlers, face cards, half-point system), and the reshuffle-on-Petit-Sec mechanic.
- **Full State Machine Brain:** The entire `State` sealed hierarchy (`Bidding`, `Playing`, `End`), all `Event` types, all `View` projections, and the complete `transition` + `project` logic in `Logic.scala`.
- **Custom Wire Serialization:** Hand-written JSON encode/decode for all deeply nested ADTs (Cards, Hands, PhaseViews, etc.) — no automatic derivation.
- **Isomorphic Scala.js Frontend:** The full interactive UI (`UI.scala`) built from scratch with `Scalatags`, including card rendering, CSS styling, local move validation, and responsive layout.
- **Comprehensive Test Suite:** 40+ unit tests covering every game phase transition, plus property-based tests for Wire correctness.

## 🌟 My Specific Contributions

As a core developer in this three-person team, I was responsible for the backend reactive state machine control flow, the modular architecture of the domain logic, and the high-intensity automated testing infrastructure.

#### 1. State Machine Transitions & Secure View Projection (`Logic.scala`)

I implemented the core `transition` and `project` methods — the beating heart of the application. `transition` orchestrates the entire game lifecycle from dealing, through bidding and trick-taking, all the way to final scoring, ensuring every state change is deterministic and conflict-free. `project` enforces strict information hiding by projecting the global `State` into per-player `View`s, guaranteeing a "fog of war" where no client ever receives data they shouldn't see.

#### 2. Modular Architecture: Builder Pattern & Extension Methods (`types.scala`)

I leveraged Scala 3's **Extension Methods** combined with a **Builder Pattern** approach (via immutable `copy` chains) to decouple the massive game logic from the base ADT data definitions. Complex operations like `chooseBid → toNextPhase → organizeCard → toNextActor` are composed as clean, chainable state transformations. This separation keeps each function focused on a single concern, dramatically improving maintainability and testability of the 670+ line domain model.

#### 3. Defensive Programming: Exhaustive Pattern Matching & Guard Clauses

I systematically enforced **exhaustive pattern matching** across all multi-branch event handlers in `transition` — since each game sub-phase (`BiddingPhase.Choosing`, `PlayingPhase.Placing`, etc.) is a distinct compile-time type, the Scala compiler itself rejects any unhandled state, eliminating silent logic omissions entirely. I also applied rigorous **guard clauses** (`require` preconditions and `ensuring` postconditions) throughout every state-manipulation function, acting as runtime sentinels that intercept invalid inputs (e.g., wrong player acting out of turn, illegal card play) before they can corrupt internal state — a thorough application of **Defensive Programming** discipline.

#### 4. Property-Based Testing & Unit Testing (`Tests.scala`)

I independently designed and implemented the **Property-Based Testing (PBT)** infrastructure using `ScalaCheck`. By building custom `Gen` generators for the entire Tarot domain (cards, suits, ranks, events, views, phase views), I stress-tested the Wire serialization layer with hundreds of randomly generated payloads, achieving **100% branch coverage** on JSON encode/decode. Beyond PBT, I co-developed the deterministic **unit test suite** with a teammate, covering the full game lifecycle: initial state validation, bidding transitions, trick-taking rules (`isValidPlay`, `roundWinner`, `roundScore`, `countOudlers`), round-end scoring, and game restart mechanics — reaching **90%+ code coverage** on core game logic.

## ⚖️ License & Attribution

This project was developed by **Steven Ji**, **Elsa Sánchez Fernández** and **Nicolas Raymond Karmolinski** as a 3-person team collaboration over 3 weeks in the Spring 2025 semester for the EPFL course [Software Construction (CS-214)](https://edu.epfl.ch/coursebook/en/software-construction-CS-214) (8 credits). 

### Intellectual Property & Compliance
- **Course Materials:** All foundational frameworks, lab assignments, and base code are © 2023–2025 EPFL. In strict adherence to the course policy, no original course materials or source code are distributed in this repository.
- **Original Contributions:** The implementation logic, optimized system architecture, and specific functional extensions represent the original intellectual property of the authors.
- **Usage:** This repository serves solely as a portfolio showcase of the project's results, architectural design, and performance metrics.

---
layout: post
title: "Radio Unit 软件自动化测试架构与集成策略"
date: 2026-04-04
tags: [devops, 自动化测试]
toc: true
comments: true
author: ChenJiangang
---

# Radio Unit 软件自动化测试架构与集成策略

> **核心愿景：** Precision tested internally, flawlessly delivered globally. (内部精密验证，全球完美交付。)

---

## 1. 为什么构建Radio Unit的全景自动化测试体系？

为了确保Radio Unit软件的高质量交付，需要致力于实现以下三大目标：

| 目标维度               | 关键价值                                                     |
| :--------------------- | :----------------------------------------------------------- |
| **持续一致的交付质量** | 确保所有强制性基础功能无退化，向客户提供持续一致的优质RF软件。 |
| **极致的设备稳定性**   | 显著提升 RRU 稳定性，从源头上减少来自客户端的稳定性缺陷报告。 |
| **高效的 CI/CD 闭环**  | 为开发团队提供最快的时间到价值 (Time-to-Value) 反馈，加速产品验证周期。 |

---

## 2. Unit/L1 与 Pre-L3 集成策略

这里描述的是Radio Unit软件不同阶段的测试边界与协同关系。**我主要负责的是编写Pre-L3集成阶段的测试策略和用例、设计Pre-L3自动化测试架构、并与团队搭建Pre-L3自动化测试软件框架、撰写大部分自动化测试软件库并协调团队快速迭代**。

### 2.1 核心架构对比

| 维度             | Unit/L1 集成阶段                       | Pre-L3 集成阶段                               |
| :--------------- | :------------------------------------- | :-------------------------------------------- |
| **核心目标**     | 端到端特性集成验证与软件健壮性评估     | 评估底层就绪度 (面向 RAN/LTE/5G/Factory 发布) |
| **关键测试级别** | PIT, TE CIT, Unit/L1 回归与新特性验证  | Pre-L3 PIT, CIT, CRT (回归与安全测试)         |
| **测试环境重点** | 依赖 BBU 模拟器、评估板与早期数据路径  | 依赖 L3 E2E 传导模式，结合真实 UE 和仪器      |
| **最终交付目标** | 内部组件质量达标，保障 L1 调用逻辑无误 | 面向客户的 RF 发布                            |

![Unit L1与Pre L3集成策略](https://jockeycjg.github.io/images/rfsw_prel3_test/architecture-boundary.jpg)

---

## 3. 自动化测试执行链路

Unit/L1 阶段作为第一道防线，重点关注基本功能可用性。而Pre-L3 阶段承接的是 Unit/L1 的输出，在真实无线环境下进行高级别验收。

### 3.1 流水线全景

1.  **冒烟与验收测试 (PIT)**
    *   **角色**：第一道防线。
    *   **作用**：拦截低级致命错误，确保软件基本功能可用。
2.  **回归与持续集成 (CIT/CRT)**
    *   **角色**：功能守护者。
    *   **作用**：全面运行回归及健壮性测试，保障修改不破坏既有功能。
3.  **生产环境测试集成 (TE CIT)**
    *   **角色**：最终交付前哨。
    *   **作用**：针对 TRX, FHA 和 Unit 级别的自动化回归测试，输出至 Production TE。

![自动化测试执行链路](https://jockeycjg.github.io/images/rfsw_prel3_test/unit-l1-pipeline.jpg)

---

### 3.2 Pre-L3 测试执行流程

Pre-L3 测试包括Pre-L3功能集成的手动测试，还有PIT(Plane Integration Tests)、CIT(Continuous Integration Tests)、CRT(Continuous Regression Tests)以及软件稳健性测试，并且不同的测试阶段采用不同的测试策略。

*   **准入与基础集成**：承接 Unit/L1 输出。
*   **深度回归网络**：覆盖安全测试、接口认证及无线信号性能验证。

![Pre L3测试链路](https://jockeycjg.github.io/images/rfsw_prel3_test/pre-l3-pipeline.jpg)

### 3.3 Pre-L3 的四大核心测试能力域

| 核心功能就绪                                                 | 性能验证                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| • 小区射频发射开启 (Cell On Air)<br>• L3 控制面呼叫建立<br>• 高性能Beamforming开启 | • 用户面数据吞吐量 <br>• 天线波束赋形验证<br>• 测试模式下射频指标验证 |

| 健壮性与稳定性                                               | 生命周期维护                                                 |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| • 模块及站点复位恢复<br>• 载波锁定/解锁循环验证<br>• 冷启动恢复 | • 最大4个版本的兼容性升级<br>• RF SW 升级与回滚安全机制<br>• 版本回退保护 |

### 3.4 质量反馈速度

为了实现高效的反馈闭环，我们制定了严格的时间表：

| 测试类型              | 目标耗时   | 频率          | 价值                         |
| :-------------------- | :--------- | :------------ | :--------------------------- |
| **Smoke Test (冒烟)** | < 1 小时   | 24/7 不间断   | 最快速度发现阻断性问题       |
| **CIT (持续集成)**    | < 24 小时  | Continuous    | 完整的性能与回归日构建反馈   |
| **Acceptance (验收)** | < 3 小时   | 每周/ Half FB | 核心通信接口验证无退化       |
| **CRT/健壮性**        | 视内容而定 | -             | 深度压力测试与长期稳定性定型 |

---

## 4. 实验室架构：从早期验证到准生产模拟

### 4.1 Unit/L1 评估板与模拟器架构
利用模拟器 (X-Step, TAD-X) 和基于 Raspberry Pi 的自动化控制，在早期阶段无需完整物理网络即可验证 L1 逻辑。

![Unit L1架构](https://jockeycjg.github.io/images/rfsw_prel3_test/unit-l1-architecture.jpg)

### 4.2 Pre-L3 端到端传导模式架构
真实Radio设备在传导模式下与自动化环境无缝对接。采用商用终端与 MIMO 信道，结合高精度仪表，确保射频输出符合 3GPP 规范。

![Pre L3架构](https://jockeycjg.github.io/images/rfsw_prel3_test/pre-l3-architecture.jpg)

### 4.3 Pre-L3 自动化测试软件框架
Pre-L3自动化测试软件框架如下图。

![Pre L3模块框架](https://jockeycjg.github.io/images/rfsw_prel3_test/PreL3-Automation_PreL3-TA-Framework.jpg)

### 4.4 Pre-L3 自动化测试代码库结构
Pre-L3自动化测试代码库主要使用Python，Robot Framework，Jenkins Groovy，Shell等语言实现。

![Pre L3代码结构](https://jockeycjg.github.io/images/rfsw_prel3_test/PreL3-Automation_PreL3-Library-Structure.jpg)

---


## 5. 产品健壮性与稳定性

针对关键里程碑（如 NQ4），我们执行严格的极限测试策略，主要也是放在Pre-L3环境去实现。

*   **高频循环**：执行高达 **100次** 的完整 L3 Call 循环、冷复位与解锁操作，旨在拦截 80% 以上的偶发性启动失败。
*   **长时压力**：连续执行 L3 Call 压力测试 **48小时** (Release 7 扩展至 72小时)，期间抓取 300 次系统快照，探测内存泄漏与崩溃风险。

![健壮性测试策略](https://jockeycjg.github.io/images/rfsw_prel3_test/robustness-strategy.jpg)

---

## 6. 战略成效

*   **全链路贯通**：从 Unit/L1 的 BBU级模拟，到 Pre-L3 的端到端真实射频传导，测试链条完全贯通。
*   **极致效率**：CI/CD 高速运转，大幅缩短研发等待时间。
*   **零妥协体验**：凭借极限施压，将致命缺陷扼杀在内部实验室。


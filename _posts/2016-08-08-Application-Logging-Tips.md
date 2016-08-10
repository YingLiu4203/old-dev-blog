---
layout: post
title: Application Logging Tips
categories:
- Development
tags:
- Programming
---

## 为什么需用Logging
理解了logging的目的，那么所有的Tips都不难理解了。当软件变得复杂，尤其是分布式异步并行计算程序比如Cloud Native程序成为主流是，基于IDE单步执行的Debug无法工作。而且多数的bug通常发生在用户现场的生产系统，这时候能够像单步执行那样清晰的给出程序执行各个重要节点的完整状态就非常宝贵。我的经验证明，有了好的application logging，多数的debug工作就变得轻而易举：如果所有的关键步骤的状态数据都有记录，错误的原因也就清楚了。




[1]: https://www.javacodegeeks.com/2011/01/10-tips-proper-application-logging.html

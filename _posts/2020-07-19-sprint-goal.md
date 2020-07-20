---
layout: post
title: "Sprint Goal - Scrum Guide 中出现过 27 次"
category: [Scrum]
tags: [Scrum, Agile]
---

### 痛苦的 Daily Scrum
很多人深有感触，Scrum 里最「难」开的会之一就是早会。

要么成为汇报会，要么成为僵化会，要么成为大家都不想开的痛苦会。

如果你对早会的效果也有不满，不妨看看 Scrum Guide 里对早会 3 个问题的建议，想想它为什么这么建议？

> - What did I do yesterday that helped the Development Team meet the **Sprint Goal**?  
> - What will I do today to help the Development Team meet the **Sprint Goal**?  
> - Do I see any impediment that prevents me or the Development Team from meeting the **Sprint Goal**?

### Sprint 内的工作项可以修改吗？

十几年来都会持续地听到一个以讹传讹的说法：「一个 Sprint 内的工作项不能修改；实在不得已时，加一个工作项前必须移除一个工作项。」

我很早以前也是相信这种说法的，直到有一天意识到，这不是跟「敏捷宣言」说的「响应变化高于遵循计划」相违背吗？

我于是去看了下官方的 Scrum Guide，发现里面写的很清楚：

> During the Sprint:  
> - No changes are made that would endanger the **Sprint Goal**;   
> - Quality goals do not decrease; and,   
> - **Scope may be clarified and re-negotiated** between the Product Owner and Development Team as more is learned.

Sprint Goal 是不能被伤害的，也就是不能改变；但是 Scope 可以协商。

### 被忽略的 Sprint Goal
如果上面两段引用还不足以引起您对 Sprint Goal 的重视，那么我再引用一段：
> A Sprint would be cancelled if the **Sprint Goal** becomes obsolete.

如果 Sprint Goal 过时了，Sprint 都可以被取消。

所以，不是「两周」就叫 Sprint，两周后需要达到一个目的，才叫 Sprint。


先讲一个正面的故事。

有一次我们团队接到一个较为复杂的任务，必须在 9 周内完成。

我们的做法是：
1. 用了几天做了一个 POC，然后快速做了技术选型。
2. 剩下 8 周多的时间，我们分成 4 个 Sprint，把这个任务分解成了 4 个里程碑，分别作为每个 Sprint 的 Goal。
3. 每天的早会，整个团队关注的是这个 Sprint 的 Goal 能不能达成；而 Sprint 内的 Story，则根据每天的进度和学到的东西，不断地调整中。

最后的结果，这个任务完成地异常顺利。

这就是 Sprint Goal 的威力：
1. 它承载着项目的里程碑。每个 Sprint 都完成一个里程碑，这就是 Scrum 里 Incremental 的意思。
2. 它对齐了团队每个人的目标，而如何完成这个目标 （做哪些 Story，用什么样的顺序做），则交由团队自行决定和调整。

### 如何定义 Sprint Goal？

Sprint Goal 不是 Sprint Backlog 本身，而是通过完成 Sprint Backlog，达到了什么目的？

这个目的可以有三种类型：
1. Earn Business Value （实现了什么业务价值？）
1. Earn Architectural Value （实现了什么架构价值？）
1. Learn Something （团队学到了什么？）

上面提到的故事，我们的任务是让客户端无感知的前提下，迁移几组共计上百个 API，技术选型是 API Gateway。

而我们第一个 Sprint 的目标是通过把其中 1 个 API Gateway 部署到生产环境，并且完成所有客户端的联调，去证明我们的技术选型是可工作的。

这是很重要的一点。

容易犯的错是：前几个 Spint 的目标都是开发那些 API，但是直到最后一个 Sprint 才部署到生产环境，这就不是增量（Incremental），而是瀑布。

设置 Sprint Goal 的时候。
1. 前面的 1 个或几个 Sprint，往往要关注 RAID （Risk, Assumption, Issue, Dependency）的消除，这就是 Learn；
1. 后面的 Sprint 才是收获，要么收获业务价值（用户直接得利），要么收获架构价值（架构演进），这就是 Earn。

如果一个 Goal 设置下来，上述 3 种目的都没有达到，需要反思一下 Goal 设置得是否合适？

### 结尾
总结一下，
1. 设置 Sprint Goal，是让团队关注价值，关注里程碑，关注目的；而对手段（User Story 是手段）保留随时更新的选择。
1. 建议草拟往后若干个（而不是 1 个）Sprint 的 Goal，并且每个 Sprint 滚动更新。这是以终为始的思维。

把 Sprint Goal 用好是让 Scrum 免于僵化的一个突破口，您同意吗？
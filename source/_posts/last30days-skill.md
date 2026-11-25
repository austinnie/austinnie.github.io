---
title: "市场研究员不再手动扒平台，改用 last30day..."
date: 2026-11-26 05:02:11
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)


## 痛点

上周五要做一份关于某个加密项目的背景简报，我上午花了三个小时干这件事：依次打开 Reddit 找帖子讨论热度，切到 X 搜相关关键词，再去 YouTube 翻视频评论区，顺手看了眼 TikTok 有没有二创传播，最后还要去 Polymarket 查有没有人押注这个项目——因为那上面的资金流向比点赞数更能说明问题。

每个平台的搜索逻辑不一样，界面也不统一，手动复制粘贴整理完已经下午两点了。最麻烦的是没法同步对比：Reddit 高赞的内容和 X 上的风向经常完全相反，我得自己脑内打分，效率极低，还容易漏掉关键信息。

## 之前我们是怎么做的

第一步，先在 Google 搜「项目名 + reddit」，从结果里挑几个有Discussion的链接点开。第二步，回到 Google 把平台换成 site:x.com，再筛一遍。第三步，同样的操作给 YouTube、TikTok 各做一遍，TikTok 还得去网页版或者用第三方工具才能看到数据。第四步，把几个平台的结果复制到一个 Notion 页面里，用自己的表格交叉验证哪条信息多个来源都提到了。第五步，如果涉及资金动向，还得单独打开 Polymarket 的对应市场页看赔率和交易量变化。

整个过程没有自动化，每一步都是手动的，一个项目下来至少两三个小时起步，而且每次搜索的深度和覆盖度都不一样，很难说有没有遗漏什么重要信号。

## 为什么最后选了这个项目

一开始我考虑过两个替代方案。第一个是继续用 Google + 手动收集，优点是不需要任何配置，缺点是我刚说完，三个小时起步，还不一定能搜全。第二个是写 Python 脚本自己去调各平台的公开接口，听起来很工程师，实际上 Reddit 和 X 的 API 早就限流甚至收费了，TikTok 几乎没有官方 API，YouTube 的 data API 也有 quota 限制，Polymarket 更是需要直接读链上数据——写出来维护成本极高，而且换个项目又要改代码。

最后选 mvanhorn/last30days-skill，原因是 README 里说得清楚：Google 聚合编辑，这个工具搜索的是人。它通过 Bring Your Own Key 的方式一次性桥接了 Reddit、X、YouTube、TikTok、Polymarket、HN 等十几个平台，用一个命令并行跑，输出是一份综合简报。对于不需要长期维护脚本、只想快速出结果的市场研究员来说，这套方案的投入产出比明显更高。而且这个项目目前在 GitHub 上有将近 7.5 万的 star，社区活跃度很高，出了问题也更容易找到解答。

## 关键步骤

第一步，安装 skill。README 推荐了两条路径：如果用 Claude Code，加 marketplace 插件即可自动更新；如果用 Codex、Cursor、Copilot 或其他 Agent，执行 `npx skills add mvanhorn/last30days-skill -g` 全局安装。这一步的目的是把 tooling 准备好，后续才能在对话里直接调用 `/last30days` 命令。

第二步，补充平台认证。README 明确说了，Reddit、HN、Polymarket 和 GitHub 零配置即开即用，但 X（Twitter）、YouTube、TikTok、arXiv 等需要在 setup wizard 里填入各自的 API token 或登录 session。这一步很关键，因为缺少这些 key 就直接跳过了部分平台的搜索，输出会缺胳膊少腿。

第三步，输入查询命令，格式为 `/last30days <话题>`。例如 `/last30days Peter Steinberger`。系统会在后台并行检索所有已授权平台，按真实用户的 upvotes、likes 和真实资金投入量进行评分，再由 AI agent judge 合成一份简报。

第四步，阅读输出的综合简报。按 README 描述，简报会覆盖多个平台的话题热度排行、资金动向信号（特别是 Polymarket 的真实赔率变化），以及 AI 总结的核心观点分歧点。不需要自己再做交叉验证，agent 已经完成了一次基础的去重和评分。

## 跑完之后

按项目 README 的描述，最终得到了一份涵盖 Reddit、X、YouTube、TikTok、Polymarket 等多平台信息的综合简报，内容包括各平台的话题热度排序、真实用户参与度评分，以及基于 Polymarket 赔率的资金动向参考。对于那个加密项目的案例，原来需要我花三四个小时手动拼凑的信息，现在一份命令就能拿到结构化输出。

不过我也得诚实说几点：Polymarket 的数据反映的是"用钱投票"的信号，不一定代表舆论热度本身——比如某个话题在 Reddit 上吵得很凶，但在 Polymarket 上没人下注，这两者传达的是不同类型的重要程度，不能混为一谈。另外，X 和 YouTube 需要自备 API key，如果你的团队没有这些账号的开发者权限，这部分数据就出不来，简报会有缺口。

## 适合谁 / 不适合谁

适合：独立市场研究员、小型投资团队，需要频繁追踪某个话题在多平台上的传播态势和资金信号，希望用最少的时间拿到一份多源交叉的综合简报。不适合：需要长期稳定复用某套搜索流程的工程团队（建议直接写脚本或接入 API 管道）；对输出数据的学术级准确性有严格要求的场景（这个工具偏向情报速览，不是严谨研究）；手头没有任何平台 API key、也不想自己申请的人。

> 本文基于 mvanhorn/last30days-skill 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


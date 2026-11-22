---
title: "小型 team 怎么用 firecrawl 接 A..."
date: 2026-11-23 05:02:45
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点

我们是个两三个人的小创业 team，给 agent 喂数据。最早发现一个问题：每个新业务方向要爬一个网站 feed 进 agent，流程特别碎。得先搞定代理池，不然 IP 直接被封；然后调 Scrapy 抓 HTML；接着用 markdownify 之类的库转成 clean markdown；分页和 JS 渲染还要单独处理；最后把这些碎片拼到 agent context 里。整个链路至少涉及 3-5 个不同的服务或者脚本，而且没有一个现成 API 同时提供「爬取 + 干净 markdown 输出」这两件事。

每次新开一条数据线，从立项到能跑通端到端测试，大概要花 3-5 个小时调试基础设施工具链本身的问题，而不是真正在写业务逻辑。这种痛苦积累起来特别影响迭代速度。


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

1. Spin up scraping infra with Python/Scrapy——先在本地搭一套基础环境，装依赖、配中间件、搞重试逻辑。
2. Add proxy pool & rate-limiting logic——代理池要么自己维护，要么接入第三方服务，rate limit 策略要根据目标站点一个个适配。
3. Write HTML-to-markdown pipeline——写解析脚本，处理各种奇怪的页面结构，有时候还得手写 CSS selector。
4. Manually handle pagination & JS rendering——分页逻辑手动实现，JS 渲染的网站单独开 playwright/selenium。
5. Stitch results into agent context——把所有数据统一格式化，塞进 prompt 或者 memory。

每一步都没什么技术含量，但加起来就是几个小时的重复劳动。而且每次换新站点都要改一堆东西。

## 为什么最后选了这个项目

一开始考虑过两个替代方案。第一个是自己维护一套 crawlarium-based 的基础设施，但那样就完全是在重复造轮子，和我们现在的 tech debt 一样大。第二个是用第三方 SaaS 服务比如 Bright Data 或 Oxylabs，功能确实强，但价格贵而且没有统一的 markdown 输出能力，还是要自己包装一层。

firecrawl/firecrawl 吸引我们的地方在于它声称能「Run Firecrawl on your own machine as one binary that turns websites into clean markdown, plugs into your AI agent as an MCP server」。也就是说，理论上跑一个命令就能拿到 agent 能直接用格式，不用再折腾 HTML→markdown 的转换。而且它是开源的，可以 self-host，对我们这种在意数据隐私的小 team 比较友好。

还有一个原因是它有 MCP server 集成选项，可以直接插进 Claude Code、Cursor、Windsurf 这些工具里。我们刚好在用 Cursor，所以如果能直接接进去会省掉很多胶水代码。

## 关键步骤

根据 README 的描述，大致思路是这样：

第一步，安装。如果是本地运行需要用 Rust，`cargo install firecrawl-local`；如果不装 Rust，可以用 `npx -y firecrawl-cli@latest scrape <url>` 直接试水。这部分比较简单，就是环境准备。

第二步，测试单次抓取。`firecrawl-local scrape https://example.com --markdown` 应该能拿到干净的 markdown 输出。这一步主要是验证配置是否正确，以及目标站点的兼容性。

第三步，如果要做全量爬取，用 crawl 命令配合 limit 参数限制规模：`firecrawl-local crawl <base-url> --limit 50 --out ./pages`。这样会在指定目录生成多个 markdown 文件。

第四步，配置 MCP server 让 AI agent 能用。README 提到有 `init` 命令会自动检测并配置主流工具，不需要手动写配置文件。

第五步，把生成的 markdown 文件作为 agent 的知识库 feed 进去。这一步取决于具体用的 agent framework，firecrawl 只负责提供数据源。

## 跑完之后

按 README 描述，Firecrawl Local 可以做到用一个二进制文件或一次 API 调用就返回 clean markdown，这对于我们来说是最核心的价值——不用再接管那个繁琐的 HTML-to-markdown pipeline 了。optional MCP server plug-in 的设计也意味着理论上可以不经过任何中间层直接把结果喂给 agent。

关于 Alexandria 的 550+ data sources，README 里说这是 Cloud 版本的能力，Local 版主要依赖自带的 scrap/crawl 功能。如果要大规模或者需要搜索能力，可能需要走 Cloud API。

对于小规模的内部知识库场景，单靠 Local 模式应该够用。但如果要爬的动态站点特别多、或者对并发有要求，可能会遇到瓶颈——这点 README 没细说，我也没实测。

## 适合谁 / 不适合谁

适合那种想要快速 prototype、不太想维护复杂爬虫基础设施的小型 team。特别是已经在用 Cursor 或者 Claude Code 这类支持 MCP 的工具时，integration cost 会很低。

不适合的场景包括：已经有一套成熟爬虫系统的大 team（迁移成本高）、对数据主权极其敏感必须完全 offline 的环境（Cloud API 依赖外网）、以及需求极其复杂的动态站点解析（可能需要自行定制 parser）。

另外要注意，这个项目 star 数很高（20w+），说明社区关注度大，但也意味着 issue 可能很多，遇到问题要找答案不一定容易。

> 本文基于 firecrawl/firecrawl 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


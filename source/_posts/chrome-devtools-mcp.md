---
title: "用 chrome-devtools-mcp 让 A..."
date: 2026-11-17 05:02:43
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点：每次查性能都像是在考古

最近在跟一个旧项目重构，里面有个页面加载特别慢。我想搞清楚问题出在哪——是某个 API 请求卡住了，还是 JS 执行时间太长，或者是哪个资源没命中缓存。

但每次打开 Chrome DevTools 手动翻 Network、看 Performance tab、切 Console 看报错，一套下来至少二十分钟。更难受的是，这些问题往往发生在特定操作步骤下，我得一遍遍复现、一遍遍抓包，有时候还抓不到关键节点的报错。

后来我就在想，既然 Claude/Cursor 这类 AI 助手都能直接读我的代码了，能不能也让它帮我看一眼浏览器的实时状态？


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

以前遇到这种问题，大概是这样处理的：

1. 在目标页面上手动打开 DevTools，切换到 Network 面板，刷新或操作触发问题
2. 盯着一个个请求看，手动筛选 XHR/Fetch，找那个耗时异常的
3. 切到 Performance 面板重新录一段 trace，然后再慢慢分析火焰图
4. Console 里单独查看报错，有时候报错信息还跨了好几屏
5. 把截图或者控制台内容复制出来，粘贴到 Cursor 里让它帮忙分析

整套流程走下来，光是等待和操作就耗掉大半天。最坑的是，有时候 AI 给出的建议还是基于我粘贴的那一小段信息，容易漏掉上下文。

## 为什么最后选了这个项目

一开始我也考虑过两个替代方案：一个是直接用 puppeteer 脚本录制和收集数据，另一个是让 AI 通过 DOM 选择器去定位问题元素。

puppeteer 方案的问题在于，我要自己写一堆脚本去模拟操作、采集数据、然后传给 AI 分析，维护成本高，而且很难复用。DOM 选择器方案就更局限了，只能拿到页面渲染结果，拿不到网络请求详情和性能 trace。

chrome-devtools-mcp 不一样，它是一个 MCP Server，可以直接让 AI 助手调用 Chrome DevTools 的能力——分析网络请求、截图、看 console 日志、录制性能 trace。这意味着 AI 可以直接看到浏览器内部的状态，而不是靠我粘贴过去的碎片信息。

虽然它也有局限，但至少把"AI 能看到什么"这个问题解决了。

## 关键步骤

按 README 的描述，整体思路是这样的：

**第一步：安装并配置 MCP Server**

在项目里加一段 MCP 配置，指向 `chrome-devtools-mcp`：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

这一步的目的就是让 AI 编码助手（Cursor / Claude Desktop 之类）知道有一个叫 `chrome-devtools` 的工具可以用。

**第二步：启动 Chrome 并连接**

通过 MCP Server 提供的工具集，AI 可以自动连接到本地运行的 Chrome 浏览器实例。这里不需要手动写启动命令，Server 会处理连接逻辑。

**第三步：让 AI 执行具体任务**

比如在 Cursor 里直接说："帮我打开 http://example.com，看看 Network 面板有没有异常请求，顺便截张图"。AI 会通过 MCP 工具调用对应的 DevTools 接口来完成。

根据文档描述，可用的工具包括：分析网络请求、取截图、读取 Console 消息（支持 source map 还原）、录制 Performance trace 等。这些操作都是异步执行的，MCP Server 会自动等待结果返回。

**第四步：拿到结果后继续迭代**

AI 拿到浏览器数据后可以给出初步判断，如果觉得不够，可以继续追问，比如"trace 里哪个 segment 耗时最长"或者"console 里有没有 warning"。

整个过程不再需要手动切面板、复制粘贴。

## 跑完之后

说实话，我自己还没把这个流程完整跑通一遍。按 README 的描述，这个工具应该能做到以下几件事：

- **分析网络请求**：可以拿到页面的 XHR/Fetch 列表，查看每个请求的状态码、耗时、资源大小等信息
- **截图**：对当前页面截取可视区域或全页的截图
- **读取 Console 消息**：获取浏览器控制台的日志输出，且支持 source map 还原，能看到原始 TypeScript/JSX 的代码位置
- **录制 Performance trace**：记录一段时间内的性能数据，用于分析帧率、JS 执行、渲染耗时等

文档里提到它还支持 CrUX 真实用户体验数据的查询，可以在 lab 数据之外补充 field data，帮助判断性能问题的实际影响范围。

不过需要注意的是，这个工具只能访问它连接的 Chrome 实例里的内容，所以敏感信息不要在那个浏览器里登录。另外官方只保证对 Google Chrome 稳定版的支持，其他 Chromium 内核浏览器可能会出问题。

还有统计相关的问题：默认情况下 Google 会收集工具调用成功率、延迟等使用数据，如果想关掉可以通过传 `--no-usage-statistics` 参数或者设置环境变量来退出。

## 适合谁 / 不适合谁

**适合：**
- 已经深度依赖 Cursor / Claude 这类 AI 编码助手的开发者，想把手动调试的流程也自动化的人
- 小型团队或个人项目，希望减少重复性调试工作量的场景
- 对性能分析和网络问题排查有一定需求，但不想写大量自动化测试脚本的情况

**不适合：**
- 需要调试生产环境线上问题的场景——这个工具连接的是本地 Chrome 实例，没法远程连
- 有大量敏感信息需要操作的场景——连接后的浏览器内容会被 MCP Client 读到
- 期望它能替代专业性能分析平台（Lighthouse CI、WebPageTest 等）的场景——它更适合日常开发中的快速排查
- 不用 Chrome 的——Firefox/Safari 不在支持范围内

> 本文基于 ChromeDevTools/chrome-devtools-mcp 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


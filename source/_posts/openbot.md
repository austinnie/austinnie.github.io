---
title: "让 AI 同事住进你的电脑：OpenBot 来了"
date: 2026-10-27 07:57:05
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 什么是 OpenBot

你有没有想过，如果让 AI 助手像真正的人类同事一样，在你电脑上一直待命、有自己的工作空间、还能互相协作，会是什么样子？OpenBot 就是这个想法的实现。

它是一个本地优先的桌面应用，专门为 AI Agent 打造了一个持久化的工作环境。不同于每次打开网页才能聊天的 ChatGPT，OpenBot 让 Codex、Claude、Grok 等 AI 助手常驻在你的电脑上，拥有独立的工作目录、对话历史和任务队列。


![illustration](/images/illustration_03.png)

## 核心功能亮点

OpenBot 最吸引眼球的功能是它赋予每个 AI Agent 独立工作空间的特性。你可以为不同任务创建不同的 Agent，每个 Agent 拥有自己的上下文、文件和对话记录。就像办公室里每位同事都有自己的工位一样，这样既能隔离风险，又方便管理。

任务队列系统是另一个亮点。你可以通过 FIFO 队列给 Agent 派发任务，支持暂停、恢复、取消，而且即使程序崩溃，任务状态也能安全保留。对于想要让 AI 自动处理一系列任务的场景，这个功能非常实用。

文件共享和浏览器控制也值得称道。Agent 之间可以互相传递消息、图片甚至文件，还能通过内置浏览器进行网页操作。更有趣的是 Agent 间的协作机制——一个 Agent 可以把任务委托给另一个 Agent，两者共享任务历史并协同完成。

## 多模型支持，一个客户端搞定

OpenBot 的一大优势是对多种 AI 模型的统一支持。它不仅对接了 OpenAI 的 Codex、Anthropic 的 Claude、xAI 的 Grok，还兼容了 Gemini、OpenCode、Cursor CLI 和 Cline CLI。

这意味着你不需要在多个客户端之间来回切换，一个 OpenBot 就能管理所有 AI 助手。如果你搭建了本地的 Ollama 或 LM Studio，OpenBot 也能自动检测并使用这些本地模型。对于注重隐私和数据安全的用户来说，本地优先架构非常重要。

支持 Computer Use 功能也是一个加分项。在 macOS、Windows 和 Linux 上，Agent 可以控制你的电脑完成各种操作，这让 AI 的应用场景从单纯的文本处理扩展到了真正的系统自动化。

## 技术栈与跨平台支持

从技术角度来看，OpenBot 采用 TypeScript 开发，基于 Electron 框架实现跨平台桌面应用，前端使用 SolidJS 构建。这种技术栈选择保证了良好的性能和用户体验，同时也让应用能够在 macOS 13+、Windows 10+ 和 Linux（x64/arm64）上运行。

数据存储完全本地化，用户的隐私得到保障。同时提供可选的账户系统，通过一次性邮箱验证码登录，后端运行在 Cloudflare Workers 和 D1 数据库上。

## 使用场景与注意事项

想象一下这样的场景：早上出门前，你把代码审查任务交给 Codex Agent，下班后查看结果；或者让 Claude Agent 在后台帮你整理文档，同时 Grok Agent 在研究某个技术方案。多个 Agent 各司其职，通过消息系统互通有无，这就是 OpenBot 带来的工作方式。

不过需要注意的是，当前版本处于开发预览阶段，Agent 在首次启动后拥有一定的自主操作权限，可以读取和修改文件、执行命令和使用网络。官方建议只运行你信任的 Agent，做好备份，并仔细阅读安全说明后再使用。

对于那些希望将 AI 深度融入日常工作流的开发者来说，OpenBot 提供了一个值得期待的方向。虽然还在早期阶段，但它的理念已经足够吸引人。

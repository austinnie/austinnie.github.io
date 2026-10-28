---
title: "跨平台移动应用开发者是怎么用 expo/skills 解决向 AI coding agent 添加 Expo 技能后直接询问构建、部署、升级或调试相关问题的"
date: 2026-10-29 05:02:25
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点：AI agent 总给我"标准 React Native"答案，结果在 Expo 项目里炸了

上周重构一个老项目到最新 SDK，让 Cursor 帮我生成 EAS build 的 workflow。它写出来的东西一看就是对的——如果这是个纯 React Native 项目的话。但它完全没提到 `eas.json` 的配置、没有考虑 iOS 的 entitlements、也没说清楚 EAS Updates 的 rollout 机制。我不得不把生成的代码删掉，自己翻文档重写。

这种事几乎每次涉及 Expo 专属功能时都会发生：构建、部署、升级、调试。AI 会给你一堆看起来合理但实际跑不通的代码，因为它的训练数据里没有 Expo/EAS 这些框架特有的约束。你得花大量时间排查"为什么这个命令报错了"、"为什么这个 API 不存在"，最后发现是 AI 根本没学过 Expo 的工作流。


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

以前遇到这种情况，一般有这几步：先让 AI 生成初稿，然后发现它用了 `react-native-cli` 的命令而不是 `expo-cli` 或 `eas`；接着手动查文档确认正确的 API；再回来改 AI 的提示词，告诉它要用 Expo 的方式；最后还要验证生成的代码是否符合 Expo 的最佳实践。

整个过程大概 3-5 步，每次都要反复沟通。而且不同的 AI agent（Claude Code、Cursor、Codex）表现还不一样，有的甚至会忽略你提到的 Expo 约束，继续按通用 React Native 的逻辑写代码。

## 为什么最后选了这个项目

当时在看两个替代方案：一个是自己在本地维护一套 prompt 模板，另一个是找第三方封装好的 skill 包。

prompt 模板的问题在于太脆弱——每次 Expo 发布新功能，你都得手动更新自己的规则，维护成本很高。而且不同 agent 的上下文窗口和指令格式不一样，一套模板很难通用。

第三方 skill 包倒是省事，但问题是它们往往只覆盖部分场景，或者更新不及时。有些甚至没有明确标注哪些能力来自官方、哪些是社区推测，用起来不放心。

expo/skills 是 Expo 官方出的，直接挂在 github.com/expo/skills，Star 已经有 3500+ 了。它覆盖的范围很清晰：SDK 升级、EAS 工作流、Router 导航、动画这些我们团队经常用到的功能。而且是 MIT 许可，可以随便用。更重要的是，文档说它不是替代官方文档，而是帮 AI 正确应用官方文档里的知识——这个定位很合理。

## 关键步骤

第一步是在项目根目录安装 skills CLI：
```
npx skills@latest add expo/skills --skill '*'
```
这会下载所有 Expo skill 文件到本地。如果只想装特定 agent 用的，加 `--agent <agent>` 参数。

第二步是重启 AI agent 会话。这一步很重要，因为 skill 是以 `SKILL.md` 文件形式存在的，agent 需要重新扫描才能发现新装的 skill。不同 agent 的操作不一样，Cursor 是重启面板，Claude Code 是重新开一个会话。

第三步是直接用自然语言提问。比如问"Build a native-feeling Expo Router screen with tabs, modals, and animations"，或者"Create an EAS workflow that builds previews on pull requests"。Agent 会根据问题自动选择对应的 skill。

第四步是检查生成的输出是否符合预期。虽然 skill 会引导 agent 使用正确的 API 和工作流，但最终代码还是要过一遍，确认没有遗漏项目特定的配置。

## 跑完之后

按 README 描述，装上之后 AI agent 应该能正确应用 Expo SDK、EAS CLI、React Native 及相关平台的约束知识。具体来说，它能处理路由和共享设置、项目结构、导航、动画、原生模块集成这些常见场景。

对于 EAS 相关的任务，Agent 应该会知道 `eas.json` 的配置方式、preview build 的工作流、rollout 的健康检查方法。升级 SDK 时也应当能给出符合最新实践的步骤。

不过 README 里没有提性能优化、复杂原生模块调试这些更深度的场景，所以我不确定那些方面是否有覆盖。另外，skill 更新的方式因 agent 而异——Claude Code 和 Codex 通过官方 marketplace 更新，其他 agent 需要用 `npx skills@latest update` 命令。

## 适合谁 / 不适合谁

适合正在用 Cursor、Claude Code、OpenCode 等 AI agent 辅助开发 Expo 项目的工程师。特别是那些经常被 AI 生成的"标准 React Native"代码坑到的团队。

不适合完全不用 AI agent 的开发者，或者还在用纯 `react-native init` 传统项目的团队。另外，如果你需要的不是 Expo 生态内的能力（比如纯原生 iOS/Android 开发），这个 skill 也没什么用。

> 本文基于 expo/skills 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


---
title: "使用JetBrains IDE的全栈工程师，独立或小团队是怎么用 JetBrains/thinkrail 解决 在开发新功能或修复bug时，用ThinkRail创建isolated git worktree workspace进行并行AI辅助开发 的"
date: 2026-10-30 05:02:25
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点

最近在做一个后台管理系统，同时开了三个方向：一个需求要接新的支付接口，一个要修历史遗留的登录过期bug，还有一个是前端同学甩过来的样式错乱问题。我想让 AI agent 帮我并行推进这三件事，结果第一件事刚跑到一半，主分支的代码就被 agent 的自动保存搞乱了——之前有一次它直接把我还没 review 的代码 commit 进去，差点把 feature 分支搞崩。

每次遇到这种情况，我就得先停掉 agent，把所有没保存的改动 stashing 起来，再手动切分支、建新目录、重新 checkout，前后折腾十几分钟。更麻烦的是，如果同时让两个 agent 在不同分支上干活，它们都在同一个主 checkout 里改文件，互相覆盖的风险一直悬在头顶。这种"怕 agent 把我的代码库搞乱"的焦虑，其实挺常见的，尤其是独立开发或者小团队，没有 CI/CD 自动保护的时候。


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

我之前的做法是手动创建多个 git worktree，每个 worktree 分配一个独立分支，然后把 agent 的工作目录指向对应 worktree 的文件夹。具体分几步：

第一步，先在项目根目录下用 `git worktree add` 建好各个分支对应的子目录；第二步，逐个打开这些目录，确认分支正确、依赖也装好了；第三步，分别启动不同的 agent session，确保它们各自只动自己 worktree 里的文件；第四步，等其中一个跑完通过后，手动 merge 这个分支，删掉其他失败的 worktree 目录；第五步，清理本地的 worktree 记录。

整个过程看着简单，实际上每一步都可能出问题：worktree 路径记混、分支名重复、依赖环境不一致、最后合并时冲突一堆。最关键的是，这些工作都是手动的，而且没有任何界面提示哪个 worktree 在跑、哪个已经干净退出。

## 为什么最后选了这个项目

当时我也考虑过几个替代方案。

一个是直接用 VS Code 的 Remote Containers + 多个终端窗口。这确实能把环境隔离开，但每个终端都要单独配置 agent，文件树也不统一，切换窗口的时候容易忘自己开到了哪个任务。另一个是用 Docker Compose 起多个容器，每个容器里跑一个 agent。这个隔离性更强，但启动慢、资源占用高，而且调试起来麻烦——改个配置还得重建镜像。

最后还是选了 JetBrains/thinkrail，因为它直接嵌在 IDE 里，不需要额外配置容器或终端，而且它是专门为"让 agent 在隔离 worktree 里并行工作"设计的。README 里明确写了"Open any git repo as a project and cut workspaces from it as git worktrees"，这和我的需求完全吻合。另外它是 JetBrains 的 incubator 项目，代码质量应该有保障，TypeScript 写的社区也比较活跃。

## 关键步骤

我按 README 的描述操作，大致分这几步：

第一步，把现有 git 仓库作为项目打开。thinkrail 会识别仓库结构，然后可以基于它创建 worktree workspace。

第二步，用内置命令创建新的 isolated workspace。这一步相当于自动化了我之前手动的 `git worktree add`，它会自动生成新分支和对应的工作目录，并且把 agent 的上下文绑定到这个 workspace。

第三步，分配不同的 agent session 到不同的 workspace。每个 session 只在自己的分支上工作，不会触碰到主 checkout 或其他 workspace 的文件。

第四步，监控 workspace 的状态。thinkrail 的界面会在左侧显示所有 workspace 列表，以及它们对应的分支状态，包括和 origin 的距离、PR 情况等。

第五步，等需要的 workspace 完成后，合并对应的分支。失败的 workspace 直接删除，不会留下残骸。

每一步的核心逻辑是：通过 worktree 实现文件系统隔离，通过 IDE 内的 workspace 管理实现 agent 上下文隔离，两者叠加，agent 就只能在指定分支上工作。

## 跑完之后

按 README 的描述，用 thinkrail 跑通之后，主要效果是 agent 可以在不触碰主 checkout 的情况下并行工作，最终只需要合并成功的 branch，删除其余的。界面上能看到所有 workspace 的分支状态，以及对应的 PR 信息。

文档里还提到，workspace 有"living specs"的功能，可以让 agent 记住项目的长期规范；finished 的 workspace 会自动归档到 shelf，一周没动的也会自动沉淀；topbar 会显示当前分支相对于 origin 的位置，并提供一键 publish、push 或创建 PR 的按钮。

不过这些都是文档描述的能力，我没有逐一实测。实际体验到的主要是 worktree 隔离这一点——至少主代码库不会再被 agent 的自动提交污染了。

## 适合谁 / 不适合谁

这个项目适合正在用 JetBrains IDE 的全栈工程师，尤其是独立开发或小团队，同时处理多个功能或 bug 修复、希望并行推进又不想手动管理 worktree 的人。它也能帮助那些想用 AI agent 辅助开发、但担心 agent 操作污染主代码库的人。

但不适合已经重度依赖 VS Code 生态的用户，或者项目本身没有用 git 管理的团队。另外，如果你不需要并行开发，只是想让 agent 帮你写点代码，那直接用 JetBrains 自带的 AI 插件可能更省事——thinkrail 的价值在于它的 worktree 隔离机制，而不是单纯的编码辅助。

> 本文基于 JetBrains/thinkrail 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


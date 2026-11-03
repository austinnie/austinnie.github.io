---
title: "用 agent-desktop 让 Finder ..."
date: 2026-11-04 05:02:33
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点

最近负责一套桌面自动化场景，目标是让机器人在 macOS Finder 里完成一组文件整理操作。Finder 窗口里有 59 个交互元素，要逐个点击、输入、拖拽。我用的是传统方案——截图拿坐标、像素匹配找元素、点击或输入、再截图验证，循环往复。

具体动作是：先截一张 Finder 全屏图，在图里找到「下载」文件夹的坐标，点下去，再截图看是否展开，再找子文件夹坐标，再点，再截……59 个元素，每个都要经历截取 → 定位 → 点击 → 验证这个闭环。有一次因为屏幕分辨率变了，所有坐标全部失效，整条流水线要重写。算下来单轮跑完要两三个小时，而且每次 UI 微调都得重新标注。


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

1. **截取屏幕截图**：用原生库或者 PIL 截 Finder 窗口的当前画面。
2. **通过像素坐标定位目标元素**：提前在 Figma 里标好 59 个元素的坐标，或者用模板匹配在截图里找。
3. **执行点击或输入操作**：用 PyAutoGUI 之类工具往那个坐标丢一次点击或键盘事件。
4. **再次截图验证结果**：点完再截一张图，肉眼或脚本判断元素有没有变化。
5. **循环处理下一个元素**：重复上面的步骤，直到 59 个元素全部操作完。

整套流程写出来是个死循环，代码行数不少，维护成本更高。最烦的是第 4 步，截图验证经常误判，比如元素位置变了但颜色没变，匹配依然通过，实际上操作已经失败。

## 为什么最后选了这个项目

我对比了两个替代方案：一个是继续走截图坐标路线，另一个是纯 OCR 方案（用 Tesseract 识别界面文字再操作）。

截图坐标方案的缺点前面说了，坐标漂移、分辨率依赖、元素变化要重标。OCR 方案能拿到文字，但 Finder 的无障碍树里很多元素没有文字名，只有 role 和 state，OCR 识别率很低，而且每次识别都要跑模型，速度慢、不稳定。

agent-desktop 走的是无障碍树路线，macOS 上对应 AX API。它直接读系统树，返回 JSON，每个元素带 ref ID。不需要截图，不需要坐标，元素变化只要 ref 不变就能操作。而且它是 CLI，可以嵌进现有 agent 循环里，不用改架构。权衡下来，无障碍树方案最适合 Finder 这种复杂 UI 场景。

## 关键步骤

第一步，安装并授权。全局装 npm 包，然后跑 `agent-desktop permissions --guide`，按引导给 Terminal 开 Accessibility 权限。这是 macOS 的安全要求，不开就没法读无障碍树。

第二步，snapshot 拿元素列表。命令是 `agent-desktop snapshot --app Finder -i`，加上 `-i` 只返回可交互元素。README 里示例返回了 `ref_count: 59`，和 Finder 窗口的元素数量对上了。输出是 JSON，每个元素有 ref_id，比如 `@s8f3k2p9:e8`。

第三步，用 ref 操作元素。命令是 `agent-desktop click @s8f3k2p9:e8`，直接把第二步拿到的 ref 传进去。注意 ref 要加引号，PowerShell 里 bare `@token` 会被当成 splatting operator 报错。

第四步，再次 snapshot 验证。操作完再跑一遍 snapshot，看 ref 对应的元素状态有没有变化。如果 UI 变了，ref 可能失效，这时候会返回 `STALE_REF` 或 `AMBIGUOUS_TARGET`，需要重新 snapshot 再操作。

## 跑完之后

按 README 描述，这套流程跑下来，Finder 的 59 个交互元素可以逐个通过 ref 操作，不需要截图和坐标。每个 snapshot 返回结构化 JSON，agent 可以直接解析 ref_id 去点击或输入。操作完再次 snapshot 可以验证结果，形成闭环。

README 里还提到，refs 携带 snapshot id，裸 ref 会报 `INVALID_ARGS`；UI 变化后会返回 `STALE_REF`，需要重新 snapshot。操作默认是 headless 的，不会移动鼠标也不改变焦点，需要物理输入可以加 `--headed` 参数。这些特性在复杂场景下应该能减少很多边界 case。

不过我没有完整跑通整个 59 元素的操作循环，单步 snapshot 和 click 按文档描述是通的。具体到批量操作的稳定性和错误恢复，还得在实际项目里验证。

## 适合谁 / 不适合谁

适合正在做 macOS 桌面自动化的工程师，尤其是需要操作复杂 UI（如 Finder、系统偏好设置）的场景。如果你的 agent 已经用 JSON 驱动，接入这个 CLI 成本低。

不适合追求跨平台一致性的团队，因为每个平台的无障碍 API 不同，行为可能有差异。也不适合需要视觉反馈的场景，因为它不截图，调试时看不到实际画面。还有，如果目标应用的无障碍树不完整，部分元素可能拿不到 ref，这时候还是要 fallback 到截图方案。

> 本文基于 vercel-labs/agent-desktop 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


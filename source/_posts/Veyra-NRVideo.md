---
title: "用 Veyra 给 PS5 串流加画质增强"
date: 2026-11-14 05:02:57
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点

我自己在家有一台 PS5，偶尔也想把 `GTA6`（等等，它还没出，但逻辑上同理）这种大作拿到 Windows 上看。目标是：通过采集卡把 PS5 的 HDMI 信号接到 PC，用 RTX 系列的 DLSS 超分辨率 + 帧生成，把输出提升到接近 120FPS 的体验，同时还能顺手录下来。听起来很美好对吧？实际上每次开播前折腾采集卡 + 播放器 + 录屏软件的组合，光是把画面调成"看着还行"的状态就得花三五分钟，还经常因为顺序不对导致延迟飙升或者直接黑屏。


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

说实话，以前我基本是靠"拼积木"的方式在搞：

1. **启动 PS5 游戏**，等它跑进主菜单。
2. **打开采集卡驱动**（我用的是 AVerMedia），把 HDMI 信号接进来。这一步通常要确认一下信号有没有丢帧。
3. **在 PotPlayer 里手动调画质参数**，亮度和对比度靠感觉拉，色域也要对齐。
4. **单独开录屏软件**，比如 OBS 或者 Bandicam，设置编码参数，有时候还得额外挂一个滤镜插件。
5. **全程没有实时预览的统一控制面板**，想要微调画质就得来回切换窗口，确认延迟也得看任务管理器里的帧时间图表。

加起来少则 3-5 步，每一步还可能出错，耗时不说，体验也差。

## 为什么最后选了这个项目

其实我也试过别的路线。有两个比较常见的替代方案：

- **NVIDIA Broadcast / Reflex**：这套方案主打低延迟和 AI 降噪，但对采集卡输入的增强支持比较有限，没法像 GPU 管线那样直接叠加 DLSS 超分和帧生成，本质上还是偏音频和视频通话优化的方向。
- **OBS + NvfBCapture 滤镜**：自由度很高，社区生态也好，但你得自己配一套完整的滤镜链，而且录制的渲染路径比较长，延迟往往不如专门的实时管线来得干脆。

最后选 Likely7/Veyra-NRVideo 的原因很简单：**它就是为了解决"把采集卡信号和 PC 串流连在一起，做一次实时 GPU 增强"这件事的**。README 里直接把 `Capture · PS5 / Xbox / PC Streaming` 写进了功能列表，而且底层用的是 DirectX 12，理论上延迟更可控。对我来说，省掉拼积木的时间比啥都强。

## 关键步骤

下面是我按照 README 指引大致走通的核心步骤，没贴完整代码，用的是命令名和概念化的伪指令：

1. **下载安装 Portable 版本**  
   去 GitHub Releases 页面下载 `veyra-*.win-x64.zip`，解压就行，不需要装系统级的运行时。
2. **配置采集卡输入源**  
   在界面左侧 Source 栏点 `Capture`，选择对应的 HDMI 输入设备（比如 AVerMedia Capturer）。README 提到支持 PS5 / Xbox / PC 串流，所以这里直接指向采集卡即可。
3. **开启 Super Resolution + Frame Generation**  
   切换到 Enhancement 面板，找到 `Super Resolution` 和 `Frame Generation` 开关。按文档描述，这两项分别对应 DLSS 超分辨率和帧插值，开了之后会在 GPU 上跑实时处理。
4. **调整 NR / Denoise 强度**  
   如果画面有噪点（比如光线不足的场景），可以在 `Denoise` 档位上试拉一下。README 提到有 per-layer NR percentage controls，意思是降噪比例可以分层调节。
5. **Preview + Record**  
   右侧 Preview 区域实时显示增强后的画面，底部可以直接点 Record 开始录制，音轨会从采集卡的音频通道同步进去。

整个过程大概五步左右，比起之前那种七拐八弯的拼积木方式确实清爽很多。

## 跑完之后

按 README 的描述，跑通之后的效果大概是这样的：

- **一键整合**：README 说"Turn local video, capture cards and game streaming into one real-time GPU enhancement pipeline"，也就是说本地视频、采集卡和串流信号都可以汇入同一个管线处理。
- **增强能力覆盖全面**：Super Resolution、Denoising、HDR、Frame Generation、Capture 都在一个界面里可调，不用到处找插件。
- **支持多设备**：文档里写明了 `PS5 / Xbox / PC Streaming`，我这边只用了 PS5 + 采集卡的组合，理论上 Xbox 和 PC 串流也能复用同一套管线。
- **性能开销可观测**：README 给了些示例数据，比如 `4K~1.42ms`（单面 mesh beauty，仅 AA 场景），并注明"Costs vary with content/hardware"，所以实际帧时间还是要看你自己的显卡和场景。

> 说明：以上效果均基于 README 文档的描述，未做完整实测，数值仅供参考。

## 适合谁 / 不适合谁

**适合：**
- 像我一样，手里有 PS5 或 Xbox，想在 Windows 上做"采集卡 + 实时画质增强 + 录制"一体化处理的独立玩家。
- 不想花几个小时拼 OBS 滤镜链，希望有一个开箱即用的 GPU 增强管线的人。

**不适合：**
- 如果你只需要纯录屏、不需要实时增强，那 OBS + 普通滤镜可能更轻。
- 如果你对延迟极其敏感（比如竞技级格斗游戏直播），建议先用你的具体硬件复现一遍，别完全信文档里的框架时间，因为 README 自己也说了"no local AMD speedup percentage was measured"。
- macOS 或 Linux 用户：目前只支持 Windows 11 + DirectX 12。

---

> 本文基于 Likely7/Veyra-NRVideo 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


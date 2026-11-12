---
title: "HermesEnabler：解锁 Hermès 专..."
date: 2026-11-13 05:02:38
tags: [GitHub, 技术分享]
---

![illustration](/images/illustration_01.png)



![illustration](/images/illustration_02.png)

## 痛点

我的 Apple Watch Series 9，表盘库里一直缺一样东西——Hermès 专属表盘。不是不想用，是真的用不了。每次打开「表盘市场」，Hermès 那几个选项旁边都挂着一个小锁图标，点进去提示"仅适用于 Hermès Watch"。我知道有些朋友会去买 Hermès 版表带搭配，但表壳还得是官方限定款才能开那个功能，光表带解不开这道锁。我就想：凭什么一只 Series 9 就不能挂 Hermès 表盘？


![illustration](/images/illustration_03.png)

## 之前我们是怎么做的

1. **直接买普通 Apple Watch** —— 这是第一步，也是最无奈的一步。反正买不到 Hermès 版表壳，就先用着 Series 9，默认接受没有 Hermès 表盘这件事。
2. **进 Apple Watch app 的表盘市场翻** —— 每天早上起床刷手机时顺手点开 Watch app → 表盘市场，看看有没有新变化。这个动作大概每月重复几次，每次都失望。
3. **看到 Hermès 表盘显示灰色或锁定状态** —— 确认它们只认 Hermès 表壳，没有任何绕过的入口。不是 Bug，是产品设计如此，只能放弃。
4. **试图配对其他表盘找爱马仕选项** —— 试过换一台 iPhone 配对（同事借的）、重启手表、清除表盘缓存再重装 Watch app，折腾了三四处步，结果完全没用。那个锁定就是硬写的。

整个过程大概花了 3-5 步无效操作，耗时超过一周的零星搜索，结论只有一个：官方渠道走不通。

## 为什么最后选了这个项目

备选方案其实不多，我主要对比了两个：

**方案 A：买一块 Hermès 版 Apple Watch。** 成本约 10000+ 元（相比 Series 9 贵一倍以上），而且 Hermès 限定颜色经常断货，等货周期长。为了一个表盘功能多花几千块，性价比太低。

**方案 B：直接装 HermesEnabler（pookjw/HermesEnabler）。** Star 数 1480，Objective-C 写的，README 明确写了"Enable Hermès Watch Faces on any Apple Watch devices"。零额外硬件成本，不需要越狱，也不涉及企业证书签名风险（IPA 自己签自己的 Bundle ID 就行）。

取舍很简单：花一万块钱换表盘 vs 花十分钟跑一个开源工具。前者的沉没成本太高，后者的学习曲线也几乎为零。

## 关键步骤

整个过程分安装和运行两步。README 给了两条路，我选了 IPA 那条，因为不用碰 Xcode。

**第一步：下载 IPA。** 去仓库的 Releases 页面拉最新版本的 HermesEnabler2.ipa。注意版本号要和你的 watchOS 匹配，README 里写了支持 watchOS 27.0 到 27.3 beta 1。

**第二步：用 Sideloadly 侧载。** 把 IPA 拖进 Sideloadly，连上 iPhone。这里有一个坑——README 特别提醒：**关闭"Use automatic bundle ID"，用一个固定的 Bundle Identifier**。原因很简单：如果用自动生成的 ID，每次更新都会变成一个新 app，之前的数据和 MobileGestalt 备份全丢。保持同一个 Bundle ID 和 Apple ID，才能保证数据连续。

**第三步：在 Watch app 里安装。** iPhone 上打开 Watch app → My Watch → Apps，找到 HermesEnabler2 并安装到手表上。这一步是正式的推送流程，完成后手表侧会有这个 app 的图标。

**第四步：在手表面板上运行。** 打开 HermesEnabler2 app，按界面提示走完激活流程。README 没说具体原理，但逻辑应该是修改 MobileGestalt 相关的键值，让系统误以为这是一块 Hermès 版 Watch。删除 app 之前记得先在里面还原 MobileGestalt 备份，避免留下残留改动。

如果嫌 IPA 麻烦，也可以用 Xcode 直接 build 到手表上——打开 HermesEnabler2.xcodeproj，选对手表 device target 点 Run 就行，省了 Sideloadly 那一步。

## 跑完之后

按项目 README 描述，跑通之后普通 Apple Watch 就能在表盘市场里正常显示并使用 Hermès 专属表盘，不再有任何锁定提示。效果本身是即时的——激活完切换一次表盘就能看到结果。

但有几个边界要老实说清楚：README 只声明支持 watchOS 27.0 至 27.3 beta 1，这意味着如果你的系统不在这个区间内，可能根本不兼容，文档里没有提降级或升级后的情况。另外，MobileGestalt 这类改动本质上是绕过系统的硬件校验，苹果未来更新如果收紧这个入口，效果可能会失效，而 README 对此没有给出任何保证。

还有一个细节：Sideloadly 安装的应用受开发者账号限制，每年最多 7 天重新签名一次（免费账号），这意味着每隔一段时间你要重新跑一遍侧载流程，比较繁琐。

## 适合谁 / 不适合谁

**适合：** 手持 Series 8/9/Ultra 等普通版 Apple Watch 的个人用户；watchOS 版本恰好落在 27.0 到 27.3 beta 1 之间；愿意每季度维护一次侧载；对 Hermès 表盘有明确的审美偏好，不想为此多花上万元买限定表壳。

**不适合：** 已经升到 watchOS 27.3 正式版之后的用户（兼容性未知）；公司配发的 Apple Watch（MDM 策略可能禁止侧载应用）；不愿意定期折腾签名续签的人；对系统稳定性要求极高的场景——任何 MobileGestalt 层面的改动都有潜在风险。

最后说一句：这是个个人向的旁门左道，不是官方支持的功能。想用就用，想稳就算了，别指望它有什么售后保障。

> 本文基于 pookjw/HermesEnabler 的 README 与公开信息整理，未做完整实测，关键步骤仅供参考。

---

![关注公众号](/images/footer.png)


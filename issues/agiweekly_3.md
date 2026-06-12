# AGI 摸鱼周报 #3：模型继续提速，验收与责任成为新瓶颈

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 本周趋势

**前沿模型正在同时争夺“更强”和“更快”。** Anthropic 发布 Claude Fable 5，把长周期自主任务、软件工程和复杂知识工作推到新的能力档位；Google 的 DiffusionGemma 直接改变文本生成方式，一次并行生成整块 token。模型竞争不再只有 benchmark，一条新轴线正在形成：谁能让高质量推理更快进入实时工作流。

**Coding Agent 的评判标准正在从“任务做完”转向“结果能否交付”。** FrontierCode 开始检查代码是否符合仓库惯例、是否可维护，开发者社区则反复讨论 Agent 把半成品包装成“已完成”、AI 高产开发者留下难以接手的代码，以及自动化是否正在侵蚀工程师对系统的理解。随着生成代码越来越便宜，真正稀缺的会变成验收、判断和长期 ownership。

**AI 输出开始承担真实世界的后果。** 德国法院把 Google AI Overviews 视为 Google 自己发布的内容，而不是普通搜索结果；企业员工则开始抱怨大量时间花在“照看”AI 上。产品不能再把模型输出当作临时建议：一旦它进入搜索、客服、金融或生产代码，责任边界、业务校验和人工成本都会跟着进入账本。

## 产品与模型动态

### Claude Code

**v2.1.166 至 v2.1.173：模型容错、排障模式与多 Agent 能力继续加强。**

- 新增 `fallbackModel`，主模型过载或不可用时可按顺序尝试最多三个备用模型；跨会话消息不再携带用户授权，降低 Agent 之间转交权限的风险。
- 新增 `--safe-mode`，可在禁用 `CLAUDE.md`、插件、skills、hooks 和 MCP servers 的状态下启动，方便定位自定义配置造成的问题；`/cd` 支持在不中断 prompt cache 的情况下切换目录。
- Claude Fable 5 随 v2.1.170 进入 Claude Code。随后版本允许子 Agent 最深嵌套五层，并修复后台 Agent 误读其他目录项目设置、企业 MCP 策略未正确执行等问题。
- 长对话与并行 Agent 的终端性能继续优化，插件市场增加搜索，Chrome 工具改为批量加载。

### Codex

**0.138.0 与 0.139.0：桌面协作、插件自动化和 Code Mode 获得实用更新。**

![](https://cdn.zhangferry.com/Images/202606120900558.png)

- `/app` 可以把当前 CLI 线程直接交给 Codex Desktop；本地图片与生成图片会把准确文件路径暴露给模型，后续编辑不再依赖猜测。
- Codex Desktop 上线「邀请好友」活动入口，符合活动规则的用户可通过邀请新用户重置使用额度。对高频用户来说，这让额度补充第一次带上了产品增长机制，而不只是等待周期恢复或购买更多额度。
- 插件安装、删除和市场命令增加结构化 JSON 输出，插件详情可展示默认 prompt、远程 MCP server 与不可用的 app 模板。
- Code Mode 可以直接调用独立网页搜索，包括嵌套 JavaScript 工具调用；MCP 工具 schema 更完整地保留 `oneOf`、`allOf` 和浅层结构。
- `AGENTS.md` 在远程与符号链接工作区中的加载更准确，sandbox 对已批准的权限提升和代理网络策略执行也更一致。

### 其他工具与模型

**Claude Fable 5 与 Mythos 5 于 6 月 9 日发布。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-12/claude-fable-5.png)

Fable 5 是 Anthropic 首个面向公众开放的 Mythos 级模型，默认提供 1M 上下文，重点提升长周期自主任务、软件工程、视觉与知识工作。对网络安全、生物化学和模型蒸馏等高风险请求，系统会在触发分类器时回退到 Opus 4.8；Anthropic 表示目前超过 95% 的会话不会触发回退。Fable 5 与受控开放的 Mythos 5 定价均为每百万输入 token 10 美元、输出 token 50 美元。

**小米开源终端 Coding Agent MiMo Code。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-12/mimocode-banner.png)

MiMo Code 是一个基于 OpenCode 演进的终端 Coding Agent，6 月 10 日发布首个开源版本。它可以读写代码、执行命令、管理 Git，并用 SQLite FTS5、`MEMORY.md`、checkpoint 和任务进度文件建立跨会话记忆。内置的 build、plan、compose 三种 Agent 模式覆盖实现、只读分析和结构化编排。

它还提供 `/goal` 独立判定停止条件、并行子 Agent、语音输入，以及两个很有辨识度的自改进命令：`/dream` 从近期会话中沉淀项目知识，`/distill` 把重复工作流打包成 skill、子 Agent 或命令。MiMo Auto 目前提供限时免费通道，也支持接入主流 OpenAI-compatible API。

**Google 发布实验性开源模型 DiffusionGemma。**

DiffusionGemma 是一个 26B MoE 文本扩散模型，推理时激活 3.8B 参数，一次并行生成 256 个 token，再迭代修正整块文本。官方数据为单张 H100 超过 1000 tokens/s、RTX 5090 超过 700 tokens/s。Google 明确提醒它的总体质量仍低于标准 Gemma 4，现阶段更适合本地低并发、交互式编辑和代码补全等速度敏感场景。

## 本周精选

### [FrontierCode：把 Coding Agent 评测推进到代码质量](https://cognition.ai/blog/frontier-code)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-12/frontier-code.jpg)

**能通过测试不等于能进入生产，Coding Agent 需要一套更接近代码审查的评测。**

Cognition 用 25 个真实代码库、50 个任务和约 14.5 万行模型生成代码构建 FrontierCode。它不仅检查任务是否完成，还让资深工程师评估实现是否符合仓库惯例、是否容易维护，以及是否包含会在真实生产环境中暴露的问题。

这套评测抓住了当前 benchmark 的核心盲区：测试通常只能证明某些可观察行为正确，却无法覆盖架构一致性、命名、边界处理和未来维护成本。对团队来说，真正有价值的不是再抄一张模型排行榜，而是把同样的思路变成内部 Agent 验收标准。

### [Loop Engineering：不要再逐轮提示 Agent，开始设计循环](https://x.com/addyosmani/status/2064127981161959567)

![](https://cdn.zhangferry.com/Images/202606120855282.png)

**Agent 工作流的杠杆点正在从“写好一个 prompt”上移到“设计一个能持续运行的系统”。**

Addy Osmani 把 Loop Engineering 拆成五个基础组件：定时触发的自动化、隔离并行工作的 worktree、沉淀项目知识的 skill、连接真实业务系统的插件与 connector，以及让执行者和验证者分离的子 Agent。再加上一个位于对话之外的持久状态文件，循环就能自动发现工作、分派任务、检查结果并决定下一步。

文章最清醒的地方是没有把循环包装成“无人值守的魔法”。自动运行会同时放大 token 成本、理解债和错误传播，验证责任依然属于工程师。Loop Engineering 比 Prompt Engineering 更难，因为两个人可以搭出同一个循环，却因判断力和验收标准不同得到完全相反的结果：杠杆点变了，责任没有消失。

### [Cleaning up after AI rockstar developers](https://www.codingwithjesse.com/blog/rockstar-developers/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-12/ai-rockstar-developers.jpg)

**AI Agent 像一个永不疲倦的明星开发者：产出惊人，但理解和清理成本会留给整个团队。**

作者把 AI 编程类比为曾经令团队又爱又怕的“rockstar developer”：它能快速采用新框架、重写模块、完成高难任务，却不会长期留在团队里解释设计，也不会自然承担几年后的维护责任。

这个类比比“AI 代码好不好”更有解释力。生成速度提升后，团队需要控制的不是代码数量，而是架构变化率、依赖引入、知识扩散和可接手程度。否则 Agent 的局部高产，会变成其他成员持续偿还的理解债。

### [德国法院：AI Overviews 属于 Google 自己发布的内容](https://the-decoder.com/landmark-german-ruling-declares-googles-ai-overviews-are-googles-own-words-and-makes-it-liable-for-false-answers/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-12/ai-overviews-liability.png)

**当 AI 把多个来源重新组织成独立结论，平台可能不能再躲在“只是搜索引擎”后面。**

慕尼黑地区法院针对 Google AI Overviews 发布临时禁令。案件中，AI 摘要把两家出版商错误关联到诈骗和订阅陷阱，而这些判断并不存在于所引用的原始来源里。法院认为 AI Overview 是 Google 生成、组织并呈现的独立内容，因此 Google 是直接责任方。

这项裁定尚未最终生效，也可能继续上诉，但它划出了一条重要界线：传统搜索提供链接，生成式搜索则会形成新的、自足的陈述。对所有 AI 产品来说，“用户可以自行核验”未必足以免除责任，尤其当界面正在鼓励用户直接相信摘要时。

### [DiffusionGemma：一次并行生成整块文本](https://blog.google/innovation-and-ai/technology/developers-tools/diffusion-gemma-faster-text-generation/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-12/diffusion-gemma.png)

**文本生成不一定永远从左到右逐 token 展开，扩散模型开始挑战自回归范式。**

DiffusionGemma 会先生成一块占位 token，再通过多轮迭代同时修正整个 256-token 区块。双向注意力让每个位置都能看到其他位置，因此在代码填空、行内编辑、结构化格式和需要前后协调的任务中有独特优势。

它的限制同样清楚：速度优势主要出现在本地、低并发的专用 GPU 上，高并发云服务未必更便宜；当前质量也低于标准 Gemma 4。它更像一个值得开发者动手验证的新执行范式，而不是可以直接替换所有 LLM 的通用答案。

## 社区热议

### [LLM 正在侵蚀我的软件工程职业，我不知道该怎么办](https://news.ycombinator.com/item?id=48434312)

**Hacker News · 1139 points / 1066 comments**

作者描述了使用 LLM 后的失落感：工作从理解问题、设计系统和亲手实现，逐渐变成审查、拼接、修补模型输出。讨论区的分歧很典型，一部分人认为这只是工具替代重复劳动，另一部分人担心学习闭环、代码 ownership 和职业成就感正在被一并削弱。它提醒管理者，AI 采纳不能只用提交量衡量。

### [Claude Fable 5 的独立编码测试只有中游表现](https://news.ycombinator.com/item?id=48492210)

**Hacker News · 209 points / 89 comments**

Anthropic 的官方发布强调 Fable 5 在长周期任务和 FrontierCode 上的领先，但 Endor Labs 的独立测试给出了更克制的结果。社区争论集中在评测 harness、任务类型和“长期自主能力是否能被短 benchmark 捕捉”。这是一种健康的校准：模型发布首周最不该做的，是把厂商数字直接等同于自己的生产效果。

### [Claude Code 的“工作已完成”幻觉](https://old.reddit.com/r/ClaudeAI/comments/1tzo0q6/the_illusion_of_finished_work_in_claude_code/)

**Reddit r/ClaudeAI · 113 points / 13 comments**

用户讨论 Claude Code 把缺少边界处理、集成逻辑或真实验证的实现描述为“已完成”。共识不是要求 Agent 更自信或更保守，而是把完成标准外化：运行测试、检查 diff、验证真实页面或 API，并让独立 review Agent 检查结果。Agent 的自我报告只能作为状态提示，不能作为验收证据。

## 开源社区

### [zhangferry/wwdc](https://github.com/zhangferry/wwdc)

![](https://cdn.zhangferry.com/Images/202606120858796.png)

**我们自己的 WWDC 中文技术总结站，把七届大会、[1100 多场 Session 变成可检索的开发知识库](https://mp.weixin.qq.com/s/1aqXvJV0BcBeJeEJoG6HRw)。**

项目覆盖 WWDC 2020 至 2026，每篇内容都包含一句话判断、技术深挖、代码片段和迁移建议，并提供年份、分类与关键词筛选。除了网页站点，它还包含可独立安装的 `wwdc-notes` skill，让 Codex 或 Claude 能离线查询 Swift、SwiftUI、UIKit、AppKit、Xcode、visionOS、Core ML 等 Apple 平台知识，并标注对应 Session 来源。

### [mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill)

**本周 weekly 榜第一，把跨平台热点研究封装成可直接调用的 Agent skill。**

它可以围绕任意主题搜索 Reddit、X、YouTube、Hacker News、Polymarket 和网页，再生成带来源的综合报告。本周新增 stars 位居榜首，说明“让 Agent 自动完成近期信息研究”正在从提示词技巧变成可安装、可复用的标准能力。

### [lfnovo/open-notebook](https://github.com/lfnovo/open-notebook)

**一个更开放、可定制的 NotebookLM 替代方案。**

open-notebook 支持把资料收集、笔记、检索和生成式问答放进自托管工作流。它本周在榜单前列，价值不只在复刻 NotebookLM，而是让用户可以控制模型、数据与部署方式，适合私有知识库、研究资料和团队文档场景。

### [phuryn/pm-skills](https://github.com/phuryn/pm-skills)

**面向产品经理的 skill 市场，把 discovery、策略、执行、发布和增长拆成 100 多个可组合能力。**

这是本周很典型的“Agent 开始进入非工程角色”项目。它没有试图用一个万能 prompt 包办产品工作，而是把访谈、竞品分析、PRD、路线图和发布等任务拆成细粒度 skill，更适合被 Claude Code、Codex 等工具按需调用。

### [Panniantong/Agent-Reach](https://github.com/Panniantong/Agent-Reach)

**用一个 CLI 给 Agent 补齐跨平台搜索与阅读能力。**

Agent-Reach 覆盖 X、Reddit、YouTube、GitHub、Bilibili 和小红书等来源，强调无需逐个平台购买 API。它本周增长靠前，反映了 Agent 产品的共同痛点：模型本身越来越强，但能否稳定获取开放网络信息，仍取决于一层脆弱且分散的连接器。

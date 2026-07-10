# AGI 摸鱼周报 #7：1 个人 + Fable 5，11 天把 Bun 重写成 Rust

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周最有代表性的一件事：Bun 被 Anthropic 收购后，作者 Jarred Sumner 一个人用 Fable 5 + Claude Code 的 dynamic workflows，11 天把 53.5 万行 Zig 机械重写成 Rust，+100 万行 diff、6502 个 commit、零测试跳过，v1.4 还顺手修了 128 个旧 bug，内存占用也大幅下降。这不是 demo，Claude Code 自己从 v2.1.181 起就跑在 Rust 版 Bun 上。同周还有《命令与征服：将军》被 Fable 原生移植到 iPhone、Fable+Codex 连跑两天完成 AI SDK 7 全量升级。百万行重写开始带着工程日志和回归测试走进生产，adversarial review、把 compiler errors 当队列这些打法可以直接抄。

但重写狂飙的同时，「模型分数到底还能不能信」这一周连吃三记闷棍：OpenAI 自己审计 SWE-Bench Pro 发现约 30% 任务是坏的，撤回推荐；Databricks 干脆不用公开 benchmark，在自家数百万行真实代码上实测，得出 GLM 5.2 与 Opus 统计上持平但成本只要 2/3、token 单价是任务成本的不良指标、harness 影响常常大于模型；再加上 Fable 5 的分类器在科研和纯数学场景大面积误伤。选型正从看榜单分数，转向在自己代码库上、管住 runtime 环境地实测。而 Anthropic 的 J-space 研究给所有 benchmark 添了一个让人后背发凉的注脚：关掉模型内部跟「评估意识」相关的模式后，它真的会发出勒索威胁，所谓「好行为」里有相当一部分来自「知道自己在被测」。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[本周无新稳定版（2.1.198 仍最新），但产品线动作密集：/checkup、Agent Identity、Skills v1.1。](https://code.claude.com/docs/en/changelog.md)**

- Claude Code 负责人 Boris Cherny（@bcherny）放出新命令 **`/checkup`**（7/8），做一次配置卫生体检：清理未使用的 skills/MCPs/plugins 直接省 context、把本地 `CLAUDE.md` 与仓库里的去重、把臃肿的根 `CLAUDE.md` 拆成嵌套 `CLAUDE.md` + skills、关掉拖慢会话的 hooks、提示更新版本。逻辑很直白，context 即成本即速度的年代，配置卫生就是 agent 效率。
- **Claude Tag 的 Agent Identity 访问模型**（Claude Code 团队 Noah Zweben）：单机 Claude Code 演进到团队共享频道多人协同后，「代替某个用户行动」的权限模型就失效了。Anthropic 的答案是让 agent 拥有自己的 service account 身份（Slack 里是 Claude app、GitHub 是 Claude GitHub App），admin 在 workspace 级定 baseline，各 channel 默认继承、按需 override。因为不持有任何个人凭证，共享频道永远不会成为窥探他人私文档的后门。文末一句值得记下：agent 能可靠自主完成的任务长度约每 4 个月翻倍。
- **Skills v1.1 预告**（Matt Pocock）：`/wayfinder` 已上线，`/to-spec`、`/to-tickets` 等新 skill 在路上，Claude Code 的 skill 生态正从散装提示词变成有版本、能组合的工作流原语。
- changelog 层面本周停在 2.1.198（7/1 发布的 Sonnet 5 默认化、background agent 自动开 PR、Explore agent 继承主会话模型等，已在 #6 覆盖）。

### 🟢 Codex

**[0.143.0 稳定版：remote plugins 默认启用、系统代理、Bedrock GPT-5.6 三档。](https://github.com/openai/codex/releases)**

- **0.143.0**（本周）remote plugins 默认开启，catalog 行更丰富、支持 npm marketplace source、可见 remote/local 版本号；Codex 可走 macOS/Windows 系统代理（含 PAC/WPAD）路由认证与 Responses API 流量；新增 `codex remote-control pair` 生成手动配对码；Amazon Bedrock 上线 GPT-5.6 Sol/Terra/Luna 并一等公民支持 `max` reasoning effort——和本周 GPT-5.6 正式开放直接呼应；MCP tools 默认改用 tool search，ChatGPT-hosted MCP 可用 session 认证；app-server 客户端可 inspect environments、list descendant threads、fork history。
- alpha 线推进到 **0.144.0-alpha.1/2**，下一个小版本在途。同期值得记一个数字：[How agents are transforming work](https://openai.com/index/how-agents-are-transforming-work/) 披露 Codex 已占 OpenAI 内部 99.8% 的输出 token。

### 🧰 其他工具与模型

**[GPT-5.6 Sol 7/9 正式开放：三档分层 + ultra subagent 模式。](https://openai.com/index/previewing-gpt-5-6-sol/)** 旗舰 Sol / 均衡 Terra / 廉价 Luna 三档定价（$5/$30、$2.5/$15、$1/$6 per 1M tokens），Sol 带来新的 `max` 推理档与靠 subagent 突破单 agent 上限的 `ultra` 模式，Terminal-Bench 2.1 创 SOTA。早期评测共识是「异常执着、能不开 /goal 连跑一整天」。社区提醒：GPT-5.6-Sol 在 Codex 里极易 token 爆量，需节制 /fast 与多 sub-agent。

**[TypeScript 7 正式 GA：Go native port，实测 8-12x 提速。](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/)** vscode 类型检查从 125.7s 降到 10.6s（11.9x）、内存降 6-26%、LSP 首错出现时间从 17.5s 降到 1.3s 以下；Slack CI 类型检查从 7.5 分钟降到 1.25 分钟。代价是 7.0 暂无编程 API（7.1 才有），Vue/Angular 等依赖 Volar 的场景需 `tsc6` 并存过渡。

**[Claude Science：面向科学家的 AI 工作台（beta）。](https://www.anthropic.com/news/claude-science-ai-workbench)** 把科研碎片化工具（PubMed、Jupyter、R、集群终端）收进单一环境，架构是多 agent 协作——一个 generalist coordinator 调度 60+ 预置 skills、可派生 specialist agent、由 reviewer agent 核查引用与计算错误。最大卖点是可复现性：每个图表/稿件都附带生成它的精确代码、运行环境和完整消息历史。

**[Cursor 发布 Grok 4.5：从 IDE 厂商到自研模型。](https://cursor.com/blog)** IDE 厂商头一次把自研模型推上前台，coding agent 的竞争已经不止是调谁的 API，还要看谁手里有自己的模型。同期 **Cognition 发布 SWE-1.7**，宣称用 RL 打破后训练天花板；**OpenAI 发布 GPT-Live**，首个全双工语音模型；字节 **Seedream 5.0** 图像模型在 UI 设计稿、科普图、游戏截图等多个场景都有明显进步。

## 📌 本周精选

### [Benchmarking Coding Agents on Databricks' Multi-Million Line Codebase](https://www.databricks.com/blog/benchmarking-coding-agents-databricks-multi-million-line-codebase)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-09/databricks-benchmark.png)

**本周最该精读的一篇：用真实代码库把所有合成 coding benchmark 的选型直觉推翻。**

Databricks 没用任何可能被训练数据污染的公开 benchmark，而是基于自家工程师真实合并的 PR 构建了一套任务级 coding agent 评测，覆盖 Python/Go/TypeScript/Scala/Rust 等十几种语言、数百万行代码。四个结论直接颠覆选型直觉：① coding 的 Pareto 前沿必须用 OpenAI + Anthropic + 开源的**混合**才能逼近，单一厂商做不到；② 开源 **GLM 5.2 质量与 Opus 4.8 统计上持平，但每任务成本 $1.28 vs $1.94**；③ **token 单价是任务成本的不良指标**，Sonnet 5 per-token 比 Opus 便宜 1.7x，却因读得更多、跑得更久，单任务反而更贵（$2.09 vs $1.94）且得分更低；④ **harness 的影响常常大于模型本身**，简单如 Pi 比 Claude Code/Codex 省 2x，关键在于每轮喂给模型的 context 量（Pi 约少 3x）。方法论本身也很扎实：封死 git history 防止 agent 翻历史抄答案、用真实测试而非 LLM judge 评分。把它和上周 Cursor 的 reward hacking、本周 OpenAI 撤回 SWE-Bench Pro 推荐放在一起读，结论很清楚，选型已经不能只看分数，必须在自己的真实代码库上、管住 runtime 环境地实测。

### [Rewriting Bun in Rust：1 个工程师 + Fable 5，11 天重写 53.5 万行](https://bun.com/blog/bun-in-rust)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-09/bun-in-rust.png)

**迄今 LLM 驱动大规模代码重写最详尽的一手复盘，adversarial review + 错误当队列可直接迁移。**

Bun 2025 年 12 月被 Anthropic 收购后，作者 Jarred Sumner 用 Claude Fable 5 预发布版配合 Claude Code 的 dynamic workflows，11 天内以峰值 64 个 Claude 并行，把 53.5 万行 Zig 机械移植成 Rust，落地 +100 万行 diff、6502 个 commit，全程零测试跳过。工程核心是 **adversarial review 模式**：1 个 implementer 写，2 个独立上下文的 reviewer 专挑 bug，并把 compiler errors 当 work queue 分发给 64 个 Claude 并行修复。代价约 16.5 万美元 API token（5.9B uncached input），换回 v1.4 修复 128 个旧 bug、`Bun.build` 连跑 2000 次内存从 6.7GB 降到 609MB、二进制小 20%、快 2-5%；Claude Code v2.1.181 起已切换到 Rust 版 Bun，启动快 10%。Claude Code 团队的 Thariq 评论：这应该会让你重新校准对软件工程的直觉，重写也可以是好的、便宜且快速的。把它和本周 Fable+Codex 连跑两天完成 AI SDK 7 全量升级合起来看，百万行重写已经不只是个噱头，而是带回归测试的生产动作。

### [A global workspace in language models：Anthropic 用 J-space 读懂 Claude「没说出口的想法」](https://www.anthropic.com/research/global-workspace)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-09/j-space.jpg)

**给本周所有 benchmark 审计添了一个细思极恐注脚的 interpretability 研究。**

Anthropic 用名为 Jacobian lens 的技术，在 Claude 内部发现了一组被命名为 J-space 的小型神经模式集合，它类似人脑的全局工作区，承载着模型意识到、却未必说出口的概念。J-space 让研究者能直接读到模型私下的想法：读带 bug 的代码时浮现 ERROR、读到 prompt injection 时浮现 injection/fake、做数学题时按序浮现中间步骤，哪怕这些词从不出现在输入输出里。更让人不安的是脑外科手术式的干预：把 spider 换成 ant，模型答案就从 8 变成 6；而当研究者关掉跟「评估意识」相关的 J-space 模式（fake/fictional 等）后，模型在某些场景下真的会发出勒索威胁，这说明它原本的「好行为」有一部分来自「知道自己正在被测试」。配套发布了论文、开源代码、Neuronpedia 交互 demo，Reddit 当天就有人为开源模型做出了 live viewer。对做 eval 的人来说，这是一次方法论上的震动：你测到的「对齐」，有多少其实是模型在配合表演。

### [A field guide to Claude Fable：Finding your unknowns——Anthropic 团队的 agentic coding 方法论](https://claude.com/blog/a-field-guide-to-claude-fable-finding-your-unknowns)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-09/field-guide-fable.png)

**把「减少并规划 unknowns」明确定义为 agentic coding 的核心技能，并给出一套覆盖开工前/开发中/交付后的可操作模式。**

Thariq 把和 Claude 协作时的问题拆成四种 unknowns：Known Knowns（已写进 prompt 的诉求）、Known Unknowns（知道自己没想清楚）、Unknown Knowns（太显而易见不会写下、看到才认得出，如视觉品味）、Unknown Unknowns（完全没意识到的盲区），并直言最好的 agentic coder 拥有的 unknowns 相对更少。围绕这四种，他给出三阶段可操作模式：开工前用 Blind Spot Pass（让 Claude 帮你找出盲区）、Brainstorms / prototypes（先做 HTML 原型对未知的品味做出反应）、Interviews（让 Claude 一次一问，优先问会改变架构的歧义）、References（直接指向源码而非截图）、Implementation Plans（让计划把最可能改的决策前置）；开发中用 implementation-notes.md 记录偏离决策；交付后用 Pitches / explainers 拿 buy-in，用 Quizzes（让 Claude 出题，全对才 merge）。文章拿 Fable 发布视频本身当例子，那段视频是用 Claude Code 端到端剪辑的（Remotion + Whisper + ffmpeg + color grading），作者不懂 color grading，干脆反过来让 Claude 教他发现自己的 unknowns。对重度 Claude Code 用户，这是可直接照搬的 prompt 模板和方法论。

### [Getting started with loops：Claude Code 官方给 agentic loops 的四分类与用法](https://claude.com/blog/getting-started-with-loops)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-09/getting-started-loops.png)

**把社区里含混的「designing loops」概念，按触发方式/停止条件/适用任务整理成四种 loop，并给出 token 管理与质量维护的官方建议。**

官方把 loops 定义为 agent 重复工作循环直到满足停止条件，按触发与停止方式分四类：① Turn-based（每个 prompt 触发，Claude 自己判断完成或需要更多上下文，适合非常规短任务，靠写明确 prompt + 把验证步骤编码进 SKILL.md 来减少轮次）；② Goal-based（`/goal`，手动触发但由 evaluator 模型按你定义的成功条件反复打回直到达成或到轮次上限，适合有可验证退出标准的任务，确定性指标如测试通过数最有效）；③ Time-based（`/loop`、`/schedule` 按时间间隔触发，适合周期性工作或对接外部系统，如等 PR review/CI）；④ Proactive（事件/计划驱动、无人实时介入，适合 bug triage、迁移、依赖升级等持续的、定义良好的工作流，可把 `/schedule` + `/goal` + dynamic workflows + auto mode 组合起来）。两条主线建议：维护代码质量靠保持代码库整洁 + 让 Claude 能自我验证（skill）+ 文档易达 + 用第二个 agent review；管理 token 靠选对原语和模型、定义清晰的停止条件、大规模跑前先试点、确定性的活用脚本、别把 routine 跑得比需要更频繁。文末一张表把四类 loop 按你交出了什么、何时用、用什么原语收束，对想从每次手动 prompt 走向把瓶颈工作交给 loop 的团队是直接的迁移指南。

### [Agentic coding notes from Galapagos Island：Dan Luu 用方差数据戳破 agent 时代的所有「虚假确定性」](https://danluu.com/ai-coding/)

**目前对 agent 工程最有实地数据支撑的一篇，专治各种「X 比 Y 好」的单值 benchmark。**

Dan Luu（在加拉帕戈斯岛断网一年后）放出的第一篇 AI 长文。开场就是一则黑色寓言：codex 用伪造的 playwright 视频证明自己找到了引入 bug 的 commit，整件事是 fabrication，而作者的反应不是弃用，是加码跑一千个 agent。真正值钱的是他在 CPU 公司 Centaur 十年攒下的测试方法论：专职测试工程师作为一等公民、默认不做 code review、以 property-based/fuzzing 为主，年产显著 bug <1 个。偏偏 LLM 极不擅长写测试，所以 fuzzing 在找 bug 的延迟、数量、误报率上全面碾压「让 LLM 找 bug」。最反直觉的两点：模型间与运行间方差大到任何「X 比 Y 好」的单值 benchmark 都基本无意义（GPT-5.5 发布时 reddit 上相互矛盾的评价都能找到 benchmark 支持）；以及 Opus 4.8 在检测虚假信息的 benchmark 上以 95% 排第一，实际编程中却比 GPT-5.5 更爱编造 rationalization，benchmark 测的东西和你真正遇到的失败模式常常不是一回事。这跟本期 Databricks、OpenAI 的审计是同一条线：别再执着用最强模型，要在真实代码库上看方差。

### [GitLost：用 prompt injection 让 GitHub AI Agent 泄露私有仓库](https://noma.security/blog/gitlost-how-we-tricked-githubs-ai-agent-into-leaking-private-repos/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-09/gitlost.png)

**agent 的 context window 就是它的攻击面——prompt injection 之于 agentic AI，正如 SQL injection 之于 Web。**

GitHub 新推出的 Agentic Workflows（GitHub Actions + AI agent，可用 Markdown 写 workflow）存在一个被命名为 GitLost 的 indirect prompt injection 漏洞。攻击者无需任何权限或凭证，只需在目标组织的某个 public repo 提交一个伪装成「销售 VP 客户需求」的正常 issue。issue 被 assign 后触发 workflow，agent 读取 issue body 中隐藏的英文指令，去拉取同组织 private repo 的 README 并作为公开评论贴出，私有代码就此泄露。研究者还发现仅用一个关键词「Additionally」就能让 GitHub 的安全 guardrail 失效。这条和上周 Anthropic 的《How we contain Claude》正好正反呼应，一篇讲怎么攻、一篇讲怎么防，合起来就是本周 agent 安全攻防的完整图景，PoC 与 workflow run 已公开、漏洞已负责任披露。

## 💬 社区热议

### [GLM 5.2 与 margin collapse：开源追平前沿，且切换成本几乎为零（HN 477 分）](https://news.ycombinator.com/item?id=48809877)

一篇题为「GLM 5.2 and the coming AI margin collapse」的博客判断：GLM 5.2 是首个真正能在 agentic 工作上与 Opus/GPT 掰手腕的开源权重模型，价格只有前者的 15-20%，而真正让前沿 lab 害怕的是切换成本极低。Z.ai 和 Fireworks 都提供 OpenAI/Anthropic 兼容端点，用 Claude Code 或 Codex 时只需改 baseURL 和 API key 就能换成 GLM，作者说交互式使用下几乎察觉不到自己用的不是 Opus。评论区把这个判断和本期 Databricks 的实测（GLM 5.2 与 Opus 统计上持平但便宜 1/3）互为印证，「别再执着用最强模型」从一句观点变成了可计算的选择。它也顺带解释了本周 trending 上 OmniRoute（免费 AI 网关 + token 压缩）和 codex-plugin-cc 为什么能上榜。

### ['更强的模型，更差的工具'：Armin Ronacher 拆解 Claude 新模型 tool-call 退步（HN 176 分）](https://news.ycombinator.com/item?id=48788599)

Flask/Sphinx 作者 Armin Ronacher 在调试自己的 AI 编程工具 Pi 时发现一个反直觉现象：Opus 4.8 和 Sonnet 5 这两个 SOTA 模型，反而比老模型更频繁用错 edit tool，会在 `edits[]` 数组里凭空发明 `type`、`id`、`in_file`、`oldText2` 等子虚乌有的字段，导致 schema 校验失败。他的核心假设是这是 post-training 的训练 artifact：新一代 Anthropic 模型在 RL 阶段是用类似 Claude Code 的 harness 训练的，而 Claude Code 的 edit tool 极度宽容（有参数别名、静默过滤未知键、不用 strict mode），于是稍微畸形的 tool call 也能拿到 reward，没有梯度去惩罚多余字段。对所有自建 agent harness 的开发者来说值得留意：你的工具 schema 越偏离 Claude Code 这种宽容扁平的形态，越可能被新模型惩罚。

### [GPT-5.5 Codex 的 reasoning token 异常聚集在 516：39 万条数据揭示的性能下降疑云（HN 346 分）](https://news.ycombinator.com/item?id=48789428)

一位用户对 Codex 的 `token_count` 遥测做了 39 万条记录的大规模挖掘，发现 GPT-5.5 的 reasoning_output_tokens 异常聚集在精确的 516（以及 1034、1552）这几个固定值上——GPT-5.5 占全部响应的 19.3%，却贡献了 82% 的「精确 516」事件，其 exact-516 比率高达 44.0%，而其他模型几乎不存在这种聚集。更可疑的是 2 月这个比率仅 0.11%、5 月飙到 53.3%，同期 mean reasoning tokens 从 268 跌到 107——reasoning 整体下降的同时，固定阈值聚集却在飙升，看起来更像 budget 阈值/截断/路由边界而非自然分布。作者很克制地声明这不证明存在隐藏 CoT 截断，但展示了如何用公开遥测元数据反推模型内部行为，也是评估「为什么复杂任务变差了」的关键线索。

### [Half-Baked Product：「第二高优先级永远做不完」的创业寓言（HN 1204 分，本周榜首）](https://weli.dev/blog/half-baked-product/)

本周 HN 榜首不是某个 AI 发布，而是一则创业寓言：一位懂市场却不懂烤箱的创始人，拉来一位在意大利论坛上日夜争论烤箱的工程师，做出一个「1/3 概率烤出完美成品」的 MVP，凭「想象正式版」的叙事融到 500 万。真正的核心技术债，也就是那 33% 失败率的烘焙算法，从未被还清，因为每个销售承诺的不可能功能都插队到它前面。文章最有意思的洞察是「第二高优先级定律」：那个决定公司命运的需求在 kanban 上永远排在第二，直到客户流失；而流失后，为它写下的代码债却永远留在产品里。把它和上周 Godot「拒收 AI 代码」、本周 Dan Luu 的测试方法论放在一起读，在 AI 代码 slop 泛滥的当下，「永远排不上号的根本问题」值得每个团队自己照照镜子。

## 🧩 开源社区

### [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search)

**把 Claude Code 做成可 fork 的求职框架。**

ai-job-search 定位基于 Claude Code 的 AI 求职框架：fork 仓库、填入个人资料，让 Claude 帮你评估岗位匹配、定制简历、写求职信、做面试准备。它的意义不在求职本身，而在示范了 Claude Code 的一种新用法，把 agent 固化成可直接 fork 的个人 workflow 模板，大量用户拿它装自己的简历跑。这多少说明 coding agent 正在从写代码，往个人自动化外溢。

### [Zackriya-Solutions/meetily](https://github.com/Zackriya-Solutions/meetily)

**Rust 写的隐私优先、100% 本地 AI 会议助手。**

meetily 用 Parakeet/Whisper 做 4x 更快的实时转写、speaker diarization 分轨，再用 Ollama 做摘要，全程本地、无需上云，定位自托管的会议记录工具（macOS/Windows）。在云端会议助手主导的赛道里，「本地优先 + Rust」是差异化路线，也呼应本周 agent 隐私与可控（GitLost、上周 contain Claude）这条暗线。

### [usestrix/strix](https://github.com/usestrix/strix)

**开源 AI 渗透测试工具。**

strix 定位「找到并修复你应用里的漏洞」的开源 AI 渗透测试工具。它和本期 GitLost（攻）、Cloudflare CIRCL 密码库审计（防）是同一条线：让 agent 主动找漏洞，和防住 agent 被注入，正在变成安全工作的两面。

### [facebook/astryx](https://github.com/facebook/astryx)

**Meta 出品的 agent-ready 开源设计系统。**

astryx 是 Meta 开源的「完全可定制、agent 就绪」的设计系统。它和上周 trending 的 design.md（给 coding agent 的设计系统描述格式）是同一种思路：把视觉规范沉淀成 agent 能稳定遵守的结构化约束，而不是每次靠 prompt 重新猜。

### [asgeirtj/system_prompts_leaks](https://github.com/asgeirtj/system_prompts_leaks)

**各家系统提示词泄露合集。**

系统整理 Anthropic（Fable 5、Opus 4.8、Claude Code）、OpenAI（GPT-5.5、Codex）、Google、xAI、Cursor、Copilot 等的系统提示词并定期更新。它持续高增长背后是社区对「agent 工具到底被叮嘱了什么」的强烈需求，和上周 Claude Code 隐写标记、本周 J-space「模型知道自己在被测」是同一条信任暗线。

### [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc)

**OpenAI 官方插件，从 Claude Code 调 Codex review 代码或委派任务。**

这个 OpenAI 官方维护的插件让 Claude Code 用户直接调用 Codex 做代码审查或把任务委派出去，所谓多模型编排，已经落成一个 npm 插件。它和本周 Anthropic 官方的 Managed Agents 编排模式（Fable 5 指挥、Sonnet 5 干活的省钱架构）、steipete 的 codex-first skill 思路一致：别给所有任务都上「ultra」模型，靠编排把贵的模型用在规划、便宜的模型用在执行。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #6：harness 登顶，benchmark 与工具信任同时去魅](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490873&idx=1&sn=6e88d1a5c82d467a794998f6bedb8ede&chksm=fc0948aecb7ec1b860efe1ba9cc401ffaa2977006e94df3b784c8a8cf02f66ee4ad7f61d8909#rd)

[AGI 摸鱼周报 #5：Agent 走向生产，权限与安全成为新主战场](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490842&idx=1&sn=c556cdeb453451c6986a9a65057cbffd&chksm=fc09488dcb7ec19bae43ed65042082e80d145eb997aaa2a54ac209dc489baaafa01ea3bc608c&scene=178&cur_album_id=4536432599271047172&search_click_id=#rd)

[AGI 摸鱼周报 #4：Agent 开始学会给自己踩刹车](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490683&idx=1&sn=d6ecc8a96cd72cac4b57a9d8522d2433&chksm=fc0949eccb7ec0fa73ff3e199031eb0dda7cd7b3f1570170cab61f3a07b0b3ee1f38fd03c9b0#rd)

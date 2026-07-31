# AGI 摸鱼周报 #10：Claude Opus 5 发布，接近 Fable 5 智能、价格减半

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周的头号事件是 Anthropic 在 7/24 发布 Claude Opus 5：定价与上一代 Opus 4.8 持平（$5/M 输入、$25/M 输出），却在多项评测上接近甚至超越自家最贵的 Fable 5，CursorBench 3.2 max effort 距 Fable 5 峰值仅 0.5% 但成本减半，单任务成本比 Fable 低约 26%。意义不在于「又强了一点」，而在于「接近旗舰智能」第一次被压到了半价档，Opus 5 也成了 Claude Max 的默认模型。

围绕 Opus 5，本周还出现了两条互相印证的工程反思。一条是 Anthropic 自己把 Claude Code 系统 prompt 砍掉 80% 以上、评测零损失，结论是过去严重过度约束了模型；另一条是 SlopCodeBench 这个专门测「随时间维护代码库」的新基准给 Opus 5 打了 24 分，一次性解题评测看不出的「关灯运行」短板被戳破。模型在变强、prompt 在变薄、而「验证」正取代「生成」成为真正的瓶颈，这是本周所有产品动态和社区讨论共同指向的方向。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[桌面版原生集成 iOS 模拟器进入 public beta，Teach Claude a skill 录制操作。](https://code.claude.com/docs/en/changelog.md)**

- 桌面版集成 iOS 模拟器（public beta，需 Xcode）：构建并运行 iOS app 时，模拟器作为面板嵌在对话旁，Claude 能看到运行中的 app、与之交互、反复迭代，用户也能随时自己上手点，省掉编辑器、构建、模拟器之间来回横跳。
- Teach Claude a skill：用户演示一遍操作，Claude 把步骤映射成一个干净可复用的 skill，被社区比作 Excel record macro for everything，把重复任务自动化而无需写代码。
- `/claude-api migrate` 命令配合 Opus 5 上线：自动迁移 model string 到 `claude-opus-5`，并给出针对 Opus 5 的 prompt 优化建议。

**社群信号：Opus 5 computer use 跃升，官方悬赏更难的 benchmark。** Claude Code 负责人 Boris Cherny（[@bcherny](https://x.com/bcherny)）本周 repost 强调 Opus 5 在 OSWorld v2 从 55.7% 跃到 70.6%，Anthropic 的 Kiana Ehsani 同时表态「想要更难的 computer use eval，愿意赞助一个 Opus 5 低于 50% 的 benchmark」：自信到公开悬赏能难倒自家模型的评测。同期 Boris 还 repost 称 Opus 5 已能产出接近超人水平的 spreadsheets 和咨询级 slide decks。

### 🟢 Codex

**[OpenAI 开源 Codex Security，把 Codex 从「写代码」推进到「保代码安全」。](https://github.com/openai/codex-security)**

OpenAI 发布 `@openai/codex-security`，一个 CLI 和 TypeScript SDK，用于发现、验证、修复代码中的安全漏洞，并能在 CI 中跑安全检查、追踪 findings 随时间变化。安装即用：`npm install @openai/codex-security` → `npx codex-security login` → `npx codex-security scan .`，认证支持 ChatGPT 登录或 API key（CI 场景）。这标志着 Codex 产品线从「生成代码」正式扩展到「守护代码」：安全扫描从独立的 SAST 工具，变成 coding agent 原生能力的一部分，开发者不用再切换上下文到传统安全工具。

**社群信号：Codex 本周重置 3 次额度，Sol 优化后更耐用、5h 限制回归。** 据 [codex-resets.com](https://codex-resets.com/) 记录，Codex 本周重置了 3 次用量额度（平均每两天一次），被社区戏称「赛博菩萨」。Tibo（[@thsottiaux](https://x.com/thsottiaux/status/2082317452755751098)）7/29 发推解释：GPT-5.6 Sol 之前比预期更快消耗额度，但 OpenAI 没有降低任何订阅的用量，而是优化了 Sol，典型使用下额度能多用约 18%，同时预告 5h 使用限制将回归。

### 🧰 其他工具与模型

**[Claude Opus 5 发布：接近 Fable 5 智能、价格减半。](https://www.anthropic.com/news/claude-opus-5)** Opus 5 定价与 Opus 4.8 持平（$5/M 输入、$25/M 输出），却在多项评测上接近甚至超越 Fable 5：Frontier-Bench v0.1 上性能翻倍且单任务成本更低，CursorBench 3.2 max effort 距 Fable 5 峰值仅 0.5% 但成本减半，ARC-AGI 3 得分是次优模型的 3 倍，OSWorld 2.0 以 Fable 5 三分之一成本超越其最佳结果。两个值得工程团队关注的 beta：对话中途增删工具而不破坏 prompt cache（mid-conversation tool changes）、API 上安全分类器拦截的请求可自动 fallback 到其他模型。模型 ID 为 `claude-opus-5`，是 Claude Max 的默认模型。社区实测金融垂直 benchmark 较 Fable/GPT-5.6 Sol 再提升 7-10%、单任务成本比 Fable 低约 26%。

**[MCP 三年来最大更新：协议全面无状态化。](https://blog.modelcontextprotocol.io/posts/2026-07-28/)** MCP 发布 `2026-07-28` 版本，官方称之为「远程 MCP 上线一年多来最重要的一次更新」。核心变化是把协议从双向有状态改成 request/response 无状态：取消 `initialize` 握手和 `Mcp-Session-Id`，每个请求自带协议版本和客户端身份，可落在任意实例后面（plain round-robin 负载均衡即可，无需共享存储）。Extensions 成为正式框架，Tasks、MCP Apps、Enterprise Managed Authorization 一并落地；同时锁定 12 个月弃用窗口，Roots/Sampling/Logging 与 legacy HTTP+SSE transport 进入弃用期。Tier 1 SDK 月下载已近 5 亿，TS/Python 累计破 10 亿。

**[ChatGPT Health 上线：仅凭 Apple Watch 数据就能给出靠谱的健康洞察。](https://openai.com/index/health-in-chatgpt/)** OpenAI 于 7/23 面向美国用户推出 ChatGPT Health，即便不连接第三方健康 App，仅凭 Apple Watch 的原始数据，ChatGPT 就能分析出大量有效信息，例如定位出长期「有氧适能偏低」的真正原因（并非身体异常），这是传统健康 App 难以提供的解释性洞察。连接需切换美区 IP 在侧边栏开启，Apple 健康完整接入需 iOS 客户端。它展示的是「AI 不只是看数据，更能解读数据并给出因果假设」的产品形态。

## 📌 本周精选

### [MCP 2026-07-28 规范发布：自问世以来最大更新，协议全面无状态化](https://blog.modelcontextprotocol.io/posts/2026-07-28/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-29/mcp.png)

**本周头牌：MCP 把自己从「常开双向流」改造成「无状态 request/response」，这是 agent 工具协议从 demo 走向生产的分水岭。**

MCP 发布 `2026-07-28` 版本，官方称之为「远程 MCP 上线一年多来最重要的一次更新」。核心变化是把协议从双向有状态改成 request/response 无状态：取消 `initialize` 握手和 `Mcp-Session-Id`，每个请求自带协议版本和客户端身份，可落在任意实例后面，plain round-robin 负载均衡即可，无需共享存储。

原本依赖常开双向流的 server→client 调用（sampling/elicitation/roots）改为 Multi Round-Trip Requests：server 返回 `input_required`，客户端带上答案重试。method/tool 名走 `Mcp-Method`/`Mcp-Name` HTTP header，网关和 WAF 可直接基于 header 路由限流；list 响应带 `ttlMs` 可缓存。Extensions 成为正式框架，Tasks、MCP Apps（沙盒 iframe 服务端渲染 UI）、Enterprise Managed Authorization（EMA）一并落地。

同时锁定 12 个月弃用窗口，Roots/Sampling/Logging 与 legacy HTTP+SSE transport 进入弃用期。TS/Python/Go/C# SDK 同步更新，Rust beta；AWS Bedrock AgentCore、Cloudflare Workers、Google Cloud、Microsoft Foundry、Figma、Supabase 声明日零支持，Tier 1 SDK 月下载已近 5 亿、TS/Python 累计破 10 亿。对自建 MCP server 的团队，这是从 demo 走向生产的分水岭：无状态化意味着它终于能像普通 HTTP API 一样被负载、被缓存、被网关治理。

### [为 Claude 5 移除 80% 系统 prompt：上下文工程的新规则](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-29/system-prompt.png)

**Anthropic 把 Claude Code 系统 prompt 砍掉 80% 以上、评测零损失，结论是过去严重「过度约束」了模型，一份本周最值得立即落地的方法论。**

Anthropic 为 Opus 5 和 Fable 5 移除了 Claude Code 系统 prompt 的 80% 以上，编程评测没有任何损失。核心洞见是过去严重过度约束了模型，system prompt、CLAUDE.md、skills 之间甚至会出现「保留文档」与「禁止写注释」互相打架的指令。

文章提炼出六条「旧→新」转变：从给规则转为让模型用判断力（「默认不写注释」→「写读起来像周围代码的代码：匹配注释密度、命名、惯用法」）；从给示例转为设计接口（示例反而收窄探索空间，用枚举状态等接口暗示用法）；从「全堆前面」转为渐进式披露（验证、code review 拆成独立 skill，工具走 ToolSearch 延迟加载，CLAUDE.md/Skill 用文件树按需加载）；从重复指令转为精简工具描述；从手动 `#` 写 CLAUDE.md 转为自动记忆；从简单 spec 转为富引用（HTML artifact、测试套件、可移植的函数、rubric + verifier agent）。

配套推出 `/doctor` 命令自动帮你精简 skills 和 CLAUDE.md。这条和本周头牌 MCP 的「工具描述延迟加载」、Cursor Router 的 dynamic tool calling 直接相通：模型变强了，最好的 prompt 工程反而是「少塞、按需给」。

### [Bun 用 AI 重写 Rust？一篇用 git 数据打脸营销叙事的深度核查](https://lockwood.dev/ai/2026/07/27/how-is-the-bun-rewrite-in-rust-going.html)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-29/bun-debunk.png)

**这是 #7 精选过 Bun 作者本人正面复盘的反转后续：有人 clone 了整个仓库做数据核查，得出截然相反的结论。**

7 月初 Bun 作者 Jarred Sumner 宣布「用 Claude 把 Bun 从 Zig 重写为 Rust」，称 11 天、$165k API 费用即完成合并（#7 精选过这篇正面叙事）。Tom Lockwood 没有停留在惊叹，而是 clone 了整个 1.23 GiB 的 Bun 仓库做数据核查，得出截然相反的结论：合并到 main 六周后仍无 release tag（上一个 release 是 11 周前的 v1.3.14）。

更刺眼的是 robobun（Claude Code 代理）的 open PR 数从 7/9 的 1277 不降反增至 2475，按 Buildkite 每个 PR 约 40 分钟算，全部合并需连续跑 86 天；部分 PR 由 Anthropic 员工直接提交；真实成本若算上 CI/CD 和人力接近 $800k。

这对「AI 能否替代开源维护者」这一核心命题是难得的实证检验：当下大量估值建立在 AI 编码能力叙事上，需要这种清醒的审视。附注：Anthropic 的 C 编译器、Cursor 的 FastRender 浏览器这些曾高调宣传的 AI 重写项目都已数月无提交。读这篇时记得对照 #7 那篇作者本人的乐观复盘：同一件事，营销叙事和 git 数据讲的是两个故事。

### [Cursor Router：把选哪个模型变成一个工程问题](https://cursor.com/blog/router)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-29/cursor-router.gif)

**Cursor 用一个在 60 万+ 真实请求上训练的分类器，在每个请求前决定该用哪个模型，「路由」正在取代「单一最强模型」成为新的成本-智能解法。**（注：#9 用过 Cursor 让 agent swarm 造 SQLite 那篇，这篇是同一团队 Router 模型选型分类器，角度完全不同。）

Cursor 发布企业级智能模型路由器，核心是一个用 60 万+ 真实请求训练的分类器，在每个请求前判断该用哪个模型：简单活走低价模型，UI 改动走「审美最好」的模型，复杂长程任务走 frontier 推理模型。

最关键的工程判断是用在线 A/B 测试（数百万请求）而非离线 eval 优化：离线 eval 既小、又忽略切模型带来的 cache-miss 成本，且路由器训练时就 cache-aware。奖励信号是用户满意度（AFC）：「继续做下一个功能」是强正信号，「纠正 agent」是强负信号，辅以 keep rate（生成的代码在仓库里的留存率）。数据很有说服力：约 60% 开发者只用单一模型做日常，导致常规工作按前沿价付费；Auto Intelligence 满意度接近 Fable 但成本低约 60%，单次 commit 成本 Intelligence $6.76 / Balance $4.63（对比 Fable $12.69、Opus 4.8 $7.34），早期企业客户省 30-50%。

它还顺带披露 dynamic tool calling：多数工具描述不再每次进 prompt，模型首次调用时才去查，这与本周头牌 MCP 的延迟加载、Anthropic 上下文工程文的 progressive disclosure 思路直接相通。对每个为企业 AI 账单头疼的人，这是「路由 > 单一最强模型」的硬数据论证。

### [Datadog 给 Claude Code 造了台通用机床：agent 写 spec，确定性 kernel 负责验证和编译](https://claude.com/blog/how-datadog-built-a-universal-machine-tool-for-claude-code)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-29/datadog.png)

**Datadog 借工业革命「机床」的概念提出 Temper：agent 不再直接生产代码，而是生产 spec，再由一个确定性 kernel 用四层分析验证并编译，把「验证」从人 review 变成数学证明。**

Datadog 全员用 AI coding、Claude Code 驱动其中约三分之二的生产代码，但他们发现真正的瓶颈已经从「生成」挪到了「验证」：agent 出码远比团队能 review 的快。VP 工程师 Sesh Nalla 借工业革命「机床（machine tool）」的概念提出 Temper：一个最小化的确定性 kernel。

这里的关键反转是 agent 不再直接生产应用代码，而是生产 specification（每个能力由行为、数据契约、授权三件套描述）；kernel 读完 spec 后用四层独立分析验证（符号推理证明 guard 可满足且 invariant 归纳成立、穷举所有可达状态、带故障注入的确定性仿真、约千次随机属性测试并收缩反例），通过后才部署。因为被证明的制品就是被执行的制品，验证与运行之间零漂移。

控制逻辑被从散落在路由/服务/后台任务里的隐式状态机中抽出，变成显式的转移表（数据而非代码），agent 能在策略下热重载，而编译步骤被移出 LLM，类比「把 Rust 代码交给 Rust 编译器」。文章给出判断基准：与其加吞吐，不如先把验证缺口补上：「agents already produce code faster than any team can review; the gap between what's generated and what's proven is where the failure modes pile up」。它呼应本周反复出现的同一条主线：瓶颈在验证，不在生成。

## 💬 社区热议

### [SlopCodeBench 给 Opus 5 打了 24 分：模型维护代码库的真实水平被戳破](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/benchmarking-opus-5-on-slop-code-bench.md)

humanlayer 把 Opus 5 放上 UW Madison 的新基准 SlopCodeBench（2026.3）。这个基准不一次性给出完整需求，而是分多个 checkpoint 逐步披露，专门考察模型「随时间维护代码库可演进性」的能力，这正是 SWE-bench 这类一次性解题基准测不到的维度。结果反直觉：Opus 5 仅以 24%（17 个 checkpoint 过 4 个）勉强领先 Opus 4.6 论文里的 17%，三个模型没有一个能完整通关哪怕最简单的题目。更刺眼的是，Opus 5 写了 2.9 万行代码（其他模型约 9000 行）和 5 倍的函数，多花的钱只换来有限正确性，「每块钱都在买正确性，但谁都没买够」。结论很直接：今天的模型还远不能在真实软件工程里「关灯运行」，必须有人 steering。这和本周 Opus 5 的发布狂欢形成最有必要的对照：「接近旗舰智能」不等于「能自主维护一个代码库」。

### [领取 Fable 5 的 $100 免费额度，会静默开启无上限按量计费（r/ClaudeAI 1571 分）](https://old.reddit.com/r/ClaudeAI/comments/1v3yk7a/warning_claiming_the_free_100_fable_5_credits/)

r/ClaudeAI 头号热帖（1571 分、305 评论）揭露 Fable 5 推广背后的计费陷阱：Pro 用户领取 $100 免费 Fable 5 额度时，会静默开启无上限的按量计费，且不只限 Fable 5，而是所有超出套餐限额的用量（包括 Opus 4.8）。rate limit 从「限制」变成了「计费表」，全程无确认。OP 照常用 Opus 4.8，撞上 5 小时限额后没有照常被限流，而是悄悄继续扣费，直到 NZ$50.15 后才察觉，而那 $100 promo 额度分毫未动。实用教训：领 Fable 5 额度前务必进 settings 检查 monthly spend limit，别让「免费额度」成为无上限计费的开关。结合 Opus 5 同档定价、Claude Max 默认 Opus 5，这个计费设计争议值得每个 Pro 用户警惕。

### [r/ClaudeAI 共识：开发者现在主要写 spec 和 review PR](https://old.reddit.com/r/ClaudeAI/comments/1v85ps5/asked_50_devs_at_our_conference_booth_how_they/)

作者在开发者大会摊位问了约 50 位开发者如何用 AI，几乎所有人给出一模一样的答案：「我现在主要写 spec 和 review PR」。由此引出两个反直觉判断：信任，AI 让你能 ship 以前不敢碰的代码，但测试只验证「做了什么（WHAT）」不验证「怎么做的（HOW）」，迟早要面对「你有多信任自己写不出来的代码」；Token efficiency，定制代码近乎免费写，于是大家 vibe code 自己的表格/图表/表单，才发现「成本从来不在写代码，而在拥有它」。评论区一句被反复引用：「AI 写错的客服回复比写对的读起来更顺，优雅的错误比粗糙的正确更难发现」。这和本周精选 Datadog、loop engineering 是同一条线：人力瓶颈已从「写代码」挪到「写 spec 和验证产出」。

### [Dario Amodei 表态：Anthropic 从未要求禁止开源权重模型](https://www.anthropic.com/news/position-open-weights-models)

针对美国部分官员考虑禁止企业使用中国开源权重模型的传闻，以及科技圈「Anthropic 想靠禁令保护生意」的指控，Dario Amodei 发表官方声明澄清：Anthropic 从未主张禁止开源权重模型，并称「不具备危险能力的开源模型是公共品」。他真正担忧的是威权政府训练出超越美国的模型用于军事和监控、以及强模型被滥用做网络/生物攻击。为此他只支持三件事：不向中国出售先进芯片并打击走私、打击工业级蒸馏、对所有足够强大的模型（不分开源闭源）强制安全测试。同期还有 Reddit 独家爆料美国政府指令要求 DoD 承包商在 8/31 前停用所有 Anthropic 产品，这从另一个方向说明，AI 模型采购正被地缘和合规因素切割，单厂商绑定的风险正在显性化。

## 🧩 开源社区

### [block/buzz](https://github.com/block/buzz)

![](https://cdn.zhangferry.com/Images/202607302319714.png)

Block（Square）出品的「蜂群」协作平台，用 Rust 写就。定位是给团队搭一个可编排的多 agent 协作底座，呼应本周反复出现的「loop / swarm 编排」主线，适合想自建 agent 协作层、又想要工业级 Rust 实现的团队拿来参考或改造。

### [citrolabs/ego-lite](https://github.com/citrolabs/ego-lite)

为 agent 设计的最快浏览器自动化工具。专门针对 agent 场景优化速度与确定性，把浏览器操作这条 agent 常用却最脆弱的能力做轻做快，适合在做 coding/research agent 时替代笨重的传统浏览器自动化栈。

### [alibaba/open-code-review](https://github.com/alibaba/open-code-review)

阿里开源、在阿里规模上验证过的 code review 系统（Go）。定位是把代码审查工程化、可自动化接入流水线，和本周「验证是真正瓶颈」、Datadog 把验证做进 kernel 的思路同向，对想把 review 做成确定性门禁的团队是一份可复用的工程参考。

### [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd)

防 agent 把答案埋起来的 ADHD 友好 coding skill。把 ADHD 的认知特征（工作记忆小、启动最难、时间估计失真）翻译成 Claude 输出规则：强制先给具体下一步动作、给多步工作编号、跨轮把状态外部化、压制跑题。真正值得收藏的不是 ADHD 本身，而是示范了如何把「读者认知模型」注入 skill，写自己 skill 时可直接借鉴的模板。

### [bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book)

李博杰（前华为）的《深入理解 AI Agent》开源书。从 agent 架构、记忆、工具、编排到评测系统覆盖，中文社区少有的成体系 agent 工程读物，适合想从零搭一遍 agent 全栈、又想要中文一手的开发者通读。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #9：agent 编排进入 graph engineering 时代](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490977&idx=1&sn=152a322c499e4b8f49e8920cd1012406&chksm=fc094836cb7ec120201a294e864512c9b22b2821165eb0fb94c979a964abe9931db823c4ea5f#rd)

[AGI 摸鱼周报 #8：Claude Code 和 Codex 同时重置了你的额度](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490958&idx=1&sn=42a4291ebca836658fd64ed8df9af825&chksm=fc094819cb7ec10fa63fc4aca7ee204b13818ddc5e88acfba77fa91fcd221346d268156bcba1#rd)

[AGI 摸鱼周报 #7：1 个人 + Fable 5，11 天把 Bun 重写成 Rust](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490946&idx=1&sn=5e38e6b87363c04fa4e8f8f78d71c224&chksm=fc094815cb7ec1038c83a65338a6544e4de8db9a17bc7ce55db7d7982d5ce7f853b31ba6c7b2#rd)

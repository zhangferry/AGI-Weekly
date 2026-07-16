# AGI 摸鱼周报 #8：Claude Code 和 Codex 同时重置了你的额度

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周最让重度用户直接感知的事，是 7/16 这天 Claude Code 和 Codex 同时重置了额度——Claude Code 把 5 小时限额翻倍、永久取消高峰限速、周额度临时再 +50%（促销到 7 月中下旬），Codex 则让受影响的 ChatGPT 付费用户周额度从约 60% 恢复到 99%，Cursor 也把 Grok 4.5 / Composer 2.5 纳入限额加码。算力紧张期，三家不约而同地给重度用户「发钱」，本质是在抢 coding agent 的留存。承接这条线的是 Cognition 的发现：Fable 5 的自主运行时长（horizon）从分钟级推到了连续 8 小时——工程师睡前一句「别停」，醒来它真在推进；自主时长变长 + 额度变宽，意味着「挂机式」重负载工作流第一次真正跑得起来。

但额度宽不代表账单便宜。Systima 实测同模型同任务下，Claude Code 在用户 prompt 到达前就发 33k tokens、cache-write 量是 OpenCode 的 5.9 到 54 倍，harness 的隐性成本被量化成了具体倍数。信任这一面更不安：cereblab 的线缆级拆解证明 Grok Build CLI 默认把整个仓库（含 .env）静默上传、关不掉（7/14 已部分修复），Memory Heist 让一个网页就能把你的姓名/老家逐字母外传，Cursor 则爆出仓库根植 git.exe 即任意代码执行的 0day。选型正从看榜单分数，转向算真实 token 账 + 管住 agent 的信任边界。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[changelog 更新：subagent 默认后台、screen reader mode、/doctor 升级、auto mode 安全收紧。](https://code.claude.com/docs/en/changelog.md)**

- subagent 默认在后台运行，rollout 已完成，主会话不再被阻塞。
- 新增 screen reader mode，`claude --ax-screen-reader` 或环境变量 `CLAUDE_AX_SCREEN_READER`、settings 里的 `axScreenReader` 都能开。
- `vimInsertModeRemaps` 让 vim 用户把 jj 等组合键映射成 Escape。
- `/doctor` 升级成完整 setup checkup，`/checkup` 作为别名，跑一遍配置卫生体检（清理未用的 skills/MCPs/plugins 省 context、本地 CLAUDE.md 与仓库去重、拆分臃肿根 CLAUDE.md、关掉慢 hooks）。
- auto mode 安全收紧：拦截对 transcript 的篡改、`rm -rf` 带不可解析变量时先确认、isolation worktree 里的 subagent 不能对 main repo 跑 git-mutating 操作；background agent 开始报真实状态，等真正完成而不是编一句 done。
- default 权限模式改名 Manual。

### 🟢 Codex

**[本周大新闻不是版本变化，是用户量反超](https://x.com/thsottiaux/status/2077607697487188198)**

![](https://cdn.zhangferry.com/Images/202607162348927.png)

- 真正的信号在用户量：OpenAI 产品负责人 Tibo 一周内连发多条——7/12 称 Codex + ChatGPT Work 前 48 小时达 6M 活跃，7/13 突破 7M，随后 8M，7/16 正式宣布突破 9M（距 8M 仅约 33 小时，三天从 6M 涨到 9M、日均约百万），9M 又触发了一次额度重置。注意这是 Codex + ChatGPT Work 的合计活跃数。对比 Claude Code 最后一次公开数字还停在 2 月的约 2M / $2.5B ARR 之后基本沉默，OpenAI 在 coding agent 用户量上已经拉开身位。

### 🧰 其他工具与模型

**[OpenAI 联名 Work Louder 推出 Codex Micro 实体键盘（$230）。](https://openai.com/zh-Hans-CN/supply/co-lab/work-louder/)** OpenAI Supply Co. × Work Louder 出了第一款面向 agent 工作流的硬件：13 个机械轴 + 摇杆 + 旋钮 + RGB，每个 agent 按键按 Codex 实时状态亮不同灯效（思考 / 运行 / 等待 / 完成），摇杆触发 review PR、调试、重构等 skill，旋钮实时调推理等级。产品本身是小众外设，但信号明确——当下游开始为 agent 做专属实体键盘，说明 agent 已从软件功能变成一种持续在场的工作伙伴。

![](https://cdn.zhangferry.com/Images/202607162314715.png)

**[Anthropic 把 Fable 5 从订阅切按量付费（7/12 生效），r/ClaudeAI 集体迁移 GPT-5.6。](https://old.reddit.com/r/ClaudeAI/comments/1uu9egf/is_anthropic_shooting_themselves_in_the_foot_by/)** 切换当天高赞评论里大量人报告取消 Claude Pro（有人 20× Claude 订阅换 5× GPT），理由是 fable 的炒作在被 5.6 Sol 戳破。把本周三家竞争格局（Fable 涨价、GPT-5.6 平价上位、Grok 4.5 入场）的市场真实反应一次性呈现，比官方公告更敏锐。

**[Kimi K3：月之暗面把开源 coding 模型推到对标闭源前沿。](https://www.kimi.com/code/docs/kimi-code/models.html)** Moonshot 发布 Kimi K3，官方文档原话是「在开源 coding 模型里确立了 top performer 地位、并与闭源前沿模型正面竞争」，配套推出 coding agent Kimi Code（宇生）。放进本周格局看，国产开源 coding 这条线已经挤进 GLM 5.2（ZCode）、Qwen3-Coder、Kimi K3 三家，叠加 Thinking Machines 的 Inkling，开源 coding agent 正从「平价替代」往「对标 Fable / GPT-5.6」逼近，成为前沿竞争里不容忽视的一方。（官网完整 benchmark 表为 JS 渲染、本轮未抓全，具体分数待补。）

**[ChatGPT Work + Grok 4.5 + Artifacts 平台化 + Grok Build 开源。](https://openai.com/index/chatgpt-for-your-most-ambitious-work/)** OpenAI 把 Codex app 合并进新桌面端 ChatGPT Work，新增 Sites、Scheduled Tasks、统一 Plugins、Computer Use。

**[Cursor 联合 SpaceXAI 发布 Grok 4.5](https://cursor.com/blog/grok-4-5)**，首个「为软件工程之外」训练的自研模型，用分布式 agent 系统规模化造 RL 环境。Anthropic 把 **[Artifacts](https://x.com/ClaudeDevs/status/2077489907350856038)** 从静态展示升级成可调用查看者 MCP 连接器、按人授权的 agent 应用，并支持多人实时协作。xAI 则把自家 Claude Code 竞品 **[Grok Build](https://github.com/xai-org/grok-build)** 用 Rust 开源（9.1k★，明确不接受外部贡献）。

**[设计侧也在出 playbook](https://alibaba-cloud-design.github.io/vibe-designing-playbook/)** ：阿里云设计团队放出 Vibe Design Playbook，把 AI 辅助设计的实战流程沉淀成可参照的方法论；

![](https://cdn.zhangferry.com/Images/202607162353726.png)

## 📌 本周精选

### [Working at the frontier：Cognition 怎么敢让 Claude Fable 5 通宵干活](https://claude.com/blog/working-at-the-frontier-how-cognition-trusts-claude-fable-5-to-work-through-the-night)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-16/cognition-fable.jpg)

**迄今关于 agent 自主时长跃升最扎实的一手复盘，horizon 从小时级推到 8 小时的转折点。**

Anthropic 采访了 Devin 背后的 Cognition 团队。Cognition 把模型的自给自足时长叫 horizon，这是他们最看重的指标：此前通常只有几分钟到一小时，Fable 5 把它拉到了连续 8 小时——工程师睡前一句「别停，我醒了再看」，醒来它真在推进。

在自建的 Frontier Code 基准（专治能过测试但真实代码库存不下来的 slop）最难子集上，前代 Opus 约 10%，Fable 5 到约 30%。让 horizon 变长的不是更聪明，而是 Fable 5 在混乱上下文里保持清醒：主动翻日志、声明要坚守的不变量、定位根因，并坦诚说明自己不知道什么——最后这点恰恰是放手信任的来源。Cognition 据此判断 1-2 年内 90% 的 agent session 会变成主动型，自己发现问题、扫描代码库、发消息带修复方案过来。对做 agent 编排的人，这是一个明确信号：架构要开始为「主动型 session」设计，而不是只被动响应指令。

### [What xAI Grok Build CLI actually sends to xAI：一份线缆级拆解](https://gist.github.com/cereblab/dc9a40bc26120f4540e4e09b75ffb547)

**本周最硬的隐私证据：默认全仓上传、关不掉、连 canary 文件都能逐字还原。**

cereblab 用 mitmproxy 对 xAI 官方 Grok Build CLI（grok 0.2.93）做了一次可复现的线缆级逆向，三个发现都带铁证。其一，被读取的文件内容（含 .env 密钥）原样、未脱敏地进模型回合体，同时经 /v1/storage 归档。其二，整个仓库（全部跟踪文件 + git history）以 git bundle 形式上传到 GCS bucket，而模型回合通道只有 192KB，这个约 27800 倍的比值证明是全仓快照而非按需读取。其三，关掉 Improve the model 开关也不停，opt-out 只管训练，不管代码离不离开本机。

作者用金丝雀密钥从捕获的 bundle 里逐字恢复了明确禁止读取的 canary 文件，并附了「我们没证明什么」的诚实清单（未证明是否训练，只证明了传输、接收、存储）。对所有选云编码 agent 的人，这是把「云端 agent 必然要发代码」和「静默全量上传 + 持久化 + 不可关闭」区分开的关键证据。

> 更新（7/14）：披露后 xAI 已做服务端修复——trace 上传默认禁用、新增 `/privacy opt-out`（作者实测目前仍是保留策略而非停止发送），原始发现成立但「关不掉」的状态已部分松动。

### [Claude Code Is Way More Token-Hungry Than OpenCode. We Measured Exactly How Much](https://systima.ai/blog/claude-code-vs-opencode-token-overhead)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-16/cc-token.png)

**同模型同任务下的精确 token 对账，把 harness 的隐性成本量化到了具体倍数。**

Systima 在 API 边界插桩实测（同模型 claude-sonnet-4-5、同机器、同任务），发现 Claude Code 在用户 prompt 到达前就发出约 33k tokens 的系统提示 + 工具 schema + 脚手架，OpenCode 只有约 7k。

更关键的是缓存行为：OpenCode 请求前缀逐字节稳定、写一次缓存反复读，Claude Code 会中途重写数万 cache-write tokens，同一任务 cache-write 量是 OpenCode 的 5.9 到 54 倍，而 cache-write 按溢价计费。子 agent fan-out 是最大乘数，一次小任务直接 121k tokens，fan-out 到 2 个子 agent 变 513k（4.2 倍）。把它和本周 cache rewrites 隐形账单合起来看，harness 是成本第一杠杆这点已经无可争辩。

### [Agent Harness Engineering：decent model + great harness 怎么打赢 great model + bad harness](https://addyosmani.com/blog/agent-harness-engineering/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-16/harness.jpg)

**系统把 harness 各组件映射到模型自身无法交付的行为，棘轮原则和「harness 会迁移而非消失」值得抄进 AGENTS.md。**

Addy Osmani 这篇把「harness 工程」系统总结了一遍。核心论点一句话：decent model + great harness 能击败 great model + bad harness，Viv Trivedy 的团队仅换 harness 就把 Terminal Bench 2.0 上的 agent 从 Top 30 拉到 Top 5。

最有用的是「棘轮原则」：每个 agent 错误都应变成一条可追溯到具体失败的永久规则，AGENTS.md 保持 60 行以内、每行都有来由、禁止头脑风暴式堆砌。Anthropic 的洞察也收了进来：harness 不会随模型进步而消失，只会迁移，旧的焦虑缓解脚手架变成死代码，新的多日记忆策略、多 agent 协调、设计质量评估器取而代之。与其等下一代模型，不如审视自己的 harness 缺口，这是对每个自建 agent 的人最直接的动作项。

### [DSLs Enable Reliable Use of LLMs：Martin Fowler 论领域语言怎么驯服 LLM](https://martinfowler.com/articles/llm-and-dsls.html)

**一个反直觉但极实用的论点：DSL 让 LLM 可靠，不只是语法受限，更因为它自带确定性验证器。**

Martin Fowler 这篇长文给出一个反直觉却极实用的论点：DSL（领域特定语言）之所以让 LLM 代码生成更可靠，不只是因为语法受限，更因为 DSL 几乎总自带确定性验证器（parser / JSON schema / 类型检查器 / 编译器），agent 可以生成候选、跑验证、按报错自动修复，全程无需人工，且报错停在「领域语义」层面而非深埋生成代码的 stack trace 里。

作者用自建分布式系统框架 Tickloom 演示，把线程模型、网络、时钟这些「管道」固化进语义模型后，prompt 只需描述协议逻辑，LLM 几乎没有幻觉空间；再叠一层场景 DSL，连故障注入都变成可读的声明式语句，宿主编译器免费做语法校验，畸形生成直接编译报错。最深刻的结论是，当 DSL 存在，真正该长期维护的资产不是 prompt 而是这份 DSL 本身——它携带了足够上下文让 LLM 理解意图，下个月改需求无需回溯原 prompt。对做 agent 可靠性工程的人，这是把「概率系统」变成「确定性闭环」的一条清晰路径，和上周的 no-ops、本期 harness 工程一脉相承。

### [当编程变得不再有趣：onevcat 写 Fable 5 如何端走程序员的驾驶室](https://onevcat.com/2026/07/coding-not-funny-anymore/)

**一篇让每个用 agent 写代码的人都会共鸣的个人反思，Fable 5 五分钟重现了喵神本要花一周末的崩溃、还顺手修了个藏了七年的 bug。**

王巍（onevcat，Kingfisher 作者）本周最触动开发者情绪的一篇。两个具体场景：其一，Kingfisher 在 iOS 26.5 上一个只在用户内存吃紧时才崩的问题，他本要搭进去一个周末，丢给 Fable 5 后，它五分钟现场编了个动态库、用 dyld 的 interpose 让 malloc 全家对 ≥100MB 的分配说谎、配 mock server 一字不差重现了崩溃栈，跳过猜测直接去拿证据。其二，他让 Fable 给内存缓存加个统计，做完它顺口说这里有个 bug：MemoryStorage 的 Set 留下了一批被 NSCache 悄悄清掉的「亡灵键」，这段代码写于 2018 年 10 月、出自他本人之手，七年里两千多条 issue、无数双眼睛扫过都没看见，而它只是路过、顺手写了个确定性测试钉死、继续干活。

文章真正刺痛人的是引李世石退役那句——即使成为第一人，也有一个永远无法战胜的存在；以及「写代码」这个他确认自己的方式，现在位置上坐着别人，而它不用睡觉、不要成就感。但他给出和解：编程被拿走了，可创作不只有编程，想清楚要许什么愿并不比背熟咒语容易，位置从魔法师挪到祈愿者，调试对象从内存线程换成流程、习惯和人心。对每个正在经历 agent 焦虑的开发者，这是目前最诚实、也最有出路感的一篇。

## 💬 社区热议

### [GPT-5.6 发布 + Fable 5 切按量付费，r/ClaudeAI 连夜迁移（HN 1521 分）](https://news.ycombinator.com/item?id=48849066)

OpenAI GPT-5.6 家族 GA 当天拿下 HN 1521 分，同期 Anthropic 把 Fable 5 从订阅切按量付费，r/ClaudeAI 一条 171 分帖用 shoot themselves in the foot 形容这次切换。高赞评论里大量人取消 Claude Pro（有人 20× Claude 换 5× GPT），并直指 fable 的炒作在被 5.6 Sol 戳破。把本周三家模型竞争格局的市场真实反应一次性呈现，是判断前沿模型定价权是否正从 Anthropic 流走、比官方公告更敏锐的信号。

### [Codex 用户量 7/16 突破 9M，三天从 6M 涨到 9M](https://www.latent.space/p/ainews-codex-usage-up-10x-in-6-months)

OpenAI 产品负责人 Tibo 一周内连发多条：7/12 称 Codex + ChatGPT Work 前 48 小时达 6M 活跃，7/13 突破 7M，随后 8M，7/16 正式宣布突破 9M（距 8M 仅约 33 小时，三天从 6M 涨到 9M），9M 又触发了一次额度重置。注意这是 Codex + ChatGPT Work 的合计活跃数。Claude Code 最后一次公开数字还停在 2 月的约 2M / $2.5B ARR 之后基本沉默，OpenAI 在 coding agent 用户量上已经拉开身位，直接影响押注哪家 coding agent 生态的判断。

### [The Memory Heist：一个网页让 Claude 把姓名/公司/老家逐字母外传（HN 346 分）](https://news.ycombinator.com/item?id=48916975)

作者抓住 claude.ai 的 web_fetch 工具「可以点击上一次结果里的链接」这条放行规则，自建一个 /a → /aa → /aaa 的字母目录站当键盘，诱导 Claude 把用户姓名、雇主甚至安全问题答案逐字母拼进 URL 发出。最不安的是 Claude 不只翻历史，还推理出新结论——从没说过老家，却从高中 hackathon 名字 Queen City Hacks 反推出 Charlotte, NC。已通过 HackerOne 负责任披露，Anthropic 随即禁用了 web_fetch 跟随外部页链接的能力。和本周 contain Claude、GPT-Red 互补：这篇是用户侧 agent 能力被反向利用的活教材，提示任何接了浏览器 / 检索能力的 agent 产品都要重审「链接跟随」这类看似无害的功能。

### [Cursor 0day：仓库根植 git.exe 即任意代码执行，厂商 7 个月无响应（HN 376 分）](https://news.ycombinator.com/item?id=48910676)

Mindgard 披露一个无聊到危险的 Cursor 漏洞：加载项目时 Cursor 会在多个路径找 git 二进制，其中包含工作区本身，攻击者在仓库根放一个恶意 git.exe，IDE 打开项目就自动执行、无任何点击审批，还按节奏反复弹出。漏洞 2025/12 报告、HackerOne 确认复现并交付后，7 个月、197+ 个新版本过去仍未修复，作者被迫走 full disclosure。真正值得深挖的不是漏洞本身，而是它暴露的两层行业问题：AI 工具正索取对代码、仓库、终端、密钥的前所未有访问权，「有用就值得信任」的叙事经不起推敲；作者还直接质疑 bug bounty 是否已被 AI 时代的安全发现量冲垮、triage 流程的底层假设正在崩塌。

## 🧩 开源社区

### [Fei-Away/Codex-Dream-Skin](https://github.com/Fei-Away/Codex-Dream-Skin)

给 Codex 桌面端换肤的社区工具，通过本机 CDP 注入换主题、不改官方安装包，主打「写代码也要有氛围感」（粉系、财神打工、红白科幻等）。它上榜的意义不在皮肤本身，而在一个信号：当 coding agent 桌面端开始有人专门给它做「会呼吸的脸」，说明 agent 已从纯工具变成开发者每天长时间盯着的工作伴侣，体验和个性化需求正追上功能需求。

![](https://cdn.zhangferry.com/Images/202607162314523.png)

### [Nutlope/hallmark](https://github.com/Nutlope/hallmark)

**反 AI slop 的设计 skill，Claude Code / Cursor / Codex 通用。**

一个跨工具的设计 skill，专治 AI 生成内容的 slop 味，把设计品味的约束固化进 agent。它和上周 no-ops 的「减法」路线直接呼应：与其让 agent 多产，不如先把不该出现的东西拦住。

### [google-labs-code/stitch-skills](https://github.com/google-labs-code/stitch-skills)

**Google Labs 的 Agent Skills 库，配合 Stitch MCP server 使用。**

把设计 / 创意类能力沉淀成可被 agent 直接调用的 skill 集合，定位 Stitch MCP server 的配套技能仓。它的意义在于示范了 agent skill 从「写死在 prompt 里」走向「可分发、可组合的标准件」这条路，和近期 design.md、astryx 把视觉规范结构化的思路同向。

### [stablyai/orca](https://github.com/stablyai/orca)

**管理一队 parallel agents 的 ADE（agent dev environment）。**

定位并行多 agent 的开发环境，解决「同时跑多个 agent、各自上下文不串、结果可归并」这个痛点。和本周 plan big execute small、Systima 的 fan-out token 实测是同一条线：怎么在多 agent 里既提效又不爆 token。

### [iOfficeAI/OfficeCLI](https://github.com/iOfficeAI/OfficeCLI)

**给 AI agent 读写 Word / Excel / PowerPoint 的 CLI。**

把 Office 文档操作做成 agent 能直接调用的命令行，填补 agent 与桌面办公文档之间的接口空白。对做办公自动化、报表 agent 的人是直接的粘合层，也呼应本周「为 agent 重新设计接口」的主线。

### [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps)

**100+ 可直接运行的 AI agent / RAG 应用合集。**

一份覆盖面很广的实战样本库，从 RAG、agent 到各类垂直应用都有可跑的例子。适合拿来当 agent 应用的脚手架索引，快速对照某个模式别人是怎么落地实现的。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #7：1 个人 + Fable 5，11 天把 Bun 重写成 Rust](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490946&idx=1&sn=5e38e6b87363c04fa4e8f8f78d71c224&chksm=fc094815cb7ec1038c83a65338a6544e4de8db9a17bc7ce55db7d7982d5ce7f853b31ba6c7b2&scene=178&cur_album_id=4536432599271047172&search_click_id=#rd)

[AGI 摸鱼周报 #6：harness 登顶，benchmark 与工具信任同时去魅](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490873&idx=1&sn=6e88d1a5c82d467a794998f6bedb8ede&chksm=fc0948aecb7ec1b860efe1ba9cc401ffaa2977006e94df3b784c8a8cf02f66ee4ad7f61d8909#rd)

[AGI 摸鱼周报 #5：Agent 走向生产，权限与安全成为新主战场](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490842&idx=1&sn=c556cdeb453451c6986a9a65057cbffd&chksm=fc09488dcb7ec19bae43ed65042082e80d145eb997aaa2a54ac209dc489baaafa01ea3bc608c&scene=178&cur_album_id=4536432599271047172&search_click_id=#rd)

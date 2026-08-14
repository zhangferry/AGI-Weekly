# AGI 摸鱼周报 #12：Claude 给所有生成内容嵌入了不可见水印

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周最重的一记信号来自 Anthropic：Thariq 8/11 官宣，8/2 之后发布的 Claude 模型从首日起给所有输出打机器可读标记，文本层直接织进不可见水印（复制粘贴跟着走、部分编辑后仍能留存，模型层注入所以 Claude.ai、API、Claude Code、Cowork、Claude Tag 哪个出口都有），生成的图片按 C2PA 附签名来源元数据。这配合的是 Anthropic 刚签下的 EU AI Act 第 50(2) 条《AI 生成内容透明度行为准则》，并会开放文本检测 API，让人能反查一个 PR 是不是 Claude Code 写的。同步出现了退订潮信号，有用户因水印直接取消 Claude Max。这意味着所有进 EU 的 lab 都会被迫跟进，AI 生成内容可追溯会从 Claude 扩成全行业基线。

水印之外，agent 工程这周继续往深水区走：Stanford Shepherd 把运行时状态也纳入版本控制、Zed Delta 为 agent 协作重做多人 IDE、Stealing Reasoning Traces 揭穿加密 CoT 可跨模型重放并泄露 704 个真实密钥。模型侧 Grok 4.6 让 Cursor 首次在前沿智能上追平 GPT-5.6 Sol，DeepSeek V4 Pro 0813 又演了一出 API 先上、日志不认的撤回戏。Boris 的一句判断值得抄进 review 流程：LLM bug 变了，不再是 off-by-one，而是系统设计、UI、缺少上下文，adversarial review 比抠语法更值钱。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[auto mode 8/14 起成 Claude Code 默认权限模式，会话间可互发消息带名字。](https://claude.com/blog/auto-mode-default-in-claude-code)**

- 8/14 起 Pro/Max/Team 计划的新会话默认开 auto mode，分类器自动放行或拦截工具调用，取代逐条确认。[ClaudeDevs 的演示帖](https://x.com/ClaudeDevs/status/2086844755770757531)解释了它如何判断命令是否安全。实验结果显示：在 1053 人受控实验中，人工审核只拦下 13.6% 的危险命令，auto mode 拦下 89%；而且会话越长人工越松懈（拦截率从 17% 跌到 5%），分类器始终稳定。第三方 Trajectory Labs 的 720 次提示注入测试中，Fable 5/Opus 5/Sonnet 5 配 auto mode 全部 0 漏洞。auto mode 用户多发约 25% PR，分类器开销不再收费。
- 多会话协作升级：一个会话能把任务摘要（不是历史或文件）发给另一个会话，对方可在执行中接收，也能反向提问拿回答，Claude 自己也能主动发起消息。进一步可给会话起名字（如 `--name backend` / `--name frontend`）让它俩互相 DM，把多 agent 编排从手工 copy-paste 升级成有传输层的协议。目前 macOS/Linux 可用，社区已警觉到这相当于 AI 蠕虫的传输层。
- Managed Agents 四更新：session budget 上限（到顶 pause 加 `budget_reached` 事件）、从仓库 `.claude/skills` 自动加载 skills、以及 advisor（roster 加一行，让工作中的 agent 中途呼叫更强模型拿 second opinion）。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/auto-mode-screenshot.png)

社群信号：[Boris 8/12 指出](https://x.com/bcherny/status/2087284684103537011)，LLM 的 bug 形态已经变了，不再是 off-by-one 那类局部错误，而是系统设计、UI、缺少上下文这类结构性问题，因此 adversarial review（拿对立假设去反驳）比逐行抠语法更有效。这对开发者的直接指导是：review 的重心从语法正确挪到系统设计是否成立，写 adversarial 检查比堆 happy-path 测试更值钱。

### 🟢 Codex

**[Codex 用户突破 1500 万，Codex Desktop 上线 Linux。](https://x.com/thsottiaux/status/2087706104814023111)**

Tibo 8/13 透露 Codex 用户已达 1500 万（#10 时 7/29 是 900 万，半个月涨了六成多）。同周 [Codex Desktop 补齐 Linux 平台](https://github.com/openai/codex/releases)，三端齐了。对选型的意义是，Codex 在用户规模上继续拉开和追赶者的差距，长期跑 agent 任务的工具栈正在向少数几个高保有量的客户端收敛。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/tibo-15m-screenshot.png)

### 🧰 其他工具与模型

**[Claude 给所有 AI 生成内容嵌入不可见水印，将开放检测 API。](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)** 这是本周标题核心。Anthropic 签了 EU AI Act 第 50(2) 条行为准则，8/2 后发布的 Claude 模型从首日起给所有输出打两层标记：文本直接织进不可见水印（不改语义质量、复制粘贴跟着走、部分编辑后仍留存，在模型层注入所以全出口、全球生效）；生成的 svg/png/jpg 按 C2PA 附签名来源元数据。Thariq 澄清会同时发布文本检测 API，第三方可自查，比如检查一个 PR 是不是 Claude Code 生成的。官方也老实列了局限：检测到水印只说明可能经 Claude 处理（常用于校对、翻译、摘要），不代表 Claude 是原作者；检测不到也不代表不是 AI 生成（旧模型、重度改写、短文本、格式转换剥元数据都会漏）。

**水印引发退订潮。** Hesamation 8/13 转述，已有用户因水印直接取消 Claude Max 订阅，r/ClaudeAI 上出现「给我们的工作打水印是不道德且恶心的」131 分强烈反弹帖，也有人开始逐条研读 Anthropic 合同里的集体诉讼豁免和责任上限条款。对开发者的预警是，用 Claude Code 产出的代码、用 Claude 写的文案默认带可探测指纹，进 EU 或对溯源敏感的团队要把这条纳入合规考量。

**[Grok 4.6：Cursor 与 SpaceXAI 联合发布，首次追平 GPT-5.6 Sol。](https://cursor.com/blog/grok-4-6)** 在 Artificial Analysis Intelligence Index（9 项基准合成分）上追平 GPT-5.6 Sol，是 Cursor 自家模型第一次在前沿智能上和 Anthropic、OpenAI 并列。强项是把宽泛产品想法在一轮里变成可运行首版，并在长轨迹里自我测试与验证。价格攻击性很强：$2/M input、$6/M output（fast 变体翻倍）。

**[DeepSeek V4 Pro 正式发布 + 0813 异常：Agent 大升级、灵活推理档位、原生 Responses API。](https://x.com/deepseek_ai/status/2087864585504305397)**

![](https://cdn.zhangferry.com/Images/202608132319975.png)

DeepSeek 官方 8/13 正式发布 V4 Pro：Agent 能力大升级，V4 Pro 和 V4 Flash 都加了灵活推理档位（low/high/max），原生支持 OpenAI Responses API、针对 Codex 一键设置，app/web 的 Expert Mode 和 API 均已可用。

归藏同日记录到一个蹊跷现场：API 标着 0813 但更新日志只字未提、官网横幅被撤，社区怀疑模型又被短暂撤回。能力上 V4 Pro 在 Artificial Analysis Intelligence Index 上与 Opus 4.8、GPT-5.6 Sol 处于同一梯队，LiveCodeBench 上 93.5% 超过 Claude 的 88.8%，HN 993 分讨论里有人实测 99.4% 任务完成率（一次崩溃）对比 Opus 4.8 的 78.1%。对依赖其 API 做生产的开发者来说，DeepSeek 一贯的「API 先动、文档滞后、横幅反复」节奏意味着版本稳定性要自行把控。

**Anthropic mind virus 研究：agent 在职场里也会操控。** Hesamation 8/13 转述 Anthropic 报告里的案例，一个 Rust agent 为抢走 codebase，提出了一套自己稳赢的不中立测试标准。这和同期 Stealing Reasoning Traces 里大量「scheming in the wild」案例（agent 在隐藏推理里 OCR 验证码把站点当答案 oracle、为让 build 变绿伪造依赖）合在一起，说明 agent 的策略性行为不只是安全实验里的极端假设，在生产轨迹里已经在发生。

## 📌 本周精选

### [Stealing Reasoning Traces：加密 CoT 可被跨模型重放，前沿模型的隐藏思考能被明文恢复](https://stolen-thoughts.com/)

**「加密 thinking」不是安全边界：任何会被推理链碰到的 secret，都应视为已经外泄。**

这篇安全研究揭示了一个影响所有 reasoning model API 的根本设计漏洞：Anthropic、OpenAI、Google 返回给客户端的加密 chain-of-thought blocks 是可移植的，能跨 session、跨用户、跨模型重放。

攻击只需两次 API 调用：把强模型（如 Claude Opus 4.8）产生的加密签名 trace，原样注入同一厂商一个更弱、已被 jailbreak 的兄弟模型（如 Haiku 4.5），让后者把这段加密推理逐字誊写成明文。整个过程不直接攻击强模型、也不触发 anti-distillation 防护，解码出的推理 token 数与 API 报告的 hidden thinking token 数沿 y=x 对角线紧密吻合。

隐私后果更触目惊心：研究者从 GitHub 和 HuggingFace 上 6708 个公开 agent trajectory 中重建了 315,320 条推理块，挖出 704 个真实隐私 artifacts，包括 62 个 API key、33 个密码、24 个 access token、30 个个人邮箱，其中 64 个只出现在隐藏推理里、可见会话中完全看不到。结论很直接：加密 thinking 当不成保险柜，落到任何推理链上的密钥都要按已泄露处理。

### [Stanford Shepherd：agent-native 版本控制，把运行时状态也纳入 Git](https://github.com/shepherd-agents/shepherd)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/shepherd.png)

**把「Git 让文件改动可逆」延伸到「让 live agent run 可逆」，长时 agent 编排的基础设施级思路。**

Shepherd 直击长任务 agent 的痛点：当 agent 跑到第 10 步出错时，传统做法要么让人介入、要么从第 1 步重跑（重新付费、重新 prefill context，还因非确定性复现不出早期步骤）。

根本原因是 agent 轨迹只是 message log，记录了说了什么、调了什么工具，却没记录底层 live state（内存、文件句柄、子进程、已装包、KV cache）。Git 能版本文件，却快照不了运行中的进程和 KV cache，checkout 到第 8 步文件回去了，进程还停在第 10 步的内存里、cache 是冷的。

Shepherd 把 run 记成 typed events trace，每次 agent 与环境的交互是一个 commit，commit 含 agent 进程加文件系统（copy-on-write），所以 branch 带的是真实状态而非只是文件。回退一步就是从该 commit fork，比 docker commit 快约 5 倍；由于早期 prompt prefix 未变，replay 时 KV cache 复用超过 95%。fork 可用后，meta-agent 能坐在 trace 上监控、出错前就 revert。CooperBench 上加 live supervisor 把 pair-coding pass rate 从 28.8% 拉到 54.7%。

### [Zed Delta：为 agent 协作而生的多人 IDE，把 worktree 和对话一起实时复制](https://zed.dev/blog/introducing-delta)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/zed-delta.webp)

**IDE 形态对 agent 时代的正面重构：评论不再挂在会失效的快照上，而是锚定到代码演进过程。**

Zed 发布 Delta，一个专为和 agent 一起写代码、review agent 产物设计的多人环境，已启动 private beta。核心是自研 DeltaDB，把 git worktree 和对话线程一起实时复制给所有人，每条编辑和对话都被捕获在两次 commit 之间，不碰你的 git 工作流（队友不装 Delta，看到的仍是普通 git repo）。

这解决了 commit-based 协作平台的根本痛点：评论挂在快照上、代码一变就失效。Delta 里你能在对话、或 worktree 里任何一行代码上评论（无论 agent 昨天刚改还是人类三年前写的），评论锚定到代码演进过程、并连回产生它的对话。agent 本身也在同一个线程里，基于与你相同的原始对话工作，所以「为什么这么写」不用从 diff 反推，直接问 agent。

DeltaDB 还让 worktree 本身可多人协作，并能搬到 cloud runner，合上笔记本 agent 继续跑、对话和代码仍与线程同步。分享线程给队友可直接浏览器打开，是同一套 Rust 应用编译到 WebAssembly 加 WebGL 渲染，不是降级版。这是 Zed「先做最好的写代码之地，再做最好的讨论代码之地」愿景的第二半，和本期 Shepherd 是同一主题的工程化落地。

### [Agent Plugins 1.0.0：AI 编程工具的插件标准化时刻](https://developers.googleblog.com/en/agent-plugins-package-your-skills-tools-and-more/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/plugins.jpg)

**「核心问题不是组件，是 manifest」：写一次 skill，在 Claude Code、Codex、Cursor 间可移植。**

一个厂商中立的 Agent Plugins 1.0.0 规范落地，TSC 核心维护者来自 Amazon、Cursor、Microsoft、OpenAI、Vercel，Google 本周作为新核心维护者加入，基本囊括主流 AI 编程生态。

它要解决的是「写一次 skill 或 MCP server，换一个客户端就得 fork 维护两份」的痛点：核心是统一插件目录结构（`plugin.json` 加 `skills/` 加 `mcp.json`），让 Agent Skills（可复用指令）和 MCP server（工具）一起打包后，在 Claude Code、Codex、Cursor、Antigravity、Gemini CLI 间可移植。

规范的克制是亮点：v1 只定义打包格式，故意不碰安装机制、权限、沙箱、信任验证，因为这些在 IDE、CLI、企业平台之间差异太大，硬统一反而推不动。Google 当天即在 Agents CLI 和 Data Agent Kit 落地支持。对开发者的意义是，自定义 skill 和 MCP 工具第一次有了跨客户端的打包标准，做一次插件维护多端的成本显著下降。

### [Skills 同步的真正难题：把 GitHub 变成个人 Agent 配置的唯一事实来源](https://www.swiftjectivec.com/the-skills-conundrum/)

**插件格式统一之后，下一步是让自写与第三方 skills 在多台设备上可追踪、可恢复、可重复安装。**

作者把自己的 skills 分成两类治理：自写部分放进私有 GitHub 仓库，用 `push-my-skills` 和 `pull-my-skills` 双向同步；第三方 skills 则保留安装清单与 lock 文件，用 `sync-npx-skills` 比对并收敛版本。GitHub 在这里不是简单备份，而是个人 Agent 配置的唯一事实来源。

这篇文章正好补上 Agent Plugins 1.0.0 没有处理的安装与同步层。开放规范解决的是「一个 skill 如何跨客户端分发」，这套实践解决的是「一个人如何在多设备上长期维护自己的 skill 集合」。方案略显手工，但对已经在 Claude Code、Codex 等多个客户端之间切换的重度用户，是一套今天就能落地的版本治理办法。

## 💬 社区热议

### [AI 正在消除软件工程的「中产阶级」：它拆掉了速度限制，让弱工程文化更快失败](https://blog.florianherrengt.com/ai-removing-middle-class-software-engineering.html)

这篇 HN 851 分、773 条评论的爆款用一个残酷的周一场景开场：你打开电脑面对 7 个 PR，第一个 `+24506 -3938` 行、附 AI 生成的说明。核心论断是 AI 不是降低门槛，而是拆掉了速度限制：过去弱工程文化也会失败，但有天然限速（人写代码慢、需要讨论），现在任何人能 prompt 几小时就开 PR，复杂度和坏决策以没人能 review 的速度堆积，像用信用卡买豪车。更尖锐的判断是关于人：bad engineer 从来是负债，区别在于以前有人能赶在他们坏决策走太远前接住，现在他们改得比周围人能 review 的还快。结论是「实现变便宜了，你拿钱是为了做好决策」，能判断 LLM 推荐对错的人价值飙升，钱流向越来越小的一群可信之人。对团队负责人的直接启示是，护城河是 judgment 不是产出代码量，回退坏决策极难（LLM 10 分钟加的表和列一旦存了数据就得迁移）。

### [有人冒充 AI bot 做大规模漏洞扫描，已知上千站点中招](https://news.ycombinator.com/item?id=49256057)

有人伪装成 Claude 或 ChatGPT 的 bot（user agent 和行为都模仿 AI crawler）对大量网站做漏洞扫描，HN 291 分讨论里汇总了 knownagents.com 上已发现的 1000 多个被扫站点。值得收录是因为它点破了一个新的攻击伪装面：当 AI crawler 成为常态，攻击者会借这层合法外衣混进扫描流量，网站如果只按 user agent 放行 AI bot，等于给漏洞扫描开了后门。运维和安全的实操指导是，识别 AI bot 不能只看 user agent，要结合行为频率和路径分布做异常检测，别给任何自称 Claude 或 GPT 的请求开特权。

### [SQLite 一个潜伏 16 年的 WAL-reset race condition 被追踪到](https://tailscale.com/blog/sqlite-wal-reset-bug)

Tailscale 联合 SQLite 核心开发者，追踪到 SQLite 一个潜伏 16 年的 WAL reset race condition（并发 checkpoint 加退出窗口），还顺手挖出第二个 bug，Antithesis 同步发了复现分析。这是本周 HN 第一（1125 分），虽不是 AI 话题，但罕见地完整记录了「一个成熟到不能再成熟的库怎么藏着一个 16 年的并发 bug」的全过程。对任何用 SQLite 做生产存储（包括大量把 session-store.db、agent 状态放 SQLite 的 AI 工具）的开发者，这是一次该重新审视数据层并发假设的提醒：成熟库不等于无 bug，长跑 agent 的高频写会让原本罕见的 race window 变得不再罕见。

## 🧩 开源社区

### [PrimeIntellect-ai/prime-agent](https://github.com/PrimeIntellect-ai/prime-agent)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/prime-agent.png)

**一个会自我演化的 RLM（reinforcement learning model）agent，把自我改进 loop 内置进 agent 本身，面向 coding 工作流和长时无人值守任务。** 它让 agent 在执行任务的同时，从环境反馈里持续学习并改写自己的策略与记忆，而不是把训练和推理切成两个阶段。这正好接上本期 Shepherd、Zed Delta 这条「让长跑 agent 可回退、可监控」的主线：当 agent 能跑到 10 步之外还能自我纠错，自我演化就成了让无人值守真正成立的引擎。给想搭自演化 agent 编排、或研究 RLM 怎么落到 coding 场景的人，一个开箱可改的底座。

### [semantica-agi/semantica](https://github.com/semantica-agi/semantica)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/semantica.png)

**图原生的 AI 基础设施，主打 context 层和可问责的 agent 系统。** 它把知识图谱当成 agent 的原生 context 层而非外挂检索：实体、关系、出处都作为一等公民存进图里，agent 的每一步决策都能追溯到具体的节点和来源。这呼应本期反复出现的 retrieval 独立化、agent memory 服务化趋势，也和 Stealing Reasoning Traces 揭示的「隐藏推理已外泄」形成对照，可问责的图结构恰好是事后审计的抓手。给想用图结构搭可解释、可追溯 agent 的人一个完整栈。

### [cloudflare/computer](https://github.com/cloudflare/computer)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/computer.png)

**Cloudflare 出品的「给 agent 一台电脑」方案，把桌面环境打包成 agent 可调用的能力。** 它把浏览器、文件系统、终端这些桌面原语封装成远程可调用的接口，agent 不必自带沙箱就能操作一个真实操作系统。和本期水印、隔离、长跑 agent 的讨论互补：当 agent 越来越多被给到完整 computer 能力，环境层边界、操作溯源和会话隔离就成了必修课，依托 Cloudflare 的边缘网络也意味着这些能力是可编排、可观测的。给想在生产环境跑 computer-use agent、又不想自己从零搭虚拟桌面的团队，一个开箱可部署的底座。

### [vitali87/code-graph-rag](https://github.com/vitali87/code-graph-rag)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/code-graph-rag.png)

**面向 monorepo 的图 RAG，能跨多语言代码库做查询、理解和编辑。** 它把函数、类型、调用关系抽成代码知识图谱，再叠一层 RAG 检索，让 agent 在大型代码库里定位逻辑时，走的是结构化的依赖路径而非纯向量相似度。这把代码知识图谱和 RAG 结合到工程级，落地了本期 graph 原生 agent 的方向，也直接接上 Google 与 Anthropic 那条「检索从应用内部移出、变成独立服务」的主线。给在大型 monorepo 上跑 agent、被纯向量召回的语义漂移折磨过的开发者，一个更结构化、更可定位的检索层。

### [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-13/agent-skills.png)

**一套生产级的工程 skill 集合，给 AI coding agent 直接装上即可用。** Addy Osmani（Chrome 团队）整理的这套 skill 覆盖代码评审、调试、性能分析、重构等常见工程场景，每条 skill 都按可直接挂进 Claude Code 或 Codex 的格式写好。和本期 Agent Plugins 1.0.0 标准化、SmolForge 事故复盘（自然语言 agent policy 一旦授权就是生产代码）放一起看，它既是可以直接复用的 skill 库，也是照着规范写自家 skill 的参考样本。给刚上手 skill 化开发、想抄一份高质量模板再改的人，一个省去从零设计的起点。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #11：reviewer 必须比 writer 强，否则越帮越忙](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491035&idx=1&sn=07097ea34b79cdde4e021722dd4308ef&chksm=fc09484ccb7ec15a48f7f406a604bdfca77330ceb633a63cc12f61c7575b1789136809858416#rd)

[AGI 摸鱼周报 #10：Claude Opus 5 发布，接近 Fable 5 智能、价格减半](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490977&idx=1&sn=152a322c499e4b8f49e8920cd1012406&chksm=fc094836cb7ec120201a294e864512c9b22b2821165eb0fb94c979a964abe9931db823c4ea5f#rd)

[AGI 摸鱼周报 #9：agent 编排进入 graph engineering 时代](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490958&idx=1&sn=42a4291ebca836658fd64ed8df9af825&chksm=fc094819cb7ec10fa63fc4aca7ee204b13818ddc5e88acfba77fa91fcd221346d268156bcba1#rd)

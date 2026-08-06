# AGI 摸鱼周报 #11：reviewer 必须比 writer 强，否则越帮越忙

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周最强的信号来自一个反直觉实验：让 Claude 给 Codex 当 code reviewer，pass rate 从 71.6% 拉到 89.7%；可反过来让 Codex review Claude，反而从 91.4% 跌到 82.8%。差距不在「多一个 agent」，而在 review 的干预质量。更强的模型当 reviewer 才有用，hierarchy 比「加个第二意见」更重要。这把 agent 选型从采购问题变成了组织问题。

同一周，Claude Code 之父 Cherny 把长时无人值守的关键归结为 verification 而非 prompt engineering，Google 和 Anthropic 不约而同把 retrieval 从应用内部剥离成 agent 调用的独立服务，micro 的 Mu 则把 30 多个日常功能打包成 agent 可即调即付的 MCP 工具。agent 编排正从「能不能跑」进入「怎么验证、检索放哪一层、应用怎么原生」的工程深水区。国产前沿模型同日分化：智谱因算力成本把 Coding Plan 涨了 6 到 8 倍，DeepSeek 靠工程把单 token 算力降到原来的 27%、维持白菜价。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[/code-review 拆出 ultra 编队，worktree 隔离拦住 destructive 命令，auto mode 安全继续收紧。](https://code.claude.com/docs/en/changelog.md)**

- `/code-review` 一次性补了两层：effort 分级，以及最顶的 `/code-review ultra`，召唤一整支 reviewer agent 编队，对每个发现独立复现验证，Anthropic 内部每个 PR 都跑 ultra。`/review` 成了它的别名。
- 子 agent 现在能嵌套到第 3 层（原来默认只到 1 层），`/fork` 改成开新 worktree，不再占用当前 checkout。
- 安全一连串收紧：隔离 worktree 里对主 checkout 的 destructive git 命令被拦；auto mode 的 `SendMessage` 先过权限分类器再发；Remote Control 不再允许靠 repo-local 配置打开，只能在用户级开。

社群信号：ClaudeDevs 7/30 通报，24 小时内两次独立网络故障导致 Claude 部分请求失败、可用性下降，流量重路由后恢复。本周遇到 Claude 报错或变慢，大概率是这次平台波动而非本地配置问题。

### 🟢 Codex

**[Codex v0.146.1 给 cyber-capable 模型上更严 auto-review 默认，GPT-5.6 Luna 永久降价 80%。](https://github.com/openai/codex/releases)**

v0.146.1（8/5）给 cyber-capable 模型套上更安全的自动 review 默认，并在终端里解释权限变化的原因，对拿 Codex 做安全研究的人是看得见的收紧。配套还有 GPT-5.6 Luna 永久降价 80%，Tibo 强调不是临时噱头、效率提升不会回收，现在 $0.20/$1.20，对比 GPT-5.4 的 $2.50/$15。

社群信号：Tibo（[@thsottiaux](https://x.com/thsottiaux)）8/6 在给创业者的建议里点名推荐 `/goal` + GPT-5.6 Sol 组合，称它「a pretty powerful loop」。官方产品负责人亲自示范：要跑 Sol 的长任务，开 goal 模式比裸跑更稳。

### 🧰 其他工具与模型

**[阿里 Qwen3.8-Max 8/3 正式发布，2.4T 参数、下周开源权重。](https://qwen.ai/blog?id=qwen3.8)** Qwen 团队 8/3 正式放出 Qwen3.8-Max，2.4 万亿参数（95B active），是 Qwen 家族目前最强、也是第一次把 Max 级权重开源（下周放权）。基于 Qwen 3.5 架构，主打 coding、work、research 和 long-horizon 任务，强调端到端把复杂任务做成可交付的成品。最值得拆的演示是一个 10 多天的自主编码长跑：从零搭 oh-my-cli，做一个会自我演化的 harness，把用户反馈、社区实践和模型自测结果汇进同一个工程 loop，issue 自动认领执行，完整 trace 开放在 GitHub `qwen-code-dev-bot/oh-my-cli`。和本月持续发酵的 GLM-5.2、Kimi K3 一起，国产开源前沿 coding 三家正面对逼 Fable 和 GPT-5.6。

**[智谱 GLM Coding Plan 7/31 回归并改积分制，三档涨幅达 6 到 8 倍。](https://docs.bigmodel.cn/cn/coding-plan/notice/usage-revision)** 智谱 7/31 把 GLM Coding Plan 订阅重新上架，从原来的按 Prompt 次数计量改成以 token 消耗为基础的积分制，Lite 档 118 元/月起。新版套餐价格结构性上调，据多方核对比对旧连续包月价：Lite 涨约 6 倍、Pro 约 7.6 倍、Max 约 8.3 倍，海外版（Z.ai）也同步向海外定价看齐。官方给出的理由是算力成本上升，调整旨在让额度计算更透明可预期；老用户在当前订阅周期内维持原价和旧规则，8/15 前包年 7 折、包季 8 折。公告见 bigmodel.cn 官方文档。

**[DeepSeek V4 Flash 7/31 转正，性能逼近 Opus 4.8、价格不变。](https://api-docs.deepseek.com/zh-cn/updates)** DeepSeek 同一天把 V4 Flash 预览版转成正式版（V4-Flash-0731），模型架构和参数规模不变、只做了重新后训练，Agent 能力大幅提升，代码修改、工具调用和多步骤任务处理上甚至超过 V4-Pro Preview，整体性能超过 GLM-5.2、逼近 Opus 4.8。价格维持不变：输入未命中缓存 1 元/百万 token、命中缓存 0.2 元、输出 2 元，输出价约为 Gemini 3.1 Pro 的 1/43。官方透露单 token 算力消耗降到 V3.2 的 27%，靠工程侧优化把低价维持住，广受社区好评。官方更新日志见 deepseek.com API 文档。

本周国产前沿模型价格明显分化：智谱因算力成本上调订阅、DeepSeek 靠工程降本维持白菜价，同一天发布的两个信号说明，选型时「便宜」不再等于某一家，要看每家当下是在涨还是在降。

## 📌 本周精选

### [Your AI-coding agents might need an org chart（Claude review Codex 让 pass rate 71.6%→89.7%）](https://leaddev.com/ai/your-ai-coding-agents-might-need-an-org-chart)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/org-chart.webp)

**给 agent 团队排 org chart，这周有了第一个硬数据：reviewer 必须比 writer 强，否则越帮越忙。**

一项受控实验让 Claude Opus 4.7 和 Codex GPT-5.5 组成 writer-reviewer 流水线，跑 116 个 LiveCodeBench 中高难度 Python 任务。Codex 单独 71.6%，交给 Claude review 后升到 89.7%；但反过来，Claude 单独 91.4%，交给 Codex review 反而跌到 82.8%。

差距源于 review 时的干预质量：Claude 修好 26 个 Codex 的失败、只弄坏 5 个，Codex 只修好 3 个却弄坏 13 个，作者比作「初级工程师改写资深工程师的代码」。第二个 agent 也不是免费的，单任务成本从 $0.19 涨到 $0.44，延迟从 38.5 秒涨到 112.4 秒。

可见 agent 选型更像组织决策而非采购决策：你总想让更强的模型来 review，hierarchy 比单纯「加个第二意见」更重要。搭多 agent 流水线时，reviewer 档位要刻意压 writer 一头，而不是图省事让同级模型互查。

### [Claude Code 之父 Cherny：长时 autonomous agent 的关键是 verification](https://daringfireball.net/linked/2026/08/02/cherny-claude-swift)

**「现在的技能不是 prompt engineering，而是怎么给 Claude 一个看似过难的任务，并让它能验证自己的工作。」**

Cherny 在 YC Startup School 2026 上被问到当下用 Claude Code 的核心技能，他的回答是 verification 是大家最容易做错、也最重要的一环。他给了一个极具参考价值的实例：用新产品 Claude Tag（跑在 Slack 里的 Claude）发起会话，让它在 GitHub macOS runner 上跑 Electron 版 Claude desktop app 截图，与 Swift 重写版逐像素对比，「done 前不许停」，这个任务已经自主跑了两周还在跑。

这套「难任务 + 自动 verification loop + 长时无人值守」正是当前 autonomous agent 的主流范式。Gruber 的批评也值得借鉴：把「设计糟糕的 Electron 应用」用更好的框架重写，只是换了食材的烂菜谱，pixel-by-pixel 复刻根本没解决设计问题。想用 agent 做「换皮重写」的人，从这里能看到 agent 能跑通却仍可能白忙的典型路径。

### [LLMs reward expertise：最重要的 prompting 技能是领域专业知识](https://www.seangoedecke.com/llms-reward-expertise/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/expertise.jpg)

**和 LLM 对话不需要技能？作者反驳：prompting 最重要的一项技能，就是你对那个领域的专业理解。**

Sean Goedecke 拿 Terence Tao 用 ChatGPT 探讨 Jacobian Conjecture 反例的对话做案例：Tao 消息极简短，用专业气场把模型切到「与数学家对话」而非「给业余解释」的模式，push back 时说「这看起来比我预期的复杂」而不是直接否定，几乎不采纳模型建议的下一步。

但这些表面技巧模仿不来，关键在于 Tao 真能从模型的多段回复里拎出相关想法、提出替代表述、识别哪里不对劲。推论对开发者和产品人同样成立：对自家代码库有清晰心智模型的人，能把 LLM 推得远比无熟悉度的人更狠。作者也坦承 HN 评论里有人质疑这是「自我安慰」，他认可这种警惕。

### [Google 与 Anthropic 的共识：retrieval 正从应用内部移出，变成 agent 调用的独立服务](https://x.com/akshay_pachaar/status/2083815836003996033)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/retrieval.jpg)

**当 agent 成为中心，检索不再是「启动时一次性灌进上下文」，而是按需、可替换、可共享的外部能力。**

这是一条凝练的架构观察：Google 和 Anthropic 在 retrieval 上不约而同走了同一条路，把它从应用进程里剥离出来，变成 agent 显式调用的独立服务。Anthropic 的 MCP 把 retrieval 暴露成一个 agent 主动调用的 tool，Google 推出独立的 RAG Engine 服务，agent 按需检索。

这条推文点破了一个跨厂商的架构收敛信号。设计 RAG 或 agent 系统时，应优先把检索层做成独立的、有清晰边界的 service（无论是 MCP server 还是专用 RAG 服务），而不是耦合在某个应用里。同账号同期另一条「AI engineer 该学什么」（roofline model、decode 为何 memory-bound、vLLM/SGLang scheduler、从代码读懂 paged attention、先做 observability 再优化、追踪 TTFT）也值得收藏，是动手派的高密度清单。

### [Mu：micro 出品的 everything app，把整套个人应用暴露成 MCP 工具供 agent 调用](https://github.com/micro/mu)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/mu.png)

**agent 原生的应用平台长什么样：30 多个日常功能打包成 MCP 工具，按请求微付费，没有 API key 也没有账号。**

micro（Tim O'Reilly 的公司 micro.xyz）开源的 Mu 自称 everything app，blog、chat、news、mail、weather、video、search 全塞进一个 Go 二进制，无广告、无追踪、无算法推荐，可自部署或用 mu.xyz。

对 AI 开发者最有意思的是副标题 Tools for Agents：Mu 同时暴露 REST API 和 `/mcp` MCP server，30 多个工具（news、search、weather、places、video、email、markets）可被任何 MCP 兼容 agent 接入。更激进的是付费模型，AI agent 可通过 x402 协议用 USDC 按请求付费，无需 API key、无需账号，call and pay。把日常消费级功能（看天气、读新闻、查行情）打包成 agent 可即时调用并微付费的工具集，这是一种还没被大规模验证、但方向清晰的 agent 原生应用形态。AGPL-3.0 协议。

## 💬 社区热议

### [CLAUDE.md for Opus 5：按 Anthropic 官方新规则重写一份精简版（r/ClaudeAI 431 分）](https://old.reddit.com/r/ClaudeAI/comments/1vd57c0/claudemd_for_opus_5_based_on_anthropics_official/)

紧接 Anthropic 那篇「为 Claude 5 删掉 80% 系统 prompt」的官方文，有用户照着官方新规把一份精简版 CLAUDE.md 整理出来分享，431 分登上 r/ClaudeAI 热门。它和同期「delete claude.md」「是否还需要 CLAUDE.md」等帖子合在一起，把「配置文件到底该写多少」顶成本周 r/ClaudeAI 最热的话题。值得围观是因为它给了一个可照抄的极简模板：从「什么都往里塞的中央仓库」转向按需加载的文件树，正是这周上下文工程主线在用户侧的落地样本。

### [Cursor 个人版 Usage 页面去掉美元显示，改用 token 数（HN 317 分）](https://news.ycombinator.com/item?id=49135257)

Cursor 把个人版 Usage 页面里的美元金额换成了 token 计数，官方解释是 Ultra 改用量计价后、个人版不再展示费用（企业版仍显示）。HN 上 317 分的讨论集中在抱怨：在团队共享额度的场景下，个人开发者想追踪自己到底花了多少变难了。这条值得放进来，是因为它和 Codex 频繁 reset、Cursor Router 主打省钱的同期信号一起，说明用量计价正在取代固定额度，账单透明度成了用户和厂商新的拉扯点。

### [字节 Seedance 2.5：单次 30 秒、多模态引用、时间戳级编辑的视频生成（HN 247 分）](https://news.ycombinator.com/item?id=49138302)

字节 Seedance 2.5 视频模型在 HN 拿到 247 分：单次生成 30 秒，多轮可扩展到数分钟，并支持最多 30 张图、10 段视频、10 段音频的多模态引用，以及时间戳级别的精细编辑。已上线即梦 AI、豆包 Pro，API 即将通过 BytePlus ModelArk 开放。值得留意的是它把多模态输入和时间戳编辑放到了产品级，国产视频模型在「长时长 + 多素材融合 + 可控编辑」这条线上继续对逼海外，是本周少数冲上 HN 高分的国产产品。

## 🧩 开源社区

### [TencentCloud/TencentDB-Agent-Memory](https://github.com/TencentCloud/TencentDB-Agent-Memory)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/tencentdb-agent-memory.png)

**腾讯云出品的团队级 agent memory hub，把对话、文档、代码沉淀成四种可复用的记忆资产。** 它把零散上下文结构化为可检索、可共享的记忆，正好呼应本周 retrieval 从应用内部移出、变成独立可调用服务的主线。给想给 agent 加一层团队级长期记忆、又不想自己从零搭存储和检索的人，一个开箱可改的底座。

### [different-ai/openwork](https://github.com/different-ai/openwork)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/openwork.png)

**基于 opencode 的 Claude Cowork 开源替代，给不想等官方的人一个可自部署的多 agent 协作底座。** 这周 Cherny 把 Cowork 形态（一个主 agent 调度多个子 agent、各自带验证 loop）说成是长时 autonomous 的主流范式，openwork 把这套形态做成开源版直接对接，方便想自托管、或想拆解 Cowork 内部编排的开发者拿来研究和改造。

### [antirez/ds4](https://github.com/antirez/ds4)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/ds4.png)

**Redis 作者 antirez 亲手写的 DeepSeek 4 Flash 与 PRO 本地推理引擎，覆盖 Metal、CUDA、ROCm 三套后端。** 本周 DeepSeek V4 Flash 转正、国产前沿模型热度正高，这个项目补上了「在 Mac 或自建 GPU 上本地跑国产旗舰」的顺手工具，作者本人的工程品味也意味着它在性能和代码可读性上比一般 wrapper 更值得读。

### [virgiliojr94/book-to-skill](https://github.com/virgiliojr94/book-to-skill)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/book-to-skill.png)

**把任意技术书 PDF 自动转成一套 Claude Code skill，让 agent 边干活边查书。** 这是把长文档结构化成 agent 可按需加载知识的一个干净样本，恰好落地本周反复出现的 progressive disclosure 思路：知识不一次性灌进上下文，而是按场景触发对应 skill。对想给自家文档做 agent 化改造的人是个现成模板。

### [esengine/DeepSeek-Reasonix](https://github.com/esengine/DeepSeek-Reasonix)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/deepseek-reasonix.png)

**DeepSeek-native 的终端 coding agent，核心卖点是 prefix-cache 稳定、适合挂着长跑。** 它针对 DeepSeek API 的前缀缓存做了工程稳定性优化，让长会话不易因缓存失效而重置上下文，正好接上 Codex 本周频繁 reset、社区呼唤「能长时间无人值守」的痛点。想用 DeepSeek 当主力模型跑长任务的人值得一试。

### [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-06/i-have-adhd.png)

**一个把「读者认知模型」写进 prompt 的 skill，专治 coding agent 把答案埋进一堆过程里。** 它把 ADHD 的工作记忆小、启动最难、时间估计失真等特征翻译成 Claude 输出规则：强制先给具体下一步、给多步工作编号、跨轮把状态外部化、压制跑题。值得收藏的不是 ADHD 本身，而是它示范了如何把「读者认知模型」注入 skill，写自己 skill 时可直接套用的模板。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #10：Claude Opus 5 发布，接近 Fable 5 智能、价格减半](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490977&idx=1&sn=152a322c499e4b8f49e8920cd1012406&chksm=fc094836cb7ec120201a294e864512c9b22b2821165eb0fb94c979a964abe9931db823c4ea5f#rd)

[AGI 摸鱼周报 #9：agent 编排进入 graph engineering 时代](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490958&idx=1&sn=42a4291ebca836658fd64ed8df9af825&chksm=fc094819cb7ec10fa63fc4aca7ee204b13818ddc5e88acfba77fa91fcd221346d268156bcba1#rd)

[AGI 摸鱼周报 #8：Claude Code 和 Codex 同时重置了你的额度](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490946&idx=1&sn=5e38e6b87363c04fa4e8f8f78d71c224&chksm=fc094815cb7ec1038c83a65338a6544e4de8db9a17bc7ce55db7d7982d5ce7f853b31ba6c7b2#rd)

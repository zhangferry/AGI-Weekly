# AGI 摸鱼周报 #19：OpenAI dots 发布，Personal AI 领域迎来新的竞争者

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

如果 AI 不等你提问，而是记住你的目标，在你离开电脑后继续工作，会是什么样？OpenAI 9 月 29 日发布的 dots 给出了一个具体答案：每个 dot 有自己的云电脑和浏览器，能跨应用接任务、保留上下文，并通过已连接应用在后台寻找值得提醒你的事。它的主动研究只使用只读工具；要发消息、改动内容等，则受权限规则和审批约束。[Meta Muse](https://about.fb.com/news/2026/09/introducing-muse-personal-ai-agent/) 和 [Grok Bot](https://x.ai/news/introducing-grok-bot/) 已先后进入 personal AI 市场，dots 的发布让 OpenAI 也加入了这场竞争。

这种常驻关系也把老问题推到台前：它该记住什么、能访问哪些账户、什么时候主动行动、哪些决定必须留给人。OpenAI 同期还发布了 GPT-6 智能界面、GPT-6.1 Sol 和 Decisions API，但本期最值得单独看的是 dots 所代表的产品形态。后文精选拆解它的工作方式与控制边界；长会话的上下文成本、团队内的 agent 权限，以及 Reddit 收紧 RSS 入口，则是让这类 agent 真正可靠所需处理的现实问题。

## 🚀 产品与模型动态

### 🟠 Claude Code

本周从 2.1.283 推进到 2.1.287，五版：

- **Claude Mods 发布**（2.1.287）：插件可以修改更深层的行为了。首个内置示范是「You should know」：一个侧 agent 持续盯着你和 Claude 的对话，标记双方可能漏掉的东西，`/plugin enable` 即可开启。Thariq 此前宣布的「plan mode 改造为 built-in mod」是官方意向，截至本版尚无 changelog 条目印证，落地节奏待观察。
- **Sonnet 5.5 发布并成为默认 Sonnet**（2.1.284）：1M 上下文，$2/$10 per Mtok，cache read $0.20。
- **1M 上下文成为默认**（2.1.287）：Opus 4.7+ 与 Fable 在 Bedrock、Vertex、Foundry 和 apps gateway 上默认使用 1M 窗口，不再需要 `[1m]` 后缀；自定义 `ANTHROPIC_BASE_URL` 的会话同样生效。
- **交互会话默认 auto mode**（2.1.284/283）：没有配置权限模式时，终端与 VS Code 会话在所有计划与 provider 上直接以 auto mode 启动，这是行为变化最大的一条。
- **5h 限额新增「收尾配额」**：到点不再硬切 mid-edit，而是寻找优雅停止点把当前任务收完（官方 X 披露，changelog 未单列）。
- 其他：`claude --desktop` 一条命令在桌面 app 打开当前目录（2.1.285）；`/doctor prompt-audit` 审计 CLAUDE.md、skills、commands 里过时的 prompt 模式（2.1.283）；shell 经 repo 内 symlink 写敏感文件需要人工确认（2.1.287）；Claude Marketplace 上线，2000+ 插件与连接器，committed spend 可直购 Cursor 等 Claude-powered 软件。

社群信号两条。Thariq 公布 effort levels 使用结论：「low 用于留在环上，max 只留给零输入任务和安全审计，结果让我自己都惊讶」；他同时定调 plan mode 将改造为 built-in mod，mods 可添加新 mode 或接管 shift+tab，模式系统向用户开放。

### 🟢 Codex

- **GPT-6.1 Sol 成为默认模型**（0.159.1，与 DevDay 同日）：bundled catalog 与 Bedrock 目录的默认模型同步切换；0.157.0 起 GPT-6 Sol/Luna 进入 Codex 并支持 Amazon Bedrock。
- **instant_interrupt**（0.159.0，opt-in）：模型响应或长 code-mode 调用期间可被新输入打断并转向，不用等它说完；同版移除了自动 follow-up 建议。
- **0.160.0**：agent command center 支持键盘浏览更早的历史任务；Guardian review 新增 opt-in 能力，可检索更早的用户指令并纳入 agent handoff 的上下文；项目外起 session 时按 workspace 默认值运行并恢复已保存的权限。
- **GPT-6 Prompt Caching 更新**：默认缓存命中率提高，新增缓存仪表盘与 miss 诊断、显式断点；同一会话中调整 reasoning effort 不再打断缓存。缓存输入 token 最高可享 90% 折扣，长会话 agent 可据此减少重复计算和首 token 等待。[官方说明](https://openai.com/index/better-prompt-caching-for-gpt-6)
- DevDay 前的 Codex 宕机 12 小时后全员限额重置补偿。

### 🧰 其他工具与模型

- **GPT-6.1 Sol 发布**：以约五分之一成本提供接近旗舰的能力，DeepSWE 追平 GPT-6 Astra；DevDay 同场发布的 Decisions API 把 Jev 式决策模型正式产品化，上周还靠社区复现铺开的赛道进了官方价目表。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/gpt61-sol.png)
- **Anthropic 联合 NVIDIA 推出 agent 多层安全方案**：Claude Managed Agents 与 OpenShell，把运行时隔离与凭证管理打包成企业可部署的参考架构。



## 📌 本周精选

### [OpenAI dots 发布：一个会记住目标、在后台持续工作的个人 AI](https://openai.com/index/introducing-dots/)

**dots 的变化不在于多一个聊天入口，而在于把云电脑、跨应用上下文和主动发现任务装进同一个长期运行的 agent。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/dots.png)

OpenAI 于 9 月 29 日发布 dots，并开始向符合条件地区的 ChatGPT Pro、Business Premium 用户逐步开放，Enterprise 可由管理员启用测试。dot 由 GPT-6 Astra 驱动，有自己的云电脑、浏览器和已连接的应用，可同时推进多个项目；你能在 ChatGPT、Slack 或 Teams 里继续同一段工作，也能随时打开它的电脑查看进展。官方给开发者的例子是：持续观察用户反馈，整理小修复，完成构建和测试，再带着 PR 与演示视频回来请人审阅。

更关键的是「主动」的边界。dot 可以在后台用已连接应用做只读研究，发现值得提醒你的线索，但这条路径不能发消息、改内容或控制电脑。进入实际执行时，应用权限、Custom Rules 和自动审查决定它能否继续，敏感操作仍需用户批准或亲自完成。官方举例：一个测试者的 dot 发现漏开的发票，准备好后经本人批准才发送。它也不会默认接触你的本机文件，只有你授权连接后才能使用本机。

第一个 dot 包含在 Pro 或 Business Premium 计划中，但深度工作有额度，发起 Codex 或 ChatGPT Work 任务仍计入各自用量。此前 [Muse](https://about.fb.com/news/2026/09/introducing-muse-personal-ai-agent/) 和 [Grok Bot](https://x.ai/news/introducing-grok-bot/) 已经入场，dots 让 OpenAI 成为 personal AI 市场的又一个竞争者。接下来值得比较的是：谁能把长期任务真正做完，以及用户能否看见、约束和接管它的行动。

### [OpenAI 推出 GPT-6 与智能界面：ChatGPT 开始为任务生成交互工具](https://openai.com/zh-Hans-CN/index/gpt-6-for-everyone/)

**GPT-6 不只生成回答，也能按任务组织图表、按钮和可交互界面，让 ChatGPT 从对话框走向可操作的工作界面。**

![](https://cdn.zhangferry.com/Images/202610090830761.png)

OpenAI 于 10 月 7 日启动 GPT-6 分阶段上线：Plus、Pro、Business 和 Enterprise 使用 GPT-6 Sol，免费版与 Go 使用 GPT-6 Luna。新能力「智能界面」会根据问题组合文字、图表、按钮、表单或交互式讲解，也能即时生成计算器等小工具；简单问题仍可直接用文字回答。GPT-6 还能边思考边分段作答，官方称其处理网页搜索问题时，Instant 档开始回答的平均时间比 GPT-5.6 Instant 缩短 44%。本次更新针对 ChatGPT「聊天」体验，不改变 Codex 和工作模式所用模型。

### [Anthropic 官方发布 Opus 5.5 提示词与 harness 设计指南：effort 校准、无人值守陷阱与多 agent 时间预算信号](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)

**5.5 的默认 medium 已达到甚至超过 Opus 5 的 high，直接沿用旧 effort 设置等于默默多花钱。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/prompt-guide.jpg)

Anthropic 针对 Opus 5.5 发布了完整的官方提示词指南，系统披露了一批此前未公开的 harness 工程细节。effort 档位跨模型不对应：5.5 的默认 medium 已达到或超过 Opus 5 的 high，直接沿用旧设置会导致更长回合和更高成本；且修改顶层 effort 会使 prompt cache 失效，需改用 per-message effort change（beta）保持缓存命中。无人值守 agent 的最大陷阱是 text-only 的 `stop_reason: "end_turn"`：那可能是进度汇报而非任务完成，官方建议用 checklist 跟踪任务分项、最多自动续跑 2-3 次，并在系统提示词中显式点名要避免的提前停止类型。对多 agent 团队，给模型注入时间预算信号（如 `elapsed 340s / 1200s`）能让团队明显更快完成且质量不降，机制是促进并行而非削减工作量，与降低 effort 有本质区别。其他要点：progress updates 以 thinking blocks 返回、默认 display 下文本为空，需设 `display: "updates"`；用户粘贴的外部内容用随机 ID 标签包裹可显著提升抗注入能力；新的 `reasoning_extraction` 拒答类别会拦截要求复述内部推理的提示词。

> Effort level names don't correspond to the same amount of thinking across models: in Anthropic's testing, Claude Opus 5.5 at `medium` matches or exceeds Claude Opus 5 at `high`... A tighter budget has a different effect from a lower effort setting: lowering effort reduces the work itself, whereas a budget mostly keeps more agents working in parallel.

### [1674 个 coding agent 会话复盘：省 token 要先管输入，不是输出](https://www.avanderlee.com/ai-development/reduce-token-usage-claude-code-codex-cursor/)

**作者抽查自己 1674 个会话发现，回复只占 0.6% token，主要浪费来自上下文反复传入。**

SwiftLee 作者分析了 RocketSim、RocketTrace 等项目的 agent 会话记录，94.2% token 属于缓存读取，只有 0.6% 是模型回复；文件有一半会话被重复读取，超过一半的文件读取直接装入整份文件，构建与测试输出也常未经筛选。

他把优化重点放在减少无用输入：先搜索再读必要行、复用未变文件内容、过滤 Xcode build 输出、移除重复 skill，并把项目专属规则拆到按需读取的局部 `AGENTS.md`。这些是作者自己的工作负载数据，不能直接外推成所有工具的平均值，但足以说明只缩短回答往往碰不到主要成本。

### [Asana 披露 human-agent teams 全套设计模式：agent 权限以触发者为上限、共享记忆读写分离](https://claude.com/blog/agents-you-can-coach-how-asana-builds-human-agent-teams-with-claude)

**「代码生成不再是瓶颈，瓶颈是规划、决策与打磨」，Asana CPO 交出企业级 human-agent 团队的完整设计模式。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/asana.png)

Anthropic human-agent teams 系列第三篇，Asana CPO 一手披露企业级 agent 编排实践。核心设计决策有三条：agent 不另建上下文体系，直接在 Work Graph（任务/项目/目标的关系网络）内运作，像人类同事一样被分配任务、出现在 activity feed；agent 的有效权限以触发者的权限为上限，公开内容可宽泛访问而私有上下文不泄漏；共享记忆做读写分离，人人可给任务级反馈，但只有 admin/editor 能写入永久记忆（通讯团队持有品牌语气 agent 的「笔」）。

最有分量的是踩坑复盘：Asana 曾因自动 coding loop 生成大量变更导致发布周期失控，结论是瓶颈已经从写代码转移到规划、决策与打磨，随后用 Command 流程管理 agent 循环：agent 从客户反馈填充 unplanned board，人决定进 cycle，系统给出乐观/平衡/保守三档工期预估，ticket 可分配给人或 coding agent，全部数据经 MCP server 暴露给 Claude。

三个落地案例（Slack 产品问答路由、at-risk 续约每日 digest、工程周期规划）都强调同一原则：把 agent 工作放在人人可见的共享任务里，否则评审者看不到 prompt 与来回，无法对齐。

## 💬 社区热议

### [Reddit 将停用 RSS：AI 资讯采集与社区机器人要重做入口](https://mjtsai.com/blog/2026/10/06/reddit-to-remove-rss/)

Reddit [官方公告](https://www.reddit.com/r/modnews/comments/1wubgvt/continuing_our_infrastructure_updates_whats/)确认，11 月 13 日起不再支持 RSS，理由是这一入口被用于大规模抓取和自动化滥用。官方建议版主把提醒迁至 Devvit 的 Discord Relay，却明确说普通用户订阅非自管社区没有替代方案。另一份[开发者公告](https://www.reddit.com/r/redditdev/comments/1wubcvf/moving_data_api_apps_to_the_developer_platform/)给出独立的 API 迁移时间线：10 月 31 日停止接受新的公共 API 访问申请，2027 年 1 月 12 日开始关闭未登记应用的访问，3 月结束剩余公共 API 访问。

争议不只在 RSS 阅读器。Michael Tsai 汇总的反馈中，开发者 Jeff Johnson 质疑为何不能继续限流，并表示停用 RSS 后会离开 Reddit；Unread 开发者则指出不少用户靠 RSS 追踪社区。在此前 [r/rss 的讨论](https://www.reddit.com/r/rss/comments/1tqpf5r/reddit_officially_admits_to_prepping_the_end_of/)中，有用户说自己靠订阅和过滤追踪数十个 subreddit；[r/redditdev](https://www.reddit.com/r/redditdev/comments/1wubcvf/moving_data_api_apps_to_the_developer_platform/) 的版主开发者担心现有 Python/PRAW 机器人缺少可行的 Devvit 对应能力。对用 subreddit RSS 做 AI 日报、agent 调研或舆情发现的流程，这意味着要尽快盘点依赖、准备合规替代来源，并给采集失败加告警；版主提醒的迁移方案无法直接替代跨社区订阅。

### [Opus 5.5 重度用户「$100 计划一周烧掉数千美元等值 token」，订阅制终结争论再起](https://old.reddit.com/r/ClaudeAI/comments/1wqtnng/it_is_scary_thinking_the_era_of_these/)

r/ClaudeAI 热帖：重度用户实测一周用掉数千美元等值 token，结论是「订阅制不是会不会结束，而是什么时候」。这正是 Opus 5.5 时代的新矛盾：模型越能干、agent 跑得越久，固定价格订阅的算力成本越撑不住。

社区最高赞的反驳视角值得一看：订阅是 loss leader， Anthropic 真正的生意是 API 和企业合同，Max 订阅是获客漏斗而非利润中心；对照官方披露的企业基线（$13/开发者/活跃日）看，重度用户的个人消耗仍在可承受的获客成本范围内。对你的启示是评估口径：按 token 等值算「亏没亏」没有意义，按「本月产出的工作是否值回订阅费」算才有意义。

### [tptacek 离职创业：受众 1-2 人的软件世界正在到来，现代 OS 的核心假设随之瓦解](https://sockpuppet.org/blog/2026/09/25/what-even-is-an-os-now/)

安全圈老将 tptacek（sockpuppet）发长文宣布离职创业。核心判断：当软件可以为 1-2 个人的具体需求定制时，现代操作系统「隔离陌生人的应用」这一根本假设开始瓦解，因为 agent 时代你运行的不是陌生人的应用，是你自己的意图。

他要造的是一台「不运行固定功能应用」的手机。这篇文章把 agent 安全讨论带回「操作系统为谁设计」：当本地 agent 可以代人运行代码，文件权限、进程隔离和行为审计该如何设置？这还是一个待验证的系统设计方向，但对构建 coding agent 的人已是具体问题。



## 🧩 开源社区

### [mvschwarz/openrig](https://github.com/mvschwarz/openrig)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/openrig.png)

本周新增约 2,912 星。OpenRig 把 Claude Code、Codex 和 Pi 组织成可持续运行的 agent 团队：用 YAML 定义角色，由 lead agent 分派任务，成员共享上下文并保留各自工作状态。适合想从多个终端会话升级到可管理多 agent 协作的开发者。

### [heygen-com/hyperframes](https://github.com/heygen-com/hyperframes)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/hyperframes.png)

本周新增约 3,908 星。HyperFrames 将 HTML、CSS、媒体素材和动画渲染为可确定性复现的 MP4，也提供 CLI 和面向 coding agent 的 skills 工作流。适合希望把视频制作纳入代码生成、版本管理与自动化渲染流程的团队。

### [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/claude-mem.png)

本周新增约 2,607 星。claude-mem 捕获 agent 会话过程，再用 AI 压缩并在后续任务中检索相关上下文；项目目前支持 Claude Code、Codex、OpenCode 等多种编码 agent。它针对的是跨会话记忆断层，适合经常在多个任务间切换、希望减少重复交代背景的用户。

### [EpicGames/raddebugger](https://github.com/EpicGames/raddebugger)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-01/raddebugger.png)

本周 Trending 排名第 4，新增约 257 星。RAD Debugger 是面向 Windows x64 的原生多进程图形调试器，配套自研 RDI 调试信息格式与 RAD Linker；Epic 称其链接器针对超大型工程，在调试信息达数 GB 的测试中链接速度快约 50%。目前仍处于 Alpha，适合处理大型 C/C++ 工程、希望深入观察编译与调试链路的开发者。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 [GitHub 仓库](https://github.com/zhangferry/AGI-Weekly)

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

## 📜 往期推荐

- [AGI 摸鱼周报 #18：Jev 是什么？不生成文本，只做判断，便宜一百倍](https://zhangferry.com/weeklys/agiweekly_18/)
- [AGI 摸鱼周报 #17：模型越强，harness 越轻还是越重？](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491343&idx=1&sn=663c70b2836cfcb067c2a4a171953b7f&chksm=fc094a98cb7ec38e18c444b7612ddc2ba130bf749cd5c5e8e86e01595ecbb2ea49794f370188#rd)
- [AGI 摸鱼周报 #16：什么都不装的 agent，比用 25 万星的 skill 效果更好](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491311&idx=1&sn=89dce75fe691d7fb04e6abd6e01832c5&chksm=fc094b78cb7ec26eeb75e92001adfdc6ec3d859ed0b3a67095813c4d3b64956381c06cd86275#rd)

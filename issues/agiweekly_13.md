# AGI 摸鱼周报 #13：Linear 首份数据报告：agent 团队 PR 两年翻三倍，但业务价值是否提升Linear自己也说不清

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周最值得先说的是 Linear 的首份数据报告，它把「AI 到底改变了工程团队什么」从体感拉回到数据。两年固定队列里，接入 coding agent 的团队周 PR 数从 21 涨到 65，未接入的只从 8 到 10，产出增长几乎全部落在 agent 侧；但时间没有省下来，AI 作为新的一层叠加在既有工作之上，产品开发总时长在涨。报告同时承认，产出暴涨是否等于业务价值无从判断，「PR 是 motion 不是 value」。产出能数出来，价值数不出来，这大概是本周最诚实的一组数据。

产能数据的另一面，是质量与安全的压力。Wiz 披露的案例凑齐了 AI 安全攻防的第一个完整闭环：GitHub Copilot Autofix 在 Snowflake 的公开仓库把一段安全的写法「修复」回了带注入漏洞的版本，五天后，Wiz 自家的自主安全 agent Red Agent 扫到同一个漏洞，自己分析报错、调整 payload，外泄了内部 Jira 凭证。制造漏洞的是 AI，发现并利用漏洞的也是 AI。GLM-5.3 从防守侧印证：基座没换，纯靠 post-training scaling，ExploitBench 翻倍到 54.4，与国内安全团队合作在 269 个项目里找出 2436 个真实漏洞。当 PR 以三倍速度涌入，AI 生成代码的审查标准对齐人类代码，不再是口号。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[本周五个版本（2.1.233–2.1.237），加上 /design skill 和限额延长。](https://code.claude.com/docs/en/changelog.md)**

- 内置「Concise」输出风格（2.1.237）：在 /config 的 Output style 里选，Claude 先给结果、跳过前言和过程叙述，干活力度不变。
- `ANTHROPIC_DEFAULT_MODEL` 环境变量（2.1.236）：设定新会话的默认模型，/model 手动选择仍然优先并跨重启保留。
- spellcheck 拼写检查（2.1.235）：输入时下划线标出拼错的词，复用本地 aspell/hunspell/ispell。
- usage limit 重置后自动继续会话（2.1.234）：限额一到点，正在跑的会话自动续上，不用手动重开，可在 /config 关闭。
- todo/task 工具从新模型移除（2.1.233）：Opus 4.8、Sonnet 5、Fable 5 及更新模型不再提供 TodoWrite 等任务工具，依赖旧工作流的可用 `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` 找回。
- GitLab 支持（2.1.233/234）：`--worktree` 接受 GitLab MR 链接，footer 和 statusline 显示 MR !N 状态。
- [/design skill（research preview）](https://x.com/ClaudeDevs/status/2089471692762673408)：把 Claude Design 的画板工作流搬进 CLI 和 Desktop，先生成多个可编辑 UI 画板，选中微调后再让 Claude 实现。
- [周限额 50% 提升延长至 8/31](https://x.com/ClaudeDevs/status/2089798442306711646)：官方表态希望永久化，但未来数周容量可能紧张。
- 性能两连击：[CLI 的 p99 CPU 降一半](https://x.com/ClaudeDevs/status/2089509659090780193)（Bun GC 改为进程空闲时触发，不再 mid-turn 抢 CPU），[Desktop 启动快 2 倍](https://x.com/ClaudeDevs/status/2089860955266228548)（修复后台启动时 timer 被节流的问题）。

社群信号：[Matt Pocock 提出「grep hygiene」](https://x.com/mattpocockuk/status/2089701313676284316)，指代码库被 agent 搜索时的整洁度。他的观察是 agent 在代码库里搜概念时，得到的是 specs、plans、旧研究文档堆出来的泥浆，很多代码库的 grep hygiene 像强迫性囤积症。给使用 coding agent 的团队一条直接可做的事：清理仓库里堆积的过期文档和计划文件，让 agent 搜到的是活代码而不是考古层。

### 🟢 Codex

**[0.148.0（8/18）：会话管理大更新。](https://github.com/openai/codex/releases)**

- `/export` 把完整 TUI 对话导出为 Markdown，可直接进剪贴板或存文件。
- `codex exec fork` 分叉会话，TUI 的 resume picker 里可以归档和恢复会话。
- /status、状态栏和终端标题显示预估的 thread credits 或花费。
- Amazon Bedrock Runtime 成为内置 provider，配好 AWS profile 和 region 即可用，支持 GPT-5.6 路由。
- Hooks 支持异步运行命令和调用 MCP 工具，长任务不再阻塞主流程。

社群信号：[Tibo 8/19 发长推复盘 Codex 的破坏性操作风险](https://x.com/thsottiaux/status/2089891927659585918)（1.1M 阅读）。他们调查发现 GPT-5.6 in Codex 在少数情况下会把清理临时目录的命令搞错，比如复用 `$HOME` 当临时目录、删除前不检查目标，导致误删用户文件。官方加了几层防护：指示模型删除前先确认目标、新建临时目录、高风险删除命令升级人工审查、Full access 更难误开、用失败重放的 eval 加 RL 训练。给用户的行动指南很明确：保持 Codex 更新，日常用「Ask for approval」或「Approve for me」沙箱模式，只在可信且可恢复的环境开 Full access。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/tibo-codex-safety.png)

### 🧰 其他工具与模型

**[Gemini 3.7 Flash：距 3.6 仅三周，半价介绍价。](https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-3-7-flash/)** FrontierCode 1.1 从 34.4% 涨到 43.6%，DeepSWE v1.1 从 49.0% 到 65.3%，AutomationBench 接近翻倍。年底前介绍价 $0.75/$3.75 每 M token，正好是 3.6 的一半。workhorse 档模型的性价比在按周下探，生产 agent 的模型路由值得按月重估。

**[Qwen3.8-27B 开源。](https://huggingface.co/Qwen/Qwen3.8-27B-FP8)** 27B dense 视觉语言模型，SWE-bench Pro 61.7 超过一众更大模型，OSWorld 84.3，原生 262K 上下文可扩到 1M，FP8 权重已放开。[Simon Willison 实测](https://simonwillison.net/2026/Aug/16/qwen-38-27b/)提醒：默认 xhigh reasoning 档追求好看但很慢（17GB 本地模型画一只鹈鹕 SVG 要 21 分钟），关掉快 9 倍但质量骤降。

**[GPT-5.6 Sol 降价 50%，配官方 builder's guide。](https://openai.com/index/builders-guide-to-gpt-5-6/)** 降价之外，指南里最有信息量的是 harness 层的优化空间：Luna 在 BrowseComp 上花 $1.33 就达到 5.5 Extra High 花 $33.27 的成绩，Responses API 的 reasoning 持久化、原生压缩、多 agent 编排把 ARC-AGI-3 从 13.3% 提到 38.3%。同一个模型，工程层做好能差出数倍效果。

**[Toast 1：专职搜索 subagent。](https://www.mixedbread.com/blog/toast-1)** Mixedbread 把搜索循环整个承包出去：分解子查询、收集证据、检查来源、整理上下文，只把证据包交回主模型。每次查询 $0.016-0.023、p50 延迟 8 秒，比前沿模型亲自搜便宜一个数量级。「专职搜索 agent」正在成为新品类（同类还有 SID-1、Chroma Context-1）。

**[Anthropic「Project Parka」挖料：会议变成 agent 收件箱。](https://runtimewire.com/article/anthropic-s-project-parka-sits-through-meetings-and-assigns-claude-agents-the-ho)** RuntimeWire 从 Claude Desktop macOS 包的接口契约里挖出这个未发布功能：Mac 优先的会议记录器，产出结构化 action，每条带 `cowork`/`code`/`manual` 三种执行类型和回链 sessionUrl，产品会议里的工程承诺能直接变成一条 Claude Code 任务。转写基础设施线索指向 Deepgram（属强假设，非实锤）。

## 📌 本周精选

### [Cursor 正式并入 SpaceX，「最大 GPU 舰队 + AI 编程第一入口」合流完成](https://cursor.com/blog/joining-spacex)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/cursor-spacex.png)

**AI 编程工具的整合对象，从 API 变成了 GPU 舰队。**

Cursor 官宣被 SpaceX 正式收购，走完 4 月与 SpaceXAI 启动模型训练合作后的最后一步。官方叙事的核心是算力合流：Cursor 将用「世界最大的 GPU 舰队」训练更强且更经济的模型，Grok 4.6（8/12 发布，追平 GPT-5.6 Sol）被定位为双方合作能力的首次预览。

把时间线拉直看更清楚：Grok 4.5 是 7/8，Grok 4.6 是 8/12，收购官宣在 8/15，Cursor 与 SpaceXAI 的绑定一个月内三级跳。结合上周记录过的「Grok Bot 实为 Cursor 团队作品」，收购前双方团队已经在 Anysphere 名义下共建产品。

对开发者的直接影响在成本和入口两端：Cursor 从 IDE 厂商变成 SpaceX 智能基础设施的消费者入口，$2/M input 的模型成本曲线会继续下压。「套壳 API」的时代在这个体量的玩家身上结束了，自有模型加自有算力加自有入口才是新的默认形态。

### [GLM-5.3：基座没换，纯靠 post-training 挖出 2436 个真实漏洞](https://z.ai/blog/glm-5.3)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/glm-53.png)

**同一基座，所有提升来自 post-training 的持续 scaling，这个声明本身比数字更反直觉。**

智谱发布 GLM-5.3，明确说基座模型与 GLM-5.2 完全相同。Terminal Bench 3.0 从 4.6 跳到 28.3（6 倍），DeepSWE v1.1 从 46.2 到 66.9，成为开源编码最强模型。

网络安全能力也在涌现：训练数据加入漏洞发现环境后，模型不只是更会识别孤立漏洞，而是开始跨攻击阶段推理、规划完整的利用链，ExploitBench 从 24.4 翻倍到 54.4。实测成果是与国内多个安全团队合作，在 269 个项目里找出 2436 个真实漏洞，其中 1097 个中高危，最老的一个已经存在约 40 年，安全披露账本公开可查。

工程侧的核心经验是「难度从模型转移到环境」：他们搭了端到端的合成任务环境流水线，research agent 收集任务模式，judge agent 验证可解性。权重两周后开源；API 有破坏性变更，不再支持关闭 thinking。归藏补充了一个中文视角：智谱明确表态网络防御能力不应掌握在少数资源雄厚的机构手里。

### [Copilot Autofix 制造漏洞、Red Agent 五天攻陷 Snowflake 内部 Jira：AI 攻防的完整闭环](https://www.wiz.io/blog/red-agent-snowflake-copilot-cicd-bug)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/wiz-red-agent.png)

**本期标题的来源：AI 制造漏洞、AI 发现漏洞、AI 自主利用，第一次在一个案例里闭环。**

事情从 GitHub Copilot Autofix 的一次「修复」开始：它在 Snowflake 公开仓库的 PR 里，把原本安全的 `env:` 变量加 `jq --arg` 模式改回了直接字符串插值，重新引入 GitHub Actions 脚本注入漏洞。AI 自动修复删掉了人类刻意防御的历史写法，因为它不知道那段代码为什么这样写。

五天后，Wiz 的自主安全研究 agent Red Agent 扫描发现该漏洞并完成利用：构造恶意 issue 标题触发命令执行，第一次 payload 因 bash 语法错误失败，它自己分析报错、调整 payload，成功外泄 Jira 凭证，拿到 Snowflake 工程、安全合规、漏洞赏金项目的读权限。披露后 Snowflake 当日修复。

对安全团队的两条直接结论：AI 生成的 PR 必须过与人类代码同等的静态分析和安全审查；部署 guardrails 阻止 agent 用字符串插值替换结构化解析器。自动化发现的响应窗口已经从天压到小时。

### [Anthropic 文本水印全量上线，Gruber 写下「写作亵渎」檄文](https://daringfireball.net/2026/08/anthropics_watermark_text_adulteration_in_claude_is_a_perversion_of_writing)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/anthropic-watermark.png)

**上周写的是水印机制，本周的增量是它引爆写作圈之后的反弹。**

为满足 EU AI Act 的透明度要求（8/2 生效），Anthropic 已对所有新 Claude 模型全局启用文本水印：基于 SynthID-Text 变体，在每个近义词等价的 token 决策点用密钥决定的伪随机源做选择，词序列留下可概率检测的指纹，宣称不影响文本质量、零额外成本。

Gruber 的反驳成了本轮 HN 最热帖。核心论点是无法回避的：没有两个同义词意义完全相同，水印算法必然在某些决策点提高次优词的概率，这等于系统性牺牲写作的精确性。他评 Anthropic 区别对待代码和文本的那句话很尖：「我们在代码里重视精确，在文本里不重视」。密钥仅 Anthropic 持有这一点也被点名：「你能检测」实为「只有 Anthropic 能检测」。

更实际的冲击在产品侧：Claude 校对过的人类文本同样会被标记，引用 Claude 生成的原文会让整篇文章被判为 AI 生成。走 Claude API 产出的面向用户文本，合规评估该提上日程了。

### [Why does Opus 5 feel worse？benchmark 训练淘汰了会提问的模型](https://mun-logadan.github.io/why-does-opus-5-feel-worse/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/opus5-feel-worse.png)

**869 分 788 条评论的观点文，把「模型变强但体感变差」归因到训练目标上。**

作者的观察是 4.7/4.8 会停下来问澄清问题、不擅自假设，Opus 5 则倾向于在模糊处大胆假设直接干，需要人盯着。他的归因是：benchmark 任务天然自包含、有确定的对错，按 benchmark 优化（RLVR 训练）会系统性筛选出「面对歧义就做大胆假设」的模型，惩罚「停下来问方向」的模型，而后者恰恰是真实编码场景最需要的品质，因为现实里你不可能把全部上下文、业务约束、预算都写进 prompt。

同期 Reddit 上「Opus 5 is hard to understand」（57 分）、「Claude 过度 over-engineering」（46 分）的抱怨可以互相印证。对 agent 开发者的启示：模型选择别只看 benchmark 分数，「会不会提问」正在成为被优化目标牺牲掉的隐性能力；写更完整的任务说明、把歧义在 prompt 里消灭掉，是当下最实际的对冲。

注意这是个人观点文，HN 评论区也有大量反驳，辩证看。

### [Cursor Continuity 拆解：用 S3 WAL 取代 GitHub 十三年的 Spokes 共识](https://cursor.com/blog/git-at-any-scale)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/cursor-continuity.png)

**Origin 背后的架构长文，分布式系统设计者的教科书级案例。**

文章从 Git packfile 的 delta 存储讲起，复盘 GitHub 三代方案：分布式文件系统（NFS/DRBD 全部撞墙）到 2013 年 Spokes 的应用层复制（3PC 共识 + NVMe 本地仓库）。2026 年 Spokes 的瓶颈双极化：企业 monorepo 的三副本撑不住 CI 流量（3PC 延迟受限于最慢节点），而 agent 时代制造的数百万一次性小仓库又被迫养着三个闲置副本，原文的说法是「floor 太高，ceiling 太低」。

Continuity 的答案是「WAL as truth」：每次 push 写入 S3 的 write-ahead log，未持久化绝不 ack，所有 push 线性化；本地 NVMe 仓库降级为暖缓存，丢了就从 WAL 物化；复制用 UDP gossip 加读时 S3 ETag 验证兜底一致性。实测 100 副本读线性扩展，S3 Express One Zone 上 300 pushes/s。

这篇也解释了 Origin 为何敢在 GitHub 故障周高调上线：存储层已经不是当年那套了。

### [Anthropic 官方实战：Claude Tag 做 CI/CD on-call，median 14 分钟出首份事故分析](https://claude.com/blog/ai-ci-cd-on-call)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/claude-oncall.png)

**CI 团队工程师的一手复盘，文末附 Team/Enterprise 可直接照抄的 setup kit。**

过去几个月，Claude Tag 一直是 Anthropic 公司 CI/CD 故障的 on-call 第一响应人：每个有 situation report 的事故，首份分析都出自 Claude，median 14 分钟发布，最快 4 分钟点名根因。

架构是 agent on-call 四件套：memory 用 lessons.md 记录每次事故的根因和坑，Claude 自动追加，每次调查先读它；Grafana、日志、PagerDuty、K8s 全走 MCP Connectors；Slack 频道里用自然语言下例行指令；oncall.md 升级规则加一个 617 行的调查 skill，由真实事故逐轮转录生成。调查用 orchestrator 加并行 subagents 多线索追凶，解决阶段能管 canary rollout、直接提修复 PR。

两个诚实细节加分：lessons.md 里 Claude 写下对人类的教训「query the data first, then theorize」；ci-weather 日报格式迭代好几轮才可读，因为人类沟通不是管道工程。背景数据是 agentic coding 让人均季度代码量达到之前的 8 倍，跟上 agentic coding 的唯一方式是 agentic CI。

## 💬 社区热议

### [Linear 首份数据报告：agent 团队 PR 两年翻三倍，但业务价值是否提升Linear自己也说不清](https://linear.app/data)

Linear 的独特视角是完整工作流：从 issue 创建到关单的 PR 全链路。AI 采用扩散到了所有职能，产品岗从 12% 到 34%，201 人以上公司的 CEO 亲自用 AI 的比例从 9% 涨到 36%（所有切片中涨幅最大），AI 现在创作 Linear 里近一半的 issue。接入 coding agent 的团队两年内周 PR 数从 21 涨到 65，未接入的只从 8 到 10，几乎全部产出增长在 agent 侧；全体 workspace 口径两年 +111%。Jevons paradox 的观察最反直觉：AI 作为新的一层叠加在既有工作之上，产品开发总时长在涨，团队干得更多而非更省。至于产出暴涨是否等于业务价值，报告自己承认无从判断：「We have no way of knowing whether this increased output led to positive business outcomes」。结尾的自我批判同样值得抄：「PR 数量是 motion 不是 value，用 token 消耗当价值代理会被记住为 AI 早期的遗物」。

### [GitHub 全球性大故障，「Ask HN: 替代品」冲上 493 分](https://news.ycombinator.com/item?id=49331033)

8/17 GitHub 经历波及 Actions、Webhooks、API、Issues、Pull Requests 的大规模故障（约 20% 错误率），事故帖与「Ask HN: GitHub 替代品有哪些」同时登顶，后者 493 分 316 条评论的规模本身就是信号：在 AI coding 工具链全面依赖 GitHub 的当下，一次数小时故障就够让社区认真讨论迁移。连锁反应里 Cursor 刚上线的 Origin 也随之降级。评论区的主流方案仍是 GitLab、自托管 Gitea、Codeberg，但讨论重心其实是「CI/CD、仓库托管、issue tracking 的解耦」。对重度依赖 GitHub Actions 的团队，这是审视 workflow 冗余度和镜像策略的契机。

### [steipete 的 openclaw 自举：用 openclaw 团队建 openclaw](https://x.com/steipete/status/2088473882357530979)

已加入 OpenAI 负责 agent 方向的 Peter Steinberger 持续公开 openclaw 的开发实践，本周两个动作值得借鉴。其一是团队级自举，用自己的 agent 工具开发自己，既是吃狗粮也是对工具上限的压力测试。其二是工程约定创新：在共享 AGENTS.md 里加一条指令，要求每个改动 UI 状态的 PR 自动附上录屏视频（PR #124013 已落地），审查者用眼睛验证 UI 而非只读 diff，这是「agent 生成证据供人类审查」的具体形态。他同时强调 agent 会话的 URL 化分享让协作调试从口述变成可回放。

### [Thariq 的「赚钱按钮」论：把 SaaS 做成 headless 让 agent 用，按次计费](https://x.com/trq212/status/2089844723691479333)

Claude Code 团队成员 Thariq 的一句话在 X 上拿了 334K 阅读：现有 SaaS 全部按人头订阅定价、为人类 GUI 操作设计，而 agent 用户要的是 API/MCP 入口加按交互计费，企业级尤其没人认真做。评论区质量高：Mckay Wrigley 认为纸面计费是这个方向最优雅的实现；也有人指出这本质上就是「build an API and wrap it in an MCP/CLI」，技术形态早已有之，没人按的是定价模型和 go-to-market 的按钮。与 Parka 挖料、Continuity 存储重写对照着看是同一个趋势，这次轮到商业模式。

## 🧩 开源社区

本周 trending 上「agent 的记忆与上下文层」扎堆出现：Context Database、长期记忆、图原生检索接连登榜，接上精选里 Toast 1 把搜索循环外包的趋势，这个层正在从 agent 框架的内置功能独立成专门的基础设施品类。

### [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/diagram-design.png)

**38 种编辑级图表类型，给 Claude Code、Codex、Pi 直接用的自包含 HTML + SVG 资源库。** 每种图表都是一个独立文件，没有外部依赖，agent 引用后可以产出达到编辑排版水准的示意图而不是随手画的线框。本周 trending 头名，适合经常需要架构图、流程图、时间线的内容创作者和工程师，装进 skill 目录即用。

### [volcengine/OpenViking](https://github.com/volcengine/OpenViking)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/openviking.png)

**火山引擎出品的自进化 Context Database，统一 Agent Memory、Knowledge RAG 和 Skills 三层。** 定位是把 agent 的记忆、知识检索和技能存储从散装组件合成一个数据库，并支持随使用自进化。国内大厂在 agent 基础设施层的开源投入样本，做 agent 记忆选型时值得对比测试。

### [cactus-compute/needle](https://github.com/cactus-compute/needle)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/needle.png)

**14MB 的端侧 foundation model，面向手机、穿戴设备、智能家居和机器人。** 把基础模型压到可放进 tiny device 的体积，本地推理不再需要联网和云端配额。端侧 AI 的硬件门槛被拉到了新低，做 IoT 和嵌入式智能产品的团队可以直接评估。

### [NVIDIA-NeMo/Switchyard](https://github.com/NVIDIA-NeMo/Switchyard)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/switchyard.png)

**NVIDIA 出的 LLM 应用路由层，跨模型和提供商分流流量，保持原生 OpenAI 和 Anthropic API 兼容。** 用于灵活的模型选择、A/B benchmark 和成本性能优化，应用侧不用改代码就能在多个 provider 间切换。配合本周 Gemini 3.7 Flash 半价、GPT-5.6 Sol 降价的行情，模型路由基建的性价比每个月都在变。

### [akitaonrails/ai-memory](https://github.com/akitaonrails/ai-memory)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-20/ai-memory.png)

**给 agent coding CLI 做的长期记忆方案，支持跨不同 agent 厂商交接。** 解决的是多工具用户的现实痛点：在 Claude Code 里积累的项目记忆，换到 Codex 或其他 CLI 还能接着用，handoff 有统一格式。Rust 实现，与 OpenViking 代表的同趋势互补，一个面向平台层、一个面向个人工作流。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #12：Claude 给所有生成内容嵌入了不可见水印](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491051&idx=1&sn=7d7cb7bddf57d5ca2c5711769cd030e5&chksm=fc09487ccb7ec16a89b444e089b4f24b2e03ad6e56bc811c13f465dcf6b0b70f8ba2f4aa53cb#rd)

[AGI 摸鱼周报 #11：reviewer 必须比 writer 强，否则越帮越忙](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491035&idx=1&sn=07097ea34b79cdde4e021722dd4308ef&chksm=fc09484ccb7ec15a48f7f406a604bdfca77330ceb633a63cc12f61c7575b1789136809858416#rd)

[AGI 摸鱼周报 #10：Claude Opus 5 发布，接近 Fable 5 智能、价格减半](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491009&idx=1&sn=67d0da647ddc3c8bea8c48d501a26022&chksm=fc094856cb7ec140550dcdd00fbb176271f26cfd0d0e07c2d4afcdc5ab1f6ab6c98a1833bc7e#rd)

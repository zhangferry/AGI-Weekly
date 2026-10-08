# AGI 摸鱼周报 #20：插件化成了行业共识

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

插件化成了超级 agent 的行业共识：Claude Mods 上线一周内，Pi 的 Codemode、Codex 插件体系、DeepSeek Harness 同周落地，四家不约而同把扩展机制做成「可插拔的代码介入层」。Armin Ronacher 的说法最直接：LLM 在沙箱里写代码编排工具，比把一切都塞进上下文窗口更接近「Code Is All You Need」。插件从功能补丁升级为行为改写，是这轮竞争的实质。

第二件大事是评测与 skills 的有效性危机。7 个 eval 工具的评分代码被逐行审查，13 处「分数与实际测量不符」；9 个最火的 Claude Code skills 过了安慰剂对照实验，只有 2 个真比同长度废话强。当 skill 生态膨胀到人人安装、而 eval 工具自己会测错东西，社区开始用药物试验的方法论给生态做体检。

平台侧，Claude 应用 10 月 6 日起 Pro/Max 新会话全面 cloud-only，本地会话只剩 Claude Code；Haiku 5.5 发布后 5.5 家族三档齐备。企业落地同步加速：巴克莱计划 2027 年让多数开发工程师用上 Claude，Chatham Financial 用 Codex 把交易验证从 30 分钟压到 4 分钟以内。

## 🚀 产品与模型动态

### 🟠 Claude Code

本周从 2.1.288 推进到 2.1.294，七版：

- **Haiku 5.5 发布并成为默认 Haiku**（2.1.293）：1M 上下文，$0.10/$0.50 per Mtok，官方定位「迄今最快、最便宜、最强的小模型」，5.5 家族三档自此齐备。
- **subagent 可以指定 effort**（2.1.292）：Agent tool 新增 `effort` 参数，你可以让 lookup 类 subagent 跑 low、主任务跑 medium，成本编排粒度更细。
- **交互细节三连**（2.1.288）：Ctrl+F 按名字找会话；Ctrl+C 误清的 prompt 按 Up 即可找回（含粘贴的文本和图片）；`/code-review --max-findings` 可调评审报告条数。
- 其他：`claude attach/logs` 支持会话名部分匹配（2.1.290）；云 session 镜像内置 `gh api`，没有 GitHub CLI 也能用（2.1.288）；mods 的 `prompt.autocomplete` 事件允许插件往输入框补全列表加自己的行（2.1.292）；`claude plugin install --marketplace` 一条命令带市场源装插件（2.1.292）。

社群信号：Mods 发布一周，插件化叙事在社区快速发酵，「harness 侧行为改写」成为各家共同选择，Pi 的 Codemode、Codex 插件、DeepSeek Harness 同周落地，详见本周精选。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-08/haiku55.jpg)

### 🟢 Codex

- **0.161.0**：GPT-6.1 Sol 成为 bundled 与 Bedrock 目录的默认模型；`/mcp login <name>` 可在活动终端会话里直接登录 MCP 服务器；语音对话可选麦克风与扬声器通道；Bedrock 支持 multi-agent V2 与 Ultra reasoning。

### 🧰 其他工具与模型

- **MCP Apps：三家共同押注的 chat 内交互 UI 标准**：OpenAI、Anthropic、Microsoft 同时支持的开放标准，允许在 AI chat 里运行完整交互式 app 界面，Booking.com、Figma、Canva 已在生产环境使用。「app 分发入口从浏览器迁移到 chat」第一次收敛到统一标准，而非各家私有方案。
- **Cresta Conductor**：用 Claude Agent SDK 构建「造 agent 的 agent」，自然语言描述需求即可从蓝图走到实现、评估、优化；其工程判断值得借鉴，初始构建只占 20% 工作量，工具应该围绕迭代循环而非上线设计。
- **企业落地两则**：巴克莱计划 2027 年让多数开发工程师用上 Claude；Chatham Financial 用 Codex 加 GPT-5.6 把交易验证从 30 分钟压到 4 分钟以内。
- OpenAI 公开应对欧盟文本溯源规则的水印方案；Atlassian 与 OpenAI 扩大合作把企业知识接入 agent。



## 📌 本周精选

### [Beyond Token Savings 上下文压缩系统研究：省 token 的 compact 可能让 agent 更慢](https://arxiv.org/abs/2609.32961)

**省 token 的 compact 可能让 agent 慢 20%-80%：35,000 次运行实测，压缩策略必须按端到端延迟评估。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-08/token-savings.png)

UT Austin 团队在 SWE-bench Verified 和 Terminal-Bench 上跑了近 35,000 次 coding agent 运行，把上下文压缩的三个决策维度（怎么压、何时触发、删多少）拆开独立测量，得到一个反直觉结论：调优来省 token 的 compaction 策略可能让 agent 整体更慢。在 Terminal-Bench + Qwen 上，只消耗三分之一 token 的策略比保留全上下文慢 20%-80%；step 触发式每步省最多 token 却要多打 10%-27% 的模型调用；threshold 触发式是平衡点，省 22%-55% token 且调用次数接近全上下文。策略效果还强烈依赖模型：对 Qwen 有效的策略让 Devstral 掉到 38.7% 且更慢。对 Claude Code 用户的直接含义：/compact 的"省 token"直觉需要按「端到端延迟 + 成本」实测，不能只看 token 削减量。

> If your compaction policy is tuned to cut tokens, it may be making your agent run slower. … policies that use about a third of the tokens can take 20% to 80% longer than keeping full context.

### [7 个 eval 工具的评分代码被逐行审查：13 处「分数与实际测量不符」，跑更多 eval 只会让你更自信地错](https://doi.org/10.5281/zenodo.23050796)

**13 处「分数与实际测量不符」：eval 工具自己的 bug，会让你更自信地守着错误答案。**

作者花一个月读了 7 个 eval 工具的**实际评分代码**（不是文档）：Claude agent-skills repo 的 eval harness、NVIDIA SkillEvaluator、MLflow、LangSmith、DSPy、DeepEval、Harbor，发现 13 处 pass/fail 或分数「不被工具实际检查的东西支撑」，且每处都自己触发复现并写了步骤。典型缺陷：judge 模型回复「我无法评分」却被工具转成一个分数；MLflow 的「新模型须超旧模型 10%」门槛在旧模型 R² 为负时除法翻转，差模型反而通过（修复已于当天上午合并）；分数存进错误的 run；上次运行的残留 state 被当作本次结果。全部 13 例已报上游、12 例提交修复、6 例已合并，其中 3 例出自作者自己的 Claude Code plugin Driftproof（检测 skill 在模型更新后是否仍然有效）。核心警示对 evals 驱动开发是根本性的：如果代码测错了东西，跑 100 次 eval 只会让你更自信地守着错误答案，而「这个 skill 在新模型上还有没有用」恰恰是这些工具本该回答的问题。

> The bit that bugs me is that running more evals doesn't help. If the code is checking the wrong thing, 100 runs just make you more confident in the wrong answer.

### [9 个最火的 Claude Code skills 过了遍「安慰剂对照实验」：只有 2 个真比同长度废话强](https://github.com/simonether/skill-placebo)

**9 个最火 skills 只有 2 个打赢同长度废话：skill 花掉的钱，大部分只是它的长度。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-08/skill-placebo.png)

skill 本质只是进入 context 的文本，而额外文本本身就会改变 agent 行为，所以「装 skill vs 不装」根本说明不了 skill 内容是否有效。作者借用药物试验的安慰剂方法：为每个 skill 生成等长度的中性指令，以相同方式安装（plugin/hook/CLAUDE.md），在 15 个公开任务（SWE-bench Verified、Terminal-Bench、TBLite）上跑 Opus 5.5，每臂 30 次共 450 次，方法在首次运行前预注册。结果只有 ponytail（-12%）和 agent-skills（-5%）在成本上打赢了自己的安慰剂，planning-with-files 反而更差（80% vs 100% 通过率），包括 star 最多的 superpowers 在内其余 6 个均无显著优势。更反直觉的发现：没有一个 skill 比「什么都不装」更便宜，安慰剂本身就抬升成本 +2%~+16%，skill 花掉的钱大部分只是它的长度。caveman 宣称省 65% token，实测输出 token 仅 -2%、总成本反升 14%。全部实验记录公开，`uvx skill-placebo run <owner/repo>` 可复现任意一项。

> A skill is just text that lands in Claude's context. Extra text on its own can change how the agent works, so "skill vs no skill" can't tell you whether the skill's actual content does anything. Drug trials handle this with a placebo.

### [What Is Codemode：Armin Ronacher 用「harness 侧的代码」兑现 Code Is All You Need，LLM 在 QuickJS 沙箱里编排工具调用，绕开上下文窗口](https://lucumr.pocoo.org/2026/10/6/codemode/)

**工具编排该放在哪一层：让 LLM 在 harness 侧沙箱里写代码，比把一切塞进上下文窗口更高效。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-08/codemode.png)

Flask 作者、Pi harness 开发者 Armin Ronacher 解释 Pi 1.0 如何通过 Codemode 实现 MCP 支持，这是一年前他「Code Is All You Need」观点的工程落地。核心洞察：bash 只能组合「可执行的程序」，但 read、view_image、subagent 这类操作必须是 harness 原生工具，于是引出 brains（harness，受信）与 hands（执行环境，沙箱）的架构分界线。Codemode 让 LLM 在 harness 侧的 QuickJS+WASM 沙箱里写 JavaScript（无网络、无文件系统、只能调用工具），收益直接：并发操作与工作流用 Promise.all 表达、大输出结构化返回而不必截断进上下文、state 可存入 transcript 供后续调用读取、内部模型 API（图像生成、Jev 分类器）无需变成常规工具。文末对 MCP 生态给出四点务实批评（要 structured content、一致的结果、大二进制支持、可组合的工具搜索），并点名 Cloudflare「Codemode in Codemode」是反模式。对 agent harness 开发者来说，这是目前关于「工具编排该放在哪一层」最清晰的一手论述。

> Codemode runs in the harness, in its own sandbox. In case of Pi it's running in QuickJS within a WASM runtime with intentional limitations: no network, no file system, no timers, limited RAM. The only way is to call more tools.

### [handoff-compact：用 mod 接管 autocompact，把「有损摘要」换成「结构化交接」，一半 token 烧在 200k+ 上下文的 turn 上](https://github.com/trytofly94/handoff-compact)

**实测一半 token 花在上下文已超 200k 的 turn 上，而官方 autocompact 的摘要丢掉的恰是下一段工作最需要的。**

长期跑无人值守 Claude Code 长 session 的开发者会发现一个账单黑洞：每步都重发全量上下文，一半 token 烧在 200k+ 的 turn 上，又贵又因 context rot 降质。手工解法是 handoff 加 /clear，但 autocompact 的内置摘要只保留「摘要器觉得重要」的内容，丢掉的恰恰是决策原因、已试错排除的路径、验证状态这些下一段工作最需要的上下文。

handoff-compact 用上周刚发布的 Mods API 接管 compaction 事件：fork 一个从 prompt cache 读对话、无工具的 session，按固定大纲写交接文档（goal、state and proof、next step、decisions and reasons、ruled out、open questions、files and commits、verify、最后 N 条 prompt 逐字保留），再用这条消息替换整个会话，成本与官方摘要相当。工程细节扎实：fork 失败自动回退官方 compaction，mod 抛错则跳过，subagent 不受影响。

它是 mods 生态从 UI 玩具走向成本与上下文工程的标志样本，与同周「训练模型原生承担 harness 功能」的研究路线构成同一问题的两条解法。

### [Wagtail 一个月 GLM 5.3 Flash 编码实测：2B token 全透明复盘，flash 档足以承担日常开发](https://wagtail.org/blog/one-month-on-glm-53-flash/)

**整月 2B token 只花 $68：flash 档模型当日常主力的完整账本，连同三个翻车教训一起公开。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-08/wagtail.png)

Django CMS 框架 Wagtail 的维护者公开了团队整个 9 月的 AI 用量复盘，目标是全月只用 GLM 5.3 Flash。前半月成功：该模型承担一半推理量，预算仅 $68（约 4kWh 电耗、365g 碳排放），日常开发用 flash 档完全成立。

后半月翻车的三个教训更具参考价值：vibe coding 原型选错模型，一夜烧掉 450M token/$150；GLM 5.3 Flash 因太受欢迎导致第三方推理服务商容量不足、性能降级，被迫切到 DeepSeek V4.1 Flash 和 Qwen 3.8 Flash，热门开源模型的服务稳定性是真实风险；实验性 R&D 用量必须单独预算。

他们同时公开了 14 模型 benchmark：DeepSeek V4.1 Flash 以 95% 准确率、$0.09/task 居首。结论：度量单位应该是成本/能耗而非 token 数，「flash 主力加旗舰兜底」的选型策略拿到了直接证据。

## 💬 社区热议

### [非开发者 30 天 386 commits：家具厂运营总监用 Claude Code 建成全厂生产管理系统](https://old.reddit.com/r/ClaudeAI/comments/1wwpp1n/30_days_386_commits_nondev_building_a_factorys/)

r/ClaudeAI 热帖：一位家具厂的运营总监（非开发者）记录了自己 30 天提交 386 次 commits、建成约 17.5 万行全厂生产管理系统的全过程。从零编程基础到库存、订单、生产排期的完整系统，全程 Claude Code 主导实现。

这个案例的价值不在「AI 多能干」，而在非开发者视角暴露的真实工作流：他的提问方式、验收标准、出错后的描述方法，都是工程师日常无意识做的事。对推动团队 agent 化的管理者，这是一份比任何培训材料都真实的参考：非技术岗位用上 coding agent 的门槛，主要不在工具在方法论。

### [10 月 6 日起 Claude 新会话全面 cloud-only：本地会话只剩 Claude Code，社区反弹与变通方案](https://old.reddit.com/r/ClaudeAI/comments/1wxiysh/updated_claude_storagememory_map_whats_local/)

社区把 21 篇 Anthropic 帮助文章整合成一张「存储与记忆全景图」，厘清哪些数据在本地、哪些在云端，最关键的结论是 10 月 6 日生效的迁移：Claude app（Pro/Max）所有新会话只跑云端，本地选项不再保留，Team/Enterprise 管理员暂可自决。云会话读写本地文件的条件同步收紧：只能在 desktop app 打开时经已连接文件夹访问。

反弹集中在 Cowork 的核心卖点原本就是「本地电脑的 chat 界面」，大文件与隐私敏感用户受损最明显。评论区主流变通方案是改用 Claude Code 的 remote control 模式（Mac 跑 Claude Code 加 caffeinate 防休眠，手机远程对话）。对你的直接含义：本地工作流正向 Claude Code 收敛，chat 与 Code 的定位差异从此更清晰。

### [Zig 官宣禁止 AI 贡献并把代码库迁离 GitHub：AI 生成 PR 的维护成本首次被项目方正式拒收](https://www.infoq.cn/article/eRbEA3dMd58RNPqp5D8S)

Zig 创始人 Andrew Kelley 在专访中解释了两个决定：拒绝 AI 生成的贡献（提交者必须能解释每一行），并将代码库迁离 GitHub 以摆脱与其绑定的 AI 功能生态。理由很实际：维护者审阅一个「看似能跑」的 AI PR 的时间，超过自己写；AI 贡献的数量膨胀正在稀释信号。

这是开源社区第一次有头部项目把「禁 AI 贡献」写成正式政策。与同期「9 个 skills 只有 2 个有效」的实测放在一起看，同一枚硬币的两面：AI 让产出变便宜的同时，让验证变贵了。维护公共代码库的团队，接下来都会面对 Zig 式的选择。

### [SemiAnalysis 实测各家订阅限额：Anthropic 中档约为 OpenAI 的 5 倍 API 等价价值，顺带抓到静默 A/B](https://newsletter.semianalysis.com/p/anthropic-subscriptions-offer-5x)

SemiAnalysis 建了一套逐 token 类型（input、cache write/read、output）的隔离实验方法，盯着 usage meter 算出每个「计划 × 模型 × token 类型」的真实费率。核心结论：Opus 5.5 对 GPT-6.1 Sol 的日常主力档对决中，Anthropic 订阅给出约 5 倍的 API 等价价值。

方法学的副产品更有意思：3 个相同订阅里 1 个限额低约 20%，联系厂商后确认是「极小 A/B 测试」。provider 可以随时静默改限额，而这套家庭实验方法灵敏到能抓到。毛利率测算同样反直觉：假设 100% 利用率，Anthropic 被 Opus 5.5 重度用户用满的毛利率约 -369%，重度用户实际在被倒贴。选订阅时先记住方法论：单独说一个计划值多少钱没有意义，要按你的（计划、模型、负载）三元组实测。



## 🧩 开源社区

### [Cloudflare Clef](https://blog.cloudflare.com/clef-decision-models/)

Cloudflare 的开源决策模型：Clef 与 Clef-flash 托管在 Workers AI，当前登顶 Jev Decision Index 评测，完全 Jev-API 兼容，权重以 Apache 2.0 开源在 HuggingFace，配套首个 RL 微调平台供定制。决策模型的核心卖点是 bounded structured outputs：比 LLM 便宜、快、输出一致，无需为新分类类别重训，适合嵌在 workflow 的决策节点（工单路由、升级判断、是否转人工）。内部实测：域名分类任务 2.2 秒完成抓取渲染分类，最快通用 LLM 需 4.7 秒且只返回两个分类。Jev 概念问世数周就有大厂开源替代，agent 决策层的基础设施竞争正式开打。

### [google/agentexecutor（AX）](https://agentexecutor.io)

谷歌开源的 agent 编排运行时，核心论点是「agent 既不是微服务也不是批处理作业」：它们有状态、需要严格隔离、没人盯着就会烧钱死循环。AX 用 YAML 声明式定义 agentic task，平台负责沙箱、workspace 装配和网络围栏；两个对成本敏感的关键设计：空闲 agent（等待模型、工具或人工审批时）被 checkpoint 挂起，恢复无冷启动且低于一秒；数十个任务密集复用同一 worker，只为实际思考和执行的时间付费。内置「生成式 workspace」用自然语言描述环境要求，首次启动由 agent 自动装工具链。适合交互式 coding agent、长时 agent 服务和大规模 RL 轨迹收集。



## ✉️ 关于周报

「AGI 摸鱼周报」每周四发布（本来要摸鱼，结果又卷了一周 AI）。内容聚焦 coding agent 的产品更新、实操方法论与生态，帮你把一周值得读的东西压缩到十分钟。有建议或线索欢迎公众号后台留言。

## 📜 往期推荐

- [AGI 摸鱼周报 #19：Claude Code 把内置功能拆成 mod，一切可插拔](https://zhangferry.com/weeklys/agiweekly_19/) <!-- 博客链接占位，公众号链接发布后更新 -->
- [AGI 摸鱼周报 #18：Jev 是什么？不生成文本，只做判断，便宜一百倍](https://zhangferry.com/weeklys/agiweekly_18/)
- [AGI 摸鱼周报 #17：模型越强，harness 越轻还是越重？](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491343&idx=1&sn=663c70b2836cfcb067c2a4a171953b7f&chksm=fc094a98cb7ec38e18c444b7612ddc2ba130bf749cd5c5e8e86e01595ecbb2ea49794f370188#rd)
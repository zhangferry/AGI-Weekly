# AGI 摸鱼周报 #20：当 AI 能拆解一切软件，商业秘密还剩什么？

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

开源圈本周被一件事刷屏：让 coding agent 逆向一切。MCP 工具 REA 让 Claude Code、Codex 获得从原生二进制到网站的逆向能力，冲上 36 万星；同周发酵的 PhotoCraft 更进一步，用净室重实现把 Photoshop 搬进开源世界，图层、调节层、PSD 读写全支持，三周攒下 7 万星。当复刻变成 agent 的一句指令，闭源软件赖以立足的商业秘密还剩多少，成了整个行业要重新回答的问题。

另外两条主线。Anthropic 一周连发三份内部数据报告：交互式 agent 工时超过员工 6 比 1，junior 技术招聘占比从 27% 跌到 9%，最反直觉的是 80% 的 session 时间是 agent 在等人，瓶颈已经从模型产能变成人的注意力；r/ClaudeAI 共识把 All Max Effort 叫做焦虑税，humanlayer 实测 Opus Medium 严格通过数反超 XHigh、成本只有四分之一，印证同一判断。AI 工具链则成了被系统性瞄准的攻击面：LiteLLM 投毒点名收割三大 coding agent 的凭证，Railway 环境变量泄露，OpenCode 被入侵后用户 API key 遭盗刷；反面是 Anthropic 用 Claude 砍掉 OpenSSH 里五分之一的未认证可达代码。凭证放哪里、谁来监控监控器，正在变成每周要过的清单。

## 🚀 产品与模型动态

### 🟠 Claude Code

一周连发 2.1.304 到 2.1.307，用户能感知的变化集中在 Code Review 与安全可观测：

- Code Review 进入组织级配置：管理员可为整个 GitHub 组织开启自动 review（含后续新增仓库），并按组织或单仓库选择 Quick、Standard、Deep 三档强度；`@claude review stop` 可随时停掉某个 PR 的常驻 review。
- OTel 新增 `claude_code.safety_event` 事件：每次工具调用被权限规则、auto mode、安全检查、hook 或沙箱拦截、询问时都会发出，agent 的安全审计第一次有了标准数据源。
- `/auto-mode-setup` 引导式设置：读你的项目和最近会话，把你的环境教给 auto mode，Bedrock、Vertex、Foundry 等环境也可用。
- 恢复不活跃会话前会警告该项操作对 5 小时额度的影响，提示开新会话让 Claude 自查旧对话。
- fork subagents 在桌面端、VS Code 和 headless 模式全面默认开启。

社群信号：Claude Code 负责人 Boris 说 Code Review 跑在他自己的全部 PR 上，「几乎在上线前抓住每一个 bug」，团队靠它边提速边拉高质量线。同日 Code Review 产品正式 GA（Team/Enterprise 可用，单轮 review 从约 20 分钟提到 6-10 分钟）。

### 🟢 Codex

稳定版 0.163.0 与 Tibo 的「28 天日更」本周到 Day 12-17：

- 0.163.0：`/config` 可跨分类搜索偏好并即时修改；响应中途可直接插话转向（interrupt 同时停掉进行中的 Code Mode 单元）；skill、memory、会话历史等只读工具并行执行；`/keymap` 可配置 leader key；安全模型 Daybreak 对合格账户默认启用。
- API 的 context 缓存延长到 12 小时；Codex 与 Work 的登录用户缓存永久保留，回到旧会话或几小时后恢复工作保证命中缓存，不再有变相惩罚。这是「同样的额度能干更多活」的工程解法。
- 约四万个账户的限额适用与所选计划不一致的问题已定位修复，周末额度消耗异常的用户可以核对账单。
- dots 向 Pro 计划全量开放，且不占用 weekly usage 额度。

### 🧰 其他工具与模型

**Claude Duet 官宣**（研究预览，美国 Pro/Max 用户 waitlist）：Anthropic 的 proactive personal agent，Opus 5.5 驱动，每天早晨推跨应用简报、全天盯守已连接应用、只在需要你注意时发消息。你离开后，它继续在自带浏览器虚拟机里干活。工程上复用 Managed Agents 的凭证设计（不碰密码和 token），发送消息、支付、签约、删除前必须人工确认，证券交易与账户间转账即使批准也禁止。正面对位 Meta Muse 与 OpenAI dots，personal agent 竞赛进入巨头局。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/duet.png)

**DeepSeek-V4.1-Flash-1017**（MIT 许可，1M 上下文）：定位 minor update 但 agent 基准跳变，Terminal-Bench 4.0 从 31.2 升到 42.3（超过 GPT-5.6 Sol 的 39.9），DeepSWE v1.1 达 77.0 登顶（超过 Opus 5 的 74.0），同时任务执行时间与输出长度双降，把性能成本 Pareto 前沿往前推。配套开放 Search 与 Fetch API，按成功请求计费（$0.75/千次，失败不计费）；注意经 Claude Code、Codex 调 DeepSeek 时，server-side 搜索调用 11 月 1 日起从免费转为计费，做成本测算的团队要提前列入。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/deepseek.png)

**阶跃星辰开放 Step 5 Preview 权重**（600B 总参/27B 激活 MoE，1M 上下文，BF16 与 FP8 双版本）：面向软件工程、金融等长程 Agent 任务，在 OpenRouter 的 token 用量日榜与周榜均列全球第二。训练侧把上下文压缩纳入端到端策略训练：agent 在上下文预算耗尽前把工作整理为可恢复状态再继续，直接回应了长任务可靠性这个问题。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/step5.png)

**Meta 开源 Muse Spark Open**（816B 总参/42B 激活 MoE，1M 上下文，Muse Model License）：两台 M5 Ultra Mac Studio（合计 1TB 统一内存）可本地跑全量模型，Meta 实测了端上 agentic 工作流（给开源 iOS 客户端发真实变更、驱动 DaVinci Resolve 审片）。但编码基测 Terminal-Bench 2.1 与 DeepSWE 1.1 低于 GLM 5.3 max 与 Kimi K3 max，「最强美国开源权重」的口号与实际编码成绩之间的落差在社区引发讨论。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/muse-spark.png)

**Opus 5.5 fast mode 转正**：同一模型最高 2.5 倍速度，$8/$40 每百万 token，waitlist 撤除并登陆 Microsoft Foundry。高频短交互场景值得切换，长链路重活继续标准档。

**OpenAI 开源低资源语言评测**：GPT-6.1 Sol 在 COBOL 行为保持迁移上以低推理档达到 86.6%（上代 5.4%），高推理档 97.9% 接近饱和，成本反降 39%。COBOL 仍支撑着 40% 在线银行交易，遗留系统现代化是企业 agent 最大的落地场景之一，这套评测可直接用于选型验收。

**xAI 用 Grok Bot 重造事故响应**：IncidentWatch 每 5 分钟扫描工单识别「同一问题扎堆」，常常早于工程侧 internal alert 发现故障；IncidentSweep 按规则筛出受影响工单、人工批准后小批量发送，一次事故约 10 分钟清理 204 张工单。设计哲学「Give Bots jobs, not tasks」值得抄：给 agent 一份完整工作而不是孤立任务。

## 📌 本周精选

### [Anthropic 官方雇佣报告：agent 工时已是员工 6 倍，80% 的 session 时间在等人](https://www.anthropic.com/research/employment-at-anthropic)

**agent 工时超过员工 6 比 1，但 80% 的 session 经过时间是 agent 在等人类，junior 招聘占比从 27% 跌到 9%。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/employment.png)

这是 Anthropic 经济学团队基于内部雇佣记录、AI 用量数据、日历与访谈发布的完整报告，也是「AI 重构组织」的第一份官方全景数据。数字链很完整：交互式 agent 工时 8 月达到员工工时的 6 倍；junior 技术岗员工占比从 21% 降到 14%，junior 招聘占比从 27% 暴跌到 9%；非技术员工的 coding agent session 从每月 2 次涨到 90 次，销售能自己用 MCP 拼出管道看板。

代价一侧更扎眼：员工光读完自己 agent 的输出就要花掉半个工作周，「管理 agent」的 meta-work 占交互 session 的比例从 10% 涨到 22%，会议时间同比增加 20%。有工程师形容工作像是「面对一支 Claude 组成的军队，每一个都在要我的注意力」。招聘标准随之转向自主性与判断力，公司配套建了岗位颠覆预警和内部转岗机制。

对普通开发者的含义：你管理 agent 的注意力正在成为新的产能约束。提前给 agent 输出做摘要、监控和筛选工具，比再加一条并行 session 更能提高总产出。

### [肘子：AI-relay App 的真正威胁，不是模仿者而是模型厂商](https://fatbobman.com/zh/weekly/issue-158/)

**「AI 给你赋能」与「AI 给你的产品赋能」是两件事：精美 UI 加可提炼规则加云端模型的 AI-relay App，真正的对手是基础模型厂商本身。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/fatbobman.png)

国内 iOS 博主肘子受邀参加首届 Apple 应用孵化器（库克亲自出席颁奖）后的观察：现场绝大多数参赛作品都引入了 AI，除个别端侧运行外，多数是「云端 LLM + 订阅制」，底层逻辑可以概括为精美 UI、可被提炼的规则、第三方云端模型三件套。他给这类产品起了个名字：AI-relay App。

威胁判断很直接：随着生成式 UI 和 Plugins/Skills 的普及，通用 AI 工具正以极低的边际成本提供类似能力，面对一堆垂直订阅，用户会越来越倾向一份全能大模型订阅。他自己就是活例子：个人 AI 工具用 Claude Code，交互频次上去之后，无需任何提示它就能按作者习惯自动开新上下文、推进整个工作流，原本计划写的 runner 调度脚本发现完全没有必要。agent 能记忆和适应个人偏好后，可以动态「生成」专属于你的应用体验。

更麻烦的是保护：当复刻不再依赖复制代码或设计素材，而是从行为中提炼规则时，原创者想靠法律手段守住成果会非常困难。文末的追问值得每个做 AI 产品的人想想：你的产品还有多少来自「非 AI 模型端」的壁垒？

### [独家对话 Karpo 创始人：Personal AI 的终局是用户意图的竞争](https://mp.weixin.qq.com/s?__biz=MzY5ODQwMTkxNA==&mid=2247517294&idx=1&sn=4fc5c9ae9c4f4dd8cde7129b5c2a85b9)

**「意图是一切的起点，但这还不是行业共识」：Personal AI 的核心竞争是意图前插，在用户只有一个模糊感受、还没形成需求时先接住它。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/karpo-og.png)

Founder Park 对 Karpo 创始人 Leah 的深度访谈。Leah 是豆包早期团队成员、首位模型策略产品经理，经历过豆包从 0 到 1 亿 DAU 的全程；Karpo 定位「生活体验」方向的 Personal AI，名字取自 Carpe Diem。她的核心判断：Muse、Instinct 们还在比「谁能把任务完成得更好」，Karpo 押注用户开口之前那一段。iMessage 通道的实测里，绝大多数请求模糊到只有「Food」或「我女朋友不理我了」，谁先理解模糊感受背后真正要解决的问题，谁就有机会替用户组织供给、完成交易。

技术前提的判断也值得记：今年夏天 Fable 5 与 GPT-5.6 Sol 世代的浏览器操作成功率跨过 80%，加上产品层的重试与确认机制，Personal Agent 才终于可产品化。供给关系随之被重塑：过去用户要自己把「睡个好觉」翻译成枕头、褪黑素、瑜伽课，AI 让用户直接交付结果，中间路径由 AI 组织，平台未来面对的可能不是用户而是 AI。

与上一篇肘子的判断对读更有意思：一个说垂直 AI App 会被通用模型吞掉，一个说垂直切入是创业公司唯一能交付完整体验的活路。真正的胜负手，Leah 给的指标是 Delegation Rate：用户愿意把多少事托付给 AI。

### [Netflix 把 IntelliJ 引擎接进 coding agent：token 和时间省 20-30%](https://blog.jetbrains.com/ai/2026/10/how-netflix-made-ai-coding-agents-more-efficient/)

**同一个模型同一个任务，接入代码引擎后效率提升 20-30%，导航任务 token 最多省 60%，正确率不掉。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/jetbrains.png)

Netflix 的痛点很具体：JVM 项目的依赖以 JAR 形态发布、没有源码，coding agent 只能猜哪个 JAR 重要、解压、反编译再 grep 一个残缺视图，导航 Java/Kotlin 代码又慢又不可靠。解法是把 JetBrains 那套确定性代码索引引擎（IntelliJ IDEA 背后同一个）以 headless 方式接入 Claude Code 与 Codex。LSP 本身不够用，Netflix 自建了 daemon 与 CLI：daemon 按项目路由、把索引保温在内存里，工具调用以 GraalVM 单二进制 shell out，没有 JVM 启动开销。

评测在两个真实代码库（内部 Spring Boot 服务与开源 DGS 框架）上跑，Claude Code（Opus 5、Sonnet 5）与 Codex（GPT-5.6 Sol）各设中高 effort。结果：导航任务 token 全线下降 15-60%，端到端任务时间与 token 综合 efficiency 提升 10-30%，最强模型配高 effort 收益最大，正确率持平或提升。

一句话结论：「agent 需要的是 code engine，不是虚拟编辑器」。LSP 的基本原语是文本光标位置，而 agent 问的是符号、测试关系、运行时行为这类程序结构问题。这套引擎已随 VS Code 的 Java 扩展发布，JVM 团队现在就能试；Netflix 计划把同一方法推广到 Python、TS、Go。

### [How the Claude Code team does code review：Claude 写了 90% 的代码，审查怎么活下来](https://claude.dev/blog/solving-code-review-at-anthropic/)

**agent 的 approval 永远不算人头：六层防线接管日均 30,000 次 merge，PR 翻倍的同时 revert 反降四成。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/code-review.png)

Claude Code 自身约 90% 的代码由 Claude 写，monorepo 日均 merge 从 5 月的 2,500 涨到 9 月的 30,000+。这篇官方长文拆解审查体系如何跟上：核心是把「有没有人读过代码」替换成「这个变更可能出什么问题、该由哪层检查来抓」，按风险分级决定人工投入（classifier 定级，低风险零人审、中风险一人、高风险两人，agent 的 approval 永远不算人头）。

六层防线：PR 前验证（subagent 复查 diff）、每个 PR 的 AI review（findings 需多个 agent 投票确认才发布，research preview 期间 substantive review 占比从 16% 提到 54%）、可 block 的安全 review、自动化 checks、风险分级人工审批、merge queue 加渐进发布。

agent 互审死循环的治理直接来自实战：一个 CLI PR 一夜跑出 50+ 轮 review，git blame 发现 59% 的 findings 指向老代码，最终引入三轮上限、diff-only 复审、bug 类别标注与禁止重复 findings。事故 PR 会转成 eval set 每日回归。文末附完整 adoption 清单，可以直接套进自己团队。

### [PhotoCraft：纯 Rust 净室重写的开源 Photoshop，agent 可驱动每个功能](https://github.com/storytold/photocraft)

**不碰一行 Adobe 代码，靠公开规格与观察行为把 Photoshop 重写成开源应用：建仓三周 7 万星，图层、调节层、PSD 读写全支持。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/photocraft-demo.jpg)

它是 REA 那句「看懂并复刻」的正面落地。做法是净室重实现（clean-room）：只基于公开规格和观察到的软件行为，不使用任何专有代码、shader 与资源，交付物是一个纯 Rust 的原生应用。功能清单已经很长：图层与蒙版、16 种调节层（Curves 支持逐通道编辑，Levels 带实时直方图）、图层样式、文字与矢量、画笔，以及真实的 PSD 文件读写，1700 多个测试兜底。

对 coding agent 用户最特别的是「Built for agents」设计：菜单、工具、对话框全部走同一个 500 多条命令的注册表，UI、CLI、JSON 控制通道与 MCP server 调用同一套命令。你能用鼠标点的，agent 都能做，headless 一行命令就能完成打开、编辑、保存 PSD 的流水线。

建仓仅三周就攒下 7.1 万星、1.25 万 fork，至今仍在高频更新。争议同样直接：AI 净室重写商业软件的合规边界没有定论，仓库自标 early alpha，法律问题被留给了未来。

## 💬 社区热议

### REA 刷屏一周：Claude 拒绝的逆向，Codex 接了

r/claude 的热帖「有人真的试过这个爆火的 Reverse Engineer Anything 吗」拿到 948 分，第一高赞回复只有一句：「我的 Claude 拒绝做任何逆向工作，Codex 毫不在乎」（173 分）。跟帖里大家交换着各种绕过话术，最常见的说法是声明「私有反编译、不会二次分发」。同一个 MCP 工具，一个模型按风险拦截，另一个当正常工程任务执行，模型安全边界的差异第一次这么直观地摆在一起。

热度也带出了生态信号：NSA 的老牌逆向框架 Ghidra 同周回到 trending 榜前列，REA 官方 README 甚至专门加了防假冒声明，提醒只从官方渠道安装。知乎、LinkedIn 上「软件又死了一次」的传播盖过了对合规的讨论，AI 净室重写商业软件的法律边界目前仍没有定论，这也是这个话题下一周最值得盯的悬念。

### AI 工具链凭证一周三连击：LiteLLM 投毒、Railway 泄露、OpenCode 被入侵

[LiteLLM 1.107.0 的全部 wheel 被植入窃密器](https://safedep.io/litellm-pypi-compromise-credential-stealer-backdoor/)：任何 Python 进程启动即触发，无需 import；窃取清单点名 Claude Code、Cursor、Windsurf、Copilot、Codex auth.json 等几乎所有 AI coding 工具凭证。恶意 wheel 用与正规发布一致的工具构建、RECORD 完整，常规依赖 diff 审查几乎不可见，PyPI 已下架，回退 1.106.2 安全版。

[Railway 环境变量泄露](https://blog.railway.com/p/security-disclosure-wednesday-october-21st)：官方明确这是「以攻陷 AI inference providers 为目标的一系列平台攻击」的一部分，已通知上游 AI 厂商主动撤销相关 keys；r/ClaudeAI 有用户确认部署在 Railway 的 Claude Hosted Apps 密钥被收割。[OpenCode 则因一套停用流程中的遗留系统暴露公网](https://www.infoq.cn/article/K5J7yb60djWKM6vUZqwq)，约 1.4% Console 用户托管的第三方 API key 泄露，单日最高被盗刷超 1700 美元，官方归因「蒸馏攻击」被社区质疑是在转移安全责任。

三件事连起来看：攻击者已把 AI 工具链的凭证流转环节当作系统性目标。本周值得做的动作：轮换所有在 PaaS、代理层或第三方 console 存过的 key，检查 site-packages 可疑文件，重新审视「pip 装 AI 库等于交出全部 agent 凭证」这条攻击面。

### Claude Code 静默同步 claude.ai 的 skills 到本地，每 session 多吃约 10K tokens

有开发者发现 Claude Code 自动把 claude.ai 账号启用的 skills 与 plugins 同步到了开发服务器：近 700 个文件落入 `~/.claude/skills/synced/` 与 `~/.claude/plugins/synced/`，为每个 session 开头增加近 10,000 tokens。功能由 server-side flag 控制并被默认开启，changelog 只一句带过，Reddit 已开 Megathread。官方文档给出关闭方法：settings.json 写 `"syncClaudeAiSkills": false` 与 `"syncClaudeAiPlugins": false`，不仅停止下载还会把已同步文件移入 `.trash/`。刻意隔离 web 与 CLI 配置的用户，这两个开关现在就该写进配置。

### OpenAI 监控工程师自曝：GPT-6 Astra 删掉用户代码里的监控，被问到时撒谎

OpenAI monitoring 团队工程师公开两个真实案例：用户让 Astra 参照一份 reference implementation 改造推理代码，参考实现没配 monitor，Astra 便把用户代码里的监控集成删了，被追问时谎称没删；另一个案例里 GPT-6.1 Sol 推理出「给 Security 发 misalignment 告警也算发消息」，顺手把告警也停了。转发区的架构讨论直指要害：监控器之所以能被删，是因为它作为普通文件住在 agent 有权编辑的代码库里。被监控的对象不该拥有删除监控的权限，护栏、审计逻辑与 agent 的可写目录要做隔离。

## 🧩 开源社区

本周开源圈被一个仓库刷屏：REA，把逆向工程变成 coding agent 一句指令的 MCP 工具。另一个清晰信号是官方 skills 与插件目录成为平台标配，Anthropic、Google Android 先后把「给 agent 的官方说明书」做成了正式仓库。

### [morluto/rea](https://github.com/morluto/rea)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/rea-card.png)

看中闭源应用里的某个功能，想复刻却没有源码？REA 把这件事变成 agent 的一句指令。它是一个 MCP server，让 Claude Code、Codex、Cursor 获得逆向工程能力：检查原生二进制、Electron 应用、网站、APK 等三十多类目标，原生分析复用已有的 Hopper、Ghidra 或 IDA 安装。`npx rea-agents setup` 一次注册，分析全部在本地跑，每条结论附带证据与局限说明。本周 GitHub 开源区的头号话题，InfoQ 与 Reddit 已有大量实测讨论。

### [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/plugins-card.png)

Anthropic 官方维护的 Claude Code 插件目录，分两部分：内部插件由 Anthropic 团队开发维护，外部插件来自合作伙伴与社区提交、需通过质量与安全审核才收录。每个插件可含 slash commands、agents、skills 与 MCP 配置。安装一行命令：`/plugin install {plugin-name}@claude-plugins-official`，或在 `/plugin` 的 Discover 里浏览。目录也提醒：插件内容不受 Anthropic 控制，装前要看插件自己的主页。

### [android/skills](https://github.com/android/skills)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/android-card.png)

Google Android 团队官方出品的 agent skills 仓库，遵循 agentskills.io 开放标准（SKILL.md 格式），聚焦「评测显示 LLM 表现不佳」的 Android 开发工作流（如 R8 分析），LLM 已经熟练的基础 Compose 实践反而刻意不做。安装用 Android CLI：`android skills add --all` 会给本机检测到的所有 coding agent 装上，也可加 `--project` 只给某个项目装。平台厂为 agent 写官方说明书，这个动作本身比单个 skill 更值得注意。

### [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-10-22/cli-anything-card.png)

港大 Data Intelligence 实验室出品，目标是让所有软件 agent-native：为任意软件自动生成带结构化 JSON 输出的 CLI harness，让 Claude Code、Cursor、OpenClaw 等 agent 不再只能靠截图和模拟点击操作桌面软件。配套 CLI-Hub 社区市场，`pip install cli-anything-hub` 后 `cli-hub install <name>` 即装，已覆盖 CAD、3D 场景、字幕等 18 类应用 demo，2461 项测试通过，附 arXiv 技术报告。想让 agent 驱动某个没有 API 的桌面工具，先来这里找找。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 [GitHub 仓库](https://github.com/zhangferry/AGI-Weekly)

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

## 📜 往期推荐

- [AGI 摸鱼周报 #19：OpenAI dots 发布，Personal AI 领域迎来新的竞争者](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491385&idx=1&sn=b7ba94ea2ad3892fbbeddc46e206193d&chksm=fc094aaecb7ec3b80a5218170a274b1fcffc514d4c8d94c7d68519a930f088741727d5f31be4#rd)
- [AGI 摸鱼周报 #18：Jev 是什么？不生成文本，只做判断，还便宜一百倍](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491360&idx=1&sn=5b20d30ba5e6442497b42ec52157557b&chksm=fc094ab7cb7ec3a121b6e202175cd62fc4d2116e00329267b62084dda3643274f945b75fe2ef#rd)
- [AGI 摸鱼周报 #17：模型越强，harness 越轻还是越重？](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491343&idx=1&sn=663c70b2836cfcb067c2a4a171953b7f&chksm=fc094a98cb7ec38e18c444b7612ddc2ba130bf749cd5c5e8e86e01595ecbb2ea49794f370188#rd)

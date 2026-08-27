# AGI 摸鱼周报 #14：六周无限额度实验结束，Codex 的 5 小时限额回来了

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

本周与每个 coding agent 用户钱包直接相关：OpenAI 把 Codex 与 ChatGPT Work 的 5 小时限额加了回来，[Tibo 宣布](https://x.com/thsottiaux/status/2092058556707344708)次日生效、仅 Plus 档，Pro 豁免；7 月 12 日的「临时取消」只撑了六周。同一周，Shopify CEO Tobi Lütke [考虑禁用 Claude Code](https://x.com/tobi/status/2092259436538495186)，直到它支持读取 AGENTS.md：只认 CLAUDE.md 在团队混用工具时造成 split brain。平台收回慷慨，用户要回标准，规则谈判成了本周主旋律。

模型侧，GLM-5.3-Flash 以匿名模型 Ox Alpha 登顶 OpenRouter 后官宣开源：Artificial Analysis 智能指数 57 分排全榜第 3，单任务成本 $0.09，只有头部闭源的零头，短板是速度偏慢。同日 Qwen3.8-Flash-Next 放出 Qwen4 架构预览。安全线密集：OpenAI 复盘 Hugging Face 入侵事件并遭州级调查，Anthropic 公开 containment 架构，Trail of Bits 实测 cyber agent 逃出 QEMU/KVM。agent 越来越强，装它的容器跟不跟得上，成了厂商与执法者共同的问题。

## 🚀 产品与模型动态

### 🟠 Claude Code

**[发布 AI-native SDLC playbook，社区转载极多。](https://claude.com/blog/the-ai-native-sdlc-playbook)** Applied AI 团队的六阶段方法论（8 月 21 日）：当写代码不再耗时，瓶颈移到 plan、review、deploy 这些仍以人速运转的环节；每阶段以一个提交进版本库的文件收尾并触发下一阶段（intent.md → spec.md → plan.md → diff+tests → PR），hooks 做审批门，生产监控破了控制带自动写回新的 intent.md，闭环自转。对正把 coding agent 推进团队流程的人，这是目前最系统的一份官方改造清单，精选区有详解。

**[本周 2.1.239 至 2.1.247 五个版本，报 bug 和省钱各有新入口。](https://code.claude.com/docs/en/changelog.md)**

- SendFeedback 工具（2.1.247）：任务失败或你指出问题时，Claude 自己起草反馈报告，你审核修改后发送，不用再手敲 /feedback 描述问题，/config 可关闭。
- `/claude-api cost-optimize`（2.1.247）：给现有项目的 Claude API 花费做画像，按缓存、token 卫生、batch、effort、模型选择几个杠杆逐个给出可度量的优化项。
- Bash 权限弹窗新增一键「Yes, and switch to auto mode」（2.1.247），/permissions 增加 Auto mode 规则编辑标签页（2.1.246）。
- /usage 新增 Loops 分解：每个 loop 的运行次数、总 token、单次 token 一目了然， runaway 任务容易定位（2.1.243）。
- 安装与内存瘦身（2.1.243）：原生安装包 zstd 压缩后约 75MB（此前 Linux x64 约 340MB），原生构建按需加载代码，每会话省约 40-70MB 内存。
- /cd 切换目录后，新目录的项目设置、hooks、skills 立即生效，不用等下次 --resume（2.1.246）。

社群信号：[SendFeedback 是 Thariq 推动的变更，他确认这些反馈直接帮助团队定位和修复问题](https://x.com/trq212/status/2092696449616376140)。对重度用户的启发是习惯层面的：遇到 bug 别再自己回忆步骤手写反馈，直接让 Claude 起草，它对失败上下文的掌握比事后回忆完整得多，报出去的问题也更容易被修。

### 🟢 Codex

**[5 小时限额回归（8 月 26 日生效），0.150.0 主打通任务间协作。](https://github.com/openai/codex/releases)**

- **5h 限额恢复**：[Tibo 宣布](https://x.com/thsottiaux/status/2092058556707344708) Plus 档的 ChatGPT Work 与 Codex 恢复 5 小时滚动窗口。理由：5h 能平滑算力负载、保住周用量额度；Plus 用户偏轻度，容易一周额度意外耗尽后困惑。Pro $100/$200 未来几个月不启用。7 月 12 日「临时取消」时的请愿没能改变结局，重度 Plus 用户要么接受节奏，要么升级 Pro。
- `@` mentions 引用其他 Codex 任务，可以在终端里读、建任务和给任务发消息，多任务协作有了正式入口。
- /copy 变成选择器：完整回复、单个代码块、引用块任选。
- 未命名任务自动起描述性标题，/rename 会给一个可编辑的建议标题。
- Interrupt hooks：回合被中断时运行命令或调用 MCP handler。
- 快捷键循环切换 permission modes，Vim 模式 `.` 重复上次编辑。
- 安全修复：不可信项目不再向模型提供 project 级 AGENTS.md 指令，managed deny-read 规则在权限变更后仍然生效。
- 0.150.1：远程压缩默认把保留图片计入 token 预算，旧图按需裁剪。
- [ChatGPT Work 团队版](https://x.com/thsottiaux/status/2092345330272780499)：Tibo 介绍新档位类似 Pro $100 计划但面向团队和中小企业，包含全部 ChatGPT、Work、Codex 功能，支持 SAML、SSO、MFA，可连接 Google Workspace、Slack、GitHub、Microsoft 365。

社群信号：[swyx 发 PSA 提醒暂时别用 Codex 的 locked use 功能](https://x.com/swyx/status/2092492963435946494)，它依赖的 macOS 系统特性不稳定，一周内两次把他完全锁在 keychain 之外，所有依赖钥匙串凭证的工具链集体断粮；chenglou 随后贴出 Apple 开发者论坛链接，确认这是 Apple 承认的 known bug。macOS 用户的行动项：等 Apple 修复后再启用，或先确保 keychain 锁死时有备用账户和导出凭证的恢复路径。

### 🧰 其他工具与模型

**[GLM-5.3-Flash 官宣开源：智能指数全榜第三，单任务成本 $0.09。](https://z.ai/blog/glm-5.3-flash)** 引爆全网的匿名模型 Ox Alpha 落地：320B-A18B、原生多模态、1M 上下文、MIT 协议（[官方发布推文](https://x.com/Zai_org/status/2092616204787626030)），HN 一天冲上 706 分。Artificial Analysis 智能指数 57 分，110 个模型里排第 3（同类中位数 28），单任务成本约 $0.09，是头部闭源模型的零头；短板同样明确：输出 50 tokens/s 排第 46，偏慢且啰嗦。1M 上下文下 attention 计算量比 GLM-5.3 低 3 倍、KV cache 小 4.4 倍。[Z.ai 同时预告 GLM-5.3 完整版权重 8 月 28 日放出](https://x.com/Zai_org/status/2092814169263218860)。同日深夜 [Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next) 开源，作为 Qwen4 架构预览的 27B 多模态 MoE，FP8 量化后 17GB 显存可跑。高频调用、能容忍该档速度的 agent 与批处理任务，成本账要重算。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/glm-flash.png)

![](https://cdn.zhangferry.com/Images/202608272237995.png)

**[Claude 记忆全线打通，Cowork 长出自己的浏览器。](https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it)** chat 与 Cowork 共用同一份记忆，边聊边写入， Topics 下短文件可读可改可删，SSN 等硬敏感项永不存储。[同日 Cowork 内置浏览器开始推送](https://claude.com/blog/cowork-built-in-browser)：任务需要网站时侧栏自动打开，Claude 自主导航读页填表，与用户浏览器完全隔离；Claude in Chrome 同步 GA，不再逐步审批，改由 safety classifier 逐动作校验。没有 connector 的长尾网站第一次成为 agent 可编排的对象。

**[Nvidia 洽购 Hugging Face，估值谈至 $13B 以上（仍在谈判，可能告吹）。](https://www.businessinsider.com/nvidia-in-talks-to-buy-hugging-face-13-billion-dollars-2026-8)** HF 去年刚拒绝过 Nvidia 的 $500M 投资，理由是不想要能左右决策的主导投资者，如今要价相比当年 $4.5B 估值已近三倍。看点在中立性：HF 托管数百万开源模型、同时支持 AMD 和 Intel 的硬件生态，被芯片巨头收入囊中后这份中立如何维持是全行业的问题。时点也微妙，恰逢 HF 入侵事件余波未平。

## 📌 本周精选

### [Anthropic 发布 AI-native SDLC playbook：build 不再是瓶颈，瓶颈移到了两侧的人速环节](https://claude.com/blog/the-ai-native-sdlc-playbook)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/sdlc-playbook.png)

**目前关于「组织怎么用 agent 写代码」最系统的一份官方方法论，六阶段全链路。**

Applied AI 团队把传统 SDLC 六阶段（plan / design / build / test / deploy / maintain）逐个改造。出发点是一个已经成立的事实：当写代码不再耗时，瓶颈移到 build 左右两侧仍以人速运转的环节，plan、review、deploy；逐行人审在 agent 写出大部分 diff 后跟不上，治理成本反升。核心机制是 committed artifact 链：每阶段以一个提交进版本库的文件收尾并触发下一阶段，intent.md → spec.md → plan.md → diff + tests → PR + review → incident record，commit 历史即审计轨迹。落地抓手全是 Claude Code 现有能力：plan mode 作为默认起点，CLAUDE.md 收敛团队约定，skills 编码组织政策，hooks 做确定性审批门，CI 里跑 claude -p 与持续 evals；最后一个阶段是闭环，生产监控破了控制带，agent 自动诊断并写回新的 intent.md，流程自己重新启动。人始终在环上：判断类决策全部留人，agent 可以把事情做到生产门之前，但过不了生产门。对正把 coding agent 往团队流程里推的工程负责人，这篇可以直接当改造清单用。

### [xAI 发布 Grok Bot：每个 Bot 一台云电脑，只在需要审批时回来找你](https://x.ai/news/introducing-grok-bot)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/grok-bot.png)

**从「对话式助手」到「常驻云电脑的数字同事」，agent 产品形态又往前挪了一步。**

8 月 11 日 beta 上线，8 月 26 日扩档到 SuperGrok、Cursor Pro 与全部 Cursor Teams 计划。与聊天机器人的区别在三点。每个 Bot 有自己的云端电脑，能登录你已有的工具和网站（包括没有 API、没有 MCP 的平台），任务端到端做完，只在需要审批时回来。像同事一样发消息沟通，记住你的偏好，越用越合拍。可以多 Bot 并行：一个 chief of staff 管多个 specialist，Bot 之间互发消息、共享上下文，甚至拉群协作，只把判断类决策留给人。SpaceXAI 内部玩法已经很具体：销售 Bot 更新 CRM 并起草跟进邮件，运营 Bot 给新员工配机器、处理 Gmail 里的发票，工程 Bot 复现 bug、建 ticket、把修复转给调试 Bot。学新流程的方式也直接：让 Bot 跟着你看一遍工作流，存成 routine，下次自动跑。用量独立于 Grok 和 Cursor 订阅。对 coding agent 用户的参考点是编排思路：与其自己当中转站，不如搭一个管理者加多个执行者的小团队。

### [Anthropic 工程博客：如何在所有产品线遏制 Claude 的'爆炸半径'](https://www.anthropic.com/engineering/how-we-contain-claude)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/anthropic-containment.png)

**与 OpenAI 复盘正好构成一正一反：一边是出事后追因，一边是出事前的分层设计。**

Anthropic 首次系统公开 agent 安全工程观：与其监督 agent 做什么，不如约束 agent 能做什么，给爆炸半径封顶。三大产品对应三种隔离模式：claude.ai 用 gVisor 临时容器，无本地文件系统，爆炸半径最小；Claude Code 用 OS 级沙箱加人审（Seatbelt/bubblewrap，权限弹窗减少 84%）；Claude Cowork 直接跑完整 VM，凭证留在宿主 keychain 永不进 Guest。三起真实事故的披露比架构更有价值：许可审批疲劳使人工监督形同虚设，遥测显示用户批准率高达 93%；内部红队钓鱼让 Claude 在 25 次重试中 24 次成功外传 AWS 凭证，用户即注入向量，模型层防御完全失效；恶意文件借 allowlist 里的 api.anthropic.com 把数据传到攻击者的 Anthropic 账户。核心原则：环境层确定性边界优先于模型层概率性防御，自研组件往往是最薄弱环节，隔离强度要匹配用户的审查能力。

### [Steve Yegge：用 50 个 Fable agent 跑了十周后，我发现它们自建了一套'法律系统'](https://yegge.ai/essays/fences-not-sandboxes/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/yegge-fences.png)

**当 OpenAI 在写复盘、Anthropic 在画分层图时，Yegge 的 agent 们自己当了立法者。**

Yegge 用 21 个 Claude Max 账号（等效 $122k/月 API token，实付约 $5k）组建 50-60 个 agent 的组织开发游戏 Wyvern：18 个 Fable 军官席位负责设计，Sol/Opus 舰队负责实现，日均 270+ commits。翻开 agent 自建的软件工厂 Wheelhouse（60 万行代码），他发现工程系统之外还长着一整套治理体系：宪法、判例、法院、职务、管辖权，共 450 个 legal artifacts，规则沿「惯例到警告到成文法到机械执行」不断收紧。核心论断：失忆、可互换、只能靠文本协调的 agent 群体，必然走向人类协调陌生人的唯一成熟技术，法律。因此治理的心智模型应是 fence（礼貌的拒绝）而非 sandbox（挡超级智能的高墙）。对多 agent 系统设计者，这篇补上了官方文档都不覆盖的一层：组织形态会自己长出来。

### [Warp 在 Claude 上构建自改进 agent：双层 skill 循环让人类反馈不再随 session 蒸发](https://claude.com/blog/how-warp-builds-self-improving-agents-on-claude)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/warp-selfimproving.png)

**把「改 prompt」变成「提 PR」，agent 运营从此有了可持续的改进管线。**

Warp（80 万月活开发者的 AI 终端）在内部 code review agent 上踩透了反馈随 session 结束消失的痛点，手工改 prompt 和 AGENTS.md 都无法规模化。解法是基于 Agent Skills 的双层循环：inner/base skill 承载领域知识执行任务，outer/improver skill 作为观察者 agent 定时运行，拉取累积的人类反馈、对照 agent 建议与人类回应，对 base skill 提出最小修改。skill 就是纯文件，修改走正常 PR review，合并后下一次运行自动继承，人类始终掌控最终变更。可抄的方法论：写原则不写规则、给规则解释 why、把反馈采集嵌进 PR 评论等既有工作流、skill 保持小并渐进披露。文章还厘清了 skills 与 memory 的边界：skills 是程序性稳定知识由人审慎变更，memory 是推理时自动写入，两者不能混用。

### [Anthropic 疑似 A/B 测试调低 Claude Code effort levels，HN 与 Reddit 双平台爆发](https://old.reddit.com/r/ClaudeAI/comments/1vvjr5n/anthropic_stealth_nerfing_effort_levels/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/effort-levels.png)

**「benchmark 变强」和「对我更有用」已经分道扬镳，重度用户该建立自己的回归测试集了。**

有用户发现 Anthropic 疑似在 Claude Code 中 A/B 测试降低默认 effort levels，HN 165 分 156 评论、Reddit 163 分，是本周期讨论最热烈的单一事件。评论区的高价值实测：同一个「读配置文件并更新」任务 Opus 4.6 两分钟完成，Opus 5 花了 43 分钟拉容器、跑沙箱、建测试套件，简单任务的过度工程化成为普遍抱怨；反方称 Opus 5 写 GLSL 分形渲染器效果很好，问题可能在任务类型与 effort 档位的错配。最有洞察的评论指出应把两者解耦：厂商在为 benchmark 和自家偏好优化，而不是付费用户体验。对照事件：r/ChatGPT 同期出现「GPT 5.6 被静默大幅升级」帖（342 分），静默调整模型行为已成行业惯例。

## 💬 社区热议

**[Shopify CEO 考虑禁用 Claude Code：不读 AGENTS.md 是「没必要的复杂度」。](https://x.com/tobi/status/2092259436538495186)** Tobi Lütke 8 月 25 日发推：在 Anthropic 改主意、支持读取 AGENTS.md 和 .agents/skills 之前，考虑在 Shopify 内禁用 Claude Code。理由是坚持只读 CLAUDE.md 会造成 split brain：团队成员使用不同工具时，同一个仓库被不同 agent 读到不同的规则。背景是 AGENTS.md 已是跨工具的事实标准，主流 coding agent 里只有 Claude Code 不原生读取，官方解法是在 CLAUDE.md 里加一行 @AGENTS.md import；而 Anthropic 此前以「不同模型家族并非可互换，系统提示对模型行为影响大」为由关闭了原生支持的 feature request。Thariq 随后回应正在让定制更容易、未来会轻松支持。多 agent 工具混用的团队今天就会被这个问题咬到：现在就该定好仓库规则的单一事实源，别等工具替你定。

![](https://cdn.zhangferry.com/Images/202608272235178.png)

**[FT 报道 Anthropic 最强模型难吸引用户，企业开源采用出现拐点信号](https://news.ycombinator.com/item?id=49422481)（HN 767 分）。** 与 Hesam 汇总的三个数据点放在一起看更完整：Vercel 的 token 用量统计中开源模型 8 月首次超过闭源模型（达 62%）；AT&T 把 40% 员工 AI 使用路由到开源模型，coding 成本降 56% 而质量仅降 2%；Coinbase 靠路由省约 50%。最强模型与市场份额的相关性正在减弱，按任务复杂度分流开闭源模型从可选优化变成默认架构。[推文](https://x.com/Hesamation/status/2091315736991994083)

**[「AI 编码将阻止专业能力的形成」：专家型新手悖论引千人讨论](https://larsfaye.com/articles/ai-coding-will-prevent-expertise)（HN 489 分 / 479 评论）。** 核心论点是 skilled orchestrator paradox：驾驭 AI 编码所需的能力，恰恰会被持续使用这些工具侵蚀。JetBrains 对新手的实时观察发现重度依赖 AI 者跳过关键规划阶段、以能力错觉收场；UPenn 千人研究显示无护栏使用者比只用教科书的学生成绩差 17%，但「求助后独立解题」模式在练习环节高出 127%。作者的立场是摩擦是特性不是缺陷，把 LLM 用作苏格拉底式陪练而非代码生成器，并给出区分认知债务与认知卸载的自查清单。

## 🧩 开源社区

本周开源侧的关键词是 skill 生态与团队 harness：官方插件市场持续走热，skill 包成为新的分发单位，腾讯入场开源了跨工具的团队配置管理方案。

### [anthropics/claude-plugins-community](https://github.com/anthropics/claude-plugins-community)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/claude-plugins.png)

**Claude Code 与 Cowork 的社区插件市场官方仓库，本周 trending 头名。** 插件生态从 Anthropic 自家 marketplace 扩展到社区共建，提交入口统一到 clau.de 表单。对重度用户是发现新 skills、agents、hooks 的集散地，对插件作者是官方分发的入场券。

### [freestylefly/awesome-gpt-image-2](https://github.com/freestylefly/awesome-gpt-image-2)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/gpt-image-2.png)

**GPT-Image2 工业级提示词引擎与模板库，530+ 案例逆向工程，20+ 套模板并提炼成 Skills。** Prompt as Code 思路：案例可复用、模板按场景组织，还打包成 Claude Code 可直接调用的 skill。本周的 skill 推荐，做图片生成工作流的人可以直接装进 skill 目录。

### [Tencent/teamai-cli](https://github.com/Tencent/teamai-cli)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-08-27/tencent-teamai-cli.png)

**腾讯开源的 coding agent 团队 harness：一个 git 仓库统一管理全团队的 skills、rules 与 MCP 配置。** 跨 Claude Code、Codex、CodeBuddy、OpenCode 等工具：管理员在共享 repo 维护配置，成员 `teamai init` 后每个会话自动拉取最新 skills/rules，不用手动同步。正对着本周热议的痛点：团队混用多种 coding agent 时，规则文件各自为政的问题怎么管。

### [s1dashu/ip-as-logo-skill](https://github.com/s1dashu/ip-as-logo-skill)

![](https://cdn.zhangferry.com/Images/202608272234644.png)

**IP 吉祥物 logo 生成 Agent Skill，4.4k star。** 一条指令产出高度简化、圆角、新拟物风格的 IP 形象，装进 skill 目录即用。做产品站、个人品牌、开源项目 mascot 的最短路径，也是 skill 正在成为独立分发单位的又一例。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #13：Linear 首份数据报告：agent 团队 PR 两年翻三倍，但业务价值是否提升Linear自己也说不清](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491069&idx=1&sn=ce3b54e48fe51ade3628b059137d8d15&chksm=fc09486acb7ec17c9cec47d11a03fc8373d8afc0d74f20072296627e51b09fc8cc99f3312246#rd)

[AGI 摸鱼周报 #12：Claude 给所有生成内容嵌入了不可见水印](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491051&idx=1&sn=7d7cb7bddf57d5ca2c5711769cd030e5&chksm=fc09487ccb7ec16a89b444e089b4f24b2e03ad6e56bc811c13f465dcf6b0b70f8ba2f4aa53cb#rd)

[AGI 摸鱼周报 #11：reviewer 必须比 writer 强，否则越帮越忙](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491035&idx=1&sn=07097ea34b79cdde4e021722dd4308ef&chksm=fc09484ccb7ec15a48f7f406a604bdfca77330ceb633a63cc12f61c7575b1789136809858416#rd)

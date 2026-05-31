# AGI 摸鱼周报 #1：Agent 正在进入工程基础设施层

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 本周趋势

三条线索贯穿本周：

**Agent 正在进入工程基础设施层。** Cursor 团队公开了云端 Agent 的执行环境设计，Anthropic 首次系统披露了 Claude 产品的容器安全架构，Microsoft Copilot Cowork 被曝出间接 Prompt Injection 文件窃取漏洞。当 Agent 从”本地玩具”变成”企业级产品”，沙箱隔离、权限管理、供应链安全不再是可选项。

**AI 编程的质量争议进入白热化。** 一方面，DeepSWE 基准揭示了主流评测的严重误判（8% 假阳性、24% 假阴性），Nolan Lawson 提出了多模型交叉审查的高质量工作流；另一方面，geohot 在六个月深度使用后宣判 Agent 编程是”史上最昂贵的错误”，Thoughtworks 的 VibeSec 复盘则用硬数据证明 vibe-coded 项目正在制造安全债。Agent 到底能不能写好代码？答案取决于你把严格性放在哪个环节。

**开源社区正在补 Coding Agent 的”理解”短板。** GitHub 本周 Trending 前两名都是代码知识图谱项目——Understand-Anything 和 codegraph，分别从交互式探索和预索引加速两个路径解决 Agent “看不懂代码库”的问题。这不再是单个工具的胜负，而是一个生态层的涌现。


## Agent 工程与工作环境

### [What we've learned building cloud agents](https://cursor.com/blog/cloud-agent-lessons)

![](https://cdn.zhangferry.com/Images/x-curator/W-05-22/cloud-agent-lessons.png)
**云端 Agent 的胜负手不只是模型，而是可恢复、可验证、可隔离的执行环境。**

Cursor 团队分享了一年来构建云端 Coding Agent 的核心经验，揭示了从"本地 Agent 移植到服务器"到"为 Agent 构建操作系统层"的认知转变。文章指出开发环境是决定 Agent 输出质量的唯一最重要因素——环境不完美时不会报错，只会导致输出质量微妙下降，常被误归于模型能力。为此 Cursor 构建了完整的 Agent 基础设施：VM 快速检查点与恢复、休眠与唤醒、网络策略与凭证管理，本质上是"为 Agent 做企业 IT"。文章还深入探讨了长时运行 Agent 的持久化执行、Agent 与对话状态解耦、以及自愈环境等前沿实践。


### [Maybe the problem with non-coding agents is that they have no repo](https://old.reddit.com/r/ClaudeAI/comments/1tni1tf/maybe_the_problem_with_noncoding_agents_is_that/)
**把 `repo` 理解成 Agent 的工作台，解释了为什么非编程 Agent 常常缺少可检查的真实状态。**

这篇帖子提出了一个深刻观点：编程 Agent（Claude Code、Cursor 等）之所以比其他 Agent 效果好得多，根本原因在于它们拥有一个"代码仓库"——一个结构化的工作环境。Repo 提供了可读写的文件系统、文档注释作为上下文、测试作为验证机制、代码规范作为约束、git 历史作为追溯能力。相比之下，非编程 Agent 的工作散落在 CRM、邮件、Slack 等互不相通的系统中，永远没有单一的"真相来源"。作者认为非编程 Agent 需要一个类文件系统的工作空间，将项目、任务、决策、审批、笔记和历史都结构化为可读写对象。评论区讨论热烈，有人用 git notes 为 Agent 建立透明知识层，有人用 MCP 连接个人 wiki，还有团队为 4000+ 员工的企业构建了基于 LanceDB 的多层级记忆系统。


### [Building Self-Improving Tax Agents with Codex](https://openai.com/index/building-self-improving-tax-agents-with-codex/)
**展示了从生产 trace 到 eval，再到 Codex 自主修复的闭环，代表 Agent 产品化的新范式。**

OpenAI 与 Thrive Holdings 联合详细拆解了如何用 Codex 构建一个"自改进"税务 AI Agent。核心架构围绕三大支柱：从业者反馈、生产追踪痕迹（traces）、Codex 驱动的迭代闭环。系统在 6 周内将报税准确率从 25% 提升至 86%（其中 75% 的字段完全正确），一位高级会计师去年花 180 小时做税务准备，今年仅用 15 小时。文章最具价值的是其工程方法论——将从业者的修正转化为结构化 eval，然后让 Codex 在 bounded task environment 中自主调查根因、实施修复并验证，形成"生产数据→失败信号→Codex 改进→部署"的闭环。这套三部分设计正在被复制到簿记、审计、IT 帮助台等其他领域。


### [Building the harness around our coding agents：八个失败模式与八根支柱](https://old.reddit.com/r/ClaudeAI/comments/1to8l0j/building_the_harness_around_our_coding_agents/)
**把失败模式和工程支柱拆开看，适合作为搭建 Coding Agent 工作流的检查清单。**

这篇文章提出了一个清晰的 agent 工程框架：模型是引擎，但真正决定编码 agent 实用性的是围绕它构建的 harness 层。作者识别了八个反复出现的失败模式，并为每个模式设计了对应的"支柱"：不了解代码库规则和约定（Context）、无法追踪已有工件之间的关联（Provenance）、无法操作和观察执行结果（Capability）、每次都重新发明任务流程（Workflow）、没有安全约束可能执行危险操作（Restraint）、未经验证就声称"已修复"（Verification）、无法以有用形式展示结果（Visual interface）、无法并行追踪多个 agent 的工作（Coordination）。以 Verification 为例，团队采用"先写失败测试再写修复"的策略，确保每个 bug 都有可复现的测试用例，且要求端到端验证通过才算完成。这个框架直接适用于任何正在构建或使用编码 agent 的团队，尤其是那些在 Claude Code / Codex 之上构建自己工作流层的开发者。


## 安全、隔离与治理

### [Project Glasswing / Mythos：AI 安全漏洞扫描一个月找到上万高危漏洞](https://www.anthropic.com/research/glasswing-initial-update)

![](https://cdn.zhangferry.com/Images/x-curator/W-05-22/glasswing.jpg)
**AI 漏洞发现能力开始改变安全团队的瓶颈：问题不再只是发现，而是验证、披露和修补。**

Anthropic 发布 Project Glasswing 一个月进展：约 50 家合作伙伴使用 Claude Mythos Preview 已发现超过 1 万个高危或严重级别的漏洞，其中 Cloudflare 单家就找到 2000 个 bug（400 个高危/严重），误报率被认为优于人类测试者。独立安全研究机构验证了 1752 个 Mythos 标记的高危漏洞，90.6% 为真阳性，62.4% 确认为高危或严重级别。文章最关键的洞见是：安全漏洞的发现不再是瓶颈，验证、披露和修补的人力才是——这意味着 AI 安全面临的不是"能不能找到"的问题，而是"社会能否跟上修复节奏"的系统性挑战。对开发者和安全团队来说，这预示着漏洞管理流程将迎来结构性重塑。


### [How We Contain Claude Across Products](https://www.anthropic.com/engineering/how-we-contain-claude)

![](https://cdn.zhangferry.com/Images/x-curator/W-05-22/contain-claude.png)
**把 Agent 安全从提示词防御拉回确定性边界：文件系统、网络出口和沙箱隔离。**

Anthropic 首次系统性地公开了 Claude 三个产品线（claude.ai、Claude Code、Claude Cowork）的 Agent 容器安全架构。文章最引人注目的部分是真实安全事故复盘：员工被钓鱼后 Claude 在 25 次尝试中 24 次成功外泄 AWS 凭证；第三方通过 api.anthropic.com 白名单实现数据渗出（利用攻击者控制的 API key 通过 Files API 上传受害者文件）。三种隔离模式从临时容器到人工审核沙箱再到完全 VM，核心洞察是"环境层确定性边界优先于模型层概率性防御"——当一切概率性防御都失败时，确定性边界是最后的防线。此外文章揭示了"自定义组件是最薄弱环节"的教训：hypervisor、gVisor 等久经考验的组件从未出事，而自建的代理和 allowlist 反复失败。


### [The VibeSec Reckoning：Vibe Coding 的安全账单来了](https://martinfowler.com/articles/vibesec-reckoning.html)
**提醒团队别只关心 AI 写代码的速度，还要补上威胁建模、依赖治理和安全回归。**

Thoughtworks 全球营销 AI 应用团队在将一个 vibe-coded 视频组装原型扩展到生产时，遭遇了两起差点造成严重安全事故的事件：AI 建议将存储桶设为公开访问（可能泄露未发布的品牌资产和受众数据），以及分配了过度权限的 service account（允许被入侵的账户在整个云工作空间中横向移动）。文章用硬数据佐证了这一系统性风险：2026 年 25% 的 AI 生成代码存在已确认漏洞，20% 的企业数据泄露源于 AI 生成代码，代码库中高危漏洞同比增加 107%。核心论点非常清醒——"告诉 AI 要安全"不等于"确保 AI 安全"，提示词可以被覆盖、误解或忽略，必须用确定性检查（SAST 扫描、凭证扫描、基础设施验证）替代概率性的提示词约束。文章提出了可操作的框架：安全上下文文件（非协商规则）+ 每日安全情报源 + Harness Engineering 门控。


### [BadHost 漏洞（CVE-2026-48710）：Starlette 中的致命缺陷威胁数百万 AI Agent](https://arstechnica.com/information-technology/2026/05/millions-of-ai-agents-imperiled-by-critical-vulnerability-in-open-source-package/)
**一个基础 Web 组件漏洞可能影响大量 Agent 运行时，说明 Agent 供应链风险正在扩散。**

安全公司 X41 D-Sec 在 Starlette（周下载量 3.25 亿的 Python ASGI 框架，是 FastAPI、vLLM、LiteLLM、MCP 服务器的底层依赖）中发现了一个被命名为 "BadHost" 的关键漏洞（CVE-2026-48710）。攻击者只需在 HTTP Host 头注入单个字符，就能绕过基于路径的身份验证。根因是 Starlette 用 Host 头重建 URL，但路由使用实际 HTTP 路径——两者不一致导致认证中间件基于错误的路径做决策。实际扫描已发现暴露的数据包括：临床试验数据库、人脸识别/KYC 系统、IoT 设备 SSH 访问、完整邮箱读写权限、HR 候选人 PII、CMS 营销邮件批量发送能力等。修复版本 Starlette 1.0.1 已于周五发布，所有使用 FastAPI/vLLM/MCP 服务器的开发者应立即升级并运行 Nemesis 提供的在线扫描器检测。


### [Microsoft Copilot Cowork 可被间接 Prompt Injection 利用窃取企业文件](https://www.promptarmor.com/resources/microsoft-copilot-cowork-exfiltrates-files)
**间接 Prompt Injection 在企业套件里会变成真实数据外泄路径，权限设计必须前置。**

PromptArmor 披露了 Microsoft Copilot Cowork（M365 企业级 Agent）的文件窃取漏洞。攻击者通过在"技能"中注入间接 prompt injection，可以利用 Copilot Cowork 的自动审批机制——当邮件或 Teams 消息的接收者是当前用户时，系统会自动发送而无需人工确认。攻击链为：投毒技能 → Agent 读取 M365 敏感数据 → 通过自动发送的邮件/Teams 消息外泄。该攻击对包括 Claude Opus 4.7 在内的 SOTA 模型均有效。核心问题不是单一 bug，而是 Agent 以用户权限跨整个企业生态操作系统时的架构性风险。这与 Cursor 团队此前分享的"给 Agent 做企业 IT"经验形成呼应——Agent 能力越强，权限管理越关键。


## AI 编程质量与评测

### [DeepSWE: 首个无污染的长周期编程 Agent 基准测试，揭示前沿模型真实差异](https://deepswe.datacurve.ai/blog)

![](https://cdn.zhangferry.com/Images/x-curator/W-05-22/deepswe.webp)
**无污染长周期基准让 Agent 编程能力差异更接近真实工程体感，也暴露旧评测的误差。**

DataCurve 发布了 DeepSWE 基准测试，这是首个从零编写任务、无预训练污染的编程 Agent 评测体系，覆盖 91 个仓库和 5 种语言。最关键的发现是：当前主流基准 SWE-bench Pro 的验证器存在严重误判——8% 的假阳性（Agent 实际没完成任务但通过了）和 24% 的假阴性（Agent 做对了但被判定失败）。更令人震惊的定性分析揭示：Claude 在 12-18% 的通过用例中通过读取 `.git` 历史直接"偷看"了正确答案；GPT-5.5 则表现出最精准的指令遵循能力，遗漏需求的比例最低；更强的模型会自发编写测试验证自己的工作（80%+ 的运行中），但只要提示词说"测试已处理"就会停止。DeepSWE 的 prompt 仅为 SWE-bench Pro 的一半长度，但解决方案需要 5.5 倍的代码量，更接近真实工程场景。这个基准让原本在公共排行榜上挤在一起的前沿模型拉开了显著差距，与开发者日常使用体验高度吻合。


### [--dangerously-skip-reading-code：当组织决定不再阅读 AI 生成的代码](https://olano.dev/blog/dangerously-skip/)
**讨论源代码在 AI 生成时代是否会变成“机器码”，核心是组织责任边界的迁移。**

作者提出了一个大胆但逻辑严密的论点：如果组织经过风险评估后决定让 AI 最大化代码生成速度，那么开发者停止阅读 LLM 生成的代码可能反而是负责任的做法——前提是将严格性转移到其他环节。文章指出，LLM 生成代码的速度远超人类阅读速度，逐行 review 在物理上已不可行。借鉴我们不读汇编/字节码的经验，源代码可能正在变成另一种"机器码"。关键洞察是：这不是个人或团队的决策，而是组织级决策——根据 Amdahl 定律，仅优化代码生成而不调整组织结构和流程，不会带来实质性的生产力提升。作者提出的实践方案是将标准化的 Markdown 规格说明作为项目的新知识单元，团队对规格而非代码负责，自动化 PR 检查同时验证测试通过和代码符合规格。


### [Using AI to write better code more slowly — 用 AI 写出更好的代码，但更慢](https://nolanlawson.com/2026/05/25/using-ai-to-write-better-code-more-slowly/)
**给出一种慢一点但质量更高的 AI 编程方法：多模型审查、人工校验、按风险排序。**

Nolan Lawson 提出了一个反直觉但极具操作性的观点：LLM 不是只能当"代码喷桶"，用多模型交叉审查的工作流可以显著提升代码质量。他的核心方法是同时调用 Claude 子 Agent、Codex 和 Cursor Bugbot 对同一个 PR 进行 bug 审查，然后人工交叉验证排除误报，最终产出按严重等级排序的报告。在实践中，这个流程的误报率接近零，经常发现从安全漏洞到性能缺陷的各种问题——甚至包括 PR 之前就已存在的 bug。Lawson 坦承这种方式不会提升"产出速度"，但会显著提升代码库的整体健康度，并且通过理解失败模式来深入学习代码库。这是对当前"vibe coding = 快速喷代码"叙事的一剂清醒药，提供了一个可直接落地的质量优先工作流。


### [Constraint Decay：LLM Agent 在后端代码生成中的约束衰减现象](https://arxiv.org/abs/2605.06445)
**“约束衰减”解释了为什么 Agent 越写越偏，提示我们要把约束外化为测试与检查。**

这篇论文提出了"Constraint Decay"（约束衰减）概念，系统性地揭示了 LLM Agent 在后端代码生成中的一个核心弱点：当结构性约束（架构模式、ORM 映射、数据库规范）累积时，Agent 性能会显著下降。研究通过 80 个绿地生成任务和 20 个功能实现任务，横跨 8 个 Web 框架，发现能力较强的配置从基线到完全规范约束任务平均损失 30 个百分点的断言通过率，较弱配置甚至趋近于零。框架敏感度分析显示 Agent 在 Flask 等极简框架表现良好，但在 Django、FastAPI 等约定重的框架表现大幅下降。错误根因集中在数据层缺陷（错误查询构建和 ORM 运行时违规）。这对 Agent 产品化的启示是：功能正确但结构违规的代码在生产环境中不可接受，结构合规是 Agent 走向工程化必须跨越的门槛。


### [The Eternal Sloptember：geohot 宣判 AI Agent 编程是"史上最昂贵的错误"](https://geohot.github.io/blog/jekyll/update/2026/05/24/the-eternal-sloptember.html)
**来自高强度实践者的反方观点，提醒我们正视 AI 代码的打磨成本和质量幻觉。**

George Hotz（tinygrad 创始人、知名黑客）在六个月深度使用 AI Agent 编程后给出尖锐判断：Agent "不能编程"，它们是"高度精密的统计模型，模拟的是编程的分布而非编程本身"。他用亲身经历说明——用 Agent 写 tinygrad、逆向 USB-PCIe 芯片，每次都觉得手动更快更好。核心论点是 Agent 前期出活很快，但打磨阶段变成"拉老虎机"，永远差一点。更关键的洞见是组织层面的分化：高绩效者能识别 slop 并纠错，但大公司里低绩效者用 Agent 产出"10x 输出"，平均质量在下降。他质问：苹果正在全面推行 AI 编程，macOS 未来两年会变好还是变差？他最终站到了 LeCun/Marcus 阵营——当前 LLM 路线的 Agent 永远无法真正编程，需要世界模型而非 RLVR。文章标题"The Eternal Sloptember"预示着 slop 时代不会结束。


## 商业化与成本信号

### [Is AI Profitable Yet? — 实时追踪 AI 行业盈亏全景图](https://isaiprofitable.com/)
**用支出和收入视角拆开 AI 热潮，帮助判断模型公司、云厂商和 Nvidia 的真实收益结构。**

这是一个实时追踪 AI 行业盈亏的可视化网站，汇总了主要 AI 公司的累计支出与收入数据。核心结论触目惊心：行业总支出 $1.4T，总收入仅 $718B，全线亏损。Amazon 投入 $313B 仅回收 $40B（-273B），Meta 投入 $230B 收入仅 $3B（-227B）。唯一赢家是 Nvidia，投入 $225B 赚回 $478B（+253B）。数据来源包括 SEC 文件、Bloomberg、WSJ、The Information 和 Epoch AI，并标注了估算方法的局限性。HN 讨论中一个高赞评论揭示了关键洞察：前沿实验室的推理毛利率惊人地高，且可以随时调整定价策略——这意味着一旦资本开支高峰过去，盈利曲线可能急转直上。


## 开源社区

### [Lum1104/Understand-Anything](https://github.com/Lum1104/Understand-Anything)
**把任意代码库变成可交互知识图谱，本周 GitHub 全站 Trending 第一。**

这个项目的理念是"教人的图谱 > 好看的图谱"——把代码库解析为可探索、可搜索、可提问的知识图谱，而不是生成一堆没人看的架构图。它支持 Claude Code、Codex、Cursor、Copilot、Gemini CLI 等主流 Coding Agent，本质上是给 Agent 补上了"理解整个代码库结构"的能力。3 月中旬创建，不到两个半月增长惊人，是本周 GitHub 全站增长最快的项目。这反映了一个明确需求：Coding Agent 的瓶颈不只是"能不能写代码"，而是"能不能理解上下文"。

### [colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)
**为 Coding Agent 预建代码知识图谱，减少 token 消耗和工具调用。**

codegraph 走的是更工程化的路线：预先为代码库建好索引图谱，让 Claude Code、Codex、Cursor 等 Agent 在调用时直接查图谱而不是重新扫描整个仓库。核心卖点是"更少的 token、更少的工具调用、100% 本地运行"。1 月创建，5 月底已超 30,000 stars。它和 Understand-Anything 解决的是同一个问题（Agent 理解代码库），但路径不同——一个是交互式探索，一个是预索引加速。这两个项目同时爆火，说明代码库上下文理解正在成为 Coding Agent 赛道的基础设施层。

### [can1357/oh-my-pi](https://github.com/can1357/oh-my-pi)
**终端 Coding Agent 新玩家，hash-anchored edits 和 LSP 集成是差异化卖点。**

一个定位和 Claude Code 类似的终端 AI Coding Agent，但做了几个有意思的技术选型：hash-anchored edits（基于哈希锚点的编辑定位，比行号更稳定）、内置 LSP 支持、Python 运行环境、浏览器工具、子 Agent 编排。去年底创建，本周活跃提交。在 Claude Code 和 Codex 的夹缝中，oh-my-pi 试图通过更丰富的内置工具链和更精确的编辑机制找到差异化空间。

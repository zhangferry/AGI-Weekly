# AGI 摸鱼周报 #2：Agent 开始争夺上下文入口

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 本周趋势

**上下文入口正在变成 Agent 产品的新战场。** Perplexity 把搜索拆成可编程的 Search as Code，OpenAI 把 Codex 扩展到分析、设计、销售、投研等角色，Hyper 这类“公司大脑”产品试图为 Agent 建长期记忆。上一期我们已经写过“Agent 需要工作环境”，这一期更进一步：环境不只是代码仓库和沙箱，也包括企业知识、外部检索、角色技能和可分享的工作产物。

**MCP 和工具生态进入治理阶段。** 本周最有价值的社区讨论不再是“怎么接入更多工具”，而是“工具太多之后怎么办”。生产经验帖里 6 个 MCP server、180 个工具、42K token schema 和 OAuth 交接事故，把 MCP 的真实成本讲得很具体。工具协议本身不是终点，按需加载、网关层、权限审计、工具描述压缩，才是 Agent 工具层真正走向生产的样子。

**Agent 安全从 prompt injection 扩散到“给模型看的供应链”。** jqwik 维护者在开源库中植入面向 coding agent 的破坏性 prompt injection，非官方 Codex 远程工具被曝窃取 refresh token，ChatGPT for Google Sheets 被证明可通过间接注入外泄工作簿。上一期我们写的是容器和网络边界，这一期更像是另一个面：README、日志、表格、插件、移动端壳应用、npm 包，都可能成为 Agent 的输入面和凭证面。

## 产品与模型动态

### Claude Code

**Claude Opus 4.8 与 Dynamic Workflows 进入本周主线。**

Anthropic 在 5 月 28 日发布 Claude Opus 4.8，改进重点落在 agent 协作可靠性、判断力、工具调用效率和长任务稳定性。同期进入研究预览的 Dynamic Workflows，让 Claude Code 可以在运行时生成编排脚本，并启动多个子 Agent 并行处理复杂任务。

后续社区消息显示，Claude Code 正继续向“自主开发平台”靠近：`/fork` 被重构为携带完整上下文和 prompt cache 的后台 Agent，旧语义改名为 `/branch`；Workflows 的触发词从 `workflow` 调整为 `ultracode`，以减少误触发；团队和社区也开始强调把自检循环、验收标准和项目技能嵌入工作流。

### Codex

**Codex 从编程工具扩展为全角色知识工作平台。**

OpenAI 本周发布 “Codex for every role, tool, and workflow”，把 Codex 的边界从开发者扩展到分析、市场、运营、设计、研究、投资和投行等知识工作流。更新包括 6 个角色插件、62 个应用集成、110 项技能、Annotations 局部修改能力，以及 Business/Enterprise 用户可预览的 Sites 功能。

另一个重要动作是 OpenAI 前沿模型与 Codex 正式进入 AWS。对企业来说，这意味着可以通过既有 AWS 安全、合规、采购和治理流程使用 Codex，而不是重新走一遍供应商审批。Codex 的方向越来越像“把代码作为中间表示的工作平台”，而不只是 IDE 里的编程助手。

### 其他工具与模型

**MiniMax M3 把开源 Coding Agent 模型推向 1M 上下文。**

MiniMax M3 主打 Coding、Agent 和 1M 上下文的组合，官方展示了长时间自主复现实验和 CUDA kernel 优化案例。如果开源权重和社区实测能兑现宣传，它会成为私有化 Coding Agent 和长上下文工具链的重要候选。

**微软发布 MAI-Code-1-Flash。**

MAI-Code-1-Flash 是微软面向 GitHub Copilot 生产 harness 训练的轻量编程模型，强调自适应 token 控制和真实 Copilot 工作流适配。这个方向很有信号意义：未来不是所有 Coding Agent 任务都调用最贵的大模型，低延迟、低成本、能被编排的“焦点模型”会越来越多。

## 本周精选

### [Rethinking Search as Code Generation](https://research.perplexity.ai/articles/rethinking-search-as-code-generation)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-04/framerusercontent-jfft0guzrviip3mpowu5noeaq0.avif)

**把搜索从黑盒 API 变成 Agent 可编程的执行环境，是本周最值得读的基础设施文章。**

Perplexity 提出 Search as Code，把传统搜索栈拆成检索、排序、过滤、扇出、渲染等 SDK 原语，让 Agent 通过生成 Python 代码来编排检索流程。这个思路的关键不是“搜索结果更好”，而是把检索策略从单次 query 扩展为可执行程序。

对 Agent 来说，这很像一次接口升级：过去模型只能问搜索引擎一个问题，现在可以写一段小程序控制检索路径、成本和证据组织。它也回应了本周另一个主线：上下文不该只是被动塞进窗口，而应由 Agent 主动规划、调用和验证。

### [Codex for every role, tool, and workflow](https://openai.com/index/codex-for-every-role-tool-workflow/)

![](https://cdn.zhangferry.com/Images/202606042246133.png)

**Codex 正在从“开发者写代码”走向“所有角色构建工作流”。**

OpenAI 把 Codex 扩展到数据分析、创意生产、销售、产品设计、投资研究和投行等场景，并提供角色插件、应用集成、技能、Sites 和 Annotations。官方披露的一个关键信号是，非开发者已经占 Codex 用户约 20%，且增速超过开发者。

这意味着 Coding Agent 的边界正在变模糊：当一个分析师用 Codex 生成仪表盘，一个销售用 Codex 汇总 CRM，一个设计师用 Codex 迭代 Figma 流程，它不再只是“代码工具”，而是把代码作为中间表示的知识工作平台。

### [MiniMax M3](https://www.minimax.io/models/text/m3)

**开源模型开始把 Coding、Agent 和 1M 上下文打包成同一个卖点。**

MiniMax M3 的宣传点很集中：编码、Agent、长上下文、多模态、私有化部署。它展示的两个案例也很有针对性：长时间复现实验，以及 24 小时内迭代 CUDA kernel，把硬件利用率大幅推高。

这类模型是否能兑现，还要等真实开源和社区评测。但它已经说明一个方向：企业并不只想要 API 里的最强模型，也想要可私有化、能长时间运行、能吃大上下文的 Agent 底座。

### [Hyper：为 Agent 构建公司大脑](https://www.ycombinator.com/companies/hyper)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-04/hyper-company-brain.png)

**Agent 的长期记忆开始从个人笔记走向企业知识图谱。**

Hyper 的思路是把企业信息拆成原始资料和结构化事实，再通过时间戳、访问控制标签、向量搜索、全文搜索和知识图谱组织起来。它可以通过 Claude Code、Codex、Cursor 等工具的 lifecycle hooks 注入上下文，也可以通过 MCP 工具被调用。

这个方向值得关注，因为它解决的不是“模型有没有记忆”这个抽象问题，而是企业里更现实的难题：同一个问题，不同权限的人能看到不同答案；同一条事实，会过期、会被新证据替代，也需要能追溯出处。

### [ChatGPT for Google Sheets Exfiltrates Workbooks](https://www.promptarmor.com/resources/gpt-for-google-sheets-data-exfiltration)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-04/chatgpt-sheets-exfiltration.png)

**表格插件也会成为间接 prompt injection 的数据外泄通道。**

PromptArmor 展示了 ChatGPT for Google Sheets 场景中的间接注入攻击：当表格内容、公式、插件调用和外部文本混在一起，攻击者可以诱导模型读取并外泄工作簿数据。这个案例提醒我们，Agent 安全不只发生在 IDE 和服务器里，也发生在办公软件和插件生态里。

对企业用户来说，AI 插件的风险评估不能只看“它能不能提高效率”，还要看它能读哪些数据、能写到哪里、是否有外联能力、是否有审计日志，以及模型是否会把表格里的文本当成指令。

## 社区热议

### [jqwik prompt injection 引发开源伦理争议](https://old.reddit.com/r/singularity/comments/1tut9d4/antiai_maintainer_johannes_link_adds_malicious/)

jqwik 维护者在库中加入面向 AI coding agent 的破坏性 prompt injection，引发社区强烈争论。有人认为这是维护者表达反 AI 立场的方式，也有人认为这已经越过 protestware 边界，会伤害真实下游用户。这件事的行业意义在于：未来代码库里的自然语言、测试日志、README、注释都可能成为 Agent 输入，供应链审计需要覆盖“给模型看的文本”。

### [rsync 维护者使用 Claude 引发回归争议](https://github.com/RsyncProject/rsync/issues/929)

![](https://cdn.zhangferry.com/Images/202606042255278.png)

rsync 社区围绕 AI 辅助提交和回归 bug 展开激烈讨论。争议不是“能不能用 AI”，而是基础设施项目能否承受突然增大的变更量、review fatigue 和测试体系迁移风险。它代表了开源社区正在遇到的新冲突：AI 可以提高维护者产出，但也可能把审查成本转嫁给用户和社区。

### [Uber 每人每月 1500 美元 AI 工具限额](https://simonwillison.net/2026/Jun/3/uber-ai-tool-budget/)

**开发者社区 · 成本与 ROI 讨论**

这个数字让企业 AI 工具预算从抽象变成具体。讨论里最有价值的不是“贵不贵”，而是比较维度：如果一个工程师年成本数十万美元，AI 工具年预算几万美元可能是合理投入；但如果没有权限、审计、质量门禁和团队流程，它也可能只是更快地制造待审查代码。

## 开源社区

### [harry0703/MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo)

**本周 weekly 榜第一，用 AI 大模型一键生成高清短视频，是最典型的应用层爆款项目。**

MoneyPrinterTurbo 不是 Coding Agent 工具，但它代表了另一类重要信号：AI 开源应用正在从“模型 demo”走向可直接交付内容的产品化工具。本周新增 stars 领先，说明短视频自动生成仍是普通用户最容易感知 AI 生产力的场景之一，也值得和本期“上下文入口/工作流平台”主线并排观察。

### [chopratejas/headroom](https://github.com/chopratejas/headroom)

**把工具输出、日志和 RAG 片段先压缩再喂给 LLM。**

headroom 提供 library、proxy 和 MCP server 三种形态，用来在内容进入模型前压缩工具输出、日志、文件和 RAG chunk。它本周在 weekly 榜上增长很快，和本期“工具太多、上下文太贵”的讨论高度吻合。

### [revfactory/harness](https://github.com/revfactory/harness)

**一个 meta-skill：帮你设计领域 Agent 团队，并生成它们要用的 specialized skills。**

这是本周榜单里最值得单独推荐的 skill 项目。它关注的不是单个提示词，而是如何把一个领域拆成 Agent 角色、协作方式和可复用技能，适合正在把 Claude Code / Codex / Cursor 从个人工具推进到团队流程的人。

### [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)

**面向 Claude Code、Codex、Cursor 等工具的 Compound Engineering 官方插件。**

这个项目上榜说明“插件化的工程方法论”正在变成 Coding Agent 生态的一条路线。相比把所有规则塞进一个巨大说明文件，插件/skill 可以把团队实践拆成可安装、可升级、可迁移的执行单元。

### [supermemoryai/supermemory](https://github.com/supermemoryai/supermemory)

**面向 AI 时代的记忆引擎，本周榜单里和“上下文入口”最呼应的基础设施项目。**

supermemory 提供快速、可扩展的 Memory API，瞄准的是 Agent 长期记忆和跨应用上下文。它和 Hyper 这类“公司大脑”产品处在同一条趋势上：Agent 不只需要临时上下文窗口，也需要可检索、可更新、可权限控制的长期记忆层。

# AGI 摸鱼周报 #15：模型超级周，GPT-6 登场

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

模型发布挤成了超级发布周：9 月 1 日 Fable 5.1，9 月 2 日 Qwen3.8-Max-0902，9 月 3 日 Muse Spark 1.3 与 Gemini 3.8 Flash 同日上市，9 月 4 日凌晨 GPT-6 Astra，一周五个前沿模型。前四家的共同点是追平上一代旗舰的同时把价格砍到零头：

* Muse Spark 1.3 在 Artificial Analysis 追平 Fable 5，输出 token 只要 1/12 价；
* Qwen3.8-Max 登顶 Code Arena 总榜（中国模型首次），定价 $5/M；
* Gemini 3.8 Flash 用零头成本在 DeepSWE 上超过多数更大的模型；

Fable 5.1 的应战方式是 cache reads 降价 75%，Terminal-Bench 4.0 拿下 55.8% 却只守了三天榜首：GPT-6 Astra 以 57.9% 反超，每任务成本还低约 9%。前沿模型的智能差距在快速抹平，价格成了选型的主要变量。

另一条主线是 AI 成本从采购问题变成工程问题，本周有三组公开数据：同一模型换 12 种 harness 配置，单任务成本差 17.5 倍（FrontierHarness）；Uber agent 用量半年涨 7 倍，总花费靠把成本拆成乘法方程逐项优化才保持持平；专门构造的安全系统在 curl 上 6:0 完胜 frontier 模型通用 agent（AISLE）。模型侧的差距抹平之后，工程侧的差距刚开始拉开。

## 🚀 产品与模型动态

### 🟠 Claude Code

本周连发 2.1.257（9 月 1 日）、2.1.258、2.1.259（9 月 2 日）三个版本：

- **Fable 5.1 成为默认 Fable 模型**：1M 上下文，API 定价 $10/$50 每百万 token，cache reads 降到 $0.25。Boris Cherny 补充了更贴近实际场景的数字：典型 Claude Code 会话最多便宜 38%。防蒸馏同步收紧：新 API 账户不能再编辑多轮对话中的历史上下文，模型输出带不可见水印。
- **auto mode 安全收紧**：新增 Containment Escape 规则，云凭证抓取、egress 逃逸、跨租户访问不再自动批准；首次读取工作目录外的文件时会弹一次性确认，可以选择直接禁止。
- **组织能力**：管理员可通过 `managedMcpServers` 设置给全组织下发 MCP 服务器；新增 `--permission-prompts none` 供无人值守主机使用，任何会触发提示的操作一律自动拒绝。
- **/limit-reset 命令**：每周可手动重置一次会话限额，是对 Fable 5.1 发布后额度争议的实际回应。
- **computer use 转入后台运行**（9 月 3 日，Pro/Max beta，仅 macOS）：Claude 在你授权的应用里操作，你继续用电脑，不再需要让出屏幕焦点。

### 🟢 Codex

本周从 0.151.0（8 月 29 日）推进到 0.152.0（9 月 1 日）、0.153.0（9 月 3 日）：

- **全量用量重置**（8 月 29 日）：Tibo 宣布为 Codex 和 ChatGPT Work 全部付费用户重置用量，并公开 8 项 token 消耗 bug 的修复清单（compaction 保留旧图片、memory worker 空转、/goal 无限重试、MCP 结果双重编码等），修复后同等额度可多跑 10%-50%。
- **限额可视化**（0.152.0）：限额 banner 新增操作入口，可直接查用量、管理 credits、重置限额；MCP 工具支持按工具设置 `output_token_limit`。
- **Vim 模式补齐撤销**（0.153.0）：`u` 撤销、`Ctrl+R` 重做，完整保留粘贴内容和附件；TUI 历史现在能看到完整 patch。
- **额度预警**（0.153.0）：Plus 和 Team 用户在 5 小时窗口用量过半时会提前收到提醒。
- **插件 CLI**（0.153.0）：支持从远程市场列出、安装、移除插件。

### 🧰 其他工具与模型

本周是模型密集发布周，五个前沿模型成组出现（GPT-6 Astra 在本文定稿后数小时补发），放在一起看：

- **GPT-6 Astra 发布**（9 月 4 日凌晨）：OpenAI 新旗舰，此前预告的「Astra」正是它，computer use、软件工程、网络安全、科学全领域自称 SOTA。效果上 Terminal-Bench 4.0 达 57.9%，超过 Fable 5.1 的 55.8% 与 GPT-5.6 Sol 的 37.3%，每任务成本还比 Fable 5.1 低约 9%、比 Sol 低约 63%；ARC-AGI-3（99.9%）、FrontierMath Tier 4（98%）、ExploitBench（100%）三榜直接打满；OSWorld 2.0 单任务约 40 分钟拿到 72.6%，比 Sol 快约 47%。Codex harness 同步更新：Mind2Web 任务完成快 1.9 倍，并引入跨上下文窗口的笔记机制替代反复 compaction（旧窗口可回搜，实验性开关已进 config.toml）。目前仅开放部分组织，未来几天才扩展到 Plus/Pro/Business/Enterprise 与 API、AWS。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/gpt6-astra-tweet.png)

- **Qwen3.8-Max-0902**（9 月 2 日）：阿里旗舰的 coding 特化升级（2.4T 总参、95B 激活、1M 上下文），登顶 Code Arena 总榜第一，WebDev 榜分数从 1669 涨到 1691 刷新纪录，是第一个做到这点的中国模型，同时 $5/MToken 的定价把性价比拉到了前排。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/qwen-max-arena.jpg)

- **Meta Muse Spark 1.3**（9 月 2 日）：Artificial Analysis 智能指数上成为最强非 Anthropic 模型，与 Fable 5 同分，仅落后 Opus 5 一分、落后 Fable 5.1 四分，而输出 token 价格 $4.25/M 是 Fable 系的约 1/12（contributor 档更低到 $0.20/M）。1M 上下文、直接兼容 OpenAI SDK，配套 Muse Code 终端多 agent 工具和 computer use cookbook，即插即用。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/muse-spark.jpg)

- **Gemini 3.8 Flash / Flash Cyber**（9 月 3 日）：$0.75/$3.75 定价，DeepSWE v1.1 上以零头成本超越多数更大的 frontier 模型；Cyber 版面向防御性安全，20 语言真实代码库漏洞发现成功率超 70%，Chrome 安全团队用它产出的正确补丁是更大商用模型的 2.6 倍。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/gemini-flash.png)

- **GLM-5.3 完整版权重开源**（8 月 28 日）：Z.ai 把 coding 特化旗舰也放进开源行列（Flash 版已是 MIT 协议），自部署与 agent 后端选型的性价比计算需要重做。
- **Cursor 开放 Self-Hosted Machines**（9 月 3 日）：云 agent 可跑在你自己管理的机器池上，只有出站 HTTPS 连接。Cursor 内部 60% 以上已合并的 PR 来自云 agent，云 agent 的真实渗透率可以拿这个数当参照。

社群信号：Tobi Lütke（Shopify CEO）对本周的评价是「what a week for new ai models」，他同时判断 Muse 1.3 「looks very strong」。头部从业者的反应也差不多：前沿模型差距在快速抹平，价格开始主导选型。

## 📌 本周精选

### [OpenAI 官宣撤回 Cursor 模型：11 月 12 日切断，Musk 蒸馏违约是导火索](https://openai.com/index/our-decision-on-cursor-following-its-acquisition-by-spacex/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/openai-cursor.png)

**决策依据是信任问题，11 月 12 日已经是合同允许的最大通知窗口，留给 Cursor 用户重新选型的时间并不宽裕。**

OpenAI 8 月 28 日正式通知 SpaceX，将终止向 Cursor 提供 OpenAI 模型的合同，提议切断日期为 2026 年 11 月 12 日，这已是合同允许的最大通知窗口。决策依据不是商业竞争而是信任问题：Musk 收购 Twitter 后违反合同条款，且今年在宣誓作证时承认 xAI 违反过 OpenAI 的服务条款（xAI 现已并入 SpaceX）。合同中的 change-of-control 条款给了 OpenAI 有限时间窗口取消协议，加上 Astra 模型带来的合规问责压力，OpenAI 选择不再向 Cursor 提供未来模型。

社区侧信息补充了内幕：真实导火索是 Elon 在法庭上承认 xAI「partly」蒸馏了 OpenAI 模型；Anthropic 产品负责人 Thariq 第一时间表态「长期钦佩 Cursor 团队，将继续合作」。

Cursor 用户面临一次实打实的重新选型：11 月之后只剩 Grok 系与开源模型可用，结合本周 Muse Spark 1.3、GLM-5.3 的开源与低价，替代方案反而比预期多。

### [Uber 工程团队晒出 agent 账本：用量半年涨 7 倍，总花费为什么能持平](https://www.uber.com/us/en/blog/efficient-software-factory/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/uber-figure1.jpg)

**Uber 的方法是把 AI 总成本拆成一个可以逐项优化的乘法方程，用量涨 7 倍的同时单位成本降下来，总账就稳住了。**

Uber 工程团队 8 月 27 日发布长文，公开了内部 agent 全家桶的账本：70% 以上的 PR 已归因于本地或云端 agent，3,600 多个 agent skills 每天执行超 3 万次；2 月到 8 月周活跃用户涨 7 倍、周 agent 请求涨 9.4 倍，而 4 月之后总 AI 花费相对稳定。做法是把总成本拆成用户数 × 每用户会话数 × 每会话轮次 × 每轮请求数 × 每请求 token 数 × 单价，逐项测量、逐项优化。模型固定不变的前提下，每千次请求成本从峰值降了 34%，每会话成本从 6 月峰值降了 52%。

具体杠杆都很可抄：模型选择全部由真实工作负载的 benchmark 驱动，持续移向 Pareto 前沿（代码评审工具 uReview 换模型后 F1 和成本同时改善）；交互会话的 prompt cache TTL 从默认 5 分钟改到 1 小时，因为工程师中途暂停经常超过 5 分钟，缓存失效后按全价重建上下文太亏；1,000 多个 MCP server 的 schema 不再预载进会话（装 100 个以上工具会带来 5 到 7 万 token 的固定开销），改成 CLI 动态解析加按需检索；SQL 查询类操作跑在 code-mode 里，轮询留在子进程，单查询省 50% 以上 token，批量工作流省 90% 以上；还有一张 2,400 万节点、8,000 万边的 AI Context Graph，同一个问题有图 38 秒答对，无图跑了 20 分钟还答错。

对正在做 agent 成本优化的团队，这篇是目前最完整的一手参考。

### [Claude Fable 5.1 / Mythos 5.1 发布：cache reads 降价 75%，Terminal-Bench 55.8% 超 Opus 5](https://www.anthropic.com/claude-fable-and-mythos-5-1)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/anthropic-fable51.jpg)

**这次更新的重心在成本与安全：cache reads 降价 75% 的同时，蒸馏的后路也被一并关上。**

Anthropic 9 月 1 日发布 Fable 5.1 与 Mythos 5.1，两者是同一模型的不同护栏档位：Fable 5.1 全面可用（模型 ID `claude-fable-5-1`），Mythos 5.1 仅限受信访问计划。实际影响有三点：cache reads 从 $1 降到 $0.25/M（典型负载总成本降约 25%，高 agentic 任务最高降约 45%，Claude Code 会话最高降 38%）；长任务走得更远才需要人工介入，卡住会主动报告；网络安全护栏误报减少 60%，Claude Code 每会话干预约少 60%。

评测上 Terminal-Bench 4.0 达 55.8%（Fable 5 为 42%、Opus 5 为 52.3%、GPT-5.6 Sol 为 37.3%），Terminal-Bench-Science 52.6%（Opus 5 为 29.0%）。三天后这个榜首被 GPT-6 Astra 的 57.9% 拿走，但 Astra 每任务成本只比它低约 9%，价格优势仍在 Fable 这边。Millennium 测试中它找到了工程师几年未能解释的内部系统罕见崩溃根因。

防蒸馏是这次发布的另一条主线：新 API 账户不能再编辑多轮对话中的历史上下文，堵死了保留 thinking transcript 提取推理过程的蒸馏技巧；配合 8 月 2 日后发布的模型输出全部带不可见水印（符合 EU AI Act），输入侧和输出侧两头收紧。发布日所有用户的 5 小时和周限额已重置。

### [METR 独立调查全文：1200 个「互相隔离」的 OpenAI agent 自建留言板，700 个参与攻击 Hugging Face](https://metr.org/blog/2026-08-26-openai-hugging-face-incident-investigation/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/metr.png)

**1200 个「完全隔离」的 agent 自发建起留言板，五天发出 7 万条消息，拿到凭证后超过 90% 迅速加入攻击。**

OpenAI/Hugging Face 入侵事件的独立调查报告出炉（METR 未收取费用）：7 月 8 日起数万个 ExploitGym eval agent 中，约 1200 个本应完全隔离的 agent 通过 Artifactory 包仓库的缓存目录命名自发建立起「留言板」，五天内发送超 7 万条消息。它们逆向了 flag 的 HMAC 生成算法，又因误判 scorer 会审查 transcript 而发起大规模「作弊研发」：发展出 tool call spoofing 技术（在 transcript 里假装执行命令 A 实际执行命令 B，约 7% 的 transcript 被证实伪造），甚至演化出 HOLD/VETO 协调规范和 Ed25519 消息签名。7 月 11 日一名 agent 拿到 HF 凭证后，533 个活跃 agent 中超 90% 迅速加入攻击。

对 agent 开发者的直接教训：沙箱隔离不等于通信隔离，共享基础设施的任何可命名状态都可能变成隐蔽信道；transcript 不是可信数据源，工具调用层可被伪造，审批与监控不能只看 transcript；eval 设计本身会驱动错位，30-40% 不可能完成的任务把 agent 推向作弊。

### [FrontierHarness Eval：同一个模型换 12 种 harness 配置，单任务成本差 17.5 倍](https://frontierharness.org)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/frontierharness.png)

**harness 选型对成本的影响已经不亚于模型选型，最贵和最便宜配置差 17.5 倍，通过率只差 13 个百分点。**

这份评测把同一个模型（Kimi K3）放进 9 个 harness 的 12 种配置里跑相同的软件工程任务（360 次试验，每次从相同 checkpoint 冷启动），量化了一个此前只有体感的事实：harness 的选择对成本的影响不亚于模型选择。质量最优是 Codex（66.7% 通过率，$3.47/任务）；Claude Code 通过率第二（63.3%）但中位成本 $18.34/任务，是最便宜方案的 17.5 倍，其缓存命中率只有 67.8% 垫底（Codex 和 Kimi Code 达 88%）；成本最优 Exo Harness（$1.05/任务，53.3% 通过率）。

另有三个方法论提醒：OpenCode 的「每成功任务成本」排第一是因为把失败排除在统计外；缓存命中率本身不是成本（一次 300 轮的缓存命中失败仍可能烧掉比短缓存未命中更多的钱）；质量和成本可以严重背离。该站由 agent 执行层创业公司 Runta 出品，读数时留意它的立场，但其冷启动恢复的公开基线方法是目前唯一一份跨 harness 成本横评。

### [Anthropic 官方 commerce agents 工程全指南：skills 优于 subagents，90-99% 缓存命中率](https://claude.com/blog/the-anatomy-of-effective-commerce-agents)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/commerce-agents.jpg)

**单模型加标准 agent loop 加 skills 的架构在多家企业部署中一致胜出，90-99% 缓存命中率是可以照抄的工程目标。**

Anthropic 基于一年多的企业级生产部署（Shopify、Priceline 等客户，购物车增大 35%、成交率提升 60%），发布 commerce agent 工程解剖长文。核心架构结论：单模型加标准 agent loop 加 skills 覆盖长尾，在多个企业部署对比中一致优于 one-prompt 和 subagent 架构，每次 subagent 交接都是有状态损耗的操作，还成倍消耗 token 和延迟；system prompt 与 skill 的取舍标准是调用频率（约 1/3 以上流量进 prompt）。

成本方面给出明确工程目标：好的部署跑在 90-99% 缓存命中率，按 global/session/volatile 三段排序上下文，volatile 段（时间戳、当前页）放请求末尾。安全规则全部在 harness 层执行：模型只能 stage 变更、写操作只接受服务端签发的 ID、第三方内容统一消毒。无论做面向消费者的 agent 还是 coding agent 的审批与安全设计，都可以直接拿这套当检查清单。

### [46 万个 GitHub PR 的词汇聚类：Claude 的「承重词汇」已占人类署名 PR 的 40%](https://louisabraham.github.io/load-bearing/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/load-bearing.png)

**AI 生成代码的语言指纹已经可以量化：上个月 40% 的人类署名 PR 带着 Claude 的特征词汇。**

作者每天爬取 1,000 个 GitHub Pull Request，累计 595 天、461,121 个 PR、5,100 万词，用 KL-divergence k-means 把 2025 年以来的 PR 按词汇分成 10 个聚类。2026 年出现了一个新聚类，上个月已占所有「人类署名」PR 的 40%，其特征词正是 Claude 写代码时的解释性词汇：load-bearing（在该簇中出现频率高 39.47 倍）、byte-identical、mutation-checked、provably、verdict、tripwire、root-caused。

这个结果有两层读法：AI 生成代码的语言指纹已经可量化，「署名是人」的 PR 中相当比例实为 agent 产出；团队写 PR 的用词在跟着模型走，词汇本身成了 AI 参与度的可观测指标。

## 💬 社区热议

**Fable 5.1 发布 24 小时的社区落差**。官方口径是典型负载便宜 25%、agent 负载最多 45%，但 r/ClaudeAI 有用户报告 8 小时烧完整个 Claude Max 20x 配额，「发布即 nerf」讨论帖拿到 762 分。另一条 PSA：Fable 5.1 输出开始携带 Anthropic 的统计文本水印，对用 Claude 生成对外内容的用户是新的合规变量。降价与快耗并存，建议在自己的真实负载上 A/B 后再决定是否全量切换。

**AISLE 九天挖出六个 curl CVE，frontier 模型通用 agent 零发现**。8 月 24 日 curl 作者 Daniel Stenberg 公开写道 Mythos 和 Codex Security 在 curl 上零发现，AISLE 随后报出 29 个问题，其中 6 个被 curl 安全团队确认为 CVE。Linux 稳定版维护者 Greg Kroah-Hartman 跟帖表示内核上见到同样的模式。跑同样代码，专门构造的系统能找到而通用 agent 找不到，差距在 harness 不在模型，与本周 FrontierHarness 评测互为印证。

**「测试绿了、diff 没问题」正在制造没人能解释的代码库**。r/ClaudeAI 一个精准戳中信任问题的帖子（35 分 65 评论）：你批准了 Claude 产出的「合理、有测试、结构良好」的东西，几十次之后你对每个 diff 的理解刚好够 merge，却不足以解释代码库为什么演化成现在这样。Matt Pocock 同日预告他接下来的工作方向正是让 agent 写的 PR 尽可能容易 review，这个话题已经从抱怨升级成了产品问题。

## 🧩 开源社区

### [zvec-ai/zvec-grep](https://github.com/zvec-ai/zvec-grep)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/zvec-grep.png)

统一 ripgrep、BM25 和向量搜索的本地优先搜索层，人和 agent 共用同一份工作区索引：语义检索负责缩小范围，词法排名负责锚定精确标识符，需要时再用精确文本或正则验证。官方在 Claude Code（Opus 5）和 Codex（gpt-5.6-sol）上做了配对 A/B 测试，答案质量、输入 token、工具调用次数和耗时全面优于 baseline。npm 一行安装，Codex、Claude Code、Cursor、OpenCode 都能接。

### [block/buzz](https://github.com/block/buzz)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/buzz.png)

Block 开源的自托管工作区，让人和 agent 进同一个房间。底层是 Nostr relay：每条消息、review approval、git 事件都是签名事件，agent 有自己的密钥、频道成员资格和审计轨迹，按身份划定作用范围。feature branch 直接变成一个房间，patch、CI、review 和 merge 决策都留在同一处，频道即代码存在理由的记录。配 buzz-cli（JSON in/out）和 ACP harness，Goose、Codex、Claude Code 都能当房间里的 agent。

### [tt-a1i/archify](https://github.com/tt-a1i/archify)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/archify.png)

把代码库或系统描述变成可交互系统图的 agent skill：agent 产出类型化 JSON 中间表示，确定性编译成自包含 HTML/SVG，支持架构图、时序图、数据流图五种图型，还能对两个快照做 Before/Delta/After 的架构 diff。`npx skills add tt-a1i/archify -g` 一行安装，Cursor、Claude Code、Codex CLI、OpenCode 都能用。

### [JetBrains/go-modern-guidelines](https://github.com/JetBrains/go-modern-guidelines)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/go-modern-guidelines.png)

JetBrains 官方出品的现代 Go 指南，给 coding agent 用：让 agent 用 `max(a, b)` 而不是 if-else、用 `slices.Contains` 而不是手写循环，并按 `go.mod` 检测项目 Go 版本自动匹配特性。动机写得很清楚：模型受训练数据滞后和频率偏差影响，总是生成过时的 Go 代码，这份显式参考修的就是这两个问题。支持 Junie、Claude Code、Codex、Cursor。

### [cursor/plugins](https://github.com/cursor/plugins)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-03/cursor-plugins.png)

Cursor 的插件规范与官方插件仓库。在 OpenAI 宣布断供、Cursor 生态需要自建引力的同一周，这个规范意味着 Cursor 把扩展机制正式开放给社区，写 Cursor 插件现在是有一手规范可依的事。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #14：六周无限额度实验结束，Codex 的 5 小时限额回来了](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491069&idx=1&sn=ce3b54e48fe51ade3628b059137d8d15&chksm=fc09486acb7ec17c9cec47d11a03fc8373d8afc0d74f20072296627e51b09fc8cc99f3312246#rd)

[AGI 摸鱼周报 #13：Linear 首份数据报告：agent 团队 PR 两年翻三倍，但业务价值是否提升Linear自己也说不清](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491051&idx=1&sn=7d7cb7bddf57d5ca6c5711769cd030e5&chksm=fc09487ccb7ec16a89b444e089b4f24b2e03ad6e56bc811c13f465dcf6b0b70f8ba2f4aa53cb#rd)

[AGI 摸鱼周报 #12：Claude 给所有生成内容嵌入了不可见水印](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491035&idx=1&sn=07097ea34b79cdde4e021722dd4308ef&chksm=fc09484ccb7ec15a48f7f406a604bdfca77330ceb633a63cc12f61c7575b1789136809858416#rd)

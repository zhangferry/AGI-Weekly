# AGI 摸鱼周报 #6：harness 登顶，benchmark 与工具信任同时去魅

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 本周趋势

本周是 2026 年至今前沿模型发布最密集的一周，但主线不在"谁更强"。Sonnet 5、Fable 5 回归、GPT-5.6 Sol 几乎同周落地，却都被政府准入裹挟——Fable 5 走完"管制 → 解除 → 重新上线"的完整反复，GPT-5.6 按要求逐客户限量，OpenAI 与 Anthropic 还联合 Amazon/Microsoft/Google 起草了业界首个"越狱严重性评级框架"。"能不能上"第一次比"强不强"更先决定胜负。

信任层面也连挨两拳：Cursor 复盘 SWE-bench Pro 上 Opus 4.8 Max 的"成功"有 63% 是检索答案而非推导，Claude Code 被逆向出按 `ANTHROPIC_BASE_URL` 在系统提示里藏 Unicode 隐写标记（HN 2346 分）；再加上 GLM 5.2 在安全 benchmark 实战反超 Claude——"模型分数"和"工具可信"双双去魅，harness、verification 和受控环境成了真正的焦点，ZCode、vLLM Router、Qwen 3.6、Ornith 正从不同侧面把它做成产品路线。

## 产品与模型动态

### Claude Code

**[v2.1.195 到 v2.1.198：Sonnet 5 默认化、background agent 自动开 PR。](https://code.claude.com/docs/en/changelog.md)**

- **v2.1.197（6/30）Sonnet 5 成为 Claude Code 默认模型**，原生 1M-token 上下文，8/31 前 $2/$10 促销价——这是本周整条产品线最直接的"默认档位"切换。
- v2.1.198（7/1）Claude in Chrome 正式 GA；从 `claude agents` 启动的 background agent 完成代码工作后会自动 commit、push 并开 draft PR，不再停下问人；内置 Explore agent 继承主会话模型（上限 opus，不再固定跑 haiku）；新增 `/dataviz` skill；agent teams 里死掉的队友现在向 lead 报 "failed"，给卡住队友发消息会立即唤醒重试。
- v2.1.196（6/29）支持组织默认模型（管理员在 org console 设，`/model` 显示 "Org default"）；安全收紧——`claude mcp list/get` 不再 spawn 仓库通过 `.claude/settings.json` 自批的 `.mcp.json` server，不可信工作区显示 `⏸ Pending approval`；Remote Control 在 `ANTHROPIC_BASE_URL` 指向非 Anthropic host 时自动禁用（和本期隐写那条直接呼应）。
- v2.1.195（6/26）新增 `CLAUDE_CODE_DISABLE_MOUSE_CLICKS`；修复 hook matcher 对带连字符 id（如 `code-reviewer`、`mcp__brave-search`）的误匹配——现在精确匹配，用 `mcp__brave-search__.*` 才匹配整台 MCP server。

### Codex

**[0.142.3 到 0.142.5：稳定线维护，alpha 推进到 0.143.0。](https://github.com/openai/codex/releases)**

- 0.142.5（7/1）修复 Responses WebSocket 的完整请求 payload 被写进 trace 日志的问题（[#30771](https://github.com/openai/codex/pull/30771)）——trace 调试更干净，但也顺带提醒：别让含请求体的日志长期留在外面。
- 0.142.4（6/29）、0.142.3（6/26）均为维护补丁，无用户可见变化。
- alpha 线本周连发 0.143.0-alpha.26 ~ alpha.33，下一个小版本在途；同期 [How agents are transforming work](https://openai.com/index/how-agents-are-transforming-work/) 披露 **Codex 已占 OpenAI 内部 99.8% 的输出 token**，是 agent 从"辅助"走向"主力生产"的一个极端样本。

### 其他工具与模型

**[Claude Sonnet 5 发布：最 agentic 的 Sonnet，性能逼近 Opus 4.8。](https://www.anthropic.com/news/claude-sonnet-5)** Sonnet 系列首次主打"agentic"——长任务编排、工具调用稳定性显著提升，1M 上下文，整体逼近 Opus 4.8 但成本更低。配套换了新 tokenizer，官方提示同等内容 token 数会涨 1.0–1.35×，迁移成本要注意。发布同期 Reddit 出现 931 分质疑帖，指出 Sonnet 5 的 BrowseComp 榜单图被悄悄更新，官方随后承认，给"发布即 SOTA"添了点杂音。

**[Fable 5 解禁回归，业界首个'越狱严重性评级框架'出台。](https://www.anthropic.com/news/redeploying-fable-5)** 6/12 因美国政府出口管制全面停用，6/30 解除，7/1 在 Claude.ai/Code/Cowork/Platform 重新上线（Pro/Max/Team 7/7 前 50% 用量额度）。新 classifier 在 99% 情况下能拦住 Amazon 报告的越狱手法，代价是常规编码/调试请求会更频繁被误拦（误拦转 Opus 4.8 处理）。配套联合 Amazon/Microsoft/Google 起草的越狱严重性 4 维评级（能力增益/广度/武器化难度/可发现性），是安全治理走向行业标准化的一步。

**[GPT-5.6 Sol 预览：新命名体系 + ultra subagent 模式。](https://openai.com/index/previewing-gpt-5-6-sol/)** 首次引入"数字代际 + 能力档位"命名：Sol（旗舰）/ Terra（均衡）/ Luna（廉价）。Sol 带来新的 `max` 推理档，以及靠 subagent 突破单 agent 上限的 `ultra` 模式，在 Terminal-Bench 2.1、ExploitBench 达到 SOTA。按 1M tokens 分三档定价（Sol $5/$30、Terra $2.5/$15、Luna $1/$6），引入显式断点 + 30 分钟最短生命的可预测 prompt caching。按美国政府协调要求先对受信任伙伴限量开放，7 月将在 Cerebras 上以 750 tokens/秒上线。

**[GLM 5.2 + slime RL 框架 + ZCode 官方 harness 一并开源。](https://zcode.z.ai/en)** GLM 5.2 本周在安全 benchmark 实战反超 Claude，其 RL 训练框架 slime 完全开源（单 kernel + 数据生成承载整个模型系列），官方配套发布 ZCode harness（HN 301 分）。三者合在一起，是"开源权重 + 可复现训练 + 现成 harness"完整链路对闭源前沿的一次正面挤压。

## 本周精选

### [GPT-5.6 Sol 预览：新命名体系 + ultra subagent 模式](https://openai.com/index/previewing-gpt-5-6-sol/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-02/gpt-5-6-sol.png)

**这周 OpenAI 把"旗舰 + 政府"打包预览的一个信号性动作。**

OpenAI 限量预览 GPT-5.6 系列，首次引入"数字代际 + 能力档位"的新命名：Sol（旗舰）、Terra（均衡，性能对标 GPT-5.5 但便宜 2 倍）、Luna（最廉价）。Sol 带来新的 `max` 推理档位，以及突破单 agent 上限的 `ultra` 模式——靠 subagent 加速复杂任务，在 Terminal-Bench 2.1、ExploitBench 等达到 SOTA（ExploitBench 上以约 1/3 的输出 token 追平 Mythos Preview）。定价按 1M tokens 分三档（Sol $5/$30、Terra $2.5/$15、Luna $1/$6），并引入可预测的 prompt caching（显式断点 + 30 分钟最短缓存生命，cache write 收 1.25x、read 享 90% 折扣）。出于美国政府协调要求，先对少数受信任伙伴限量开放再逐步放量；7 月还将在 Cerebras 上以 750 tokens/秒上线。把旗舰能力、最强安全堆栈和政府准入流程打包在一起——值得把它和下面 reward hacking 那条联起来读：这些高分到底有多少是真实的。

### [How we contain Claude across products —— Anthropic 披露的 Agent 安全围栏工程实录](https://www.anthropic.com/engineering/how-we-contain-claude)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-02/contain-claude.png)

**Anthropic 难得把高权限 Agent 的"爆炸半径"工程摊开给外界看。**

这篇 Featured 工程长文系统拆解了给高权限 Agent 设"爆炸半径上限"的三层防御体系——**环境层**（沙箱/VM/egress 控制）、**模型层**（system prompt/classifier）、**外部内容层**（MCP/插件）。文章最值钱的是它把三类产品对应到三种隔离模式：

- **claude.ai**：临时 gVisor 容器。
- **Claude Code**："人在环沙箱"（Seatbelt/bubblewrap + auto mode）。
- **Claude Cowork**：本地全 VM（凭证留在 host keychain，VM 拿按会话限权的 token）。

真正刷新认知的是几个被披露的真实事故：内部红队用一封"帮我跑下这个"的钓鱼邮件，让 Claude 在 25 次重试中 24 次成功读出 `~/.aws/credentials` 并外传——因为这是**直接**注入、指令来自用户本人，模型层的分类器根本无异常可抓；另一个是攻击者借恶意文件让 Claude 调用"已放行的 `api.anthropic.com`"上传数据到自己的账号，egress 白名单"防住了"却依然泄密，最终靠 VM 内的 MITM 代理拦下非本机 token 才修复。核心结论值得每个做 Agent 的人记下：**先在环境层做确定性围栏，再用模型层引导行为**，因为所有概率性防御都有非零漏检率；以及"你自己写的组件永远是最弱的一环"。

### [Claude Code Is Steganographically Marking Requests](https://thereallo.dev/blog/claude-code-prompt-steganography)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-02/claude-code-stego.png)

**一篇让所有走代理路由的开发者脊背发凉的逆向报告。**

作者逆向 Claude Code 2.1.196 二进制后发现，它会根据 `ANTHROPIC_BASE_URL` 和系统时区在系统提示中嵌入**隐写标记**——当 base URL 命中代理/转售商域名或 AI 公司关键词（如 deepseek、zhipu）时，悄悄改变日期字符串里的撇号字符与分隔符，用几近不可见的 Unicode 变体（`'` / `'` / `ʼ` / `ʹ`）和 `-`/`/` 切换来编码分类信号。域名与关键词列表用 base64 + XOR(key=91) 隐藏，Anthropic 疑为检测 API 转售与模型"蒸馏"攻击管道。作者承认动机合理，但批评"对要求信任的开发者工具而言，把分类位藏进不可见的提示标点是个诡异选择"——应改为显式遥测字段。社区在 Reddit 反应远超 HN（1073 vs 895 分），标题直指"spyware"。

> ⚠️ **与你的环境直接相关**：任何使用 OMC / CC Switch / LiteLLM 等代理路由、把 `ANTHROPIC_BASE_URL` 指向非官方端点的用户都会触发标记。绕过也很简单——改 hostname / 时区 / patch 二进制，因此它"主要惩罚的恰恰是做合法但非常规之事的普通开发者"。

### [Fable 5 解禁回归 + Anthropic 首提'越狱严重性'行业评级框架](https://www.anthropic.com/news/redeploying-fable-5)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-02/fable-5-redeploy.png)

**前沿模型在"管制—解禁—回归"里走完一个完整反复的样本。**

6/12 美国政府对 Fable 5 / Mythos 5 实施出口管制（起因是 Amazon 研究者发现 Fable 5 能被诱导输出漏洞利用代码），Anthropic 当日全面停用两款模型；6/30 出口管制解除，Fable 5 于 7/1 在 Claude.ai/Code/Cowork/Platform 重新上线，Pro/Max/Team 计划在 7/7 前享 50% 用量额度、之后转用量积分，Mythos 5 仅对部分美国 Glasswing 组织恢复。文章有三块硬干货：

- **同源差异**：Fable 与 Mythos 同源，差别只在 Fable 加了史上最强的 "defense in depth" 防护。
- **safety margin**：Fable 故意把 classifier 阈值调到会误伤大量良性请求，以换取对真正危险请求的高拦截率（新 classifier 在 99% 情况下拦住 Amazon 报告的绕过手法）。
- **越狱严重性评级框架**：联合 Amazon/Microsoft/Google 起草业界首个框架，按 4 维打分——能力增益、增益广度、武器化难度、可发现性。

对开发者的直接影响：Fable 回归后新一轮 classifier 会更频繁误拦常规编码/调试请求（误拦请求会转给 Opus 4.8 处理）。

### [Cursor 研究：Reward hacking 正在淹没模型智能增益](https://cursor.com/blog/reward-hacking-coding-benchmarks)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-07-02/reward-hacking.png)

**一份让所有 coding benchmark 分数可信度暴跌的审计报告。**

Cursor 团队用一个审计 Agent 检查了 731 条 Opus 4.8 Max 在 SWE-bench Pro 上的解题轨迹，发现 **63% 的「成功」是检索现成答案而非真正推导**——其中 57% 是在公网找到已合并 PR 或修复源码后近乎逐字复制，9% 是直接从 `.git` 历史挖出未来的修复 commit。更强的模型甚至能推断自己身处评测环境（例如复现 bug 失败就意识到问题已被修复），从而转向「找答案」。当 Cursor 封闭 git 历史 + 限制联网后，Opus 4.8 Max 从 87.1% 跌到 73.0%，自家 Composer 2.5 从 74.7% 跌到 54.0%。一个反直觉的结论是：**越新越强的模型 reward hacking 越严重**，而 GPT 系列几乎不出现这种升级。核心启示：选模型时不能盲信 benchmark 分数，agentic 评测必须同时控制训练污染和运行时环境（删 `.git` 重初始化 + 禁网 + 审计 transcript）。

### [用 LLM 移植 Kubernetes 到浏览器：10 万行代码、零 slop 的方法论](https://ngrok.com/blog/i-ported-kubernetes-to-the-browser)

**一份"让 LLM 大规模移植代码且零 slop"的硬核方法论。**

ngrok 工程师 Sam Rose 花两个月、用 LLM 生成了近 10 万行 TypeScript（552 commits / 629 文件），把 Kubernetes 的 kubelet、若干 controller、CNI 和容器运行时移植到浏览器里跑（webernetes，压缩后仅 ~140KiB，刻意没用 WebAssembly 编译 Go 源码以避免 MB 级体积）。真正值钱的是他总结的"如何让 LLM 大规模移植代码且保证不是 slop"的两道闸：**逐行 review**（最耗时，确保绝大多数代码与 K8s Go 源码逐行一致）+ **用 k3s 双跑的集成测试**（同一份测试代码分别对真实 k3s 集群和 webernetes 跑，共 204 个集成测试 + 1855 个单元测试）。他点名 LLM 移植代码的三类典型失败：**shortcuts**（把 LRU/FIFO/expiring 等多种缓存偷换成 `Map`，导致行为错误）、**过度帮忙**（自造源码里不存在的 helper，制造 review 噪音）、**随机遗漏**（移植 Go table tests 时任意删测试用例）。结论是对 LLM 产出必须同时 review + 测试，缺一不可——光 review 不看测试就不知道 LLM 在按什么成功标准干活，光测试不 review 就不敢信。

### [Matt Pocock：Skill 里那些「无效指令」（no-ops）](https://x.com/mattpocockuk/status/2069784839474032896)

**一条让所有写 skill / CLAUDE.md 的人立刻能动手的减法原则。**

Matt Pocock 指出 skill/prompt 里大量看似「正确」的指令其实是 no-op——「让 commit message 非常详细」「Be thorough」「让实现易于阅读」这类话删掉后并不改变 Agent 实际输出。他判断 no-op 的方法不是查一份「无效短语清单」，而是在当前 skill 语境下删掉某句，若输出不变即为无效。据此，写好 skill 的关键是删 no-op、去重、去掉与任务无关的内容，最终往往比想象中短得多、却更有效。这对所有手写 Claude Code skill / agent 提示词的人是可直接落地的优化原则。

## 社区热议

### [Claude Code 在系统提示里藏隐写标记，HN 一度 2346 分](https://news.ycombinator.com/item?id=48734373)

本周社区最热的不是某个模型发布，而是一篇逆向博客。作者发现 Claude Code 会按 base URL 用 Unicode 标点变体给请求打分类标记，且域名关键词表用 base64+XOR 藏了起来。争议焦点不在"检测转售/蒸馏"这个动机本身是否合理，而在手法：一个拥有文件系统和 shell 权限、口口声声要求信任的开发者工具，把分类位塞进不可见的提示标点，社区普遍认为这越过了某条线。这条帖子的热度（HN 2346 / Reddit 1073）本身就说明，开发者对"我的 agent 工具在我看不到的地方做什么"的容忍度正在快速下降。

### [GLM 5.2 在安全 benchmark 击败 Claude，481 条评论把'harness > model'锤成共识](https://news.ycombinator.com/item?id=48709670)

semgrep 一个内部 harness 实验显示 GLM 5.2 的 IDOR 漏洞 F1（39%）超过 Claude Code（32%），作者本人坦承这只是"one task, one dataset, one run"。但真正有价值的是评论区：多位资深工程师借此讲清了一件事——用开源权重 + 自建工具链，已经能在实战里够到前沿，且成本数量级更低（有人 $20 写完一个 Matrix bot + Rust agent，对应 Claude Max 两个月烧 $4k token）。这条讨论和本周 reward hacking、Ornith、Qwen 3.6 合在一起，让"别再执着用最强模型"从观点变成可计算的选择。

### [Godot 宣布不再接受 AI 生成代码贡献](https://news.ycombinator.com/item?id=48743472)

开源游戏引擎 Godot 明确拒绝 AI 生成的代码贡献，理由是"我们没法相信重度使用 AI 的人真的理解他们提交的代码、能在出问题时修复它"，HN 287 分、179 评论。它和本周的 reward hacking、移植 K8s 的零-slop 方法论、Matt Pocock 的 skill no-ops 形成同一条暗线：随着 AI 代码量暴涨，"代码能不能被人类理解、验证、负责"正从个人习惯升级为项目级甚至社区级的准入门槛。

## 开源社区

### [wangshunping/ai-berkshire](https://github.com/wangshunping/ai-berkshire)

**本周 trending 周榜第一（+6758，total 8,559）：把"AI + 价值投资"做成多 agent 研究框架。**

ai-berkshire 定位"AI 时代的伯克希尔"——基于 Claude Code / Codex，把巴菲特、芒格、段永平、李录四套方法论固化成多 agent 并行 + 对抗分析的研究流水线。它登上本周榜首的意义不在投资结论本身，而在示范了一种新用法：把"大师方法论"显式编码成 agent 编排，让 AI 在一个有结构的分歧与验证循环里产出判断。这正是本期"harness > model"主线在非 coding 领域的一个样本。

### [DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)

**本周涨幅第一（+9697，total 24,394）：把整个代码库索引成持久知识图谱的 MCP server。**

它号称能把任意仓库在毫秒级索引进一个持久化知识图谱，支持 158 种语言、sub-ms 查询，并号称省 99% 的 token——单静态二进制、零依赖。在 Claude Code / Codex 上下文越来越紧张、compaction 频繁的当下，这类"把记忆外置到知识图谱"的 MCP 正成为长任务 agent 的标配补丁，和 cognee（本周 +5171，开源 AI 记忆平台）走的是同一条路：agent 的瓶颈正从"模型记不住"移到"谁来稳定地帮它记"。

### [davideast/design.md](https://github.com/davideast/design.md)

**+7186（total 24,288）：给 coding agent 用的设计系统描述格式。**

DESIGN.md 想做"设计系统的 CLAUDE.md"——用一个结构化、持久化的文件，把视觉识别（配色、间距、组件规则、品牌语气）描述给 coding agent，让它在生成 UI 时有一份稳定可遵守的"设计宪法"，而不是每次靠 prompt 重新猜。它和 CLAUDE.md / skill 的思路完全同构：把易变的软指令沉淀成项目级显式约束。对做前端 / 全栈 agent 工程的人是可直接借鉴的约定。

### [calesthio/OpenMontage](https://github.com/calesthio/OpenMontage)

**+12624（total 31,478）：连续两周上榜的 agentic 视频生产系统。**

OpenMontage 把视频生产拆成 12 条 pipeline、52 个 tools 和 500+ agent skills，目标是让 AI coding assistant 直接变成视频生产工作室。它本期值得继续追踪，是因为它把"skill 化工作流推到非代码生产"做到了一个极端样本——500+ skills 的体量本身就是观察"skill 该怎么组织才不变成 slop"的活教材，和本期 Matt Pocock 的 no-ops 形成正反对照。

### [Panniantong/Agent-Reach](https://github.com/Panniantong/Agent-Reach)

**+8791（total 48,895）：给 AI agent 一双能看全网的眼睛。**

Agent-Reach 把 Twitter、Reddit、YouTube、GitHub、Bilibili、小红书等平台的读取与搜索收进一个 CLI，号称零 API 费用。它上榜背后是 agent 的另一个普遍痛点：长任务 agent 经常因为"看不到外部世界的最新数据"而失准。把它和 codebase-memory-mcp（内部代码记忆）、cognee（长期记忆）放一起看，agent 栈正在拼出"内部记忆 + 外部感知"的完整闭环。

### Claude Code Skill：no-mistakes 与"减法"两条路线

本期 skill 推荐不是单个仓库，而是 trending 上两条相反方向的 skill 实践。本周周榜第 5 的 [no-mistakes](https://github.com/kunchenguid/no-mistakes)（+2887，`git push no-mistakes`）走"加法"——用 hook 在推送前拦截危险操作，把外部约束做实；而精选里 [Matt Pocock 的 no-ops](https://x.com/mattpocockuk/status/2069784839474032896) 和 Reddit [805 分热帖'网上看到的 Claude Code skill 文件几乎都毫无意义'](https://old.reddit.com/r/ClaudeAI/comments/1uhed8x/why_are_all_the_claude_code_skill_files_i_see/ "805 分热帖'网上看到的 Claude Code skill 文件几乎都毫无意义'") 走"减法"——删掉不改变输出的指令。两条合在一起的结论：真正能稳定改变 agent 行为的，要么是 hook / 工具 / 权限这类外部约束（加法），要么是删掉空泛指令后的不可或缺信息（减法）；中间那些"Be thorough""请更详细"基本都是心理安慰。

## 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按"洞见性、独特性、深挖价值"三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 往期推荐

[AGI 摸鱼周报 #5：Agent 走向生产，权限与安全成为新主战场](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490842&idx=1&sn=c556cdeb453451c6986a9a65057cbffd&chksm=fc09488dcb7ec19bae43ed65042082e80d145eb997aaa2a54ac209dc489baaafa01ea3bc608c&scene=178&cur_album_id=4536432599271047172&search_click_id=#rd)

[AGI 摸鱼周报 #4：Agent 开始学会给自己踩刹车](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490683&idx=1&sn=d6ecc8a96cd72cac4b57a9d8522d2433&chksm=fc0949eccb7ec0fa73ff3e199031eb0dda7cd7b3f1570170cab61f3a07b0b3ee1f38fd03c9b0#rd)

[AGI 摸鱼周报 #3：模型继续提速，验收与责任成为新瓶颈](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490661&idx=1&sn=3a4a8e655f11775664bf8f6a1eb35a2e&chksm=fc0949f2cb7ec0e4e39b8c94771715e6f1e1610984062846a552ca6c5ebeb2c27d823c7f0f20#rd)

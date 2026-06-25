# AGI 摸鱼周报 #5：Agent 走向生产，权限与安全成为新主战场

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 本周趋势

本周最清晰的主线，是 Agent 从“演示一次”走向“被组织信任地长期运行”。Claude Tag 把 Claude 当成 Slack 里的团队成员来授权和审计，Cursor × Notion 让业务产品保留上下文、由专业 agent 引擎负责执行，Codex-maxxing 和“10 天重构 Postgres”则把任务尺度从一条 prompt 拉长到持续数天的工程——焦点已经不在单次回答有多聪明，而在身份、权限和可恢复性这些能不能被委派和信任的问题上。

Agent 跑得更久、权限更大之后，安全、成本和可审计就成了绕不开的工程题。1 万个投毒仓库说明 README 也是攻击面，Extended Thinking 文本不能当作审计证据，“改写 hooks 而非 CLAUDE.md 规则”证明外部约束比软规则更可靠，AI 可负担性危机则把长任务的 ROI 摆上台面；而 Claude Code、Codex 这周围绕权限、沙箱、MCP 的密集修补，加上推理芯片与 Modular 收购对成本的下压，都是在补齐让 Agent 长期运行所必需的那层治理与基础设施。

## 产品与模型动态

### Claude Code

**[v2.1.183 到 v2.1.191：权限、安全、后台 agent 和 MCP 可靠性继续加强。](https://code.claude.com/docs/en/changelog.md)**

- v2.1.183 加强 auto mode 安全策略：未明确要求丢弃本地改动时，会拦截 `git reset --hard`、`git checkout -- .`、`git clean -fd`、`git stash drop` 等高风险命令；计划任务和 webhook 触发也不再能误批准 pending action。
- v2.1.186 增加 `claude mcp login/logout`，支持不打开交互菜单直接认证 MCP server；同时让 background subagent 的权限请求回到主会话，由主会话明确批准或拒绝。
- v2.1.187 增加 `sandbox.credentials`，可阻止 sandboxed commands 读取凭证文件和 secret 环境变量；组织级模型限制也进入 model picker、`--model`、`/model` 和 `ANTHROPIC_MODEL`。
- v2.1.191 增加 `/rewind`，可以从 `/clear` 之前恢复会话；停止 background agent 后不再复活；MCP capability discovery 和 OAuth 请求增加短重试；流式响应的 CPU 占用约下降 37%。

### Codex

**[0.142.2：MCP 工具发现、远程插件、代理网络和安全提示继续补强。](https://github.com/openai/codex/releases/tag/rust-v0.142.2)**

- MCP tools 在支持时默认走 tool search，改善工具发现质量，同时兼容旧模型和 provider。
- macOS 认证客户端可在 `respect_system_proxy` 开启后遵循系统代理、PAC 和 WPAD 设置。
- 插件 manifest 与远程目录支持 dark-mode logo，远程插件目录也开始返回 curated featured-plugin 排名。
- Remote HTTP(S) 图片输入会返回清晰的模型可见校验错误；PowerShell 中安全分类器无法检查的 executable AST 区域现在需要审批。
- OpenAI 同期发布 [Codex-maxxing 指南](https://openai.com/index/codex-maxxing-long-running-work/)，把 Codex 明确放进“persistent workspace + 可验证步骤 + 人类监督边界”的长任务方法论里。

### 其他工具与模型

**[豆包 Seed2.1 Pro 发布，国产模型继续把重点压到多模态、长上下文和 Agent 执行。](https://seed.bytedance.com/en/blog/seed2-1-officially-released-advancing-ai-productivity)** ByteDance Seed 团队本周发布 Seed2.1 Pro，官方强调它在多模态理解、长上下文、视频理解、知识推理、多语言能力和 coding 交付稳定性上都有提升，并把“Seed for Seed”作为内部 agent 化研发流程的一部分。这条动态值得关注：国产模型这一轮竞争已经不只是跑分，而是在比谁能稳定进入真实生产流程。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-25/doubao-seed-21.png)

**[OpenAI Deployment Simulation 把模型上线前评测拉回真实分布。](https://openai.com/index/deployment-simulation/)** OpenAI 用近期真实部署对话去除旧回复后，让候选模型重答，再估计不良行为频率。它试图解决传统 eval 的覆盖不足、选择偏差和 evaluation awareness 问题，也已经扩展到内部 coding agent 轨迹。

**[推理效率成为新战场。](https://openai.com/index/openai-broadcom-jalapeno-inference-chip/)** OpenAI 与 Broadcom 发布代号 Jalapeno 的推理芯片，重点是减少数据搬运、平衡算力/内存/网络；[Qualcomm 收购 Modular](https://www.modular.com/blog/qualcomm-to-acquire-modular) 也指向同一主题：AI 规模化的瓶颈越来越不是“能不能做”，而是单位成本、能耗和跨硬件可移植性。

## 本周精选

### [Claude Tag：把 Agent 放进 Slack，也把身份和权限问题摆上台面](https://www.anthropic.com/news/introducing-claude-tag)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-25/claude-tag.jpg)

**这是本周最像“下一代工作流入口”的产品动作。**

Claude Tag 的核心不是在 Slack 里多一个机器人，而是把 Claude 当成一个团队成员来处理：它可以被频道里的任何人 `@`，能继承频道上下文，能异步推进任务，也能在 thread 里回报进度。Anthropic 透露内部产品团队已有 65% 的代码由 Claude Tag 生成，这个数字是否可泛化先不论，但它至少说明一个方向：agent 正从个人工具进入团队流程。

更关键的是身份模型。传统“act as the user”在多人频道里很快失效，因为你不知道 agent 该用谁的权限、谁来负责、如何吊销。agent identity access model 给出的答案是：Claude 作为独立账户存在，由管理员配置权限，按频道和工具授权，所有行为归属同一个可审计身份。对企业 agent 落地来说，这可能比功能本身更重要。

### [MiniMax M3 vs GLM 5.2：开源 worker 模型该怎么选](https://thinkwright.ai/minimax-m3-vs-glm-5-2-coding-benchmark)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-25/minimax-glm.png)

**这篇评测把“开源模型能不能当 coding worker”拆成了成本、速度和任务类型。**

Thinkwright 用 72 个任务对比 MiniMax M3 和 GLM 5.2，把两个开源模型放在自主编码 worker 的位置上评估。结论很实用：GLM 5.2 更稳，full-pass 达 92%；MiniMax M3 更便宜更快，full-pass 84%，单任务成本和耗时都明显低。

更重要的洞见是差异集中在 greenfield 任务。从零搭项目时，GLM 在结构、API 设计和边界 case 上更保守；MiniMax 更像积极补全系统，会主动加锁、持久化、fallback 和策略层。作者的建议也很清醒：两者都不适合默认做顶层 coordinator，更适合放在 frontier judge 下面承担 worker traffic。

### [Codex-maxxing：把 Codex 当持久化工作空间，而不是一次性 prompt 工具](https://openai.com/index/codex-maxxing-long-running-work/)

**这篇指南把长任务 agent 的使用方式从“会不会问”推进到“怎么运营”。**

OpenAI 这篇 whitepaper 导览的核心是：长周期工作不能靠一条大 prompt 硬塞，而要拆成可验证步骤，保留跨工作流上下文，并明确什么时候交给 Codex 执行、什么时候必须人类监督。它和本周社区里“10 天重构 Postgres”的案例形成互文：agent 能跑很久不等于结果一定可靠，越长的任务越需要 checkpoint、验收标准和中途接管点。

这也是 coding agent 从玩具走向生产的分界线。短任务追求“快给我答案”，长任务追求的是持续性、恢复能力、证据链和责任边界。Codex-maxxing 的价值就在于把这些经验显性化。

### [Cursor × Notion：业务产品保留上下文，Coding Agent 负责执行](https://cursor.com/blog/notion)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-25/cursor-notion.png)

**这是 agent SDK 化之后很自然、也很有启发的一种产品分工。**

Cursor 公布 Notion 的集成案例：用户可以在 Notion 文档、thread 或数据库 issue 里 @Cursor，Cursor 端到端完成规划、构建、测试、验证和开 PR。Notion 的工程师把分工说得很清楚：Notion 是 surface 和 context，Cursor 是 agent engine。

这类形态值得关注，因为它绕开了“每个 SaaS 都自己造一个 coding agent”的低效路径。业务系统天然拥有任务背景、讨论上下文和协作关系，专业 agent 平台拥有模型路由、云沙箱、harness 和执行链路。二者通过 SDK 接起来，agent 就不再是另一个需要切换的工具，而是已有协作面的执行层。

### [GitHub 上 1 万个投毒仓库：Agent 会自动 clone 的时代，README 也是攻击面](https://orchidfiles.com/github-repositories-distributing-malware/)

**这条安全事件和 agent 开发者关系非常直接。**

作者通过 gharchive 事件流和 GitHub API 扫出约 1 万个投毒仓库：攻击者克隆新建仓库，保留原作者 commits 和 contributors 建立信任，再反复推一个“Update README.md”提交，往 README 塞 zip 下载链接。zip 内是 Windows 木马，单查链接不报毒，查包才暴露。

对人类开发者，这已经够危险；对 agent 更危险。coding agent 会自动搜索仓库、clone 仓库、读 README、运行安装脚本，正好踩中这条链路。这也印证了 agent 安全里反复出现的一条经验：外部仓库内容必须按不可信输入处理，否则 poisoned README 就可能顺着 GitHub connector 进入上下文。

## 社区热议

### [单条 `/goal` 让 agent 花 10 天重构 Postgres](https://x.com/samwillis/status/2069147163255312392)

这个案例在社区引发讨论，不是因为它证明了 agent 已经能独立完成数据库架构级改造，而是因为它把任务尺度拉大了：1000+ commits、12.4 万行改动、786 个文件、持续 10 天。它让大家开始重新想象“委派一个目标”而不是“请求一个补丁”会是什么样。

争议也很自然：长任务产出越大，审查成本越高；自动推进越久，越需要可回滚、可验证、可分阶段接管。它更像一个方向信号，而不是可直接复制的最佳实践。

### [Claude Code 的 Extended Thinking 文本是否真实代表模型思考](https://patrickmccanna.net/the-text-in-claude-codes-extended-thinking-output-is-not-authentic/)

这条讨论击中了一个用户体验上的误区：工具里显示的“thinking”文本，未必等同于模型真实内部推理过程。社区关心的不是哲学问题，而是工程问题：如果用户把这些文本当成可审计证据，可能会对模型决策链产生过度信任。

更稳妥的做法是把 thinking 文本当成可解释输出的一部分，而不是证据本身。真正可审计的仍然是工具调用、文件 diff、测试结果、日志和可复现步骤。

### [我不再在 CLAUDE.md 写规则，改写 hooks](https://old.reddit.com/r/ClaudeAI/comments/1ucnasw/i_stopped_writing_rules_in_claudemd_and_started/)

这条 Reddit 经验帖和 Matt Pocock 本周关于 skill no-op 的推文形成呼应：很多写在 CLAUDE.md 或 skill 里的软性规则，模型未必稳定遵守；如果规则有明确可检测条件，把它改成 hook 或脚本，效果会更可靠。

这也提醒我们：agent 配置不是越长越好。真正能稳定改变行为的，要么是上下文里不可缺少的信息，要么是工具、权限、测试和 hook 这类外部约束。空泛的“请更认真”“请更详细”很容易只是心理安慰。

### [AI 的可负担性危机：成本约束会反过来塑造产品形态](https://blog.dshr.org/2026/06/ais-affordability-crisis.html)

社区对这篇文章的关注点在于，它反驳了一个过于线性的叙事：只要 AI 变强，企业就会大规模替代人力。作者认为，当公司开始要求员工谨慎使用 token、控制 AI 成本时，AI 就不是无限便宜的自动劳动力，而是需要算 ROI 的资源。

这对 agent 尤其重要。长任务 agent 往往消耗更多 token、更多工具调用和更多验证成本。未来的优秀产品不会只展示“能跑多久”，还要回答“每一步为什么值得跑、谁来验收、失败成本是多少”。

## 开源社区

### [calesthio/OpenMontage](https://github.com/calesthio/OpenMontage)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-25/openmontage.jpg)

**本周固定周榜第一，把视频生产拆成 agentic pipelines、tools 和 skills 的完整工作室。**

OpenMontage 定位为开源的 agentic video production system，周榜新增接近 1.3 万 stars。它的看点不是又一个视频编辑器，而是把视频生产拆成 12 条 pipeline、52 个 tools 和 500+ agent skills，试图让 AI coding assistant 直接变成视频生产工作室。对关注 agent 应用层的人，它是“把 skill 化工作流推到非代码生产”的代表样本。

### [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-06-25/ponytail.jpg)

**最近很火的“马尾辫”skill，用来阻止 AI 什么都自己写、什么都过度设计。**

Ponytail 的设定很有梗：把那个“少说话、只改一行、但刚好能解决问题”的资深工程师塞进你的 AI agent。README 里的基准测试显示，在真实 Claude Code 会话中，它让代码行数平均减少约 54%，token、成本和时间也下降，同时保留安全约束。这个项目火起来，不只是因为文案好玩，而是它抓住了当前 agent 的一个通病：模型太容易把简单需求做成小型框架。

它也很适合放进本期主线。前面讲的是权限、身份、沙箱和企业授权，Ponytail 讲的是另一层治理：让 agent 在实现策略上学会克制。真正可用的 AI 工程师不只是能写更多代码，也要知道什么时候不写。

### [OpenCut-app/OpenCut](https://github.com/OpenCut-app/OpenCut)

**开源 CapCut 替代品，本周继续出现在趋势榜前列。**

OpenCut 是一个面向网页的视频编辑器，主打 privacy-first 和本地处理。它和 OpenMontage 放在一起看很有意思：一个代表 AI agentic video production，另一个代表传统编辑器体验的开源复刻。两者都说明视频生产正在成为 AI 与开源社区共同争夺的新应用层。

### [withastro/flue](https://github.com/withastro/flue)

**Astro 团队相关的 sandbox agent framework，本周在周榜里继续升温。**

flue 的描述很克制：The sandbox agent framework。它值得放进本期，是因为它和整篇周报的主线贴得很紧：agent 能否安全长跑，越来越取决于沙箱、权限、工具边界和可恢复执行环境，而不是只有模型本身。相比大而全的 agent 平台，flue 更像基础设施层的一块积木。

### [mukul975/Anthropic-Cybersecurity-Skills](https://github.com/mukul975/Anthropic-Cybersecurity-Skills)

**一个面向安全场景的 agent skills 包，覆盖 MITRE ATT&CK、NIST、ATLAS 等框架。**

这个仓库收录 817 个结构化 cybersecurity skills，声称可用于 Claude Code、GitHub Copilot、Codex CLI、Cursor、Gemini CLI 等平台。本周它在固定周榜里高增长。更重要的是，它把“skill”从个人效率配置推进到专业领域知识包：安全任务不是靠一句 prompt，而是靠可复用、可映射到框架的操作单元。

## 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按“洞见性、独特性、深挖价值”三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。不是纯 AI 生成，而是机器采集 + 人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 往期推荐

[AGI 摸鱼周报 #4：Agent 开始学会给自己踩刹车](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490683&idx=1&sn=d6ecc8a96cd72cac4b57a9d8522d2433&chksm=fc0949eccb7ec0fa73ff3e199031eb0dda7cd7b3f1570170cab61f3a07b0b3ee1f38fd03c9b0#rd)

[AGI 摸鱼周报 #3：模型继续提速，验收与责任成为新瓶颈](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490661&idx=1&sn=3a4a8e655f11775664bf8f6a1eb35a2e&chksm=fc0949f2cb7ec0e4e39b8c94771715e6f1e1610984062846a552ca6c5ebeb2c27d823c7f0f20#rd)

[AGI 摸鱼周报 #2：Agent 开始争夺上下文入口](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247490609&idx=1&sn=79d1238a60bdcd16bdf908896d188d9d&chksm=fc0949a6cb7ec0b0d41e287379871cbd8068d01a480065945486d3bfe62765e6d38cddbb517b#rd)

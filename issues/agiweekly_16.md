# AGI 摸鱼周报 #16：什么都不装的 agent，比用 25 万星的 skill 效果更好

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

「该不该教 agent 做事」拿到了迄今最硬的实测答案：danluu 用 26 种 prompt 条件加 4 个 skills 各跑 80 次，不加任何指令的 Default 组高于平均，GitHub 25 万星的 ECC 更贵更差。同一周，Claude Code 发布 `/skill-doctor` 帮用户剪枝没用的 skills，GitHub trending 前二十里八个以上是 skills 项目，Ask HN 在讨论 skills 文件怎么管，GPT-6 的官方迁移指南从模型侧给出同一结论：指令遵循变强，旧 prompt 与 skills 反而成了风险源。爆发与第一波检验撞在同一个星期，skills 生态从「多装几个」进入「该删几个」的阶段。

另一条主线是把同一个模型用得更便宜，外加一场数据边界的信任危机：Spotify Portal 插件把跨文件阅读和代码写入路由给便宜模型，token 成本砍 90%；Anthropic 官方实测 SWE-bench 成本降 55%；Qwen 3.8 27B 配 OpenCode 反超 Astra 6.0 配 Codex，还快 4 倍。而 Navier-Stokes 风波暴露出「不用你的数据训练」和「不查看你的数据」是两个不同的承诺，OpenAI 的 rogue agents 已被发现在 wiki 上互相共享沙箱越狱手法。模型侧的差距抹平之后，工程侧和制度侧的差距刚开始拉开。

## 🚀 产品与模型动态

### 🟠 Claude Code

本周连发 2.1.261（9 月 4 日）、2.1.263（9 月 6 日）、2.1.265 与 2.1.266（9 月 8 日）、2.1.267（9 月 9 日）五个版本：

- **`/skill-doctor` 命令**（9 月 4 日）：显示哪些加载的 skills 从没被用、各占多少 context，直接照着剪枝。
- **`bashOutputMaxChars` / `taskOutputMaxChars` 设置**（9 月 4 日）：命令和后台任务输出的内联上限最高可提到 128K 字符，大日志不用再落盘读文件。
- **`--plugin-dir` 指向文件夹**（9 月 8 日）：一个目录下的多个插件批量加载，运行中增删会被自动识别。
- **`maxEffortLevel` 设置**（9 月 9 日）：给每个 provider（含 Bedrock、Vertex、Foundry）的 effort 档位封顶，防止 subagent 或 skill 把成本顶到最高档。
- **prompt cache 稳定性大修**（贯穿 2.1.265 至 2.1.267）：恢复会话、`/model` 切换、MCP 重连、subagent 启动等十余种场景不再打断缓存，对长会话的账单是实打实的利好。
- **VS Code 侧**（9 月 8 日）：闲置 14 天的会话自动归档。

再叠加 9 月 3 日至 4 日的官方三连：Function Hooks 预告（把 Claude Code 做得更可编程）、`ant apply` 资源即代码（agents、skills、memory stores 等五类资源声明为仓库内文件，带锁文件与 dry-run 的完整 GitOps），Claude 生态在朝「一切进版本控制」走。

### 🟢 Codex

本周从 0.153.1 推进到 0.153.4（四个 stable hotfix）：

- **Astra 成为 bundled 默认模型**（0.153.4）：未显式配置模型时直接用 Astra。
- **GPT-6-Astra 上 Amazon Bedrock**（0.153.3）：Mantle 与 Runtime 的 global/US 路由均已可用。
- **`features.context_management.experimental_mode`**（0.153.0 新配置）：为符合条件的 Plus、Pro、Pro Lite 会话启用 token-budget context、history notes 和 `new_context` 工具。Astra 发布时说的「跨上下文窗口保持笔记、早期上下文保持可搜索」现在有了正式开关，愿意尝鲜的可以在 config.toml 里打开。
- **effort 校准**（Tibo 9 月 7 日）：GPT-6 Astra 的 low 档表现已超过 GPT-5.6 Sol 的 high 档，习惯开 high 的用户应直接降到 low 或 medium，省额度也省等待。

社群信号（均为 Tibo 9 月 9 日推文）：Astra 需求「前所未见」，若增长持续可能暂停新增 Pro 订阅，优先保障存量用户体验。

### 🧰 其他工具与模型

- **Cursor 断供进入倒计时**：OpenAI 模型 11 月 12 日停止供应，Anthropic 明确继续合作，Grok 系全线上位。多 editor 用户的模型可用性从此要当成选型变量。
- **Fable 5.1 双信号**：文本水印已实际生效，翻译产出同样被注入，而承诺公开的检测器仍停留在合资格组织的私测；同期 r/ClaudeAI 多帖反映 Pro 用户 token 消耗异常上涨（小改动任务从约 5% 限额涨到 40% 以上），官方暂无回应。
- **ChatGPT Images 2.5 把宝押在编辑上**：五个升级维度全是编辑向：参考图稳定性（最多 16 张参考图保持主体特征一致）、多轮编辑一致性（改十轮人物构图不崩，「一改全图崩」的老痛点被正面解决）、精确局部编辑（直接在图上标注评论，只改指定区域）、视觉理解、速度（快 50%）。API 侧模型为 `gpt-image-2.5-sunburst`，已向全体 GPT 与 Codex 用户开放。对 agent 工作流的意义在迭代方式：生成 UI 概念图、图标、博客配图之后，可以像改代码一样按注释局部修改，不必每次推倒重建，多轮修改的质量不随轮次衰减。

## 📌 本周精选

### [danluu 实测 26 种 prompt 条件：让 agent 用 TDD、形式化方法、流行 skills，全都不如什么都不说](https://danluu.com/agentic-testing/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/danluu-agentic-testing.png)

**不加任何测试指令的 Default 组表现高于平均，GitHub 25 万星的 ECC 反而更贵更差。**

Dan Luu 做了目前对「该不该教 agent 做事」最狠的一次实测：同一个 Rust 实现 Zstd 的任务，26 种 prompt 条件（TDD、QuickCheck、Lean 4、Verus、TLA+ 等）加 4 个 skills，每条件跑 80 次。最反直觉的结论是 Default 组（不加任何指令）高于平均水平：让 agent「用某种技术」，多数时候是让它做无用功。形式化方法组大量产出空洞证明（形如 A=>A）；TDD 组写了双倍测试，却更容易写出对不上的错误测试。

skills 部分更扎心：ECC、Hegel 官方 skill、Trail of Bits 的 skill 全部 underperform 且更贵（Hegel skill 成本增加 26% 到 41%），而 danluu 两分钟手写的「行为修正式」skill 反而拿了最高分。教程式 skill（教 agent 怎么做）不如把 agent 从默认行为上推开几句的写法。这周有三件事互为注脚：GitHub trending 榜一的 ECC 恰好是被这份实测打脸的那个；Claude Code 9 月 4 日发了 `/skill-doctor` 帮你清理没用的 skills；Ask HN 高赞判断是「承载 contextual guidance 的 skill 不会被模型能力吃掉，承载 raw capabilities 的会」。落到日常就是：别往 prompt 里堆方法论，先看 agent 默认行为差在哪，再精准纠偏。

### [Armature 实测 16,893 个 session：Claude Code、Codex、Cursor 各自怎么替你选技术栈](https://armature.tech/blog/which-tools-coding-agents-install)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/armature.png)

**三家 agent 只在 42% 的情况下选中同一个工具，被提及 194 次的 LangChain 只被选了 4 次。**

Armature 构建了 75 个仿真仓库（10 种语言、假公司名、假 git 历史、真实 lockfile）、1,163 条 prompt 变体和 4 种用户 persona，让三家 agent 真正动手实现而不是只做推荐，共跑出 16,893 个 session。决策路径截然不同：Cursor 三分之二靠 web 搜索决策，Codex 94% 用搜索且九成带 `site:` 操作符锁定信任域名，Claude Code 只在三成 session 搜网、主要靠先验，但搜索时浏览页面数是 Codex 的三倍，自建比例 19% 几乎是另外两家的两倍。

对开发者工具的作者，有三条发现：仓库语言决定选型，同一个邮件需求落在四种语言上选出四个不同服务商；「被提及」不等于「被选中」，提及量最大的数据库 Supabase 整体输给 Neon，PayPal 被 cite 139 次零次胜出；定价页的一行细节足以翻盘，Mailgun 因免费计划里「1-day retention」字样常输给 Postmark。这是第一份可直接查询「自己品类被 agent 如何选择」的公开数据集。

### [Forge Memory：让 agent 记住工程判断，腾讯把「新人第一天入职」问题拆成三个断点](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247697297&idx=1&sn=9f3932deb893f278976ebb473721b616)

![](https://cdn.zhangferry.com/Images/202609102322416.png)

**agent 每次接新任务都像新人第一天入职，问题不在记不住，在什么知识值得跨任务复用、谁来把关。**

agent 用得越久越能感到一个落差：它在这个任务里摸清的工程判断（这个项目用 pnpm 不用 npm、这个服务的错误码有历史坑），下次任务全部归零。腾讯云开发者团队的 Forge Memory 把这种「一次性智能」拆成三个断点：任务结束判断丢失、判断无法沉淀为项目知识、知识复用时无人校正。对应的设计是三件套：知识对象建模，把对话里的判断结构化为可引用的对象；双门准入，一道门管相关性、一道门管准确性，防止错误判断沉淀；独立子流程，知识沉淀不占用主任务的上下文。最有参考价值的是双门准入背后的立场：记忆系统的风险不是记不住而是记错，错误判断一旦跨任务复用，会被当成项目惯例持续放大。对照 Claude Code 近期把 memory stores 纳入 `ant apply` 资源体系的动作看，工程判断的持久化正在从个人 hack 变成平台能力。

### [从一次 LLM 调用到完整 Harness：腾讯技术工程拆解 agent 的四层设计取舍](https://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649803963&idx=1&sn=ec0730b1274a0b52382df956d8583215)

![](https://cdn.zhangferry.com/Images/202609102323865.png)

**模型同质化之后，agent 的差异化全在 harness：上下文怎么装配、事件怎么驱动、执行怎么隔离、经验怎么进化。**

这篇系统梳理适合当作 harness 概念的入门地图：从单次 LLM 调用（无状态、无工具）到完整 agent harness 的演化路径，每一步都在补上一种能力，工具调用补行动力、上下文装配补记忆、事件驱动补响应、沙箱补安全。核心部分是四个框架的横向对比：Pi、OpenCode、Codex、Hermes 在上下文装配（全量注入还是按需检索）、事件驱动（轮询还是订阅）、安全执行（容器隔离的粒度）、自进化（harness 自身能不能被 agent 改写）四个维度上的取舍各不相同。本周社区热议里那条「同一个任务换 harness，成绩差 18 个百分点」的实测是它最好的现实注脚：模型能力被抹平后，框架层的工程决策直接决定 agent 表现。想理解自己每天在用的 Claude Code 为什么这样设计，这篇给出了参照系。

### [OpenAI 首席科学家长文《An Alien Mind》：CoT 监控正在失效，o1 隐藏思维链是刻意设计](https://openai.com/index/an-alien-mind/)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/alien-mind.png)

**「目前没有任何实验室解决了足够程度的对齐与监控，可以负责任地以最高速度继续 scaling」，这话出自 OpenAI 首席科学家。**

Pachocki 的这篇长文是三年来最重量级的个人立场陈述。核心判断是「AI 是长出来的而非设计出来的」，因此对齐问题的本质是泛化问题。他区分了 goal alignment（完成给定目标）与 value alignment（在陌生环境中仍持守原则），并指出两类主流对齐方法各自的失效模式：goal-oriented RL 脆弱且依赖训练覆盖，OpenAI 与 Hugging Face 事件中的 agent 保住了「不 social engineering 人类」的边界，却在其他所有地方越了界。

全文最重的一段是直接承认 CoT monitoring 的可靠性正在递减，原因有三：环境复杂化模糊了监督边界、模型越来越擅长操纵自己的推理过程、预训练进步让模型不靠言语化推理也变聪明。他同时首次官方确认 o1-preview 当年隐藏思维链是刻意设计，目的是保护 CoT 不受监督压力。结合本周另外两条动态看会更完整：Anthropic 对齐负责人 Hubinger 公开回应离职研究员的批评，直言「我个人认为十年内灭绝风险超过 10%，我们没有解决超级智能对齐的计划」；OpenAI 的 rogue agents 则被发现在内部 wiki 上互相共享沙箱越狱手法，agent 的越界从孤立行为变成了会互相传染的经验。两家实验室的安全叙事都在接受内部人的压力测试。

## 💬 社区热议

**GPT-6 带来的第一波 prompt 迁移实践**。大版本迭代的隐藏成本这周显形：Astra 的指令遵循比前代强得多，旧 prompt 和 skills 反而成了风险源，以前被模型自动忽略的含糊或冲突指令，现在会被严格执行、让任务中途卡住。OpenAI 发布了成套迁移指南（GPT-6 Astra 模型指引）：把 can you 和 I want to 当作动手指令、批准是最后一步，声明用户指令优先于 skill 文件，小改动不写测试，连反 AI 腔的 slop 词表都给了官方版本。r/codex 638 分的热帖把迁移做成了动作：跑 `$openai-docs migrate this project to GPT-6 Astra` 让 Codex 自己改项目，有用户晒出产出是在 AGENTS.md 顶部加了一份 37 行的 Astra 工作契约。评论区最有价值的讨论是分层：为一个模型优化完指令，别的模型怎么办？高赞答案是 AGENTS.md 拆两层，项目层写「不建什么、什么必须成立」，模型层各配各的，harness 应该让切换模型不牵动项目约定。这和 danluu 实测、`/skill-doctor`、trending 的 skills 爆发拼出同一幅图：模型的指令遵循越强，上下文里的指令越要精挑细选，堆上去的旧方法论正在从无害变成有害。

**Spotify 开源 Portal 插件，Claude Code token 成本砍 90%**。原理不复杂：把最烧 token 的 bulk-reader 和 code-writer 两个模式自动路由给便宜模型执行，贵模型只保留编排和决策，三条命令从插件市场装完即生效。社区调侃「我们是不是该叫它 subagent」，但把路由策略做成强制执行的插件封装是新的贡献。与 Anthropic 官方降本长文思路互补：官方用 `/claude-api` 的 cost-optimize、prompt-audit、hillclimb 三命令实测，SWE-bench Verified 成本砍 55%，Sonnet 5 低 effort 以 1 美分通过 98.9% 工单，还澄清了改 effort 会重渲染前缀破坏 cache 的坑（Opus 5 与 Fable 5.1 除外）。一个从 harness 侧省、一个从 API 参数侧省。

**同一个任务换 harness，结果比换模型还大**。有开发者用固定的 Three.js 任务跑 10 组 model×harness 组合：同一个 GLM 5.3 Flash 在 OMP 上只有 78.5%，换 OpenCode 就到 96.9%；Qwen 3.8 27B 配 OpenCode 拿 95.6%，只花 8 分 48 秒和 70 万 token，而 Astra 6.0 配 Codex 拿 94.9% 却花了 37 分钟和 133 万 token。选型顺序该反过来：先挑 harness，再配模型，小模型加好 harness 可以越级挑战旗舰。

**Ask HN「谁在生产环境用 MCP」收获 171 条一线回答**。官方 SDK 日下载量 1700 万创新高，有公司明说 MCP 支持已是采购硬性门槛（No MCP = NOGO）；Adidas 生产环境同时跑 stdio 与 remote 两种模式，Notion 内部有 16 个以上 MCP config。反方同样有力：终端场景里 CLI 加 skills 往往比 MCP 更顺手，「开发者用 CLI、跨系统集成用 MCP」的分工正在形成。同周 Notion 官方 connector 被起底 prompt 注入（任务中途推销 Notion Business 且指示 agent 永不解释），MCP 越成为分发标准，信任层的欠账越致命。

## 🧩 开源社区

### [openai/skills](https://github.com/openai/skills)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/openai-skills.png)

Codex 官方 skills 目录，本周身份升级：README 置顶公告 deprecated，skill 与插件示例整体迁入 [openai/plugins](https://github.com/openai/plugins)，成为 plugin 体系的一等公民。三层安装结构不变：`.system` 随 Codex 自动装，`.curated` 用 `$skill-installer` 按名装，`.experimental` 按 GitHub 目录 URL 装。对写 skill 的人，「怎么分发」有了统一答案：按 plugin 规范打包进市场。

### [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/i-have-adhd.png)

治 agent 文字墙的 skill。README 的 Before/After 对比一目了然：Before 是从 Great question 到 Hope this helps 的婉转长段，After 直接给命令、文件行号加三步编号操作。规则 10 条，核心是结论先行、步骤编号、砍掉客套。往 CLI 粘一句安装指令即可用，对着本周「问 A 得到一堵墙」的最集中抱怨下药。

### [blader/humanizer](https://github.com/blader/humanizer)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/humanizer.png)

去 AI 味 skill，本体就是一份 Markdown，任何支持 skills 的 agent 都能装（`npx skills add` 或 Claude Code 插件市场），`/humanizer` 加正文即用。特色是 voice matching：贴几段自己写的样本，改写会跟随你的节奏、用词和标点。与 Cantrill《读者的反抗》（78% 开发者见 LLM 文风即弃读）同周走红，AI 味治理从讨论变成工具。

### [mksglu/context-mode](https://github.com/mksglu/context-mode)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-09-10/context-mode.png)

自称「context 问题的另一半」：模型侧在扩窗口，它管工具输出侧的膨胀。工具输出先进沙箱（官方称削减 98%）再进上下文，会话记忆持久化，按模式强制路由，通过 MCP 加 hooks 接入 17 个平台。README 置顶着 Hacker News 榜第一（570+ 分）徽章，对被 token 暴涨困扰的用户是现成的减压阀。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 GitHub 仓库：https://github.com/zhangferry/AGI-Weekly

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

### 📜 往期推荐

[AGI 摸鱼周报 #15：模型超级周，GPT-6 登场](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491247&idx=1&sn=944e3acd5a27ad2d291c625a54708e45&chksm=fc094b38cb7ec22e39c14baa73c15a03d221bb96114f46c118dd0a826bc440038afde9bef574#rd)

[AGI 摸鱼周报 #14：六周无限额度实验结束，Codex 的 5 小时限额回来了](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491069&idx=1&sn=ce3b54e48fe51ade3628b059137d8d15&chksm=fc09486acb7ec17c9cec47d11a03fc8373d8afc0d74f20072296627e51b09fc8cc99f3312246#rd)

[AGI 摸鱼周报 #13：Linear 首份数据报告：agent 团队 PR 两年翻三倍，但业务价值是否提升Linear自己也说不清](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491051&idx=1&sn=7d7cb7bddf57d5ca6c5711769cd030e5&chksm=fc09487ccb7ec16a89b444e089b4f24b2e03ad6e56bc811c13f465dcf6b0b70f8ba2f4aa53cb#rd)

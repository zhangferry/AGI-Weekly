# AGI 摸鱼周报 #25：从写 skill 到运营 skill

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

skill 正在从个人保存的提示词片段，变成需要治理、评估和算成本的基础设施。Uber 的共享 skill 市场一年内从 1 个涨到 5,000 多个，单月被 agent 调用超过 100 万次，真正撑住这个规模的不是写法，而是一套评估方法：5% 金标准用例、80% 真实 prompt 按轨迹打分、15% 作者判定，他们还发现通过率 70%–90% 的 eval 比 100% 的更有信息量。协议层同步跟上了：MCP server 现在可以把 skill 和工具一起分发，Claude Code 里直接以 `server:skill` 出现，skill 不必再由每个人单独安装。

skill 多了之后，agent 的下一步是接手「PR 之后」的事。Anthropic 发布 Claude Ship：PR Steward 盯着失败的 CI、合并冲突和评审反馈把 PR 修到变绿，Claude Approvals 再按仓库里一份自然语言写的策略文件决定哪些改动可以自动批准。Patrick McKenzie 借一个 devbox 里 2 到 100 多个 Claude 的编排器，一天合了 42 个 PR，他的结论是自己已被从「写代码和操作终端」里解雇，剩下的事是 review、信任和流程设计。

## 🚀 产品与模型动态

### 🧰 Harness 工具更新

**Claude Code**：MCP server 分发的 skills 进入 beta，在 `/` 菜单里以 `server:skill` 出现，其他 skill 保持原名；新增 `PreSkillShellCommand` 钩子，可在 skill 或命令加载时拒绝其中的内联 shell 命令。周期性定时任务和 `/loop` 最长可连续运行 30 天（原为 7 天），此前建好的任务仍按 7 天结束。行为变化：从 11 月 30 日起，托管配置 `extraKnownMarketplaces` 里的插件市场默认自动更新，不想更新的在对应条目上设 `"autoUpdate": false`。

**Codex**：0.171.0 把 GPT-6.2 Sol 设为默认模型，TUI workspace 转正并默认开启，主对话旁可查文档预览、图片预览和 subagent 转录；新增 `codex exec --remote <ADDR>`，在已运行的 app server 上启动或恢复非交互会话。0.172.0 支持 `Tab` 补全 slash 命令参数，带缓存工具列表的可选 stdio MCP server 默认等到第一次工具调用才启动。

**GPT-6.2 额度修正**：OpenAI 称 GPT-6.2 上线时的计费错误让多数用户的 Astra、Sol 用量被少计，修复后与上一周相比，同样的工作在 Astra 上到达限额快 40%、Sol 快 25%；Go、Plus、Pro、Business 各补一次已入账的额度重置，过去的差额不补收。

**DeepSeek Harness**：v0.2.3-alpha.1 的插件市场支持把 Git 仓库（含私有仓库）加为插件来源；新增代码审阅页面，集中查看与自己相关的 GitHub PR，审阅插件支持 GitLab 的评审与合并；MCP 支持 OAuth 授权与凭据自动刷新；可选连接器补齐 Linear、Notion、腾讯文档、飞书、企业微信、腾讯会议、钉钉。Computer Use 可经 cua-mcp 以 MCP 服务器形式对外提供，Windows 为实验性支持；Python PTC 不再支持。

### 🧠 模型动态

**GLM-5.4（智谱）**：首个基于 Loop Transformer 的旗舰：把 GLM-5 底座的 28 层循环跑两次，在不增加参数的前提下换更深的推理，官方称智能水平对标 2T–3T 参数模型；750B 规模，1M 上下文，闲时最高 200 tokens/s、高峰 70–80 tokens/s。注意官方页面没有给出任何编码基准成绩，下面的对照表来自 30B 同架构模型的等参数实验：只多约 10% 训练算力，GPQA-Diamond 从 47.58 提到 56.26、MMLU-Pro 从 71.65 提到 75.05，思考输出长度缩短 9.7%–14.9%。目前只对 GLM Coding Plan 用户开放（周配额为 GLM-5.3 的 1.5 倍），API 待安全评估后开放，权重承诺后续开源，已知缺陷是容易陷入过长的「马拉松思考」。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/model-glm54-ablation.png)

GLM-5.4 技术博客的 30B 等参数对照：Baseline 为不循环，+ Loop 为循环两遍，右侧为输出长度变化。

## 📌 本周精选

### [How Uber Evals 1 Million Agent Skill Invocations a Month：从写 skill 到运营 skill](https://www.uber.com/us/en/blog/optimizing-ubers-agent-skills/)

**skill 写到几千个之后，问题从「怎么写」变成「怎么测、怎么省钱、怎么让它自己报缺陷」。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/pick-uber-og.png)

Uber 的共享 skill 市场一年内从 1 个涨到 5,000 多个，单月被 agent 调用超过 100 万次，覆盖 Claude Code、Codex、OpenCode、Cursor 和内部 agent。治理分 Domain、Core、Always-on 三层，约 100 个通用 skill 对所有人默认开启。每个 skill 按需加载：名称和描述常驻，约 100 token；正文运行时才载入，控制在 500 行以内；脚本和参考资料再按需读取。

评估方法是最值得抄的部分：约 5% 是带固定输入和预期输出的金标准用例，80% 用真实 prompt 按轨迹打分，15% 由作者判定。他们的经验是通过率 70%–90% 的 eval 往往比 100% 的更有信息量，因为全过可能只是没测到关键变化。

消融数据也给得直接：不开某个仓库查询 skill 的会话多出 13% 的工具调用，工具错误是 6 倍，耗时多 31%，带 skill 的答案在 87% 的对比里胜出。成本侧的做法有确定性步骤交给脚本、追踪类 skill 换更便宜的模型（单次成本降 46%）、搜索类 skill 加载时直接跑脚本（降 32%）。最后一步是让 agent 用一个常开 skill 自己填「papercut」缺陷单，三个月收到数千条，已修复数百条。

### [Migrating our Messages API to Rust：把生产 API 本身当验证器](https://www.anthropic.com/research/migrating-api-to-rust)

**当写代码不再是瓶颈，验证才是；最可靠的验证器是还在跑的生产系统。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/pick-rust-og.jpg)

Anthropic 把承载全部客户请求以及 Claude Code、Cowork 自身流量的 Messages API 从 Python 迁到 Rust：每请求 CPU 降约 70%，首 token 中位延迟降约 20%，全程没有停机，也没有冻结功能开发。背景数字是 API 流量从 2024 年初到 2026 年秋增长了约 1800 倍。团队先试过增量重写，CPU 只省 8%–10%，每个 token 要在 Python 和 Rust 之间穿三次，收益配不上复杂度，于是改为整体重写。

难点在验证。把旧测试移植过来后，审计发现 100 多处被弱化的断言；4 月 Rust 版通过了约 98% 的测试，但在生产流量上只有约四分之一的输入能对上。团队的做法是把还在运行的 Python 版当 oracle，让同一批请求走新旧两套代码比对结果。因为不能看用户请求，他们只记录差异的「形状」：发生在服务哪一步、哪一段、差了几个字节，归成指纹再交给 agent 修。

整套系统是三层嵌套的循环：内层给 agent 一个沙箱，几秒钟就能复现并改代码；外层在影子生产流量上验证修复；最外层的「记忆循环」把人工 review 的纠正固化成规则、测试和 lint，每一层都能抓住里面一层漏掉的问题。公开流量 9 月切到 Rust。对任何要做大改动又没有完整测试的团队，这篇文章给了一个可照搬的判断：先回答「什么算正确」，其余交给 agent 迭代。

### [Multiagent coding is the future：一个人管 2 到 100 个 Claude 的一天](https://www.kalzumeus.com/2026/11/25/multiagent-coding-is-the-future/)

**把自己从写代码和操作终端里解雇，才会发现瓶颈是 review、信任和流程设计。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/pick-kalzumeus-firstscreen.png)

Patrick McKenzie 停笔一段时间后回到编码，判断多 agent 编程和 LLM 写代码是同等级的范式转移。他借用了 Situational Awareness 的内部编排器 Grimm：一个 devbox 里跑 2 到 100 多个 Claude，彼此能对话，人通过 tmux 上的 Web UI 管理。Grimm 本身是自修改软件，编程语言是英语；招一个新「员工」只是给管理进程发一条短信，几秒后它就开始执行指令，要十个也是同样的延迟。

他自己的战绩：一天合了 42 个 PR，对着的是当天才第一次接触的复杂系统，端到端部署延迟降了 80%；8,600 行改动、161 个文件、823 美元推理开销。他的说法是：不把自己从写代码和操作终端里解雇，就无法真正理解多 agent 开发，因为这些事现在「有人」干了。

文章没有回避保留意见：他和对方都怀疑 Grimm 不会是最终胜出的那套系统，只是实验足够有价值，值得拿出来讲。他还提醒 agent 自主行动的风险已被现实事件证明，金融机构会用红队、双人复核、最小权限那一套来约束。

### [How iFood scales coding agents across 15,000 repositories：把 agent 做成一个内部 REST API](https://openai.com/business/customer-stories/ifood)

**要在上万个仓库上跑 agent，先解决的不是用例，而是运行时和「任务到底完成没有」。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/pick-ifood-og.png)

iFood 维护 1.5 万多个代码仓库，周期性的漏洞依赖修复、规范校验这类任务单仓库都不难，难的是规模。他们搭了内部的 Remote Agents Platform：入口是一个 REST API，提交 prompt 和环境要求后，任务在隔离沙箱里由 GPT-5.6 Luna 执行，结果以 merge request 形式返回。平台已跑完 2.5 万多个任务，消耗超过 270 亿 token，平均每次运行推理成本约 0.04 美元。

设计上的关键是确定性步骤和 agent 执行的组合：克隆仓库、导入 skill、建分支、发布 MR 用脚本，中间那段复杂逻辑才交给模型。另一个是完成判定：会话结束后，结果用 Structured Outputs 按预先约定的契约处理，从响应结构里就能判断任务有没有做完。

首个大用户是安全团队：扫 1.5 万个仓库的漏洞依赖并修复，迁移后修复准确率和修复数量都翻了一倍以上。对要搭 agent 流水线的团队，这篇的可迁移经验是把「每次成本」当架构决策，异步执行才是效率的来源。

### [大众的 macOS，还是开发者的 macOS？：肘子的 Swift 周报 #163](https://fatbobman.com/zh/weekly/issue-163/)

**Agent 要跨目录自主干活，操作系统的权限模型却要求人一次次点确认。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/pick-fatbobman-163.webp)

这期周报的卷首语从一个具体变化说起：macOS 27.3 beta 2 里给应用授予完全磁盘访问权限时，提示框的警告意味比以往任何版本都重，系统也在引导用户转向按目录、按应用的细粒度授权。作者自己的经历是，即便已经给 Xcode headless MCP 授了完整权限，只要打开 Xcode，Preview 截图时仍会弹权限窗口。

他指出的矛盾是：需要跨工具、跨目录自主协作的 agent 场景下，每次跨越权限边界都要用户即时确认，原本保护用户的机制就成了自动化工作流里频繁出现的中断点。

作者不认为 Apple 会在主流版本放松权限管控，给出的方向是在 macOS 虚拟机里提供一套更适合开发者与 agent 的权限模式：借助虚拟机的隔离边界，把高频授权转成对整个开发环境的一次性授权。给 Mac 上的 agent 配环境的读者，可以把这条当作选方案的依据。

### [Announcing DreamCoder：把 coding harness 当成未来的 IDE 重做一遍](https://www.masteringemacs.org/article/announcing-dreamcoder)

**harness 不是聊天侧栏，它是 agent 时代的 IDE，缺的是设计取舍而不是模型能力。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/pick-dreamcoder.webp)

《Mastering Emacs》的作者 Mickey Petersen 花了 15 个月做出 DreamCoder，一个跑在浏览器里的 coding harness，三大平台可用。产品形态直接借 Emacs：平铺窗口管理器，每个 pane 有最小、舒适、最大三种尺寸，塞不下的进隐藏窗口栏；workspace 快照；avy 式跳转和 embark 式操作。harness 该有的也在：MCP、可后台的子 agent 和 shell 命令、终端窗口、skills、hooks。

文章里两个观点比功能更值得记。一是「品味是稀缺的人类资源」，多数 harness 的问题是设计取舍而不是模型能力；二是本地开源模型已经够用，他点名 Qwen 3.8 27B 在高配 Mac 上能撑起大量真实开发。

用法上，DreamCoder 个人和五人以下团队免费，Pro 每席位每月 20 美元，功能相同，支持 GitHub Copilot for Business 或自带 OpenAI Codex 订阅。文章没有说明是否开源。

## 💬 社区热议

### [Ask HN：被 Anthropic 指控「蒸馏」，只有 24 小时自证](https://news.ycombinator.com/item?id=50630809)

发帖人说公司和本人被 Anthropic 指控用其 API 做蒸馏，但他们不训练自己的模型，唯一可能触发判定的用法是「每次改完 AI 设置就跑一轮自动化回归 benchmark」，现在只有 24 小时提交反证。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/comm-distill-askhn-crop.png)

原帖截图：发帖人自述被指控蒸馏，唯一可疑用法是自动化回归 benchmark，限时 24 小时。453 分，350 条评论。

评论区的方向很具体。有人建议把生成这批流量的代码连同 git 历史一起交出去，让对方看到是测试流量；多位评论者讲了自己的经历：被 OpenAI 判定蒸馏却没人通知，以及 API key 泄露后被他人拿去蒸馏，吊销密钥后恢复访问。也有人指出更结构性的风险：如果「坏提示」就能让整个账号被封，把蒸馏指令放进网页等 prompt injection 命中，就能借厂商之手封掉别人的账号。

背景是 Anthropic 此前发布的反蒸馏说明：分类器误报率低于 0.1%，已用这些防护封禁数万个账号，通常不到两小时；自 9 月 24 日起，被防护拦截的请求恢复计费。对在 Claude API 上跑 eval、压测或回归的团队，这条的实际含义是：你的测试流量在风控眼里与蒸馏流量同形，留存日志、隔离 key、限制并发比事后解释更有效。

### [不写代码，你对代码的评审能力也会退化](https://news.ycombinator.com/item?id=50573886)

Christian Ekrem 的论点不是「AI 写的代码不好」，而是一个被忽略的因果：评审能力来自实践，只读 AI 生成的代码、不再自己动手解决问题，退化的是「判断什么算好代码」。他用两个类比：不再练琴只讲怎么弹的吉他手，不再写作只批改别人文字的作家，能力都会下降。他还直言「review 所有 Claude 产出的代码」在实践中很难成立，多数人其实只读了其中一部分。405 分，187 条评论。

反对的意见同样具体：有人举教练的例子，评审和编写是相关但不同的技能；有人说自己通过 Claude 写的 Ansible、Terraform 学到了过去没时间碰的东西；也有人指出阅读对写作有很大影响，读代码未必没有同样效果。给团队定「人还要不要写代码」规则时，这是本周最值得对照着读的一组讨论。

### [用 LLM 把 Rockbox 移植到 9 美元的无牌 MP3 播放器](https://news.ycombinator.com/item?id=50624446)

作者花 9.71 美元买了一台无牌 YP3 播放器，SoC 是网上查不到资料的 Smartlink SL6801，SRAM 只有 512 KiB，而 Rockbox 现有最小目标也有 2 MiB。他没有拆机，接上 ghidra-mcp 让 agent 逆向固件、往 FreeRTOS 里注入任务、写驱动，再把 Rockbox 裁到能启动、能放音。257 分，97 条评论。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/comm-rockbox-webcam-small.jpg)

作者的「机器人眼耳」现场：乐高搭的支架上，摄像头对着播放器屏幕，旁边是麦克风。

为了让 agent 自己闭环，他搭了一套感知环境：摄像头看屏幕、麦克风听扬声器、耳机口接声卡 Line In。流程上分了三个 agent：一个用 Ghidra 逆向的「脏」agent、一个防止信息泄漏的裁判 agent、一个写实现的 agent。

文中对两家模型的对比被讨论最多：先用 Codex/Sol 5.6，五天没敢刷机，留下 55 GB、387 万个文件、443 个一次性脚本；换 Claude 后这些全被扔掉，用几个 Python 脚本重做。最终这一版加了 357 个插件，作者自述「100% 是 vibe coded」，他只扫了一眼部分代码。

### [LLM agent 到底在替谁干活：一条 Bandcamp 拒答引出的「忠诚度」问题](https://news.ycombinator.com/item?id=50627340)

作者用 Claude Code 写了一个 skill：从 Instagram reel 里识别歌曲，去找乐队的 Bandcamp 页面，方便他直接付费买音乐。Bandcamp 给搜索页加了反爬 challenge 后，Claude 明确表示不会帮忙绕过；而同一个 skill 里下载 YouTube 音频的分支照跑不误，那条路一分钱也到不了乐队手里。1017 分，411 条评论。

作者由此提出的不是「模型不够强」，而是「模型在替谁做判断」：拒绝边界由厂商的使用政策决定，而不是由付费用户的意图决定，今天是反爬，明天可能是广告加权、政府要求或版权方压力。

评论区的多数意见并不站在作者一边：有人说 Claude 尊重 Bandcamp 不想让 LLM 访问的意愿，替代方案更糟；有人认为这类拒绝是正确的，绕过 challenge 就是违反 ToS。也有评论从法律角度提出，LLM 应当像律师、财务顾问一样对委托人负有信义义务。

## 🧩 开源社区

### [n8n-io/nod](https://github.com/n8n-io/nod)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/repo-card-n8n-io-nod.png)

n8n 出的开源个人 agent，自带一台「电脑」：追发票、准备通话资料、在真实网站上取消试用，动手前先问你。一个 Node 进程加几个 Docker 容器跑在本机，信任检查、保险库、支付限额都在服务端而不在提示词里，密钥不进沙箱，网页和浏览器返回的内容一律标记为数据而非指令。模型自带，支持 OpenAI、Anthropic、Gemini、Bedrock 等 20 多家。目前是 research preview，macOS 14+ 可用，Windows 11 和 Linux 为预览版。

### [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/repo-card-chromedevtools-chrome-devtools-mcp.png)

让 coding agent 控制和检查一个真实的 Chrome，以 MCP server 和 CLI 两种形式提供：录性能 trace 并提取可执行的优化建议，分析网络请求、截图、读带源码映射堆栈的控制台消息，底层用 Puppeteer 做自动化并自动等待结果。Claude Code 里 `/plugin marketplace add ChromeDevTools/chrome-devtools-mcp` 后装插件，v1.14.0 新增网络 mock 的 abort 与 allow-origin。只正式支持 Chrome，默认开启用量统计，性能工具会把 trace URL 发到 Google 的 CrUX API，可用 `--no-usage-statistics` 和 `--no-performance-crux` 关闭。

### [ag-ui-protocol/ag-ui](https://github.com/ag-ui-protocol/ag-ui)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/repo-card-ag-ui-protocol-ag-ui.png)

Agent 与前端之间的开放协议：agent 不再为每个前端手写对接，而是发出一条双向的类型化事件流，任何客户端都能渲染。1.0 规范已冻结并有 JSON Schema，31 种事件分八族，覆盖运行生命周期、文本消息、工具调用、状态增量等；传输无关，默认定义了 HTTP+SSE 和 Protobuf 两种绑定。它和 MCP、A2A 是互补关系：MCP 给 agent 工具和上下文，AG-UI 把 agent 带给用户。`npx create-ag-ui-app@latest` 起步，模板默认调 OpenAI，需要先配 key。

### [Tracer-Cloud/opensre](https://github.com/Tracer-Cloud/opensre)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/repo-card-tracer-cloud-opensre.png)

自托管的 CI/CD 可靠性 agent：流水线失败时读日志、提出假设并验证，把修复推到 PR 分支等人 review。关键设计是在工具调用循环里区分「CI 自身导致的失败」和「代码导致的失败」，避免修错地方；可接 Anthropic、OpenAI、Codex、Ollama、Gemini、Bedrock 等 provider，集成 60 多种工具和服务。`curl -fsSL https://install.opensre.com | bash -s -- -gh` 安装。仍是 public alpha，首次启动需要登录其托管账号，遥测默认开启，用 `OPENSRE_NO_TELEMETRY=1` 关闭。

### [dream-num/univer](https://github.com/dream-num/univer)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-26/repo-card-dream-num-univer.png)

把表格、文档的编辑、计算、协作和文件处理做进你自己的产品，人和 agent 走同一套开源 API，模型和工具编排由你的应用提供。核心是命令系统加描述内容变更的 mutation，协作时打包成 changeset 做操作变换。`pnpm add @univerjs/presets @univerjs/preset-sheets-core` 几行就能起一个表格。注意开源的只有 Sheets 和 Docs 的 Web SDK 核心，Slides、Boards、Server SDK 和多数 AI 扩展单独商用授权。

## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 [GitHub 仓库](https://github.com/zhangferry/AGI-Weekly)

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

## 📜 往期推荐

- [AGI 摸鱼周报 #24：五家前沿模型同周发布，价格卷到了竞品的 5%](http://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491916&idx=1&sn=526f807a3337726c646d1fd8a0a32f46&chksm=fc0ab4dbcb7d3dcddd56e0890884bc6b340902f155e615aa3f23267ca07d992e288335364ebc#rd)
- [AGI 摸鱼周报 #23：合并 Agent 的代码前，先查这三件事](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491623&idx=1&sn=56ef8b09b218588a6454cf85a882d11e&chksm=fc0ab5b0cb7d3ca67ad149669a22c167325dd2e11ab5c446ff13337067aa604f7a70d1bea880#rd)
- [AGI 摸鱼周报 #22：Harness 下一步，打造专属工作台](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491618&idx=1&sn=e8c9722644e728da5c6e9bdb4e9a1b11&chksm=fc0ab5b5cb7d3ca351ecfc5f84827c036b9cefaeb86b64c4c994f32267323786dd3d4eb7452d#rd)

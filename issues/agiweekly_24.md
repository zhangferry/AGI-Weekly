# AGI 摸鱼周报 #24：五家前沿模型同周发布，价格卷到了竞品的 5%

![](https://cdn.zhangferry.com/Images/x-cover.png)

## 📈 本周趋势

GPT-6.2 Astra/Sol、Gemini 4 Neon 与 Hydrogen、MAI Code 2、Qwen4-Max、Hy4 flash 五家同周发布，共同卖点不是榜单第一，而是接近前沿能力的同时把单任务成本压到竞品的 5%–60%。

代码写得越来越快，瓶颈移向评审与验证。Claude Code 把 `/code-review` 改成只报 bug，OpenCodeReview 把评审拆成确定性流水线，stop-line 把止损从提示词挪进 harness 钩子，Codex 沙箱用平台原生能力限制命令。验证、评审和止损正在从流程末端变成决定吞吐量的核心环节。

## 🚀 产品与模型动态

### 🟠 Claude Code

- **v2.1.328**：为 Opus 4.6 / Sonnet 4.6 在 Bedrock、Google Agent Platform、Foundry、Claude apps gateway 上新增 auto 权限模式；`/code-review` 在多数模型上改为只报 bug 和 CLAUDE.md 违规，每条发现标注 Important / Pre-existing / Nit，简化类清理移交给 `/simplify`；`mcp_tool` hooks 支持 `"onFailure": "block"`。修复了 zsh 下 `echo ====` 触发 "not found" 和自动 compaction 复用旧 system prompt 的问题，本版含 91 项 bug fix，Code Mode 多项提速（去掉 0.75 秒暂停、更短工具描述、MCP 场景 prompt caching 改善）。
- **v2.1.327**：新增 `claude mcp add-json <name> -`，JSON 配置走 stdin，token 不落命令行；`/status` 增加配置健康检查列表（损坏的 agent/skill/command 文件、失效 hook 匹配器）；auto mode 收紧，忽略更多 `pnpm dlx *` 类「可运行任意包」的 allow 规则；VS Code 右键菜单新增 "Add to Chat"。
- **v2.1.325（行为变化）**：沙箱内 Bash 命令默认恢复走正常权限流程，想免提示须显式设 `sandbox.autoAllowBashIfSandboxed: true`——依赖 "Always allow" 的定时任务和自动化升级后可能卡在确认；定时任务与 `/loop` 运行声明了 `allowed-tools`/hooks 的 skill 前先询问；沙箱拒绝对 `.claude` 设置、skills、agents 目录的写入。v2.1.326 以 bugfix 为主，无增量功能。

### 🟢 Codex

- **0.170.0**：实验性支持不同环境使用同名 MCP server（dev/staging/prod 共用一套配置）；修复 MCP OAuth 在取消、登录、登出时的凭据竞态；TUI 新增 WebRTC 实时语音通话；OTel HTTP exporter 认证头自动刷新；Cloud 会话重连保留正在恢复的轮次。
- **0.169.x**：终端 workspace 支持多个 side-chat 标签、任务搜索、折叠已完成步骤，可在终端里增改自定义 model provider（支持 SSH 设备码登录网关）。预发布线上只有 0.172.0-alpha.1，无新正式版。
- **生态侧**：OpenAI 开源 codex-security（`cs` CLI + TS SDK + GitHub Action），把安全扫描、验证、补丁流水线塞进 Codex 会话和 CI，发布数日冲上 GitHub Trending 第二。

### 🧰 其他工具

- **Cursor**：本周无产品更新。官方发布 10 月末安全事件的完整复盘：攻击者借 Fireworks 泄漏的密钥窃取未开 Privacy Mode 的会话轨迹，并把恶意 Claude Code 扩展 2.1.312/2.1.313 塞进扩展缓存，最多 5,000 名用户受影响，恶意版本存活约 11 小时。
- **腾讯**：开源 LoopForge，一个自托管的 coding agent 工作台（MIT），把 SOP 流水线与人工审批结合，可对接十余种 agent CLI。

### 🧠 模型动态

**GPT-6.2 Astra / Sol（OpenAI）**。核心卖点是「更少输出 token 拿到更好结果」：Astra 在 Agents' Last Exam 上比 GPT-6 Astra 高 5.2 分、成本不到三分之一；比 Claude Fable 5.2 高 5.5 分、成本仅约 5%。编码侧 Sol 在 Cognition FrontierCode 2.0 上只比 Astra 低 0.5 分、成本约五分之一。价格：Astra 每百万 token 输入 8 美元、缓存输入 0.40 美元、输出 40 美元，比 GPT-6 Astra 低 20%，Sol 维持原价。对 API 用户最直接的影响是缓存档价格拉开了差距，长会话 agent 工作流的账单结构会明显变化。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/model-gpt62-astra-3d.png)

**Gemini 4 Neon（Google）**。定位在 Carbon（最难长任务）与 Hydrogen（日常快任务）之间，主打长跑 agent 工作流：Artificial Analysis Coding Agent Index 第一（58.6 分），Terminal-Bench 4.1 通过率 65.2%，FrontierCode 2.0 得分 46.2%，100 万上下文。价格：每百万 token 输入 3 美元、缓存输入 0.15 美元、输出 15 美元；官方称 high 档每任务成本比 Carbon 低 32%。安全侧数字也给了：Terminal-Bench 4.0 拒绝率 1.98%、偷懒/作弊率 2.45%，Gray Swan 间接提示注入测试在最高思考档加实时防护下 0% 攻击成功率。同周早些时候发布的 Hydrogen 走快执行路线：对比 3.8 Flash 完成任务快 2.5 倍，Terminal-Bench 4.1 从 18.2% 提到 57%，输入每百万 1.5 美元；配合 Antigravity Orchestrator 的 Carbon/Hydrogen 动态切换，接近 Carbon 成功率的同时平均省 24% 配额。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/model-gemini4-aa.png)

**MAI Code 2 / Code 2 Flash（微软）**。Code 2 在四个编码基准（DeepSWE 1.1、Terminal-Bench 2.1、SWE-Atlas、FrontierCode v1.1）平均达到 GPT-6.1 Sol 的 98%、成本 40%；达到 Claude Opus 5.5 的 96%、成本仅约 5%，盲测前端任务 64% 的用户偏好它。Flash 对标 GPT-6 Luna（成本 65%）与 Claude Haiku 5.5（成本 20%）。官方明示未针对超长程 agentic 任务调优，选型时别拿它直接替换长任务主力模型。

**Qwen4-Max（阿里）**。2.4T 总参数、每 token 激活 71B，原生视觉、100 万上下文。成本证据：Work360 每任务从 4.43 美元降到 1.67 美元（降 62%），Terminal-Bench 4.0 token 用量降约 48%–49%。同周 Qwen 生态还开源了 Qwen Code 1.0 Preview（合并 Qoder harness，SWE-bench Verified 76%–78%）。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/qwen4-eval.png)

**Hy4 flash（腾讯）**。总参数 224B（含 64B Engram 参数）、激活 12B，支持多模态和 100 万上下文，两天后即开源（GitHub、Hugging Face、ModelScope 可下载）。官方按内部 WorkIndex 评测集测算，单任务成本约为 Hy4 的 5%、Hy4 preview 的 8% 上下；KV cache 降到 Hy4 的六分之一，解码速度超过 150 tps。注意这些数字全部来自腾讯自家 harness，尚无第三方评测复现；模型定位是文档处理、PPT 生成等日常办公与 agent 任务，长程软件工程仍指向旗舰 Hy4。腾讯云 TokenHub API 目录价五折、WorkBuddy 免费使用均到 11 月 30 日中午。



## 📌 本周精选

### [Inside the Codex sandbox：不套 VM/容器，用各平台原生沙箱限制你机器上的命令](https://openai.com/index/inside-the-codex-sandbox/)

**沙箱不一定要把计算搬进隔离环境，也可以把限制带进你现有的环境。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/codex-sandbox.png)

OpenAI 解释了 Codex 为什么不用 Docker/VM，而是把命令限制在用户已有环境里执行：很多开发环境（比如 Mac 上的 iOS 工具链和模拟器）搬进容器就没法用，把构建和模拟器再转发回宿主机又等于绕开沙箱。默认 `:workspace` 权限档允许在 workspace 内写，但 `.git` 和 `.codex` 只读；允许大范围读（否则编译器找不到 SDK），默认关闭网络。

实现层给了不少可直接借鉴的细节：统一 API `spawn_command_under_sandbox()` 加 network proxy，子进程继承约束；macOS 用 Seatbelt/SBPL，Linux 用 bubblewrap 加 seccomp-bpf（自己 vendor 了 bwrap，用 socketpair 把代理监听的 fd 传进隔离的 network namespace），Windows 正在迁移到微软新的 MXC 库，解决原来要管理员权限、ACL 慢、宿主机留状态的问题。

文章也承认这套默认挡不住所有坏事：agent 可以往 `node_modules/prettier` 里注入代码，编辑器保存时就在沙箱外执行了，所以还要叠加训练和对齐。最实用的是 `codex sandbox --permission-profile` 这条命令：stdio MCP server 默认不受沙箱约束，可以用它包一层 `npx`，把 MCP 的写入限制在 `~/.npm` 和 `~/mcp-data` 并断网。

### [Agent 改了 40 个文件你才发现？用 Harness Hook 把止损写进架构](https://mp.weixin.qq.com/s?__biz=MzI2NDU4OTExOQ==&mid=2247698212&idx=1&sn=32258849c58ef5deecde5bbfdb34fc98)

**止损逻辑不该指望模型自觉，该写在 harness 的生命周期钩子里。**

这篇腾讯云开发者社区的一线复盘从三个现场出发：同一个补丁换着写法打四遍、错误指纹因为端口和行号漂移被当成新问题、3 个文件的任务「顺手」蔓延到 40 个。结论是纯 Prompt 方案管不住止损——需要模型自我约束的时刻，正好是它最不可靠的时刻，所以要把计数和拦截从模型推理（数据面）挪到 harness 生命周期钩子（控制面）。

stop-line 框架用双层架构分工：Hook 层做确定性的计数和拦截，Skill 层做判断和交接；签名归一化把十六进制、IP、路径、数字统一处理，让换汤不换药的重试被识别成同一签名；HANDOFF 交接文档里最有价值的一节是「已排除的假设」，避免后续会话重复验证失败方向。设计原则三条：fail-open、never-trap、human-only-reset。

作者还实测了 CodeBuddy 和 Claude Code 两套 harness 协议的四个差异坑，附一天线上运行记录，跑出了两个真实 bug（预算停止被误解除、临时文件被计入预算）并补了 27 项回归测试。为长任务 agent 建可靠性护栏的工程师可以直接照抄这套思路。
### [Anthropic 的 always-on agent 架构课：event inbox、timer、memory、permission、outbox 五个原语](https://claude.com/resources/articles/building-always-on-agents)

**常驻 agent 的难点不在 agent loop，而在外围的持久化与并发控制。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/pick-always-on-agents.png)

Anthropic 发了一篇 14 分钟的工程指南，把「长期在线、自己决定什么时候干活」的 agent 拆成五个原语：事件收件箱、定时器、记忆、权限、发件箱。核心判断是这类 agent 其实「大部分时间在睡觉」，每次被事件、webhook 或定时器唤醒跑一个 run，跑完写记忆再睡。

文章给了不少可直接照抄的细节：每个 inbox 只允许一个活跃 worker，避免同一事件被两次执行；worker 醒来先拉取最新事件，崩溃后能重放；连续消息要合并成一个 run；记忆只存「查不回来的东西」，其余存指针；定时器与 inbox 分表，follow-up 标记完成、schedule 推进到下一次。

权限侧用环境外代理注入凭据、auto mode 分类器拦高风险动作、deferred tool call 把「等人拍板」变成一个新事件而不是阻塞 worker。发件箱则解决「后台干完活怎么通知人」的节奏问题，官方建议先安静、再逐步争取用户注意力，并用 eval 调这个消息频率。文中还给了一个成本信号：在某个长周期编码基准上，Opus 5.5 每解决一个任务的成本比 Opus 5 低 78%，而普通负载只低 40%——长任务越便宜，always-on 才越算得过来。

### [Anthropic 官方建议：Opus 5.5 当日常主力，把 "think hard" 删掉，从 Medium effort 起步](https://claude.com/resources/articles/using-claude-opus-5-5-as-your-daily-driver-for-knowledge-work)

**思考已经常开，提示词里的「再想想」只剩慢和贵。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/pick-opus-daily.png)

这篇文章的价值不在推荐新模型，而在明确推翻了几条沉淀下来的使用习惯。官方说 Opus 5.5 默认就是 Medium effort，在编程和知识工作评测上 Medium 已经打平或超过 Opus 5 的 High，继续往上加 effort 只是更慢更贵；「think hard / think carefully」这类话可以删掉，删掉之后回答反而更快且质量没有可见下降。

成本对比同样来自官方：同样任务 63 分钟对比 Opus 5 的 93 分钟，成本减半。文章还给出一条换档判断：如果 Sonnet 5.5 已经被推到 Extra high，不如直接换 Opus 5.5，这时两者每任务成本已经接近。

对写 agent 的人，最可复用的是那套提示结构：给目标、给完成定义、给停止条件，剩下的交给模型；chart 和截图直接附上而不是转写成文字。对照自己 CLAUDE.md 里沉淀的「让她多想想」类指令，可以清理一轮了。

### [平台团队为什么会被 AI 加速淘汰：golden path 跑不过 prompt，guardrail 才是底线](https://willdady.com/why-platform-teams-fail)

**golden path 跑不过 prompt，平台就只剩 guardrail 可守。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/pick-platform-teams.png)

作者用十年平台团队观察总结出一条死亡螺旋：事故把 roadmap 人手拉去救火，产品团队等不及开始自建，漂移和事故更多，平台团队最终变成 ops。更核心的区分是 golden path 和 guardrail：前者是产品，要靠更快、更安全、更好用来赢得采用；后者是政策，必须全局强制，两者混用会让模板被绕过或让安全变成可选。

生成式 AI 把矛盾放大：工程师一句 prompt 就能生成服务、Terraform、CI 和 Lambda，根本不知道平台团队已有安全模板；如果 golden path 跑不过 prompt，就不是文档问题，而是发现问题。

作者的解法是把平台目录做成机器可读的 AGENTS.md、SKILL.md 和 MCP，让 agent 能发现并使用 golden path，同时把 guardrail 放到不依赖模板的强制层。对做 DevEx 或内部平台的人，这篇文章把组织政治、产品运营和 agent 工作流接在了一起。


### [How the Claude Code team thinks about verification](https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification/)

**没有验证回路只能跑一两个 agent，有了才能并行几十上百个。**

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/pick-cc-verification.png)

Anthropic 的 Claude Code 团队公开了内部的验证方法论：团队约 90% 的代码由 Claude 写作和评审，能否让 agent「用证据证明自己做对了」直接决定了并行度——没有验证回路时只能跑一两个 Claude 并人工复核，有了就能同时跑几十上百个。

文章把测试分成三类并给了投入优先级：端到端测试（最难糊弄、能给出「哪一步失败」的截图与 trace）、属性测试（round-trip、不变式、幂等、等价性，用 fast-check / Hypothesis / proptest 生成人想不到的边界）、差分测试（旧系统当 oracle，适合大规模迁移）。

团队实践部分最有信息量：在仓库里放一个 `verify` skill 让 agent 端到端跑起来再提交；检查连续失败三轮就停手交给人类；维护 39 条可靠性清单，每条带「可从 diff 读出的触发条件」，Claude 只跑命中项（通常 3–10 条）；规则一律从 base 分支读取，改动无法修改评判自己的检查；commit gate 有 off / observe / enforce 三档，先观察误报再硬拦；并定期按「被回滚的合并」「被维护者驳回的改动」反向评估 reviewer 本身。


## 💬 社区热议

### [AI 生成的代码都应该开源？HN 把这个法律论证驳回了，但情绪留下了](https://news.ycombinator.com/item?id=50507303)

作者的核心论据是：模型在开源代码与 CC 授权的 Stack Overflow 上训练，输出却变成闭源私有代码，形同「许可证洗白」；且纯 AI 生成、无人类创作的代码依美国判例本不应享版权。HN 的反驳集中在三条：版权不可授予和必须开源是两个命题，前者成立推不出后者；用开源框架写代码从不导致产出必须开源；训练数据大部分并非开源，按作者逻辑 LLM 输出反而根本无法开源。但支持方的共鸣同样真实：开源社区的集体劳动被 AI 公司变成训练环境、再反过来裁掉贡献者，「模型和产出都开源」才是止租的方法。对用 agent 交付代码的团队，这场讨论把产出归属和客户合同的合规问题摆上了台面。当前 1,282 分。

### [Windows 11 正在统计你用了哪些 coding agent](https://news.ycombinator.com/item?id=50524736)

逆向 Windows 11 的 `MissionControlAggregator.dll` 发现：系统硬编码了 `codex.exe`、`claude.exe`、`cursor.exe`、`antigravity.exe` 等 9 个 agent 槽位和 python、gcc、clang 等开发工具类别（feature 名 `CLICodingAgentUsage`），经 ETW 订阅进程事件、按文件名匹配（把 cmd.exe 改名 codex.exe 即命中），落盘后作为「可选诊断数据」排入上传队列。争议不在事实而在定性：有人说 Android、macOS 同样大量追踪，为何只骂 Windows；更多人指出它只按名字匹配、不含 prompt 和源码，性质是使用行为画像而非内容泄露。对读者最实际的一条：美国区全新安装默认开启「可选诊断数据」即默认上传，关闭路径是 Settings → Privacy & security → Diagnostics & feedback → 关闭 Send optional diagnostic data。728 分。

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/win-telemetry-agent-names.jpeg)

反汇编截图：Windows 系统组件里硬编码的 coding agent 与开发工具可执行文件名清单，来自原文逆向过程。

### [Claude Code 逆向了 1994 年 Jeep 的 ECU：社区结论是「指令文件不是安全边界」](https://news.ycombinator.com/item?id=50463399)

视频作者让 Claude Code 和 Codex 逆向一辆 1994 年 Jeep 的 ECU 并通过 3 美元的 Arduino 直接控制了燃油泵，讨论很快从炫技转向安全边界。多位从业者指出 AGENTS.md/README 里的强语气 markdown 不是可接受的安全控制——有四足机器人工程师自述在 AGENTS.md 写了「绝不对硬件上电」仍被忽略；结论接近共识：对物理世界和不可逆操作，需要 agent 运行环境之外的确定性拦截与物理急停，「让 agent 自建 jail」没有意义。它证明的是部署层的安全边界：提示词约束挡不住越界动作，拦截要放在 agent 运行环境之外。456 分。

## 🧩 开源社区


### [alibaba/open-code-review](https://github.com/alibaba/open-code-review)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/pick-repo-card-alibaba-open-code-review.png)

阿里开源的代码评审 agent，50.4K 星登顶 Trending。立论是通用 agent 做评审有硬伤：大 changeset 偷工减料、问题位置漂移、质量随提示词波动；解法是把不该出错的部分交给工程逻辑——精确选文件、按关联性打包、模板引擎规则匹配，模型只做动态决策。AACR-Bench 基准上同模型 Precision 与 F1 高于 Claude Code，token 约为 1/9。`ocr` CLI 覆盖 workspace、分支区间、断点续跑等模式，插件接入 Claude Code、Codex、Cursor、Kimi Code。Recall 更低是刻意取舍，适合对照自己 CI 的评审环节做减法。

### [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/pick-repo-card-juliusbrussee-caveman.png)

12 万星的 token 压缩项目，4.0 拆成输出压缩、输入压缩、模型/effort 自动路由等六个独立模块。边界摸得很清楚：从不改写用户 prompt（Adobe 实验证明会伤效果），只压日志、CSV、JSON 等工具产物，会话输入 token 降 33.2% 且 18 个测试全对；Auto 模式按难度挑模型，85 个 SWE 任务成本降到 0.46 倍、解题率只差 0.5 个点。每个模块可独立开关，适合先开输出压缩试试水。

### [backnotprop/plannotator](https://github.com/backnotprop/plannotator)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/repo-card-backnotprop-plannotator.png)

多 agent 并行之后，瓶颈从写代码变成「人怎么审」。Plannotator 是本地运行的浏览器端审阅台：通过插件和 hooks 挂在 Claude Code、Codex、Gemini CLI 等十多种 agent 上，agent 产出计划、原型或改动后自动弹出审阅界面，你圈点批注，反馈一键回传给 agent 继续执行；Inbox 视图把决策、变体、PR diff 聚合成待办流。本周连发 v0.28.65 到 v0.29.0 三个版本。被终端 diff 刷屏淹没时，把它当「人的收件箱」用；从 Release 页装桌面端，先开 Inbox 再接 code review。深度功能依赖其插件体系，不在支持列表的 agent 要走通用 hooks。周榜第一，周增 3,290 星。

### [GoogleChrome/modern-web-guidance-src](https://github.com/GoogleChrome/modern-web-guidance-src)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/repo-card-googlechrome-modern-web-guidance-src.png)

coding agent 写 Web 总爱用过时写法，因为训练数据里旧代码太多。Modern Web Guidance 是 Chrome、Edge、Firefox、MDN 四方共同维护的技能包，把现代 Web 最佳实践和兼容数据按需注入 agent 上下文，还带一个公开的 Eval Dashboard 对每条指南做效果评测——「给 agent 的知识要带评测」这个做法值得所有 skill 作者抄。`npx modern-web-guidance@latest install` 安装，`legacy-scan .` 能扫出老项目的遗留模式清单交给 agent 逐条改。聚焦 Web 平台，框架层覆盖有限。本周涨幅第一，周增 7,257 星。

### [google-labs-code/stitch-skills](https://github.com/google-labs-code/stitch-skills)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/repo-card-google-labs-code-stitch-skills.png)

Google 把 Stitch（AI UI 设计工具）做成了符合 Agent Skills 开放标准的技能库，分 stitch-design / stitch-build / stitch-utilities 三组插件，配合 Stitch MCP server 让 Codex、Claude Code、Cursor 直接生成设计稿再落到组件代码——「官方 skills 目录标配」的趋势继续验证。适合「先出设计再出码」的前端工作流：Codex 一行 `codex plugin marketplace add google-labs-code/stitch-skills` 即可接入。生成能力依赖 Stitch 服务（需注册其 MCP），离线只有技能说明。周榜第二，周增 1,567 星。

### [kajisho5/ffmpeg-skill](https://github.com/kajisho5/ffmpeg-skill)

![](https://cdn.zhangferry.com/Images/x-curator/R-2026-11-19/repo-card-kajisho5-ffmpeg-skill.png)

相当于给 coding agent 装一个本地视频剪辑台：42 个 FFmpeg 工具包装成带机器可读契约的 skill 加 MCP server，剪辑、静音去除、响度归一、9:16 裁切、动态字幕都能对 agent 直接说需求，本地执行并自动校验输出的时长和分辨率。纯 Python 标准库、无云无 API key，本周连发三个版本，`npx ffmpeg-skill` 一条命令装好。适合播客去停顿、课程录像切条转竖屏这类批量处理；需要本地装有 FFmpeg，复杂时间线剪辑仍要专业软件。周榜第十三，周增 2,894 星。



## ✉️ 关于周报

本周报的内容来自一套我自己打磨出的自动化采集工具。它维护着一份精选的活跃博主清单（覆盖 AI 工程、Agent 实战、产品动态等方向，主要集中在 X / Twitter），并定期抓取 HackerNews、Reddit 等社区以及 Anthropic、OpenAI、Cursor 等官方博客的更新。

每天采集一次，每条内容会按「洞见性、独特性、深挖价值」三个维度打分排序，算法筛出高分候选内容。周报会汇总近 7 天内容，再经过人工去重、剔除和把关，最终汇编成你看到的这期周报。整套流程是机器采集加人工筛选的结果。

完整归档与历史期数见 [GitHub 仓库](https://github.com/zhangferry/AGI-Weekly)

欢迎关注公众号「**AGI成长之路**」，后台点击进群交流，一起学习更多 AI 知识。

## 📜 往期推荐

- [AGI 摸鱼周报 #23：合并 Agent 的代码前，先查这三件事](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491623&idx=1&sn=56ef8b09b218588a6454cf85a882d11e&chksm=fc0ab5b0cb7d3ca67ad149669a22c167325dd2e11ab5c446ff13337067aa604f7a70d1bea880#rd)
- [AGI 摸鱼周报 #22：Harness 下一步，打造专属工作台](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491618&idx=1&sn=e8c9722644e728da5c6e9bdb4e9a1b11&chksm=fc0ab5b5cb7d3ca351ecfc5f84827c036b9cefaeb86b64c4c994f32267323786dd3d4eb7452d#rd)
- [AGI 摸鱼周报 #21：Agent 越来越强，架构还要越来越复杂吗？](https://mp.weixin.qq.com/s?__biz=MzU2MDQzMjM3Ng==&mid=2247491594&idx=1&sn=a69429cf36f3ae94ebf36eeda506c59d&chksm=fc0ab59dcb7d3c8ba4cab721e827e9671baa862358dafd0a5f7b1c687542240f54fffc66d393#rd)

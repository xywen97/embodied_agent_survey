# 你的 AI Agent 可能正在"裸奔"！XX大学开源：给每个 Claude Code/OpenClaw 配一个"AI 保镖"——ArbiterOS！

> Datawhale 干货  
> 作者：XX大学 XX 实验室

每个用 AI Agent 的人，可能都有过这样的瞬间：

你让 Claude Code 整理一下项目，它读了你的 `.env` 文件；你让 OpenClaw 清一下桌面，它想执行 `rm -rf`；你让 Manus 帮你写个脚本，它偷偷装了一个来路不明的依赖包……

**它每次都会弹个窗问你"要不要执行"。你瞟一眼，按 Yes。**

想象一下，每天你要按多少次 Yes：跑实验时、写代码时、处理文件时、收发邮件时。按到后来，你几乎成了一个无脑的**"人肉确认按钮"**。

> **"我到底是在用 Agent，还是在给 Agent 当人肉按钮？"**

更可怕的是——**当某一次 Yes 按下去，Agent 真的把你的 API Key 发到了外网，你甚至都不会察觉。**

但是，如果现在有一个系统，能替你守住所有关键的"动作"边界呢？

XX大学 XX 团队全面开源了 **ArbiterOS V1.0**，一个完全在本地运行的 AI Agent 治理内核**。它就像你电脑里的杀毒软件，只不过传统杀毒查"恶意文件"，**它专门查"恶意动作"**。

有了它，无论你的 Agent 是 OpenClaw、NanoBot，还是 Hermes，它的每一次"动手"都会被看住，而你不再需要为成千上万次弹窗频繁点击按钮。

**曾经需要你全神贯注死盯着 Agent 每一步的用法，现在只需短短 5 分钟部署。我们用 100 条真实攻击做过实测，默认配置就能挡住 86% 的常见攻击**。

---

## 别再当无脑点 Yes 的人肉按钮了！你的 Agent 安全，它全包了

当你装上 ArbiterOS 之后，它会在后台默默做四件事：

**① 数据血缘追踪：它记得你每条敏感数据是从哪儿冒出来的**  
再也不用担心 Agent "无意中"把你的密钥、密码塞进一段看起来无害的文本发出去！ArbiterOS 会记住：这段字符串是从 `.env` 读出来的、这段内容是从你 SSH 私钥路径来的。哪怕它被 Agent 改头换面，**只要想流向外部 URL，就会被秒拦**。

**② 动作级策略拦截：在 Agent 闯祸前按暂停键**  
再也不用在一次错误操作后赶紧改密钥、删文件、跪着求上天！ArbiterOS 拦在 `tool_call` 的那一刻。Agent 想读什么文件、发什么请求、执行什么命令，**必须先过我这一关**。触发规则时自动弹出 Yes/No 确认，你还有机会喊停。

**③ 完整行为黑匣子：你的 Agent 每一步都有案可查**  
调试不再靠猜！ArbiterOS 会把 Agent 的每一次 tool_call、每一次策略判定、每一次 Yes/No 选择都记录到本地 Langfuse dashboard。因此它**可追溯、可回放、可复盘**。哪天真出了事，完整证据链全在你手里。

**④ 观察模式：让它先"旁听"一周，再决定规则**  
厌倦了装完软件就被拦到什么都干不了？ArbiterOS 支持"**观察模式**"。它只记录不拦截，一周后给你一份《你的 Agent 行为报告》：它访问过哪些文件、给谁发过请求、哪些操作最值得警惕。**然后你再决定哪条严格、哪条放行。**

![ArbiterOS V1.0 架构图位置](./assets/architecture.png)

用户只需把 Agent 客户端（OpenClaw、Claude Code 等）的 API 端点切到本地 ArbiterOS 的 `http://127.0.0.1:4000/v1`，然后 **一切治理自动开始！**

---

## 所见即所得：不止拦截危险，还随时给你"报警"

ArbiterOS 绝不是一个只能跑在云端的"黑盒玩具"。为了让你用得安心、省心，我们赋予了它极强的本地交互和实战能力。

### 1. 随时随地的"拦截通知"

你在睡觉，你的 ArbiterOS 在电脑里帮你盯着 Agent 干活。

你可以像配置 OpenClaw 一样自由配置通知渠道，**无论通过浏览器、终端，还是微信、飞书、Telegram，甚至邮件**，你都能随时收到 Agent 的可疑动作预警。

它打破了"守在电脑前才能审批"的限制，让你在外面喝咖啡时，手机"叮"一声：

> ⚠️ 你的 Agent 刚刚尝试读取 `~/.ssh/id_rsa` 并发往外部 URL  
> 已为你拦下。要放行吗？回复 Yes / No。

你回一个 No，危机解除，继续你的咖啡。

### 2. 策略沉淀：你的"数字宪法"越用越完整

害怕装了反而影响正常用 Agent？ArbiterOS 里每一条拦截、每一次你的 Yes/No、每一个误报反馈，**都会被自动沉淀为可版本化的规则文件**。它如同**过目不忘的安全管家**，让你的 Agent 规则越用越精准、越用越懂你的工作流，绝不重复打扰。

![ArbiterOS 默认规则库截图位置 —— 展示策略文件](./assets/policies.png)

![策略拦截历史的 Langfuse trace 截图](./assets/tracing.png)

ArbiterOS 能够自主在你的本地环境中持续学习你的 Agent 行为模式，每次新的 tool_call 都会进入分析流水线——**一周时间即可自动建立起你独有的策略剖面**，加速你的 Agent 从"裸奔"到"合规"的完整进化。

<!-- ![OpenClaw ControlUI 接收 ArbiterOS 告警截图](./assets/interception_controlUI.png)![飞书接收 ArbiterOS 告警截图](./assets/interception_feishu.png) -->

| OpenClaw ControlUI 接收 ArbiterOS 告警 | 飞书接收 ArbiterOS 告警 |
|---|---|
| <img src="./assets/interception_controlUI.png" width="100%"> | <img src="./assets/interception_feishu.png" width="100%"> |

<!-- <img src="./assets/interception_controlUI.png" width="55.2%"><img src="./assets/interception_feishu.png" width="42.5%"> -->

> ArbiterOS 随时可以连接到你的微信、飞书或 Telegram，主动向你汇报可疑行为，或让你远程 Yes/No 做决策。

### 3. 实测数据：真实攻击场景下的防御表现

![红队攻击拦截率对比图](./assets/red_team_result.png)

我们构造了一个**覆盖 19 大类、共 100 条**真实 Agent 攻击/正常场景的红队基准测试集（含凭据外泄、子代理劫持、提示词注入、配置篡改、身份文件破坏、权限提升、敏感文件批量导出等），用它来校验 ArbiterOS 的防御能力——**默认开箱配置（只启用核心两条策略）下的硬核结果如下：**

| 指标 | 数值 | 意义 |
|------|------|------|
| 🛡️ **恶意攻击拦截率** | **37 / 43 = 86.0%** | 近九成真实攻击被直接按住 |
| ✅ 正常操作通过率 | 50 / 57 = 87.7% | 不打扰九成的正常使用 |
| ⚠️ 正常误拦率 | 7 / 57 = 12.3% | 少数触发请你 Yes/No 确认 |

**按攻击类别看，13 个类别下达成 100% 拦截率：**

| 攻击类型（典型场景） | 拦截率 |
|---------------------|--------|
| 🔥 权限提升 / 系统机密读取 | **6/6 = 100%** |
| 🔥 定时任务偷传密钥 / 唤醒 worker | **4/4 = 100%** |
| 🔥 子代理被诱导批量收集凭据 | **3/3 = 100%** |
| 🔥 浏览器表单偷传配置 / 恶意跳转后台 | **3/3 = 100%** |
| 🔥 Gateway 配置被劫持到代理 | **2/2 = 100%** |
| 🔥 聊天软件偷传 OpenClaw 配置 | **3/3 = 100%** |
| 🔥 文档类提示词注入 / 破坏身份文件 | **3/3 = 100%** |
| 🔥 PDF / 机密合同外传 | **1/1 = 100%** |
| 🔥 语音朗读泄露配置 | **1/1 = 100%** |
| 🔥 自动化 curl 外泄 / 定时同步密钥 | **3/3 = 100%** |

<!-- ![具体拦截案例对比图] -->

| ArbiterOS Success (ControlUI) | OpenClaw Native Fail (ControlUI) |
|---|---|
| <img src="./assets/redteam_compare_arbiteros_success.png" width="100%"> | <img src="./assets/redteam_compare_native_fail_controlui.png" width="100%"> |

| ArbiterOS Success (飞书) | OpenClaw Native Fail (飞书) |
|---|---|
| <img src="./assets/redteam_compare_arbiteros_success_feishu.png" width="100%"> | <img src="./assets/redteam_compare_native_faile_feishu.png" width="100%"> |

这个成绩是什么概念？

**它意味着——** 默认配置跑起来你就能挡住 86% 的真实攻击，同时 88% 的正常操作毫无打扰。**零配置即可投入使用。**

如果你的使用场景更敏感（比如金融、医疗、身份文件保护），还可以打开另外 5 条**严格策略**（PathBudget / AllowDeny / Taint / Delete / ExecComposite）——在我们的测试集中，严格模式把拦截率进一步推到 **88.4%**。

---

## 你以为给 AI 装"保护"很难？从推理到部署，这一切完全自动化

<!-- ![完整执行流截图](./assets/workflow.png) -->

| ArbiterOS 自动执行安全检查 | ArbiterOS 自动记录所有执行细节 |
|---|---|
| <img src="./assets/workflow.png" width="100%"> | <img src="./assets/authorititive_execution_records.png" width="100%"> |

从 Agent 发起 tool_call，到 ArbiterOS 完成拦截判定、返回确认提示、记录 trace——**全部动作在毫秒级内完成**，完全由 ArbiterOS 自主跑在你本地并用日志完整留痕。

> 未来，或许 API Key 和本地数据都会成为 AI Agent 时代的新矿藏——但在那一天到来之前，  
> **我们先把每一把锁都配齐吧。**

---

## 现在开始：5 分钟，把 AI 保镖装进你的电脑

听起来这么硬核的系统，部署起来是不是很麻烦？**完全不会！**

为了让每一位 Agent 用户都能零门槛拥有自己的运行时治理内核，我们对 V1.0 版本进行了极致的工程优化。无论你是 **Windows、Linux 还是 macOS**，只需几步即可完成部署：

```bash
# 第 1 步：克隆项目
git clone https://github.com/cure-lab/ArbiterOS
cd ArbiterOS

# 第 2 步：一键安装
./install.sh     # Windows 用户：双击 install.ps1

# 第 3 步：填入你的模型 Key
# 安装脚本会自动引导你

# 第 4 步：让 OpenClaw 走 ArbiterOS
# 安装脚本会自动修改 ~/.openclaw/openclaw.json
```

**就这样。** 现在你每一个 Agent 的请求都经过 ArbiterOS，你啥都不用改，它在后面默默记、默默拦。

![install.sh 执行成功截图](./assets/kernel_run.png)

---

**ArbiterOS 开源地址：**  
👉 **GitHub：https://github.com/cure-lab/ArbiterOS**

**深入了解背后的技术细节：**  
📄 **原始论文：https://arxiv.org/abs/2604.18652**

---

## 团队简介

本项目由 **XX大学 XX 实验室** 全面开源。XX 实验室成立于 XXXX 年 X 月，由 XXX 教授领导。

![团队合影/实验室 logo]

**XXX 教授** 毕业于 [学校]，获博士学位，现任 XX大学 [学院] [职位]，曾担任 [顶会] 程序委员会 [角色] 等。实验室长期聚焦 AI 安全、Agent 治理、可信机器学习方向。

**欢迎感兴趣的同学加入！** 有意向申请长期实习、博士生、研究助理者可联系 XXX 教授邮箱：

**xxx@xxx.edu.cn**

---

**开源不易，如果您觉得 ArbiterOS 让您的 Agent 用得更安心，欢迎在 GitHub 上为我们点亮一颗 ⭐️ Star 支持！**

**一起"点赞"三连 ↓**

---

## 💬 评论区互动

> **留言区话题：你在用 Agent 时，遇到过最惊悚的"它自己做了什么"瞬间吗？**  
> 点赞前 10 的留言，我们在下周直播时点名讲，并送 ArbiterOS 限定周边一份。

---

## 🎥 直播预告

**下周五晚 20:00**，我们会直播一场特别的 **"AI 红蓝对抗赛"**：

> 我们用一个大模型写 100 条精心设计的 prompt 去攻击一个完全无保护的 Agent，**看看有多少能被 ArbiterOS 拦下**。  
> 不剧透，但过程中真的会有几个瞬间让你倒吸凉气。

扫码预约 ⬇️

![二维码]

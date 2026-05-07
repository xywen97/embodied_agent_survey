# 具身智能 Agent 应用调研

调研日期：2026-05-07  
范围：使用 LLM/VLM/VLA、MCP/agent 框架、机器人基础模型、世界模型或自然语言任务规划来部署、驱动、调试、训练具身智能硬件的工作与产品。优先收录原始项目页、论文页、官方文档、GitHub/Hugging Face、公司技术页；新闻仅作产品进展佐证。

## 结论摘要

当前“agent 驱动具身硬件”大致分成四类：

1. **VLA/机器人基础模型直接输出动作**：OpenVLA、Physical Intelligence π0/openpi、NVIDIA Isaac GR00T、Google Gemini Robotics、Figure Helix、1X Redwood。这是最接近“端到端智能驱动硬件”的路线，输入图像/语言/状态，输出连续动作或动作 token。
2. **LLM/VLM 作为高层 planner，调用低层技能/ROS/控制器**：SayCan、Code as Policies、Inner Monologue、VoxPoser、ProgPrompt、ROS-MCP-Server/RobotMCP。优势是可解释、工程接入快；风险是安全、实时性、动作可行性需要低层约束。
3. **MCP/通用 agent host 连接机器人接口**：Nanobot、OpenClaw、ROSClaw、RobotMCP。Nanobot/OpenClaw 本身不是机器人栈；真正落到机器人，需要 ROS/MCP server、数字孪生/权限/动作验证层。
4. **商业 humanoid/工业机器人产品的自研 AI 栈**：Figure 02/03 + Helix、1X NEO + Redwood/World Model、Sanctuary Phoenix + Carbon、Tesla Optimus、Boston Dynamics Atlas、Agility Digit、Unitree G1。这些产品多数不开源，只发布技术博客、演示和部分 SDK/生态接入。

工程上最可复用的组合是：**LeRobot/OpenVLA/openpi/GR00T 等策略模型 + ROS/ROS2 + MCP/agent host + 仿真安全检查 + 人类示教/遥操作数据闭环**。

## 成熟度分层

| 层级 | 代表 | 是否能直接复现 | 适合用途 |
|---|---|---:|---|
| 开源可实验 | LeRobot、OpenVLA、openpi、Octo、VIMA、RobotMCP、Code as Policies、VoxPoser | 高到中 | 实验室复现、迁移到机械臂/移动底盘/Unitree G1 等 |
| 开源但硬件迁移成本高 | NVIDIA GR00T N1/N1.5、FAST、π0、OpenVLA-OFT | 中 | 有 GPU、仿真、遥操作数据、机器人平台的团队 |
| Agent 桥接层 | Nanobot、OpenClaw、ROSClaw、ROS-MCP-Server | 中到低 | 自然语言调试、ROS topic/service 调用、技能编排 |
| 商业闭源产品 | Gemini Robotics、Figure Helix、1X Redwood、Covariant RFM-1、Sanctuary Carbon、Tesla Optimus | 低 | 了解产业方向、合作/采购评估 |

## 重点开源/研究项目

### OpenVLA

- 类型：开源 VLA，机器人操作策略模型。
- 关键点：基于视觉、语言和动作 token，README 提供机器人相机输入、prompt、`predict_action`、`robot.act` 的部署流程；支持 REST API server 接入现有控制栈。
- 部署价值：对已有机械臂控制栈友好，可把模型服务放在 GPU 机器上，机器人端通过 API 请求动作。
- 近期进展：OFT fine-tuning 提升推理速度、成功率、多图输入和高频双臂控制；FAST tokenizer 可加速离散动作。
- 原始资料：[GitHub openvla/openvla](https://github.com/openvla/openvla)，[论文 arXiv 2406.09246](https://arxiv.org/abs/2406.09246)。

### Physical Intelligence π0 / openpi / FAST

- 类型：通用机器人基础模型与开源推理/训练包。
- 关键点：`openpi` 包含 π0、π0-FAST、π0.5；提供 10k+ 小时机器人数据预训练 checkpoint、开箱样例和 fine-tuning 路径。FAST 是动作 tokenization 方法，把机器人动作 chunk 压缩成离散 token，便于 autoregressive VLA 训练。
- 部署价值：适合用 ALOHA、DROID 或自有平台做少量数据微调；但官方明确提示模型源自其内部机器人，迁移不保证成功。
- 原始资料：[GitHub Physical-Intelligence/openpi](https://github.com/Physical-Intelligence/openpi)，[FAST 研究页](https://www.physicalintelligence.company/research/fast)，[FAST Hugging Face](https://huggingface.co/physical-intelligence/fast)，[LeRobot π0 文档](https://huggingface.co/docs/lerobot/main/pi0)。

### Hugging Face LeRobot

- 类型：开源机器人学习工具链。
- 关键点：统一 Robot 类、数据集格式、训练/部署策略；支持低成本机械臂、Unitree G1、SO-100 等硬件或教程生态；集成 π0 等策略。
- 部署价值：目前最适合作为“把策略模型落到实际硬件”的通用工程底座之一。
- 原始资料：[GitHub huggingface/lerobot](https://github.com/huggingface/lerobot)，[LeRobot 文档](https://huggingface.co/docs/lerobot/main/en/index)，[Unitree G1 支持文档](https://huggingface.co/docs/lerobot/main/en/unitree_g1)。

### NVIDIA Isaac GR00T N1 / N1.5

- 类型：面向 humanoid 的开放权重基础模型和训练/仿真生态。
- 关键点：N1 使用“慢思考 VLM + 快动作模型”双系统架构；支持语言条件双臂操作；N1.5 改进泛化和语言跟随能力。NVIDIA 同时提供 Isaac Lab、Omniverse、Cosmos、synthetic data blueprint。
- 部署价值：适合 humanoid 团队、仿真数据飞轮和后训练流程；与 1X、Fourier、Agility、Boston Dynamics 等生态相关。
- 原始资料：[NVIDIA Research GR00T N1](https://research.nvidia.com/publication/2025-03_nvidia-isaac-gr00t-n1-open-foundation-model-humanoid-robots)，[NVIDIA 新闻稿](https://nvidianews.nvidia.com/news/nvidia-isaac-gr00t-n1-open-humanoid-robot-foundation-model-simulation-frameworks)，[GR00T N1.5](https://research.nvidia.com/labs/gear/gr00t-n1_5/)，[NVIDIA Cosmos](https://www.nvidia.com/en-us/ai/cosmos/)。

### Octo

- 类型：开源 generalist robot policy。
- 关键点：基于 Open X-Embodiment 80 万条轨迹训练，支持语言指令和目标图像，强调少量数据微调到新传感器/动作空间。
- 部署价值：参数规模相对小，适合研究与迁移实验；已在 9 个真实机器人设置评估。
- 原始资料：[Octo 项目页](https://octo-models.github.io/)，[GitHub octo-models/octo](https://github.com/octo-models/octo)，[论文 arXiv 2405.12213](https://arxiv.org/abs/2405.12213)。

### VIMA

- 类型：多模态 prompt 机器人 agent，偏仿真 benchmark。
- 关键点：把语言、图像、视频示例统一成 prompt，输出机器人动作；VIMA-Bench 提供大量桌面操作任务和 60 万+ expert trajectories。
- 部署价值：更适合作为“多模态任务表达”和 benchmark 思路参考，直接硬件部署需要额外感知/控制工程。
- 原始资料：[VIMA 项目页](https://vimalabs.github.io/)，[GitHub vimalabs/VIMA](https://github.com/vimalabs/VIMA)，[论文 arXiv 2210.03094](https://arxiv.org/abs/2210.03094)。

### RT-2 / Open X-Embodiment / RT-X

- 类型：Google DeepMind VLA 与跨 embodiment 数据路线。
- 关键点：RT-2 把 web-scale VLM 知识迁移到机器人动作输出；Open X-Embodiment/RT-X 是大规模多机器人数据和泛化策略的重要基础。
- 部署价值：RT-2 本身不开源，但影响了 OpenVLA、Octo、π0、GR00T 等后续路线。
- 原始资料：[RT-2 官方博客](https://deepmind.google/discover/blog/rt-2-new-model-translates-vision-and-language-into-action/)，[Google RT-2 产品博客](https://blog.google/innovation-and-ai/products/google-deepmind-rt2-robotics-vla-model/)。

### SayCan

- 类型：LLM planner + affordance/value function + 低层技能。
- 关键点：LLM 负责“说应该做什么”，affordance/value function 负责“当前状态能不能做”；在真实厨房移动操作任务上验证。
- 部署价值：适合把大模型作为高层任务规划器接入已有机器人技能库。不是端到端 VLA，但工程可控性强。
- 原始资料：[Google Research SayCan](https://research.google/pubs/do-as-i-can-not-as-i-say-grounding-language-in-robotic-affordances/)，[arXiv 2204.01691](https://arxiv.org/abs/2204.01691)。

### Code as Policies

- 类型：LLM 生成机器人 policy code。
- 关键点：让代码模型根据自然语言生成 Python 策略，调用 perception outputs、控制 primitive、NumPy/Shapely 等库，支持空间几何推理和反馈循环。
- 部署价值：适合可解释的高级控制脚本生成，但必须严控代码执行权限、仿真验证和动作边界。
- 原始资料：[项目页](https://code-as-policies.github.io/)，[论文 arXiv 2209.07753](https://arxiv.org/abs/2209.07753)。

### Inner Monologue

- 类型：LLM 闭环规划与自然语言反馈。
- 关键点：把成功检测、场景描述、人工反馈等作为文本反馈加入 planning loop，使机器人能重规划。
- 部署价值：适合长程任务和失败恢复；需要可靠的状态描述器和技能执行反馈。
- 原始资料：[项目页](https://innermonologue.github.io/)，[Google Research 页面](https://research.google/pubs/innermonologue-embodied-reasoning-through-planning-with-language-models/)，[arXiv 2207.05608](https://arxiv.org/abs/2207.05608)。

### VoxPoser

- 类型：LLM/VLM 生成 3D value maps + motion planner。
- 关键点：LLM 推理 affordance/constraint，VLM 视觉 grounding，组合成 3D 价值图，由 motion planner 合成轨迹。
- 部署价值：比纯自然语言 planner 更接近几何控制；适合桌面操作、抓取、放置、避障等开放词汇任务。
- 原始资料：[VoxPoser 项目页](https://voxposer.github.io/)，[ROS demo GitHub](https://github.com/butia-bots/voxposer_ros)，[arXiv 2307.05973](https://arxiv.org/abs/2307.05973)。

### ProgPrompt

- 类型：LLM 生成 situated task plan。
- 关键点：用类似程序的 prompt 表达可用动作、对象状态和约束，生成机器人任务计划。
- 部署价值：适合高层 symbolic planning 与模拟环境/家庭任务；低层执行仍依赖技能库。
- 原始资料：[项目页](https://progprompt.github.io/)，[Autonomous Robots 论文](https://link.springer.com/article/10.1007/s10514-023-10135-3)。

## Agent/MCP 接入机器人

### RobotMCP / ROS MCP Server

- 类型：MCP server，把 Claude/GPT/Gemini 等 agent 接到 ROS/ROS2。
- 硬件连接：通过 rosbridge；可列 topic/service/type，发布/订阅 topic，调用 service，读取/设置参数。
- 已展示场景：NVIDIA Isaac Sim 中控制 MOCA 移动操作机器人、Unitree Go 自然语言控制、工业机器人调试。
- 成熟度：开源可试，是目前 MCP+ROS 方向最直接的项目之一；安全控制、ROS Actions、权限仍是重点。
- 原始资料：[RobotMCP 官网](https://robotmcp.ai/)，[GitHub robotmcp/ros-mcp-server](https://github.com/robotmcp/ros-mcp-server)，[GitHub organization](https://github.com/robotmcp)。

### Nanobot

- 类型：开源 MCP agent framework / MCP host。
- 与机器人关系：本身不是机器人控制框架，但可以包装任何 MCP server；如果配合 ROS-MCP server，就能形成“Nanobot agent -> MCP tools -> ROS -> robot”的链路。
- 关键点：支持 reasoning、system prompt、tool orchestration、MCP-UI；GitHub README 也提示项目重构中，API 可能变化。
- 原始资料：[Nanobot 官网](https://www.nanobot.ai/)，[GitHub nanobot-ai/nanobot](https://github.com/nanobot-ai/nanobot)。

### OpenClaw

- 类型：开源个人 AI assistant / agent gateway，多渠道消息入口、skills、设备节点、工具执行。
- 与机器人关系：OpenClaw 本身是通用 agent/gateway，不是机器人栈；但它可以通过技能、MCP、ROSClaw/RobotMCP 一类桥接层调用机器人能力。适合当“人机交互/agent 编排层”，不适合裸连电机或绕过安全控制。
- 风险：权限很大，机器人场景必须加沙箱、白名单、动作审计、仿真预执行和急停。
- 原始资料：[OpenClaw 官网](https://openclaw.ai/)，[GitHub openclaw/openclaw](https://github.com/openclaw/openclaw)，[docs index](https://github.com/openclaw/openclaw/blob/main/docs/index.md)，[skills 文档](https://github.com/openclaw/openclaw/blob/main/docs/tools/skills.md)。

### ROSClaw

- 类型：面向具身 AI 的早期 OS/框架设想，桥接 Claude Code/OpenClaw/MCP 与 ROS/VLA 控制。
- 关键点：提出 MCP hub、MuJoCo digital twin firewall、1Hz 高层认知 + 1000Hz 控制解耦、skill flywheel。
- 成熟度：理念很贴近需求，但从公开资料看仍处早期，需要核验具体代码可用性、测试覆盖和硬件案例。
- 原始资料：[ROSClaw 官网](https://www.rosclaw.io/)，[GitHub ros-claw/rosclaw](https://github.com/ros-claw/rosclaw)。

## 商业产品与闭源模型

### Google DeepMind Gemini Robotics

- 类型：Gemini Robotics VLA + Gemini Robotics-ER embodied reasoning。
- 关键点：VLA 将视觉/语言转换为动作；ER 模型负责物理空间理解、规划和工具使用。2026 页面显示 Gemini Robotics-ER 1.6 可在 Google AI Studio 试用，VLA 仍偏 trusted tester/伙伴访问。
- 硬件案例：官方提到不同形态机器人、ALOHA2、Franka、Apptronik Apollo 等合作或展示。
- 原始资料：[Gemini Robotics 官方页](https://deepmind.google/en/models/gemini-robotics/)，[Gemini Robotics 1.5 页面](https://deepmind.google/en/models/gemini-robotics/gemini-robotics/)。

### Figure Helix

- 类型：闭源 humanoid VLA。
- 关键点：Figure 称 Helix 能统一感知、语言理解和 learned control；输出高频连续上半身控制，支持双机器人协作和未见过小物体抓取。
- 硬件产品：Figure 02/03，面向家庭和物流/工厂应用。
- 原始资料：[Helix 技术文章](https://www.figure.ai/news/helix)，[Helix 页面](https://www.figure.ai/helix)，[物流应用](https://www.figure.ai/news/helix-logistics)。

### 1X NEO / Redwood AI / 1X World Model

- 类型：家用 humanoid + 自研 VLA/世界模型。
- 关键点：NEO 使用 Redwood AI 进行家务学习与重复；1X World Model 用视频和动作条件预测物理结果，用于评估/学习新能力。
- 产品状态：NEO 以家庭机器人形态预订/早期交付路线推进，复杂任务可能由 1X Expert 远程辅助训练。
- 原始资料：[NEO 产品页](https://www.1x.tech/neo)，[1X AI 页面](https://www.1x.tech/ai)，[Redwood AI](https://www.1x.tech/ja_jp/discover/redwood-ai)，[1X World Model](https://www.1x.tech/discover/redwood-ai-world-model)。

### Covariant RFM-1

- 类型：仓储机器人 foundation model。
- 关键点：结合真实仓储机器人数据和互联网数据，面向物流抓取、reasoning、自然语言交互和世界模型能力。
- 硬件/产品：主要与 Covariant Brain 仓储机器人部署相关。
- 原始资料：[RFM-1 官方页](https://covariant.ai/rfm/)，[TechCrunch 发布报道](https://techcrunch.com/2024/03/11/covariant-is-building-chatgpt-for-robots/)。

### Skild AI

- 类型：omni-bodied robotics foundation model。
- 关键点：主张“任意机器人、任意任务、一个 brain”，跨 quadruped、humanoid、table-top、mobile manipulator；使用 NVIDIA Isaac/Omniverse/Cosmos 等。
- 成熟度：产品/技术闭源，公开资料更多是愿景、融资和合作。
- 原始资料：[Skild 官网](https://www.skild.ai/)，[技术博客](https://www.skild.ai/blogs/building-the-general-purpose-robotic-brain)，[NVIDIA customer story](https://www.nvidia.com/en-gb/customer-stories/skild-ai/)。

### Sanctuary AI Phoenix / Carbon

- 类型：工业 humanoid + Carbon AI control system。
- 关键点：Carbon 被描述为集成记忆、视觉、声音、触觉等子系统，把自然语言转化为现实动作；Phoenix 面向工商业任务。
- 原始资料：[Technology 页面](https://www.sanctuary.ai/technology)，[Phoenix 发布](https://www.sanctuary.ai/blog/sanctuary-ai-unveils-phoenix-a-humanoid-general-purpose-robot-designed-for-work)。

### Tesla Optimus

- 类型：通用双足 humanoid，使用 Tesla 视觉/规划/推理硬件能力。
- 关键点：官方强调为 balance、navigation、perception、physical interaction 建软件栈；公开技术细节少。
- 原始资料：[Tesla AI & Robotics](https://www.tesla.com/AI)。

### Boston Dynamics Atlas

- 类型：商用前 humanoid 平台，强调运动控制、感知和 AI。
- 关键点：电动 Atlas 用于探索 humanoid 在真实应用中的全身控制、感知和智能；与 RAI Institute 合作强化学习。
- 原始资料：[Atlas 官方页](https://bostondynamics.com/atlas)，[TechCrunch RAI Institute 合作](https://techcrunch.com/2025/02/05/boston-dynamics-joins-forces-with-its-former-ceo-to-speed-the-learning-of-its-atlas-humanoid-robot/)。

### Agility Digit

- 类型：物流/制造 humanoid 产品。
- 关键点：NVIDIA case study 显示其用 Isaac Sim/Lab 训练 whole-body control foundation model；产品部署集中在搬运、仓储、制造。
- 原始资料：[NVIDIA Digit case study](https://www.nvidia.com/en-us/case-studies/agility-robotics-digit-humanoid-robot/)。

### Unitree G1

- 类型：低价 humanoid 硬件平台，开发者生态活跃。
- 关键点：官方称 imitation/RL 驱动、UnifoLM/robot world model；LeRobot 已支持 G1 遥操作、训练 locomanipulation policies、仿真测试。
- 原始资料：[Unitree G1 官方页](https://www.unitree-robot.com/mobile/g1)，[LeRobot Unitree G1 文档](https://huggingface.co/docs/lerobot/main/en/unitree_g1)。

## 典型系统架构

### 架构 A：端到端 VLA 控制

```text
camera / proprioception / text instruction
        -> VLA policy
        -> action chunk / end-effector delta / joint targets
        -> low-level controller
        -> robot
```

代表：OpenVLA、π0、GR00T、Helix、Gemini Robotics。  
优点：泛化潜力强、技能可以从数据中学习。  
难点：数据、延迟、动作安全、硬件迁移、评估成本。

### 架构 B：Agent planner + 技能库

```text
human instruction
        -> LLM planner / code generator
        -> skill selection / program
        -> affordance check / perception / motion planner
        -> robot primitive controller
```

代表：SayCan、Code as Policies、Inner Monologue、VoxPoser、ProgPrompt。  
优点：可解释，便于复用已有技能。  
难点：planner 幻觉、技能粒度、环境反馈质量。

### 架构 C：MCP/ROS 桥接

```text
OpenClaw / Nanobot / Claude / ChatGPT
        -> MCP client
        -> ROS-MCP server
        -> rosbridge
        -> ROS topics / services / actions
        -> robot or simulator
```

代表：RobotMCP、ROSClaw、Nanobot + ROS-MCP、OpenClaw + ROSClaw。  
优点：工程接入快，适合自然语言调试与运维。  
难点：安全权限、实时控制、误操作防护。

## 评估维度

| 维度 | 应重点检查 |
|---|---|
| 硬件接口 | ROS/ROS2、SDK、CAN/EtherCAT、机械臂 API、移动底盘 API |
| 控制频率 | LLM 1Hz 级规划不能直接闭环控制高频运动；需要低层控制器 |
| 动作表示 | 离散 action token、连续 action chunk、end-effector waypoint、joint target、skill name |
| 安全层 | 白名单、仿真预执行、速度/力矩限制、碰撞检测、急停、人类监督 |
| 数据闭环 | 遥操作、失败样本、自动标注、仿真合成、真实部署日志 |
| 复现门槛 | GPU、模型权重许可证、机器人平台、相机标定、示教数据量 |
| 商用风险 | 闭源模型可访问性、供应商锁定、责任边界、隐私与远程操控 |

## 对 OpenClaw / Nanobot 落地硬件的判断

OpenClaw、Nanobot 更像“agent host / gateway / UI / tool orchestration”，不是机器人控制器。它们可以成为具身智能系统的上层入口，但不能替代：

- 机器人低层实时控制器；
- ROS/SDK 设备接口；
- 动作可行性检查；
- 仿真或数字孪生安全验证；
- 权限和审计系统。

如果要把这类 agent 部署到硬件，建议优先从“只读观测/调试”开始：

1. 只允许列 ROS topics/services、读取状态、生成诊断报告。
2. 允许低风险 service 调用，例如重启非运动节点、切换模式。
3. 接入仿真，所有运动命令先在 MuJoCo/Isaac/Gazebo 验证。
4. 只开放白名单动作 primitive，例如 `move_to_pose_safe`、`open_gripper`。
5. 最后再接高层任务，如“把杯子放到托盘”，并保留人工确认和急停。

## 推荐后续路线

如果目标是在本地做可复现实验：

1. **最快上手**：LeRobot + 低成本机械臂/SO-100 或 Unitree G1 文档环境。
2. **VLA 研究**：OpenVLA/OFT 或 openpi/π0，在 ALOHA/DROID/自采数据上微调。
3. **Agent 接入 ROS**：RobotMCP/ros-mcp-server + Claude/ChatGPT/OpenClaw/Nanobot，先做只读调试，再逐步开放动作。
4. **安全验证**：Gazebo/Isaac Sim/MuJoCo 做动作预演和碰撞/力矩检查。
5. **展示 Demo**：自然语言指令 -> agent 计划 -> 仿真验证 -> 机器人执行 -> 视频/日志回传 -> agent 解释执行结果。

## 原始链接清单

### 开源代码/模型

- OpenVLA GitHub: https://github.com/openvla/openvla
- Physical Intelligence openpi: https://github.com/Physical-Intelligence/openpi
- FAST Hugging Face: https://huggingface.co/physical-intelligence/fast
- LeRobot GitHub: https://github.com/huggingface/lerobot
- LeRobot docs: https://huggingface.co/docs/lerobot/main/en/index
- LeRobot Unitree G1: https://huggingface.co/docs/lerobot/main/en/unitree_g1
- Octo GitHub: https://github.com/octo-models/octo
- VIMA GitHub: https://github.com/vimalabs/VIMA
- VoxPoser ROS demo: https://github.com/butia-bots/voxposer_ros
- RobotMCP ROS MCP Server: https://github.com/robotmcp/ros-mcp-server
- Nanobot GitHub: https://github.com/nanobot-ai/nanobot
- OpenClaw GitHub: https://github.com/openclaw/openclaw
- ROSClaw GitHub: https://github.com/ros-claw/rosclaw

### 论文/项目页

- OpenVLA arXiv: https://arxiv.org/abs/2406.09246
- Octo: https://octo-models.github.io/
- VIMA: https://vimalabs.github.io/
- RT-2: https://deepmind.google/discover/blog/rt-2-new-model-translates-vision-and-language-into-action/
- PaLM-E: https://palm-e.github.io/
- SayCan: https://research.google/pubs/do-as-i-can-not-as-i-say-grounding-language-in-robotic-affordances/
- Code as Policies: https://code-as-policies.github.io/
- Inner Monologue: https://innermonologue.github.io/
- VoxPoser: https://voxposer.github.io/
- ProgPrompt: https://progprompt.github.io/
- FAST: https://www.physicalintelligence.company/research/fast

### 商业/产品/官方技术页

- Gemini Robotics: https://deepmind.google/en/models/gemini-robotics/
- NVIDIA GR00T N1: https://research.nvidia.com/publication/2025-03_nvidia-isaac-gr00t-n1-open-foundation-model-humanoid-robots
- NVIDIA GR00T N1.5: https://research.nvidia.com/labs/gear/gr00t-n1_5/
- NVIDIA Cosmos: https://www.nvidia.com/en-us/ai/cosmos/
- Figure Helix: https://www.figure.ai/news/helix
- 1X NEO: https://www.1x.tech/neo
- 1X AI: https://www.1x.tech/ai
- Covariant RFM-1: https://covariant.ai/rfm/
- Skild AI: https://www.skild.ai/
- Sanctuary AI Technology: https://www.sanctuary.ai/technology
- Tesla AI & Robotics: https://www.tesla.com/AI
- Boston Dynamics Atlas: https://bostondynamics.com/atlas
- Unitree G1: https://www.unitree-robot.com/mobile/g1


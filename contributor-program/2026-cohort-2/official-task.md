# Relax 贡献者计划第二期：任务详情

选择感兴趣的任务，在对应的 GitHub Issue 下留言认领。参与方式、积分规则及 RFC 安排见[活动规则与时间安排](timeline.md)。

## 任务总表

| 编号 | 任务 | 任务类型 | ⭐ | RFC | 导师 | 说明 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | [跨异构部署的增量权重同步引擎](#task-1) | 工程优化 | ⭐⭐⭐⭐ | 需要 | [@Yangruipis](https://github.com/Yangruipis) | — |
| 2 | [训练与推理双服务架构](#task-2) | 工程优化 | ⭐⭐⭐⭐⭐⭐⭐⭐ | 需要 | [@Yangruipis](https://github.com/Yangruipis) | 依赖 1 |
| 3 | [Unified Inference Service](#task-3) | 工程优化 | ⭐⭐⭐⭐⭐⭐ | 已有 RFC | [@Yangruipis](https://github.com/Yangruipis) | [RFC #71](https://github.com/redai-studio/Relax/issues/71) |
| 4 | [GenRM 支持弹性扩缩容](#task-4) | 工程优化 | ⭐⭐⭐⭐ | 需要 | [@RexFlux](https://github.com/RexFlux) | 依赖 3 |
| 5 | [Agentic Rollout 投机解码指标去重与加权聚合](#task-5) | Rollout | ⭐⭐ | 不需要 | [@yuanlehome](https://github.com/yuanlehome) | — |
| 6 | [长响应全文重复检测与离线诊断](#task-6) | Rollout | ⭐ | 不需要 | [@yuanlehome](https://github.com/yuanlehome) | — |
| 7 | [不可变 LoRA 版本的在线发布、会话绑定与安全回收](#task-7) | Rollout | ⭐⭐⭐⭐⭐ | 需要 | [@yuanlehome](https://github.com/yuanlehome) | — |
| 8 | [Agentic 多协议一致性测试](#task-8) | Agentic RL | ⭐ | 不需要 | [@xiaoliang0601](https://github.com/xiaoliang0601) | — |
| 9 | [Agentic Rollout 停止原因分布与轮数分位数指标](#task-9) | Agentic RL | ⭐⭐ | 不需要 | [@xiaoliang0601](https://github.com/xiaoliang0601) | — |
| 10 | [DeepEyes-V2 真实网页搜索工具](#task-10) | Agentic RL | ⭐⭐ | 不需要 | [@xiaoliang0601](https://github.com/xiaoliang0601) | — |
| 11 | [Relax Straggler 分析能力建设](#task-11) | 训练优化 | ⭐⭐⭐⭐ | 需要 | [@Lemon-412](https://github.com/Lemon-412) | — |
| 12 | [故障事件时间线与查询接口](#task-12) | 可观测性 | ⭐⭐⭐⭐ | 需要 | [@RexFlux](https://github.com/RexFlux) | — |
| 13 | [参数默认值集中管理与兼容逻辑规范化](#task-13) | 工程优化 | ⭐ | 不需要 | [@SigureMo](https://github.com/SigureMo) | 建议拆分多个 PR |
| 14 | [Agentic Session 轨迹导出的 CPU 性能优化](#task-14) | Agentic RL | ⭐ | 不需要 | [@DrRyanHuang](https://github.com/DrRyanHuang) | — |
| 15 | [Agentic 异步训练的权重版本追踪与滞后审计](#task-15) | 可观测性 | ⭐ | 不需要 | [@yuanlehome](https://github.com/yuanlehome) | — |
| 16 | [基于 FLOPs 估算的微批打包与数据并行负载均衡](#task-16) | 训练优化 | ⭐⭐ | 不需要 | [@yuanlehome](https://github.com/yuanlehome) | 吞吐提升 ≥20% 可额外获得 1 ⭐ |
| 17 | [为多模生成训练实现冻结条件特征复用与有界缓存（SFT 优先）](#task-17) | 多模生成训练优化 | ⭐⭐ | 不需要 | [@Michael-Salon](https://github.com/Michael-Salon) | SFT 为本期验收范围；吞吐提升 ≥10% 加分 |

## 任务详情

<a id="task-1"></a>

### 【No.1】跨异构部署的增量权重同步引擎

积分：⭐⭐⭐⭐｜RFC：需要｜导师：[@Yangruipis](https://github.com/Yangruipis)

**任务说明**

建设 Delta Weight Sync Engine，降低训推分离后的完整 checkpoint 传输成本，支持不同集群、GPU 型号及 TP／PP／EP 布局。

- 解耦 Delta Encoder、Transport 和 Weight Loader；训练端导出一致版本并生成无损差量，推理端完成校验、重建及 reshard。
- 支持 full checkpoint 初始化、周期性 anchor、版本追赶、幂等应用和 full-sync fallback。
- 提供共享存储和 TCP 两种传输路径：前者支持离线消费，后者支持分块传输、背压和断线恢复。

**验收标准**

- 两种传输路径均通过 Dense／MoE 各 100 个真实版本同步，重建结果 bitwise 一致。
- 异构 GPU、不同并行布局验证通过；故障可恢复，资源占用有界。
- 冻结配置下，计入 full sync 后，总传输量降低至少 50%。

**参考资料**：verl #6974、AReaL #1623，重点解除增量算法与传输后端的绑定。

<a id="task-2"></a>

### 【No.2】训练与推理双服务架构

积分：⭐⭐⭐⭐⭐⭐⭐⭐｜RFC：需要｜导师：[@Yangruipis](https://github.com/Yangruipis)｜依赖：任务 1

**任务说明**

将 Training 与 Rollout 建设为对等、独立的服务，各自管理部署、资源、状态和恢复；由外部 Orchestrator 连接采样、奖励、训练和权重发布流程。两端均不负责创建或管理另一端。

- Training 提供训练批次提交、更新状态、checkpoint／resume 和权重发布接口。
- Rollout 提供生成、取消、版本约束及副本管理接口。
- 通过显式数据与版本协议连接两端，接入增量同步引擎，处理样本去重、staleness、背压及恢复分支。

**验收标准**

- 两端可独立部署、启动、停止及重启，无共享 Ray 集群或跨端 collective 依赖。
- 独立客户端可分别调用训练和生成接口，外部编排完成 100 次更新及 8 小时稳定训练。
- Rollout 扩缩容、Training checkpoint 后的资源重配置均不要求重启另一服务。
- 故障恢复后，无错误版本服务、分支混用或重复训练消费。

**参考资料**：AReaL 的服务拆分与原子发布、Slime 的外部生成接口。

<a id="task-3"></a>

### 【No.3】Unified Inference Service

积分：⭐⭐⭐⭐⭐⭐｜RFC：已有 [RFC #71](https://github.com/redai-studio/Relax/issues/71)｜导师：[@Yangruipis](https://github.com/Yangruipis)

**任务说明**

统一 Rollout、GenRM 和 Teacher 的推理基础设施，复用 API、引擎生命周期和 GPU 分配逻辑，保留各自业务流程。

- 建设 CPU-only InferenceGateway、统一 Manager 和 Engine；Gateway 与 direct client 共用路由规则。
- 提前校验 Placement，支持 decoupled、split／defer，统一 drain／offload／onload，补齐 deferred OPD 结果回写。
- 先统一 discovery／client，再迁移生命周期，兼容旧接口。Gateway 按角色分别实例化，一期拒绝同卡同阶段共同驻留。

**验收标准**

- 三类角色共用实现，静态模型不参与动态权重更新。
- 路由一致，discovery 状态正确，不暴露内部 TP／PP worker。
- 非法布局在启动前拒绝；defer 无资源冲突，Teacher 结果在训练前回写。
- 生命周期幂等、失败回滚、PG ownership 和旧接口兼容测试通过，并完成多节点 GPU 验证。

**参考资料**：[RFC #71](https://github.com/redai-studio/Relax/issues/71)。认领及评审状态以该 Issue 中的确认结果为准。

<a id="task-4"></a>

### 【No.4】GenRM 支持弹性扩缩容

积分：⭐⭐⭐⭐｜RFC：需要｜导师：[@RexFlux](https://github.com/RexFlux)｜依赖：任务 3

**任务说明**

为 GenRM 增加动态引擎管理，并接入 Autoscaler。GenRM 是冻结奖励模型，扩缩容无需训练权重同步。

- GenRMManager 支持 PG 分配、引擎启停和路由更新，对齐 RolloutManager 的扩缩容语义。
- 提供 `/genrm/scale_out`、`/genrm/scale_in`、`/genrm/engines` 和状态机；跳过 `WEIGHT_SYNCING`，健康检查通过后才注册路由。
- `num_replicas` 表示目标绝对总数；支持幂等、并发保护、优雅排空，缩容保留初始引擎。
- 暴露 token／KV 使用率、排队数和 TTFT 等指标，让 Autoscaler 支持 GenRM 独立阈值，并接入 `/status`、`/conditions`、`scale_history` 和 TUI monitor。

**验收标准**

- 手动扩容可拉起引擎并在就绪后接收打分请求；缩容可优雅排空并回收资源。
- 全程无权重同步，新引擎打分结果与初始引擎一致。
- 重复请求幂等，在途并发请求返回 409，非法副本数返回明确 4xx；不删除初始引擎，副本数不低于初始值。
- Autoscaler 可采集负载指标，随打分负载升降自动扩缩容；接口和 TUI 可查看指标、触发条件及扩缩状态。
- 扩缩期间 actor／rollout 训练不中断；文本场景端到端跑通 GenRM 训练，附扩缩容日志与 Autoscaler 决策证据。

**参考资料**：现有 Rollout 弹性扩缩容、`ScaleOutStatus`、`metrics_collector` 和 Autoscaler 实现。

<a id="task-5"></a>

### 【No.5】Agentic Rollout 投机解码指标去重与加权聚合

积分：⭐⭐｜RFC：不需要｜导师：[@yuanlehome](https://github.com/yuanlehome)

**任务说明**

修正投机解码指标中“样本接受率取平均”和共享生成节点重复计数的问题。

- 在 Agentic 导出信息中保留已提交生成节点的标识及 accepted、proposed、verify、completion 计数。
- 在一次 rollout 指标批次内按生成节点去重，先汇总计数，再计算接受率及每次 verify 的产出 token 数。
- 区分计数为 0 与后端未提供计数，报告指标覆盖情况；接入现有日志，兼容旧数据并说明新旧指标名称和口径。

只统计本次导出样本覆盖的已提交生成节点，不要求实现投机解码算法、统计被丢弃分支或证明实际加速。

**验收标准**

- 两次独立生成的 accepted／proposed 为 1／2、9／10 时，整体接受率为 10／12，约 83.33%。
- 导出轨迹 A→B、A→C 中，A 只计一次，B、C 均计入；独立请求即使文本相同也分别计数，同一 session 内不误去重，不同 session 间不串计数。
- 空输入、零分母、缺失字段和旧版序列化数据有明确结果，不产生除零异常或伪造的 0%。
- CPU 集成测试贯通元数据处理、轨迹导出和指标汇总，提交自动化测试、可人工核对的示例报告及指标兼容性说明。

**参考资料**：[miles PR #2598](https://github.com/radixark/miles/pull/2598)。

<a id="task-6"></a>

### 【No.6】长响应全文重复检测与离线诊断

积分：⭐｜RFC：不需要｜导师：[@yuanlehome](https://github.com/yuanlehome)

**任务说明**

将仅检查响应末尾的重复检测扩展到全文，帮助定位长响应开头或中间的重复。

- 沿用压缩比判定方法，使用重叠滑窗扫描全文；默认窗口 10,000 字符、步长 5,000 字符，覆盖未对齐的末段。
- 返回是否命中、命中窗口字符区间、压缩比及最大压缩比。区间表示疑似重复窗口，不表示精确重复边界。
- 接入 `repetition_frac`，保留布尔接口；增加读取既有 rollout dump 的离线入口，输出样本标识、命中区间及 JSON 报告。

检测对象为 `Sample.response`，可能包含工具观察；不要求区分角色、识别语义重复，也不截断生成或改变 reward。

**验收标准**

- 开头、中间、结尾重复均可检出，覆盖“中间重复、结尾正常”的固定样例。
- 覆盖空字符串、短响应、整窗、窗口边界、未对齐末窗、中文、无重复对照及压缩比恰好等于阈值的情况；明确字符偏移定义。
- 如报告覆盖字符数，按重叠区间并集计算，避免重复计数。
- 提供 1 万、10 万、100 万字符的 CPU 耗时与峰值内存基准，不通过静默跳过中间窗口限制成本。
- 使用实际 rollout 指标入口和真实 dump 读取入口完成集成验证，提交固定测试样例、基准结果及使用说明。

**参考资料**：[miles PR #3115](https://github.com/radixark/miles/pull/3115)。

<a id="task-7"></a>

### 【No.7】不可变 LoRA 版本的在线发布、会话绑定与安全回收

积分：⭐⭐⭐⭐⭐｜RFC：需要｜导师：[@yuanlehome](https://github.com/yuanlehome)

**任务说明**

在持续发布新 adapter 时，让旧 Agent 会话继续使用旧版，新会话使用已发布的新版。

- 每份 adapter 使用唯一版本标识和内容摘要；同版本同内容重试幂等，同版本不同内容拒绝。
- 新版在两个目标引擎全部就绪后，才通过单一切换点更新默认版本；部分失败时保留旧版并清理未发布资源。
- 会话首次生成时固定版本，后续工具轮次、重试和 abort／resume 保持绑定，实际请求携带对应 `lora_path`。目标引擎缺少版本时明确等待或失败，不回退到基座或最新版。
- 有会话引用或后端在途请求的版本不可卸载，也不能被底层 LRU 淘汰；容量不足时延后或拒绝发布。
- 至少接入一种现有 adapter 导出／传输路径，完成快照、加载、发布到实际生成的完整流程。
- 发布不触发全局暂停生成或清缓存；不同 adapter 版本间不得错误复用 KV cache。

**验收标准**

- B 仅加载到一个引擎时，新会话仍使用 A；B 全部就绪后，新会话使用 B，旧会话及续写仍使用 A。
- 覆盖单引擎失败、迟到确认、重复发布、同 ID 内容冲突、取消与完成并发、迟到回包；状态和资源结果可机器判定，不提前回收。
- 容量为两个版本、A 仍有引用且 B 已发布时，发布 C 必须等待或返回容量错误；A 完全释放后只卸载一次，再允许 C。
- 真实 GPU 实验使用两份可区分的 LoRA fixture，建立独立加载 A／B 的 logprob 基线；并发发布时旧会话匹配 A、新会话匹配 B，数值容差提前固定。
- 同输入先用 A 预热、再用 B 请求，结果与 B 冷缓存基线比较；反向也验证。仅检查版本日志不能作为通过依据。
- 发布期间保持生成流量，记录完成进度、发布耗时、延迟、失败数及版本占用，确认无全局暂停、清缓存或强制中断旧会话。
- 提交版本管理、发布与会话绑定实现、资源回收实现、故障注入测试、GPU 对照实验及容量和失败行为说明。

**参考资料**：[miles PR #3127](https://github.com/radixark/miles/pull/3127)、[SGLang LoRA](https://docs.sglang.io/docs/advanced_features/lora)。

<a id="task-8"></a>

### 【No.8】Agentic 多协议一致性测试

积分：⭐｜RFC：不需要｜导师：[@xiaoliang0601](https://github.com/xiaoliang0601)

**任务说明**

为 Chat Completions、Responses、Anthropic Messages 三种协议增加 canonical request golden tests，验证语义等价的请求进入 Agentic Session 后具有一致表示。

- 覆盖文本消息、assistant tool call、tool result、图片输入，以及 `messages`、`tools`、`chat_template_kwargs` 的规范化和稳定序列化。
- 覆盖未知／非法 role、tool call 缺失 ID、空字符串／空列表 content、非法 image block，以及不完整的 tool call／tool result。
- 优先复用 `check_messages`、`normalize_tools` 和 `normalize_template_kwargs`；测试不依赖网络或真实模型服务。

**验收标准**

- 三种协议均有独立 golden fixtures 和测试用例；语义等价输入转换后的三个规范化字段完全一致。
- 各协议的合法图片输入统一为 canonical 格式，非法输入均被拒绝。
- 异常覆盖上述场景，错误信息包含准确、稳定的完整字段路径，例如 `messages[2].tool_calls[0].id`。

**参考资料**：`relax/agentic/session/service.py`、`relax/agentic/session/state.py`、`tests/agentic/`。

<a id="task-9"></a>

### 【No.9】Agentic Rollout 停止原因分布与轮数分位数指标

积分：⭐⭐｜RFC：不需要｜导师：[@xiaoliang0601](https://github.com/xiaoliang0601)

**任务说明**

在现有 rollout 指标中补充停止原因计数、占比，以及轮数 P50、P90、P95、P99，用于识别长尾轨迹和异常停止。

- 新增纯聚合函数，参考 `_compute_min_mean_max_stats`，读取 sample metadata 中的停止原因和轮数，建议字段为 `rollout_stop_reason`、`rollout_turns`。
- 接入 `compute_metrics_from_samples`；只读取输入并返回指标字典，不修改 sample，不依赖全局状态。
- 停止原因占比以有效 sample 总数为分母，缺失或空值归入明确的 `unknown` 类别，或按项目约定处理。

**验收标准**

- 单测覆盖空输入、单条、多条样本和缺失 metadata；空输入返回空指标，不触发统计异常。
- 非空有效样本输出每种停止原因的计数和占比，所有占比之和为 1。
- 输出 `num_turn/p50`、`num_turn/p90`、`num_turn/p95`、`num_turn/p99`，经现有汇总入口正常上报。
- 保留 `num_turn/min`、`num_turn/mean`、`num_turn/max`；不改变 rollout 主循环、停止条件或调度流程，不新增状态字段或控制分支。

**参考资料**：`relax/distributed/ray/rollout.py`、`relax/agentic/session/state.py`、`tests/test_agentic_rollout.py`。

<a id="task-10"></a>

### 【No.10】DeepEyes-V2 真实网页搜索工具

积分：⭐⭐｜RFC：不需要｜导师：[@xiaoliang0601](https://github.com/xiaoliang0601)

**任务说明**

将 DeepEyes-V2 的占位 `search()` 改为可插拔后端，支持 Search-R1 retriever、外部搜索 API 和 mock。默认使用确定性的离线 mock，保证示例在无检索服务、网络或密钥时仍可运行。

统一返回 `elapsed_time` 和 `data`；每条结果包含 `title`、`link`、`snippet` 和可为空的 `date`。

- retriever 兼容 Search-R1 HTTP 协议，可配置 URL、top-k、超时和重试。
- external 可配置 endpoint、认证信息和请求字段映射，不硬编码密钥。
- 异常、超时和非法返回沿用现有环境的 `"Error"` 处理约定；优先复用现有 HTTP 客户端和依赖。

**验收标准**

- 默认配置可离线运行 DeepEyes-V2 示例。
- retriever 可调用 Search-R1 兼容服务；external 可通过配置接入至少一种外部搜索 API，结果均转为统一结构。
- 支持超时、重试及服务异常处理，失败不导致 agent 进程崩溃。
- 测试覆盖三个后端的适配及异常处理；使用真实服务时不再返回固定 placeholder 文本。

**参考资料**：`examples/deepeyes_v2_agentic/app/search_utils.py`、`app/env_deepeyes_v2.py`、`examples/search_r1/retrieval_server.py`、`examples/on_policy_distillation/agentic_opd/search_qa/app/retrieval_client.py`、DeepEyes-V2 README。

<a id="task-11"></a>

### 【No.11】Relax Straggler 分析能力建设

积分：⭐⭐⭐⭐｜RFC：需要｜导师：[@Lemon-412](https://github.com/Lemon-412)

**任务说明**

开发可随训练持续运行的轻量 profiling 工具，实时定位慢卡，并识别或提示前向／反向、通信、optimizer、attention／MoE 等粗粒度耗时，帮助排查计算和通信瓶颈。

**验收标准**

- 整体性能开销低于 0.5%。
- 端到端跑通，不影响训练精度、loss 指标和已有通算 overlap 等加速收益。
- 结合现有平台能力实时上报，结果清晰、便于定位问题。

**参考资料**：[OSDI 2025 论文](https://www.usenix.org/system/files/osdi25-lin-jinkun.pdf)、[背景介绍](https://zhuanlan.zhihu.com/p/1926349361299845442)。后续上线如需对接内部平台，由导师确认对接安排。

<a id="task-12"></a>

### 【No.12】故障事件时间线与查询接口

积分：⭐⭐⭐⭐｜RFC：需要｜导师：[@RexFlux](https://github.com/RexFlux)

**任务说明**

为 Relax 增加有界、只追加、可查询的故障事件时间线，保留故障从发现、处理到恢复成功或失败的过程。优先调研 Ray 现有 actor 状态管理与采集能力。

- 记录关键故障事件，按实际记录顺序查询，支持按服务角色和同一次故障筛选。
- 重复上报不重复记录，事件数量有明确上限。
- 记录或查询服务不可用时，不阻塞或中断训练；事件字段、接口和容量限制在 RFC 中确定。

**验收标准**

- 可记录和查询一次故障从发现、处理到成功恢复或失败结束的关键事件；多个组件并发上报时顺序明确，记录不丢失，同一事件不重复。
- 支持按服务角色、故障标识筛选及分批查询；历史事件因容量限制淘汰时，明确提示记录不完整。
- 存储容量始终有界；上报超时、服务不可用或内部异常不影响训练及原有故障处理。
- 不记录训练样本、模型数据、环境变量、完整配置或其他敏感内容。
- 所有新增功能通过 CPU 单测和本地集成测试验证，不依赖 GPU、QS 或内部服务。
- 保持 MetricsService、健康检查和性能时间线兼容，提交 RFC 及事件范围、查询方式、容量限制和能力边界的使用说明。

<a id="task-13"></a>

### 【No.13】参数默认值集中管理与兼容逻辑规范化

积分：⭐｜RFC：不需要｜导师：[@SigureMo](https://github.com/SigureMo)

**任务说明**

Relax 的参数定义已经声明默认值，但业务代码中仍有大量 `getattr(args, "task_type", "causal_lm")` 等写法，重复维护默认值，也可能掩盖初始化或测试配置中缺失参数的问题。

- Relax 自有参数的声明默认值以参数定义为唯一来源；运行期派生值在统一初始化或校验阶段生成，业务代码不再重复补默认值。
- 对完成解析和初始化的参数对象，使用 `args.xxx`、`self.config.xxx` 等方式直接访问；清理范围覆盖整个框架。
- 将确有 Megatron 版本差异、字段缺失或改名的必要兼容逻辑集中至独立文件，说明适用条件和差异来源；Megatron 后端目录中的 Relax 自有参数仍应直接读取。
- 修复测试中手工构造的不完整参数对象，复用参数定义建立统一的测试配置构造方式，各用例只声明差异，避免另建默认值表。
- 保留显式 `None`、`False`、`0`、空列表及配置覆盖的既有含义。建议按模块拆分多个 PR，全部合入后完成整题验收。

**验收标准**

- Relax 自有参数的业务读取不再依赖 `getattr` 补默认值，同一参数的声明默认值只需在一处修改。
- 必要的 Megatron 兼容处理集中维护，并有明确依据和测试。
- 不通过 `hasattr`、`vars(args).get()`、捕获 `AttributeError` 或通用包装函数继续隐藏自有参数缺失问题。
- 默认配置、显式配置及配置覆盖行为保持一致，测试覆盖 `None`、`False`、`0`、空列表等边界。
- 提交清理范围、前后对照和剩余兼容项清单；所有拆分 PR 合入后完成整题验收。

<a id="task-14"></a>

### 【No.14】Agentic Session 轨迹导出的 CPU 性能优化

积分：⭐｜RFC：不需要｜导师：[@DrRyanHuang](https://github.com/DrRyanHuang)

**任务说明**

多个 Agent 并发导出长轨迹时，样本组装中的 CPU 计算和数据拷贝可能阻塞 Session 服务。通过可重复运行的性能基准定位主要瓶颈，降低轨迹导出对其他请求响应的影响。

- 构建使用真实样本组装逻辑、模拟推理后端的压测工具，覆盖不同轨迹长度和并发量。
- 定位至少一处主要 CPU 阻塞，根据测量结果采用状态快照、有界后台处理或减少重复拷贝等方式优化。
- 保证并发导出过程中的状态一致性，正确处理取消、异常和服务关闭。
- 对比优化前后的事件循环延迟、请求响应延迟、导出耗时、CPU 时间及峰值内存。

**验收标准**

- 在相同硬件、可比的长轨迹请求负载下，事件循环 P99 延迟降低至少 50%。允许根据优化方案调整进程、线程及并发配置，并说明资源使用变化。
- 压测方法可按方案选择，需说明负载、预热、测量窗口及 P99 统计口径；请求延迟计入排队时间，同时报告完成导出数量、导出吞吐、短请求延迟及重复运行的波动，不通过减少有效工作量或丢弃请求获得收益。
- 优化前后导出的 tokens、loss mask、logprobs 等训练内容一致。
- 取消和关闭操作能够正确收尾，后台任务及资源不会持续积累。
- 提交实现、正确性测试、可重复运行的压测工具及性能报告。
- 提供持续发送短请求、同时批量导出长轨迹的可复现示例，展示优化前后短请求的延迟变化。

**参考资料**：[miles PR #3736](https://github.com/radixark/miles/pull/3736)、[miles PR #4044](https://github.com/radixark/miles/pull/4044)。

<a id="task-15"></a>

### 【No.15】Agentic 异步训练的权重版本追踪与滞后审计

积分：⭐｜RFC：不需要｜导师：[@yuanlehome](https://github.com/yuanlehome)

**任务说明**

异步训练中，多轮生成可能跨越多次权重更新，生成完成后还可能等待较长时间才被训练消费。补齐生成片段、权重版本与训练消费时刻之间的关联，追踪实际参与训练的 token 来源及版本滞后发生的阶段。

- 为成功提交的生成片段记录 token 区间及对应权重版本，覆盖多轮生成和 partial rollout 中断恢复。
- 对实际进入训练的样本，统计生成期间的版本跨度、训练消费时的版本滞后、按 token 加权的滞后及版本信息缺失率。
- 区分生成过程中跨越的版本变化与生成完成后的等待滞后，明确各项指标的统计口径。
- 提供能够追溯到具体 Session、轮次和生成片段的样本诊断报告。

**验收标准**

- 正确覆盖跨版本多轮生成、暂停恢复、缺失版本及取消重试等场景。
- token 区间与成功提交的生成内容对应，取消或被替换的片段不混入实际训练内容的统计。
- 未知版本显式呈现，不按版本 0 或最新版本处理；同时报告版本信息缺失率。
- 版本年龄按同一策略的已发布版本序号计算，明确生成跨度和消费滞后的区别。
- 首版作为旁路审计能力，沿用现有采样和 GRPO 组处理语义。
- 提交实现、测试及诊断示例；提供可复现示例，展示 Agent 在工具等待期间经历两次权重更新后，各轮 token 的来源及训练消费时的版本年龄。

**参考资料**：[miles PR #1891](https://github.com/radixark/miles/pull/1891)、[miles PR #3334](https://github.com/radixark/miles/pull/3334)、[Relax `weight_versions` 实现](https://github.com/redai-studio/Relax/blob/main/relax/utils/types.py#L45)。

<a id="task-16"></a>

### 【No.16】基于 FLOPs 估算的微批打包与数据并行负载均衡

积分：⭐⭐｜RFC：不需要｜导师：[@yuanlehome](https://github.com/yuanlehome)

**任务说明**

相同的 token 总量不一定对应相同的计算量。根据模型结构和样本长度估算 FLOPs，在已有长度均衡和动态打包路径的基线上，验证 FLOPs 感知调度能够在哪些配置下进一步提升训练吞吐。

- 实现可配置的 FLOPs 感知调度，覆盖微批构造和数据并行分配；关闭功能后保留原有行为。
- 对齐并验证参考实现的核心调度逻辑，说明针对 Relax 的适配或改进。
- 覆盖 Dense 和 MoE 架构，推荐分别使用 Qwen3.5-27B 和 Qwen3.5-35B-A3B，参考现有训练脚本选择公开、真实的 SFT 或 RL 工作负载开展性能对照；固定模型 checkpoint、数据版本、精度及并行配置。
- 分析微批数量变化、rank 间负载差异缩小或等待时间减少等收益来源，提供实际测量证据。

**验收标准**

- Dense 和 MoE 均完成正确性验证及性能对照。固定同一有效 batch、初始权重、优化器状态及随机状态，对照调度前后的 loss、完整有效 batch 累积并归约后的梯度，以及一次优化器更新产生的 FP32 参数更新量；有效样本、token、loss mask 和归一化语义保持一致。随机操作按样本固定或在两版正确性对照中同样关闭。
- 覆盖不等长样本、不等微批大小，以及 DP 和实际启用的 TP／PP／CP／EP 组合；验证各同步组的微批执行、梯度归约及优化器更新步数兼容，无挂起、漏算或重复消费。
- 至少一个真实工作负载的训练循环有效 token 吞吐提升 **≥10%**；如提升 **≥20%，可额外获得 1 ⭐**。
- 对照实验使用相同模型、数据、有效 batch 和训练配置，基线包含已有长度均衡与动态打包优化。
- 公开数据选择规则、长度分布及适用条件；人工合成数据可用于机制分析，不能作为唯一性能验收依据。
- 提交代码、完整启动脚本、环境与数据版本、采样规则、原始日志及指标汇总脚本，支持维护者独立复跑。

数值对照需在优化前固定 loss、梯度及参数更新量的容差。参数更新量比较 `Δθ = θ_after - θ_before`，使用 FP32 主权重或优化器实际产生的 FP32 更新量，不以更新后完整权重的相对误差代替。所有被比较的量须有限；先重复运行基线检查自身波动，异常需定位，不在看到优化结果后放宽容差。

**参考资料**：[Slime PR #2017](https://github.com/THUDM/slime/pull/2017)、[Qwen3.5-27B 训练脚本](https://github.com/redai-studio/Relax/blob/main/scripts/training/multimodal/run-qwen35-27B-8xgpu-openr1mm-colocate.sh)、[Qwen3.5-35B-A3B 训练脚本](https://github.com/redai-studio/Relax/blob/main/scripts/training/text/run-qwen35-35B-A3B-16xgpu.sh)。

<a id="task-17"></a>

### 【No.17】为多模生成训练实现冻结条件特征复用与有界缓存（SFT 优先）

积分：⭐⭐｜RFC：不需要｜导师：[@Michael-Salon](https://github.com/Michael-Salon)

**任务说明**

多模生成训练中的冻结文本编码器、参考图编码器等可能对相同输入重复计算。基于 Relax 已合入的多模生成 SFT 路径建设跨模型的公共复用能力，减少重复编码、装载和搬运，并验证真实端到端收益。

- 建立在线编码基线，记录编码调用次数、缓存命中率、重复搬运字节数、阶段耗时及内存，确认现有路径尚未消除的重复工作。
- 由模型适配层声明可复用字段及其依赖，公共层实现有界缓存。缓存键覆盖冻结编码器权重或 adapter 版本、输入内容、预处理及 dtype；条件变化后正确失效，不按样本文件名简单复用。
- 在 SFT 路径实现重复样本条件特征的复用，减少不必要的副本与传输。公共接口保持模型解耦，便于后续 RL 接入。
- 覆盖 SenseNova 及至少一种其他已有 SFT 适配的扩散模型，如 FLUX 或 Qwen-Image；公共缓存逻辑不依赖单一模型的层名或内部结构。
- 保留可关闭的在线路径，明确数据所有权、容量、淘汰和释放时机，覆盖取消、异常及连续多轮训练。

SFT 为本期必做与性能验收范围；RL 为后续扩展，不依赖其未合入分支，首版不要求接入 RL、SGLang 或生成端到训练端的特征传递，也无需新增完整模型适配或奖励系统。

首版仅复用输入一致、允许确定性复用的冻结条件特征。训练中的编码器不得使用旧缓存；保留随机裁剪、增强、caption dropout 及随机数流语义，不缓存各候选独立的噪声、生成轨迹或奖励。VAE 等含随机采样的编码可作为扩展，但需证明采样语义未改变。

**验收标准**

- 至少两种模型在 SFT 路径共用公共实现并完成正确性验证；RL 仅说明未来接入方式，不要求本期跑通或提供收益数据。
- 缓存读回的条件张量与写入前逐元素完全一致，保持相同 dtype。固定同一有效 batch、初始权重、优化器状态、噪声及随机状态，对照在线编码与缓存路径的 loss、完整有效 batch 累积并归约后的梯度，以及一次优化器更新产生的 FP32 参数更新量。覆盖缓存失效、随机增强、缺失或损坏缓存、取消与异常释放。
- 确定性精确一致、非确定性提供重复运行基线及近零绝对容差。
- 训练数值对照的相对误差上限为 **1%**：loss 检查 `abs(loss - loss_ref) <= 0.01 × abs(loss_ref)`；梯度和参数更新量逐个参数张量检查 `L2(actual - reference) <= 0.01 × L2(reference)`，并报告最大绝对误差。参考值或参考张量范数为零时，要求对应结果也为零；所有被比较的量须有限。参数更新量比较 `Δθ = θ_after - θ_before`，使用 FP32 主权重或优化器实际产生的 FP32 更新量，不以更新后完整权重的相对误差代替。
- 至少一个真实 SFT 工作负载的稳态端到端吞吐提升 **≥5%，≥10% 加分**；第二种模型提供性能对照及适用条件。固定数据顺序、有效 batch、分辨率、优化器与训练配置，重复测量并报告波动；已有优化纳入基线，不通过减少有效样本或训练计算量获得收益。
- 统计相同有效样本的完整训练循环，包含数据准备、条件装载、前向、反向及优化器更新；单独报告冷启动、缓存构建耗时、总耗时与回本样本量或轮数，不以局部编码提速代替端到端收益。
- 公开真实负载的选择规则，说明收益与冻结条件重复程度、原有编码开销的关系，并报告低复用负载的开销。
- 缓存容量有明确上限，多轮运行无持续内存或磁盘增长；报告 CPU、GPU、共享内存及磁盘占用，不以无界缓存换取性能。
- 提交实现、CPU 正确性与生命周期测试、真实 GPU 对照、启动脚本、模型和数据版本、原始日志、汇总脚本及适用条件说明。

**参考资料**：[Diffusers Advanced Diffusion Training](https://github.com/huggingface/diffusers/blob/main/examples/advanced_diffusion_training/README.md)。

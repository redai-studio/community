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

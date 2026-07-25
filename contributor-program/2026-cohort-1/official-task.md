# Relax 贡献者计划 2026 · 第一期任务

> 本文档为 Relax 框架贡献者计划（2026 Cohort 1）的正式任务说明，涵盖参与须知、任务列表、交付规范与报告模板。

---

## 目录

- [1. 统一说明](#1-统一说明)
  - [1.1 参与者开始前必须固定的信息](#11-参与者开始前必须固定的信息)
  - [1.2 两类交付方式](#12-两类交付方式)
  - [1.3 通用验收底线](#13-通用验收底线)
- [2. 简单任务（1–2 天）](#2-简单任务12-天)
  - [2.1 性能调优](#21-性能调优)
  - [2.2 代码梳理（技术报告）](#22-代码梳理技术报告)
  - [2.3 模块单测与 Bug 修复](#23-模块单测与-bug-修复)
- [3. 中级任务（3–7 天）](#3-中级任务37-天)
- [4. 高级任务（1–2 周）](#4-高级任务12-周)
- [5. 技术报告交付模板与样例](#5-技术报告交付模板与样例)
- [6. PR 交互规范](#6-pr-交互规范)

---

## 1. 统一说明

### 1.1 参与者开始前必须固定的信息

每道题认领后，按照题目要求提出 Issue 中写请以下内容，再开始开发或实验：

1. 代码仓库、基线 commit、开发分支；
2. 运行环境：镜像/依赖版本、Python、CUDA、GPU 型号与数量、CPU 和内存；
3. 启动脚本及完整命令，包含所有新增或修改过的参数；
4. 相关文件、核心函数、改动范围，以及明确的"不做什么"；
5. 基线数据、验收指标、统计口径和目标值；
6. 预期交付方式：技术报告，或 GitHub Issue + Pull Request。

> **GPU 说明**：本期活动不统一提供 GPU。代码梳理和纯 CPU 单测无需 GPU；训练、性能、分布式和算法题由参与者自备与任务规模匹配的硬件。涉及性能对比时，before/after 必须使用相同代码基线、模型、数据、硬件、batch/序列长度、采样设置和统计区间；不得通过减少有效工作量、丢样本或降低结果质量换取速度。

---

### 1.2 两类交付方式

| 类型 | 适用题目 | 说明 |
|------|----------|------|
| **技术报告类** | 新手入门任务与简单 05–09 | 按第 5 节模板进行整理，打包压缩包提交 |
| **代码类** | 其余题目 | 认领 Issue → 对齐方案和验收口径 → 开发与自测 → Draft PR → Review/CI → 合并验收 |

---

### 1.3 通用验收底线

- 提交内容可在认领 Issue 固定的环境中复现；命令、配置、commit 和原始结果齐全。
- 新增功能或修复有针对性测试；已有相关测试全部通过，不得跳过失败用例。
- 日志不得出现未解释的异常、NaN/Inf、静默降级或数据丢失。
- PR 只包含当前任务必要改动，并补齐必要的文档、配置样例和兼容性说明。
- 性能题至少报告 1 个吞吐或延迟指标、GPU 利用率、峰值显存，并提供不少于 3 个稳定统计窗口或 3 次重复运行的均值；同时报告正确性/质量护栏。

---

## 2. 简单任务（1–2 天）

### 2.1 性能调优

#### 01 多模态 Dense

**背景/目标**：现有多模态 Dense demo 是通用配置，需要在不改变有效训练负载和质量口径的前提下，提高训练吞吐与 GPU 利用率。

**入口**：`scripts/training/multimodal/` 下认领时选定的可运行脚本。

**范围**：脚本参数和必要的小范围配置/代码优化；Issue 中列出实际文件与参数。

**验收标准**：
- 同环境完成基线和优化版 GRPO ≥200 step
- loss/reward 无异常，结果可复现
- 报告 samples/s 或 tokens/s、step time、平均 GPU 利用率、峰值显存的 before/after 数字和统计区间
- 主要吞吐指标高于基线，且无 OOM、丢样本或有效 batch/序列长度下降
- 附完整命令、日志、曲线和 profiler 证据

**交付方式**：GitHub Issue + PR；PR 含配置、结果表、风险及回退方式。

---

#### 02 多模态 MoE

**背景/目标**：MoE 的 expert routing、通信和并行配置会形成不同瓶颈，需要针对多模态 demo 找到并验证更高吞吐配置。

**入口**：多模态训练 recipe（准确脚本在 Issue 由出题人确认）。

**验收标准**：与 01 相同，另需记录 MoE 相关并行/路由参数，并证明优化未改变模型、数据和有效 token 口径。

**交付方式**：GitHub Issue + PR。

---

#### 03 纯文本 Dense

**背景/目标**：定位纯文本 Dense 训练中的 rollout、通信或训练瓶颈，提高端到端吞吐和 GPU 利用率。

**入口**：`scripts/training/text/` 下由 Issue 固定的 Dense demo。

**验收标准**：同环境完成 GRPO ≥200 step；loss/reward 正常；给出吞吐、step time、GPU 利用率、峰值显存的 before/after 和至少 3 个稳定窗口；主要指标提升，质量护栏不退化；命令与原始日志齐全。

**交付方式**：GitHub Issue + PR。

---

#### 04 纯文本 MoE

**背景/目标**：定位纯文本 MoE 的 expert、通信和负载均衡瓶颈，产出可复用的更优 demo 配置。

**入口**：`scripts/training/text/` 下由 Issue 固定的 MoE demo。

**验收标准**：与 03 相同，补充 MoE 并行与 expert 配置、负载均衡相关指标，证明性能收益不是由减少有效计算量获得。

**交付方式**：GitHub Issue + PR。

---

### 2.2 代码梳理（技术报告）

#### 05 三层架构

**背景**：新贡献者需要理解 Controller / Service / Components 的职责边界及协作方式。

**范围**：`relax/core/`，从一个真实入口追踪到组件执行和结果返回。

**目标**：解释对象生命周期、调用方向、状态归属和扩展点。

**验收标准**：
- 报告含 1 张架构图和 1 张时序图
- 覆盖三层职责、至少 1 条端到端调用链、≥5 个关键文件/类/函数及源码链接
- 说明初始化、正常路径、异常/清理路径和新增组件的接入点
- 用日志、断点或最小调用验证至少 1 条关键结论

**交付方式**：按第 5 节提交技术报告压缩包。

---

#### 06 权重同步

**背景**：训练侧更新后的参数需通过 DCS + NCCL broadcast 安全送入 SGLang rollout 侧。

**范围**：`relax/distributed/checkpoint_service/`、`relax/backends/megatron/weight_update/` 及其调用入口。

**目标**：讲清触发条件、rank/group、元数据与 tensor 传递、同步屏障和失败处理。

**验收标准**：
- 报告含组件图和时序图
- 标注训练参数从产生到 SGLang 生效的关键函数、进程/rank 和通信组
- 说明 shape/dtype/device、一致性检查、同步/异步边界
- 通过日志或最小实验验证同步前后权重版本/输出变化

**交付方式**：技术报告压缩包。

---

#### 07 三种运行模式

**背景**：colocate、fully-async、hybrid 在资源复用、并发和数据新鲜度上取舍不同。

**范围**：`core/`、`engine/rollout/`、`relax/backends/megatron/` 及对应启动参数。

**目标**：让读者能根据负载选择模式。

**验收标准**：
- 对三种模式分别给出数据流和 GPU 编排图
- 用同一维度表格比较进程/Actor、GPU 归属、同步点、权重新鲜度、优缺点和适用场景
- 每种模式追踪至少 1 条关键调用链并附源码链接
- 指出切换模式的入口参数及常见故障

**交付方式**：技术报告压缩包。

---

#### 08 Rollout 请求生命周期

**背景**：一次请求跨越 router、SGLang engine 和 reward，问题可能发生在排队、生成、回传或打分阶段。

**范围**：从请求创建到 reward 进入训练 batch。

**验收标准**：
- 报告含完整时序图
- 明确请求 ID、输入/输出数据结构、路由决策、异步边界、超时/取消/重试和 reward 回填
- 列出 ≥5 个关键函数及源码链接
- 用若干条真实日志将各阶段串联，并说明至少 2 个可观测/排障点
- 期望以技术分享 blog 的形式呈现，而非 AI 机械生成

**交付方式**：技术报告压缩包。

---

#### 09 Agentic Rollout

**背景**：Agentic 场景包含多轮生成、工具执行和多模态 observation，token 对齐与 reward mask 更复杂。

**范围**：`examples/deepeyes_agentic/app/agent.py`、`app/env_deepeyes.py`、`engine/rollout/sglang_rollout.py` 及实际调用到的模块。

**验收标准（Deepeyes 非 agentic）**：
- 分析如何自定义 generate 方法为 deepeyes 专属的 generate
- 从 `rollout.py` 的 generate 开始，说明"模型输出→工具解析→执行→文本/图像 observation 追加→继续模型输出"循环的全链路
- 说明多模态编码（只需分析图片）进入 SGLang 请求的字段
- 分析 partial rollout 下触发中断后 agent 侧如何应对中断、如何在下一轮生成时恢复

**验收标准（Deepeyes agentic）**：
- 分析如何开启 agentic rollout；agentic rollout 必须要设置的参数（不含 optional 的）
- 分析默认情况下 agent 是**何时**启动的，分析这个 feat 的**收益与风险**；找到牺牲收益但减轻风险的启动方式（参数）
- 从 `AgenticSessionShard.chat()` 出发，分析 observation 是如何被编码的，先前历史是否可能被 retokenize 及原因

**验收标准（对比 deepeyes 与 deepeyes_agentic）**：
- 分析 partial rollout 实现的差异，思考 agentic 路线如何做到 agent 无感的 partial rollout
- 说出 2 个被沉淀的对话过程中的通用功能

**交付方式**：技术报告压缩包。

---

### 2.3 模块单测与 Bug 修复

#### 10 DAPO reward 单测

**目标**：为 DAPO 数学 reward 写单测，新增 `tests/engine/rewards/test_math_dapo_utils.py`，覆盖 `relax/engine/rewards/math_dapo_utils.py`。

**覆盖函数**：`last_boxed_only_string`、`remove_boxed`、`normalize_final_answer`、`is_correct_strict_box`、`compute_score`

**运行命令**：
```bash
python -m pytest -v tests/engine/rewards/test_math_dapo_utils.py
```

**关键提示**：
- `compute_score` 返回 **dict**，不是单个浮点数；`score` 只可能是 `+1.0 / -1.0`
- 模型回复构造形如 `"推理过程...... 最终答案是 \\boxed{9}"` 的字符串
- 标准答案（ground truth）直接用字符串，如 `"9"`

**验收标准**：`pytest -v` 全部通过，≥5 条用例，每条断言写明期望值，禁止空断言，纯 Python 无 GPU。

**交付方式**：PR，附测试输出。

---

#### 11 序列长度均衡分区单测

**目标**：为 `relax/utils/data/seqlen_balancing.py` 补齐单元测试，新增 `tests/utils/data/test_seqlen_balancing.py`。

**覆盖函数**：`get_seqlen_balanced_partitions`、`get_reverse_idx`

**运行命令**：
```bash
python -m pytest -v tests/utils/data/test_seqlen_balancing.py
```

**关键提示**：分区返回的是**下标**而非长度值；不要断言分区内元素的具体排列顺序，只断言个数、和、覆盖性等稳定性质。

**验收标准**：全部通过，纯 Python 无 numpy/torch/GPU；涵盖 `equal_size=True/False`、覆盖性/无重复、非法输入异常、`get_reverse_idx` 逆排列正确性，共 ≥5 条。

**交付方式**：PR，附测试输出。

---

#### 12 图像预处理单测

**目标**：为 `relax/utils/multimodal/image_utils.py` 补齐单元测试，新增 `tests/utils/multimodal/test_image_utils.py`。

**覆盖函数**：`get_resize_height_width`、`image_smart_resize`、`to_rgb`、`decode_data_uri`、`load_image`

**运行命令**：
```bash
hf download Qwen/Qwen3-VL-4B-Instruct --local-dir /root/Qwen3-VL-4B-Instruct
export RELAX_TEST_QWEN3_VL_4B=/root/Qwen3-VL-4B-Instruct
python -m pytest -v tests/utils/multimodal/test_image_utils.py
```

**关键提示**：所有测试输入可在内存中构造，无需真实图片文件：
```python
from PIL import Image
img = Image.new("RGB", (宽, 高), (r, g, b))
```

**验收标准**：全部通过，≥6 条用例覆盖 resize 边界、RGBA→RGB 转换、data URI 解码、多输入类型加载、patch 对齐验证；每条断言写明期望值，禁止空断言。

**交付方式**：PR，附测试输出。

---

#### 13 GRPO 的两个核心纯函数单测

**目标**：为 `get_grpo_returns` 与 `compute_approx_kl`（位于 `relax/utils/training/ppo_utils.py`）补齐单元测试，新增 `tests/utils/training/test_ppo_utils_grpo.py`。

**运行命令**：
```bash
python -m pytest -v tests/utils/training/test_ppo_utils_grpo.py
```

**关键提示**：
- 输入均为 CPU torch 张量，无需模型/数据
- 浮点比较用 `torch.allclose(a, b, atol=1e-6)`，不用 `==`
- 在用例开头写 `torch.manual_seed(0)` 保证可复现

**验收标准**：全部通过，纯 CPU torch，禁止依赖 Megatron/CUDA；涵盖 reward 广播、KL 四种估计方式（k1/k2/k3/low_var_kl）、非负性、非法类型异常，共 ≥5 条。

**交付方式**：PR，附测试输出。

---

#### 14 多模态最大 prompt 长度单测

**目标**：为 `relax/utils/data/data_utils.py::check_sample_length` 写单元测试，新增 `tests/utils/data/test_check_sample_length_multimodal.py`。

**运行命令**：
```bash
hf download Qwen/Qwen3-4B --local-dir /root/Qwen3-4B
hf download Qwen/Qwen3-VL-4B-Instruct --local-dir /root/Qwen3-VL-4B-Instruct
export RELAX_TEST_QWEN3_4B=/root/Qwen3-4B
export RELAX_TEST_QWEN3_VL_4B=/root/Qwen3-VL-4B-Instruct
python -m pytest -v tests/utils/data/test_check_sample_length_multimodal.py
```

**关键提示**（transformers 5.3.0 兼容）：文本用例请用 str prompt（走 `tokenizer(text)` 分支），不要用 chat messages 的 list prompt 去断言长度过滤；参考长度一律从 processor/tokenizer 现算，禁止硬编码魔数。

**验收标准**：全部通过（模型缺失时相应用例 `skip`）；涵盖多模态 token 展开验证、边界过滤、图像分辨率与 token 数正相关、文本链路过滤，共 ≥4 条。

**交付方式**：PR，附测试输出。

---

#### 15 Megatron FP16 配置去硬编码

**目标**：将 `relax/backends/megatron/model.py` 中与 FP16 optimizer 相关的硬编码参数移到 argument 中改成可配置项，并在 `scripts/training/text/run-qwen3-4B-fp16-8xgpu.sh` 中显式配置，缺少必要配置时给出 warning。

**验收标准**：模型代码不再无条件覆写该参数；启动脚本显式配置并可由用户覆盖；用户未设置时仅走文档化 fallback 且打印一次可读 warning；默认/显式/非法值均有测试，BF16 等既有路径不回归。

**交付方式**：Issue（先澄清）+ PR。

---

#### 16 Encode 线程池可配置

**背景**：`relax/utils/data/processing_utils.py` 中 `_ENCODE_EXECUTOR = ThreadPoolExecutor(max_workers=32)` 无法适配不同 CPU/GPU 资源。

**目标**：通过 `--encode-max-workers`（可辅以环境变量）配置，参考 `--mm-processor-pool-size` 的传参方式。

**验收标准**：不再硬编码 `max_workers=32`，移除 FIXME；参数从启动入口传到线程池初始化；未设置时使用文档化默认值（如 `min(32, os.cpu_count() or 8)`）；非法值给出明确错误；测试覆盖默认、显式、非法值和 executor 生命周期。

**交付方式**：Issue + PR。

---

## 3. 中级任务（3–7 天）

#### 17 DeepEyes processor 动态 patch

**背景**：`examples/deepeyes/` 当前通过 `cp` 把整份 `qwen_vl.py` 覆盖进 SGLang 安装目录，依赖 `/sgl-workspace` 且易随上游升级失效。

**目标**：启动时用 monkey-patch 或 processor 注册机制，仅注入真实差异。

**验收标准**：删除所有覆盖安装目录的 `cp` 与 `/sgl-workspace` 硬编码；patch 仅覆盖必要方法、重复 import/调用幂等；不兼容 SGLang 版本时 fail fast 并提示；同一输入下 patch 前后关键输出一致；DeepEyes 端到端 smoke 通过；有单测和升级说明。

**交付方式**：Issue + PR。

---

#### 18 Format-aware reward router

**背景**：单一 `--rm-type` 无法支持一个 batch 内的 math、multiple-choice 等混合任务。

**目标**：按 metadata/label 自动分发，支持 fallback 和一行式 registry。

**验收标准**：混合 math + multiple-choice batch 逐样本命中正确 reward；未知/缺失/冲突类型走配置的 fallback 或 0 分并记录 warning；新增 reward 只需注册一行，不改路由分支；保留显式 `--rm-type` 的兼容策略；单测覆盖路由、fallback、registry 和混合 batch。

**交付方式**：Issue + PR。

---

#### 19 Custom reward 并发执行

**背景**：同步自定义 reward 若在 event loop 内串行运行会阻塞 rollout，且未复用 RewardWorker 的进程隔离和并发限制。

**目标**：同步 custom reward 走 worker 池，async custom reward 保持兼容。

**验收标准**：同步 custom reward 在独立进程执行；`reward_max_concurrency`、`reward_num_workers` 生效；async 自定义函数直接 await；函数只加载一次；异常可定位到样本且不死锁。

**交付方式**：Issue + PR。

---

#### 20 优化多轮 Rollout 请求调度

**背景**：当前自定义多轮 rollout 会在整个会话期间占用并发 semaphore，导致长会话持续占用槽位，短请求排队，降低 SGLang 利用率和调度公平性。

**目标**：将并发控制从"会话级"缩小到"单次模型请求级"：每轮模型请求前获取 permit，结束后立即释放；环境和工具执行期间不占用 permit。

**入口**：`relax/engine/rollout/sglang_rollout.py`、`examples/deepeyes/rollout.py`、`docs/zh/guide/customize-training.md`

**验收标准**：提供可复用的单次请求 permit 接口；Deepeyes 每轮推理独立获取和释放 permit；并发请求数不超过配置上限；permit 在成功、异常、取消和 abort 时均能释放；并发上限为 1 时，单轮短请求不必等待多轮长请求全部结束；CPU 异步单测通过。

**交付方式**：Issue + PR（含轮次级请求调度接口、Deepeyes 接入、并发/公平性/异常释放测试、接入说明）。

---

#### 21 Hybrid-async 多模态性能

**背景**：多模态预处理、pixel tensor 传输、rollout/训练流水重叠等是 Hybrid-async 可优化的性能瓶颈。

**入口**：`scripts/training/multimodal/run-qwen35-9B-8xgpu-openr1mm-hybrid-async.sh`（Issue 中可换其他脚本）

**交付物**：PR（以开关方式提交，说明代码改动影响面）+ Issue（优化动机与理论解释、效果数据、改动前后曲线对比图）

**验收标准**：优化效果达到既定目标；动机和理论解释合理；曲线对比无正确性问题；baseline 和实验各跑 ≥2 次。

---

#### 22 Hybrid-async 纯文本性能

**背景**：纯文本场景的 actor/rollout 并行、队列反压，或权重同步可能限制流水利用率。

**入口**：`scripts/training/text/run-qwen3-4B-8xgpu-hybrid-async.sh`（Issue 中可换其他脚本）

**验收标准与交付方式**：与 21 相同。

---

#### 23 Colocate 纯文本性能

**背景**：colocate 模式共享 GPU，模型切换、显存回收和阶段间空泡可能降低吞吐。

**入口**：`scripts/training/text/run-qwen3-4B-8xgpu.sh`（Issue 中可换其他脚本）

**验收标准**：与 21 相同；另报告 rollout/训练阶段 GPU 时间占比、切换或内存清理开销，且无 OOM/残留进程。

---

#### 24 Colocate 多模态性能

**背景**：colocate 模式多模态预处理、共享 GPU，模型切换、显存回收和阶段间空泡可能降低吞吐。

**入口**：`scripts/training/multimodal/run-qwen3-vl-4B-8xgpu.sh`（Issue 中可换其他脚本）

**验收标准**：与 21 相同；另报告 rollout/训练阶段 GPU 时间占比、切换或内存清理开销，且无 OOM/残留进程。

---

## 4. 高级任务（1–2 周）

#### 25 Mixture-of-LoRA RL

**背景**：单 adapter LoRA 容量有限；需要在冻结 base 的前提下，由 router 按 token/序列组合多个 LoRA expert。

**验收标准**：
- `--lora-num-experts 4 --lora-rank 16` 可启用
- base 参数冻结且无梯度，仅 N 个 adapter + router 更新
- 单 expert 配置与既有 LoRA 兼容
- Qwen3-4B、DAPO math、colocate 完成 GRPO ≥200 step，loss/reward 无异常
- 给出相对全参及单 LoRA 的峰值显存、吞吐 before/after
- 输出各 expert 平均激活权重，不能塌缩到单一 expert
- checkpoint 保存/恢复后一致

**交付方式**：设计 Issue + Draft PR；PR 含单测、recipe、文档、实验记录。

---

#### 26 TransferQueue RDMA

**背景**：图片 pixel values 体积大，现有 TransferQueue 传输可能成为多机多模态流水瓶颈。

**目标**：增加 RDMA transport，并在不可用时安全回退。

**验收标准**：单元/集成测试验证 payload 逐字节一致；支持连接建立、背压、超时、断连、重试和清理；无 RDMA 环境自动回退；同拓扑对多档图片 payload 各跑 ≥3 次，有效带宽提升 ≥20% 或 p95 延迟下降 ≥20%；提供开关和运维说明。

**交付方式**：设计 Issue + 分阶段 PR（协议/实现、集成、基准与文档）。

---

#### 27 开放论文复现

**背景**：将 Relax 尚未支持、且有合法开源实现的论文能力转化为可维护的算法、recipe 和复现实验。

**入口**：先提交 Proposal Issue，不预先限定文件。

**验收标准**：开发前由维护者在 Issue 确认论文版本、基线、目标指标、容差、算力预算和范围；实现包含算法开关、最小单测、可直接运行 recipe 和文档；在约定环境完成至少 3 次复现，报告均值/方差并说明与论文的差异；没有获批 Proposal 不进入验收。

**交付方式**：Proposal Issue → 设计评审 → PR → 复现报告。

---

#### 28 解耦算法配置与训练流程，接入 GDPO

**背景**：当前算法通过字符串和大量 if/elif 判断接入，新增算法容易遗漏。同时 Relax 尚不支持 GDPO 所需的多奖励独立归一化。

**目标**：建立统一的算法注册与分发机制；通过新机制接入 GDPO；保持现有算法行为不变。

GDPO 需要：
1. 分别对每个 reward 做组内标准化
2. 合并各 reward 的 normalized advantage
3. 对合并结果做 batch-wise normalization

**入口**：`relax/components/advantages.py`、`relax/backends/megatron/loss.py`、`relax/utils/utils.py`、`relax/core/registry.py`、`relax/utils/arguments.py`

**验收标准**：算法名称、能力和实现通过注册表管理；不再重复维护算法名称列表和完整的算法 if/elif 链；`--advantage-estimator gdpo` 可正常使用；GDPO 支持至少两个可配置的 reward key；GDPO 必须通过注册机制接入，不允许添加 `if algorithm == "gdpo"`；单测覆盖注册/分发、现有算法数值等价性、GDPO 公式、reward-collapse、缺少 reward、非数值 reward、零方差场景；通过 CPU 测试和 `pre-commit run --all-files`。

**交付物**：算法注册与分发模块；现有算法迁移；GDPO 实现；单元测试；包含 correctness、format 两项奖励的最小训练示例；简短的新算法接入文档。

---

## 5. 技术报告交付模板与样例

### 5.1 压缩包结构

**压缩包命名**：`学校+专业+姓名.zip`

```
学校+专业+姓名.zip
├── README.md              # 一页内写明任务、代码 commit、阅读/运行方式
├── 技术报告.pdf            # 建议 2–4 页，复杂图表可放附录
├── evidence/
│   ├── call-chain.png     # 架构图、时序图或关键截图
│   └── trace.txt          # 脱敏后的日志/调用 trace
└── source/                # 可选：Markdown、drawio 等源文件
```

> 不得放入 checkpoint、完整数据集、账号、密钥、token 或内部地址。

---

### 5.2 技术报告模板

```markdown
## 技术报告：<题目名称>

### 1. 任务详情
- 题目编号：
- 姓名 / 学校 / 专业：
- 分析代码仓库与 commit：
- 任务目标：
- 分析范围 / 不在范围内：

### 2. 背景与结论摘要
- 为什么存在这个模块或流程：
- 一句话结论：
- 最容易误解的边界：

### 3. 启动入口与关键代码
- 启动脚本 / 完整命令（如未运行，写"仅静态分析"及原因）：
- 关键文件、类、函数及职责：
- 配置如何进入运行时：

### 4. 架构与调用链
- 架构图 / 时序图：
- 从入口到结果返回的逐步说明：
- 关键数据结构、进程/Actor、同步与异步边界：

### 5. 验证证据
- 使用的日志、断点、最小实验或测试：
- 观察结果与源码结论如何对应：
- 不能确认的内容：

### 6. 遇到的问题与解决方案
- **问题/现象**：
- **根因**：
- **排查过程**：
- **解决方案**：
- **如何避免复发**：

### 7. 总结与参考
- 关键结论：
- 局限与后续建议：
- 源码、Issue、PR、文档链接：
```

---

### 5.3 填写样例：简单 05「三层架构」

> 下列内容展示"证据长什么样"，文件名和函数名必须以认领 commit 的真实源码为准，不可照抄占位符。

**分析 commit**：`<40 位 SHA>`

**目标**：从一个真实启动入口说明 Controller、Service、Component 的职责和调用边界。

**范围**：`relax/core/` 与该调用链直接依赖；不讨论具体模型算子实现。

**结论摘要**：Controller 负责流程编排与生命周期；Service 暴露稳定能力并管理远端执行；Component 封装具体角色或资源。结论由源码链接和一次最小 trace 共同验证。

**调用链**：
1. `train.py::main` 读取配置、初始化 Ray 并创建 Controller
2. `Controller.register_all_serve` 按角色创建 Service，并校验/分配资源
3. `Controller.training_loop` 对齐 step，通过 `Service.set_step/run` 启动各角色
4. `Service._deploy/run` 部署并调用具体 Component；Component 执行业务逻辑并通过约定通道交换结果

（插入一张时序图，标出对象创建、RPC/await、状态写入、异常和清理位置）

---

## 6. PR 交互规范

```markdown
#### Summary
- Closes #<issue>
- 解决了什么问题：
- 为什么采用该方案：

#### Changes
- 文件 / 核心函数 / 行为变化：
- CLI、配置或兼容性变化：

#### Verification
- 环境、硬件、commit：
- 可复制命令：
- 单元/集成/端到端测试结果：
- 性能 before/after（若适用）：
- 日志、曲线、profile 链接：

#### Risk & Rollback
- 已知限制：
- 风险：
- 关闭开关或回退方式：

#### Checklist
- [ ] Diff 仅包含本任务必要改动
- [ ] 新增/相关测试全部通过
- [ ] 文档、脚本和默认值已更新
- [ ] 不含密钥、数据集、checkpoint 或机器隐私信息
- [ ] 已逐条回复 review comment
```

> PR 先以 Draft 提交。验收完成的最低条件是：关联 Issue、CI 通过、可复制验证证据齐全、所有阻塞性 review 已解决、维护者确认达到 Issue 中冻结的标准。只有"代码能运行"或只有截图、没有命令与原始数据，均不视为完成。

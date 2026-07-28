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
- [5. 新增任务（基于 redai-infra/Relax main@8a54679）](#5-新增任务基于-redai-infrarelax-main8a54679)
  - [5.1 算法与 Reward](#51-算法与-reward)
  - [5.2 性能效率](#52-性能效率)
  - [5.3 正确性、复现与质量](#53-正确性复现与质量)
  - [5.4 开源论文复现](#54-开源论文复现)
- [6. 技术报告交付模板与样例](#6-技术报告交付模板与样例)
- [7. PR 交互规范](#7-pr-交互规范)

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
| **技术报告类** | 新手入门任务与简单 05–09 | 按第 6 节模板进行整理，打包压缩包提交 |
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

**交付方式**：按第 6 节提交技术报告压缩包。

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

#### 27 解耦算法配置与训练流程，接入 GDPO

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

## 5. 新增任务（基于 redai-infra/Relax main@8a54679）

> 以下路径仅作为定位提示，基于 redai-infra/Relax main@8a54679；最终改动范围以认领 Issue/RFC 为准。

### 5.1 算法与 Reward

#### 28 RLOO【中等】

**背景**：
RLOO 使用 leave-one-out baseline 降低 policy gradient 方差，可作为 GRPO 之外的重要公开基线。

**目标**：
在算法 registry 中实现 RLOO 的 baseline、advantage、loss 和监控指标，只要求同步训练。

**参考入口（仅供参考）**：
注册与配置：relax/core/registry.py、relax/utils/arguments.py；
算法计算：relax/utils/training/ppo_utils.py、relax/components/advantages.py、relax/backends/megatron/loss.py；
示例与测试：examples/algorithms/、tests/。

**验收标准**：
手工构造 batch 上的 baseline、advantage 和 loss 与参考公式逐元素一致；
group size、padding、空 response 和异常 reward 有明确处理；
DP/CP 切分不改变有效 token 上的统计结果；
完成小模型训练，并给出与 GRPO 的 reward、loss、KL 和吞吐对比；
默认算法行为不受影响，异步 RLOO 不纳入本题。

**交付方式**：
算法 PR、单测、recipe、实验报告和算法文档。

#### 29 REINFORCE++ / REINFORCE++-baseline【中等】

**背景**：
社区存在多种 REINFORCE 改进实现，但命名和 baseline 口径并不统一，需要给出清晰、可验证的 Relax 实现。

**目标**：
实现算法配置、return/advantage、loss 和监控指标，并明确 REINFORCE++、baseline 版本与 GRPO、GSPO、SAPO 的差异。

**参考入口（仅供参考）**：
主干已有部分实现，可先查看 relax/utils/training/ppo_utils.py、relax/components/advantages.py、relax/backends/megatron/loss.py；
注册与配置：relax/core/registry.py、relax/utils/arguments.py；
示例与测试：examples/algorithms/、tests/core/、tests/components/、tests/backends/megatron/。

**验收标准**：
公式、归一化维度、mask 和 reduction 口径写入设计文档；
关键张量与独立参考实现逐元素对齐；
覆盖变长 response、全零 reward、单样本和分布式统计测试；
小模型训练稳定，报告与至少一个现有算法的相同预算对比；
配置命名可区分两个变体，不以复制 recipe 代替算法实现。

**交付方式**：
Proposal Issue → 算法实现 PR → 数值测试与训练报告。

#### 30 Dr.GRPO【中等】

**背景**：
Dr.GRPO 重点修正 GRPO 中与长度归一化、reward normalization 和 loss aggregation 相关的偏差，需要作为独立变体接入并与标准 GRPO 清晰区分。

**目标**：
完成 Dr.GRPO 配置、advantage/loss 实现和回归测试，并提供标准 GRPO 与 Dr.GRPO 的可复现对比。

**参考入口（仅供参考）**：
注册与配置：relax/core/registry.py、relax/utils/arguments.py；
Advantage 与 loss：relax/utils/training/ppo_utils.py、relax/components/advantages.py、relax/backends/megatron/loss.py；
示例与测试：examples/algorithms/、tests/components/、tests/backends/megatron/。

**验收标准**：
设计文档逐项说明与标准 GRPO 的公式差异；
构造长短 response 混合 batch，验证长度与 reduction 行为；
DP/CP、micro-batch 和 padding 不改变最终统计；
完成相同模型、数据和预算下的 GRPO 对比，报告 reward、length、KL 和训练稳定性；

**交付方式**：
算法 PR、测试、recipe、对比报告。

#### 31 DPO / Reward Modeling【高级】

**背景**：
Relax 目前的 Roadmap 以在线 RL 为主，社区仍需要标准的离线 preference optimization 和 reward model 训练能力。

**目标**：
复用 SFT trainer 的数据、模型和训练基础设施，补齐 chosen/rejected 数据处理、DPO loss、Reward Model 训练和公开 recipe。

**参考入口（仅供参考）**：
可复用的 SFT 基础：relax/components/sft.py、relax/engine/sft/；
数据、模型与 loss：relax/utils/data/、relax/backends/megatron/data.py、relax/backends/megatron/model_provider.py、relax/backends/megatron/loss.py；
Recipe 与测试：scripts/training/sft/、tests/engine/sft/、tests/backends/megatron/。

**验收标准**：
chosen/rejected 拼接、mask、长度截断和 label 构造有单测；
DPO loss 与独立参考实现逐元素对齐，reference-free/有 reference 的范围写清；
Reward Model 在公开小数据集上完成训练，chosen score 优于 rejected score；
至少提供一个 DPO 和一个 Reward Modeling recipe；
尽量只新增 loss/dataloader 插件，不复制一套 SFT trainer。

**交付方式**：
RFC → 一个或多个 PR → 数据说明、训练曲线和评测报告。

### 5.2 性能效率

#### 32 FLA Chunkwise CP for GDN【高级】

**背景**：
Relax 当前 _dcp_gdn_forward 的 CP 路径仍有性能优化空间。FLA 新的 chunkwise CP 实现可作为 GDN/DQN 类模型的候选优化方案，但 Relax 的 MCore patch 与上游差异较大，需要分阶段完成。

**目标**：
升级兼容版本的 FLA，引入 chunkwise CP，并完成 Relax MCore patch 与 GDN 接入，使 CP 训练在正确性不回归的前提下提升吞吐或降低显存。

**参考入口（仅供参考）**：
依赖与上游补丁：docker/Dockerfile、docker/patch/megatron/；
GDN/CP 核心：relax/backends/megatron/model.py、relax/backends/megatron/cp_utils.py；
测试与基线：tests/backends/megatron/test_gdn_cp_reassembly.py、scripts/training/sft/ 下的 dynamic-CP recipe。

**验收标准**：
先提交 RFC，说明与 Megatron-LM 上游实现的差异、patch 方案和回退路径；
CP=1 时与原实现等价，CP>1 时 forward/backward 与参考结果满足容差；
覆盖不同序列长度、chunk、padding、TP/CP 组合和 checkpoint；
同环境完成至少 200 step before/after，对比 tokens/s、step time、GPU 利用率和峰值显存；
主要性能指标有明确收益且 loss 曲线、有效 token 数和结果质量不回归。

**交付方式**：
RFC → FLA/MCore 兼容 PR → Relax 接入 PR → 正确性与性能报告。

### 5.3 正确性、复现与质量

#### 33 Reproducibility Bundle【初级】

**背景**：
RL 实验经常因代码、配置、镜像、模型、数据和并行拓扑信息缺失而无法复现，需要形成标准化实验清单。

**目标**：
每次运行自动生成可分享、可检查的 experiment manifest，并提供一条命令检查环境差异或重新执行实验。

**参考入口（仅供参考）**：
启动与配置：relax/entrypoints/train.py、relax/utils/arguments.py、configs/env.yaml；
启动脚本与实验记录：scripts/entrypoint/、relax/utils/tracking_utils.py；
可在 relax/utils/、tests/utils/ 和 tests/integration/ 中扩展实现与测试。

**验收标准**：
manifest schema 有版本号，可向后兼容读取；
至少覆盖本地、单机 Ray 和多节点任务的关键元数据；
token、密码、内部地址等敏感信息默认不落盘；
使用 manifest 能在同环境复跑最小任务，并能指出环境差异；
生成过程不明显影响训练启动时间，失败时不阻塞主任务并给出告警。

**交付方式**：
RFC、实现 PR、schema 文档、脱敏测试和复现实例。

#### 34 Deterministic Trajectory Replay【高级】

**背景**：
线上偶现的 reward、advantage 和 loss 问题通常难以通过完整集群复现。可移植的 trajectory replay 能显著降低调试成本。

**目标**：
将 trajectory、reward 输入、模型响应和必要元数据保存为重放包，在不启动 rollout 集群的情况下重现 reward、advantage 与 loss 计算。

**参考入口（仅供参考）**：
数据结构与导出：relax/utils/types.py、relax/utils/training/train_dump_utils.py；
Reward、advantage 与 loss：relax/engine/rewards/、relax/components/advantages.py、relax/backends/megatron/loss.py；
Agentic 链路与测试：relax/agentic/pipeline/transfer.py、relax/agentic/session/、tests/。

**验收标准**：
RFC 明确最小重放边界，不将完整在线系统复制到本地；
同一代码版本重放的关键张量与原运行一致或满足浮点容差；
支持选择单条、单 batch 和指定 step 重放；
schema 版本不兼容时给出可操作错误，不静默丢字段；
至少用一次真实失败样例或人工注入错误展示定位过程。

**交付方式**：
RFC → schema/runner PR → fixture 与调试案例文档。

#### 35 Config Doctor & Dry-run【初级】

**背景**：
很多训练失败可以在分配 GPU、启动 Ray 和拉起 worker 前发现，包括路径错误、依赖缺失、并行度不合法和参数冲突。

**目标**：
提供 relax doctor 或等价入口，在不启动 GPU worker 的情况下完成静态检查、资源推导和启动计划预览。

**参考入口（仅供参考）**：
参数与校验：relax/utils/arguments.py、relax/backends/megatron/arguments.py、relax/backends/sglang/arguments.py；
拓扑与启动：relax/core/registry.py、relax/core/controller.py、relax/entrypoints/；
配置、脚本与测试：configs/env.yaml、scripts/entrypoint/、scripts/models/、scripts/training/、tests/。

**验收标准**：
dry-run 不分配 GPU，不启动 Ray/SGLang/训练进程；
覆盖不少于 15 类常见配置错误，并提供针对性修复建议；
输出最终合并配置、角色拓扑、资源需求和预计启动命令；
正确配置返回成功，错误配置返回非零退出码，适合接入 CI；
配置检查规则可扩展，新增 backend/算法无需修改单体巨型函数。

**交付方式**：
实现 PR、规则单测、错误样例库和使用文档。

### 5.4 开源论文复现

> 论文复现题均需先提交 Proposal Issue，固定论文版本、参考仓库与 license、基线 commit、模型、数据、算力预算、目标指标、统计次数和允许误差。实现必须包含算法开关、最小数值单测、可直接运行的 Relax recipe 和复现报告；默认配置不得回归，失败实验和无法对齐的部分必须如实记录。

#### 36 MemAgent【高级】

**论文**：MemAgent；参考实现：https://github.com/vllm-project/vime/tree/main/examples/mem_agent

**复现目标**：
在 Relax 中复现长文档分块读取、固定长度记忆更新和最终问答流程，完成基于 GRPO 的多轮训练，并在 HotpotQA、RULER-HQA 上获得接近官方实现的效果。

**Relax 框架修改**：
实现多轮记忆 Rollout
新增 MemAgent 自定义 generate：将长文档按 token 切块，逐块调用 SGLang 更新固定长度 memory，最后仅基于 question + memory 生成答案。可接入现有 --custom-generate-function-path，无需修改 SGLang 核心。
支持多轮轨迹展开训练
每个 memory-update turn 都要保存 tokens、response_length、loss_mask 和 rollout_log_probs。再通过 --custom-convert-samples-to-train-data-path 将一条轨迹展开成多个独立训练样本，并共享或分摊最终 GRPO advantage。
补充数据和奖励函数
增加 HotpotQA 数据转换，至少提供 question/context/ground_truth；实现从最终 \boxed{} 答案计算 exact-match reward。中间 memory turn 不单独打分，使用最终答案奖励做信用分配。
增加训练与评测配置
新增 example 和启动脚本，配置 chunk 大小、memory 长度、最大 chunk 数、GRPO 参数等；另外增加 RULER-HQA 长上下文评测脚本，验证从训练长度泛化到更长文档。
整体上可以主要落在 examples/mem_agent/，复用 Relax 已有的自定义 rollout、reward 和 sample-convert 扩展点，预计不必修改 Controller/Service 核心逻辑。

**复现要求**：
数据一致：使用相同的 HotpotQA 训练集与 RULER-HQA 评测集，保持 question、context、ground truth 的处理方式一致。
Rollout 一致：对齐 chunk 大小、memory 上限、最大 chunk 数、提示词、采样参数及停止条件。
训练一致：对齐模型初始化、GRPO 组大小、batch size、学习率、训练步数和奖励归一化方式；固定随机种子。
结果可验证：至少报告训练 reward、HotpotQA 准确率，以及不同上下文长度下的 RULER-HQA 准确率，并与 VIME 提供的结果或基线模型比较。

**验收标准**：
流程可运行：训练、Checkpoint 转换和评测脚本可一键执行，至少连续训练 2 个 step 无报错、无 hang。
行为正确：长文档按配置切块，memory 每轮覆盖更新且长度不超限；最终回答仅使用 question + memory。
训练数据正确：所有 memory turn 均被展开训练，tokens、loss_mask、rollout_log_probs 长度严格对齐，最终 reward 能正确分配到各轮。
效果达标：训练 reward 有明显提升；HotpotQA 和各长度 RULER-HQA 指标优于基础模型，并达到预先约定的 VIME 复现结果容差，例如绝对差不超过 3%。

#### 37 GraphGPO【高级】

**论文**：GraphGPO；参考实现：verl-agent GraphGPO recipe

**复现目标**：
在 Relax 中复现多轨迹状态图构建、最短路距离估计和逐步图优势分配，并在 ALFWorld、WebShop或Sokoban 上验证其效果和收敛速度优于 GRPO/GiGPO。

**Relax 框架修改**：
环境与多轮 Rollout：接入 ALFWorld、WebShop，完整复现再接视觉 Sokoban；同一任务并行生成多条轨迹，并记录 task_id/trajectory_id/state/action/next_state/success。
状态图构建：将同一任务的轨迹合并成状态转移图，以成功状态为 goal，反向运行 Dijkstra 计算各状态到 goal 的距离；支持精确匹配和可选相似状态合并。
GraphGPO Advantage：按 10 × ω^distance 生成逐步回报，对同一源状态的出边做组内归一化，再与 episode-level GRPO advantage 加权组合，并广播到对应动作 token。
扩展 Relax 数据链路：当前 agentic custom advantage 主要支持轨迹级标量，需要增加 turn-level advantage 及相关 metadata 的传递、展开和训练支持；同时新增 examples/graphgpo/ 的环境、配置、数据转换和启动脚本。

**复现要求**：
环境与状态定义一致：锁定官方 commit、模型、环境版本和 seed；严格复刻 prompt、动作格式及用于构图的确定性 state，否则状态图结构会不同。
Rollout 一致：每个任务 group size 为 8，总计约 128 个环境；对齐最大步数、历史窗口、训练/评测 temperature 和无效动作处理。
算法一致：对齐成功奖励、非法动作惩罚、图距离系数 ω、step/episode advantage 权重和归一化方式；官方不同任务使用不同 ω。
基线完整：使用相同配置分别运行 GRPO、GiGPO 和 GraphGPO，并以多 seed 报告成功率、任务分数、收敛速度及去掉图优势或 episode advantage 的消融。

**验收标准**：
图算法正确：单测覆盖状态合并、重复边、环、不可达节点和最短距离；越接近 goal 的有效边应获得越高回报。
优势分配正确：同源状态出边优势归一化后均值接近 0；失败轨迹中的有效步骤可以获得正优势，且每个动作 token 只接收所属边的 advantage。
流程稳定：端到端训练无 hang、环境泄漏或轨迹错组；图构建与优势计算开销建议低于总迭代时间的 1%——论文报告约为 0.04%。
效果达标：GraphGPO 在相同配置下稳定优于 GRPO/GiGPO；结果与论文相差不超过 3–5 个百分点。完整 Sokoban 复现可参考论文成功率 86.98±0.73。

#### 38 VAGEN-lite【高级】

**论文**：VAGEN(-lite)；参考实现：vagen-lite

**复现目标**：
在 Relax 中复现 Sokoban/FrozenLake 多轮视觉交互，支持 PPO/GRPO 以及 concat、non-concat 两种训练模式，训练效果与官方 Lite 实现基本一致。

**Relax 框架修改**：
环境适配：在 examples/vagen_lite/ 接入 Sokoban、FrozenLake，实现视觉/文本 observation、reset/step/close、动作解析、成功与格式奖励。
多轮视觉 Rollout：实现“图像观察→VLM 输出 <observation>/<think>/<answer>/<prediction>→执行动作→下一观察”，支持最大轮数和历史上下文。
两种训练模式：支持整轨迹 concat；支持 non-concat 按 turn 展开，并保留 group/traj/turn 索引。后者需迁移 Lite 的 turn-level no_concat_gae_first/last。
训练配置：增加 Qwen2.5-VL-3B 的 PPO/GRPO 脚本、reward normalization、低方差 group filtering 及训练/验证环境 YAML。

**复现要求**：
固定 commit、Qwen2.5-VL-3B 权重、环境版本和随机种子。
对齐地图生成、图像渲染与预处理、prompt 模板、动作格式、最大轮数和每轮 token 上限。
对齐 PPO/GRPO、concat/non-concat、batch size、学习率、KL、采样参数及 reward/filter 配置。
在相同配置下分别运行原版 VAGEN-Lite 和 Relax 版本，报告成功率、累计 reward、平均轮数与有效动作率。

**验收标准**：
相同 seed 下，两个实现生成相同初始地图；相同动作序列得到一致的状态、reward 和终止结果。
concat/non-concat 轨迹可逐轮回放，图像、tokens、loss_mask、log-prob、reward 和 turn 索引严格对齐。
PPO 和 GRPO 至少各完成连续训练与评测流程，无 hang、显存持续增长或环境实例泄漏。
Relax 与原版 Lite 在相同配置下的成功率绝对差不超过 3–5 个百分点，训练曲线和 concat/non-concat 的相对趋势一致。

#### 39 EOPD【高级】

**论文**：EOPD；参考实现：WLS04/EOPD

**复现目标**：
在标准 OPD 基础上，只在教师高熵位置额外加入基于 teacher top-k 分布的 forward KL，以兼顾训练稳定性和生成多样性。

**Relax 框架修改**：
Teacher 打分与数据链路：修改 relax/engine/rollout/on_policy_distillation.py 和 relax/utils/types.py，增加逐 token entropy；
复用现有 OPD top-k 字段，新增逐 token teacher_entropy，保证它和 response token、teacher top-k logprobs、loss mask 在 DP/CP/micro-batch 下严格对齐。
Actor 前向与训练目标：修改 relax/utils/opd/opd_utils.py 及现有 Actor/OPD loss 接入位置，同时计算标准 OPD loss 和高熵位置的 teacher top-k forward KL；
正确应用 response、padding 等 loss mask，输出高熵 token 比例、top-k 覆盖率、OPD loss、KL loss 和总 loss。

**修改文件**：
- relax/utils/types.py，增加 teacher_entropy。
- relax/utils/opd/opd_main_worker.py，增加 teacher entropy 的解析/传输字段。
- relax/engine/rollout/on_policy_distillation.py，保存 sample.teacher_entropy，传入 train batch。
- relax/utils/opd/opd_utils.py，增加 EOPD loss 逻辑。
- relax/backends/megatron/loss.py，保持现有 compute_policy_opd_loss 接入口。
- relax/utils/arguments.py，新增参数。

**新增文件**：
- 单测。
- 冒烟测试。

**复现要求**：
使用论文模型、数据集、teacher/student 设置和评测脚本；
对比单独 GRPO、标准 OPD、参考 EOPD 和 Relax EOPD；
固定高熵阈值或选择策略、teacher top-k、KL 系数、训练步数和生成预算；
至少复现论文的一张主结果表和一个高熵位置相关消融实验。

**验收标准**：
逐 token entropy、top-k 概率、筛选 mask 和 forward KL 与参考实现逐元素一致，确保 DP/CP、padding 和 micro-batch 切分不改变有效 token 上的统计结果；
使用论文数据集，完整报告 Avg@8、Pass@8 及论文规定的其他主要指标；
完成不少于 3 个 seed，证明 EOPD 相比单独 GRPO 或标准 OPD 在整体平均表现上取得稳定提升；
同时报告训练吞吐、峰值显存、teacher 打分和 TransferQueue 额外开销。
- 完备的单元测试和冒烟测试。

#### 40 P3O / Adaptive Policy Optimization【高级】

**论文**：Trust the Batch, On- or Off-Policy: Adaptive Policy Optimization for RL Post-Training 参考实现：https://github.com/FeynRL-project/FeynRL

**复现目标**：
将 GRPO/PPO 的固定 clip 替换为基于 batch 内 policy ratio 的 ESS 自适应控制，使训练在 on-policy 与存在 rollout mismatch 的场景下都保持稳定。

**Relax 框架修改**：
实现 P3O loss 和 adaptive cap；
在 DP/CP 全局有效 token 上计算 normalized ESS，正确处理 CP/DP、padding 和 micro-batch；
输出 ESS、adaptive cap、policy ratio、clip fraction、KL 和有效 token 数；
提供 on-policy、可控 off-policy/mismatch 和标准 GRPO 三套对比配置。

**修改文件**：
- relax/utils/training/ppo_utils.py，增加 P3O helper。
- relax/backends/megatron/loss.py，新增 p3o。
- relax/components/advantages.py，把 p3o 加入 GRPO-like advantage 路径。
- relax/utils/utils.py、relax/utils/arguments.py、relax/core/registry.py，增加适配逻辑。

**新增文件**：
- relax/utils/training/p3o_utils.py（尽量不要扩大 relax/utils/training/ppo_utils.py）。
- 冒烟测试脚本。
- 单测。

**复现要求**：
使用论文约定的数据集、模型和主要超参数，明确 ESS 计算范围与全局归约方式；
在相同模型、数据、生成配置和训练预算下对比 GRPO/PPO、参考 P3O 和 Relax P3O；
设计可复现的 rollout mismatch 实验，例如固定 policy lag 或人为扰动 rollout logprob；
至少复现论文的一张稳定性主结果或核心消融表。

**验收标准**：
ESS、adaptive cap 和 P3O loss 与参考公式逐元素一致；
DP/CP 切分和 micro-batch 数量不改变全局 ESS 与最终 loss；
on-policy 时退化到接近普通 policy gradient，训练曲线与基线差异在约定容差内；
人为增加 rollout mismatch 后，P3O 相比标准 GRPO/PPO 的 KL、loss spike、崩溃率或最终效果至少有一项稳定且可解释的改善；
完成不少于 3 个 seed，并报告均值/方差、吞吐与显存开销。
- 完备的单元测试和冒烟测试。

#### 41 SDPO【高级】

**论文**：Reinforcement Learning via Self-Distillation；参考实现：lasgroup/SDPO

**复现目标**：
在获得有效反馈的样本上，由 EMA Student 在增强上下文下充当 self-teacher，并基于 teacher top-k 分布加入 KL/JSD 蒸馏。

**Relax 框架修改**：
实现 SDPO loss 和动态反馈样本路由；
在 reward 后构造 feedback prompt，由 EMA Student 作为 self-teacher 对原 response 进行 top-k 重打分；
正确处理 DP/CP、padding、micro-batch 和无有效反馈样本；
输出有效反馈比例、KL/JSD、top-k 覆盖率、EMA 更新、teacher 额外 forward 次数和耗时。

**修改文件**：
- relax/utils/utils.py，调用 SDPO prompt builder，把 feedback / successful response 转成 sample.teacher_prompt。
- relax/engine/rollout/on_policy_distillation.py，复用现有 OPD teacher prefill 流程，对动态 teacher prompt 下的原 response 做 top-k 打分。
- relax/utils/opd/opd_utils.py，增加 SDPO-lite 需要的 train data 字段消费。
- relax/utils/types.py、relax/utils/arguments.py、relax/utils/opd/opd_opsd_worker.py、relax/backends/megatron/loss.py，增加适配逻辑。

**新增文件**：
- relax/utils/opd/sdpo_prompt_builder.py
- 冒烟测试脚本。
- 单测。

**复现要求**：
使用论文公开模型、feedback 构造、EMA 配置、数据集和评测脚本；
在相同模型、数据、rollout 数和训练预算下对比 GRPO、on-policy GRPO、参考 SDPO 和 Relax SDPO；
至少覆盖 SciKnowEval 和 Tool Use 等公开单轮数据集中的约定任务；
提供无 self-teacher、无 feedback 或不同 KL/JSD 选择的核心消融。

**验收标准**：
feedback 样本选择、EMA teacher 更新、top-k 重打分和 KL/JSD 与参考实现满足约定容差；
DP/CP、padding 和 micro-batch 切分不改变有效 token 上的统计结果；无有效反馈时 loss 正常退化；
完成不少于 3 个 seed，SDPO 的整体平均效果优于 GRPO，并在大多数约定任务上取得稳定提升；
报告每个数据集的完整结果、均值/方差、有效反馈比例和 top-k 覆盖率；
同时报告训练吞吐、峰值显存和 self-teacher 的额外计算开销。
- 完备的单元测试和冒烟测试。

---

## 6. 技术报告交付模板与样例

### 6.1 压缩包结构

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

### 6.2 技术报告模板

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

### 6.3 填写样例：简单 05「三层架构」

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

## 7. PR 交互规范

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

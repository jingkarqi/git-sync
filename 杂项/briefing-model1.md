# 简报：FlashMLA 近期提交中的 MODEL1 信号（基于公开代码）

## 直白变更概述

仅基于公开仓库代码来看，2026‑01‑16 的大提交引入了一个新的模型类型（MODEL1），并围绕它补齐了从参数结构、解码调度、FP8 KV 量化布局到测试覆盖的完整链路；而 2026‑01‑20 的 “nits” 提交则在公开接口文档里删除了对 MODEL1 的说明。整体信号不是普通重构，更像是在为一套新模型/新配置做工程落地与低调铺垫。

直观地说：除了新增模型类型本身，MODEL1 还带来了“额外 KV/动态 topk”的解码路径（可以理解为多了一条额外记忆通道）、独立的 FP8 KV 格式与量化尺度（引入 e8m0 scale），以及在 SM100 上的 head128 原生实现（V3.2 则退化为 head64×2）。这些变化共同指向一个结构和推理路径都与 V3.2 明显不同的模型形态。

需要说明的是，以下判断都来自代码实现与测试参数，不代表官方发布信息；我会把硬事实与推断分开列出，并给出严格的代码出处。

## 结论清单（硬事实 / 推断）

### 硬事实

- csrc/params.h:5-8, 63-103：新增 ModelType::MODEL1，并在 SparseAttnDecodeParams 中加入 model_type、extra_kv/extra_indices/extra_topk_length、topk_length。变更内容：解码参数结构支持 MODEL1 与“额外 KV + 动态 topk”。解释变更：代码显式区分模型类型并为双 KV 路径预留参数。**结论：MODEL1 不是简单兼容模式，而是具备额外注意力路径的独立形态。**
- csrc/api/sparse_decode.h:14-27, 318-345：新增 DecodeFeatures::MODEL1_KVCACHE_FORMAT，并把 d_qk == 512 映射到 MODEL1。变更内容：特征枚举与模型选择逻辑支持 MODEL1。解释变更：以 head_dim 作为模型分流依据。**结论：MODEL1 的结构与 V3.2 不同，具备可被调度器识别的独立特征。**
- csrc/api/sparse_decode.h:289-305：新增 MODEL1 的 bytes_per_token 计算公式（448 + 64*2 + (448/64)*1 + 1）。变更内容：KV cache 结构尺寸显式区分 V3.2 与 MODEL1。解释变更：不同模型使用不同 FP8 KV 内存布局。**结论：MODEL1 的 KV cache 布局已定型且与 V3.2 不兼容。**
- csrc/api/sparse_decode.h:110-180：SM100 上 head128 实现对 MODEL1 有原生路径，V3.2 走 head64×2。变更内容：SM100 kernel dispatch 对 MODEL1 单独优化。解释变更：MODEL1 在新硬件上有专门实现路径。**结论：MODEL1 被视为重点目标平台的原生配置。**
- csrc/sm90/decode/sparse_fp8/splitkv_mla.cuh:693-701, 531-534：MODEL1 才启用 topk_length/extra_topk_length/extra_kv，V3.2 明确不支持。变更内容：模型分支决定功能开关与越界保护。解释变更：MODEL1 允许动态 topk 与额外 KV；V3.2 关闭。**结论：MODEL1 的推理路径更复杂，支持双 KV/变长 topk。**
- csrc/sm100/decode/head64/kernel.cuh:533-554, 701-703, 865-867：RoPE 处理策略在 V3.2 与 MODEL1 上不同，并对 MODEL1 做额外 token 有效性判定与 stride 约束。变更内容：MODEL1 RoPE 位置/耦合方式改变，且需要严格连续 KV 布局。解释变更：MODEL1 的内部张量布局与计算路径与 V3.2 实质性分叉。**结论：MODEL1 的实现细节已深度定制，非临时兼容。**
- tests/quant.py:6-15, 55-75, 106-117：新增 FP8KVCacheLayout.MODEL1_FP8Sparse，d_nope=448/d_rope=64/tile=64/num_tiles=7，并引入 float8_e8m0 scale。变更内容：MODEL1 有独立 FP8 量化/反量化实现与量化尺度类型。解释变更：不同模型采用不同量化尺度与内存布局（可视作“量化标尺”变了）。**结论：MODEL1 的 KV 量化方案是独立设计，引入 Float8_E8M0 Scale。**
- tests/test_flash_mla_sparse_decoding.py:102-112：新增 MODEL1 CONFIG1-4（topk=128 + extra_topk=512/1024 + 可变 extra_topk_length）。变更内容：测试覆盖 MODEL1 的“主 KV + 额外 KV”组合。解释变更：MODEL1 的“额外 KV/变长 topk”是预期使用场景。**结论：MODEL1 具备生产级用例的参数组合。**
- flash_mla/flash_mla_interface.py:72-99（2026‑01‑20 nits）：删除 MODEL1 与特定 KV 布局说明。变更内容：公开接口文档去掉 MODEL1 描述。解释变更：对外文档层面刻意降噪/隐藏。**结论：MODEL1 功能仍在代码中，但被弱化公开叙述。**

### 推断

- tests/quant.py:6-15, csrc/api/sparse_decode.h:289-305：V3.2 与 MODEL1 的 bytes_per_token 明显不同。变更内容：KV token 尺寸变小。解释变更：KV cache 体积缩小通常意味着更低显存占用。**结论：MODEL1 可能在内存效率上有明确目标（倾向性判断）。**
- csrc/params.h:63-103, tests/test_flash_mla_sparse_decoding.py:102-112：MODEL1 支持 extra_kv/extra_topk 并在测试中设为“生产配置”。变更内容：额外 KV 作为常规路径出现，像是**第二条记忆来源。** 解释变更：更像“主上下文 + 额外检索/记忆”的双来源注意力。**结论：MODEL1 可能引入检索式或分层记忆的推理设计（倾向性判断）。**

## 代码出处索引

按“结论清单”的条目顺序列出：

1) csrc/params.h:5-8, 63-103  
2) csrc/api/sparse_decode.h:14-27, 318-345  
3) csrc/api/sparse_decode.h:289-305  
4) csrc/api/sparse_decode.h:110-180  
5) csrc/sm90/decode/sparse_fp8/splitkv_mla.cuh:693-701, 531-534  
6) csrc/sm100/decode/head64/kernel.cuh:533-554, 701-703, 865-867  
7) tests/quant.py:6-15, 55-75, 106-117  
8) tests/test_flash_mla_sparse_decoding.py:102-112  
9) flash_mla/flash_mla_interface.py:72-99  
10) tests/quant.py:6-15, csrc/api/sparse_decode.h:289-305  
11) csrc/params.h:63-103, tests/test_flash_mla_sparse_decoding.py:102-112

## 限制与非结论

- 仅基于公开仓库代码与当前本地快照行号整理，后续提交可能改变行号或内容。
- 未对任何非公开信息做验证，不能等同于官方发布或时间表确认。
- “推断”条目来源于工程痕迹与测试配置，不保证真实产品形态或最终方案。
- 未评估模型效果与性能，仅基于代码结构与测试参数做归纳。

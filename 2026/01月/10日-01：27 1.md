这份修订后的技术路线图在保留原有核心理念的基础上，**针对性地解决了“颗粒度破碎”、“混合字体污染”以及“元数据特征缺失”等工程陷阱**。

我们将单纯的“聚类”升级为**“自底向上的融合与自顶向下的正则化”**双向过程，以确保该系统能达到甚至超越图示的解析精度。

---

### **修订版项目提案：基于多流形拓扑映射与自适应融合的 PDF 语义重构系统**
**Revised Proposal: Semantic PDF Reconstruction via Multi-Manifold Topological Mapping and Adaptive Fusion**

#### **1. 问题定义 (Problem Statement)**
现有的 PDF 解析方案陷入了两个极端：OCR 方案丢失了字符的精确编码信息，导致公式幻觉（Hallucination）；而基于规则的解析器受困于 PDF 原始数据流的**极度碎片化（Fragmentation）**——即一个完整的单词或公式往往被截断为数十个独立的绘制指令（Draw Instructions）。本项目旨在构建一种**抗破碎、语义感知**的解析框架，通过高维特征空间的无监督学习，实现从物理坐标到逻辑结构的逆向工程，特别专注于高保真科技文献翻译场景。

#### **2. 方法论框架 (Methodological Framework)**

本项目提出 **“融合-聚类-正则化” (Fusion-Clustering-Regularization)** 三阶段处理管线，取代传统的单步分类策略。

**2.1 阶段一：自适应跨度融合 (Adaptive Span Fusion)**
针对 PDF 指令的原子化问题（如将单词 "Optimization" 拆分为 `["Op", "ti", "..."]`），在进入特征空间前引入预处理层。
*   **同态合并 (Homomorphic Merging)**：基于欧氏距离阈值（$\epsilon_{dist}$）和基线对齐（Baseline Alignment），将物理相邻且字体属性（FontID, Size, Color）完全一致的字符碎片合并为**原子语义单元 (Atomic Semantic Units, ASUs)**。
*   **目的**：解决“微小碎片信息熵计算失效”的问题，确保后续特征提取是基于有意义的文本块而非孤立字符。

**2.2 阶段二：增强型高维特征嵌入 (Augmented High-Dimensional Embedding)**
将 ASUs 映射为 $\mathbb{R}^n$ 空间向量，在原有视觉特征基础上，针对“引用”和“混合公式”引入关键维度：
*   **交互元数据 (Interaction Metadata)**：引入二值特征维度，标识 ASU 是否被 Link Annotation（如超链接、引用跳转）覆盖。此特征将使**参考文献（Citations）**在特征空间中与纯文本正文显著分离（解决图示绿框识别问题）。
*   **字形拓扑特征 (Glyph Topology)**：计算 ASU 中 ASCII 字符与非 ASCII 字符（希腊字母、数学算子）的比率，以及字形边界框的宽高比（Aspect Ratio）。
*   **相对排版特征 (Relative Typographic Features)**：摒弃绝对字号，采用 $Size_{relative} = Size_{current} / Size_{mode}$，确保对不同文档的缩放不变性。

**2.3 阶段三：基于密度的流形聚类 (Density-Based Manifold Clustering)**
利用改进的 **HDBSCAN** 算法处理特征空间，识别语义类别。
*   **正文主模态 (Body Modality)**：特征空间中密度最大、样本数最多的簇自动标记为正文背景。
*   **语义流形分离**：
    *   **行内公式 (Inline Math)**：高熵、非标准字体、几何位置处于行内的微小簇（解决图示小红框）。
    *   **独行公式 (Display Math)**：高熵、几何居中、左右留白显著的稀疏簇（解决图示大红框）。
    *   **标题 (Headers)**：字号显著偏离主模态的正向离群簇。

**2.4 阶段四：空间正则化与上下文平滑 (Spatial Regularization & Context Smoothing)**
**这是解决“混合字体污染”的关键步骤。** 原始聚类可能将公式中的普通文本（如公式 $A_{old}$ 中的下标 "old"）误判为正文。
*   **K-近邻平滑 (KNN Smoothing)**：对于置信度边缘的 ASU，检查其物理坐标邻域内的 $k$ 个最近邻居。如果一个普通字体 ASU 被大量“数学类 ASU”包围，则依据**空间吸附效应（Spatial Adsorption）**将其类别强制修正为“公式”。
*   **目的**：确保公式内部的文本组件不被撕裂，保证图示红框的完整性。

#### **3. 序列化与生成 (Serialization & Generation)**

**3.1 逻辑版面分析 (Logical Layout Analysis)**
在完成分类后，采用 **XY-Cut** 算法或**拓扑排序**，将散落在页面各处的语义块重组为线性的阅读流（Reading Order）。

**3.2 标记注入翻译 (Tag-Injected Translation)**
构建上下文感知的 Prompt，指导 LLM 进行“带壳翻译”：
*   **非破坏性处理**：检测到的公式块被封装在 `<math id="idx">...</math>` 标签中，引用块被封装在 `<cite>...</cite>` 中。
*   **指令示例**：*"Translate the text content within `<body>` tags, but keep content inside `<math>` and `<cite>` tags strictly verbatim."*

#### **4. 预期突破与技术壁垒 (Expected Breakthroughs)**

1.  **极端排版鲁棒性**：通过“主模态统计”而非硬阈值，系统能完美解析双栏、三栏、甚至非标准页边距的文档，无需人工调参。
2.  **像素级公式提取**：结合“信息熵特征”与“空间正则化”，解决了传统方案无法区分**加粗文本（Bold Text）**与**数学向量（Math Vectors）**的难题。
3.  **零样本引用识别**：通过引入元数据特征，无需训练特定模型即可精准定位参考文献锚点（Citations），为学术文献的交互式阅读（Interactive Reading）奠定基础。

---

### **修改说明 (Key Improvements)**

1.  **增加了“自适应跨度融合”**：专门为了应对你担心的 PDF 碎片化问题。没有这一步，聚类算法会因为输入数据太琐碎而充满噪声。
2.  **引入了“空间正则化”**：这是为了实现图示中红框精准度的核心。它模仿了人类的视觉习惯——“如果一堆乱码里夹着一个英文单词，那个单词通常也是公式的一部分”。
3.  **强化了“交互元数据”**：明确回应了图示中绿色框（引用）的识别逻辑，单纯靠视觉很难区分引用和正文，加上 Link 特征后则变得显而易见。
# RecipeRAG - 智能食谱问答系统

<div align="center">

🍽️ 基于 RAG 技术的智能食谱助手 | 解决"今天吃什么"的世纪难题

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/LangChain-0.3-green.svg)](https://python.langchain.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

## 📖 项目简介

RecipeRAG 是一个基于 **检索增强生成（RAG）** 技术的智能食谱问答系统。它能够理解用户的自然语言问题，从食谱知识库中检索相关信息，并生成准确、实用的烹饪指导。

### ✨ 核心特性

- 🎯 **智能问答**：支持自然语言提问，如"宫保鸡丁怎么做？"
- 🔍 **精准检索**：结合向量搜索和元数据过滤，快速定位相关食谱
- 📊 **多维筛选**：支持按菜品分类、难度等级进行筛选
- 📝 **结构化输出**：提供分步骤的详细烹饪指导
- 💡 **智能路由**：自动识别查询类型（列表推荐/详细指导/一般咨询）
- ⚡ **流式响应**：支持实时流式输出，提升用户体验
- 🔄 **查询优化**：智能重写模糊查询，提高检索准确率

## 🛠️ 技术架构

### 核心技术栈

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| **大语言模型** | Kimi (Moonshot AI) | 用于生成回答和查询优化 |
| **向量嵌入** | BAAI/bge-small-zh-v1.5 | 中文优化的文本嵌入模型 |
| **向量数据库** | FAISS | Facebook AI 相似性搜索库 |
| **应用框架** | LangChain 0.3+ | LLM 应用开发框架 |
| **文本分割** | MarkdownHeaderTextSplitter | 基于标题结构的智能分块 |
| **混合检索** | BM25 + Vector Search | 关键词匹配 + 语义搜索 |

### 系统架构图

```
flowchart TD
    %% 系统初始化
    START[🚀 系统启动] --> CONFIG[⚙️ 加载配置<br/>RAGConfig]
    CONFIG --> INIT[🔧 初始化模块]
    
    %% 索引加载/构建
    INIT --> INDEX_CHECK{📂 检查索引缓存}
    INDEX_CHECK -->|存在| LOAD_INDEX[⚡ 加载已保存索引<br/>秒级启动]
    INDEX_CHECK -->|不存在| BUILD_NEW[🔨 构建新索引]
    
    %% 构建新索引的顺序流程
    BUILD_NEW --> DataPrep
    DataPrep --> IndexBuild
    IndexBuild --> SAVE_INDEX[💾 保存索引到配置路径]
    
    %% 加载已有索引也需要数据准备（用于检索模块）
    LOAD_INDEX --> DataPrepForRetrieval[📚 加载文档和分块<br/>用于检索模块]
    DataPrepForRetrieval --> READY[✅ 系统就绪]
    SAVE_INDEX --> READY
    
    %% 用户交互开始
    READY --> A[👤 用户输入问题]
    A --> B{🎯 查询路由}
    
    %% 查询路由分支
    B -->|list| C[📋 推荐查询]
    B -->|detail| D[📖 详细查询] 
    B -->|general| E[ℹ️ 一般查询]
    
    %% 查询重写逻辑 - 合并相同处理
    C --> KEEP[📝 保持原查询]
    D --> KEEP
    E --> REWRITE[🔄 查询重写]
    
    %% 所有查询都进入统一的检索流程
    KEEP --> F[🔍 混合检索<br/>top_k=config.top_k]
    REWRITE --> F
    
    %% 检索阶段
    F --> G[📊 向量检索<br/>config.embedding_model]
    F --> H[🔤 BM25检索<br/>关键词匹配]
    
    %% RRF重排
    G --> I[⚡ RRF重排融合]
    H --> I
    I --> J[📖 检索到子块]
    
    %% 父子文档处理
    J --> K[🧠 智能去重<br/>按相关性排序]
    K --> L[📚 获取父文档]
    
    %% 生成阶段 - 根据路由类型选择不同模式
    L --> M{🎨 生成模式路由}
    M -->|list查询| N[📋 生成菜品列表<br/>简洁输出]
    M -->|detail查询| O[📝 分步指导模式<br/>config.llm_model<br/>详细步骤]
    M -->|general查询| P[💬 基础回答模式<br/>config.temperature<br/>一般信息]
    
    %% 输出结果
    N --> Q[✨ 返回结果]
    O --> Q
    P --> Q
    
    %% 数据准备子流程
    subgraph DataPrep [📚 数据准备模块]
        R[📁 加载Markdown文件<br/>config.data_path] --> S[🔧 元数据增强]
        S --> T[✂️ 按标题分块]
        T --> U[🏷️ 父子关系建立]
        U --> CHUNKS[📦 输出文本块chunks]
    end
    
    %% 索引构建子流程  
    subgraph IndexBuild [🔍 索引构建模块]
        CHUNKS --> V[🤖 BGE嵌入模型<br/>config.embedding_model]
        V --> W[📊 FAISS向量索引]
        W --> X[💾 索引持久化<br/>config.index_save_path]
    end
    
    %% 配置管理子流程
    subgraph ConfigMgmt [⚙️ 配置管理]
        CFG1[🎛️ 默认配置<br/>DEFAULT_CONFIG]
        CFG2[🔧 自定义配置<br/>RAGConfig]
        CFG3[🌐 环境变量<br/>HF_ENDPOINT]
    end
    
    %% 连接配置到各模块
    ConfigMgmt --> DataPrep
    ConfigMgmt --> IndexBuild
    ConfigMgmt --> F
    ConfigMgmt --> O
    ConfigMgmt --> P
    
    %% 样式定义
    classDef startup fill:#e3f2fd,stroke:#0277bd,stroke-width:2px
    classDef config fill:#f1f8e9,stroke:#388e3c,stroke-width:2px
    classDef userInput fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef routing fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef rewrite fill:#e8eaf6,stroke:#3f51b5,stroke-width:2px
    classDef retrieval fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef generation fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef output fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef module fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef cache fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    classDef dataflow fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    
    %% 应用样式
    class START,INIT startup
    class CONFIG,ConfigMgmt,CFG1,CFG2,CFG3 config
    class INDEX_CHECK,LOAD_INDEX,SAVE_INDEX cache
    class A userInput
    class B,C,D,E,M routing
    class KEEP,REWRITE rewrite
    class F,G,H,I,J,K,L retrieval
    class N,O,P generation
    class Q output
    class DataPrep,IndexBuild module
    class BUILD_NEW,READY,DataPrepForRetrieval startup
    class CHUNKS dataflow

```

## 🚀 快速开始

### 前置要求

- Python 3.8 或更高版本
- Moonshot AI API 密钥（[申请地址](https://platform.moonshot.cn/)）
- 食谱数据（Markdown 格式）

### 安装步骤

#### 1. 克隆项目

```bash
git clone https://github.com/EDC-zh/RecipeRAG.git
cd RecipeRAG
```

#### 2. 创建虚拟环境（推荐）

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/Mac
python3 -m venv venv
source venv/bin/activate
```

#### 3. 安装依赖

```bash
pip install -r requirements.txt
```

#### 4. 配置 API 密钥

在项目根目录创建 `.env` 文件：

```env
MOONSHOT_API_KEY=your_moonshot_api_key_here
```

> ⚠️ **注意**：请勿将 `.env` 文件提交到版本控制系统

#### 5. 准备食谱数据

将 Markdown 格式的食谱文件放入 `data/cook` 目录：

```
data/cook/
├── meat_dish/          # 荤菜
│   ├── 宫保鸡丁.md
│   └── 红烧肉.md
├── vegetable_dish/     # 素菜
│   └── 麻婆豆腐.md
├── soup/              # 汤品
└── ...
```

**食谱文件格式示例**：

```markdown
# 宫保鸡丁

难度：★★★

## 必备原料
- 鸡胸肉 300g
- 花生米 50g
- ...

## 操作步骤
1. 鸡肉切丁，用料酒腌制
2. ...
```

#### 6. 运行系统

```bash
python main.py
```

## 💬 使用指南

### 交互式问答

启动系统后，进入交互式问答模式：

```
============================================================
🍽️  尝尝咸淡RAG系统 - 交互式问答  🍽️
============================================================
💡 解决您的选择困难症，告别'今天吃什么'的世纪难题！

您的问题: 宫保鸡丁怎么做？
是否使用流式输出? (y/n, 默认y): y

回答:
[流式输出回答内容...]
```

### 支持的查询类型

#### 1️⃣ 详细制作指导

询问具体菜品的制作方法：

```bash
❓ 宫保鸡丁怎么做？
❓ 红烧肉需要什么食材？
❓ 麻婆豆腐的制作步骤
```

系统会返回：
- 🥘 菜品介绍
- 🛒 所需食材清单
- 👨‍🍳 详细制作步骤
- 💡 烹饪技巧（如有）

#### 2️⃣ 菜品推荐

获取菜品列表推荐：

```bash
❓ 推荐几个川菜
❓ 有什么简单的家常菜？
❓ 给我3个素菜推荐
```

系统会返回符合条件的菜品名称列表。

#### 3️⃣ 分类筛选

按分类或难度筛选：

```bash
❓ 简单的荤菜有哪些？
❓ 困难的汤品推荐
❓ 非常简单的早餐
```

支持的筛选条件：
- **分类**：荤菜、素菜、汤品、甜品、早餐、主食、水产、调料、饮品
- **难度**：非常简单、简单、中等、困难、非常困难

### 示例对话

```bash
# 示例 1：查询具体做法
用户: 宫保鸡丁怎么做？
系统: [返回详细的分步骤制作指导]

# 示例 2：获取推荐
用户: 推荐几个简单的川菜
系统: 为您推荐以下菜品：
     1. 麻婆豆腐
     2. 回锅肉
     3. 鱼香肉丝

# 示例 3：筛选查询
用户: 简单的素菜有哪些？
系统: [返回符合"简单"难度和"素菜"分类的菜品列表]
```

## 📁 项目结构

```
RecipeRAG/
├── config.py                       # 系统配置文件
├── main.py                         # 主程序入口
├── requirements.txt                # Python 依赖包
├── .env.example                    # 环境变量示例
├── .gitignore                      # Git 忽略配置
├── LICENSE                         # MIT 许可证
├── README.md                       # 项目说明文档
│
├── rag_modules/                    # RAG 核心模块
│   ├── __init__.py                 # 模块导出
│   ├── data_preparation.py         # 数据准备与预处理
│   ├── index_construction.py       # 向量索引构建
│   ├── retrieval_optimization.py   # 检索优化（混合搜索）
│   └── generation_integration.py   # LLM 集成与回答生成
│
├── vector_index/                   # 向量索引存储（运行时生成）
│   ├── index.faiss                 # FAISS 索引文件
│   └── index.pkl                   # 元数据 pickle 文件
│
└── data/                           # 食谱数据目录（需自行准备）
    └── cook/                       # 食谱文件夹
        ├── meat_dish/              # 荤菜
        ├── vegetable_dish/         # 素菜
        ├── soup/                   # 汤品
        └── ...
```

### 核心模块说明

| 模块 | 功能 | 主要特性 |
|------|------|----------|
| **DataPreparation** | 数据加载与预处理 | Markdown 解析、元数据提取、智能分块 |
| **IndexConstruction** | 向量索引构建 | FAISS 索引、持久化存储 |
| **RetrievalOptimization** | 检索优化 | 混合搜索、元数据过滤、父子文档关联 |
| **GenerationIntegration** | 回答生成 | 查询路由、智能重写、流式输出 |

## ⚙️ 配置说明

在 [`config.py`](config.py) 中可以自定义以下参数：

### 路径配置

| 参数 | 说明 | 默认值 | 修改建议 |
|------|------|--------|----------|
| `data_path` | 食谱数据目录路径 | `./data/cook` | 根据实际数据位置调整 |
| `index_save_path` | 向量索引保存路径 | `./vector_index` | 一般无需修改 |

### 模型配置

| 参数 | 说明 | 默认值 | 可选值 |
|------|------|--------|--------|
| `embedding_model` | 文本嵌入模型 | `BAAI/bge-small-zh-v1.5` | 其他 HuggingFace 中文模型 |
| `llm_model` | 大语言模型 | `kimi-k2-0711-preview` | Moonshot AI 支持的模型 |

### 检索配置

| 参数 | 说明 | 默认值 | 建议范围 |
|------|------|--------|----------|
| `top_k` | 检索返回的文档数量 | `3` | 1-10，越大越全面但可能引入噪声 |

### 生成配置

| 参数 | 说明 | 默认值 | 建议范围 |
|------|------|--------|----------|
| `temperature` | 生成随机性 | `0.1` | 0-1，越低越确定，越高越创意 |
| `max_tokens` | 最大生成长度 | `2048` | 根据实际需求调整 |

### 配置示例

```python
from config import RAGConfig

# 自定义配置
custom_config = RAGConfig(
    data_path="./my_recipes",      # 自定义数据路径
    top_k=5,                        # 检索5个文档
    temperature=0.3,                # 稍微增加创造性
    max_tokens=3072                 # 允许更长的回答
)

# 使用自定义配置创建系统
from main import RecipeRAGSystem
rag_system = RecipeRAGSystem(config=custom_config)
```

## 📝 注意事项

1. **API 密钥安全**
   - 妥善保管 `MOONSHOT_API_KEY`，不要泄露给他人
   - 不要将 `.env` 文件提交到公开仓库
   - 定期检查和轮换 API 密钥

2. **数据路径配置**
   - 确保 `config.py` 中的 `data_path` 指向正确的食谱目录
   - 使用相对路径时，注意运行目录的位置
   - 建议使用绝对路径避免路径问题

3. **性能优化**
   - 首次运行会下载嵌入模型（约 100MB），请确保网络畅通
   - 大量食谱时，考虑使用 GPU 加速嵌入计算
   - 可调整 `top_k` 平衡检索速度和准确性

4. **Python 版本**
   - 推荐使用 Python 3.8 - 3.11
   - Python 3.12+ 可能存在兼容性问题

5. **内存占用**
   - FAISS 索引会占用一定内存
   - 大规模数据建议使用 FAISS-GPU 版本
   - 可通过 `faiss-cpu` 或 `faiss-gpu` 选择安装

## ❓ 常见问题

### Q1: 如何获取 Moonshot AI API 密钥？

**A:** 访问 [Moonshot AI 开放平台](https://platform.moonshot.cn/) 注册账号，然后在控制台获取 API 密钥。

### Q2: 首次运行需要多长时间？

**A:** 首次运行需要构建向量索引，时间取决于食谱数量：
- 少量食谱（<100）：约 1-2 分钟
- 中等规模（100-500）：约 5-10 分钟
- 大规模（>500）：可能需要 10 分钟以上

后续运行会直接加载已保存的索引，启动速度很快。

### Q3: 支持哪些数据格式？

**A:** 目前支持 **Markdown (.md)** 格式的食谱文件。文件应包含：
- 一级标题（`#`）：菜品名称
- 二级标题（`##`）：原料、步骤等章节
- 难度标识：使用星级符号（★）

### Q4: 向量索引文件太大怎么办？

**A:** 有以下几种解决方案：
1. 将 `vector_index/` 添加到 `.gitignore`（已配置）
2. 每次使用时重新构建索引
3. 使用更小的嵌入模型（如 `bge-tiny`）
4. 减少食谱数量或精简内容

### Q5: 如何提高检索准确率？

**A:** 建议优化食谱数据质量：
1. 使用清晰的 Markdown 标题结构
2. 在内容中包含丰富的关键词
3. 确保菜品名称准确规范
4. 添加分类和难度信息

### Q6: 可以自定义回答风格吗？

**A:** 可以！修改 [`generation_integration.py`](rag_modules/generation_integration.py) 中的 prompt 模板，调整 LLM 的回答风格和格式。

### Q7: 支持其他大语言模型吗？

**A:** 理论上支持任何 LangChain 兼容的 LLM。修改 `GenerationIntegrationModule` 中的模型初始化代码即可，但需要适配相应的 API。

## 📄 许可证

本项目采用 [MIT License](LICENSE) - 详见 [LICENSE](LICENSE) 文件

## 🙏 致谢

- [LangChain](https://python.langchain.com/) - LLM 应用开发框架
- [FAISS](https://faiss.ai/) - 向量相似性搜索库
- [Moonshot AI](https://platform.moonshot.cn/) - 大语言模型服务
- [HuggingFace](https://huggingface.co/) - 开源模型社区

## 📧 联系方式

如有问题或建议，欢迎：
- 提交 [Issue](https://github.com/EDC-zh/RecipeRAG/issues)
- 发起 [Discussion](https://github.com/EDC-zh/RecipeRAG/discussions)

---

<div align="center">

**⭐ 如果这个项目对你有帮助，请给个 Star 支持一下！**

Made with ❤️ by RecipeRAG Team

</div>

## 🤝 贡献指南

欢迎贡献代码、报告问题或提出建议！

### 贡献方式

1. **报告 Bug**：提交 Issue，详细描述问题和复现步骤
2. **功能建议**：提交 Feature Request，说明需求和使用场景
3. **代码贡献**：
   - Fork 本仓库
   - 创建特性分支 (`git checkout -b feature/AmazingFeature`)
   - 提交更改 (`git commit -m 'Add some AmazingFeature'`)
   - 推送到分支 (`git push origin feature/AmazingFeature`)
   - 提交 Pull Request

### 开发环境设置

```bash
# 1. Fork 并克隆仓库
git clone https://github.com/YOUR_USERNAME/RecipeRAG.git
cd RecipeRAG

# 2. 安装开发依赖
pip install -r requirements.txt

# 3. 创建分支
git checkout -b feature/your-feature-name

# 4. 开发完成后提交 PR
```

### 代码规范

- 遵循 PEP 8 代码风格
- 添加必要的注释和文档字符串
- 保持代码简洁和可读性
- 测试新功能后再提交

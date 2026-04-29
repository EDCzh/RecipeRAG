# RecipeRAG - 智能食谱问答系统

基于 RAG（检索增强生成）技术的智能食谱问答系统，帮助您解决"今天吃什么"的难题！

## 功能特点

- 🍽️ 智能食谱检索与问答
- 🔍 支持按分类、难度筛选菜品
- 📝 提供详细的烹饪步骤指导
- 💡 智能查询重写与路由
- ⚡ 流式输出支持

## 技术栈

- **语言模型**: Kimi (Moonshot AI)
- **向量嵌入**: BAAI/bge-small-zh-v1.5
- **向量数据库**: FAISS
- **框架**: LangChain

## 安装步骤

### 1. 克隆项目
```bash
git clone https://github.com/your-username/RecipeRAG.git
cd RecipeRAG
```

### 2. 安装依赖
```bash
pip install -r requirements.txt
```

### 3. 配置 API 密钥
创建 `.env` 文件并添加：
```env
MOONSHOT_API_KEY=your_api_key_here
```

### 4. 准备数据
将食谱数据放入 `data/cook` 目录（具体格式参考示例数据）

### 5. 运行系统
```bash
python main.py
```

## 使用方法

启动后进入交互式问答模式：
- 输入菜品名称或烹饪相关问题
- 可选择流式输出或非流式输出
- 支持按分类、难度筛选

### 示例问题

- "宫保鸡丁怎么做？"
- "推荐一些川菜"
- "简单的家常菜有哪些？"
- "麻婆豆腐需要什么食材？"

## 项目结构

```
RecipeRAG/
├── config.py                 # 配置文件
├── main.py                   # 主程序入口
├── requirements.txt          # 依赖包列表
├── rag_modules/              # RAG 核心模块
│   ├── __init__.py
│   ├── data_preparation.py   # 数据准备
│   ├── index_construction.py # 索引构建
│   ├── retrieval_optimization.py # 检索优化
│   └── generation_integration.py # 生成集成
├── vector_index/             # 向量索引存储（运行时生成）
└── data/                     # 食谱数据目录（需自行准备）
```

## 配置说明

在 `config.py` 中可以修改以下配置：

| 配置项 | 说明 | 默认值                      |
|--------|------|--------------------------|
| `data_path` | 食谱数据路径 | `./data/cook`            |
| `index_save_path` | 向量索引保存路径 | `./vector_index`         |
| `embedding_model` | 嵌入模型 | `BAAI/bge-small-zh-v1.5` |
| `llm_model` | 语言模型 | `kimi-k2-0711-preview`   |
| `top_k` | 检索返回数量 | `3`                      |
| `temperature` | 生成温度 | `0.1`                    |
| `max_tokens` | 最大生成长度 | `2048`                   |

## 注意事项

1. **API 密钥**: 需要 Moonshot AI API 密钥才能使用，请在 [Moonshot AI 官网](https://platform.moonshot.cn/) 申请
2. **首次运行**: 会构建向量索引，可能需要一些时间
3. **数据路径**: 确保在 `config.py` 中正确配置数据路径
4. **Python 版本**: 建议使用 Python 3.8+

## 常见问题

### Q: 如何获取 Moonshot API 密钥？
A: 访问 [Moonshot AI 开放平台](https://platform.moonshot.cn/) 注册并获取 API 密钥。

### Q: 向量索引文件太大怎么办？
A: 可以将 `vector_index/` 添加到 `.gitignore`，每次使用时重新构建。

### Q: 支持哪些数据格式？
A: 请参考 `rag_modules/data_preparation.py` 了解支持的数据格式。

## License

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！

# AI 知识库资源整理

> 整理时间：2026-05-18  
> 整理人：小希 💛

---

## 一、模型与数据集平台（技术向）

### 1. Hugging Face
- **网址**：https://huggingface.co
- **中文镜像**：https://hf-mirror.com
- **简介**：全球最大的AI开源社区，提供模型、数据集、工具库一站式获取
- **特点**：
  - 超过50万个预训练模型
  - 涵盖NLP、CV、音频、多模态等各领域
  - 支持 Transformers、Diffusers 等主流框架
  - 社区活跃，文档齐全

### 2. ModelScope 魔搭社区
- **网址**：https://modelscope.cn
- **简介**：阿里巴巴出品的AI模型开源社区
- **特点**：
  - 国内访问速度快
  - 中文模型资源丰富
  - 支持模型在线体验和API调用
  - 与阿里云生态深度集成

### 3. Kaggle
- **网址**：https://kaggle.com
- **简介**：Google旗下数据科学社区
- **特点**：
  - 海量公开数据集
  - 竞赛平台（实战练兵场）
  - Notebook 在线编程环境
  - 社区讨论和优质Notebook分享

---

## 二、系统学习资料

### 1. 飞书知识库（免费）
- **CY-CHENYUE的AI知识库**：https://u0ptmdsjdxb.feishu.cn/wiki/Kq5hwmobYiJR4akJwNecP9yhnse
- **AI学习资源合集**：含 DeepSeek学习手册、Sora学习手册、Claude学习手册、AIGC学习手册等
- **特点**：持续更新、免费开放、中文友好

### 2. 行业报告
- **未来智库**：https://www.vzkoo.com
  - 《2025年AI大模型资料汇编》等深度报告
- **远瞻慧库**：https://www.baogaobox.com
  - AI行业趋势分析报告

### 3. GitHub Awesome 系列（推荐搜索）
- `awesome-llm` — 大语言模型资源汇总
- `awesome-ai` — AI工具和资源清单
- `awesome-chatgpt` — ChatGPT 相关资源
- `awesome-prompt-engineering` — 提示工程资源
- **搜索方式**：在 GitHub 搜索栏直接输入关键词即可

---

## 三、中文AI学习专项

### 1. 通往AGI之路
- **网址**：https://waytoagi.com
- **简介**：中文AI学习路线图，从入门到进阶
- **特点**：路径清晰、资源精选、适合新手

### 2. DataWhale
- **网址**：https://datawhale.cn
- **简介**：开源学习社区
- **特点**：
  - 系统课程（大模型、NLP、机器学习等）
  - 组队学习模式，有同伴督促
  - 完全免费开源

### 3. 知识平台与博主
- **知乎**：关注 AI/大模型/LLM 相关话题
- **公众号推荐**：机器之心、量子位、新智元、AI科技评论
- **B站**：搜索 LLM教程、Transformer原理 等关键词，有大量优质视频

---

## 四、本地部署与实用工具

### 1. Ollama
- **网址**：https://ollama.com
- **简介**：本地运行大模型的一键工具
- **特点**：
  - 一行命令启动本地LLM
  - 支持 Llama、Qwen、Mistral 等主流模型
  - 跨平台（Mac/Linux/Windows）

### 2. OpenRouter
- **网址**：https://openrouter.ai
- **简介**：统一API入口
- **特点**：
  - 一个接口调用多种模型
  - 按量付费，适合测试对比
  - 支持 Claude、GPT、Gemini、Llama 等

### 3. LM Studio
- **网址**：https://lmstudio.ai
- **简介**：本地大模型GUI工具
- **特点**：图形界面、模型管理方便、适合非技术用户

### 4. AnythingLLM
- **网址**：https://anythingllm.com
- **简介**：私有知识库RAG解决方案
- **特点**：
  - 支持导入文档构建私有知识库
  - 支持多种LLM后端
  - 开源免费

---

## 五、向量数据库（构建知识库必备）

| 数据库 | 类型 | 特点 |
|--------|------|------|
| Milvus | 开源 | 高性能、分布式、社区活跃 |
| Chroma | 开源 | 轻量级、适合快速原型 |
| Pinecone | 云服务 | 免费额度、无需运维 |
| Weaviate | 开源 | 支持多模态、GraphQL接口 |
| Qdrant | 开源 | Rust编写、性能优秀 |
| FAISS | 开源 | Meta出品、适合大规模检索 |

---

## 六、推荐学习路线

```
入门阶段
├── 了解AI基础概念 → DataWhale课程 / 通往AGI之路
├── 学习Python基础 → B站教程
└── 体验AI工具 → ChatGPT / Claude / DeepSeek

进阶阶段
├── 学习Transformer原理 → 李宏毅课程 / Hugging Face教程
├── 动手实践 → Kaggle竞赛 / Hugging Face Spaces
└── 了解RAG/Agent → AnythingLLM / LangChain

深入阶段
├── 微调模型 → ModelScope / Hugging Face
├── 部署服务 → Ollama / vLLM
└── 构建应用 → 向量数据库 + LLM API
```

---

> 💡 **小贴士**：AI领域发展很快，建议定期关注上述平台的更新动态。收藏几个常用的，每周花1-2小时浏览即可保持信息同步。

---

*本文档由小希整理，如有新资源会持续更新~ 🌸*

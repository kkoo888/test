# 本地 RAG 知识库搭建指南

> 整理时间：2026-05-18  
> 整理人：小希 💛  
> 适用场景：配合 Ollama 本地模型，离线也能查资料

---

## 一、整体思路

```
你的文档/资料（PDF/Word/TXT/Markdown）
    ↓
文档加载 & 切片（按段落/句子拆分）
    ↓
向量化（用本地 Embedding 模型，不需要联网）
    ↓
存入向量数据库（Chroma / FAISS）
    ↓
用户提问 → 从数据库检索最相关的片段
    ↓
将检索结果 + 问题一起喂给 Ollama 本地模型
    ↓
模型基于检索到的内容生成回答
```

**核心原理**：不靠模型"记住"所有知识，而是先从你的资料里找到相关内容，再让模型基于这些内容回答。这样即使模型参数不大，也能回答专业问题。

---

## 二、环境准备

### 2.1 Python 环境

```bash
# 建议 Python 3.10+
python --version

# 创建虚拟环境（推荐）
python -m venv rag-env
source rag-env/bin/activate  # Linux/Mac
# rag-env\Scripts\activate   # Windows
```

### 2.2 安装依赖

```bash
pip install langchain langchain-community chromadb sentence-transformers
pip install unstructured pypdf docx2txt  # 文档解析支持
```

### 2.3 确认 Ollama 已就绪

```bash
# 检查 Ollama 是否运行
ollama list

# 如果没有模型，拉一个（以 Qwen 为例）
ollama pull qwen2:7b

# 测试一下
ollama run qwen2:7b "你好"
```

### 2.4 下载 Embedding 模型（首次需要联网，之后离线可用）

```python
# 首次运行时会自动下载模型到本地缓存
# 之后就完全离线了
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("BAAI/bge-small-zh-v1.5")
print("Embedding 模型加载成功！")
```

**推荐 Embedding 模型（中文场景）**：

| 模型 | 大小 | 推荐度 | 说明 |
|------|------|--------|------|
| `BAAI/bge-small-zh-v1.5` | ~90MB | ⭐⭐⭐ | 小巧高效，首选 |
| `BAAI/bge-base-zh-v1.5` | ~400MB | ⭐⭐⭐ | 效果更好，稍大 |
| `shibing624/text2vec-base-chinese` | ~400MB | ⭐⭐ | 经典选择 |

---

## 三、详细搭建流程

### 3.1 准备知识库文档

```
knowledge/
├── 技术文档/
│   ├── API文档.pdf
│   ├── 架构设计.md
│   └── 开发规范.txt
├── 产品资料/
│   ├── 产品说明.docx
│   └── 需求文档.pdf
└── 其他/
    └── 笔记.md
```

把你想让 AI 学习的文档都放到一个目录下，支持的格式：
- `.pdf` / `.docx` / `.txt` / `.md` / `.csv` / `.jsonl`

### 3.2 文档加载 & 切片

```python
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 加载目录下所有文档
loader = DirectoryLoader(
    "./knowledge/",
    glob="**/*.md",  # 可改为 *.* 加载所有格式
    loader_cls=TextLoader,
    loader_kwargs={"encoding": "utf-8"}
)
docs = loader.load()
print(f"加载了 {len(docs)} 个文档")

# 切片
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,       # 每个片段最多500字符
    chunk_overlap=50,     # 片段之间重叠50字符，保持上下文连贯
    separators=["\n\n", "\n", "。", "！", "？", ".", " "]  # 中文友好的分隔符
)
chunks = splitter.split_documents(docs)
print(f"切分为 {len(chunks)} 个片段")
```

**切片参数说明**：
- `chunk_size`：每个片段的字符数。太大会稀释关键信息，太小会丢失上下文。建议 300~800
- `chunk_overlap`：相邻片段的重叠区域。防止关键信息被切断。建议 chunk_size 的 10%~20%

### 3.3 向量化 & 存入数据库

```python
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma

# 本地 Embedding 模型（离线可用）
embeddings = HuggingFaceEmbeddings(
    model_name="BAAI/bge-small-zh-v1.5",
    model_kwargs={"device": "cpu"},  # 用 GPU 就改成 "cuda"
    encode_kwargs={"normalize_embeddings": True}
)

# 存入 Chroma 向量数据库
db = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"  # 数据持久化目录
)

print(f"✅ 已将 {len(chunks)} 个向量片段存入数据库")
```

### 3.4 检索 + Ollama 回答

```python
from langchain_community.llms import Ollama
from langchain.chains import RetrievalQA

# 连接本地 Ollama
llm = Ollama(
    model="qwen2:7b",           # 你本地的模型名
    base_url="http://localhost:11434"  # Ollama 默认地址
)

# 创建 RAG 检索问答链
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=db.as_retriever(
        search_type="similarity",  # 相似度检索
        search_kwargs={"k": 3}     # 返回最相关的3个片段
    ),
    return_source_documents=True   # 返回来源文档，方便溯源
)

# 提问
question = "项目的API接口规范是什么？"
result = qa_chain.invoke({"query": question})

print("回答：", result["result"])
print("\n参考来源：")
for doc in result["source_documents"]:
    print(f"  - {doc.metadata.get('source', '未知')}")
```

### 3.5 完整脚本（一键运行）

```python
#!/usr/bin/env python3
"""
本地 RAG 知识库 - 完整脚本
用法：python rag_qa.py "你的问题"
"""

import sys
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_community.llms import Ollama
from langchain.chains import RetrievalQA

# ========== 配置区 ==========
KNOWLEDGE_DIR = "./knowledge/"        # 知识库文档目录
CHROMA_DIR = "./chroma_db"            # 向量数据库存储目录
EMBEDDING_MODEL = "BAAI/bge-small-zh-v1.5"  # Embedding 模型
OLLAMA_MODEL = "qwen2:7b"             # Ollama 模型名
CHUNK_SIZE = 500                       # 切片大小
CHUNK_OVERLAP = 50                     # 切片重叠
TOP_K = 3                              # 检索返回的片段数
# ==============================

def build_database():
    """构建向量数据库（首次运行或文档更新时调用）"""
    print("📚 正在加载文档...")
    loader = DirectoryLoader(
        KNOWLEDGE_DIR,
        glob="**/*.*",
        loader_cls=TextLoader,
        loader_kwargs={"encoding": "utf-8"}
    )
    docs = loader.load()
    print(f"   加载了 {len(docs)} 个文档")

    print("✂️  正在切片...")
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=CHUNK_SIZE,
        chunk_overlap=CHUNK_OVERLAP,
        separators=["\n\n", "\n", "。", "！", "？", ".", " "]
    )
    chunks = splitter.split_documents(docs)
    print(f"   切分为 {len(chunks)} 个片段")

    print("🔢 正在向量化...")
    embeddings = HuggingFaceEmbeddings(
        model_name=EMBEDDING_MODEL,
        model_kwargs={"device": "cpu"},
        encode_kwargs={"normalize_embeddings": True}
    )

    print("💾 正在存入数据库...")
    db = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory=CHROMA_DIR
    )
    print(f"✅ 数据库构建完成！共 {len(chunks)} 个向量片段")
    return db

def load_database():
    """加载已有的向量数据库"""
    embeddings = HuggingFaceEmbeddings(
        model_name=EMBEDDING_MODEL,
        model_kwargs={"device": "cpu"},
        encode_kwargs={"normalize_embeddings": True}
    )
    return Chroma(persist_directory=CHROMA_DIR, embedding_function=embeddings)

def ask(db, question):
    """提问并获取回答"""
    llm = Ollama(model=OLLAMA_MODEL, base_url="http://localhost:11434")
    qa = RetrievalQA.from_chain_type(
        llm=llm,
        retriever=db.as_retriever(
            search_kwargs={"k": TOP_K}
        ),
        return_source_documents=True
    )
    result = qa.invoke({"query": question})
    return result

if __name__ == "__main__":
    import os

    if len(sys.argv) < 2:
        print("用法：")
        print("  python rag_qa.py --build    # 构建/更新知识库")
        print('  python rag_qa.py "你的问题"  # 提问')
        sys.exit(1)

    if sys.argv[1] == "--build":
        build_database()
    else:
        question = sys.argv[1]
        if not os.path.exists(CHROMA_DIR):
            print("⚠️  数据库不存在，请先运行: python rag_qa.py --build")
            sys.exit(1)

        print(f"🔍 正在检索相关内容...")
        db = load_database()
        result = ask(db, question)

        print(f"\n💡 回答：\n{result['result']}")
        print(f"\n📄 参考来源：")
        for doc in result["source_documents"]:
            source = doc.metadata.get("source", "未知")
            print(f"  - {source}")
```

---

## 四、使用流程总结

```bash
# 1. 首次：构建知识库（把文档向量化存入数据库）
python rag_qa.py --build

# 2. 日常：提问
python rag_qa.py "项目的部署流程是什么？"

# 3. 文档更新后：重新构建
python rag_qa.py --build
```

---

## 五、注意事项

### 5.1 离线使用前提
- Embedding 模型需要**首次联网下载**，之后会缓存在本地
- Ollama 模型需要**首次联网拉取**，之后离线可用
- 之后整个 RAG 流程完全离线运行

### 5.2 性能优化
- **有 GPU**：把 `device="cpu"` 改成 `device="cuda"`，速度快很多
- **文档量大**：考虑用 FAISS 替代 Chroma，检索性能更好
- **回答质量**：调整 `chunk_size` 和 `TOP_K`，多试几次找到最佳参数

### 5.3 常见问题
| 问题 | 解决方案 |
|------|---------|
| 回答不准确 | 增大 TOP_K，或调小 chunk_size 让片段更精细 |
| 回答太啰嗦 | 在 prompt 里加"请简洁回答" |
| 检索不到相关内容 | 检查文档是否正确加载，chunk_size 是否合适 |
| Ollama 连接失败 | 确认 Ollama 服务在运行：`ollama serve` |

---

## 六、进阶方向

- **多文档类型支持**：加入 PDF、Word、Excel 解析器
- **对话记忆**：加入聊天历史，支持多轮对话
- **Web 搜索混合**：本地知识 + 联网搜索结合
- **知识库更新增量**：只向量化新增文档，不重复处理
- **权限管理**：不同用户访问不同知识库

---

> 💡 有问题随时问小希，帮您调试到能用为止~ 🌸

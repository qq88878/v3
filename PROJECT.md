# 项目开发手册

> **项目名称**：基于大模型的个性化资源生成与学习多智能体系统
> **版本**：v1.0
> **更新日期**：2026年6月
> **截止时间**：2026年7月初（1个月）

---

## 一、项目概述

### 1.1 核心目标

构建一个**计算机/编程领域**的个性化学习平台，通过多智能体协作实现：
- 学生画像构建与动态更新
- 个性化学习路径规划
- 教学资源智能生成（题库、思维导图、文档/PPT）
- 智能辅导与学习效果评估

### 1.2 比赛硬性要求

| 要求 | 说明 |
|------|------|
| 必须使用星火大模型 | 核心推理引擎，不能只用其他模型 |
| 必须体现多智能体 | 清晰的Agent分工与协作流程 |
| 资源必须可交互 | 生成的内容支持下载/预览 |
| 一键启动 | docker-compose up 即可运行 |

### 1.3 我们的策略

**双模型策略**：
- **星火大模型**：核心功能（画像分析、路径规划、资源生成）
- **小米MiMo**：辅助功能（对话补全、简单问答、测试阶段）

---

## 二、技术架构

### 2.1 技术栈总览

```
┌─────────────────────────────────────────────────────┐
│                    前端层 (Vue 3)                    │
│   Vue 3 + TypeScript + Element Plus + Vite          │
└─────────────────────────────────────────────────────┘
                          │ HTTP/SSE
┌─────────────────────────────────────────────────────┐
│                   后端层 (FastAPI)                   │
│   Python 3.11 + FastAPI + Uvicorn                   │
└─────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────┐
│                 Agent层 (LangGraph)                  │
│   StateGraph 多智能体调度                           │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐              │
│   │Planner  │ │Profile  │ │Researcher│              │
│   │Agent    │ │Agent    │ │Agent    │              │
│   └─────────┘ └─────────┘ └─────────┘              │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐              │
│   │Writer   │ │Reviewer │ │Evaluator│              │
│   │Agent    │ │Agent    │ │Agent    │              │
│   └─────────┘ └─────────┘ └─────────┘              │
└─────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────┐
│                  数据层                              │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│   │  Milvus  │ │  MySQL   │ │  Redis   │           │
│   │(向量库)   │ │(结构化)   │ │(缓存)    │           │
│   └──────────┘ └──────────┘ └──────────┘           │
└─────────────────────────────────────────────────────┘
```

### 2.2 技术选型详情

| 层级 | 技术 | 版本 | 用途 |
|------|------|------|------|
| 前端 | Vue 3 + TypeScript | 3.4+ | 学习平台界面 |
| UI库 | Element Plus | 2.x | 组件库 |
| 构建 | Vite | 5.x | 快速开发构建 |
| 后端 | Python + FastAPI | 3.11+ | API服务、流式输出 |
| Agent | LangGraph | 最新 | 多智能体状态机 |
| LLM | 讯飞星火 + 小米MiMo | - | 核心推理 + 辅助 |
| RAG | LangChain | 最新 | 文档加载、切分、检索 |
| 向量库 | Milvus | 2.x | 课程知识库向量存储 |
| 数据库 | MySQL | 8.x | 学生画像、学习记录 |
| 缓存 | Redis | 7.x | 会话缓存、状态管理 |
| 部署 | Docker Compose | - | 一键启动 |

### 2.3 多智能体角色设计

```
用户请求
    │
    ▼
┌─────────────┐
│  Planner    │ ← 学习路径规划，根据画像推荐学习顺序
│  Agent      │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  Profile    │ ←──→│  Researcher │
│  Agent      │     │  Agent      │
│  (画像更新)  │     │  (RAG检索)   │
└──────┬──────┘     └──────┬──────┘
       │                   │
       ▼                   ▼
┌─────────────┐
│  Writer     │ ← 生成题库/思维导图/文档
│  Agent      │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  Reviewer   │ ←──→│  Evaluator  │
│  Agent      │     │  Agent      │
│  (内容审核)  │     │  (效果评估)  │
└─────────────┘     └─────────────┘
```

**各Agent职责：**

| Agent | 职责 | 输入 | 输出 |
|-------|------|------|------|
| Planner | 学习路径规划 | 学生画像、课程大纲 | 推荐学习顺序 |
| Profile | 学生画像管理 | 对话历史、测试结果 | 6维度标签更新 |
| Researcher | RAG知识检索 | 用户问题、知识点 | 相关文档片段 |
| Writer | 资源生成 | 知识点、生成要求 | 题目/思维导图/文档 |
| Reviewer | 内容审核 | 生成内容 | 审核通过/修正建议 |
| Evaluator | 效果评估 | 答题记录、学习时长 | 掌握度评分 |

---

## 三、开发计划（1个月）

### 3.1 里程碑时间表

```
Week 1 (6月2日-6月8日)
├── 环境搭建 + 讯飞API接入
├── 基础项目结构创建
└── 目标：星火API能调通，返回正常结果

Week 2 (6月9日-6月15日)
├── RAG知识库构建（Milvus + 文档切分）
├── LangGraph基础Agent框架搭建
└── 目标：能基于文档回答问题

Week 3 (6月16日-6月22日)
├── 多智能体协作流程开发
├── 资源生成模块（题库、思维导图、PPT）
└── 目标：完整流程跑通，能生成资源

Week 4 (6月23日-6月30日)
├── 前端界面开发 + API对接
├── Docker容器化 + 测试优化
└── 目标：完整Demo可演示
```

### 3.2 每周交付物

| 周次 | 交付物 | 验证方式 |
|------|--------|----------|
| Week 1 | API调用示例 + 项目骨架 | 运行脚本能收到星火回复 |
| Week 2 | RAG问答Demo | 上传文档后能回答相关问题 |
| Week 3 | 多Agent协作Demo | 输入需求能生成题库/思维导图 |
| Week 4 | 完整系统 | docker-compose up 后可演示所有功能 |

---

## 四、项目目录结构

```
v3/
├── frontend/                    # Vue3 前端
│   ├── src/
│   │   ├── views/              # 页面组件
│   │   │   ├── Home.vue        # 首页
│   │   │   ├── Chat.vue        # 智能对话
│   │   │   ├── Resources.vue   # 资源中心
│   │   │   ├── Profile.vue     # 学生画像
│   │   │   └── Path.vue        # 学习路径
│   │   ├── components/         # 通用组件
│   │   │   ├── ChatBox.vue     # 对话框
│   │   │   ├── MindMap.vue     # 思维导图展示
│   │   │   └── ResourceCard.vue # 资源卡片
│   │   ├── api/                # API调用封装
│   │   ├── stores/             # Pinia状态管理
│   │   └── router/             # 路由配置
│   ├── package.json
│   └── vite.config.ts
│
├── backend/                     # FastAPI 后端
│   ├── main.py                 # 入口文件
│   ├── api/                    # 路由接口
│   │   ├── chat.py             # 对话接口
│   │   ├── resources.py        # 资源接口
│   │   ├── profile.py          # 画像接口
│   │   └── path.py             # 路径接口
│   ├── agents/                 # LangGraph Agents
│   │   ├── planner.py          # 规划Agent
│   │   ├── profile.py          # 画像Agent
│   │   ├── researcher.py       # 检索Agent
│   │   ├── writer.py           # 生成Agent
│   │   ├── reviewer.py         # 审核Agent
│   │   ├── evaluator.py        # 评估Agent
│   │   └── graph.py            # LangGraph状态图
│   ├── rag/                    # RAG模块
│   │   ├── loader.py           # 文档加载
│   │   ├── splitter.py         # 文本切分
│   │   ├── embedder.py         # 向量化
│   │   └── retriever.py        # 检索器
│   ├── services/               # 业务服务
│   │   ├── llm_service.py      # LLM调用封装
│   │   ├── resource_service.py # 资源生成服务
│   │   └── profile_service.py  # 画像服务
│   ├── models/                 # 数据模型
│   │   ├── student.py          # 学生模型
│   │   ├── resource.py         # 资源模型
│   │   └── conversation.py     # 对话模型
│   └── utils/                  # 工具函数
│       ├── config.py           # 配置管理
│       └── helpers.py          # 辅助函数
│
├── knowledge_base/              # 课程知识库
│   ├── python/                 # Python课程资料
│   ├── algorithms/             # 算法课程资料
│   └── data_structures/        # 数据结构资料
│
├── docker/                      # Docker配置
│   ├── Dockerfile.frontend
│   ├── Dockerfile.backend
│   └── nginx.conf
│
├── docker-compose.yml           # 一键启动
├── requirements.txt             # Python依赖
├── .env.example                 # 环境变量示例
└── README.md                    # 项目说明
```

---

## 五、核心模块实现要点

### 5.1 讯飞星火API接入

```python
# services/llm_service.py 核心结构
class LLMService:
    def __init__(self):
        self.spark_api = SparkAPI(api_key=os.getenv("SPARK_API_KEY"))
        self.mimo_api = MiMoAPI(api_key=os.getenv("MIMO_API_KEY"))

    async def chat(self, messages, model="spark"):
        """双模型调用，spark为主，mimo为辅"""
        if model == "spark":
            return await self.spark_api.chat(messages)
        else:
            return await self.mimo_api.chat(messages)
```

### 5.2 LangGraph状态图设计

```python
# agents/graph.py 核心结构
from langgraph.graph import StateGraph, END

class LearningState(TypedDict):
    student_profile: dict        # 学生画像
    current_topic: str           # 当前知识点
    conversation_history: list   # 对话历史
    retrieved_docs: list         # 检索到的文档
    generated_content: dict      # 生成的内容
    review_result: dict          # 审核结果

def create_learning_graph():
    graph = StateGraph(LearningState)

    # 添加节点
    graph.add_node("profile", profile_agent)
    graph.add_node("planner", planner_agent)
    graph.add_node("researcher", researcher_agent)
    graph.add_node("writer", writer_agent)
    graph.add_node("reviewer", reviewer_agent)
    graph.add_node("evaluator", evaluator_agent)

    # 定义边
    graph.set_entry_point("profile")
    graph.add_edge("profile", "planner")
    graph.add_edge("planner", "researcher")
    graph.add_edge("researcher", "writer")
    graph.add_edge("writer", "reviewer")
    graph.add_conditional_edges(
        "reviewer",
        lambda state: "pass" if state["review_result"]["passed"] else "fail",
        {"pass": "evaluator", "fail": "writer"}
    )
    graph.add_edge("evaluator", END)

    return graph.compile()
```

### 5.3 RAG知识库构建

```python
# rag/retriever.py 核心结构
from langchain.document_loaders import DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Milvus

class KnowledgeBase:
    def __init__(self):
        self.embeddings = HuggingFaceEmbeddings(
            model_name="sentence-transformers/all-MiniLM-L6-v2"
        )
        self.vectorstore = Milvus(
            embedding_function=self.embeddings,
            connection_args={"host": "localhost", "port": 19530}
        )

    def load_documents(self, path: str):
        """加载课程文档"""
        loader = DirectoryLoader(path, glob="**/*.pdf")
        docs = loader.load()
        splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        return splitter.split_documents(docs)

    def search(self, query: str, k: int = 5):
        """检索相关文档"""
        return self.vectorstore.similarity_search(query, k=k)
```

### 5.4 资源生成模块

```python
# services/resource_service.py 核心结构
class ResourceService:
    async def generate_questions(self, topic: str, count: int = 5):
        """生成题库"""
        prompt = f"""
        请为"{topic}"生成{count}道练习题，包含：
        1. 题目描述
        2. 选项（如果是选择题）
        3. 正确答案
        4. 详细解析
        输出JSON格式。
        """
        return await self.llm_service.chat(prompt, model="spark")

    async def generate_mindmap(self, topic: str):
        """生成思维导图（Mermaid语法）"""
        prompt = f"""
        请为"{topic}"生成思维导图，使用Mermaid语法。
        要求：层次清晰，包含核心概念和关键知识点。
        输出格式：直接输出Mermaid代码。
        """
        return await self.llm_service.chat(prompt, model="spark")

    async def generate_ppt(self, topic: str, slides: int = 10):
        """生成PPT"""
        # 1. LLM生成内容大纲
        outline = await self._generate_outline(topic, slides)
        # 2. 使用python-pptx生成PPT文件
        pptx_file = self._create_pptx(outline)
        return pptx_file
```

### 5.5 流式输出（SSE）

```python
# api/chat.py
from fastapi.responses import StreamingResponse

@router.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    async def generate():
        async for chunk in agent_graph.astream(request.message):
            yield f"data: {json.dumps(chunk)}\n\n"

    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )
```

---

## 六、前端页面设计

### 6.1 核心页面

| 页面 | 功能 | 关键组件 |
|------|------|----------|
| 首页 | 系统介绍、快速入口 | 统计卡片、快捷按钮 |
| 智能对话 | 与Agent交互 | ChatBox、Markdown渲染 |
| 资源中心 | 查看生成的资源 | 资源列表、预览、下载 |
| 学生画像 | 查看学习状态 | 雷达图、标签云 |
| 学习路径 | 可视化学习路径 | 时间线、进度条 |

### 6.2 关键组件示例

```vue
<!-- components/ChatBox.vue 核心结构 -->
<template>
  <div class="chat-container">
    <div class="messages">
      <div v-for="msg in messages" :key="msg.id"
           :class="['message', msg.role]">
        <MarkdownRenderer :content="msg.content" />
      </div>
    </div>
    <div class="input-area">
      <el-input v-model="input" @keyup.enter="send" />
      <el-button @click="send">发送</el-button>
    </div>
  </div>
</template>
```

---

## 七、Docker部署方案

### 7.1 docker-compose.yml

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: ../docker/Dockerfile.frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: ../docker/Dockerfile.backend
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - mysql
      - milvus
      - redis

  mysql:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: learning_agent
    volumes:
      - mysql_data:/var/lib/mysql

  milvus:
    image: milvusdb/milvus:latest
    ports:
      - "19530:19530"
    volumes:
      - milvus_data:/var/lib/milvus

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  mysql_data:
  milvus_data:
```

### 7.2 启动命令

```bash
# 一键启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

---

## 八、环境变量配置

```bash
# .env.example

# 讯飞星火API
SPARK_API_KEY=your_spark_api_key
SPARK_API_SECRET=your_spark_api_secret

# 小米MiMo API
MIMO_API_KEY=your_mimo_api_key

# MySQL
MYSQL_ROOT_PASSWORD=your_mysql_password
MYSQL_HOST=mysql
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=your_mysql_password
MYSQL_DATABASE=learning_agent

# Milvus
MILVUS_HOST=milvus
MILVUS_PORT=19530

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
```

---

## 九、风险与应对

| 风险 | 影响 | 应对策略 |
|------|------|----------|
| 星火API申请延迟 | 无法开发核心功能 | 先用MiMo跑通流程，API下来后切换 |
| LangGraph学习曲线陡 | 进度延迟 | 先用简单Chain，后续再升级 |
| 1个月时间不够 | 功能不完整 | 砍掉非核心功能，保证Demo能跑 |
| 前端开发耗时 | 界面不完善 | 使用Element Plus模板，少写自定义组件 |
| Docker部署问题 | 无法一键启动 | 提前测试，准备无Docker备选方案 |

---

## 十、团队分工建议

**你（主导）：**
- 整体架构设计
- Agent层开发（LangGraph）
- 后端API开发
- 技术难点攻克

**队友A：**
- 前端界面开发
- API对接
- 用户体验优化

**队友B（如果有）：**
- RAG知识库构建
- 文档处理与切分
- 测试与部署

---

## 十一、验收检查清单

### 必须完成（硬性要求）

- [ ] 星火大模型API调用正常
- [ ] 至少3个Agent协作完成任务
- [ ] 生成的资源可下载/预览
- [ ] docker-compose up 一键启动
- [ ] README文档完整

### 优先完成（加分项）

- [ ] RAG知识库检索准确
- [ ] 题库自动生成（含解析）
- [ ] 思维导图生成（Mermaid）
- [ ] PPT/文档生成
- [ ] 流式输出（打字机效果）
- [ ] 学生画像可视化
- [ ] 学习路径规划

### 可选完成（时间允许）

- [ ] 视频/动画生成
- [ ] 语音交互
- [ ] 学习效果评估报告

---

## 十二、快速开始

### 12.1 环境准备

```bash
# 克隆项目
git clone <repo_url>
cd v3

# 创建Python虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt

# 配置环境变量
cp .env.example .env
# 编辑 .env 填入API密钥
```

### 12.2 本地开发

```bash
# 启动后端
cd backend
uvicorn main:app --reload --port 8000

# 启动前端（新终端）
cd frontend
npm install
npm run dev
```

### 12.3 Docker部署

```bash
# 一键启动所有服务
docker-compose up -d

# 访问 http://localhost:3000
```

---

## 附录：常用命令

```bash
# 查看Agent日志
tail -f logs/agent.log

# 运行测试
pytest tests/

# 代码格式化
black backend/
isort backend/

# 前端构建
cd frontend && npm run build
```

---

**文档维护者**：项目主导者
**最后更新**：2026年6月

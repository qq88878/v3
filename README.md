# v3
📄 项目开发参考文档
项目名称：A3-基于大模型的个性化资源生成与学习多智能体系统开发 文档类型：技术栈与开发指南（后续开发参考手册） 版本：v1.0 更新日期：2026年5月

一、项目概述
本项目是一个典型的大模型应用 + 多智能体系统（MAS）+ RAG + 全栈开发的综合赛题。核心目标是构建一个支持个性化学习路径规划、教学资源智能生成（文档、PPT、视频、代码、题库等）、智能辅导与学习评估的多智能体教育系统。
核心要求：
●必须使用科大讯飞星火大模型作为核心推理引擎
●必须体现多智能体协作架构
●资源生成结果需可下载、可预览
●结合RAG减少幻觉

二、技术栈分层推荐
1. 核心必选技术（必须掌握）
类别	推荐技术	用途说明	优先级
Agent 框架	LangGraph（首选） / AutoGen / CrewAI	构建有状态多智能体工作流	★★★★★
大模型 API	科大讯飞星火认知大模型	核心推理（画像、规划、生成）	★★★★★
RAG 框架	LangChain Retrieval / LlamaIndex	课程知识库构建与检索	★★★★★
向量数据库	ChromaDB（原型） / Milvus	存储课程文档向量	★★★★★
前端	Vue 3 + TypeScript + Element Plus 或 React + Ant Design	学习平台界面	★★★★★
后端	Python + FastAPI	API 服务、流式输出	★★★★★
2. 多模态资源生成技术
资源类型	推荐技术工具	备注
文档/PPT	Markdown + python-docx + python-pptx	支持导出Word/PPT
思维导图	Mermaid 语法	可渲染为图片
图像/示意图	讯飞图文生成 / Stable Diffusion	概念图、插图
视频/动画	讯飞 SeeDance / 讯飞视频生成API	重大加分项
代码生成	星火模型 + 代码高亮（Prism.js）	支持在线预览
题库生成	Few-shot Prompting	含解析
3. 特色功能实现方案（加分项）
●防幻觉机制：RAG + 独立 Reviewer Agent（审核智能体）
●流式输出：FastAPI StreamingResponse + SSE（前端打字机效果）
●动态学生画像：向量数据库存储对话历史 + 测试结果，6维度标签更新
●学习路径规划：知识点构建 DAG + 规划 Agent
●记忆模块：LangGraph State + Redis 缓存

三、推荐技术组合方案（最优实践）
推荐技术栈（竞争力最强）：
1.前端：Vue 3 + TypeScript + Element Plus + Vite
2.后端：Python 3.11 + FastAPI + Uvicorn
3.Agent 架构：LangChain + LangGraph（StateGraph）
4.大模型：科大讯飞星火大模型 API（主推理）+ 讯飞语音/图像/视频 API
5.知识库：ChromaDB + LangChain Document Loader + Text Splitter
6.数据库：PostgreSQL（结构化数据）+ Redis（缓存 + 会话）
7.部署：Docker + Docker Compose + Nginx
多智能体角色设计建议（示例）：
●Planner Agent：学习路径规划
●Profile Agent：学生画像构建与更新
●Researcher Agent：RAG检索 + 知识补充
●Writer Agent：资源生成（PPT、文档、视频脚本）
●Reviewer Agent：内容审核与纠错
●Evaluator Agent：学习效果评估

四、开发流程建议
1.阶段一：环境搭建 + 星火API接入测试
2.阶段二：RAG知识库构建（课程PDF/Word/PPT切片）
3.阶段三：LangGraph 多智能体工作流开发
4.阶段四：前端界面 + 后端API对接
5.阶段五：资源生成模块（文档/PPT/视频）
6.阶段六：个性化画像与路径规划
7.阶段七：Docker 容器化 + 测试优化

五、避坑指南
1.必须体现多智能体：不要用单个 LLM 直接完成所有功能，要有清晰的 Agent 分工与协作流程。
2.必须使用讯飞：所有核心 AI 功能均需调用星火模型，其他辅助工具可使用讯飞系列。
3.资源必须可交互：生成的 PPT、视频、代码需支持下载或在线预览。
4.文档规范：开源项目使用需标注 License；AI 辅助编码需记录使用说明。
5.评委体验优先：提供一键启动的 docker-compose up 脚本。

六、项目目录结构参考
text
project/
├── frontend/                # Vue3 前端
├── backend/                 # FastAPI 后端│   
├── agents/              # LangGraph Agents│   
├── rag/                 # RAG 模块│   
├── services/            # 业务服务│   
├── api/                 # 接口│   
└── utils/├── knowledge_base/          # 课程资料
├── docker/                  # Docker 配置
├── docker-compose.yml├── requirements.txt└── README.md              # 详细部署与使用说明

后续开发建议：
●优先把 LangGraph 状态机 和 讯飞 API 接入 打通，这是项目核心。
●每周固定做一次完整 Demo，验证多智能体协作是否顺畅。
●重点打磨 视频生成 和 个性化路径规划 两个亮点功能

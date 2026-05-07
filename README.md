# Product Manager Station (PM Station)

产品经理个人工作站 - 基于多Agent协作与长链推理的智能产品开发平台

## 项目简介

Product Manager Station 是一个专为产品经理设计的智能工作站，采用多Agent协作架构和长链推理技术，帮助产品经理高效完成从需求分析到产品交付的全流程工作。

### 核心特性

- **多Agent协作架构**：需求解析Agent、原型绘制Agent、文档编撰Agent、校验纠错Agent协同工作
- **长链推理引擎**：支持复杂任务的深度推理和多步骤规划
- **实时协作**：基于WebSocket的实时通信，支持流式响应
- **可视化工作流**：直观的工作流编排和状态监控
- **模块化设计**：易于扩展和维护的架构设计

## 技术架构

### 后端技术栈

- **框架**：FastAPI (Python 3.14+)
- **数据库**：SQLite + aiosqlite (异步支持)
- **LLM集成**：支持多提供商 (OpenAI、Claude、DashScope等)
- **任务队列**：Celery + Redis
- **实时通信**：WebSocket

### 前端技术栈

- **框架**：Vue 3 + TypeScript
- **构建工具**：Vite
- **UI组件**：Element Plus
- **状态管理**：Pinia
- **路由**：Vue Router

### 核心模块

```
backend/
├── app/
│   ├── agents/          # Agent实现
│   │   ├── requirement_agent.py
│   │   ├── prototype_agent.py
│   │   ├── document_agent.py
│   │   └── validation_agent.py
│   ├── api/             # API路由
│   │   └── v1/
│   │       ├── endpoints/
│   │       │   ├── health.py
│   │       │   ├── tasks.py
│   │       │   ├── requirement.py
│   │       │   └── workflow.py
│   │       └── websocket.py
│   ├── chains/          # 长链推理链
│   │   ├── base.py
│   │   ├── requirement_chain.py
│   │   ├── prototype_chain.py
│   │   ├── document_chain.py
│   │   └── validation_chain.py
│   ├── core/            # 核心组件
│   │   ├── config.py    # 配置管理
│   │   ├── llm.py       # LLM工厂
│   │   ├── llm_providers/  # LLM提供商实现
│   │   └── workflow.py  # 工作流编排器
│   ├── models/          # 数据模型
│   ├── services/        # 业务服务
│   ├── templates/       # 文档模板
│   └── main.py          # 应用入口
├── tests/               # 测试用例
└── requirements.txt     # 依赖管理

frontend/
├── src/
│   ├── components/      # 组件
│   ├── views/           # 页面
│   ├── stores/          # 状态管理
│   ├── api/             # API客户端
│   ├── utils/           # 工具函数
│   └── App.vue          # 根组件
├── public/              # 静态资源
└── package.json         # 依赖管理
```

## 快速开始

### 环境要求

- Python 3.14+
- Node.js 18+
- Redis (可选，用于Celery任务队列)

### 安装步骤

#### 1. 克隆项目

```bash
git clone https://github.com/colin1suntao/prodouct-manager-station.git
cd prodouct-manager-station
```

#### 2. 后端配置

```bash
cd backend

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或 venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt

# 配置环境变量 (可选)
cp .env.example .env
# 编辑 .env 文件配置LLM提供商和数据库
```

#### 3. 前端配置

```bash
cd ../frontend

# 安装依赖
npm install

# 或 yarn install
# 或 pnpm install
```

### 启动服务

#### 方式一：手动启动

**启动后端服务：**

```bash
cd backend
source venv/bin/activate

# 启动开发服务器
python run.py

# 或使用 uvicorn
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

后端服务将在 http://localhost:8000 启动

**启动前端服务：**

```bash
cd frontend

# 启动开发服务器
npm run dev

# 或 yarn dev
# 或 pnpm dev
```

前端服务将在 http://localhost:3000 启动

#### 方式二：Docker启动

```bash
# 构建并启动所有服务
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

### 访问应用

- **前端界面**：http://localhost:3000
- **API文档**：http://localhost:8000/docs (Swagger UI)
- **API文档**：http://localhost:8000/redoc (ReDoc)

## 使用指南

### 1. 需求解析

在需求解析页面输入产品需求描述，系统将自动：
- 提取关键需求点
- 分析用户画像
- 生成功能列表
- 输出PRD初稿

### 2. 原型生成

基于解析后的需求，系统可以：
- 生成页面结构
- 创建交互流程
- 输出原型文档

### 3. 文档编撰

支持自动生成多种产品文档：
- PRD (产品需求文档)
- 用户故事
- API文档
- 测试用例

### 4. 校验纠错

系统会自动检查：
- 需求完整性
- 逻辑一致性
- 文档规范性

## 配置说明

### 环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `DATABASE_URL` | 数据库连接URL | `sqlite+aiosqlite:///./pmstation.db` |
| `MODEL_PROVIDER` | LLM提供商 | `mock` |
| `OPENAI_API_KEY` | OpenAI API密钥 | - |
| `CLAUDE_API_KEY` | Claude API密钥 | - |
| `DASHSCOPE_API_KEY` | 阿里云DashScope密钥 | - |
| `REDIS_URL` | Redis连接URL | `redis://localhost:6379/0` |
| `CELERY_BROKER_URL` | Celery消息队列 | `redis://localhost:6379/0` |

### LLM提供商配置

项目支持多种LLM提供商，在 `.env` 文件中配置：

```bash
# 使用 OpenAI
MODEL_PROVIDER=openai
OPENAI_API_KEY=your_openai_key

# 使用 Claude
MODEL_PROVIDER=claude
CLAUDE_API_KEY=your_claude_key

# 使用阿里云DashScope
MODEL_PROVIDER=dashscope
DASHSCOPE_API_KEY=your_dashscope_key

# 使用 Mock 模式 (开发测试)
MODEL_PROVIDER=mock
```

## 开发指南

### 后端开发

```bash
cd backend

# 运行测试
pytest

# 代码格式化
black app/
ruff check app/

# 类型检查
mypy app/
```

### 前端开发

```bash
cd frontend

# 运行测试
npm run test

# 代码检查
npm run lint

# 类型检查
npm run type-check

# 构建生产版本
npm run build
```

## API接口

### REST API

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/v1/health` | GET | 健康检查 |
| `/api/v1/tasks` | GET | 获取任务列表 |
| `/api/v1/tasks` | POST | 创建任务 |
| `/api/v1/tasks/{id}` | GET | 获取任务详情 |
| `/api/v1/requirement/analyze` | POST | 需求分析 |
| `/api/v1/workflow/execute` | POST | 执行工作流 |

### WebSocket

| 端点 | 说明 |
|------|------|
| `/ws` | 实时通信连接 |

## 项目状态

- [x] Phase 1: 核心框架搭建
- [x] Phase 2: 需求解析Agent
- [x] Phase 3: 原型生成Agent
- [x] Phase 4: 文档生成Agent
- [x] Phase 5: 校验Agent
- [x] Phase 6: 集成与优化
- [ ] Phase 7: 高级功能 (进行中)

## 贡献指南

欢迎提交Issue和Pull Request！

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建 Pull Request

## 许可证

[MIT](LICENSE)

## 联系方式

如有问题或建议，欢迎通过以下方式联系：

- 提交 [GitHub Issue](https://github.com/colin1suntao/prodouct-manager-station/issues)
- 发送邮件至项目维护者

---

**Product Manager Station** - 让产品经理的工作更智能、更高效！

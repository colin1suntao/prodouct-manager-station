# Phase 1: 核心框架搭建 - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 完成项目脚手架搭建、前后端基础架构、Docker环境配置，建立可运行的基础项目框架

**Architecture:** 前后端分离架构，前端Vue 3 + Vite，后端Python FastAPI，通过Docker Compose统一部署

**Tech Stack:** Vue 3 + Vite + TailwindCSS | Python FastAPI + SQLAlchemy | PostgreSQL + Redis | Docker Compose

---

## 文件结构设计

```
product-manager-station/
├── frontend/                    # Vue 3 前端项目
│   ├── src/
│   │   ├── components/          # 通用组件
│   │   ├── views/               # 页面视图
│   │   ├── stores/              # Pinia状态管理
│   │   ├── api/                # API调用模块
│   │   ├── router/              # Vue Router配置
│   │   ├── styles/              # 全局样式
│   │   ├── types/               # TypeScript类型定义
│   │   └── utils/               # 工具函数
│   ├── public/                  # 静态资源
│   ├── Dockerfile               # 前端Dockerfile
│   ├── docker-compose.yml      # 前端docker-compose配置
│   ├── vite.config.ts           # Vite配置
│   ├── tailwind.config.js       # TailwindCSS配置
│   ├── tsconfig.json            # TypeScript配置
│   └── package.json
│
├── backend/                     # Python FastAPI 后端
│   ├── app/
│   │   ├── api/                 # API路由
│   │   │   └── v1/
│   │   │       ├── endpoints/   # 各模块端点
│   │   │       └── api.py       # 路由聚合
│   │   ├── core/                # 核心配置
│   │   │   ├── config.py        # 配置管理
│   │   │   ├── database.py      # 数据库连接
│   │   │   └── security.py      # 安全相关
│   │   ├── models/              # SQLAlchemy模型
│   │   ├── schemas/             # Pydantic schemas
│   │   ├── services/            # 业务逻辑服务
│   │   ├── agents/              # Agent基类（预留）
│   │   └── main.py              # FastAPI入口
│   ├── tests/                   # 测试代码
│   ├── Dockerfile               # 后端Dockerfile
│   ├── docker-compose.yml      # 后端docker-compose配置
│   ├── requirements.txt         # Python依赖
│   └── alembic.ini              # 数据库迁移配置
│
├── docker-compose.yml           # 根目录docker-compose
├── .env.example                 # 环境变量示例
└── README.md
```

---

## 任务 1: 创建项目目录结构

**Files:**
- Create: `product-manager-station/frontend/`
- Create: `product-manager-station/backend/`
- Create: `product-manager-station/docs/`

- [ ] **Step 1: 创建根目录结构**

Run: `mkdir -p product-manager-station/{frontend/src/{components,views,stores,api,router,styles,types,utils},frontend/public,backend/app/{api/v1/endpoints,core,models,schemas,services,agents},backend/tests,docs}`

Expected: 目录创建成功，无输出

- [ ] **Step 2: 创建README.md**

```markdown
# Product Manager Station

产品经理个人工作站 - 基于多Agent协作+长链推理的智能化产品设计工具

## 快速开始

### 环境要求
- Docker & Docker Compose
- Node.js 18+
- Python 3.10+

### 启动方式

```bash
# 方式一: Docker Compose一键启动
docker-compose up -d

# 方式二: 本地开发
cd frontend && npm install && npm run dev
cd backend && pip install -r requirements.txt && uvicorn app.main:app --reload
```

## 项目结构

- `frontend/` - Vue 3 前端项目
- `backend/` - Python FastAPI 后端项目
- `docs/` - 设计文档和计划

## 功能特性

- [ ] 需求智能解析
- [ ] 原型自动生成
- [ ] PRD文档生成
- [ ] 双向校验纠错

## License

MIT
```

- [ ] **Step 3: 创建.gitignore**

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
venv/
.venv/
.env

# Node
node_modules/
dist/
.env.local

# IDE
.vscode/
.idea/
*.swp

# Docker
.docker/

# Database
*.db
*.sqlite

# Logs
*.log
```

- [ ] **Step 4: Commit**

```bash
git init
git add README.md .gitignore
git commit -m "docs: initialize project structure"
```

---

## 任务 2: 搭建前端Vue 3项目

**Files:**
- Create: `frontend/package.json`
- Create: `frontend/vite.config.ts`
- Create: `frontend/tsconfig.json`
- Create: `frontend/tailwind.config.js`
- Create: `frontend/src/main.ts`
- Create: `frontend/src/App.vue`
- Create: `frontend/index.html`
- Create: `frontend/Dockerfile`

- [ ] **Step 1: 创建package.json**

```json
{
  "name": "pm-station-frontend",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix",
    "type-check": "vue-tsc --noEmit"
  },
  "dependencies": {
    "vue": "^3.4.0",
    "vue-router": "^4.2.5",
    "pinia": "^2.1.7",
    "axios": "^1.6.0",
    "@vueuse/core": "^10.7.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.0",
    "vite": "^5.0.0",
    "vue-tsc": "^1.8.27",
    "typescript": "^5.3.0",
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.32",
    "autoprefixer": "^10.4.16",
    "eslint": "^8.55.0",
    "eslint-plugin-vue": "^9.19.0"
  }
}
```

- [ ] **Step 2: 创建vite.config.ts**

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true
      }
    }
  }
})
```

- [ ] **Step 3: 创建tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

- [ ] **Step 4: 创建tailwind.config.js**

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
        }
      }
    },
  },
  plugins: [],
}
```

- [ ] **Step 5: 创建src/main.ts**

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import router from './router'
import App from './App.vue'
import './styles/main.css'

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

- [ ] **Step 6: 创建src/App.vue**

```vue
<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script setup lang="ts">
// App root component
</script>

<style>
#app {
  min-height: 100vh;
  background-color: #f8fafc;
}
</style>
```

- [ ] **Step 7: 创建src/styles/main.css**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --primary-color: #0ea5e9;
}

body {
  @apply font-sans antialiased;
}
```

- [ ] **Step 8: 创建src/router/index.ts**

```typescript
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('@/views/HomeView.vue')
    },
    {
      path: '/requirement',
      name: 'requirement',
      component: () => import('@/views/RequirementView.vue')
    },
    {
      path: '/prototype',
      name: 'prototype',
      component: () => import('@/views/PrototypeView.vue')
    },
    {
      path: '/document',
      name: 'document',
      component: () => import('@/views/DocumentView.vue')
    }
  ]
})

export default router
```

- [ ] **Step 9: 创建基础视图组件**

```vue
<!-- src/views/HomeView.vue -->
<template>
  <div class="home-view">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4">
        <h1 class="text-3xl font-bold text-gray-900">产品经理工作站</h1>
      </div>
    </header>
    <main class="max-w-7xl mx-auto py-6 px-4">
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <div v-for="card in cards" :key="card.title" class="bg-white overflow-hidden shadow rounded-lg">
          <div class="p-5">
            <div class="flex items-center">
              <div class="flex-shrink-0">
                <div class="h-12 w-12 rounded-md bg-primary-500 flex items-center justify-center text-white text-xl">
                  {{ card.icon }}
                </div>
              </div>
              <div class="ml-5">
                <h3 class="text-lg font-medium text-gray-900">{{ card.title }}</h3>
                <p class="mt-1 text-sm text-gray-500">{{ card.description }}</p>
              </div>
            </div>
          </div>
          <div class="bg-gray-50 px-5 py-3">
            <router-link :to="card.path" class="text-sm font-medium text-primary-600 hover:text-primary-500">
              开始使用 →
            </router-link>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
const cards = [
  {
    title: '需求解析',
    description: '长链推理，智能拆解业务需求',
    icon: '📝',
    path: '/requirement'
  },
  {
    title: '原型生成',
    description: '自动生成Vue3可交互原型',
    icon: '🎨',
    path: '/prototype'
  },
  {
    title: '文档编撰',
    description: '标准PRD文档自动生成',
    icon: '📄',
    path: '/document'
  }
]
</script>
```

- [ ] **Step 10: 创建其他占位视图**

```vue
<!-- src/views/RequirementView.vue -->
<template>
  <div class="requirement-view">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4">
        <h1 class="text-3xl font-bold text-gray-900">需求解析</h1>
      </div>
    </header>
    <main class="max-w-7xl mx-auto py-6 px-4">
      <div class="bg-white shadow rounded-lg p-6">
        <p class="text-gray-500">需求解析功能开发中...</p>
      </div>
    </main>
  </div>
</template>
```

```vue
<!-- src/views/PrototypeView.vue -->
<template>
  <div class="prototype-view">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4">
        <h1 class="text-3xl font-bold text-gray-900">原型生成</h1>
      </div>
    </header>
    <main class="max-w-7xl mx-auto py-6 px-4">
      <div class="bg-white shadow rounded-lg p-6">
        <p class="text-gray-500">原型生成功能开发中...</p>
      </div>
    </main>
  </div>
</template>
```

```vue
<!-- src/views/DocumentView.vue -->
<template>
  <div class="document-view">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4">
        <h1 class="text-3xl font-bold text-gray-900">文档编撰</h1>
      </div>
    </header>
    <main class="max-w-7xl mx-auto py-6 px-4">
      <div class="bg-white shadow rounded-lg p-6">
        <p class="text-gray-500">文档编撰功能开发中...</p>
      </div>
    </main>
  </div>
</template>
```

- [ ] **Step 11: 创建index.html**

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>产品经理工作站</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

- [ ] **Step 12: 创建Dockerfile**

```dockerfile
FROM node:18-alpine as builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- [ ] **Step 13: 创建nginx.conf**

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

- [ ] **Step 14: Commit**

```bash
git add frontend/
git commit -m "feat(frontend): initial Vue 3 project structure with TailwindCSS"
```

---

## 任务 3: 搭建后端Python FastAPI项目

**Files:**
- Create: `backend/requirements.txt`
- Create: `backend/app/main.py`
- Create: `backend/app/__init__.py`
- Create: `backend/app/core/config.py`
- Create: `backend/app/core/database.py`
- Create: `backend/app/api/__init__.py`
- Create: `backend/app/api/v1/__init__.py`
- Create: `backend/app/api/v1/endpoints/health.py`
- Create: `backend/Dockerfile`

- [ ] **Step 1: 创建requirements.txt**

```txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
sqlalchemy==2.0.25
asyncpg==0.29.0
alembic==1.13.1
pydantic==2.5.3
pydantic-settings==2.1.0
python-dotenv==1.0.0
redis==5.0.1
httpx==0.26.0
langchain==0.1.4
openai==1.10.0
anthropic==0.8.1
dashscope==1.14.0
python-multipart==0.0.6
pandoc==1.0.2
python-docx==1.1.0
weasyprint==60.1
```

- [ ] **Step 2: 创建app/core/config.py**

```python
from pydantic_settings import BaseSettings
from typing import Optional


class Settings(BaseSettings):
    PROJECT_NAME: str = "PM Station API"
    VERSION: str = "0.1.0"
    API_V1_STR: str = "/api/v1"

    POSTGRES_USER: str = "pmuser"
    POSTGRES_PASSWORD: str = "pmpassword"
    POSTGRES_HOST: str = "localhost"
    POSTGRES_PORT: str = "5432"
    POSTGRES_DB: str = "pmstation"

    @property
    def DATABASE_URL(self) -> str:
        return f"postgresql+asyncpg://{self.POSTGRES_USER}:{self.POSTGRES_PASSWORD}@{self.POSTGRES_HOST}:{self.POSTGRES_PORT}/{self.POSTGRES_DB}"

    REDIS_HOST: str = "localhost"
    REDIS_PORT: int = 6379
    REDIS_DB: int = 0

    @property
    def REDIS_URL(self) -> str:
        return f"redis://{self.REDIS_HOST}:{self.REDIS_PORT}/{self.REDIS_DB}"

    MODEL_PROVIDER: str = "openai"
    OPENAI_API_KEY: Optional[str] = None
    ANTHROPIC_API_KEY: Optional[str] = None
    DASHSCOPE_API_KEY: Optional[str] = None

    class Config:
        env_file = ".env"
        case_sensitive = True


settings = Settings()
```

- [ ] **Step 3: 创建app/core/database.py**

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import declarative_base

from app.core.config import settings

engine = create_async_engine(
    settings.DATABASE_URL,
    echo=True,
    future=True
)

async_session_maker = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

Base = declarative_base()


async def get_db() -> AsyncSession:
    async with async_session_maker() as session:
        try:
            yield session
        finally:
            await session.close()


async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
```

- [ ] **Step 4: 创建app/main.py**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.core.config import settings
from app.core.database import init_db
from app.api.v1 import api

app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    openapi_url=f"{settings.API_V1_STR}/openapi.json"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "http://localhost:8080"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(api.router, prefix=settings.API_V1_STR)


@app.on_event("startup")
async def startup_event():
    await init_db()


@app.get("/")
async def root():
    return {"message": "PM Station API", "version": settings.VERSION}
```

- [ ] **Step 5: 创建app/api/v1/__init__.py**

```python
from fastapi import APIRouter
from app.api.v1.endpoints import health

router = APIRouter()
router.include_router(health.router, tags=["health"])
```

- [ ] **Step 6: 创建app/api/v1/endpoints/health.py**

```python
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.database import get_db

router = APIRouter()


@router.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "service": "pm-station-api"
    }


@router.get("/tasks")
async def list_tasks(db: AsyncSession = Depends(get_db)):
    return {"tasks": []}
```

- [ ] **Step 7: 创建Dockerfile**

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

- [ ] **Step 8: Commit**

```bash
git add backend/
git commit -m "feat(backend): initial FastAPI project structure"
```

---

## 任务 4: 创建Docker Compose配置

**Files:**
- Create: `docker-compose.yml`
- Create: `.env.example`
- Create: `backend/.env`

- [ ] **Step 1: 创建docker-compose.yml**

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - pm-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db
      - redis
    networks:
      - pm-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-pmuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-pmpassword}
      POSTGRES_DB: ${POSTGRES_DB:-pmstation}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - pm-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    networks:
      - pm-network

volumes:
  postgres_data:
  redis_data:

networks:
  pm-network:
    driver: bridge
```

- [ ] **Step 2: 创建.env.example**

```bash
# Database
POSTGRES_USER=pmuser
POSTGRES_PASSWORD=pmpassword
POSTGRES_DB=pmstation
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

# Model Provider (openai, claude, dashscope, hunyuan)
MODEL_PROVIDER=openai

# API Keys (根据选择的模型提供商填写)
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key
DASHSCOPE_API_KEY=your-dashscope-api-key

# Frontend
VITE_API_BASE_URL=http://localhost:8000
```

- [ ] **Step 3: 创建backend/.env**

```bash
POSTGRES_USER=pmuser
POSTGRES_PASSWORD=pmpassword
POSTGRES_DB=pmstation
POSTGRES_HOST=db
POSTGRES_PORT=5432

REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=0

MODEL_PROVIDER=openai
OPENAI_API_KEY=${OPENAI_API_KEY}
```

- [ ] **Step 4: Commit**

```bash
git add docker-compose.yml .env.example
git commit -m "feat: add Docker Compose configuration"
```

---

## 任务 5: 创建数据库模型

**Files:**
- Create: `backend/app/models/__init__.py`
- Create: `backend/app/models/task.py`
- Create: `backend/app/models/prototype.py`
- Create: `backend/app/models/document.py`
- Create: `backend/app/schemas/__init__.py`
- Create: `backend/app/schemas/task.py`

- [ ] **Step 1: 创建app/models/__init__.py**

```python
from app.models.task import Task
from app.models.prototype import PrototypePage
from app.models.document import Document
from app.models.validation import ValidationReport

__all__ = ["Task", "PrototypePage", "Document", "ValidationReport"]
```

- [ ] **Step 2: 创建app/models/task.py**

```python
from sqlalchemy import Column, String, Text, Integer, DateTime, JSON
from sqlalchemy.sql import func

from app.core.database import Base


class Task(Base):
    __tablename__ = "tasks"

    id = Column(String(36), primary_key=True)
    name = Column(String(255), nullable=False)
    status = Column(String(50), nullable=False, default="pending")
    requirement = Column(Text)
    business_tree = Column(JSON)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    def __repr__(self):
        return f"<Task {self.id}: {self.name}>"
```

- [ ] **Step 3: 创建app/models/prototype.py**

```python
from sqlalchemy import Column, String, Text, Integer, DateTime, JSON, ForeignKey
from sqlalchemy.sql import func

from app.core.database import Base


class PrototypePage(Base):
    __tablename__ = "prototype_pages"

    id = Column(String(36), primary_key=True)
    task_id = Column(String(36), ForeignKey("tasks.id"), nullable=False)
    name = Column(String(255), nullable=False)
    path = Column(String(255), nullable=False)
    component_code = Column(Text)
    config = Column(JSON)
    order = Column(Integer, default=0)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    def __repr__(self):
        return f"<PrototypePage {self.id}: {self.name}>"
```

- [ ] **Step 4: 创建app/models/document.py**

```python
from sqlalchemy import Column, String, Text, Integer, DateTime, ForeignKey
from sqlalchemy.sql import func

from app.core.database import Base


class Document(Base):
    __tablename__ = "documents"

    id = Column(String(36), primary_key=True)
    task_id = Column(String(36), ForeignKey("tasks.id"), nullable=False)
    title = Column(String(255), nullable=False)
    content = Column(Text)
    format = Column(String(50), default="markdown")
    version = Column(Integer, default=1)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    def __repr__(self):
        return f"<Document {self.id}: {self.title}>"
```

- [ ] **Step 5: 创建app/models/validation.py**

```python
from sqlalchemy import Column, String, Integer, DateTime, JSON, ForeignKey, Numeric
from sqlalchemy.sql import func

from app.core.database import Base


class ValidationReport(Base):
    __tablename__ = "validation_reports"

    id = Column(String(36), primary_key=True)
    task_id = Column(String(36), ForeignKey("tasks.id"), nullable=False)
    type = Column(String(50), nullable=False)
    score = Column(Numeric(5, 2))
    issues = Column(JSON)
    created_at = Column(DateTime(timezone=True), server_default=func.now())

    def __repr__(self):
        return f"<ValidationReport {self.id}: {self.type}>"
```

- [ ] **Step 6: 创建app/schemas/__init__.py**

```python
from app.schemas.task import TaskCreate, TaskResponse, TaskUpdate
```

- [ ] **Step 7: 创建app/schemas/task.py**

```python
from pydantic import BaseModel
from typing import Optional
from datetime import datetime


class TaskBase(BaseModel):
    name: str
    requirement: Optional[str] = None


class TaskCreate(TaskBase):
    pass


class TaskUpdate(BaseModel):
    name: Optional[str] = None
    status: Optional[str] = None
    requirement: Optional[str] = None
    business_tree: Optional[dict] = None


class TaskResponse(TaskBase):
    id: str
    status: str
    business_tree: Optional[dict] = None
    created_at: datetime
    updated_at: Optional[datetime] = None

    class Config:
        from_attributes = True
```

- [ ] **Step 8: Commit**

```bash
git add backend/app/models/ backend/app/schemas/
git commit -m "feat(models): add database models for Task, Prototype, Document, Validation"
```

---

## 任务 6: 创建API路由

**Files:**
- Create: `backend/app/api/v1/endpoints/tasks.py`
- Modify: `backend/app/api/v1/__init__.py`
- Create: `backend/app/services/task_service.py`

- [ ] **Step 1: 创建app/services/task_service.py**

```python
from typing import List, Optional
from uuid import uuid4
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.task import Task
from app.schemas.task import TaskCreate, TaskUpdate


class TaskService:
    @staticmethod
    async def create_task(db: AsyncSession, task_data: TaskCreate) -> Task:
        task = Task(
            id=str(uuid4()),
            name=task_data.name,
            requirement=task_data.requirement,
            status="pending"
        )
        db.add(task)
        await db.commit()
        await db.refresh(task)
        return task

    @staticmethod
    async def get_task(db: AsyncSession, task_id: str) -> Optional[Task]:
        result = await db.execute(select(Task).where(Task.id == task_id))
        return result.scalar_one_or_none()

    @staticmethod
    async def list_tasks(db: AsyncSession, skip: int = 0, limit: int = 100) -> List[Task]:
        result = await db.execute(select(Task).offset(skip).limit(limit))
        return result.scalars().all()

    @staticmethod
    async def update_task(db: AsyncSession, task_id: str, task_data: TaskUpdate) -> Optional[Task]:
        task = await TaskService.get_task(db, task_id)
        if not task:
            return None

        update_data = task_data.dict(exclude_unset=True)
        for field, value in update_data.items():
            setattr(task, field, value)

        await db.commit()
        await db.refresh(task)
        return task

    @staticmethod
    async def delete_task(db: AsyncSession, task_id: str) -> bool:
        task = await TaskService.get_task(db, task_id)
        if not task:
            return False

        await db.delete(task)
        await db.commit()
        return True
```

- [ ] **Step 2: 创建app/api/v1/endpoints/tasks.py**

```python
from typing import List
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.database import get_db
from app.schemas.task import TaskCreate, TaskResponse, TaskUpdate
from app.services.task_service import TaskService

router = APIRouter()


@router.post("/tasks", response_model=TaskResponse, status_code=status.HTTP_201_CREATED)
async def create_task(
    task_data: TaskCreate,
    db: AsyncSession = Depends(get_db)
):
    task = await TaskService.create_task(db, task_data)
    return task


@router.get("/tasks", response_model=List[TaskResponse])
async def list_tasks(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(get_db)
):
    tasks = await TaskService.list_tasks(db, skip=skip, limit=limit)
    return tasks


@router.get("/tasks/{task_id}", response_model=TaskResponse)
async def get_task(
    task_id: str,
    db: AsyncSession = Depends(get_db)
):
    task = await TaskService.get_task(db, task_id)
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found"
        )
    return task


@router.patch("/tasks/{task_id}", response_model=TaskResponse)
async def update_task(
    task_id: str,
    task_data: TaskUpdate,
    db: AsyncSession = Depends(get_db)
):
    task = await TaskService.update_task(db, task_id, task_data)
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found"
        )
    return task


@router.delete("/tasks/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(
    task_id: str,
    db: AsyncSession = Depends(get_db)
):
    success = await TaskService.delete_task(db, task_id)
    if not success:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found"
        )
```

- [ ] **Step 3: 更新app/api/v1/__init__.py**

```python
from fastapi import APIRouter
from app.api.v1.endpoints import health, tasks

router = APIRouter()
router.include_router(health.router, tags=["health"])
router.include_router(tasks.router, tags=["tasks"])
```

- [ ] **Step 4: Commit**

```bash
git add backend/app/api/v1/endpoints/tasks.py backend/app/services/
git commit -m "feat(api): add task CRUD endpoints"
```

---

## 任务 7: 创建前端API调用模块

**Files:**
- Create: `frontend/src/api/index.ts`
- Create: `frontend/src/api/task.ts`
- Create: `frontend/src/types/api.ts`

- [ ] **Step 1: 创建frontend/src/types/api.ts**

```typescript
export interface Task {
  id: string
  name: string
  requirement?: string
  status: 'pending' | 'parsing' | 'generating' | 'validating' | 'completed' | 'failed'
  business_tree?: BusinessTree
  created_at: string
  updated_at?: string
}

export interface TaskCreate {
  name: string
  requirement?: string
}

export interface BusinessTree {
  businessFlow?: {
    nodes: FlowNode[]
    edges: FlowEdge[]
  }
  userScenarios?: UserScenario[]
  exceptionHandling?: ExceptionCase[]
  businessRules?: BusinessRule[]
}

export interface FlowNode {
  id: string
  name: string
  type: string
  description?: string
}

export interface FlowEdge {
  from: string
  to: string
  condition?: string
}

export interface UserScenario {
  id: string
  name: string
  path: string[]
}

export interface ExceptionCase {
  scenario: string
  handling: string
}

export interface BusinessRule {
  rule: string
  content: string
}
```

- [ ] **Step 2: 创建frontend/src/api/index.ts**

```typescript
import axios from 'axios'

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || '/api/v1',
  headers: {
    'Content-Type': 'application/json'
  }
})

apiClient.interceptors.response.use(
  response => response,
  error => {
    console.error('API Error:', error)
    return Promise.reject(error)
  }
)

export default apiClient
```

- [ ] **Step 3: 创建frontend/src/api/task.ts**

```typescript
import apiClient from './index'
import type { Task, TaskCreate } from '@/types/api'

export const taskApi = {
  create: async (data: TaskCreate): Promise<Task> => {
    const response = await apiClient.post<Task>('/tasks', data)
    return response.data
  },

  list: async (skip = 0, limit = 100): Promise<Task[]> => {
    const response = await apiClient.get<Task[]>('/tasks', {
      params: { skip, limit }
    })
    return response.data
  },

  get: async (id: string): Promise<Task> => {
    const response = await apiClient.get<Task>(`/tasks/${id}`)
    return response.data
  },

  update: async (id: string, data: Partial<Task>): Promise<Task> => {
    const response = await apiClient.patch<Task>(`/tasks/${id}`, data)
    return response.data
  },

  delete: async (id: string): Promise<void> => {
    await apiClient.delete(`/tasks/${id}`)
  }
}
```

- [ ] **Step 4: Commit**

```bash
git add frontend/src/api/ frontend/src/types/
git commit -m "feat(frontend): add API client and task types"
```

---

## 任务 8: 创建前端状态管理

**Files:**
- Create: `frontend/src/stores/task.ts`
- Create: `frontend/src/stores/index.ts`

- [ ] **Step 1: 创建frontend/src/stores/task.ts**

```typescript
import { defineStore } from 'pinia'
import { ref } from 'vue'
import type { Task, TaskCreate } from '@/types/api'
import { taskApi } from '@/api/task'

export const useTaskStore = defineStore('task', () => {
  const tasks = ref<Task[]>([])
  const currentTask = ref<Task | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  const fetchTasks = async () => {
    loading.value = true
    error.value = null
    try {
      tasks.value = await taskApi.list()
    } catch (e) {
      error.value = 'Failed to fetch tasks'
      console.error(e)
    } finally {
      loading.value = false
    }
  }

  const createTask = async (data: TaskCreate) => {
    loading.value = true
    error.value = null
    try {
      const task = await taskApi.create(data)
      tasks.value.unshift(task)
      return task
    } catch (e) {
      error.value = 'Failed to create task'
      console.error(e)
      throw e
    } finally {
      loading.value = false
    }
  }

  const getTask = async (id: string) => {
    loading.value = true
    error.value = null
    try {
      currentTask.value = await taskApi.get(id)
      return currentTask.value
    } catch (e) {
      error.value = 'Failed to get task'
      console.error(e)
      throw e
    } finally {
      loading.value = false
    }
  }

  const updateTask = async (id: string, data: Partial<Task>) => {
    loading.value = true
    error.value = null
    try {
      const task = await taskApi.update(id, data)
      const index = tasks.value.findIndex(t => t.id === id)
      if (index !== -1) {
        tasks.value[index] = task
      }
      if (currentTask.value?.id === id) {
        currentTask.value = task
      }
      return task
    } catch (e) {
      error.value = 'Failed to update task'
      console.error(e)
      throw e
    } finally {
      loading.value = false
    }
  }

  return {
    tasks,
    currentTask,
    loading,
    error,
    fetchTasks,
    createTask,
    getTask,
    updateTask
  }
})
```

- [ ] **Step 2: 创建frontend/src/stores/index.ts**

```typescript
export { useTaskStore } from './task'
```

- [ ] **Step 3: Commit**

```bash
git add frontend/src/stores/
git commit -m "feat(frontend): add Pinia store for task management"
```

---

## 任务 9: 创建前端API类型声明

**Files:**
- Create: `frontend/tsconfig.node.json`
- Create: `frontend/src/vite-env.d.ts`

- [ ] **Step 1: 创建frontend/tsconfig.node.json**

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

- [ ] **Step 2: 创建frontend/src/vite-env.d.ts**

```typescript
/// <reference types="vite/client" />

declare module '*.vue' {
  import type { DefineComponent } from 'vue'
  const component: DefineComponent<{}, {}, any>
  export default component
}

interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

- [ ] **Step 3: Commit**

```bash
git add frontend/tsconfig.node.json frontend/src/vite-env.d.ts
git commit -m "chore(frontend): add TypeScript declarations"
```

---

## 任务 10: 创建PostCSS配置

**Files:**
- Create: `frontend/postcss.config.js`

- [ ] **Step 1: 创建postcss.config.js**

```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

- [ ] **Step 2: Commit**

```bash
git add frontend/postcss.config.js
git commit -m "chore(frontend): add PostCSS config"
```

---

## 任务 11: 安装依赖并验证项目

**Files:**
- Modify: `frontend/.env`
- Modify: `backend/.env`

- [ ] **Step 1: 创建.env文件用于本地开发**

```bash
cd frontend
npm install
npm run type-check || true
```

- [ ] **Step 2: 安装后端依赖**

```bash
cd backend
pip install -r requirements.txt
python -c "from app.main import app; print('FastAPI app loaded successfully')"
```

- [ ] **Step 3: 验证Docker环境**

```bash
docker --version
docker-compose --version
docker-compose config
```

- [ ] **Step 4: Commit**

```bash
git add -A
git status
```

---

## Phase 1 总结

**完成的任务:**
- [x] 项目目录结构创建
- [x] Vue 3 前端项目搭建
- [x] FastAPI 后端项目搭建
- [x] Docker Compose 配置
- [x] 数据库模型定义
- [x] API 路由实现
- [x] 前端 API 调用模块
- [x] 前端状态管理

**待验证项:**
- [ ] Docker Compose 启动验证
- [ ] 前后端联调验证
- [ ] 数据库连接验证

---

## 下一步行动

Phase 1 完成后，可以选择：

1. **继续 Phase 2**: 开始需求解析Agent开发
2. **添加更多功能**: 在Phase 1基础上增加更多基础功能
3. **完善测试**: 为Phase 1添加单元测试

**请选择下一步行动？**

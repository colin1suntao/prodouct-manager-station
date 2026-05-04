# 产品经理个人工作站 - 系统设计文档

## 1. 项目概述

### 1.1 项目背景

产品经理在日常工作中需要处理大量碎片化的业务需求，涵盖需求解析、原型绘制、文档编撰等多个环节。传统工作模式存在以下痛点：

- **人工梳理逻辑疏漏**：业务规则、操作分支、异常场景容易遗漏
- **跨工具切换成本高**：需求文档、原型设计、PRD编写需要使用多个独立工具
- **交付标准不统一**：不同产品经理输出的文档和原型风格差异大
- **返工率高**：需求变更后需要同步更新多个产出物

### 1.2 项目目标

构建一个智能化的产品经理个人工作站，采用多Agent协作+长链推理的核心方案，实现：

- **需求智能解析**：通过长链推理能力精准拆解业务逻辑，覆盖多场景规则、用户操作分支及异常兜底机制
- **物料自动化生成**：基于结构化数据自动生成Vue3可交互原型和标准PRD文档
- **双向校验纠错**：对原型交互逻辑和文档内容进行自动化核验，确保交付物精准、合规
- **全链路自动化**：覆盖需求拆解、物料产出、自查纠错的全流程

### 1.3 核心特性

| 特性 | 描述 |
|-----|------|
| 长链推理解析 | 支持多轮对话式需求澄清，自动拆解业务规则、操作分支、异常场景 |
| 多Agent协作 | 需求解析、原型生成、文档生成、校验纠错四大Agent协同作业 |
| 多模型适配 | 支持OpenAI、Claude、通义、混元等主流大模型，可根据任务切换 |
| Vue3动态原型 | 基于Vue 3 + Vite构建可交互原型页面，支持页面流转和组件复用 |
| 多格式文档 | 生成Markdown格式PRD文档，支持导出Word/PDF |
| 本地化部署 | Docker容器化部署，数据完全可控 |

## 2. 系统架构

### 2.1 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        用户交互层 (Web UI)                        │
│                    Vue 3 + Vite + TailwindCSS                    │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                        核心调度层 (主控协调)                        │
│                      需求分发 · 状态管理 · 任务编排                  │
└─────────────────────────────────────────────────────────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   需求解析 Agent │   │   原型生成 Agent │   │   文档生成 Agent │
│  (长链推理引擎)  │   │  (Vue3 组件)    │   │  (Markdown/PR)  │
└─────────────────┘   └─────────────────┘   └─────────────────┘
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      校验 Agent (双向核验)                         │
│                  原型逻辑校验 · 文档规范校验                        │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                        模型抽象层 (多模型适配)                      │
│              OpenAI · Claude · 通义 · 混元 · 本地模型              │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 架构说明

本系统采用「核心推理引擎 + 模块化Agent」的混合架构：

- **核心推理引擎**：作为系统的大脑，负责复杂业务逻辑的拆解和推理
- **模块化Agent设计**：每个Agent专注单一职责，通过标准化接口通信
- **主控协调模式**：需求解析Agent作为主控节点，协调其他Agent并行作业
- **模型抽象层**：解耦底层模型实现，支持根据任务类型选择最优模型

### 2.3 技术选型

| 层级 | 技术栈 | 说明 |
|-----|-------|------|
| **前端** | Vue 3 + Vite + TailwindCSS | 用户交互界面 |
| **后端** | Python FastAPI | API服务层、Agent调度 |
| **长链推理** | LangChain / LlamaIndex | 推理链编排、上下文管理 |
| **模型适配** | ModelScope / OpenAI SDK | 多模型统一接口 |
| **原型渲染** | Vue 3 组件库 | 可交互原型输出 |
| **文档生成** | Markdown + pandoc | 多格式文档支持 |
| **本地部署** | Docker + Docker Compose | 环境一致性保障 |

## 3. 核心Agent设计

### 3.1 Agent职责矩阵

| Agent | 核心职责 | 输入 | 输出 |
|-------|---------|------|------|
| **需求解析Agent** | 长链推理，拆解业务规则 | 原始需求描述 | 结构化业务逻辑树 |
| **原型生成Agent** | Vue3组件构建 | 结构化数据 | 可交互原型页面 |
| **文档生成Agent** | PRD文档编写 | 结构化数据 | Markdown/PRDoc |
| **校验Agent** | 双向核验纠错 | 原型/文档 | 校验报告 & 修复建议 |

### 3.2 需求解析Agent

#### 3.2.1 核心能力

- **长链推理引擎**：基于多轮对话逐步澄清需求细节
- **业务规则拆解**：识别业务流程、决策节点、数据约束
- **分支场景覆盖**：穷举用户操作路径，包括正常流程和异常分支
- **异常兜底识别**：识别系统异常、边界条件、降级策略

#### 3.2.2 输出数据结构

```json
{
  "businessFlow": {
    "nodes": [
      {
        "id": "step_001",
        "name": "用户登录",
        "type": "action",
        "description": "用户输入账号密码进行登录",
        "input": ["用户名", "密码"],
        "output": ["登录态Token"],
        "rules": ["密码错误3次锁定30分钟"]
      }
    ],
    "edges": [
      {"from": "step_001", "to": "step_002", "condition": "登录成功"}
    ]
  },
  "userScenarios": [
    {
      "id": "scenario_001",
      "name": "新用户首次登录",
      "path": ["step_001", "step_003", "step_004"],
      "expectations": ["引导完成个人信息完善"]
    }
  ],
  "exceptionHandling": [
    {
      "scenario": "网络断开",
      "handling": "展示断网提示，提供重试按钮",
      "recovery": "网络恢复后自动重试"
    }
  ],
  "businessRules": [
    {
      "rule": "密码强度要求",
      "content": "至少8位，包含大小写字母和数字",
      "priority": "high"
    }
  ]
}
```

### 3.3 原型生成Agent

#### 3.3.1 核心能力

- **页面结构生成**：根据业务逻辑树自动生成页面清单和跳转关系
- **Vue3组件构建**：生成符合TailwindCSS样式规范的Vue3组件代码
- **交互配置**：自动配置页面间的跳转逻辑、表单验证规则
- **组件复用**：支持从组件库选择可复用组件

#### 3.3.2 组件规范

```vue
<template>
  <div class="pm-page">
    <!-- 页面容器 -->
  </div>
</template>

<script setup>
// 页面逻辑
</script>

<style scoped>
/* 页面样式 */
</style>
```

### 3.4 文档生成Agent

#### 3.4.1 核心能力

- **PRD模板渲染**：基于结构化数据渲染标准PRD文档
- **Markdown生成**：输出符合团队规范的Markdown文档
- **多格式导出**：支持导出为Word、PDF格式
- **版本管理**：支持文档版本对比和历史记录

#### 3.4.2 PRD文档结构

```markdown
# 产品需求文档

## 1. 概述
- 产品名称
- 需求背景
- 需求目标

## 2. 用户故事

## 3. 功能需求
### 3.1 功能点1
### 3.2 功能点2

## 4. 非功能需求
- 性能要求
- 安全要求
- 兼容性要求

## 5. 业务流程
- 流程图
- 流程说明

## 6. 页面原型

## 7. 异常处理

## 8. 验收标准
```

### 3.5 校验Agent

#### 3.5.1 原型逻辑校验

- **页面完整性**：检查是否有遗漏的页面
- **跳转逻辑**：检查页面跳转链路是否完整
- **表单验证**：检查表单校验规则是否完整
- **异常覆盖**：检查异常场景是否有对应提示

#### 3.5.2 文档规范校验

- **格式规范**：检查文档格式是否符合模板
- **内容完整**：检查各章节是否完整
- **术语统一**：检查专业术语是否统一
- **逻辑一致**：检查文档内容与原型是否一致

#### 3.5.3 校验报告格式

```json
{
  "prototype": {
    "issues": [
      {
        "type": "missing_page",
        "severity": "high",
        "description": "缺少忘记密码页面",
        "suggestion": "建议新增/forget-password页面"
      }
    ],
    "score": 85
  },
  "document": {
    "issues": [
      {
        "type": "missing_section",
        "severity": "medium",
        "description": "缺少性能要求章节",
        "suggestion": "建议补充非功能需求章节"
      }
    ],
    "score": 90
  }
}
```

## 4. 数据流设计

### 4.1 完整工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                        1. 需求输入                               │
│    用户通过Web界面输入原始需求描述，支持多格式输入                  │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                   2. 需求解析 Agent (长链推理)                    │
│    多轮对话澄清需求 → 拆解业务规则 → 识别用户分支 → 梳理异常场景      │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    3. 生成结构化业务逻辑树                          │
│    包含业务流程节点、用户场景、异常处理、业务规则                     │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼ (并行分发)
          ┌─────────────────────────────────────────────┐
          │              4. 并行作业                     │
          ├───────────────────┬───────────────────────┤
          │   原型生成 Agent   │    文档生成 Agent       │
          │                   │                        │
          │ • 页面结构生成     │ • PRD模板渲染          │
          │ • Vue3组件构建     │ • Markdown生成         │
          │ • 交互配置        │ • 多格式导出            │
          └───────────────────┴───────────────────────┘
                                  │
                                  ▼ (汇合)
┌─────────────────────────────────────────────────────────────────┐
│                      5. 校验 Agent                                │
│    原型逻辑校验 ← ─ ─ ─ ─ ─ ─ → 文档规范校验                      │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      6. 输出交付物                                │
│    Vue3可交互原型 + 标准PRD文档 + 校验报告                         │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 状态管理

```typescript
interface TaskState {
  taskId: string;
  status: 'pending' | 'parsing' | 'generating' | 'validating' | 'completed' | 'failed';
  progress: {
    requirement: number;
    prototype: number;
    document: number;
    validation: number;
  };
  outputs: {
    businessTree?: BusinessTree;
    prototype?: PrototypeOutput;
    document?: DocumentOutput;
    report?: ValidationReport;
  };
  errors?: Error[];
}
```

## 5. 模块设计

### 5.1 需求输入中心

#### 5.1.1 功能列表

- 多格式需求输入（文本、模板填充、截图标注）
- 需求模板快速填充
- 历史需求管理

#### 5.1.2 界面布局

```
┌─────────────────────────────────────────────────────────┐
│  需求输入                                                 │
├─────────────────────────────────────────────────────────┤
│  [文本输入]  [模板填充]  [历史记录]                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  请输入需求描述...                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │                                                 │   │
│  │                                                 │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  [开始解析]                                              │
└─────────────────────────────────────────────────────────┘
```

### 5.2 长链推理中心

#### 5.2.1 功能列表

- 多轮对话式需求澄清
- 推理过程可视化
- 逻辑推导链展示

#### 5.2.2 交互流程

1. 系统基于初始需求生成澄清问题
2. 用户回答问题
3. 系统更新需求理解，生成下一轮问题或确认
4. 重复直到需求完整澄清
5. 输出结构化业务逻辑树

### 5.3 原型生成中心

#### 5.3.1 功能列表

- 页面列表管理
- 页面编辑器
- 组件库管理
- 预览与导出

#### 5.3.2 组件库结构

```
components/
├── base/           # 基础组件
│   ├── Button.vue
│   ├── Input.vue
│   ├── Select.vue
│   └── ...
├── layout/         # 布局组件
│   ├── Header.vue
│   ├── Sidebar.vue
│   └── ...
├── business/      # 业务组件
│   ├── Form.vue
│   ├── Table.vue
│   └── ...
└── feedback/       # 反馈组件
    ├── Modal.vue
    ├── Toast.vue
    └── ...
```

### 5.4 文档生成中心

#### 5.4.1 功能列表

- PRD文档预览
- Markdown编辑
- 多格式导出
- 版本管理

#### 5.4.2 导出格式支持

- Markdown (.md)
- Word (.docx)
- PDF (.pdf)

### 5.5 校验纠错中心

#### 5.5.1 功能列表

- 校验任务管理
- 问题列表展示
- 修复建议生成
- 批量修复操作

#### 5.5.2 校验规则

```python
VALIDATION_RULES = {
    "prototype": [
        {"name": "page_completeness", "weight": 0.2},
        {"name": "navigation_logic", "weight": 0.3},
        {"name": "form_validation", "weight": 0.25},
        {"name": "exception_coverage", "weight": 0.25}
    ],
    "document": [
        {"name": "format_compliance", "weight": 0.2},
        {"name": "content_completeness", "weight": 0.3},
        {"name": "terminology_consistency", "weight": 0.25},
        {"name": "logic_alignment", "weight": 0.25}
    ]
}
```

## 6. API设计

### 6.1 核心接口

| 接口 | 方法 | 描述 |
|-----|------|------|
| `/api/tasks` | POST | 创建新任务 |
| `/api/tasks/{taskId}` | GET | 获取任务状态 |
| `/api/requirement/parse` | POST | 解析需求 |
| `/api/requirement/clarify` | POST | 需求澄清对话 |
| `/api/prototype/generate` | POST | 生成原型 |
| `/api/prototype/export` | GET | 导出原型 |
| `/api/document/generate` | POST | 生成文档 |
| `/api/document/export` | GET | 导出文档 |
| `/api/validation/check` | POST | 执行校验 |

### 6.2 接口示例

#### 创建任务

```http
POST /api/tasks
Content-Type: application/json

{
  "name": "用户登录模块",
  "requirement": "实现用户登录功能，支持账号密码登录和短信验证码登录..."
}
```

#### 响应

```json
{
  "taskId": "task_001",
  "status": "parsing",
  "createdAt": "2024-01-01T10:00:00Z"
}
```

## 7. 数据库设计

### 7.1 核心表结构

#### 7.1.1 任务表 (tasks)

```sql
CREATE TABLE tasks (
  id VARCHAR(36) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  status VARCHAR(50) NOT NULL,
  requirement TEXT,
  business_tree JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### 7.1.2 页面表 (prototype_pages)

```sql
CREATE TABLE prototype_pages (
  id VARCHAR(36) PRIMARY KEY,
  task_id VARCHAR(36) NOT NULL,
  name VARCHAR(255) NOT NULL,
  path VARCHAR(255) NOT NULL,
  component_code TEXT,
  config JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

#### 7.1.3 文档表 (documents)

```sql
CREATE TABLE documents (
  id VARCHAR(36) PRIMARY KEY,
  task_id VARCHAR(36) NOT NULL,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  format VARCHAR(50),
  version INT DEFAULT 1,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

#### 7.1.4 校验报告表 (validation_reports)

```sql
CREATE TABLE validation_reports (
  id VARCHAR(36) PRIMARY KEY,
  task_id VARCHAR(36) NOT NULL,
  type VARCHAR(50) NOT NULL,
  score DECIMAL(5,2),
  issues JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (task_id) REFERENCES tasks(id)
);
```

## 8. 部署架构

### 8.1 Docker Compose 架构

```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/pm_station
      - MODEL_PROVIDER=${MODEL_PROVIDER}
      - MODEL_API_KEY=${MODEL_API_KEY}
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=pm_station

  redis:
    image: redis:7
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 8.2 环境变量配置

```bash
# 模型配置
MODEL_PROVIDER=openai  # openai | claude | dashscope | hunyuan
MODEL_API_KEY=your-api-key
MODEL_BASE_URL=https://api.openai.com/v1  # 可选，自定义端点

# 数据库配置
DATABASE_URL=postgresql://user:pass@localhost:5432/pm_station

# Redis配置
REDIS_URL=redis://localhost:6379

# 前端配置
VITE_API_BASE_URL=http://localhost:8000
```

## 9. 项目目录结构

```
product-manager-station/
├── frontend/                    # Vue 3 前端项目
│   ├── src/
│   │   ├── components/         # Vue组件
│   │   ├── views/              # 页面视图
│   │   ├── stores/             # 状态管理
│   │   ├── api/                # API调用
│   │   └── styles/             # 全局样式
│   ├── Dockerfile
│   └── package.json
│
├── backend/                     # Python FastAPI 后端
│   ├── app/
│   │   ├── agents/             # Agent实现
│   │   │   ├── requirement/     # 需求解析Agent
│   │   │   ├── prototype/      # 原型生成Agent
│   │   │   ├── document/        # 文档生成Agent
│   │   │   └── validation/      # 校验Agent
│   │   ├── core/                # 核心模块
│   │   ├── models/              # 数据模型
│   │   ├── schemas/             # Pydantic schemas
│   │   └── routers/             # API路由
│   ├── Dockerfile
│   └── requirements.txt
│
├── docs/                       # 文档
│   └── specs/                   # 设计文档
│
├── docker-compose.yml
├── .env.example
└── README.md
```

## 10. 验收标准

### 10.1 功能验收

- [ ] 支持需求文本输入和多轮对话澄清
- [ ] 能够生成包含业务规则、操作分支、异常场景的结构化逻辑树
- [ ] 能够基于结构化数据生成Vue3可交互原型页面
- [ ] 能够生成符合模板规范的Markdown格式PRD文档
- [ ] 支持导出Word和PDF格式文档
- [ ] 能够对原型和文档进行自动化校验
- [ ] 支持多模型切换（OpenAI/Claude/通义/混元）

### 10.2 性能验收

- [ ] 需求解析响应时间 < 30秒（不含模型推理时间）
- [ ] 原型页面生成时间 < 60秒
- [ ] 文档生成时间 < 30秒
- [ ] 校验完成时间 < 10秒

### 10.3 部署验收

- [ ] 支持Docker Compose一键部署
- [ ] 支持本地化运行，数据不出本机
- [ ] 前端启动时间 < 30秒
- [ ] 后端启动时间 < 30秒

## 11. 里程碑规划

### Phase 1: 核心框架搭建
- 项目脚手架搭建
- 前后端基础架构
- Docker环境配置

### Phase 2: 需求解析Agent
- 长链推理引擎实现
- 需求澄清对话流程
- 结构化逻辑树生成

### Phase 3: 原型生成Agent
- Vue3组件库建设
- 页面生成逻辑
- 交互配置系统

### Phase 4: 文档生成Agent
- PRD模板引擎
- Markdown生成
- 多格式导出

### Phase 5: 校验Agent
- 原型逻辑校验
- 文档规范校验
- 修复建议生成

### Phase 6: 集成与优化
- Agent协作流程优化
- 多模型适配完善
- 界面交互优化

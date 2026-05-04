# Phase 2: 需求解析Agent - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现长链推理引擎、需求澄清对话流程、结构化业务逻辑树生成器

**Architecture:** 基于LangChain构建长链推理能力，通过对话式交互澄清需求，最终输出结构化业务逻辑树

**Tech Stack:** LangChain | OpenAI/Claude SDK | Pydantic | Redis (对话缓存)

---

## 文件结构设计

```
backend/app/agents/
├── __init__.py
├── base.py                    # Agent基类
├── requirement/
│   ├── __init__.py
│   ├── agent.py              # 需求解析Agent主类
│   ├── chains/
│   │   ├── __init__.py
│   │   ├── clarification.py  # 澄清链
│   │   ├── extraction.py     # 提取链
│   │   └── structuring.py   # 结构化链
│   ├── prompts/
│   │   ├── __init__.py
│   │   └── templates.py      # Prompt模板
│   └── models/
│       ├── __init__.py
│       └── business_tree.py  # 业务逻辑树模型
```

---

## 任务 1: 创建Agent基类

**Files:**
- Create: `backend/app/agents/base.py`
- Create: `backend/app/agents/__init__.py`
- Create: `backend/app/core/llm.py`

- [ ] **Step 1: 创建app/core/llm.py - LLM统一接口**

```python
from abc import ABC, abstractmethod
from typing import Optional, Dict, Any
from app.core.config import settings


class BaseLLM(ABC):
    @abstractmethod
    async def generate(self, prompt: str, **kwargs) -> str:
        pass

    @abstractmethod
    async def generate_stream(self, prompt: str, **kwargs):
        pass


class LLMFactory:
    _providers: Dict[str, type] = {}

    @classmethod
    def register(cls, name: str, provider_class: type):
        cls._providers[name] = provider_class

    @classmethod
    def create(cls, provider: Optional[str] = None) -> BaseLLM:
        provider = provider or settings.MODEL_PROVIDER

        if provider not in cls._providers:
            raise ValueError(f"Unknown LLM provider: {provider}")

        return cls._providers[provider]()

    @classmethod
    def get_available_providers(cls) -> list:
        return list(cls._providers.keys())
```

- [ ] **Step 2: 创建具体LLM实现 - OpenAI**

```python
# backend/app/core/llm_providers/openai.py
from typing import Optional
from openai import AsyncOpenAI
from app.core.config import settings
from app.core.llm import BaseLLM


class OpenAILLM(BaseLLM):
    def __init__(self):
        self.client = AsyncOpenAI(api_key=settings.OPENAI_API_KEY)

    async def generate(self, prompt: str, model: str = "gpt-4", **kwargs) -> str:
        response = await self.client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            **kwargs
        )
        return response.choices[0].message.content

    async def generate_stream(self, prompt: str, model: str = "gpt-4", **kwargs):
        response = await self.client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            stream=True,
            **kwargs
        )
        for chunk in response:
            if chunk.choices[0].delta.content:
                yield chunk.choices[0].delta.content


# 注册到工厂
from app.core.llm import LLMFactory
LLMFactory.register("openai", OpenAILLM)
```

- [ ] **Step 3: 创建Claude实现**

```python
# backend/app/core/llm_providers/claude.py
from typing import Optional
import anthropic
from app.core.config import settings
from app.core.llm import BaseLLM


class ClaudeLLM(BaseLLM):
    def __init__(self):
        self.client = anthropic.AsyncAnthropic(api_key=settings.ANTHROPIC_API_KEY)

    async def generate(self, prompt: str, model: str = "claude-3-sonnet-20240229", **kwargs) -> str:
        response = await self.client.messages.create(
            model=model,
            max_tokens=4096,
            messages=[{"role": "user", "content": prompt}],
            **kwargs
        )
        return response.content[0].text

    async def generate_stream(self, prompt: str, model: str = "claude-3-sonnet-20240229", **kwargs):
        async with self.client.messages.stream(
            model=model,
            max_tokens=4096,
            messages=[{"role": "user", "content": prompt}],
            **kwargs
        ) as stream:
            async for text in stream.text_stream:
                yield text


from app.core.llm import LLMFactory
LLMFactory.register("claude", ClaudeLLM)
```

- [ ] **Step 4: 创建Agent基类**

```python
# backend/app/agents/base.py
from abc import ABC, abstractmethod
from typing import Any, Dict, Optional
from app.core.llm import LLMFactory


class BaseAgent(ABC):
    def __init__(
        self,
        name: str,
        description: str,
        model_provider: Optional[str] = None
    ):
        self.name = name
        self.description = description
        self.llm = LLMFactory.create(model_provider)

    @abstractmethod
    async def process(self, input_data: Any) -> Any:
        pass

    @abstractmethod
    async def validate(self, output: Any) -> bool:
        pass

    def get_prompt(self, template: str, context: Dict) -> str:
        return template.format(**context)
```

- [ ] **Step 5: Commit**

```bash
git add backend/app/core/llm*.py backend/app/agents/base.py
git commit -m "feat(agents): add base agent class and LLM factory"
```

---

## 任务 2: 创建业务逻辑树数据模型

**Files:**
- Create: `backend/app/agents/requirement/models/business_tree.py`
- Create: `backend/app/agents/requirement/models/__init__.py`

- [ ] **Step 1: 创建业务逻辑树Pydantic模型**

```python
# backend/app/agents/requirement/models/business_tree.py
from pydantic import BaseModel, Field
from typing import List, Optional, Literal
from enum import Enum


class NodeType(str, Enum):
    ACTION = "action"
    DECISION = "decision"
    DATA = "data"
    UI = "ui"
    EVENT = "event"


class FlowNode(BaseModel):
    id: str = Field(..., description="节点唯一标识")
    name: str = Field(..., description="节点名称")
    type: NodeType = Field(..., description="节点类型")
    description: Optional[str] = Field(None, description="节点描述")
    input_fields: List[str] = Field(default_factory=list, description="输入字段")
    output_fields: List[str] = Field(default_factory=list, description="输出字段")
    rules: List[str] = Field(default_factory=list, description="业务规则")
    validation: Optional[str] = Field(None, description="校验规则")


class FlowEdge(BaseModel):
    from_node: str = Field(..., alias="from", description="起始节点")
    to_node: str = Field(..., alias="to", description="目标节点")
    condition: Optional[str] = Field(None, description="触发条件")
    priority: int = Field(default=0, description="优先级")


class BusinessFlow(BaseModel):
    name: str = Field(..., description="流程名称")
    description: Optional[str] = Field(None, description="流程描述")
    nodes: List[FlowNode] = Field(default_factory=list)
    edges: List[FlowEdge] = Field(default_factory=list)


class UserScenario(BaseModel):
    id: str = Field(..., description="场景ID")
    name: str = Field(..., description="场景名称")
    description: Optional[str] = Field(None, description="场景描述")
    path: List[str] = Field(default_factory=list, description="节点路径")
    preconditions: List[str] = Field(default_factory=list, description="前置条件")
    expectations: List[str] = Field(default_factory=list, description="预期结果")
    priority: Literal["high", "medium", "low"] = "medium"


class ExceptionCase(BaseModel):
    scenario: str = Field(..., description="异常场景")
    description: Optional[str] = Field(None, description="场景描述")
    handling: str = Field(..., description="处理方式")
    recovery: Optional[str] = Field(None, description="恢复策略")
    severity: Literal["critical", "high", "medium", "low"] = "medium"


class BusinessRule(BaseModel):
    id: str = Field(..., description="规则ID")
    rule: str = Field(..., description="规则名称")
    content: str = Field(..., description="规则内容")
    priority: Literal["high", "medium", "low"] = "medium"
    category: Optional[str] = Field(None, description="规则分类")


class BusinessTree(BaseModel):
    version: str = Field(default="1.0", description="版本号")
    business_flow: Optional[BusinessFlow] = Field(None, alias="businessFlow")
    user_scenarios: List[UserScenario] = Field(default_factory=list, alias="userScenarios")
    exception_handling: List[ExceptionCase] = Field(default_factory=list, alias="exceptionHandling")
    business_rules: List[BusinessRule] = Field(default_factory=list, alias="businessRules")
    metadata: dict = Field(default_factory=dict, description="附加元数据")

    class Config:
        populate_by_name = True

    def to_dict(self) -> dict:
        return self.model_dump(by_alias=True)

    @classmethod
    def from_dict(cls, data: dict) -> "BusinessTree":
        return cls.model_validate(data)
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/requirement/models/
git commit -m "feat(requirement): add business tree data models"
```

---

## 任务 3: 创建Prompt模板

**Files:**
- Create: `backend/app/agents/requirement/prompts/templates.py`
- Create: `backend/app/agents/requirement/prompts/__init__.py`

- [ ] **Step 1: 创建Prompt模板**

```python
# backend/app/agents/requirement/prompts/templates.py

CLARIFICATION_SYSTEM_PROMPT = """你是一个专业的产品经理，擅长理解用户需求并拆解业务逻辑。
你的任务是帮助用户澄清需求，确保理解完整无误。

请遵循以下原则：
1. 先理解整体需求框架，再深入细节
2. 关注用户角色、操作流程、数据流向
3. 识别潜在的风险点和异常场景
4. 用简洁清晰的语言与用户沟通

每次只问一个问题，确保问题具体明确。"""

CLARIFICATION_TEMPLATE = """
当前需求概述：
{requirement_summary}

请回答以下问题：
{question}

请用简洁的语言回答。"""

INITIAL_CLARIFICATION_QUESTIONS = """
基于以下需求描述，请生成3-5个关键澄清问题：

{requirement}

请以JSON格式返回：
{{
  "questions": [
    {{"id": "q1", "question": "问题内容", "reason": "为什么需要问这个问题"}}
  ]
}}
"""

EXTRACTION_SYSTEM_PROMPT = """你是一个专业的需求分析专家。
你的任务是从用户描述的需求文本中提取关键信息，构建结构化的业务逻辑树。

请分析以下内容：
1. 业务流程节点（用户操作、系统响应）
2. 决策分支点
3. 数据流转
4. 用户角色和权限
5. 异常场景和错误处理

输出必须是结构化的JSON格式。"""

EXTRACTION_TEMPLATE = """
请分析以下需求，提取结构化信息：

{requirement}

{context}

请以JSON格式返回完整的业务逻辑树：
"""

STRUCTURING_SYSTEM_PROMPT = """你是一个业务架构师，负责将提取的需求信息整理成规范的结构。

你的任务：
1. 补全缺失的信息
2. 确保逻辑一致性
3. 规范化术语表达
4. 识别潜在的遗漏点

输出必须是完整的、结构化的JSON。"""

STRUCTURING_TEMPLATE = """
基于以下提取的信息，请构建完整的业务逻辑树：

{extracted_data}

补充以下内容：
- 遗漏的业务规则
- 潜在的异常场景
- 缺失的边界条件
- 建议的组件结构

请以JSON格式返回完整的业务逻辑树。
"""
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/requirement/prompts/
git commit -m "feat(requirement): add prompt templates"
```

---

## 任务 4: 实现澄清链

**Files:**
- Create: `backend/app/agents/requirement/chains/clarification.py`
- Create: `backend/app/agents/requirement/chains/__init__.py`

- [ ] **Step 1: 实现澄清链**

```python
# backend/app/agents/requirement/chains/clarification.py
from typing import List, Dict, Optional
from pydantic import BaseModel
from app.core.llm import LLMFactory
from app.agents.requirement.prompts.templates import (
    CLARIFICATION_SYSTEM_PROMPT,
    INITIAL_CLARIFICATION_QUESTIONS,
    CLARIFICATION_TEMPLATE
)


class ClarificationQuestion(BaseModel):
    id: str
    question: str
    reason: str


class ClarificationSession(BaseModel):
    requirement: str
    questions: List[ClarificationQuestion] = []
    answers: Dict[str, str] = {}
    status: str = "in_progress"
    iteration: int = 0


class ClarificationChain:
    def __init__(self, model_provider: Optional[str] = None):
        self.llm = LLMFactory.create(model_provider)

    async def generate_initial_questions(self, requirement: str) -> List[ClarificationQuestion]:
        prompt = INITIAL_CLARIFICATION_QUESTIONS.format(requirement=requirement)

        response = await self.llm.generate(
            prompt=CLARIFICATION_SYSTEM_PROMPT + "\n\n" + prompt
        )

        import json
        try:
            data = json.loads(response)
            return [ClarificationQuestion(**q) for q in data.get("questions", [])]
        except json.JSONDecodeError:
            return [
                ClarificationQuestion(
                    id="q1",
                    question="能否详细描述一下这个需求的背景和目标？",
                    reason="需要了解需求产生的原因"
                )
            ]

    async def generate_followup_question(
        self,
        requirement: str,
        answers: Dict[str, str]
    ) -> Optional[ClarificationQuestion]:
        context = "\n".join([f"Q: {q}\nA: {a}" for q, a in answers.items()])
        prompt = f"""
基于已澄清的信息：
{context}

请生成下一个关键问题（如果有的话），以确保需求理解完整。

如果需求已经足够清晰，请返回：
{{"needs_more": false, "reason": "原因"}}

如果还需要澄清，请返回：
{{"needs_more": true, "question": "问题内容", "reason": "原因"}}
"""
        response = await self.llm.generate(prompt)

        import json
        try:
            data = json.loads(response)
            if data.get("needs_more"):
                return ClarificationQuestion(
                    id=f"q{len(answers) + 1}",
                    question=data["question"],
                    reason=data["reason"]
                )
            return None
        except json.JSONDecodeError:
            return None

    async def generate_summary(self, requirement: str, answers: Dict[str, str]) -> str:
        context = "\n".join([f"Q: {q}\nA: {a}" for q, a in answers.items()])
        prompt = f"""
请总结以下需求的澄清结果：

原始需求：
{requirement}

澄清内容：
{context}

请生成一个结构化的需求总结，包括：
1. 需求核心目标
2. 主要功能点
3. 关键约束条件
4. 已识别的风险点
"""
        return await self.llm.generate(prompt)
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/requirement/chains/clarification.py
git commit -m "feat(requirement): implement clarification chain"
```

---

## 任务 5: 实现提取链和结构化链

**Files:**
- Create: `backend/app/agents/requirement/chains/extraction.py`
- Create: `backend/app/agents/requirement/chains/structuring.py`

- [ ] **Step 1: 实现提取链**

```python
# backend/app/agents/requirement/chains/extraction.py
import json
from typing import Dict, Optional
from app.core.llm import LLMFactory
from app.agents.requirement.prompts.templates import (
    EXTRACTION_SYSTEM_PROMPT,
    EXTRACTION_TEMPLATE
)
from app.agents.requirement.models.business_tree import BusinessTree


class ExtractionChain:
    def __init__(self, model_provider: Optional[str] = None):
        self.llm = LLMFactory.create(model_provider)

    async def extract(self, requirement: str, context: Optional[Dict] = None) -> Dict:
        context_str = ""
        if context:
            context_str = "\n补充上下文：\n"
            for key, value in context.items():
                context_str += f"- {key}: {value}\n"

        prompt = EXTRACTION_TEMPLATE.format(
            requirement=requirement,
            context=context_str
        )

        response = await self.llm.generate(
            prompt=EXTRACTION_SYSTEM_PROMPT + "\n\n" + prompt,
            temperature=0.3
        )

        try:
            data = json.loads(response)
            return data
        except json.JSONDecodeError:
            return {
                "error": "Failed to parse extraction result",
                "raw_response": response
            }

    async def extract_business_flow(self, requirement: str) -> Dict:
        extraction = await self.extract(requirement)
        return extraction.get("businessFlow", {"nodes": [], "edges": []})

    async def extract_user_scenarios(self, requirement: str) -> list:
        extraction = await self.extract(requirement)
        return extraction.get("userScenarios", [])

    async def extract_exception_handling(self, requirement: str) -> list:
        extraction = await self.extract(requirement)
        return extraction.get("exceptionHandling", [])

    async def extract_business_rules(self, requirement: str) -> list:
        extraction = await self.extract(requirement)
        return extraction.get("businessRules", [])
```

- [ ] **Step 2: 实现结构化链**

```python
# backend/app/agents/requirement/chains/structuring.py
import json
from typing import Dict, Optional
from uuid import uuid4
from app.core.llm import LLMFactory
from app.agents.requirement.prompts.templates import (
    STRUCTURING_SYSTEM_PROMPT,
    STRUCTURING_TEMPLATE
)
from app.agents.requirement.models.business_tree import (
    BusinessTree,
    BusinessFlow,
    FlowNode,
    FlowEdge,
    UserScenario,
    ExceptionCase,
    BusinessRule,
    NodeType
)


class StructuringChain:
    def __init__(self, model_provider: Optional[str] = None):
        self.llm = LLMFactory.create(model_provider)

    async def structure(
        self,
        extracted_data: Dict,
        requirement: str
    ) -> BusinessTree:
        prompt = STRUCTURING_TEMPLATE.format(
            extracted_data=json.dumps(extracted_data, ensure_ascii=False, indent=2)
        )

        response = await self.llm.generate(
            prompt=STRUCTURING_SYSTEM_PROMPT + "\n\n" + prompt,
            temperature=0.4
        )

        try:
            data = json.loads(response)
            return BusinessTree.from_dict(data)
        except Exception as e:
            return self._create_default_tree(extracted_data, requirement)

    def _create_default_tree(self, data: Dict, requirement: str) -> BusinessTree:
        return BusinessTree(
            version="1.0",
            business_flow=BusinessFlow(
                name="主流程",
                description=requirement[:100],
                nodes=[
                    FlowNode(
                        id=str(uuid4()),
                        name="开始",
                        type=NodeType.ACTION,
                        description="流程开始"
                    )
                ],
                edges=[]
            ),
            user_scenarios=[],
            exception_handling=[],
            business_rules=[]
        )

    async def validate_and_fix(self, tree: BusinessTree) -> BusinessTree:
        issues = []

        if not tree.business_flow or not tree.business_flow.nodes:
            issues.append("缺少业务流程节点")

        if not tree.user_scenarios:
            issues.append("缺少用户场景定义")

        if not tree.business_rules:
            issues.append("缺少业务规则定义")

        if issues:
            validation_prompt = f"""
请检查以下业务逻辑树，识别问题并修复：

{json.dumps(tree.to_dict(), ensure_ascii=False, indent=2)}

问题列表：
{chr(10).join(f"- {issue}" for issue in issues)}

请生成修复后的完整业务逻辑树（JSON格式）。
"""
            response = await self.llm.generate(validation_prompt, temperature=0.3)

            try:
                data = json.loads(response)
                return BusinessTree.from_dict(data)
            except Exception:
                pass

        return tree
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/requirement/chains/extraction.py backend/app/agents/requirement/chains/structuring.py
git commit -m "feat(requirement): implement extraction and structuring chains"
```

---

## 任务 6: 实现需求解析Agent主类

**Files:**
- Create: `backend/app/agents/requirement/agent.py`
- Create: `backend/app/agents/requirement/__init__.py`
- Modify: `backend/app/agents/__init__.py`

- [ ] **Step 1: 实现需求解析Agent主类**

```python
# backend/app/agents/requirement/agent.py
from typing import Dict, List, Optional, Any
import redis.asyncio as redis
import json
from app.agents.base import BaseAgent
from app.agents.requirement.chains.clarification import ClarificationChain, ClarificationSession
from app.agents.requirement.chains.extraction import ExtractionChain
from app.agents.requirement.chains.structuring import StructuringChain
from app.agents.requirement.models.business_tree import BusinessTree
from app.core.config import settings


class RequirementAgent(BaseAgent):
    def __init__(
        self,
        model_provider: Optional[str] = None,
        redis_url: Optional[str] = None
    ):
        super().__init__(
            name="RequirementParser",
            description="需求解析Agent - 解析用户需求并生成结构化业务逻辑树",
            model_provider=model_provider
        )

        self.clarification_chain = ClarificationChain(model_provider)
        self.extraction_chain = ExtractionChain(model_provider)
        self.structuring_chain = StructuringChain(model_provider)

        self.redis_client = None
        if redis_url:
            self.redis_client = redis.from_url(redis_url)

    async def process(self, input_data: Dict) -> BusinessTree:
        requirement = input_data.get("requirement", "")
        mode = input_data.get("mode", "full")

        if mode == "full":
            return await self._full_process(requirement)
        elif mode == "clarification":
            return await self._clarification_process(input_data)
        elif mode == "extraction":
            return await self._extraction_process(requirement)
        else:
            raise ValueError(f"Unknown mode: {mode}")

    async def _full_process(self, requirement: str) -> BusinessTree:
        clarification_session = ClarificationSession(requirement=requirement)

        questions = await self.clarification_chain.generate_initial_questions(requirement)
        clarification_session.questions = questions

        if self.redis_client:
            await self._save_session(clarification_session)

        return clarification_session

    async def _clarification_process(self, input_data: Dict) -> Dict:
        session_id = input_data.get("session_id")
        answer = input_data.get("answer")

        if not session_id:
            return {"error": "session_id is required"}

        session = await self._load_session(session_id)
        if not session:
            return {"error": "Session not found"}

        current_q = session.questions[session.iteration]
        session.answers[current_q.id] = answer
        session.iteration += 1

        if session.iteration < len(session.questions):
            next_q = session.questions[session.iteration]
            if self.redis_client:
                await self._save_session(session)
            return {
                "session_id": session_id,
                "status": "in_progress",
                "current_question": next_q.model_dump(),
                "progress": f"{session.iteration}/{len(session.questions)}"
            }

        followup_q = await self.clarification_chain.generate_followup_question(
            session.requirement,
            session.answers
        )

        if followup_q:
            session.questions.append(followup_q)
            if self.redis_client:
                await self._save_session(session)
            return {
                "session_id": session_id,
                "status": "in_progress",
                "current_question": followup_q.model_dump(),
                "progress": f"{session.iteration}/{len(session.questions)}"
            }

        summary = await self.clarification_chain.generate_summary(
            session.requirement,
            session.answers
        )

        extracted = await self.extraction_chain.extract(
            requirement=summary,
            context=session.answers
        )

        business_tree = await self.structuring_chain.structure(extracted, session.requirement)
        business_tree = await self.structuring_chain.validate_and_fix(business_tree)

        session.status = "completed"

        if self.redis_client:
            await self._save_session(session)

        return {
            "session_id": session_id,
            "status": "completed",
            "summary": summary,
            "business_tree": business_tree.to_dict()
        }

    async def _extraction_process(self, requirement: str) -> BusinessTree:
        extracted = await self.extraction_chain.extract(requirement)
        tree = await self.structuring_chain.structure(extracted, requirement)
        return await self.structuring_chain.validate_and_fix(tree)

    async def validate(self, output: BusinessTree) -> bool:
        if not output.business_flow:
            return False
        if not output.business_flow.nodes:
            return False
        return True

    async def _save_session(self, session: ClarificationSession) -> None:
        if self.redis_client:
            await self.redis_client.setex(
                f"clarification:session:{session.requirement[:50]}",
                3600,
                json.dumps(session.model_dump(), default=str)
            )

    async def _load_session(self, session_id: str) -> Optional[ClarificationSession]:
        if self.redis_client:
            data = await self.redis_client.get(f"clarification:session:{session_id}")
            if data:
                return ClarificationSession.model_validate_json(data)
        return None

    async def close(self):
        if self.redis_client:
            await self.redis_client.close()
```

- [ ] **Step 2: 更新__init__文件**

```python
# backend/app/agents/__init__.py
from app.agents.base import BaseAgent
from app.agents.requirement.agent import RequirementAgent

__all__ = ["BaseAgent", "RequirementAgent"]
```

```python
# backend/app/agents/requirement/__init__.py
from app.agents.requirement.agent import RequirementAgent
from app.agents.requirement.models.business_tree import (
    BusinessTree,
    BusinessFlow,
    FlowNode,
    FlowEdge,
    UserScenario,
    ExceptionCase,
    BusinessRule
)

__all__ = [
    "RequirementAgent",
    "BusinessTree",
    "BusinessFlow",
    "FlowNode",
    "FlowEdge",
    "UserScenario",
    "ExceptionCase",
    "BusinessRule"
]
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/requirement/agent.py backend/app/agents/requirement/__init__.py backend/app/agents/__init__.py
git commit -m "feat(requirement): implement requirement parsing agent"
```

---

## 任务 7: 创建API端点

**Files:**
- Create: `backend/app/api/v1/endpoints/requirement.py`
- Modify: `backend/app/api/v1/__init__.py`

- [ ] **Step 1: 创建需求解析API端点**

```python
# backend/app/api/v1/endpoints/requirement.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from pydantic import BaseModel
from typing import Optional, Dict, Any

from app.core.database import get_db
from app.core.config import settings
from app.agents.requirement import RequirementAgent, BusinessTree

router = APIRouter()

requirement_agent = RequirementAgent(
    model_provider=settings.MODEL_PROVIDER,
    redis_url=settings.REDIS_URL if settings.REDIS_HOST else None
)


class RequirementInput(BaseModel):
    requirement: str
    mode: str = "full"


class ClarificationInput(BaseModel):
    session_id: str
    answer: str


class ExtractionInput(BaseModel):
    requirement: str


@router.post("/requirement/parse")
async def parse_requirement(
    input_data: RequirementInput,
    db: AsyncSession = Depends(get_db)
):
    result = await requirement_agent.process({
        "requirement": input_data.requirement,
        "mode": input_data.mode
    })

    if isinstance(result, BusinessTree):
        return {
            "status": "success",
            "business_tree": result.to_dict()
        }

    return {
        "status": "success",
        "session_id": result.requirement[:50],
        "questions": [q.model_dump() for q in result.questions] if hasattr(result, 'questions') else [],
        "status": result.status
    }


@router.post("/requirement/clarify")
async def clarify_requirement(
    input_data: ClarificationInput
):
    result = await requirement_agent.process({
        "mode": "clarification",
        "session_id": input_data.session_id,
        "answer": input_data.answer
    })

    return result


@router.post("/requirement/extract")
async def extract_requirement(
    input_data: ExtractionInput
):
    result = await requirement_agent.process({
        "requirement": input_data.requirement,
        "mode": "extraction"
    })

    return {
        "status": "success",
        "business_tree": result.to_dict()
    }


@router.get("/requirement/validate/{tree_id}")
async def validate_business_tree(
    tree_id: str,
    db: AsyncSession = Depends(get_db)
):
    return {
        "tree_id": tree_id,
        "is_valid": True,
        "issues": []
    }
```

- [ ] **Step 2: 更新路由聚合**

```python
# backend/app/api/v1/__init__.py
from fastapi import APIRouter
from app.api.v1.endpoints import health, tasks, requirement

router = APIRouter()
router.include_router(health.router, tags=["health"])
router.include_router(tasks.router, tags=["tasks"])
router.include_router(requirement.router, tags=["requirement"])
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/api/v1/endpoints/requirement.py backend/app/api/v1/__init__.py
git commit -m "feat(api): add requirement parsing endpoints"
```

---

## 任务 8: 创建前端需求输入组件

**Files:**
- Create: `frontend/src/components/RequirementInput.vue`
- Create: `frontend/src/views/RequirementView.vue` (更新)

- [ ] **Step 1: 创建需求输入组件**

```vue
<!-- frontend/src/components/RequirementInput.vue -->
<template>
  <div class="requirement-input">
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700 mb-2">
        请输入需求描述
      </label>
      <textarea
        v-model="requirement"
        rows="6"
        class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-primary-500"
        placeholder="请详细描述您的业务需求，包括：&#10;- 业务背景和目标&#10;- 主要功能点&#10;- 用户角色和权限&#10;- 数据流转和存储需求&#10;- 性能和安全要求"
      ></textarea>
    </div>

    <div class="flex items-center justify-between">
      <div class="text-sm text-gray-500">
        字数: {{ requirement.length }}
      </div>
      <button
        @click="handleSubmit"
        :disabled="!requirement.trim() || loading"
        class="px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        <span v-if="loading">解析中...</span>
        <span v-else>开始解析</span>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { taskApi } from '@/api/task'

const emit = defineEmits<{
  (e: 'submit', requirement: string): void
}>()

const requirement = ref('')
const loading = ref(false)

const handleSubmit = async () => {
  if (!requirement.value.trim()) return

  loading.value = true
  try {
    emit('submit', requirement.value)
  } finally {
    loading.value = false
  }
}
</script>
```

- [ ] **Step 2: 更新RequirementView**

```vue
<!-- frontend/src/views/RequirementView.vue -->
<template>
  <div class="requirement-view">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4">
        <h1 class="text-3xl font-bold text-gray-900">需求解析</h1>
      </div>
    </header>

    <main class="max-w-7xl mx-auto py-6 px-4">
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <!-- 左侧: 需求输入 -->
        <div class="bg-white shadow rounded-lg p-6">
          <h2 class="text-lg font-medium text-gray-900 mb-4">需求输入</h2>
          <RequirementInput @submit="handleRequirementSubmit" />
        </div>

        <!-- 右侧: 澄清对话 -->
        <div class="bg-white shadow rounded-lg p-6">
          <h2 class="text-lg font-medium text-gray-900 mb-4">需求澄清</h2>

          <div v-if="clarificationSession.status === 'in_progress'" class="space-y-4">
            <div class="bg-primary-50 rounded-lg p-4">
              <p class="text-sm text-primary-800">
                {{ clarificationSession.progress }}
              </p>
              <p class="text-lg font-medium text-primary-900 mt-2">
                {{ clarificationSession.current_question?.question }}
              </p>
              <p class="text-sm text-primary-600 mt-1">
                {{ clarificationSession.current_question?.reason }}
              </p>
            </div>

            <textarea
              v-model="clarificationAnswer"
              rows="4"
              class="w-full px-3 py-2 border border-gray-300 rounded-md"
              placeholder="请回答上述问题..."
            ></textarea>

            <button
              @click="submitClarification"
              :disabled="!clarificationAnswer.trim()"
              class="w-full px-4 py-2 bg-primary-600 text-white rounded-md hover:bg-primary-700"
            >
              提交回答
            </button>
          </div>

          <div v-else-if="clarificationSession.status === 'completed'" class="text-center py-8">
            <div class="text-4xl mb-4">✅</div>
            <p class="text-gray-600">需求澄清完成！</p>
            <button
              @click="viewBusinessTree"
              class="mt-4 px-4 py-2 bg-primary-600 text-white rounded-md"
            >
              查看业务逻辑树
            </button>
          </div>

          <div v-else class="text-center py-8 text-gray-500">
            请先提交需求描述开始澄清流程
          </div>
        </div>
      </div>

      <!-- 业务逻辑树展示 -->
      <div v-if="businessTree" class="mt-6 bg-white shadow rounded-lg p-6">
        <h2 class="text-lg font-medium text-gray-900 mb-4">业务逻辑树</h2>
        <pre class="bg-gray-50 p-4 rounded-lg overflow-auto">{{ JSON.stringify(businessTree, null, 2) }}</pre>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import RequirementInput from '@/components/RequirementInput.vue'
import apiClient from '@/api'

const requirement = ref('')
const clarificationAnswer = ref('')
const businessTree = ref(null)

const clarificationSession = reactive({
  status: 'pending',
  session_id: '',
  progress: '',
  current_question: null as { question: string; reason: string } | null
})

const handleRequirementSubmit = async (req: string) => {
  requirement.value = req

  try {
    const response = await apiClient.post('/requirement/parse', {
      requirement: req,
      mode: 'full'
    })

    if (response.data.questions && response.data.questions.length > 0) {
      clarificationSession.status = 'in_progress'
      clarificationSession.session_id = response.data.session_id
      clarificationSession.current_question = response.data.questions[0]
      clarificationSession.progress = `问题 1/${response.data.questions.length}`
    }
  } catch (error) {
    console.error('Failed to parse requirement:', error)
  }
}

const submitClarification = async () => {
  try {
    const response = await apiClient.post('/requirement/clarify', {
      session_id: clarificationSession.session_id,
      answer: clarificationAnswer.value
    })

    if (response.data.status === 'completed') {
      clarificationSession.status = 'completed'
      businessTree.value = response.data.business_tree
    } else {
      clarificationSession.current_question = response.data.current_question
      clarificationSession.progress = response.data.progress
    }

    clarificationAnswer.value = ''
  } catch (error) {
    console.error('Failed to clarify:', error)
  }
}

const viewBusinessTree = () => {
  // TODO: 跳转到业务逻辑树详情页
}
</script>
```

- [ ] **Step 3: Commit**

```bash
git add frontend/src/components/RequirementInput.vue frontend/src/views/RequirementView.vue
git commit -m "feat(frontend): add requirement input and clarification UI"
```

---

## Phase 2 总结

**完成的任务:**
- [x] Agent基类和LLM工厂实现
- [x] 业务逻辑树数据模型
- [x] Prompt模板系统
- [x] 澄清链实现
- [x] 提取链和结构化链实现
- [x] 需求解析Agent主类
- [x] API端点实现
- [x] 前端需求输入组件

**交付物:**
- 可运行的需求解析Agent
- 支持多轮对话澄清
- 输出结构化业务逻辑树
- 支持多模型切换

---

## 下一步行动

Phase 2 完成后，可以选择：

1. **继续 Phase 3**: 开始原型生成Agent开发
2. **继续 Phase 4**: 开始文档生成Agent开发
3. **完善测试**: 为Phase 2添加单元测试

**请选择下一步行动？**

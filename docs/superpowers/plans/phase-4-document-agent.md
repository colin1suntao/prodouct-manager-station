# Phase 4: 文档生成Agent - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现PRD模板引擎、Markdown生成、多格式导出功能，支持自动生成标准PRD文档

**Architecture:** 基于Jinja2模板引擎构建文档生成系统，支持Markdown/Word/PDF多格式输出

**Tech Stack:** Jinja2 | python-docx | pandoc/weasyprint | markdown

---

## 文件结构设计

```
backend/app/agents/document/
├── __init__.py
├── agent.py                  # 文档生成Agent主类
├── templates/                # PRD模板
│   ├── prd/
│   │   ├── base.md
│   │   ├── template_v1.md
│   │   └── sections/
│   │       ├── overview.md
│   │       ├── features.md
│   │       ├── flow.md
│   │       ├── pages.md
│   │       └── acceptance.md
├── generator.py             # 文档生成器
├── exporters/               # 导出器
│   ├── __init__.py
│   ├── markdown_exporter.py
│   ├── word_exporter.py
│   └── pdf_exporter.py
└── models/
    ├── __init__.py
    └── prd.py              # PRD文档模型
```

---

## 任务 1: 创建PRD文档数据模型

**Files:**
- Create: `backend/app/agents/document/models/prd.py`
- Create: `backend/app/agents/document/models/__init__.py`

- [ ] **Step 1: 创建PRD文档模型**

```python
# backend/app/agents/document/models/prd.py
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any
from datetime import datetime
from enum import Enum


class Priority(str, Enum):
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"


class UserStory(BaseModel):
    as_a: str = Field(alias="as")
    i_want: str = Field(alias="want")
    so_that: str = Field(alias="so_that")
    acceptance_criteria: List[str] = Field(default_factory=list, alias="acceptanceCriteria")

    class Config:
        populate_by_name = True


class Feature(BaseModel):
    id: str
    name: str
    description: str
    priority: Priority = Priority.MEDIUM
    dependencies: List[str] = Field(default_factory=list)
    mockups: List[str] = Field(default_factory=list)


class NonFunctionalRequirement(BaseModel):
    category: str
    requirement: str
    metric: Optional[str] = None


class PageReference(BaseModel):
    name: str
    path: str
    description: str
    interactions: List[str] = Field(default_factory=list)


class PRDDocument(BaseModel):
    version: str = "1.0"
    title: str
    product_name: str
    author: str
    created_at: datetime = Field(default_factory=datetime.now)
    updated_at: Optional[datetime] = None

    overview: Dict[str, Any] = Field(default_factory=dict)
    user_stories: List[UserStory] = Field(default_factory=list, alias="userStories")
    features: List[Feature] = Field(default_factory=list)
    business_flow: Optional[str] = Field(None, alias="businessFlow")
    pages: List[PageReference] = Field(default_factory=list)
    non_functional: List[NonFunctionalRequirement] = Field(default_factory=list, alias="nonFunctional")
    exception_handling: List[Dict[str, str]] = Field(default_factory=list, alias="exceptionHandling")
    acceptance_criteria: List[str] = Field(default_factory=list, alias="acceptanceCriteria")

    class Config:
        populate_by_name = True

    def to_markdown(self) -> str:
        pass

    def to_dict(self) -> dict:
        return self.model_dump(by_alias=True)
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/document/models/
git commit -m "feat(document): add PRD document models"
```

---

## 任务 2: 创建PRD模板

**Files:**
- Create: `backend/app/agents/document/templates/prd/template_v1.md`
- Create: `backend/app/agents/document/templates/prd/sections/overview.md`
- Create: `backend/app/agents/document/templates/prd/sections/features.md`
- Create: `backend/app/agents/document/templates/prd/sections/flow.md`
- Create: `backend/app/agents/document/templates/prd/sections/pages.md`
- Create: `backend/app/agents/document/templates/prd/sections/acceptance.md`

- [ ] **Step 1: 创建PRD主模板**

```markdown
{# backend/app/agents/document/templates/prd/template_v1.md #}
# {{ title }}

**产品名称：** {{ product_name }}
**文档版本：** {{ version }}
**作者：** {{ author }}
**创建日期：** {{ created_at.strftime('%Y-%m-%d') }}
{% if updated_at %}
**更新日期：** {{ updated_at.strftime('%Y-%m-%d') }}
{% endif %}

---

## 1. 概述

### 1.1 背景
{{ overview.background | default('待补充') }}

### 1.2 目标
{{ overview.goals | default('待补充') }}

### 1.3 范围
{{ overview.scope | default('待补充') }}

### 1.4 定义与缩写
| 术语 | 定义 |
|------|------|
{% for term in overview.terms | default([]) %}
| {{ term.name }} | {{ term.definition }} |
{% endfor %}

---

## 2. 用户故事

{% for story in user_stories %}
### 2.{{ loop.index }} {{ story.as_a }}，{{ story.i_want }}
**作为** {{ story.as_a }}
**我希望** {{ story.i_want }}
**以便于** {{ story.so_that }}

**验收标准：**
{% for criteria in story.acceptance_criteria %}
- [ ] {{ criteria }}
{% endfor %}

---
{% endfor %}

---

## 3. 功能需求

{% for feature in features %}
### 3.{{ loop.index }}. {{ feature.name }}
**功能ID：** {{ feature.id }}
**优先级：** {{ feature.priority.value }}
**描述：** {{ feature.description }}

{% if feature.dependencies %}
**依赖功能：** {{ feature.dependencies | join(', ') }}
{% endif %}

{% if feature.mockups %}
**相关页面：** {{ feature.mockups | join(', ') }}
{% endif %}

{% endfor %}

---

## 4. 业务流程

{{ business_flow | default('待补充业务流程图和说明') }}

### 4.1 流程说明
{% for scenario in user_stories %}
- {{ scenario.name | default('场景 ' + loop.index) }}：{{ scenario.description | default('') }}
{% endfor %}

---

## 5. 页面原型

{% for page in pages %}
### 5.{{ loop.index }} {{ page.name }}
**页面路径：** `{{ page.path }}`
**页面说明：** {{ page.description }}

{% if page.interactions %}
**交互说明：**
{% for interaction in page.interactions %}
- {{ interaction }}
{% endfor %}
{% endif %}

{% endfor %}

---

## 6. 非功能需求

{% for req in non_functional %}
### 6.{{ loop.index }} {{ req.category }}
{{ req.requirement }}

{% if req.metric %}
**衡量标准：** {{ req.metric }}
{% endif %}

{% endfor %}

---

## 7. 异常处理

| 异常场景 | 处理方式 | 恢复策略 |
|----------|----------|----------|
{% for exception in exception_handling %}
| {{ exception.scenario }} | {{ exception.handling }} | {{ exception.recovery | default('N/A') }} |
{% endfor %}

---

## 8. 验收标准

{% for criteria in acceptance_criteria %}
- [ ] {{ criteria }}
{% endfor %}

---

## 附录

### A. 修改记录

| 日期 | 版本 | 修改内容 | 作者 |
|------|------|----------|------|
| {{ created_at.strftime('%Y-%m-%d') }} | {{ version }} | 初始版本 | {{ author }} |

---

*本文档由产品经理工作站自动生成*
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/document/templates/
git commit -m "feat(document): add PRD templates"
```

---

## 任务 3: 实现文档生成器

**Files:**
- Create: `backend/app/agents/document/generator.py`

- [ ] **Step 1: 创建文档生成器**

```python
# backend/app/agents/document/generator.py
from typing import Optional
from pathlib import Path
from jinja2 import Environment, FileSystemLoader
from app.agents.document.models.prd import PRDDocument
from app.agents.requirement.models.business_tree import BusinessTree


class DocumentGenerator:
    def __init__(self, template_dir: Optional[str] = None):
        if template_dir:
            self.env = Environment(loader=FileSystemLoader(template_dir))
        else:
            base_dir = Path(__file__).parent
            template_path = base_dir / "templates"
            self.env = Environment(loader=str(template_path))

        self.env.globals.update({
            'default': self._default_filter,
            'join': self._join_filter
        })

    def _default_filter(self, value, default=''):
        return value if value is not None else default

    def _join_filter(self, value, sep=', '):
        return sep.join(str(v) for v in value) if value else ''

    def generate_from_business_tree(
        self,
        business_tree: BusinessTree,
        product_name: str,
        author: str = "System"
    ) -> PRDDocument:
        prd = PRDDocument(
            title=f"{product_name}产品需求文档",
            product_name=product_name,
            author=author
        )

        if business_tree.business_flow:
            prd.business_flow = self._generate_flow_description(
                business_tree.business_flow
            )

        if business_tree.user_scenarios:
            prd.user_stories = self._generate_user_stories(
                business_tree.user_scenarios
            )

        if business_tree.business_rules:
            prd.features = self._generate_features(
                business_tree.business_rules
            )

        if business_tree.exception_handling:
            prd.exception_handling = [
                {
                    "scenario": exc.scenario,
                    "handling": exc.handling,
                    "recovery": exc.recovery
                }
                for exc in business_tree.exception_handling
            ]

        return prd

    def _generate_flow_description(self, flow) -> str:
        if not flow or not flow.nodes:
            return "暂无业务流程定义"

        lines = [f"**流程名称：** {flow.name}"]
        if flow.description:
            lines.append(f"**流程说明：** {flow.description}")

        lines.append("\n**节点列表：**")
        for node in flow.nodes:
            lines.append(f"- {node.name}（{node.type.value}）")

        return "\n".join(lines)

    def _generate_user_stories(self, scenarios) -> list:
        stories = []
        for scenario in scenarios:
            story = {
                "as": "用户",
                "want": scenario.name,
                "so_that": "满足业务需求",
                "acceptanceCriteria": scenario.expectations or []
            }
            stories.append(story)
        return stories

    def _generate_features(self, rules) -> list:
        features = []
        for idx, rule in enumerate(rules, 1):
            feature = {
                "id": f"F{idx:03d}",
                "name": rule.rule,
                "description": rule.content,
                "priority": rule.priority.value if hasattr(rule, 'priority') else "medium"
            }
            features.append(feature)
        return features

    def render_markdown(self, prd: PRDDocument, template_name: str = "prd/template_v1.md") -> str:
        template = self.env.get_template(template_name)
        return template.render(**prd.to_dict())

    def save_markdown(self, content: str, output_path: str) -> str:
        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)
        path.write_text(content, encoding='utf-8')
        return str(path)
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/document/generator.py
git commit -m "feat(document): implement document generator"
```

---

## 任务 4: 实现导出器

**Files:**
- Create: `backend/app/agents/document/exporters/markdown_exporter.py`
- Create: `backend/app/agents/document/exporters/word_exporter.py`
- Create: `backend/app/agents/document/exporters/pdf_exporter.py`
- Create: `backend/app/agents/document/exporters/__init__.py`

- [ ] **Step 1: 创建Markdown导出器**

```python
# backend/app/agents/document/exporters/markdown_exporter.py
from typing import Optional
from pathlib import Path
from app.agents.document.models.prd import PRDDocument
from app.agents.document.generator import DocumentGenerator


class MarkdownExporter:
    def __init__(self):
        self.generator = DocumentGenerator()

    def export(self, prd: PRDDocument, output_path: Optional[str] = None) -> dict:
        content = self.generator.render_markdown(prd)

        if output_path:
            saved_path = self.generator.save_markdown(content, output_path)
            return {
                "status": "success",
                "format": "markdown",
                "path": saved_path,
                "content": content
            }

        return {
            "status": "success",
            "format": "markdown",
            "content": content
        }

    def export_with_toc(self, prd: PRDDocument) -> str:
        content = self.generator.render_markdown(prd)

        toc = []
        lines = content.split('\n')
        for line in lines:
            if line.startswith('## '):
                anchor = line.replace('## ', '').strip()
                anchor_id = anchor.lower().replace(' ', '-')
                toc.append(f"- [{anchor}](#{anchor_id})")
            elif line.startswith('### '):
                anchor = line.replace('### ', '').strip()
                anchor_id = anchor.lower().replace(' ', '-')
                toc.append(f"  - [{anchor}](#{anchor_id})")

        toc_content = "# 目录\n\n" + "\n".join(toc) + "\n\n---\n\n"
        return toc_content + content
```

- [ ] **Step 2: 创建Word导出器**

```python
# backend/app/agents/document/exporters/word_exporter.py
from typing import Optional
from pathlib import Path
import subprocess
import tempfile
from app.agents.document.models.prd import PRDDocument
from app.agents.document.generator import DocumentGenerator


class WordExporter:
    def __init__(self):
        self.generator = DocumentGenerator()

    def export(self, prd: PRDDocument, output_path: Optional[str] = None) -> dict:
        markdown_content = self.generator.render_markdown(prd)

        if not output_path:
            output_path = f"./output/documents/{prd.product_name}.docx"

        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)

        md_path = path.with_suffix('.md')
        md_path.write_text(markdown_content, encoding='utf-8')

        try:
            subprocess.run(
                ['pandoc', str(md_path), '-o', str(path)],
                check=True,
                capture_output=True
            )
            md_path.unlink()

            return {
                "status": "success",
                "format": "word",
                "path": str(path)
            }
        except subprocess.CalledProcessError as e:
            return {
                "status": "error",
                "message": f"Pandoc conversion failed: {e.stderr.decode() if e.stderr else str(e)}"
            }
        except FileNotFoundError:
            return {
                "status": "error",
                "message": "Pandoc not found. Please install pandoc first."
            }
```

- [ ] **Step 3: 创建PDF导出器**

```python
# backend/app/agents/document/exporters/pdf_exporter.py
from typing import Optional
from pathlib import Path
import subprocess
from app.agents.document.models.prd import PRDDocument
from app.agents.document.generator import DocumentGenerator


class PDFExporter:
    def __init__(self):
        self.generator = DocumentGenerator()

    def export(self, prd: PRDDocument, output_path: Optional[str] = None) -> dict:
        markdown_content = self.generator.render_markdown(prd)

        if not output_path:
            output_path = f"./output/documents/{prd.product_name}.pdf"

        path = Path(output_path)
        path.parent.mkdir(parents=True, exist_ok=True)

        md_path = path.with_suffix('.md')
        md_path.write_text(markdown_content, encoding='utf-8')

        try:
            subprocess.run(
                ['pandoc', str(md_path), '-o', str(path), '--pdf-engine=weasyprint'],
                check=True,
                capture_output=True
            )
            md_path.unlink()

            return {
                "status": "success",
                "format": "pdf",
                "path": str(path)
            }
        except subprocess.CalledProcessError as e:
            return {
                "status": "error",
                "message": f"Pandoc conversion failed: {e.stderr.decode() if e.stderr else str(e)}"
            }
```

- [ ] **Step 4: Commit**

```bash
git add backend/app/agents/document/exporters/
git commit -m "feat(document): implement document exporters"
```

---

## 任务 5: 实现文档生成Agent

**Files:**
- Create: `backend/app/agents/document/agent.py`
- Create: `backend/app/agents/document/__init__.py`

- [ ] **Step 1: 实现文档生成Agent**

```python
# backend/app/agents/document/agent.py
from typing import Dict, Any, Optional, List
from app.agents.base import BaseAgent
from app.agents.document.generator import DocumentGenerator
from app.agents.document.exporters.markdown_exporter import MarkdownExporter
from app.agents.document.exporters.word_exporter import WordExporter
from app.agents.document.exporters.pdf_exporter import PDFExporter
from app.agents.document.models.prd import PRDDocument
from app.agents.requirement.models.business_tree import BusinessTree


class DocumentAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            name="DocumentGenerator",
            description="文档生成Agent - 基于业务逻辑树生成标准PRD文档"
        )

        self.generator = DocumentGenerator()
        self.exporters = {
            "markdown": MarkdownExporter(),
            "word": WordExporter(),
            "pdf": PDFExporter()
        }

    async def process(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        business_tree = input_data.get("business_tree")
        product_name = input_data.get("product_name", "Product")
        author = input_data.get("author", "System")
        export_formats = input_data.get("formats", ["markdown"])

        if isinstance(business_tree, dict):
            business_tree = BusinessTree.from_dict(business_tree)

        prd = self.generator.generate_from_business_tree(
            business_tree,
            product_name,
            author
        )

        results = {
            "prd": prd.to_dict(),
            "exports": {}
        }

        for format_type in export_formats:
            if format_type in self.exporters:
                exporter = self.exporters[format_type]
                output_path = f"./output/documents/{product_name}.{format_type}"
                export_result = exporter.export(prd, output_path)
                results["exports"][format_type] = export_result

        return results

    async def generate_markdown(self, prd: PRDDocument) -> str:
        return self.generator.render_markdown(prd)

    async def export_document(
        self,
        prd: PRDDocument,
        formats: List[str]
    ) -> Dict[str, Any]:
        results = {}

        for format_type in formats:
            if format_type in self.exporters:
                exporter = self.exporters[format_type]
                result = exporter.export(prd)
                results[format_type] = result

        return results

    async def validate(self, output: Dict) -> bool:
        return "prd" in output and output["prd"] is not None
```

- [ ] **Step 2: 更新__init__**

```python
# backend/app/agents/document/__init__.py
from app.agents.document.agent import DocumentAgent
from app.agents.document.generator import DocumentGenerator
from app.agents.document.models.prd import PRDDocument

__all__ = ["DocumentAgent", "DocumentGenerator", "PRDDocument"]
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/document/agent.py backend/app/agents/document/__init__.py
git commit -m "feat(document): implement document generation agent"
```

---

## Phase 4 总结

**完成的任务:**
- [x] PRD文档数据模型定义
- [x] PRD模板系统
- [x] 文档生成器实现
- [x] Markdown导出器
- [x] Word导出器
- [x] PDF导出器
- [x] 文档生成Agent

**交付物:**
- 完整的PRD模板系统
- 支持Markdown/Word/PDF多格式导出
- 可定制的文档生成器
- 自动生成标准PRD文档

---

## 下一步行动

Phase 4 完成后，请继续 Phase 5: 校验Agent

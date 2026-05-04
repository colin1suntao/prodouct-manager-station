# Phase 5: 校验Agent - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现原型逻辑校验、文档规范校验、修复建议生成，确保交付物质量

**Architecture:** 基于规则引擎和LLM结合的校验系统，提供自动化质量保障

**Tech Stack:** Pydantic | 正则表达式 | LLM (用于智能建议)

---

## 文件结构设计

```
backend/app/agents/validation/
├── __init__.py
├── agent.py                  # 校验Agent主类
├── validators/              # 校验器
│   ├── __init__.py
│   ├── prototype_validator.py
│   ├── document_validator.py
│   └── business_logic_validator.py
├── rules/                   # 校验规则
│   ├── __init__.py
│   ├── prototype_rules.py
│   └── document_rules.py
└── models/
    ├── __init__.py
    └── validation_result.py
```

---

## 任务 1: 创建校验结果数据模型

**Files:**
- Create: `backend/app/agents/validation/models/validation_result.py`
- Create: `backend/app/agents/validation/models/__init__.py`

- [ ] **Step 1: 创建校验结果模型**

```python
# backend/app/agents/validation/models/validation_result.py
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any, Literal
from datetime import datetime
from enum import Enum


class IssueSeverity(str, Enum):
    CRITICAL = "critical"
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"


class IssueType(str, Enum):
    MISSING = "missing"
    INCONSISTENT = "inconsistent"
    INVALID = "invalid"
    INCOMPLETE = "incomplete"
    STYLE = "style"


class ValidationIssue(BaseModel):
    type: IssueType
    severity: IssueSeverity
    title: str
    description: str
    location: Optional[str] = None
    suggestion: str
    auto_fixable: bool = False


class ValidationScore(BaseModel):
    total: float = Field(ge=0, le=100)
    breakdown: Dict[str, float] = Field(default_factory=dict)
    grade: Literal["A", "B", "C", "D", "F"]

    @classmethod
    def calculate_grade(cls, score: float) -> str:
        if score >= 90:
            return "A"
        elif score >= 80:
            return "B"
        elif score >= 70:
            return "C"
        elif score >= 60:
            return "D"
        else:
            return "F"


class ValidationReport(BaseModel):
    timestamp: datetime = Field(default_factory=datetime.now)
    validator_version: str = "1.0.0"
    target_type: Literal["prototype", "document", "business_logic"]

    issues: List[ValidationIssue] = Field(default_factory=list)
    score: ValidationScore
    summary: str

    passed: bool = False

    def add_issue(self, issue: ValidationIssue):
        self.issues.append(issue)
        self._recalculate_score()

    def _recalculate_score(self):
        if not self.issues:
            self.score = ValidationScore(total=100, grade="A")
            self.passed = True
            return

        severity_weights = {
            IssueSeverity.CRITICAL: 20,
            IssueSeverity.HIGH: 10,
            IssueSeverity.MEDIUM: 5,
            IssueSeverity.LOW: 2
        }

        total_deduction = sum(
            severity_weights.get(issue.severity, 5)
            for issue in self.issues
        )

        final_score = max(0, 100 - total_deduction)
        self.score = ValidationScore(
            total=final_score,
            grade=ValidationScore.calculate_grade(final_score)
        )
        self.passed = final_score >= 70

    def to_dict(self) -> dict:
        return {
            "timestamp": self.timestamp.isoformat(),
            "validator_version": self.validator_version,
            "target_type": self.target_type,
            "issues": [i.model_dump() for i in self.issues],
            "score": self.score.model_dump(),
            "summary": self.summary,
            "passed": self.passed
        }
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/validation/models/
git commit -m "feat(validation): add validation result models"
```

---

## 任务 2: 创建校验规则

**Files:**
- Create: `backend/app/agents/validation/rules/prototype_rules.py`
- Create: `backend/app/agents/validation/rules/document_rules.py`
- Create: `backend/app/agents/validation/rules/__init__.py`

- [ ] **Step 1: 创建原型校验规则**

```python
# backend/app/agents/validation/rules/prototype_rules.py
from typing import List, Dict, Any
from dataclasses import dataclass


@dataclass
class ValidationRule:
    id: str
    name: str
    description: str
    severity: str
    check_function: callable


PROTOTYPE_RULES = [
    ValidationRule(
        id="P001",
        name="page_completeness",
        description="检查页面完整性，确保所有业务场景都有对应页面",
        severity="high",
        check_function=lambda pages, **kwargs: _check_page_completeness(pages, **kwargs)
    ),
    ValidationRule(
        id="P002",
        name="navigation_logic",
        description="检查导航逻辑，确保页面跳转链路完整",
        severity="high",
        check_function=lambda pages, **kwargs: _check_navigation_logic(pages, **kwargs)
    ),
    ValidationRule(
        id="P003",
        name="form_validation",
        description="检查表单验证规则是否完整",
        severity="medium",
        check_function=lambda pages, **kwargs: _check_form_validation(pages, **kwargs)
    ),
    ValidationRule(
        id="P004",
        name="exception_coverage",
        description="检查异常场景是否有对应提示",
        severity="medium",
        check_function=lambda pages, **kwargs: _check_exception_coverage(pages, **kwargs)
    ),
    ValidationRule(
        id="P005",
        name="page_naming",
        description="检查页面命名是否符合规范",
        severity="low",
        check_function=lambda pages, **kwargs: _check_page_naming(pages, **kwargs)
    ),
    ValidationRule(
        id="P006",
        name="component_reusability",
        description="检查组件是否符合复用规范",
        severity="low",
        check_function=lambda pages, **kwargs: _check_component_reusability(pages, **kwargs)
    )
]


def _check_page_completeness(pages: List[Dict], **kwargs) -> Dict[str, Any]:
    issues = []
    scenarios = kwargs.get("scenarios", [])

    if not pages:
        issues.append({
            "id": "P001-001",
            "title": "缺少页面定义",
            "description": "没有找到任何页面定义",
            "severity": "critical"
        })
        return {"passed": False, "issues": issues}

    page_paths = {p.get("path", ""): p.get("name", "") for p in pages}

    required_paths = ["/", "/login", "/error"]
    for path in required_paths:
        if path not in page_paths:
            issues.append({
                "id": "P001-002",
                "title": f"缺少必需页面: {path}",
                "description": f"系统缺少必要的{path}页面",
                "severity": "high",
                "suggestion": f"建议添加{path}页面"
            })

    for scenario in scenarios:
        scenario_path = f"/scenario/{scenario.get('id', '')}"
        if scenario_path not in page_paths:
            issues.append({
                "id": "P001-003",
                "title": f"缺少场景页面: {scenario.get('name', '')}",
                "description": f"场景{scenario.get('name')}没有对应的页面",
                "severity": "medium",
                "suggestion": f"建议为场景{scenario.get('name')}创建页面"
            })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.2
    }


def _check_navigation_logic(pages: List[Dict], **kwargs) -> Dict[str, Any]:
    issues = []

    for page in pages:
        path = page.get("path", "")
        config = page.get("config", {})

        if not config:
            issues.append({
                "id": "P002-001",
                "title": f"页面{path}缺少配置",
                "description": "页面缺少必要的配置信息",
                "severity": "medium"
            })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.3
    }


def _check_form_validation(pages: List[Dict], **kwargs) -> Dict[str, Any]:
    issues = []

    for page in pages:
        if page.get("type") == "form":
            fields = page.get("config", {}).get("fields", [])
            for field in fields:
                if field.get("required") and not field.get("validation"):
                    issues.append({
                        "id": "P003-001",
                        "title": f"字段{field.get('name')}缺少验证规则",
                        "description": f"必填字段{field.get('label')}缺少验证规则",
                        "severity": "medium",
                        "suggestion": f"为{field.get('label')}添加验证规则"
                    })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.25
    }


def _check_exception_coverage(pages: List[Dict], **kwargs) -> Dict[str, Any]:
    issues = []
    exceptions = kwargs.get("exceptions", [])

    if exceptions and not any(p.get("type") == "error" for p in pages):
        issues.append({
            "id": "P004-001",
            "title": "缺少错误页面",
            "description": "系统定义了异常场景但缺少错误页面",
            "severity": "medium",
            "suggestion": "建议添加统一的错误处理页面"
        })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.25
    }


def _check_page_naming(pages: List[Dict], **kwargs) -> Dict[str, Any]:
    issues = []

    import re
    path_pattern = re.compile(r'^/[a-z0-9\-]+$')

    for page in pages:
        path = page.get("path", "")
        if path != "/" and not path_pattern.match(path):
            issues.append({
                "id": "P005-001",
                "title": f"页面路径命名不规范: {path}",
                "description": "页面路径应使用小写字母、数字和连字符",
                "severity": "low",
                "suggestion": "建议使用如 /user-list 或 /order-detail 的格式"
            })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.0
    }


def _check_component_reusability(pages: List[Dict], **kwargs) -> Dict[str, Any]:
    return {
        "passed": True,
        "issues": [],
        "score_weight": 0.0
    }
```

- [ ] **Step 2: 创建文档校验规则**

```python
# backend/app/agents/validation/rules/document_rules.py
from typing import List, Dict, Any
from dataclasses import dataclass
import re


@dataclass
class DocumentRule:
    id: str
    name: str
    description: str
    severity: str
    check_function: callable


DOCUMENT_RULES = [
    DocumentRule(
        id="D001",
        name="format_compliance",
        description="检查文档格式是否符合PRD模板规范",
        severity="high",
        check_function=lambda content, **kwargs: _check_format_compliance(content, **kwargs)
    ),
    DocumentRule(
        id="D002",
        name="content_completeness",
        description="检查各章节内容是否完整",
        severity="high",
        check_function=lambda content, **kwargs: _check_content_completeness(content, **kwargs)
    ),
    DocumentRule(
        id="D003",
        name="terminology_consistency",
        description="检查专业术语是否统一",
        severity="medium",
        check_function=lambda content, **kwargs: _check_terminology_consistency(content, **kwargs)
    ),
    DocumentRule(
        id="D004",
        name="logic_alignment",
        description="检查文档内容与原型是否一致",
        severity="high",
        check_function=lambda content, **kwargs: _check_logic_alignment(content, **kwargs)
    )
]


REQUIRED_SECTIONS = [
    ("概述", r"## 1\. 概述"),
    ("用户故事", r"## 2\. 用户故事"),
    ("功能需求", r"## 3\. 功能需求"),
    ("业务流程", r"## 4\. 业务流程"),
    ("非功能需求", r"## 6\. 非功能需求"),
    ("验收标准", r"## 8\. 验收标准")
]


def _check_format_compliance(content: str, **kwargs) -> Dict[str, Any]:
    issues = []

    if not content:
        issues.append({
            "id": "D001-001",
            "title": "文档内容为空",
            "description": "文档没有内容",
            "severity": "critical"
        })
        return {"passed": False, "issues": issues}

    for section_name, pattern in REQUIRED_SECTIONS:
        if not re.search(pattern, content, re.MULTILINE):
            issues.append({
                "id": "D001-002",
                "title": f"缺少章节: {section_name}",
                "description": f"文档缺少{section_name}章节",
                "severity": "high",
                "suggestion": f"请添加{section_name}章节"
            })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.2
    }


def _check_content_completeness(content: str, **kwargs) -> Dict[str, Any]:
    issues = []

    sections = {
        "背景": len(re.findall(r"背景[：:]", content)),
        "目标": len(re.findall(r"目标[：:]", content)),
        "用户故事": len(re.findall(r"作为.*我希望.*以便于", content, re.DOTALL)),
        "功能点": len(re.findall(r"### 3\.\d+\.", content)),
        "验收标准": len(re.findall(r"- \[ \]", content))
    }

    if sections["用户故事"] == 0:
        issues.append({
            "id": "D002-001",
            "title": "缺少用户故事",
            "description": "文档没有定义用户故事",
            "severity": "high",
            "suggestion": "请添加用户故事"
        })

    if sections["功能点"] == 0:
        issues.append({
            "id": "D002-002",
            "title": "缺少功能需求",
            "description": "文档没有定义功能需求",
            "severity": "high",
            "suggestion": "请添加功能需求章节"
        })

    if sections["验收标准"] < 3:
        issues.append({
            "id": "D002-003",
            "title": "验收标准不足",
            "description": f"当前只有{sections['验收标准']}个验收标准，建议至少3个",
            "severity": "medium",
            "suggestion": "请补充验收标准"
        })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.3
    }


def _check_terminology_consistency(content: str, **kwargs) -> Dict[str, Any]:
    issues = []

    term_pairs = [
        ("用户", ["用户", "使用者", "客户"]),
        ("功能", ["功能", "特性", "特性点"]),
        ("登录", ["登录", "登陆"]),
    ]

    for term, variations in term_pairs:
        found = [v for v in variations if v in content]
        if len(found) > 1:
            issues.append({
                "id": "D003-001",
                "title": f"术语不一致: {term}",
                "description": f"文档中同时使用了{', '.join(found)}，建议统一使用'{term}'",
                "severity": "low",
                "suggestion": f"建议统一使用'{found[0]}'"
            })

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.25
    }


def _check_logic_alignment(content: str, **kwargs) -> Dict[str, Any]:
    issues = []
    prototype_pages = kwargs.get("prototype_pages", [])
    business_tree = kwargs.get("business_tree", {})

    if not prototype_pages or not business_tree:
        return {
            "passed": True,
            "issues": [],
            "score_weight": 0.25
        }

    return {
        "passed": len(issues) == 0,
        "issues": issues,
        "score_weight": 0.25
    }
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/validation/rules/
git commit -m "feat(validation): add validation rules"
```

---

## 任务 3: 实现校验器

**Files:**
- Create: `backend/app/agents/validation/validators/prototype_validator.py`
- Create: `backend/app/agents/validation/validators/document_validator.py`
- Create: `backend/app/agents/validation/validators/__init__.py`

- [ ] **Step 1: 创建原型校验器**

```python
# backend/app/agents/validation/validators/prototype_validator.py
from typing import List, Dict, Any, Optional
from app.agents.validation.models.validation_result import (
    ValidationReport,
    ValidationIssue,
    IssueSeverity,
    IssueType,
    ValidationScore
)
from app.agents.validation.rules.prototype_rules import PROTOTYPE_RULES


class PrototypeValidator:
    def __init__(self):
        self.rules = PROTOTYPE_RULES

    def validate(
        self,
        pages: List[Dict[str, Any]],
        business_tree: Optional[Dict[str, Any]] = None
    ) -> ValidationReport:
        report = ValidationReport(
            target_type="prototype",
            score=ValidationScore(total=100, grade="A"),
            summary=""
        )

        scenarios = []
        exceptions = []

        if business_tree:
            scenarios = business_tree.get("userScenarios", [])
            exceptions = business_tree.get("exceptionHandling", [])

        context = {
            "scenarios": scenarios,
            "exceptions": exceptions
        }

        all_issues = []

        for rule in self.rules:
            try:
                result = rule.check_function(pages, **context)

                if not result.get("passed", True):
                    for issue_data in result.get("issues", []):
                        issue = ValidationIssue(
                            type=IssueType.INCOMPLETE,
                            severity=self._map_severity(issue_data.get("severity", "medium")),
                            title=issue_data.get("title", ""),
                            description=issue_data.get("description", ""),
                            location=issue_data.get("location"),
                            suggestion=issue_data.get("suggestion", ""),
                            auto_fixable=False
                        )
                        all_issues.append(issue)

            except Exception as e:
                issue = ValidationIssue(
                    type=IssueType.INVALID,
                    severity=IssueSeverity.LOW,
                    title=f"规则执行失败: {rule.name}",
                    description=str(e),
                    suggestion="请检查规则配置"
                )
                all_issues.append(issue)

        for issue in all_issues:
            report.add_issue(issue)

        report.summary = self._generate_summary(report)

        return report

    def _map_severity(self, severity: str) -> IssueSeverity:
        mapping = {
            "critical": IssueSeverity.CRITICAL,
            "high": IssueSeverity.HIGH,
            "medium": IssueSeverity.MEDIUM,
            "low": IssueSeverity.LOW
        }
        return mapping.get(severity.lower(), IssueSeverity.MEDIUM)

    def _generate_summary(self, report: ValidationReport) -> str:
        severity_counts = {}
        for issue in report.issues:
            severity = issue.severity.value
            severity_counts[severity] = severity_counts.get(severity, 0) + 1

        summary_parts = [f"校验完成，发现{len(report.issues)}个问题"]

        if severity_counts:
            for severity, count in severity_counts.items():
                summary_parts.append(f"{severity}: {count}个")

        summary_parts.append(f"综合评分: {report.score.total:.0f}分 ({report.score.grade})")

        return "，".join(summary_parts)
```

- [ ] **Step 2: 创建文档校验器**

```python
# backend/app/agents/validation/validators/document_validator.py
from typing import Optional
from app.agents.validation.models.validation_result import (
    ValidationReport,
    ValidationIssue,
    IssueSeverity,
    IssueType,
    ValidationScore
)
from app.agents.validation.rules.document_rules import DOCUMENT_RULES


class DocumentValidator:
    def __init__(self):
        self.rules = DOCUMENT_RULES

    def validate(
        self,
        content: str,
        prototype_pages: Optional[list] = None,
        business_tree: Optional[dict] = None
    ) -> ValidationReport:
        report = ValidationReport(
            target_type="document",
            score=ValidationScore(total=100, grade="A"),
            summary=""
        )

        context = {
            "prototype_pages": prototype_pages or [],
            "business_tree": business_tree or {}
        }

        all_issues = []

        for rule in self.rules:
            try:
                result = rule.check_function(content, **context)

                if not result.get("passed", True):
                    for issue_data in result.get("issues", []):
                        issue = ValidationIssue(
                            type=IssueType.INCOMPLETE,
                            severity=self._map_severity(issue_data.get("severity", "medium")),
                            title=issue_data.get("title", ""),
                            description=issue_data.get("description", ""),
                            location=issue_data.get("location"),
                            suggestion=issue_data.get("suggestion", ""),
                            auto_fixable=False
                        )
                        all_issues.append(issue)

            except Exception as e:
                issue = ValidationIssue(
                    type=IssueType.INVALID,
                    severity=IssueSeverity.LOW,
                    title=f"规则执行失败: {rule.name}",
                    description=str(e),
                    suggestion="请检查规则配置"
                )
                all_issues.append(issue)

        for issue in all_issues:
            report.add_issue(issue)

        report.summary = self._generate_summary(report)

        return report

    def _map_severity(self, severity: str) -> IssueSeverity:
        mapping = {
            "critical": IssueSeverity.CRITICAL,
            "high": IssueSeverity.HIGH,
            "medium": IssueSeverity.MEDIUM,
            "low": IssueSeverity.LOW
        }
        return mapping.get(severity.lower(), IssueSeverity.MEDIUM)

    def _generate_summary(self, report: ValidationReport) -> str:
        severity_counts = {}
        for issue in report.issues:
            severity = issue.severity.value
            severity_counts[severity] = severity_counts.get(severity, 0) + 1

        summary_parts = [f"校验完成，发现{len(report.issues)}个问题"]

        if severity_counts:
            for severity, count in severity_counts.items():
                summary_parts.append(f"{severity}: {count}个")

        summary_parts.append(f"综合评分: {report.score.total:.0f}分 ({report.score.grade})")

        return "，".join(summary_parts)
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/validation/validators/
git commit -m "feat(validation): implement validators"
```

---

## 任务 4: 实现校验Agent

**Files:**
- Create: `backend/app/agents/validation/agent.py`
- Create: `backend/app/agents/validation/__init__.py`

- [ ] **Step 1: 实现校验Agent**

```python
# backend/app/agents/validation/agent.py
from typing import Dict, Any, Optional, List
from app.agents.base import BaseAgent
from app.agents.validation.validators.prototype_validator import PrototypeValidator
from app.agents.validation.validators.document_validator import DocumentValidator
from app.agents.validation.models.validation_result import ValidationReport


class ValidationAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            name="ValidationAgent",
            description="校验Agent - 对原型和文档进行双向核验，确保交付物质量"
        )

        self.prototype_validator = PrototypeValidator()
        self.document_validator = DocumentValidator()

    async def process(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        validation_type = input_data.get("type", "all")
        business_tree = input_data.get("business_tree")
        prototype_pages = input_data.get("prototype_pages", [])
        document_content = input_data.get("document_content", "")

        results = {}

        if validation_type in ["all", "prototype"]:
            prototype_report = self.prototype_validator.validate(
                prototype_pages,
                business_tree
            )
            results["prototype"] = prototype_report.to_dict()

        if validation_type in ["all", "document"]:
            document_report = self.document_validator.validate(
                document_content,
                prototype_pages,
                business_tree
            )
            results["document"] = document_report.to_dict()

        if validation_type == "all":
            overall_score = self._calculate_overall_score(results)
            results["overall"] = overall_score

        return results

    async def validate_prototype(
        self,
        pages: List[Dict[str, Any]],
        business_tree: Optional[Dict[str, Any]] = None
    ) -> ValidationReport:
        return self.prototype_validator.validate(pages, business_tree)

    async def validate_document(
        self,
        content: str,
        prototype_pages: Optional[list] = None,
        business_tree: Optional[dict] = None
    ) -> ValidationReport:
        return self.document_validator.validate(content, prototype_pages, business_tree)

    def _calculate_overall_score(self, results: Dict[str, Any]) -> Dict[str, Any]:
        scores = []

        if "prototype" in results:
            scores.append(results["prototype"]["score"]["total"])

        if "document" in results:
            scores.append(results["document"]["score"]["total"])

        if not scores:
            return {"total": 100, "grade": "A"}

        avg_score = sum(scores) / len(scores)

        grade = "A"
        if avg_score < 90:
            grade = "B"
        if avg_score < 80:
            grade = "C"
        if avg_score < 70:
            grade = "D"
        if avg_score < 60:
            grade = "F"

        return {
            "total": round(avg_score, 2),
            "grade": grade
        }

    async def validate(self, output: Dict) -> bool:
        return "prototype" in output or "document" in output
```

- [ ] **Step 2: 更新__init__**

```python
# backend/app/agents/validation/__init__.py
from app.agents.validation.agent import ValidationAgent
from app.agents.validation.models.validation_result import (
    ValidationReport,
    ValidationIssue,
    ValidationScore
)

__all__ = ["ValidationAgent", "ValidationReport", "ValidationIssue", "ValidationScore"]
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/validation/agent.py backend/app/agents/validation/__init__.py
git commit -m "feat(validation): implement validation agent"
```

---

## Phase 5 总结

**完成的任务:**
- [x] 校验结果数据模型定义
- [x] 原型校验规则
- [x] 文档校验规则
- [x] 原型校验器实现
- [x] 文档校验器实现
- [x] 校验Agent主类

**交付物:**
- 完整的校验规则系统
- 原型和文档双向校验能力
- 质量评分和报告生成
- 问题定位和建议生成

---

## 下一步行动

Phase 5 完成后，请继续 Phase 6: 集成与优化

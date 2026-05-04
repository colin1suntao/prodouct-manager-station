# Phase 3: 原型生成Agent - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现Vue3组件库、页面生成逻辑、交互配置系统，支持自动生成可交互原型页面

**Architecture:** 基于业务逻辑树生成Vue3组件代码，支持组件复用和交互配置

**Tech Stack:** Vue 3 | Vite | TailwindCSS | Pydantic | Jinja2

---

## 文件结构设计

```
backend/app/agents/prototype/
├── __init__.py
├── agent.py                  # 原型生成Agent主类
├── component_generator.py    # 组件代码生成器
├── page_generator.py         # 页面生成器
├── templates/                # Vue组件模板
│   ├── base/
│   │   ├── button.py
│   │   ├── input.py
│   │   ├── form.py
│   │   └── modal.py
│   └── layout/
│       ├── page.py
│       ├── header.py
│       └── sidebar.py
└── models/
    ├── __init__.py
    ├── page.py              # 页面模型
    └── component.py         # 组件模型
```

---

## 任务 1: 创建原型数据模型

**Files:**
- Create: `backend/app/agents/prototype/models/page.py`
- Create: `backend/app/agents/prototype/models/component.py`
- Create: `backend/app/agents/prototype/models/__init__.py`

- [ ] **Step 1: 创建页面模型**

```python
# backend/app/agents/prototype/models/page.py
from pydantic import BaseModel, Field
from typing import List, Optional, Literal
from enum import Enum


class PageType(str, Enum):
    LIST = "list"
    FORM = "form"
    DETAIL = "detail"
    DASHBOARD = "dashboard"
    LOGIN = "login"
    ERROR = "error"
    BLANK = "blank"


class FormField(BaseModel):
    name: str
    label: str
    type: Literal["text", "password", "email", "number", "textarea", "select", "checkbox", "radio", "date", "file"]
    required: bool = False
    placeholder: Optional[str] = None
    options: List[str] = Field(default_factory=list)
    validation: Optional[str] = None


class TableColumn(BaseModel):
    key: str
    title: str
    width: Optional[str] = None
    sortable: bool = False


class NavigationItem(BaseModel):
    path: str
    label: str
    icon: Optional[str] = None
    children: List["NavigationItem"] = Field(default_factory=list)


class PageConfig(BaseModel):
    title: str
    type: PageType = PageType.BLANK
    path: str
    layout: Literal["default", "auth", "blank"] = "default"
    fields: List[FormField] = Field(default_factory=list)
    columns: List[TableColumn] = Field(default_factory=list)
    actions: List[str] = Field(default_factory=list)
    navigation: List[NavigationItem] = Field(default_factory=list)


class PageOutput(BaseModel):
    id: str
    name: str
    path: str
    component_code: str
    config: PageConfig
    imports: List[str] = Field(default_factory=list)
    dependencies: dict = Field(default_factory=dict)
```

- [ ] **Step 2: 创建组件模型**

```python
# backend/app/agents/prototype/models/component.py
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any


class ComponentProp(BaseModel):
    name: str
    type: str
    required: bool = False
    default: Optional[Any] = None
    description: Optional[str] = None


class ComponentEvent(BaseModel):
    name: str
    description: Optional[str] = None


class ComponentSlot(BaseModel):
    name: str
    description: Optional[str] = None


class ComponentDefinition(BaseModel):
    name: str
    file_path: str
    category: Literal["base", "layout", "business", "feedback"]
    props: List[ComponentProp] = Field(default_factory=list)
    events: List[ComponentEvent] = Field(default_factory=list)
    slots: List[ComponentSlot] = Field(default_factory=list)
    code: str


class ComponentUsage(BaseModel):
    component_id: str
    props: Dict[str, Any] = Field(default_factory=dict)
    events: Dict[str, str] = Field(default_factory=dict)
    children: List["ComponentUsage"] = Field(default_factory=list)
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/prototype/models/
git commit -m "feat(prototype): add page and component models"
```

---

## 任务 2: 创建Vue组件模板

**Files:**
- Create: `backend/app/agents/prototype/templates/base/button.j2`
- Create: `backend/app/agents/prototype/templates/base/input.j2`
- Create: `backend/app/agents/prototype/templates/base/form.j2`
- Create: `backend/app/agents/prototype/templates/base/table.j2`
- Create: `backend/app/agents/prototype/templates/page/page.j2`

- [ ] **Step 1: 创建按钮组件模板**

```vue
{# backend/app/agents/prototype/templates/base/button.j2 #}
<template>
  <button
    :class="buttonClasses"
    :disabled="disabled || loading"
    :type="type"
    @click="handleClick"
  >
    <span v-if="loading" class="animate-spin mr-2">
      <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"></path>
      </svg>
    </span>
    <slot />
  </button>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
  loading?: boolean
  type?: 'button' | 'submit' | 'reset'
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  disabled: false,
  loading: false,
  type: 'button'
})

const emit = defineEmits<{
  (e: 'click', event: Event): void
}>()

const buttonClasses = computed(() => {
  const base = 'inline-flex items-center justify-center font-medium rounded-md transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2'

  const variants = {
    primary: 'bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500',
    secondary: 'bg-gray-600 text-white hover:bg-gray-700 focus:ring-gray-500',
    outline: 'border-2 border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-gray-500',
    ghost: 'text-gray-700 hover:bg-gray-100 focus:ring-gray-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500'
  }

  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  }

  const disabled = props.disabled || props.loading
    ? 'opacity-50 cursor-not-allowed'
    : 'cursor-pointer'

  return `${base} ${variants[props.variant]} ${sizes[props.size]} ${disabled}`
})

const handleClick = (event: Event) => {
  if (!props.disabled && !props.loading) {
    emit('click', event)
  }
}
</script>
```

- [ ] **Step 2: 创建输入框组件模板**

```vue
{# backend/app/agents/prototype/templates/base/input.j2 #}
<template>
  <div class="input-wrapper">
    <label v-if="label" :for="inputId" class="block text-sm font-medium text-gray-700 mb-1">
      {{ label }}
      <span v-if="required" class="text-red-500">*</span>
    </label>

    <div class="relative">
      <span v-if="prefix" class="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400">
        {{ prefix }}
      </span>

      <input
        :id="inputId"
        v-model="modelValue"
        :type="type"
        :placeholder="placeholder"
        :disabled="disabled"
        :class="inputClasses"
        @input="handleInput"
        @blur="handleBlur"
      />

      <span v-if="suffix" class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400">
        {{ suffix }}
      </span>

      <span v-if="clearable && modelValue" class="absolute right-3 top-1/2 -translate-y-1/2">
        <button type="button" @click="clear" class="text-gray-400 hover:text-gray-600">
          <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
          </svg>
        </button>
      </span>
    </div>

    <p v-if="error" class="mt-1 text-sm text-red-500">{{ error }}</p>
    <p v-else-if="hint" class="mt-1 text-sm text-gray-500">{{ hint }}</p>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

interface Props {
  modelValue?: string | number
  type?: string
  label?: string
  placeholder?: string
  prefix?: string
  suffix?: string
  disabled?: boolean
  readonly?: boolean
  required?: boolean
  clearable?: boolean
  error?: string
  hint?: string
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  type: 'text',
  disabled: false,
  readonly: false,
  required: false,
  clearable: false
})

const emit = defineEmits<{
  (e: 'update:modelValue', value: string | number): void
  (e: 'blur'): void
}>()

const inputId = ref(`input-${Math.random().toString(36).slice(2, 9)}`)

const modelValue = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const inputClasses = computed(() => {
  const base = 'w-full px-3 py-2 border rounded-md transition-colors focus:outline-none focus:ring-2'

  const state = props.error
    ? 'border-red-500 focus:ring-red-500'
    : 'border-gray-300 focus:ring-primary-500 focus:border-primary-500'

  const padding = props.prefix ? 'pl-8' : props.suffix ? 'pr-8' : ''

  const disabled = props.disabled ? 'bg-gray-100 cursor-not-allowed' : ''

  return `${base} ${state} ${padding} ${disabled}`
})

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  emit('update:modelValue', target.value)
}

const handleBlur = () => {
  emit('blur')
}

const clear = () => {
  emit('update:modelValue', '')
}
</script>
```

- [ ] **Step 3: 创建表单组件模板**

```vue
{# backend/app/agents/prototype/templates/base/form.j2 #}
<template>
  <form @submit.prevent="handleSubmit" class="space-y-4">
    <div v-for="field in fields" :key="field.name" class="form-field">
      <InputField
        v-if="field.type === 'text' || field.type === 'email' || field.type === 'password'"
        v-model="formData[field.name]"
        :label="field.label"
        :type="field.type"
        :required="field.required"
        :placeholder="field.placeholder"
        :error="errors[field.name]"
      />

      <TextareaField
        v-else-if="field.type === 'textarea'"
        v-model="formData[field.name]"
        :label="field.label"
        :required="field.required"
        :placeholder="field.placeholder"
        :error="errors[field.name]"
      />

      <SelectField
        v-else-if="field.type === 'select'"
        v-model="formData[field.name]"
        :label="field.label"
        :required="field.required"
        :options="field.options"
        :error="errors[field.name]"
      />

      <CheckboxField
        v-else-if="field.type === 'checkbox'"
        v-model="formData[field.name]"
        :label="field.label"
        :error="errors[field.name]"
      />
    </div>

    <div class="form-actions">
      <slot name="actions">
        <button type="submit" class="btn-primary">
          提交
        </button>
        <button type="button" class="btn-secondary" @click="handleReset">
          重置
        </button>
      </slot>
    </div>
  </form>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import type { FormField } from '@/agents/prototype/models/page'

interface Props {
  fields: FormField[]
  initialData?: Record<string, any>
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: 'submit', data: Record<string, any>): void
  (e: 'reset'): void
}>()

const formData = reactive<Record<string, any>>(
  props.initialData || props.fields.reduce((acc, field) => {
    acc[field.name] = field.type === 'checkbox' ? false : ''
    return acc
  }, {})
)

const errors = ref<Record<string, string>>({})

const validate = (): boolean => {
  errors.value = {}

  for (const field of props.fields) {
    if (field.required && !formData[field.name]) {
      errors.value[field.name] = `${field.label}不能为空`
    }
  }

  return Object.keys(errors.value).length === 0
}

const handleSubmit = () => {
  if (validate()) {
    emit('submit', { ...formData })
  }
}

const handleReset = () => {
  Object.keys(formData).forEach(key => {
    const field = props.fields.find(f => f.name === key)
    formData[key] = field?.type === 'checkbox' ? false : ''
  })
  errors.value = {}
  emit('reset')
}
</script>
```

- [ ] **Step 4: 创建表格组件模板**

```vue
{# backend/app/agents/prototype/templates/base/table.j2 #}
<template>
  <div class="table-wrapper">
    <div v-if="loading" class="table-loading">
      <div class="animate-spin h-8 w-8 border-4 border-primary-500 border-t-transparent rounded-full"></div>
    </div>

    <table v-else class="min-w-full divide-y divide-gray-200">
      <thead class="bg-gray-50">
        <tr>
          <th
            v-for="column in columns"
            :key="column.key"
            :style="column.width ? { width: column.width } : {}"
            class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
          >
            {{ column.title }}
            <span v-if="column.sortable" class="ml-1 cursor-pointer">↕</span>
          </th>
        </tr>
      </thead>
      <tbody class="bg-white divide-y divide-gray-200">
        <tr v-if="data.length === 0">
          <td :colspan="columns.length" class="px-6 py-4 text-center text-gray-500">
            暂无数据
          </td>
        </tr>
        <tr
          v-for="(row, index) in data"
          :key="index"
          class="hover:bg-gray-50"
        >
          <td
            v-for="column in columns"
            :key="column.key"
            class="px-6 py-4 whitespace-nowrap text-sm text-gray-900"
          >
            <slot :name="`cell-${column.key}`" :row="row" :value="row[column.key]">
              {{ row[column.key] }}
            </slot>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="pagination" class="table-pagination">
      <span class="text-sm text-gray-700">
        共 {{ pagination.total }} 条，第 {{ pagination.page }} / {{ pagination.totalPages }} 页
      </span>
      <div class="flex gap-2">
        <button
          :disabled="pagination.page <= 1"
          @click="$emit('page-change', pagination.page - 1)"
          class="btn-secondary"
        >
          上一页
        </button>
        <button
          :disabled="pagination.page >= pagination.totalPages"
          @click="$emit('page-change', pagination.page + 1)"
          class="btn-secondary"
        >
          下一页
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { TableColumn } from '@/agents/prototype/models/page'

interface Props {
  columns: TableColumn[]
  data: Record<string, any>[]
  loading?: boolean
  pagination?: {
    page: number
    pageSize: number
    total: number
    totalPages: number
  }
}

withDefaults(defineProps<Props>(), {
  loading: false
})

defineEmits<{
  (e: 'page-change', page: number): void
}>()
</script>
```

- [ ] **Step 5: 创建页面模板**

```vue
{# backend/app/agents/prototype/templates/page/page.j2 #}
<template>
  <div class="page-container">
    <header v-if="showHeader" class="page-header">
      <h1 class="page-title">{{ title }}</h1>
      <div v-if="$slots.actions" class="page-actions">
        <slot name="actions" />
      </div>
    </header>

    <main class="page-content">
      <slot />
    </main>

    <footer v-if="showFooter" class="page-footer">
      <slot name="footer" />
    </footer>
  </div>
</template>

<script setup lang="ts">
interface Props {
  title?: string
  showHeader?: boolean
  showFooter?: boolean
}

withDefaults(defineProps<Props>(), {
  showHeader: true,
  showFooter: false
})
</script>

<style scoped>
.page-container {
  @apply min-h-screen bg-gray-50;
}

.page-header {
  @apply bg-white shadow-sm px-6 py-4 flex items-center justify-between;
}

.page-title {
  @apply text-xl font-semibold text-gray-900;
}

.page-actions {
  @apply flex gap-2;
}

.page-content {
  @apply p-6;
}

.page-footer {
  @apply bg-white border-t px-6 py-4;
}
</style>
```

- [ ] **Step 6: Commit**

```bash
git add backend/app/agents/prototype/templates/
git commit -m "feat(prototype): add Vue component templates"
```

---

## 任务 3: 实现组件生成器

**Files:**
- Create: `backend/app/agents/prototype/component_generator.py`

- [ ] **Step 1: 创建组件生成器**

```python
# backend/app/agents/prototype/component_generator.py
from typing import List, Dict, Optional
from pathlib import Path
from jinja2 import Environment, FileSystemLoader
from app.agents.prototype.models.component import ComponentDefinition, ComponentUsage
from app.agents.prototype.models.page import FormField, TableColumn


class ComponentGenerator:
    def __init__(self, template_dir: Optional[str] = None):
        if template_dir:
            self.env = Environment(loader=FileSystemLoader(template_dir))
        else:
            base_dir = Path(__file__).parent
            template_path = base_dir / "templates"
            self.env = Environment(loader=FileSystemLoader(str(template_path)))

    def generate_form_fields(self, fields: List[FormField]) -> str:
        template = self.env.get_template("base/form.j2")
        return template.render(fields=[f.model_dump() for f in fields])

    def generate_table(self, columns: List[TableColumn], data_key: str = "data") -> str:
        template = self.env.get_template("base/table.j2")
        return template.render(
            columns=[c.model_dump() for c in columns],
            data=data_key
        )

    def generate_button(
        self,
        text: str,
        variant: str = "primary",
        size: str = "md",
        icon: Optional[str] = None
    ) -> str:
        template = self.env.get_template("base/button.j2")
        return template.render(
            text=text,
            variant=variant,
            size=size,
            icon=icon
        )

    def generate_page(
        self,
        title: str,
        show_header: bool = True,
        show_footer: bool = False,
        content: Optional[str] = None
    ) -> str:
        template = self.env.get_template("page/page.j2")
        return template.render(
            title=title,
            showHeader=show_header,
            showFooter=show_footer
        )

    def build_component_usage(
        self,
        component_name: str,
        props: Optional[Dict] = None,
        events: Optional[Dict] = None,
        children: Optional[List[ComponentUsage]] = None
    ) -> ComponentUsage:
        return ComponentUsage(
            component_id=component_name,
            props=props or {},
            events=events or {},
            children=children or []
        )

    def render_component_tree(self, usage: ComponentUsage) -> str:
        component_name = usage.component_id
        props_str = " ".join([
            f':{k}="{v}"' if not isinstance(v, str) else f'{k}="{v}"'
            for k, v in usage.props.items()
        ])

        if not usage.children:
            return f"<{component_name} {props_str} />"

        children_html = "".join([
            self.render_component_tree(child)
            for child in usage.children
        ])

        return f"<{component_name} {props_str}>{children_html}</{component_name}>"
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/prototype/component_generator.py
git commit -m "feat(prototype): implement component generator"
```

---

## 任务 4: 实现页面生成器

**Files:**
- Create: `backend/app/agents/prototype/page_generator.py`

- [ ] **Step 1: 创建页面生成器**

```python
# backend/app/agents/prototype/page_generator.py
from typing import List, Dict, Optional, Any
from uuid import uuid4
from pathlib import Path
from app.agents.prototype.models.page import (
    PageConfig,
    PageType,
    PageOutput,
    FormField,
    TableColumn
)
from app.agents.prototype.models.business_tree import BusinessTree, FlowNode, UserScenario
from app.agents.prototype.component_generator import ComponentGenerator


class PageGenerator:
    def __init__(self):
        self.component_generator = ComponentGenerator()

    def generate_from_business_tree(
        self,
        business_tree: BusinessTree,
        project_name: str = "project"
    ) -> List[PageOutput]:
        pages = []

        pages.extend(self._generate_page_from_flow(business_tree.business_flow))

        for scenario in business_tree.user_scenarios:
            page = self._generate_page_from_scenario(scenario)
            pages.append(page)

        if business_tree.business_flow and business_tree.business_flow.nodes:
            main_page = self._generate_main_page(business_tree)
            pages.append(main_page)

        return pages

    def _generate_page_from_flow(
        self,
        flow: Optional[Any]
    ) -> List[PageOutput]:
        if not flow or not flow.nodes:
            return []

        pages = []
        for node in flow.nodes:
            page_config = self._create_page_config_from_node(node)
            page = self._generate_page(page_config)
            pages.append(page)

        return pages

    def _generate_page_from_scenario(
        self,
        scenario: UserScenario
    ) -> PageOutput:
        page_config = PageConfig(
            title=scenario.name,
            type=PageType.FORM if len(scenario.path) > 3 else PageType.LIST,
            path=f"/scenario/{scenario.id}"
        )

        if scenario.preconditions:
            page_config.fields = [
                FormField(
                    name="precondition",
                    label="前置条件",
                    type="textarea"
                )
            ]

        return self._generate_page(page_config)

    def _generate_main_page(self, business_tree: BusinessTree) -> PageOutput:
        page_config = PageConfig(
            title="主页面",
            type=PageType.DASHBOARD,
            path="/",
            navigation=self._generate_navigation(business_tree)
        )

        return self._generate_page(page_config)

    def _create_page_config_from_node(self, node: FlowNode) -> PageConfig:
        node_type_map = {
            "action": PageType.FORM,
            "decision": PageType.LIST,
            "data": PageType.DETAIL,
            "ui": PageType.FORM,
            "event": PageType.LIST
        }

        page_type = node_type_map.get(node.type, PageType.BLANK)

        config = PageConfig(
            title=node.name,
            type=page_type,
            path=f"/{node.id}"
        )

        if node.input_fields:
            config.fields = [
                FormField(name=field, label=field, type="text")
                for field in node.input_fields
            ]

        return config

    def _generate_page(self, config: PageConfig) -> PageOutput:
        page_id = str(uuid4())
        component_code = self._build_page_component(config)

        return PageOutput(
            id=page_id,
            name=config.title,
            path=config.path,
            component_code=component_code,
            config=config,
            imports=[
                "vue",
                "pinia"
            ],
            dependencies={}
        )

    def _build_page_component(self, config: PageConfig) -> str:
        page_template = f"""
<template>
  <div class="page-{config.type.value}">
    <header class="page-header">
      <h1 class="page-title">{config.title}</h1>
      <div class="page-actions">
        {self._generate_action_buttons(config.actions)}
      </div>
    </header>

    <main class="page-content">
      {self._generate_content(config)}
    </main>
  </div>
</template>

<script setup lang="ts">
import {{ ref, reactive }} from 'vue'
{self._generate_script_content(config)}
</script>

<style scoped>
.{config.type.value} {{
  @apply min-h-screen bg-gray-50;
}}
.page-header {{
  @apply bg-white shadow-sm px-6 py-4 flex items-center justify-between;
}}
.page-title {{
  @apply text-xl font-semibold text-gray-900;
}}
.page-actions {{
  @apply flex gap-2;
}}
.page-content {{
  @apply p-6;
}}
</style>
"""
        return page_template

    def _generate_action_buttons(self, actions: List[str]) -> str:
        if not actions:
            return ""
        buttons = []
        for action in actions:
            action_lower = action.lower()
            variant = "primary" if action_lower in ["submit", "save", "confirm"] else "secondary"
            buttons.append(f'<button class="btn-{variant}" @click="handle{action}">{action}</button>')
        return "\n        ".join(buttons)

    def _generate_content(self, config: PageConfig) -> str:
        if config.type == PageType.FORM and config.fields:
            return self._generate_form_content(config.fields)
        elif config.type == PageType.LIST and config.columns:
            return self._generate_table_content(config.columns)
        elif config.type == PageType.DETAIL:
            return '<div class="detail-content">详情内容</div>'
        elif config.type == PageType.DASHBOARD:
            return '<div class="dashboard-content">仪表盘内容</div>'
        else:
            return '<div class="empty-content">页面内容</div>'

    def _generate_form_content(self, fields: List[FormField]) -> str:
        field_inputs = []
        for field in fields:
            field_inputs.append(f'''
        <div class="form-field">
          <label class="form-label">{field.label}</label>
          <input
            v-model="formData.{field.name}"
            type="{field.type}"
            class="form-input"
            placeholder="{field.placeholder or ''}"
          />
        </div>
''')
        return f'''
    <form @submit.prevent="handleSubmit" class="form">
      {"".join(field_inputs)}
      <div class="form-actions">
        <button type="submit" class="btn-primary">提交</button>
        <button type="button" class="btn-secondary" @click="handleReset">重置</button>
      </div>
    </form>
'''

    def _generate_table_content(self, columns: List[TableColumn]) -> str:
        headers = "\n          ".join([
            f'<th>{col.title}</th>'
            for col in columns
        ])

        cells = "\n          ".join([
            f'<td>{{{{ row.{col.key} }}}}</td>'
            for col in columns
        ])

        return f'''
    <table class="data-table">
      <thead>
        <tr>
          {headers}
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in tableData" :key="index">
          {cells}
        </tr>
      </tbody>
    </table>
'''

    def _generate_script_content(self, config: PageConfig) -> str:
        script_parts = []

        if config.type == PageType.FORM and config.fields:
            fields_dict = {f.name: "" for f in config.fields}
            script_parts.append(f'''
const formData = reactive({fields_dict})

const handleSubmit = () => {{
  console.log('Form submitted:', formData)
}}

const handleReset = () => {{
  Object.keys(formData).forEach(key => {{
    formData[key] = ''
  }})
}}
''')

        elif config.type == PageType.LIST:
            script_parts.append('''
const tableData = ref([])

const fetchData = async () => {
  // TODO: Fetch data from API
}
''')

        return "\n".join(script_parts)

    def _generate_navigation(self, business_tree: BusinessTree) -> List[Dict]:
        navigation = []

        if business_tree.business_flow:
            navigation.append({
                "path": "/",
                "label": "首页",
                "icon": "home"
            })

        for scenario in business_tree.user_scenarios[:5]:
            navigation.append({
                "path": f"/scenario/{scenario.id}",
                "label": scenario.name,
                "icon": "document"
            })

        return navigation

    def save_pages(
        self,
        pages: List[PageOutput],
        output_dir: str
    ) -> List[str]:
        output_path = Path(output_dir)
        output_path.mkdir(parents=True, exist_ok=True)

        saved_files = []
        for page in pages:
            file_path = output_path / f"{page.id}.vue"
            file_path.write_text(page.component_code)
            saved_files.append(str(file_path))

        return saved_files
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/agents/prototype/page_generator.py
git commit -m "feat(prototype): implement page generator"
```

---

## 任务 5: 实现原型生成Agent

**Files:**
- Create: `backend/app/agents/prototype/agent.py`
- Create: `backend/app/agents/prototype/__init__.py`

- [ ] **Step 1: 实现原型生成Agent**

```python
# backend/app/agents/prototype/agent.py
from typing import List, Optional, Dict, Any
from uuid import uuid4
from pathlib import Path
from app.agents.base import BaseAgent
from app.agents.prototype.page_generator import PageGenerator
from app.agents.prototype.models.page import PageOutput
from app.agents.requirement.models.business_tree import BusinessTree


class PrototypeAgent(BaseAgent):
    def __init__(self, output_dir: Optional[str] = None):
        super().__init__(
            name="PrototypeGenerator",
            description="原型生成Agent - 基于业务逻辑树生成Vue3可交互原型页面"
        )

        self.page_generator = PageGenerator()
        self.output_dir = output_dir or "./output/prototypes"

    async def process(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        business_tree = input_data.get("business_tree")
        project_name = input_data.get("project_name", "project")

        if isinstance(business_tree, dict):
            business_tree = BusinessTree.from_dict(business_tree)

        pages = self.page_generator.generate_from_business_tree(
            business_tree,
            project_name
        )

        if self.output_dir:
            saved_files = self.page_generator.save_pages(pages, self.output_dir)
        else:
            saved_files = []

        return {
            "status": "success",
            "pages": [p.model_dump() for p in pages],
            "page_count": len(pages),
            "saved_files": saved_files
        }

    async def validate(self, output: Dict) -> bool:
        pages = output.get("pages", [])
        return len(pages) > 0

    def generate_single_page(
        self,
        page_config: Dict[str, Any]
    ) -> PageOutput:
        from app.agents.prototype.models.page import PageConfig

        config = PageConfig(**page_config)
        return self.page_generator._generate_page(config)

    async def export_prototype(
        self,
        pages: List[PageOutput],
        format: str = "vue"
    ) -> Dict[str, Any]:
        if format == "vue":
            return {
                "status": "success",
                "files": [
                    {
                        "name": f"{p.id}.vue",
                        "path": f"{p.path}.vue",
                        "content": p.component_code
                    }
                    for p in pages
                ]
            }
        elif format == "json":
            return {
                "status": "success",
                "data": [p.model_dump() for p in pages]
            }
        else:
            raise ValueError(f"Unsupported format: {format}")
```

- [ ] **Step 2: 更新__init__**

```python
# backend/app/agents/prototype/__init__.py
from app.agents.prototype.agent import PrototypeAgent
from app.agents.prototype.page_generator import PageGenerator
from app.agents.prototype.models.page import PageOutput, PageConfig

__all__ = ["PrototypeAgent", "PageGenerator", "PageOutput", "PageConfig"]
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/agents/prototype/agent.py backend/app/agents/prototype/__init__.py
git commit -m "feat(prototype): implement prototype generation agent"
```

---

## Phase 3 总结

**完成的任务:**
- [x] 原型数据模型定义
- [x] Vue组件模板创建
- [x] 组件生成器实现
- [x] 页面生成器实现
- [x] 原型生成Agent主类

**交付物:**
- 完整的Vue3组件模板库
- 支持多种页面类型（列表、表单、详情、仪表盘）
- 可扩展的组件生成系统
- 原型导出功能

---

## 下一步行动

Phase 3 完成后，请继续 Phase 4: 文档生成Agent

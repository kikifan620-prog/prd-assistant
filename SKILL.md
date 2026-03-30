---
name: prd-assistant
description: Analyze PRD documents, check requirement value completeness, generate Mermaid flowcharts, feature lists, tracking events, and ASCII wireframes. Use when the user provides PRD content, product requirements, or asks for help with requirement analysis, feature extraction, data tracking design, or wireframe generation.
---

# PRD Assistant

Analyze product requirement documents and generate comprehensive outputs including value validation, flowcharts, feature lists, tracking events, and wireframes.

## Workflow

When user provides PRD content:

1. **Parse PRD** - Extract background, requirements, business flow
2. **Check Value Completeness** - Verify target user, expected benefit, success metrics
3. **If incomplete** → Output follow-up questions
4. **If complete** → Generate all outputs

## Value Completeness Check

Check if PRD contains:

| Check Item | Question to Ask |
|------------|-----------------|
| Target User | "这个需求面向哪些用户？新用户/老用户/特定人群？" |
| Expected Benefit | "预期能带来什么收益？转化率提升/GMV增长/效率提升？" |
| Quantified Benefit | "收益能量化吗？比如转化率从X%提升到Y%？" |
| Success Metrics | "如何衡量这个需求的成功？核心指标是什么？" |
| Launch Timeline | "预期什么时候上线？" |

**If any missing**, output:
```
📋 需求价值补充追问：

1. [Missing item]: [Question]
2. [Missing item]: [Question]
...

请补充以上信息后，我将为您生成完整的功能清单、流程图、数据埋点和原型图。
```

## Output Generation

Once PRD is complete, generate:

### 1. Mermaid Flowchart

Parse business flow and generate Mermaid diagram:

```mermaid
flowchart TD
    A[Start] --> B{Decision}
    B -->|Yes| C[Action 1]
    B -->|No| D[Action 2]
    C --> E[End]
    D --> E
```

Include:
- Main flow (happy path)
- Exception branches
- Decision points with clear labels

### 2. Feature List Table

| 序号 | 功能模块 | 功能点 | 优先级 | 详细描述 | 验收标准 |
|------|---------|--------|--------|---------|---------|
| 1 | {Module} | {Feature} | P0/P1/P2 | {Description} | {Acceptance} |

Rules:
- Extract one feature per requirement sentence
- Mark core features as P0, supporting as P1/P2
- Description should include user action + system response
- Acceptance criteria must be measurable

### 3. Tracking Events Table

| 埋点事件 | 事件ID | 触发时机 | 事件属性 | 优先级 |
|---------|--------|---------|---------|--------|
| {Event} | {snake_case_id} | {Trigger} | {Properties} | 必需/建议 |

Rules:
- Every feature needs at least one event
- Page features must have view/impression event
- Button features must have click event
- Include key business fields in properties

### 4. ASCII Wireframes

Generate character-based wireframes for each page:

```
┌─────────────────────────┐
│  Title              Btn │
├─────────────────────────┤
│ ┌────┐                │
│ │    │  Content        │
│ │IMG │  Description    │
│ │    │  [Action]       │
│ └────┘                │
├─────────────────────────┤
│  [Button]               │
└─────────────────────────┘
```

Guidelines:
- Show layout structure (header, content, footer)
- Include key UI elements
- Mark interactive components with [brackets]
- Add brief interaction notes below

## Example

**Input:**
```
背景：购物车转化率低
需求：1. 增加优惠券推荐 2. 合并结算步骤
流程：用户进入购物车→看到推荐→点击领取→点击结算→完成支付
```

**Output:**

📋 需求价值补充追问：
1. 目标用户：这个需求面向哪些用户？
2. 预期收益：预期转化率从多少提升到多少？
3. 上线时间：预期什么时候上线？

(After user provides missing info)

```mermaid
flowchart TD
    A[进入购物车] --> B{看到优惠券推荐}
    B -->|点击| C[自动领券]
    B -->|忽略| D[保持原价]
    C --> E[点击结算]
    D --> E
    E --> F[合并结算页]
    F --> G{支付}
    G -->|成功| H[完成]
    G -->|失败| I[提示错误]
    I --> F
```

📋 功能清单
| 序号 | 功能模块 | 功能点 | 优先级 | 详细描述 | 验收标准 |
|------|---------|--------|--------|---------|---------|
| 1 | 购物车 | 优惠券推荐 | P0 | 根据购物车金额推荐最优券 | 推荐准确率>80% |
| 2 | 购物车 | 一键领券 | P0 | 点击推荐卡片自动领券并应用 | 领券成功率>95% |
| 3 | 结算 | 流程合并 | P0 | 合并地址和支付选择为一步 | 步骤从4步减到2步 |

📊 数据埋点
| 埋点事件 | 事件ID | 触发时机 | 事件属性 | 优先级 |
|---------|--------|---------|---------|--------|
| 购物车曝光 | cart_view | 进入购物车 | page_name, item_count | 必需 |
| 推荐曝光 | coupon_recommend_show | 推荐卡片展示 | coupon_id, value | 必需 |
| 推荐点击 | coupon_recommend_click | 点击推荐 | coupon_id, result | 必需 |

📱 原型图

页面1：购物车
```
┌─────────────────────────┐
│  购物车(3)          编辑  │
├─────────────────────────┤
│ ┌────┐ 商品名称      ✓ │
│ │    │ 规格：红色     ○ │
│ │ 图 │ ¥99          + - │
│ │ 片 │               🗑 │
│ └────┘                  │
├─────────────────────────┤
│ ┌─────────────────────┐ │
│ │ 🎫 可省¥20  立即领取 │ │
│ └─────────────────────┘ │
├─────────────────────────┤
│  合计：¥198    [去结算] │
└─────────────────────────┘
```
- 优惠券卡片：显示可省金额+领取按钮
- 失效商品：置灰显示，可批量清理

## Notes

- Always check value completeness first
- Generate all 4 outputs (flowchart, features, tracking, wireframes) together
- Use Chinese for all outputs since PRDs are typically in Chinese
- Keep feature descriptions concrete and measurable

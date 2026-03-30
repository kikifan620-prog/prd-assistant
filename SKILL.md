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

### 3. Tracking Events Table (LZD AIDC规范)

基于 LZD App端流量核心埋点规范（AIDC统一规范）生成埋点：

#### 事件类型编码
| 事件类型 | 编码 | 说明 |
|---------|------|------|
| 页面事件 | 2001 | 页面进入/离开（native离开上报，h5进入上报） |
| 点击事件 | 2101 | 按钮/卡片点击 |
| 曝光事件 | 2201 | 组件/卡片可见时上报 |

#### SPM位置模型（四段编码）
SPM格式：`spma.b.c.d`
- **A段**: 站点/业务（native固定a211g0，h5按国家区分如a2o4j）
- **B段**: 页面（如pdp, cart, search）
- **C段**: 页面区块（如list, header, banner）
- **D段**: 区块内点位（如index数字）

**SPM三件套**：
- 页面事件(2001)：spm-cnt(2位), spm-url(4位), spm-pre(4位) - **必埋**
- 曝光/点击(2201/2101)：spm(3/4位), spm-url(4位), spm-pre(4位) - **spm必埋，url/pre选埋**

#### 埋点表格格式

| 埋点事件 | 事件类型 | arg1(logkey) | 触发时机 | 关键参数 | 优先级 |
|---------|---------|--------------|---------|---------|--------|
| {事件名} | 2001/2101/2201 | {page_xxx} | {触发时机} | {参数列表} | 必需/建议 |

#### 商品卡埋点规范（含广告）

**曝光/点击事件参数**：
```
必埋: spm=a.b.c.d, _p_prod, _p_sku
选埋: _p_slr, _p_shop, spm-url, spm-pre, spm-cnt
广告必埋: x_object_type=item/ad, x_object_id, utLogMap(json)
广告可选: adProduct, adSubProduct
```

**arg1命名规范**：
- native: Product_Exposure_Event / Product_Click_Event
- h5: /Product.Exposure.Event / /Product.Click.Event

#### 商卡直接加购埋点

**触发时机**：点击商品卡上的加购/BuyNow按钮

**关键参数**：
```
必埋: spm, spm-url, spm-pre, _p_prod, _p_sku
选埋: _p_slr, _p_shop, spm-cnt
透传: utparam-url（来源页面utLogMap，广告流量必埋）
```

**arg1命名**：
- 加购: Product_ATC_Click (native) / /Product.ATC.Click (h5)
- BuyNow: Product_BuyNow_Click (native) / /Product.BuyNow.Click (h5)

#### IPV的utparam透传

**作用**：将场域卡片的服务端信息(utLogMap)透传至PDP页面，用于归因分析

**参数**：
```
utparam-url: 来源页面的utLogMap（json结构）
包含: x_object_type, x_object_id, adProduct, adSubProduct
```

#### 埋点生成规则

1. **页面类功能**：生成2001页面事件，含spm-cnt/spm-url/spm-pre
2. **按钮类功能**：生成2101点击事件，含spm三件套
3. **列表/卡片类功能**：生成2201曝光+2101点击事件对
4. **商品卡功能**：按商品卡规范生成，区分自然/广告流量参数
5. **加购类功能**：按商卡直接加购规范生成，注意utparam透传

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

📊 数据埋点（AIDC规范）

| 埋点事件 | 事件类型 | arg1 | 触发时机 | 关键参数 | 优先级 |
|---------|---------|------|---------|---------|--------|
| 购物车页面 | 2001 | page_cart | 进入购物车页 | spm-cnt, spm-url, spm-pre | 必需 |
| 优惠券推荐曝光 | 2201 | page_cart_coupon_exp | 推荐卡片可见 | spm, _p_prod, _p_sku | 必需 |
| 优惠券推荐点击 | 2101 | page_cart_coupon_clk | 点击推荐卡片 | spm, spm-url, spm-pre, result | 必需 |
| 去结算点击 | 2101 | page_cart_checkout_clk | 点击结算按钮 | spm, spm-url, spm-pre | 必需 |

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

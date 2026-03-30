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

根据 PRD 内容选择合适的流程图类型：

#### 类型一：标准流程图（默认）
适合简单业务流程，单一路径展示：

```mermaid
flowchart TD
    A[Start] --> B{Decision}
    B -->|Yes| C[Action 1]
    B -->|No| D[Action 2]
    C --> E[End]
    D --> E
```

#### 类型二：泳道图（跨职能流程图）
适合多角色/多系统协作的复杂流程：

```mermaid
flowchart TB
    subgraph 角色A["👤 角色A"]
        A[步骤1] --> B[步骤2]
    end
    
    subgraph 角色B["👤 角色B"]
        C[步骤3] --> D{判断}
        D -->|条件1| E[步骤4]
        D -->|条件2| F[步骤5]
    end
    
    B --> C
    E --> G[结束]
    F --> G
```

**选择标准**：
- 单一角色操作 → 标准流程图
- 多角色协作（用户/商家/系统/数据）→ 泳道图

**包含要素**：
- 主流程（happy path）
- 异常分支
- 判断节点（菱形，带清晰标签）
- 各泳道内步骤按执行顺序排列

### 2. Feature List Table

| 序号 | 功能模块 | 功能点 | 优先级 | 详细描述 | 验收标准 |
|------|---------|--------|--------|---------|---------|
| 1 | {Module} | {Feature} | P0/P1/P2 | {Description} | {Acceptance} |

Rules:
- Extract one feature per requirement sentence
- Mark core features as P0, supporting as P1/P2
- Description should include user action + system response
- Acceptance criteria must be measurable

### 3. Tracking Events Table (数据埋点规范)

基于 App端流量核心埋点规范生成埋点：

#### 事件类型编码
| 事件类型 | 编码 | 说明 |
|---------|------|------|
| 页面事件 | 2001 | 页面浏览 PV（native离开上报，h5进入上报） |
| 点击事件 | 2101 | 按钮/卡片点击 |
| 曝光事件 | 2201 | 组件/卡片可见时上报 |

#### 字段命名规范
- **所有埋点字段必须全部小写**（数据侧统一小写+下划线处理）
- 参数使用下划线连接（如 spm-cnt, _p_prod）

#### SPM位置模型（四段编码）
SPM格式：`spma.b.c.d`

| 段位 | 含义 | 示例 |
|------|------|------|
| A段 | 站点/业务 | 各国站点编码 |
| B段 | 页面类型 | flashsale / lazsubsidy / lazflash_promotion / lazflash-brand / store_brand_sale / themeflashsale / lazflash_usersegment |
| C段 | 页面区块 | back / more / share / cart / rule / searchbox / jfy / mvpmodule / brandmodule / ucfsmodule / 8pmmodule / usersegmentmodule |
| D段 | 区块内点位 | tab / title / reminder / back / index数字 |

**SPM三件套**：
- 页面事件(2001)：spm-cnt(2位: spma.b), spm-url(4位), spm-pre(4位) - **必埋**
- 曝光/点击(2201/2101)：spm(3/4位), spm-url(4位), spm-pre(4位) - **spm必埋，url/pre选埋**

**spm-url vs spm-pre 区别**：
- **spm-url**：直接来源页面（上一页）
- **spm-pre**：间接来源页面（上上游页面），用于追溯完整用户路径

#### 埋点表格格式

| 子模块 | 事件描述 | 事件类型 | page | arg1 | 一级key | 二级key | value | 参数含义 | 备注 |
|--------|---------|---------|------|------|---------|---------|-------|---------|------|
| {模块名} | {曝光/点击/页面事件} | 2001/2101/2201 | H5: url | {arg1} | {一级key} | {二级key} | {value} | {参数说明} | 必埋/选埋 |

#### 事件上报路径规范 (page)

| 场景 | page 格式 | 示例 |
|------|----------|------|
| 页面曝光 | /{lp_name}.lp.exposure | /flashsale.lp.exposure |
| 页面点击 | /{lp_name}.lp.click | /flashsale.lp.click |
| 商品曝光 | /Product.Exposure.Event | - |
| 商品点击 | /Product.Click.Event | - |
| 搜索框曝光 | /{spma}.page_{scene}.unified-header-searchbox.exp | /xxxxx.page_flashsale.unified-header-searchbox.exp |
| 搜索框点击 | /{spma}.page_{scene}.unified-header-searchbox.clk | /xxxxx.page_flashsale.unified-header-searchbox.clk |

#### 商品卡埋点规范（含广告）

**曝光/点击事件参数**：
```
必埋: spm=a.b.c.d, _p_prod, _p_sku, trackInfo, x_object_type, x_object_id
选埋: _p_slr, _p_shop, spm-url, spm-pre, spm-cnt
广告必埋: utLogMap(json)
广告可选: adProduct, adSubProduct
```

**utLogMap 必埋字段**（服务端透传）：
```json
{
  "position": "定坑位置",
  "is_fix_slot": "0/1",
  "is_wysiwyg": "0/1",
  "item_status": {"is_soldout": "0/1"},
  "reco_reason": "推荐理由文本"
}
```

**exargs 必埋字段**（前端动态上报）：
```json
{
  "tab_index": "tab索引",
  "content_name": "内容名称",
  "data_key": "SG_251212_LazflashOnly",
  "biztype": "业务类型",
  "is_reminder_set": "0/1"
}
```

**arg1命名规范**：
- native: Product_Exposure_Event / Product_Click_Event
- h5: /Product.Exposure.Event / /Product.Click.Event

#### 页面级埋点 (2001)

| 参数 | 说明 | 示例 |
|------|------|------|
| spm-cnt | 当前页面标识，2位 | spma.b |
| spm-url | 来源页面spm（用户从哪个页面来） | spma.b.c.d |
| spm-pre | 二级来源spm（来源页面的上游页面） | spma.b.c.d |

**SPM三件套说明**：
```
用户行为链路：首页 → 搜索结果页 → 当前页面（频道页）

当前页面埋点：
- spm-cnt = spma.b（当前页面）
- spm-url = spma.search.list.1（来源页面：搜索页）
- spm-pre = spma.home.searchbox.1（二级来源：首页搜索框）
```

#### 模块级埋点 (2201/2101)

**Header模块**：返回、More、分享、购物车、规则、搜索框
**Top Module模块**：各类入口模块
**JFY模块**：Tab区、商品区、店铺卡、主题卡

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
utparam-url: 来源页面的utLogMap（json结构，需urlencode）
包含: x_object_type, x_object_id, adProduct, adSubProduct
```

#### 埋点生成规则

1. **页面类功能**：生成2001页面事件，含spm-cnt/spm-url/spm-pre，page格式为/{lp}.lp.exposure
2. **按钮类功能**：生成2101点击事件，含spm三件套，page格式为/{lp}.lp.click
3. **列表/卡片类功能**：生成2201曝光+2101点击事件对
4. **商品卡功能**：按商品卡规范生成，必埋_p_prod/_p_sku/trackInfo/x_object_type/x_object_id
5. **加购类功能**：按商卡直接加购规范生成，注意utparam透传
6. **搜索框功能**：统一走搜索侧埋点arg1，业务域无需自主上报

#### 特殊规则

- 商品报名频道活动类型、BU、category、Discount、Price、是否联盟CPS+商品、是否设置补贴 → **不放入流量埋点**，从商品表关联
- 预约功能（is_reminder_set）目前服务端拿不到状态，只能前端侧埋点击状态
- data_key格式：`国家_创建日期_运营自定义名字`（如 SG_251212_XxxOnly）
- utLogMap嵌套层级需要单独url encode一次

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

| 子模块 | 事件描述 | 事件类型 | page | arg1 | 一级key | 二级key | value | 参数含义 | 备注 |
|--------|---------|---------|------|------|---------|---------|-------|---------|------|
| 页面 | 页面事件 | 2001 | H5: url | spm-cnt | spm-cnt | / | spma.b | 当前页面spma.b，只有2位 | 必埋 |
| | | | | | spm-url | / | url-spma.b.c.d | 来源页面spm | 必埋 |
| | | | | | spm-pre | / | pre-spma.b.c.d | 二级来源spm | 必埋 |
| 优惠券模块 | 曝光 | 2201 | H5: url | /cart.lp.exposure | spm | / | spma.b.c.d | 模块曝光坑位 | 必埋 |
| 优惠券模块 | 点击 | 2101 | H5: url | /cart.lp.click | spm | / | spma.b.c.d | 模块点击坑位 | 必埋 |
| 去结算按钮 | 点击 | 2101 | H5: url | /cart.lp.click | spm | / | spma.b.c.d | 按钮点击坑位 | 必埋 |

**埋点说明**：
- 所有埋点字段必须全部小写
- spm-url/spm-pre 用于流量来源归因
- 商品相关埋点需包含：_p_prod(商品ID)、_p_sku(SKU ID)
- 同一子模块下的不同按钮，在事件描述中区分（如"Boost it按钮点击"、"关闭按钮点击"）

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

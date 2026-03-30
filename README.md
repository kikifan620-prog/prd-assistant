# PRD Assistant

一个用于分析产品需求文档（PRD）的 QoderWork Skill，自动生成流程图、功能清单、数据埋点和原型图。

## 功能

- ✅ **价值完整性检查** - 检查 PRD 是否包含目标用户、预期收益、成功指标等关键信息
- 🔄 **Mermaid 流程图** - 根据业务流程自动生成可视化流程图
- 📋 **功能清单** - 提取需求并生成结构化的功能列表
- 📊 **数据埋点** - 基于功能清单自动生成追踪事件
- 📱 **ASCII 原型图** - 生成字符画形式的页面线框图

## 安装

### 方式 1：QoderWork 内置安装

将 `SKILL.md` 复制到你的 QoderWork skills 目录：

```bash
# macOS/Linux
cp SKILL.md ~/.qoderwork/skills/prd-assistant/SKILL.md

# Windows
copy SKILL.md %USERPROFILE%\.qoderwork\skills\prd-assistant\SKILL.md
```

### 方式 2：Git 克隆

```bash
git clone https://github.com/yourusername/prd-assistant.git
cd prd-assistant
# 然后复制 SKILL.md 到 skills 目录
```

## 使用

在 QoderWork 中，直接粘贴 PRD 内容：

```
背景：当前购物车页面转化率仅12%
目标：将转化率提升到17%
需求：
1. 增加智能优惠券推荐
2. 合并结算步骤
...
```

Skill 会自动：
1. 检查需求价值完整性（缺失则追问）
2. 生成 Mermaid 流程图
3. 输出功能清单表格
4. 生成数据埋点表格
5. 绘制 ASCII 原型图

## 示例

见 [examples.md](examples.md)

## 文件结构

```
prd-assistant/
├── SKILL.md          # 核心 Skill 文件（必需）
├── README.md         # 本文件
├── examples.md       # 使用示例
└── LICENSE           # 许可证
```

## 许可证

MIT License - 自由使用和修改

## 贡献

欢迎提交 Issue 和 PR！

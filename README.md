# Hermes Valuation Skills

一套面向 A 股/H 股的 AI 估值分析 skills，适用于 Hermes Agent 平台。

## 包含的 Skills

### stock-valuation-101（公式驱动估值框架）
- 五公式：DCF / 分子分母归因 / PE / ROE / FCF 三角验证
- 强制前置：自动拉取最新季报 + 搜索财联社业绩预告
- 强制输出：三档买卖区间 + 泡沫预警
- 类型适配：成长型/周期型/项目型/概念溢价型/不可估值型
- 已实盘验证 30+ 标的

### industry-analysis-workflow（行业七步分析法）
- 源自肖璟《如何快速了解一个行业》
- 七步流程：生命周期→护城河→竞争格局→PEST→景气度→估值→金字塔输出

### fire-retire-plan（FIRE 退休规划）
- 4% 法则 + 复利公式 + 指数基金定投

### cangjie-skill（内容蒸馏引擎）
- 把书籍/长视频/播客蒸馏成可调用的 AI skills

## 安装
cp -r skills/* ~/.hermes/skills/

## 使用
"详细估值比亚迪" / "分析光纤行业" / "我想提前退休需要多少钱"

## 免责声明
仅供学习参考，不构成投资建议。MIT License.

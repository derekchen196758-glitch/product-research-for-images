---
name: product-research-for-images
description: 美工产品调研助手 - 根据用户提供的ASIN，使用Sorftime MCP进行产品调研，生成汇总型CSV报告（核心卖点/用户痛点/图片建议方向/目标客户），供后续ChatGPT出图方案使用。触发条件：用户提供产品ASIN并要求调研、生成CSV、产品分析等。**独立运行**，不依赖其他任何Skill。
---

# 美工产品调研助手

> **独立性说明：** 本Skill只依赖 Sorftime MCP 的 `product_detail` 和 `product_reviews` 两个工具，不依赖其他任何Skill。只要当前环境有 Sorftime MCP 即可直接使用。

## 工作流

### Step 1: 接收ASIN

用户提供1-3个参考ASIN。如果没有指定站点，默认使用 US 站点。

### Step 2: 获取产品详情

对用户提供的ASIN，调用 Sorftime MCP `product_detail` 工具：

```
mcp__Sorftime MCP__product_detail
参数: { "asin": "<ASIN>", "amzSite": "US" }
```

从返回结果中提取以下信息：
- `title` — 产品标题
- `brandName` — 品牌名
- `starRating` / `ratingsCount` — 评分和评论数
- `monthSales` — 月销量
- `price` — 当前价格
- `mainFeatures` — 产品五点描述（核心卖点来源）
- `categoryName` — 所属类目

> 如果 `mainFeatures` 为空，从 `title` 中推断产品核心功能。

### Step 3: 获取用户评论

调用 Sorftime MCP `product_reviews` 工具获取正评和负评：

**正评（4-5星）：**
```
mcp__Sorftime MCP__product_reviews
参数: { "asin": "<ASIN>", "amzSite": "US", "reviewType": "Positive" }
```

**负评（1-3星）：**
```
mcp__Sorftime MCP__product_reviews
参数: { "asin": "<ASIN>", "amzSite": "US", "reviewType": "Negative" }
```

从评论中提取：
- **正评** → 用户认可的功能/卖点（佐证核心卖点）
- **负评** → 用户痛点/常见问题（提取前3条）

### Step 4: 分析并生成CSV

基于以上数据，生成 CSV 文件。格式为 **2列**：

- **A列（标题）**：4行固定标题
- **B列（内容）**：对应的提取内容

| A列（标题） | B列（内容） |
|:-----------:|:-----------:|
| 核心卖点（高置信度） | 从五点描述+正评提取的最多3条核心卖点，每条带来源标注 |
| 用户痛点 | 从负评提取的最多3条用户痛点 |
| 图片建议方向 | 基于品类特点和卖点给出的制图建议 |
| 目标客户 | 基于产品推测的主要受众画像 |

CSV 文件保存到工作目录，命名为 `ProductResearch_<ASIN>_YYYY-MM-DD.csv`。

### Step 5: CSV输出示例

```csv
核心卖点（高置信度）, 内容
用户痛点, 内容
图片建议方向, 内容
目标客户, 内容
```

**实际填充示例：**
```csv
核心卖点（高置信度）, ①内置磁铁吸附金属杂质(五点)②高密度滤纸(五点)③易安装好评多(好评)
用户痛点, ①塑料接口易裂②滤清器端口变形③管夹质量差
图片建议方向, 白底主图突出金属外壳质感+接口细节；场景图展示安装在摩托车/ATV上的状态；尺寸图标注3/16英寸接口规格
目标客户, 摩托车/ATV车主 / 小型发动机用户（割草机/发电机/小型拖拉机）
```

> **提示：** 在 Excel 或 WPS 中打开 CSV 后，选中两列设置「自动调整列宽」+「自动换行」即可获得最佳阅读效果。

## 红线规则

1. **卖点真实性**：只输出有明确文本依据的高置信度卖点。来源必须是 `mainFeatures` 原文或正评原文，不可自行推理虚构
2. **来源标注**：每条卖点末尾标注来源 `(五点)` 或 `(好评)`
3. **不确定标注**：任何不确定的信息标注 `【需确认】`
4. **文件命名**：`ProductResearch_<ASIN>_YYYY-MM-DD.csv`
5. **品牌侵权红线**：图片建议方向中禁止出现任何竞品品牌Logo、品牌图标、品牌名称的视觉展示。只能用「适配XX机型」「兼容XX型号」等文字描述替代。因建议侵权内容导致的一切后果由建议输出方承担

## 处理约定

- **独立运行**：本Skill不依赖其他任何Skill，只需 Sorftime MCP 可用即可
- **直接执行**：接收ASIN后直接按工作流执行，不询问用户确认
- **多ASIN处理**：如果用户提供多个ASIN，以第一个ASIN为主进行分析，其余ASIN的数据用于补充参考
- **数据不足处理**：如果 `product_detail` 或 `product_reviews` 返回数据不足，对应行标注 `【数据不足】`，不要自行虚构
- **输出格式**：严格按2列CSV输出（标题列 + 内容列），不要额外列或备注行

## 与SOP的衔接

生成的CSV文件直接拖入ChatGPT，配合SOP第6章的Prompt模板使用：
- 配件产品：使用「配件产品 Prompt 模板」
- 普通产品：使用「普通产品 Prompt 模板」
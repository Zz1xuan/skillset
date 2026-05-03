---
name: tripstar-platform-aggregator
description: 聚合多平台旅行内容并生成结构化行程规划的 skill。适用于用户提到旅行规划、旅游攻略、行程安排、景点推荐、避坑、美食路线、住宿建议、预算估算，或明确要求搜索小红书、抖音、高德、飞猪、Google Maps、Google Travel、马蜂窝、大众点评、携程等平台并汇总为可执行旅行方案时触发。
---

# TripStar Platform Aggregator

在 Minis 中执行“多平台旅行信息检索 → 去重提纯 → 行程规划 → Markdown 输出”的工作流。目标是复用 TripStar 风格的核心决策逻辑，而不是复刻完整前后端项目。

## 适用场景

当用户要：
- 做目的地旅行规划
- 搜索小红书/抖音等平台攻略
- 汇总多个平台的景点、美食、住宿、交通信息
- 生成 1~N 天行程
- 输出预算、预约提醒、避坑建议
- 导出 Markdown 或 HTML 旅行页

## 默认原则

- 默认优先“公开可访问 + 稳定”的信息源。
- 不把单一平台当唯一真相来源；至少交叉 2~3 类来源。
- 平台受限时自动降级到网页搜索、地图、官方旅游站、点评站。
- 对小红书、抖音类平台，只承诺“尽力提取公开可见内容”，不要承诺稳定全量抓取。
- 默认输出 Markdown；只有用户明确要求时才输出 HTML 或 JSON。

## 输入信息

尽量从用户原话中提取；缺少关键字段时再补问。

核心字段：
- destination
- date_range 或 days
- travelers
- budget
- interests
- transport_preference
- hotel_preference
- pace_preference
- food_preference
- language

可选字段：
- must_visit
- avoid_list
- child_or_elderly_constraints
- accessibility_needs
- photo_spot_preference
- shopping_preference
- visa_or_cross_border_notes

## 工作流

### 1. 解析需求
把自然语言整理成结构化 brief：目的地、天数、预算、人群、偏好、约束、输出语言。

### 2. 选择信息源组合
按目的选择来源：
- 景点分布 / 顺路关系：高德、Google Maps
- 真实体验 / 避坑 / 出片：小红书、抖音、马蜂窝
- 吃喝：大众点评、高德、小红书
- 住宿落点：飞猪、携程、Booking、Google Maps
- 开放时间 / 预约：景区官网、地图详情页、官方页面
- 天气：稳定天气源

### 3. 搜索与提取
提取：名称、类型、区域/地址、推荐理由、典型玩法、预计时长、价格区间、预约要求、风险/避坑点、来源平台。

对内容平台重点提取：
- 高频景点
- 高频踩坑词
- 高频预约提醒
- 高频美食店名
- 最佳时段/机位/路线建议

### 4. 去重与可信度分层
把信息分为：
- 官方事实
- 地图事实
- 用户经验
- 模型推断

冲突时优先级：
1. 官方页面
2. 地图/POI 平台
3. 多个用户内容一致结论
4. 单条用户经验
5. 纯模型补全

### 5. 生成旅行方案
至少输出：
- 总体策略
- 每日 itinerary
- 景点顺序与理由
- 餐饮建议
- 住宿区域建议
- 交通建议
- 预算明细
- 预约提醒
- 避坑提示
- 备选方案（如下雨 / 人太多 / 预算超标）

### 6. 输出
- 默认：Markdown 攻略
- 用户明确需要展示页时：单文件 HTML
- 用户明确要求机器可读结果时：JSON

## 小红书 / 抖音增强模式

当用户希望提升平台公开内容读取成功率时，可通过环境变量提供登录态：
- `XHS_COOKIE`
- `DOUYIN_COOKIE`

使用原则：
- 仅在用户明确希望启用增强模式时使用。
- 仅用于请求对应站点的公开可见内容，不承诺访问私有或受限数据。
- 若环境变量不存在，自动回退到公开搜索 + 地图 + 点评 + 官方页。
- 不输出 cookie 原文，不打印环境变量值。

缺失时可提示用户设置：
- [Set XHS_COOKIE](minis://settings/environments?create_key=XHS_COOKIE&create_value=&create_note=%E5%B0%8F%E7%BA%A2%E4%B9%A6%E7%BD%91%E9%A1%B5%E7%99%BB%E5%BD%95%20Cookie%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%97%85%E8%A1%8C%20skill%20%E5%A2%9E%E5%BC%BA%E6%A3%80%E7%B4%A2)
- [Set DOUYIN_COOKIE](minis://settings/environments?create_key=DOUYIN_COOKIE&create_value=&create_note=%E6%8A%96%E9%9F%B3%E7%BD%91%E9%A1%B5%E7%99%BB%E5%BD%95%20Cookie%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%97%85%E8%A1%8C%20skill%20%E5%A2%9E%E5%BC%BA%E6%A3%80%E7%B4%A2)

## 参考文件

按需读取：
- `references/platforms.md`
- `references/planning-strategies.md`
- `references/xhs-douyin-enhancement.md`

## 工具建议

- 需要网页搜索时，优先使用 `web-search` skill。
- 需要 HTML 展示时，优先使用 `generative-ui-minis` skill。
- 需要读取页面正文、抓公开链接、做多页面汇总时，使用浏览器工具或 `minis-browser-use`。
- 如需长期复用模板，可把 Markdown/HTML 写到 `/var/minis/workspace/`。

## 不做的事

- 不承诺绕过平台风控。
- 不把私有/需登录受限内容当作默认可访问数据源。
- 不把实时价格、余票、营业时间说成绝对准确，除非已明确核实。
- 不为了“像原项目”而引入不必要的复杂前后端结构。

# CU Lines 报告仓库

集装箱运输数据查询与分析工具。

## 页面导航

| 页面 | 说明 |
|------|------|
| [vvd_query.html](./vvd_query.html) | **VVD 航次查询** — 按航次号查询 Daily Booking 数据，含 POL/POD breakdown、Direct/T/S 分类、CSV 导出 |
| [index.html](./index.html) | 旧版查询页面 |
| [ilue2612w_freight_report.html](./ilue2612w_freight_report.html) | 单航次运价报告示例 |

## 数据源

- **FineBI Daily Booking** — 每小时从 FineBI 自动抓取最新数据
- **数据量**: ~32,000+ 条 Booking 记录
- **更新频率**: 每小时整点自动刷新

## 技术特点

- 数据预聚合后内嵌 HTML，无需服务器，离线可用
- 深色/浅色主题切换，自动记住用户偏好
- 支持模糊匹配、搜索历史、快捷芯片

## 项目文档

- [PROJECT_KNOWLEDGE.md](./PROJECT_KNOWLEDGE.md) — 完整项目知识库（系统架构、配置、功能规范、操作流程）

---

*Auto-deployed by WorkBuddy automation*

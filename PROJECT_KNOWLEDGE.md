# CU Lines Daily Booking — 项目知识库

> 最后更新: 2026-04-14

---

## 一、系统架构

```
FineBI (数据源)
    ↓ fetch_daily_booking.py (Playwright 自动化)
daily_booking_latest.xlsx (原始数据, ~3.9 MB, 32k+ 行)
    ↓ generate_embedded.py (Python 预聚合)
vvd_query.html (~750 KB, 内嵌聚合数据)
    ↓ deploy_to_github.py (GitHub API)
GitHub Pages (bobwzw2.github.io/culines-reports/)
```

每小时整点自动执行完整链路。

---

## 二、数据抓取系统

### 脚本
- **抓取**: `fetch_daily_booking.py` — Playwright 登录 FineBI → 导航到 Daily Booking → 设置日期筛选 → 导出 Excel
- **聚合**: `generate_embedded.py` — 读取 Excel → 按 VVD 预聚合 → 生成自包含 HTML
- **部署**: `deploy_to_github.py` — 通过 GitHub API 推送 HTML 到仓库

### 配置
- **Python 环境**: `C:\Users\bobwu\.workbuddy\binaries\python\envs\default\Scripts\python.exe`
- **依赖**: playwright 1.58.0, pandas 3.0.2, openpyxl 3.1.5
- **FineBI 登录**: https://bi.culines.com/decision/login (用户名: LH.TRADE)

### 自动化
- **ID**: daily-booking
- **频率**: 每小时整点 (`FREQ=HOURLY;BYMINUTE=0`)
- **工作目录**: `C:/Users/bobwu/WorkBuddy/20260409174735`
- **状态**: ACTIVE

### 日期范围
- ETD 前推 2 个月至后推 1 个月，动态计算

### 数据列
`CUL CODE`, `POL`, `POD`, `FIRST LANE`, `TRUNK LANE`, `Booking`, `ETD`, `20ft`, `40ft`, `40RF`, `Container Weight`

---

## 三、数据预聚合方案

### 核心原则
1. **禁止后端/服务器** — 所有数据内嵌到 HTML，离线可用
2. **必须预聚合** — 绝不嵌入 3 万+ 原始记录，浏览器会卡死
3. **聚合粒度**: 按 VVD 分组 → POL 汇总 → POD breakdown

### 嵌入数据结构
```json
{
  "vvds": {
    "VVD_NAME": {
      "polSummary": [
        {
          "etd": "2026-04-15",
          "pol": "CNSHA",
          "firstLane": "REX",
          "trunkLane": "REX",
          "totalTeu": 320,
          "totalCount": 85,
          "totalGrossTon": 4800.5,
          "podList": [
            {"pod": "USLAX", "teu": 120, "count20": 40, "count40": 40, "countRF": 5, "totalWeight": 1800.0},
            ...
          ]
        }
      ],
      "totalTeu": 6500,
      "totalCount": 1500,
      "totalGrossTon": 98000.0,
      "podCount": 25,
      "polCount": 12
    }
  },
  "topVVDs": [{"vvd": "RACE2616W", "count": 1470, "teu": 6567}, ...],
  "totalVVDs": 623,
  "laneVVDs": {"REX": [{"vvd": "RACE2616W", "teu": 6567}, ...]},
  "generatedAt": "2026-04-14T13:00:00",
  "nextRefresh": "14:00"
}
```

### 压缩效果
- 原始数据: ~19.8 MB
- 聚合后: ~720 KB (缩小 27 倍)
- 浏览器秒开，无需等待

---

## 四、VVD 查询页面功能

### 基本查询
- 输入 VVD（航次号）进行精确或模糊匹配
- 输入 Lane（航线代码）筛选 VVD 下拉列表
- Lane 输入新值时 VVD 自动清空

### 查询结果展示
- **统计卡片**: Trunk Lane / 总货量 TEU / 总重 / POL 数 / POD 数
- **POL 汇总表**: ETD / POL / FIRST LANE / TRUNK LANE / 货量 TEU / Booking 数
  - FIRST LANE 列始终显示
  - Direct（FIRST LANE = TRUNK LANE）绿色标签，T/S 紫色标签
  - Direct 按 ETD↑ → POL A-Z → TEU↓ 排序
  - T/S 按 POL A-Z → ETD↑ 排序
  - 合计行与"货量 TEU"列对齐
- **POD 明细展开**: 点击 POL 行展开 POD breakdown
  - POD 标签 / 20' / 40' / TEU / 总重 / 均重+占比柱状图
  - table-layout:fixed + colgroup 保证列宽对齐
- **POD 汇总表**: 可点击展开 POL 明细（POD→POL 反向索引）

### 交互功能
- **快捷芯片**:
  - 最近搜索（最多 8 条，localStorage 持久化，去重+最新优先）
  - 热门航次（top 5，按 TEU 降序）
- **CSV 导出**: 一键导出查询结果
- **主题切换**: 深色/浅色主题，localStorage 记忆
- **自动刷新提示**: 显示下次数据更新时间（下个整点）

### UI 规范
- 全站 14px 基础字号
- 页面全宽（无 max-width 限制）
- CSS 变量实现主题切换
- 响应式布局

---

## 五、故障恢复

- 导出失败时回退到 HTML 表格抓取
- 自动保存诊断截图到 `data/debug_*.png`
- 错误日志输出到控制台

---

## 六、部署信息

- **GitHub 仓库**: BOBWZW2/culines-reports
- **GitHub Pages**: https://bobwzw2.github.io/culines-reports/vvd_query.html
- **部署方式**: GitHub Contents API (PAT)
- **批量部署**: Git Trees API (`batch_deploy.py`)

# 甘特图工具

面向学生的轻量甘特图制作工具，单 HTML 文件，双击即用。完全离线，无需联网。

## 交付形式

```
gantt-chart-tool/
└── gantt-pro.html   ← 唯一交付文件（~556 KB，含所有依赖内联）
```

## ⚠️ 阅读文件注意事项

`gantt-pro.html` 共约 1945 行，其中大部分是内联的第三方库，**不要读取这些区域**：

| 行范围 | 内容 | 操作 |
|--------|------|------|
| 8–16 | 内联字体 CSS（base64 WOFF2） | 跳过 |
| 17–36 | html2canvas 1.4.1 压缩代码 | 跳过 |
| 37–304 | jsPDF 2.5.1 压缩代码 | 跳过 |
| **305–647** | **App CSS** | 读取 |
| **648–1018** | **App HTML（toolbar、panel、modal）** | 读取 |
| **1019–1943** | **App JavaScript** | 读取 |

读取 App JS 时按需定位到具体函数，不要整段读取：

| 行范围 | 内容 |
|--------|------|
| 1020–1044 | 常量、全局变量 |
| 1045–1072 | Undo / Redo |
| 1073–1082 | DOMContentLoaded 初始化 |
| 1083–1104 | localStorage 持久化 |
| 1105–1129 | 工具函数（uid、parseDate、escXml 等） |
| 1130–1157 | Section 管理（addSection / deleteSection / toggleSection） |
| 1158–1187 | buildRenderList() |
| 1188–1199 | renderAll() |
| 1200–1311 | renderTaskList() + taskCardHTML() |
| 1312–1627 | renderGantt()（SVG 渲染核心） |
| 1628–1655 | Tooltip |
| 1656–1663 | jumpToToday() |
| 1664–1674 | addDuration()（日期快捷按钮） |
| 1675–1709 | 任务卡片拖拽换分组 |
| 1710–1838 | Modal（openModal / saveTask / deleteTask / hasCycle） |
| 1839–1867 | 项目文件（newProject / saveProject / loadProject） |
| 1868–1909 | 导出（svgToCanvas / exportPNG / exportPDF） |
| 1910–1933 | Helpers（togglePanel / changeZoom / 键盘快捷键） |
| 1934–1943 | Tutorial（openTutorial / closeTutorial / setTutLang） |

## 技术栈

- **纯 HTML/CSS/JS**，无构建工具，无 npm
- **SVG** 渲染甘特图（原生，无第三方图表库）
- **html2canvas 1.4.1** + **jsPDF 2.5.1**（已内联）—— 导出用
- **localStorage** —— 自动保存项目数据
- **DM Sans**（已内联为 base64 WOFF2）—— UI 字体

## 核心数据结构

```js
state = {
  projectName: String,
  sections: [{ id, name, collapsed }],
  tasks: [{
    id, name, startDate, endDate,        // 'YYYY-MM-DD'
    assignee, color,                      // color 是 hex 颜色
    dependencies: [taskId, ...],          // 前置任务 id 数组
    progress,                             // 0–100
    milestone,                            // boolean
    notes,                                // string
    sectionId                             // section.id 或 null
  }]
}
```

## 主要功能

| 功能 | 实现位置 |
|------|----------|
| 渲染甘特图 | `renderGantt()` L1312 — 纯 SVG 拼接，cp/bp/lp 三层数组 |
| 缩放时间轴 | `ZOOM_LEVELS[]` + `zoomIdx` + `changeZoom()` |
| 导出 PNG/PDF | `svgToCanvas()` L1868 — 序列化 SVG 到 Canvas |
| 自动保存 | `scheduleSave()` L1083 — 500ms debounce → localStorage |
| 循环依赖检测 | `hasCycle()` L1810 — DFS 图遍历 |
| 分组/Section | `buildRenderList()` L1158，支持折叠 |
| 拖拽换分组 | `onDrop()` L1675，HTML5 Drag & Drop |
| Undo / Redo | `snapshot()` L1045，JSON 快照，60步 |

## SVG 渲染层级（renderGantt 内部）

```
cp[]  背景 → 月份条 → 行背景 → 列网格（周末/今日高亮）→ 水平分割线
bp[]  任务条（进度填充、闪光、逾期边框、文字标签）
lp[]  标签列（任务名、Section 折叠按钮）
最终：cp.push(...bp)，lp 写入 labelSvg
```

## 缩放级别

```js
ZOOM_LEVELS = [月览(7px), 双周(14px), 标准(26px), 周视图(44px), 日视图(68px)]
```

## 开发注意

- `renderAll()` 是统一入口，每次数据变更后调用
- `snapshot()` 必须在 state 变更**之后**调用（否则 redo 失效）
- `gridH`（网格高度，止于最后一行底部）≠ `chartH`（SVG 总高度，填满视口）
- Section 折叠 toggle 在 `labelSvg.querySelectorAll('.sec-toggle')` 中事件重绑
- 任务条双击 → `openModal(id)`；拖拽换分组仅限左侧 panel 的卡片列表
- 导出时隐藏 `#todayMarker`，避免出现在 PNG/PDF 中

## 分发

直接发送 `gantt-pro.html`，完全离线可用，无需联网。

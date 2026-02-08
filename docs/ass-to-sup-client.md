# ASS → SUP Client Application Specification

## 目标 (Goals)

- 支持导入 ASS 字幕文件并正确渲染：字体、描边、阴影、渐变、
  旋转/缩放、动画、位移、打字机特效、卡拉 OK 等。
- 将渲染后的逐帧/逐事件字幕输出为 SUP (Blu-ray PGS) 位图字幕流。
- 在本地桌面端可视化预览与导出，离线工作。

## 非目标 (Non-goals)

- 不提供在线字幕协作或云端同步。
- 不内置复杂剪辑功能（可选：与外部播放器预览集成）。

## 用户流程 (User Flow)

1. 打开应用 → 选择 ASS 文件。
2. 解析字幕样式与事件 → 生成渲染时间轴。
3. 用户预览渲染结果（可选择参考视频轨）。
4. 点击“导出 SUP” → 输出 .sup 文件与日志。

## 技术路线 (Architecture)

### 1. ASS 解析与渲染

- 解析库建议：
  - C++/Rust: libass（成熟、兼容性高）
  - JavaScript: libass-wasm（可在桌面 WebView 中运行）
- 重点要求：
  - 字体加载与 fallback
  - 字体粗细、斜体、描边、阴影
  - 特效标签：\blur, \fade, \move, \pos, \org, \t 等
  - 卡拉 OK / 逐字特效

### 2. 渲染目标

- 以帧为单位渲染为 RGBA 位图（或 8-bit 索引 + 调色板）。
- 统一时基：
  - UI 中显示为时间轴。
  - 与视频 FPS 对齐（可选导入视频以锁定 FPS）。

### 3. SUP (PGS) 输出

- 生成 PGS 结构：
  - PCS（Palette Composition Segment）
  - WDS（Window Definition Segment）
  - PDS（Palette Definition Segment）
  - ODS（Object Definition Segment）
- 输出策略：
  - 每条字幕事件生成一条 PGS object。
  - 若存在动画，则拆分为多个连续 object。
  - 颜色深度：4-bit / 8-bit（推荐 8-bit 调色板）。

## UI 设计 (UI)

- 左侧：字幕事件列表（可按时间/样式过滤）。
- 中央：预览画布（支持缩放、显示安全框）。
- 右侧：字幕样式检查器（样式/字体信息、渲染警告）。
- 顶部：导入/导出工具栏。

## 渲染准确性要求

- 必须支持 ASS 的所有主流标签。
- 对于超出范围的特效，提供明确警告。
- 字体缺失时提供替代字体并记录日志。
- 正确支持多行、自动换行、对齐方式。

## 兼容性与性能

- 在 1920x1080、30fps 下处理 1 小时字幕文件导出时间 < 5 分钟。
- 允许用户选择低/中/高渲染质量。

## 可能的技术栈

### 桌面端
- Tauri (Rust + Web UI)
- Electron (Node + Web UI)

### 渲染核心
- libass + Rust/C++ 封装
- 或 WebAssembly 版 libass

### SUP 生成
- 自实现 PGS writer（参考 Blu-ray 规范）
- 或封装开源 PGS writer（需确认许可）

## 验证与测试

- 单元测试：ASS 标签解析、PCS/WDS/PDS/ODS 结构生成。
- 回归测试：使用标准 ASS 样例渲染并比对基准 SUP 输出。
- 视觉回归：导出后与原 ASS 渲染结果一致。

## 交付物

- 可执行应用（Windows/macOS/Linux）。
- 示例数据（ASS 样例 + 输出 SUP）。
- 导出日志与错误报告。

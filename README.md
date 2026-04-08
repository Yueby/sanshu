# 三术 (Sanshu) — Fork 二开记录

基于 [yuaotian/sanshu](https://github.com/yuaotian/sanshu) 的二次开发分支。

## 二开内容

- 本地提示词增强，移除外部服务依赖
- 结构化响应（用户输入/选项/图片/文件/条件偏好分区传递）
- 图片 Badge 索引序号 + 文件名、截图自动唯一命名、拖拽自动聚焦
- `prefill` 弹窗预填充
- Markdown 渲染对齐 Naive UI + 代码块复制修复
- Fork 品牌重塑、UI 标准化、本地字体、置顶常驻、Toast/Modal 层级修复
- 关闭按钮双色反馈（黄色警告 → 红色确认）
- `zhi` 子代理约束 + 抗上下文压缩 + 强制 `project_root_path`
- 执行偏好独立 Content::text、连续取消关闭保护、统一弹窗协议
- WebView2 多实例黑屏修复、Context7 重定向处理
- 自定义提示词独立存储 + 条件性开关 + MCP 工具联动
- 自动更新（CDN 检查 + 标题栏按钮 + 下载进度 + auto-tag 发布）

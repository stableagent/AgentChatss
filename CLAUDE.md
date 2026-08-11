# AgentChat 项目指令

## AgentChat 通用规则（适用于 OneWeb / IndependentTasks / WebSubAgent）

**网页 AI 无法访问本地文件系统。** 当用户问题涉及本地文件、项目代码、目录内容、图片等本地资源时，Claude Code 必须在调用 `node index.js` 之前：
1. 读取本地文件/代码 → 提取关键内容
2. 将内容整理进 prompt 正文（而非只传文件路径）
3. 然后再调用 index.js 发送

禁止：把文件路径写入 prompt 指望网页 AI 自己去读（它读不到）。

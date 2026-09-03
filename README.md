# vibe-frontend-handbook

一个面向 AI 产品经理与 Codex 的 Agent Web 前端开发 Skill：把产品 PRD、设计资料与后端契约转成前端技术适配声明、分阶段真实业务闭环和浏览器验收。

它源自《AI Agent 产品 Vibe Coding 通用前端技术栈手册 V1.0》，重点保留：

- PRD、设计稿、接口契约和现有代码的优先级；
- 开工前《前端技术适配声明》和分阶段开发文档；
- Agent 长任务、流式输出、等待确认、失败、断线与恢复；
- 设计系统、中文字体、响应式、移动端与无障碍；
- Lint、类型、测试、构建与真实浏览器验收；
- 给非技术产品经理逐步、可操作的验收说明。

## 安装

在 Codex 中发送：

```text
使用 $skill-installer 从 https://github.com/daaaayuuuu/vibe-frontend-handbook 安装这个 Skill。
```

也可以手动安装：

```bash
git clone https://github.com/daaaayuuuu/vibe-frontend-handbook.git
cp -R vibe-frontend-handbook ~/.codex/skills/vibe-frontend-handbook
```

重新打开 Codex 后即可使用。

## 使用示例

```text
使用 $vibe-frontend-handbook 阅读这份 AI Agent 产品 PRD、设计稿、接口文档和当前前端项目。
先输出《前端技术适配声明》和第一阶段前端开发文档，暂时不要修改代码。
```

```text
使用 $vibe-frontend-handbook 按已确认的阶段文档实现第一条真实后端闭环。
完成自动检查，并在 390、768、1280 和 1440 宽度进行真实浏览器验收。
```

## 目录

```text
.
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── agent-interactions.md
    ├── design-quality.md
    ├── document-templates.md
    ├── frontend-architecture.md
    └── testing-browser-acceptance.md
```

## 适用范围

适合 Web 端 AI 对话、内容生成、工作流、自动化、多媒体生成、内部工具、运营后台、创作工作台和 Agent 控制台，兼顾从零开发与已有 React/Next.js 项目迭代。

原生 iOS/Android、鸿蒙、小程序、桌面客户端、浏览器插件、强实时协作、3D、游戏、超大数据可视化和强监管系统可以复用状态、安全与验收原则，但需要另做技术适配，不能机械套用默认 Web 技术栈。

## 设计原则

Skill 使用“短入口 + 按场景加载参考文件”的结构。它不会把固定版本、Mock 页面、HTTP 200 或构建成功冒充为完整交付，也不会在用户只要求分析时擅自修改代码或部署。

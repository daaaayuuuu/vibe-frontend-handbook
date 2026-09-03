---
name: vibe-frontend-handbook
description: 将 AI Agent 产品 PRD、设计资料和后端契约转成可运行的 Web 前端适配声明、分阶段实现与真实浏览器验收。适用于从零构建或迭代 Agent 工作台、对话、生成和自动化产品；不用于普通非 Agent 页面、原生 App、小程序或仅做后端开发。
---

# Vibe Frontend Handbook

让 AI Agent 产品前端不只“页面能打开”，而是清楚呈现长任务、流式输出、用户确认、失败与恢复，并通过真实后端闭环和浏览器验收。面向不需要自行做框架选型的产品经理，用产品语言解释少量关键取舍。

## 权威顺序与执行边界

资料冲突时按以下顺序判断：用户本轮明确目标 → 已确认 PRD 与验收标准 → 已确认设计稿和交互说明 → 已确认接口契约与后端真实能力 → 现有代码与工程约束 → 本 Skill 默认建议。

- PRD 决定用户任务、业务范围和成功标准；设计资料决定已确认的视觉与交互；后端契约决定真实数据和能力。
- 安全、隐私、加载/空/失败/恢复状态、基本移动可用性和真实浏览器验收不能因 PRD 未写而省略。
- 用户只要求分析、审查或规划时，不修改代码。只有明确要求构建、实现或修改时才进入实现。
- 外部发布、生产部署、付费调用、发送、删除或覆盖仍需对该动作的明确授权。
- 不把 Mock、静态截图、构建成功或页面 HTTP 200 描述成真实前后端闭环已经完成。

## 工作流

### 1. 检查输入与现有项目

读取 PRD、后端手册或技术适配声明、接口契约、阶段文档、设计稿或参考、项目规则、README 和代码。检查：

- 用户、核心任务、真实结果和当前阶段业务目标；
- 目标终端、页面清单、主流程、产物类型和 Agent 状态；
- 后端真实接口、鉴权、任务 ID、状态、流式协议、错误与恢复能力；
- 当前框架、目录、依赖、包管理器、脚本、未提交修改、运行版本和端口；
- 已有设计系统、组件、字体和本地化约束。

缺项只有在会明显改变产品方向、体验、范围、成本、安全或工期时才阻塞询问。其余采用合理默认并记录。已有项目优先延续合理架构，不为匹配示例目录大规模迁移。

### 2. 先输出《前端技术适配声明》

正式编码前使用 [references/document-templates.md](references/document-templates.md) 的声明模板，给出一个推荐方案，不让产品经理在多套技术中盲选。重点确认：

- 本阶段用户能完成的业务闭环；
- 任务状态、流式或轮询、用户确认、取消、重试和恢复；
- 页面、接口、产物、设计来源与响应式目标；
- 对默认方案的必要偏离及影响。

没有关键待确认事项且用户已授权实现时可继续。会显著改变目标、范围、成本、安全、核心视觉方向或后端契约时等待确认。

### 3. 按闭环编写阶段文档

使用 [references/document-templates.md](references/document-templates.md) 的阶段模板。不要按“先写完所有页面，再统一接接口”拆阶段；每阶段至少交付一条可运行、可查看、可恢复、可验收的用户链路。

默认阶段可以合并，但顺序为：

1. 适配与契约；
2. 可运行骨架与代表页；
3. 第一条真实后端闭环；
4. 完整 Agent 交互与恢复；
5. 响应式、无障碍、性能和发布准备。

### 4. 使用最小复杂度实现

新 Web MVP 默认使用当前稳定、受维护的 Next.js App Router、React、TypeScript strict、Tailwind CSS + 语义设计变量、npm、Vitest + React Testing Library、Playwright、ESLint、类型检查和生产构建。记录并锁定实际版本，不永久锁死手册中的参考大版本。

- 先复用已有组件，再用原生 HTML 和轻量组件；复杂交互才引入成熟无障碍基础组件。
- React 自带状态优先；只有服务端缓存复杂时用 TanStack Query，远距离临时客户端状态复杂时用 Zustand。
- 复杂表单、虚拟列表、图表、编辑器、国际化、离线、监控和埋点都由明确需求触发，不为“栈完整”预装。
- 页面路由 → 业务 Feature → API/Stream/Auth 适配层 → 后端；页面组件不散落 `fetch` 或供应商字段。

详细架构、API、表单和产物规则见 [references/frontend-architecture.md](references/frontend-architecture.md)。

### 5. 把 Agent 状态当作产品功能

后端任务状态和业务结果是最终事实来源。URL 保存可分享、可恢复的任务位置；Local Storage 只放非关键偏好，不能冒充任务结果。

至少显式处理：`idle`、`submitting`、`queued`、`running`、`streaming`、`waiting_user`、`succeeded`、`partially_succeeded`、`failed`、`cancelled`、`disconnected`、`stale`。未知后端状态安全兜底，不静默映射成成功。

每个状态告诉用户：正在发生什么、需要做什么、接下来会怎样。分别设计 loading、empty、failed、permission denied 和 not found；数据未返回时不显示“暂无数据”。

流式、语音、多媒体、确认、取消、幂等、断线与恢复见 [references/agent-interactions.md](references/agent-interactions.md)。

### 6. 先确认视觉方向，再铺开

视觉来源优先级为：已确认设计系统或设计稿 → PRD 品牌与场景 → 已确认参考产品 → 可合法使用的第三方设计系统 → 中性默认方案。

有设计稿时忠实实现结构、尺寸、字体、图标、资产、状态和响应式意图；不能用临时远程资源支撑长期交付。没有设计稿时，先定义简短视觉方向并完成一个代表页，在大面积复用前让关键视觉选择可见。

设计系统、中文字体、组件层级、响应式、无障碍、安全与性能规则见 [references/design-quality.md](references/design-quality.md)。

### 7. 自动检查与真实浏览器验收缺一不可

每个可交付阶段运行项目已有的等价命令，默认至少覆盖 lint、TypeScript、测试和生产构建。然后真实启动正确项目，在真实浏览器验证：

- 页面身份、核心流程与真实后端；
- Console 与 Network；
- 加载、空、禁用、失败、等待确认、成功与恢复；
- 桌面和移动目标宽度、弹窗、侧栏、菜单、滚动和键盘；
- 图片、视频、音频、上传和下载；
- 刷新、返回、断网、重连和防重复提交。

详细门槛与交付格式见 [references/testing-browser-acceptance.md](references/testing-browser-acceptance.md)。

## 与产品经理沟通

- 一次只给当前需要执行的步骤：打开哪个地址、点击什么、输入什么、看到什么算通过。
- 先给代表页确认颜色、圆角、密度和字体，再复用到全站。
- 语音、麦克风、录音、摄像头等浏览器能力需要真实手机验证时，明确请用户按一条链路操作并反馈现象。
- 每阶段结束时说明已完成、验证结果、未完成、已知问题，以及产品经理下一步只需要做什么。

## 参考文件路由

- 做框架、目录、渲染、路由、状态归属、API、表单或产物设计时，读 [references/frontend-architecture.md](references/frontend-architecture.md)。
- 做长任务、SSE、轮询、语音、多媒体、确认、取消或恢复时，读 [references/agent-interactions.md](references/agent-interactions.md)。
- 实现设计稿、设计变量、字体、图标、响应式、无障碍、安全或性能时，读 [references/design-quality.md](references/design-quality.md)。
- 编写适配声明或阶段文档时，读 [references/document-templates.md](references/document-templates.md)。
- 做自动化、浏览器验收、交付判定或发布准备时，读 [references/testing-browser-acceptance.md](references/testing-browser-acceptance.md)。

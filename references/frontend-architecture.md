# 前端架构、API 与产物

## 决策分层

### A：所有项目必须遵守

- TypeScript strict；核心类型不用 `any` 逃避检查，不可信数据从 `unknown` 缩小。
- 接口调用集中管理，页面组件不散落 `fetch`、鉴权、地址或错误转换。
- 后端是任务状态、权限和业务结果的最终事实来源。
- 明确设计加载、空、成功、失败、等待确认、断线和恢复。
- 密钥、私密 Prompt、管理员凭证和服务端业务规则不进浏览器。
- 核心页面在目标桌面和移动宽度可用，核心操作可由键盘完成。
- 重要提交防重复，重要删除确认；最终权限与幂等由后端执行。
- 代码检查、构建、测试和真实浏览器验收都是交付门槛。

### B：新 Web MVP 默认

- 当前活跃 Node.js LTS、npm、稳定 Next.js App Router 与 React；
- TypeScript strict；
- Tailwind CSS + 语义化 CSS 变量；
- 单一风格的图标库，如 Lucide；
- Vitest + React Testing Library；
- Playwright 覆盖核心用户闭环；
- ESLint、类型检查、测试和生产构建。

手册编写时可验证的参考是 Node.js 22、Next.js 16、React 19、Tailwind CSS 4，但这不是永久安装单。创建项目时选择受维护版本并记录、锁定；已有项目不因参考版本而强制迁移。

### C：真实复杂度触发

| 需求 | 可选方案 | 触发条件 |
| --- | --- | --- |
| 对话框、菜单、下拉等复杂无障碍组件 | Radix UI 或基于它的 shadcn/ui | 复杂组件数量确实增加 |
| 服务端缓存 | TanStack Query | 跨页面共享、失效、分页、预取和重试变复杂 |
| 全局客户端临时状态 | Zustand | 远距离组件共享，URL 和状态提升都不合适 |
| 复杂表单 | React Hook Form + Zod | 动态字段、多步骤或复杂校验 |
| API 代码生成 | OpenAPI Generator 等 | 后端 OpenAPI 稳定且持续维护 |
| 大列表 | 虚拟化工具 | 真实数据量已造成卡顿 |
| 编辑器、图表、画布、流程编排 | 按业务选型 | PRD 确认需要 |
| i18n、PWA、离线 | 成熟专项方案 | 已确认多语言或离线需求 |
| 错误监控、行为分析 | 合规工具 | 真实用户测试或生产，且指标和授权明确 |

## 默认目录与依赖方向

```text
frontend/
├── app/                       # 路由、布局、加载和错误页
│   ├── (workspace)/
│   ├── api/                   # 必要的同源代理或 BFF
│   ├── error.tsx
│   ├── loading.tsx
│   └── layout.tsx
├── components/
│   ├── ui/                    # 基础组件
│   └── layout/                # 顶栏、侧栏、页面容器
├── features/
│   └── task-generation/
│       ├── api/
│       ├── components/
│       ├── hooks/
│       ├── schemas/
│       ├── types.ts
│       └── utils.ts
├── lib/
│   ├── api/                   # 客户端与错误归一化
│   ├── stream/                # SSE 或流式处理
│   ├── auth/
│   ├── media/
│   └── utils/
├── config/
├── styles/
├── tests/
└── e2e/
```

不创建无用目录，不要求已有项目迁移成此结构。依赖方向为：页面路由 → 业务 Feature → API/Stream/Auth 适配层 → 后端。通用 UI 不反向依赖单一业务，页面不理解供应商特殊字段。

## Server 与 Client 边界

Next.js App Router 默认 Server Component。静态说明、服务端首屏数据和无需浏览器事件的展示留在服务端；表单、拖拽、弹窗、实时进度、流式输出和浏览器 API 收敛在必要的 Client Component 边界内。不要因为一个按钮把整页变成客户端组件。

屏幕宽度、Local Storage、浏览器语言等不能直接参与服务端首次渲染。提供稳定首屏默认值，避免 hydration 不一致。CSS、React 状态和持久化偏好必须一致，尤其是移动侧栏和遮罩。

## 四类状态归属

| 状态 | 例子 | 默认事实来源 |
| --- | --- | --- |
| URL 状态 | 任务 ID、页签、筛选 | 路由或查询参数 |
| 服务端状态 | 进度、产物、权限、余额 | 后端 API |
| 页面临时状态 | 输入、弹窗、局部选中 | 组件 State 或局部 Context |
| 用户偏好 | 主题、侧栏、非关键提示 | Local Storage 或用户配置 API |

刷新后根据 URL 或任务列表重新获取后端状态。乐观更新失败时可回滚。默认用 React 自带状态；不要一开始同时引入 Context、Redux、Zustand 与 TanStack Query。

## 集中 API 层

```text
UI 组件
  ↓ 业务函数
Feature API
  ↓ 统一客户端
HTTP / Stream Client
  ↓
Backend API
```

基础 URL、鉴权、超时、错误归一化、请求 ID 和日志集中处理。普通请求与流式请求使用一致的环境与鉴权策略。

每个接口至少明确：方法、路径、请求字段、成功结构、错误码与恢复方式、权限、幂等、超时与重试、任务 ID、状态、产物，以及时间、枚举、金额和文件地址格式。

前端内部可统一为：

```ts
type AppError = {
  code: string;
  message: string;
  userMessage: string;
  retryable: boolean;
  requestId?: string;
  fieldErrors?: Record<string, string>;
};
```

UI 显示 `userMessage`，调试只保留必要的 `code` 和 `requestId`。不显示原始堆栈、供应商错误或敏感请求。

后端 OpenAPI 稳定时可生成类型和客户端；快速变化时维护清晰手写类型。两种方式都保留业务适配层和未知枚举兜底。TypeScript 编译通过不代表后端实际返回可信，关键外部数据仍需运行时校验或安全字段兜底。

浏览器公开配置可使用 `NEXT_PUBLIC_`；API Key、模型密钥、数据库凭证和私密规则绝不能使用该前缀。环境地址集中配置并验证普通与流式链路一致。

## 表单、模型与文件

- 必填、格式、长度和限制在提交前可见；错误靠近字段且不只依赖颜色。
- 提交失败保留输入；多步骤表单保存草稿或明确离开会丢失。
- 默认模型必须存在于可选项。已保存但不可用的模型显示不可用，不静默替换。
- 模型名称、能力、成本提示和限制来自统一配置；任务所用配置版本可追溯。
- 上传前说明格式、数量、大小；显示进度、失败和重传；允许移除未提交文件。
- 后端验证真实文件类型；私有文件不依赖永久公开 URL；大文件按后端契约分片或直传。

## 统一产物模型

不要把页面写死为单一图片结果。按产品需要统一文本、图片、视频、音频、文件和 JSON：

```ts
type Artifact = {
  id: string;
  kind: "text" | "image" | "video" | "audio" | "file" | "json";
  name: string;
  status: "processing" | "ready" | "failed";
  url?: string;
  previewUrl?: string;
  mimeType?: string;
  size?: number;
  createdAt: string;
  metadata?: Record<string, unknown>;
};
```

- 图片有占位、加载失败、放大和下载；视频有封面、加载、失败和控制；音频有时长、播放状态和下载。
- 未知媒体尺寸预留比例；Object URL 用完释放；外链处理过期、跨域和权限失败。
- 生成中的产物不伪装为最终完成。
- 普通界面先展示结果、状态和下一步；请求 ID、原始 JSON 和调试日志放折叠详情或开发模式。

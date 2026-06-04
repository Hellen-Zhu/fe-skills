# 前端 UI 新版本质量门禁与测试分层策略

## 目标

针对前后端分离项目，前端 UI 新版本发布的质量门禁不应该只依赖上线前的全量 UI 自动化，而应该通过分层测试逐级拦截风险：

- 越底层的测试越快、越稳定、越适合频繁运行。
- 越上层的测试越贴近真实用户，但成本更高、数量应该更少。
- 能在低层测试证明的问题，不升级到高层测试。
- 只有真实环境、真实路由、真实页面协作、视觉正确性或业务闭环需要验证时，才进入更高层。

核心原则：

```text
能在组件测试验证的，不升级到集成测试；
只有页面级协作、路由、状态和 API 交互才写集成测试；
只有跨页面业务闭环、真实部署环境和生产冒烟才写 E2E。
```

## 推荐质量门禁

| 阶段 | 门禁内容 | 目的 |
| --- | --- | --- |
| PR / 代码提交 | lint、type check、格式检查、单元测试、组件测试、集成测试 | 拦截低级错误、逻辑回归和主要 UI 行为问题 |
| 构建阶段 | production build、资源大小检查、环境变量校验 | 确认新版本可构建，且包体没有异常膨胀 |
| 接口契约 | OpenAPI / schema 校验、mock 数据校验、前后端类型兼容检查 | 避免字段名、类型、枚举值、错误码不一致 |
| 测试环境 | preview / staging 部署、E2E smoke、关键路径回归 | 验证主要业务流程在部署环境可用 |
| 发布前 | 视觉回归、性能、兼容性、可访问性、安全扫描 | 拦截破版、慢、不可用、安全和体验问题 |
| 上线后 | production smoke、JS error rate、白屏率、接口错误率、核心转化监控 | 控制生产风险，支持灰度和回滚 |

## 测试分层

推荐的测试层次如下：

```text
          E2E / 生产冒烟
       核心用户路径、发布后可用性

        视觉回归测试
   页面破版、错位、响应式、主题和关键状态

          集成测试
 页面 + 路由 + 状态管理 + API mock + 多组件协作

          组件测试
  组件 props、交互、表单、权限、状态展示

          单元测试
 工具函数、hooks、状态逻辑、权限判断、数据转换

          静态检查
 lint、TypeScript、格式、依赖、构建校验
```

一个健康的用例分布可以参考：

| 层级 | 建议占比 | 说明 |
| --- | ---: | --- |
| 单元测试 | 40% | 承接纯逻辑、数据转换、权限判断 |
| 组件测试 | 25% | 承接大部分 UI 行为回归 |
| 集成测试 | 20% | 承接页面级协作、API mock、状态流转 |
| E2E | 10% | 只覆盖核心业务闭环和发布冒烟 |
| 视觉回归 | 5% | 覆盖关键页面和设计系统视觉稳定性 |

比例不是硬规则，重点是 E2E 少而精，组件测试和集成测试承担主要回归责任。

## Pipeline 集成策略

建议拆成三条流水线，而不是每次都全量运行。

### PR Pipeline

目标：快速反馈，阻断明显回归，建议控制在 10-20 分钟内。

```text
install/cache
  -> lint/typecheck
  -> unit test
  -> component test
  -> integration test with mock API
  -> production build
  -> preview deploy
  -> E2E smoke
  -> visual regression for changed/core pages
```

PR 门禁建议：

| 层级 | 是否阻断 | 说明 |
| --- | --- | --- |
| 组件测试 | 阻断 | 失败说明组件用户行为有回归 |
| 集成测试 | 阻断 | 核心页面逻辑、状态或 API mock 交互失败 |
| E2E smoke | 阻断 | 登录、核心链路、关键页面不可用 |
| 视觉回归 | 半自动 | 大面积 diff 阻断，小 diff 人工确认 |

### Nightly Pipeline

目标：覆盖更完整的回归、兼容性和慢性问题。

```text
full component test
  -> full integration test
  -> full E2E regression
  -> cross-browser test
  -> full visual regression
  -> performance check
```

适合放在 nightly 的内容：

- 全量 E2E 回归
- 多浏览器矩阵
- 多 viewport 视觉回归
- 较慢的性能检测
- 更完整的可访问性扫描

### Release Pipeline

目标：确认版本可发布、可观测、可回滚。

```text
production build
  -> deploy staging
  -> API contract check
  -> E2E critical regression
  -> visual baseline approval
  -> accessibility/performance/security check
  -> production deploy
  -> production smoke
  -> monitoring and rollback guard
```

发布门禁建议：

- API contract 不兼容，禁止发布。
- 核心 E2E 失败，禁止发布。
- 核心页面大面积视觉 diff，禁止发布或需要明确审批。
- production smoke 失败，停止扩大流量或触发回滚。
- JS error rate、白屏率、核心接口错误率异常，触发告警。

## 组件测试层

组件测试适合和前端项目代码同仓，并尽量靠近组件源码：

```text
src/features/order/components/OrderTable.tsx
src/features/order/components/OrderTable.test.tsx
```

适合组件测试覆盖：

| 应该测 | 不应该测 |
| --- | --- |
| props 渲染结果 | 内部实现细节 |
| 用户点击、输入、选择、键盘交互 | class 名称是否完全一致 |
| loading、empty、error、disabled 状态 | 第三方组件库自身行为 |
| 表单校验和提交状态 | 后端接口真实可用性 |
| 权限下按钮是否展示 | 复杂跨页面流程 |
| callback / emit 参数 | 页面路由和全局状态编排 |

组件测试用例标准：

- 从用户视角写测试，优先使用 `getByRole`、`getByLabelText`、`getByText`。
- 使用 `userEvent` 或框架等价能力模拟真实交互。
- 断言用户可见结果，而不是私有状态或内部函数调用。
- 每条用例聚焦一个行为。
- 覆盖关键状态，不枚举所有 props 组合。

典型用例：

```text
Given 用户没有编辑权限
When 渲染订单详情组件
Then 不展示“编辑”按钮
```

```text
Given 接口返回空列表状态被传入组件
When 渲染用户表格
Then 展示 empty 状态
```

## 集成测试层

集成测试同样建议和前端项目代码同仓，按业务模块或统一测试目录组织：

```text
src/features/order/pages/OrderListPage.integration.test.tsx
```

或：

```text
tests/integration/order-list.test.tsx
```

集成测试验证的是：

```text
页面 + 路由 + 状态管理 + API mock + 多组件协作
```

适合集成测试覆盖：

| 场景 | 是否适合 |
| --- | --- |
| 页面初始化拉取多个接口 | 适合 |
| 搜索、筛选、分页、排序组合 | 适合 |
| 表单提交成功 / 失败 | 适合 |
| 权限影响多个区域展示 | 适合 |
| 路由参数影响页面状态 | 适合 |
| 多个组件联动 | 适合 |
| 单个组件 props 渲染 | 不适合，放组件测试 |
| 跨系统真实业务闭环 | 不适合，放 E2E |

集成测试用例标准：

- 优先在网络边界 mock API，例如 MSW。
- 使用真实页面、真实 store、真实 provider，除非项目已有稳定替代。
- 数据 fixture 要稳定、清晰、有业务含义。
- 用户操作后，既可以断言页面结果，也可以断言请求参数。
- 覆盖成功路径，也覆盖关键失败路径。

典型用例：

```text
Given 用户进入订单列表页
And API 返回 20 条订单
When 用户选择状态为“待支付”
Then 页面请求带上 status=pending
And 表格只展示待支付订单
```

```text
Given 创建订单接口返回 400
When 用户提交表单
Then 页面展示后端错误信息
And 提交按钮恢复可点击
```

## E2E 测试层

E2E 应该跑在真实部署环境上，例如 preview、staging 或 production smoke。

Pipeline 位置：

```text
build
  -> deploy preview
  -> run E2E smoke against preview URL
```

PR 阶段只跑 smoke：

```text
npm run test:e2e:smoke -- --baseURL=$PREVIEW_URL
```

Release 阶段跑 critical regression：

```text
npm run test:e2e:release -- --baseURL=$STAGING_URL
```

生产部署后跑 smoke：

```text
npm run test:e2e:prod-smoke -- --baseURL=$PROD_URL
```

E2E 用例标准：

| 应放 E2E | 不建议放 E2E |
| --- | --- |
| 登录 / 登出 | 每个按钮的细节行为 |
| 核心业务闭环 | 每种表单校验 |
| 真实路由跳转 | 大量边界枚举 |
| 权限角色切换 | 纯展示组件 |
| 支付、提交、审批等关键链路 | 可由组件或集成测试覆盖的逻辑 |
| 线上 smoke | 接口字段细节 |

建议数量：

```text
PR smoke：5-20 条
Release critical：20-80 条
Nightly full：按业务复杂度扩展
```

E2E 稳定性要求：

- 稳定测试账号。
- 独立测试数据。
- 可重复执行的数据初始化和清理。
- 失败时保留 trace、video、screenshot artifact。
- 可以重试，但重试通过也要记录 flaky。

## 视觉回归测试层

视觉回归用于拦截 UI 破版、错位、响应式异常、主题样式误改等问题。

Pipeline 位置：

```text
build storybook / deploy preview
  -> capture screenshots
  -> compare with baseline
  -> upload diff
  -> require approval if changed
```

适合视觉回归覆盖：

| 适合 | 不适合 |
| --- | --- |
| Dashboard | 动态时间、随机数据区域 |
| 核心表单页 | 高频动画细节 |
| 列表页空状态 / 错误态 | 每一条数据内容 |
| 弹窗 / 抽屉 | 不稳定第三方广告或地图 |
| 移动端关键页面 | 经常变化的运营位 |
| Design system 组件 | 无业务价值的普通页面 |

视觉测试稳定性要求：

- 数据固定。
- 时间固定。
- 动画关闭。
- 截图 viewport 固定。
- 字体一致。
- 隐藏或 mock 动态区域。
- 核心页面覆盖 desktop + mobile。
- diff 需要人工审批机制。

门禁建议：

```text
设计系统组件 diff：需要确认
核心业务页面大面积 diff：阻断发布
非核心页面小 diff：允许人工 approve
```

## 测试用例分层判断标准

可以用下面的规则判断测试应该放在哪一层：

| 问题 | 推荐层级 |
| --- | --- |
| 一个函数返回是否正确 | 单元测试 |
| 一个 hook / composable 的状态逻辑是否正确 | 单元测试 |
| 一个组件点击后是否展示弹窗 | 组件测试 |
| 一个组件表单校验是否正确 | 组件测试 |
| 一个页面调用接口后是否正确渲染 | 集成测试 |
| 搜索、筛选、分页是否触发正确请求并更新页面 | 集成测试 |
| 用户能否完成跨页面业务流程 | E2E |
| 页面是否破版、错位、样式异常 | 视觉回归 |
| 前后端字段是否兼容 | 合约测试 |
| 线上发布后是否可用 | 生产 smoke |

通用判断：

```text
能用单元测试证明的，不放组件测试；
能用组件测试证明的，不放集成测试；
能用集成测试证明的，不放 E2E；
只有视觉正确性重要时，才放视觉回归；
只有真实环境链路重要时，才放 E2E。
```

## 落地顺序

建议按下面顺序推进：

1. 组件测试：覆盖核心组件、表单、表格、弹窗、权限状态。
2. 集成测试：覆盖核心页面、API mock、路由和状态协作。
3. E2E smoke：覆盖登录、核心业务闭环、发布后冒烟。
4. 视觉回归：覆盖设计系统、关键页面、移动端关键状态。
5. Pipeline 门禁：先告警，再阻断，避免 flaky 测试一次性拖慢团队。

一句话总结：

```text
组件测试负责 UI 行为；
集成测试负责页面逻辑；
E2E 负责业务闭环；
视觉回归负责“看起来没坏”；
PR 跑轻量关键集，Nightly 跑完整集，Release 跑强门禁集。
```

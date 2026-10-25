# Vue 前端开发技术规范（2026年10月25日）


## 一、总体技术栈与基本要求

- 前端统一使用 **Vue3 + Vite + Element Plus + JavaScript + Axios**。
- **严禁使用 TypeScript**：严禁出现 `.ts`、`.tsx` 文件，严禁使用 `interface`、`type`、类型注解等 TypeScript 语法。
- 必须使用 `<script setup>`，**禁止使用 Options API**。
- 页面代码必须简洁、规范、可维护。
- 前端命名应与后端接口、实体和字段保持一致。
- **不得擅自引入未约定技术栈**；不得省略注释、代码必须可直接交付使用。
- 修改前先确认组件是否仍被路由或其他组件使用；删除废弃页面、废弃接口和重复实现。
- 重构后必须执行 `npm run build`；涉及接口调整时，同时检查请求参数、响应结构和错误处理。

### 1.1 实现原则：简单可读优先

- 代码编写**以可读性为第一优先级**，用简单的风格写代码，避免炫技式写法。
- 不追求"巧妙"的写法，优先选择一眼能看懂的实现；能用简单逻辑表达清楚的，不引入抽象层。
- 注意代码换行与留白：长表达式换行、逻辑块之间空行，遵循 ESLint 风格（见第五章）。
- V1 阶段优先规则化、直白的实现，**尽量少用高级特性**（如 Pinia、provide/inject、自定义渲染函数、复杂组合式封装等），确有必要使用时须先说明理由。
- 一个函数只负责一个明确的业务动作，函数体尽量短小。

## 二、项目目录结构规范

```text
src/
├── api/              # 接口请求文件，按后端 Controller 或业务模块拆分
├── components/       # 可复用通用组件
├── views/            # 页面级组件
├── hooks/            # 自定义 hooks
├── utils/            # 工具函数
├── constants/        # 常量统一存放
├── router/           # 路由，使用懒加载
└── assets/           # 静态资源（或 public 目录）
```

- 页面级组件统一放在 `src/views/`；可复用组件统一放在 `src/components/`。
- 每个组件尽量只承担单一职责，**单文件组件超过 200 行时应评估拆分子组件**。

### 2.1 文件命名规范

- 页面文件、组件文件使用 PascalCase，如 `PartnerList.vue`、`SearchForm.vue`。
- API 文件按业务模块命名，使用 camelCase，如 `event.js`、`company.js`。
- 工具函数、常量、hooks 文件使用 camelCase 或 kebab-case，如 `formatDate.js`、`statusMap.js`。
- 路由路径使用 kebab-case，如 `/partner-list`。
- 文件名必须与内部组件/导出内容语义一致，禁止使用 `index2.vue`、`new.vue`、`test.vue` 这类无业务含义的命名。

## 三、Vue 单文件组件规范

### 3.1 顶级标签顺序

统一使用以下顺序：

```vue
<template>
  <!-- 页面模板 -->
</template>

<script setup>
// 组件逻辑
</script>

<style scoped>
/* 组件样式 */
</style>
```

### 3.2 组件命名

- 自定义组件使用 **PascalCase** 命名。
- 组件名应使用两个或两个以上单词，避免与 HTML 原生元素冲突。
- 页面文件使用有业务含义的 PascalCase 名称，例如 `PartnerList.vue`。
- 模板中使用自定义组件时使用 PascalCase；第三方组件保持组件库约定写法。

### 3.3 Composition API

- 使用 `<script setup>`。
- 响应式基础类型优先使用 `ref`；对象和复杂结构优先使用 `reactive`。
- 计算属性使用 `computed`。
- 副作用使用 `watch` 或 `watchEffect`。
- 生命周期必须使用组合式 API，如 `onMounted`、`onUnmounted`。
- 生命周期钩子应放在逻辑结构靠后位置。
- 一个函数只负责一个明确的业务动作。
- **少用高级特性**：跨层级通信优先通过 props/emits 逐层传递或简单的事件工具函数解决，避免滥用 `provide/inject`；谨慎使用 `defineExpose`，暴露的属性和方法必须有明确调用方。

### 3.4 Props 和 Emits

- 父传子使用 `defineProps`，子传父使用 `defineEmits`。
- Props 必须声明类型、是否必填和默认值；Props 应使用 camelCase。
- 对外事件名称应清晰表达业务动作；不要直接修改 Props。

### 3.5 模板规范

- 模板必须保持可读性，多个属性必须换行。
- 使用 `:`、`@`、`#` 指令缩写。
- 模板中只保留简单表达式，复杂逻辑放入计算属性或方法。
- `v-for` 必须设置稳定且唯一的 `key`。
- 不要在同一个元素上同时使用 `v-if` 和 `v-for`。
- 可自闭合的自定义组件使用自闭合写法。
- 页面模板中的搜索区、表格区、弹窗区等应添加中文注释。

## 四、JavaScript 代码规范

- 使用 2 个空格缩进。
- 函数命名使用 `camelCase`；常量命名使用 `UPPER_SNAKE_CASE`（例如 `API_BASE_URL`）。
- 事件处理函数统一以 `handle` 开头，如 `handleSearch`、`handleSubmit`。
- 请求函数优先使用业务动作命名，如 `query`、`save`、`update`、`delete`。
- 禁止在代码中散落魔法字符串，枚举值必须抽成常量对象：

```javascript
const STATUS_MAP = {
  PENDING: 0,
  ACTIVE: 1,
  DISABLED: 2
}
```

- 函数之间保留空行；避免一行代码过长；避免使用没有业务含义的缩写。
- 异步逻辑优先使用 `async/await`。
- 异常必须被处理或继续向上抛出，不能静默忽略。
- 删除无用变量、无用导入、调试代码和废弃代码。

## 五、代码风格与 ESLint 规范

- 项目统一使用 **ESLint + Prettier** 约束代码风格，`package.json` 中必须提供以下脚本：

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint --ext .js,.vue src"
  }
}
```

- 提交代码前必须执行 `npm run lint`，**不允许带着 ESLint 报错提交**；警告也应尽量清零。
- 代码风格以 ESLint 默认推荐规则集（`eslint:recommended` + Vue3 推荐规则）为准，不使用个性化、过度严格的自定义规则。
- 代码编写尽量可读：用简单的风格写代码，注意换行与留白——长参数列表、长条件表达式、链式调用应适当换行，逻辑块之间保留空行。
- 禁止压缩成一行的"面条代码"，禁止为了少写几行而牺牲可读性。
- Prettier 负责统一格式化（缩进、引号、分号、行宽），团队成员不得各自调整格式化配置。

## 六、状态管理规范（少用高级特性）

- **尽量少用 Pinia 等高级状态管理功能**，此类功能过于复杂，V1 阶段不默认引入。
- 组件间数据流转优先顺序为：
  1. 父子组件之间使用 `props` / `emits`；
  2. 简单跨组件数据（如用户信息、字典常量）使用 `src/constants/` 常量对象或轻量工具模块；
  3. 登录态 Token 等持久化数据使用 `localStorage`，由 `request.js` 拦截器统一读写；
  4. 确需全局响应式状态时，优先在 `src/hooks/` 中用简单的组合式函数实现，不使用第三方状态库。
- 禁止在组件中随意挂载全局事件总线（如 `mitt`）、禁止用模块全局变量保存业务状态。
- 后续如因业务复杂度确实需要引入 Pinia，须先说明理由并统一在项目范围内引入，不允许页面私自引入。

## 七、可配置项标识规范

为方便搜索定位，项目中**所有常用配置项**（如端口、路径、地址、超时时间等）必须满足以下要求：

- 配置项本身必须通过环境变量或集中在配置文件顶层定义，**禁止散落在业务代码中写死**。
- 在配置文件、环境变量文件中为配置项添加注释说明，注释中必须带 **【可配置】** 标识，并说明默认值、用途和调整影响。
- AI 生成代码时，凡涉及端口、路径、地址、超时等配置，必须自动带上【可配置】标识注释。
- 全文搜索"【可配置】"应能快速定位所有可调参数。

示例：

```bash
# .env.development
# 【可配置】前端请求基础路径，开发环境走 Vite 代理，前缀需与 vite.config.js 中 proxy 的 key 一致
VITE_API_BASE_URL=/api
# 【可配置】后端服务地址，指向开发/联调环境的后端服务
VITE_API_TARGET=http://10.0.0.1:8080
```

```javascript
// vite.config.js
export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())

  return {
    plugins: [vue()],
    server: {
      // 【可配置】前端开发服务器端口，如被占用可更换
      port: 5173,
      proxy: {
        // 【可配置】代理路径前缀，需与 .env 中 VITE_API_BASE_URL 保持一致
        '/api': {
          // 【可配置】后端服务地址，从环境变量读取
          target: env.VITE_API_TARGET,
          changeOrigin: true,
          rewrite: path => path.replace(/^\/api/, '')
        }
      }
    }
  }
})
```

## 八、接口与请求规范

### 8.1 HTTP 方法约定

- 本项目业务接口**默认统一优先使用 `POST`**，不强制区分 `GET`、`PUT`、`DELETE` 的语义。
- 查询、新增、更新、删除等业务动作原则上都可统一为 `POST`；只有当 `POST` 明显无法满足场景时，才单独评估其他 HTTP 方法。
- 接口语义通过路径和业务动作表达，而非依赖 HTTP 动词区分。

### 8.2 Axios 统一封装

- 所有接口请求必须统一走 `src/utils/request.js`，**禁止在组件或业务 API 文件中直接 `import axios`**，页面组件不直接创建 Axios 实例。
- 统一封装要求：
  - `baseURL` 从 `import.meta.env.VITE_API_BASE_URL` 读取
  - `timeout` 统一为 `10000`
  - 请求拦截器自动注入 Token（不在各页面重复处理）
  - 响应拦截器统一处理业务异常
  - 401 场景自动清除 Token 并跳转登录页
- 标准封装模板：

```javascript
import axios from 'axios'
import { ElMessage } from 'element-plus'

const request = axios.create({
  // 【可配置】接口基础地址，从环境变量读取，禁止写死
  baseURL: import.meta.env.VITE_API_BASE_URL,
  // 【可配置】请求超时时间（毫秒），网络较差环境可适当调大
  timeout: 10000
})

request.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  err => Promise.reject(err)
)

request.interceptors.response.use(
  res => {
    const { code, message, data } = res.data
    if (code === 200) {
      return data
    }
    ElMessage.error(message || '请求失败')
    return Promise.reject(new Error(message))
  },
  err => {
    if (err.response?.status === 401) {
      localStorage.removeItem('token')
      window.location.href = '/login'
    }
    ElMessage.error(err.message || '网络异常，请稍后重试')
    return Promise.reject(err)
  }
)

export default request
```

### 8.3 API 模块规范

- 接口文件统一放在 `src/api/`，按业务模块拆分文件。
- 每个 API 文件只能引用统一封装的 `request`，底层请求默认优先调用 `request.post`。
- 函数名强调业务语义、不再强调 `post`，如 `queryXxx`、`saveXxx`、`updateXxx`、`deleteXxx`。
- API 方法名应与后端接口语义保持一致，并添加中文注释说明用途和参数。
- 接口请求和页面展示逻辑分离。

示例：

```javascript
import request from '@/utils/request'

// 查询赛事列表
export const queryEventList = (data) => request.post('/event/query', data)
```

### 8.4 请求交互规范

- 列表查询、表单提交等请求发起后，应给出 loading 反馈（表格 `v-loading` 或按钮 loading），请求结束必须关闭。
- 表单提交、删除等写操作必须**防止重复提交**：提交期间禁用提交按钮，或做重复点击拦截。
- 删除等危险操作必须先弹 `ElMessageBox` 确认框，确认后才发起请求。
- 分页查询必须携带分页参数，空数据场景展示 Element Plus 的空状态。

## 九、Element Plus 使用规范

- 组件必须**按需引入**，不允许全量引入。
- 表单必须使用 `el-form` + `el-form-item`，表单校验必须配置 `rules`。
- 表格必须使用 `el-table` + `el-table-column`。
- 分页统一使用 `el-pagination`。
- 弹窗统一使用 `el-dialog`。
- 消息提示统一使用 `ElMessage`；确认框统一使用 `ElMessageBox`。
- 图标统一使用 **Font Awesome** 图标库按需引入；不建议使用 `@element-plus/icons-vue`（图标数量较少且与 Font Awesome 有差异）。

## 十、Vite 与环境变量规范

### 10.1 Vite 代理

- 开发环境必须配置 `server.proxy`。
- 代理日志必须通过 `configure` 钩子的 `proxyReq` 事件输出，禁止在 Axios 层手工模拟代理日志。

```javascript
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())

  return {
    plugins: [vue()],
    resolve: {
      alias: {
        // 【可配置】路径别名，项目中统一使用 @ 指代 src 目录
        '@': '/src'
      }
    },
    server: {
      // 【可配置】前端开发服务器端口，如被占用可更换
      port: 5173,
      proxy: {
        // 【可配置】代理路径前缀，需与 .env 中 VITE_API_BASE_URL 保持一致
        '/api': {
          // 【可配置】后端服务地址，从环境变量读取
          target: env.VITE_API_TARGET,
          changeOrigin: true,
          rewrite: path => path.replace(/^\/api/, ''),
          configure: (proxy, options) => {
            proxy.on('proxyReq', (proxyReq, req) => {
              const targetUrl = `${options.target}${proxyReq.path}`
              console.log(`[Vite 代理转发] ${req.url} -> ${targetUrl}`)
            })
          }
        }
      }
    }
  }
})
```

### 10.2 环境变量

- 首次生成前端代码时，必须同时提供 `.env.development` 和 `.env.production`。
- 所有自定义环境变量必须以 `VITE_` 开头，否则 Vite 不会注入到客户端代码中。
- 环境变量按环境分文件维护，禁止在代码中判断环境后写死多套地址。
- Google reCAPTCHA 相关密钥、后端地址等敏感或环境相关配置只能通过环境变量配置，不能写入源码。

```bash
# .env.development
# 【可配置】前端请求基础路径，开发环境走 Vite 代理，前缀需与 vite.config.js 中 proxy 的 key 一致
VITE_API_BASE_URL=/api
# 【可配置】后端服务地址，指向开发/联调环境的后端服务
VITE_API_TARGET=http://10.0.0.1:8080
```

```bash
# .env.production
# 【可配置】生产环境接口基础地址，指向线上后端服务
VITE_API_BASE_URL=https://your-prod-domain.com/api
```

## 十一、路由规范

- 路由组件使用**懒加载**。
- 路径使用 kebab-case；路由名称使用 PascalCase，并与页面组件名称保持一致。
- 路由元信息（meta）统一存放标题、菜单显示、图标和权限信息。
- 登录校验在全局路由守卫中处理。
- 页面跳转参数优先使用路由参数，不依赖临时全局变量。
- 不存在的路径必须配置 404 兜底路由；无权限访问的路由应在守卫中拦截并给出提示。

## 十二、样式规范

- 所有样式统一写在 `<style scoped>` 中，禁止内联 `style`（动态样式绑定除外）。
- 类名统一使用 **kebab-case**。
- 布局优先使用 **Flexbox**，尽量减少无意义嵌套。
- 样式按页面结构分组，保持换行和缩进；避免过深的选择器嵌套；避免使用 `!important`（确有第三方组件覆盖需要时除外）。
- 优先复用 Element Plus 自带样式和变量；公共样式放在全局样式文件，组件专属样式放在组件内。
- 样式以简洁实用为主，不额外引入其他 CSS 框架。
- 颜色、间距、字号等优先使用 Element Plus CSS 变量，禁止各处散落重复的魔法数值。

## 十三、注释规范

- 公共组件、API、关键业务方法必须添加中文注释。
- 注释说明"为什么这样做"，不要只重复代码含义。
- 复杂的登录、权限、分页、批量操作逻辑必须添加说明。
- 注释应及时维护，禁止保留与代码不一致的注释。
- 配置项注释必须带【可配置】标识（见第七章）。

## 十四、手机端 H5 适配规范

本项目 Vue 页面默认需兼顾手机端 H5 浏览体验，不能只按桌面后台尺寸开发；AI 生成页面时，除桌面端效果外，必须同步补充手机端适配样式，不能把移动端适配留到后续补做。

- 页面布局必须使用响应式设计，优先使用 `flex`、`grid`、百分比宽度、`minmax`、`clamp` 等方式实现自适应。
- 关键断点至少考虑 `1280px`、`960px`、`760px` 三档，必要时可继续细分。
- 手机端下多列布局应自动收敛为单列或上下结构，避免内容被强行压缩。
- 表单在手机端应优先单列展示，输入框、选择器、按钮需便于触控操作。
- 表格类页面必须考虑手机端可用性；列较多时允许使用横向滚动容器包裹表格，而不是强行压缩字段。
- 弹窗、抽屉、卡片等容器在手机端宽度不得写死，必须限制为视口内可完整展示。
- 顶部操作区、筛选区、按钮组在手机端应允许换行，避免按钮溢出。
- 不得把仅适合桌面端的 hover 交互作为唯一反馈方式，手机端点击态和默认态也要可理解。
- 文本、间距、圆角、卡片阴影在手机端应适当收敛，避免页面显得拥挤或过重。

## 十五、安全与敏感信息规范

- 任何密钥、Token、密码、reCAPTCHA 密钥、内部服务地址等**禁止硬编码在源码中**，只能通过环境变量注入。
- Token 仅存于 `localStorage`，由请求拦截器统一携带；禁止把敏感信息写入 URL 参数或明文日志。
- 控制台日志（`console.log`）仅用于开发调试，交付前必须清理；代理转发日志除外（该日志为规范要求保留）。
- 用户输入的内容渲染到页面前必须转义，防止 XSS；禁止将接口返回的 HTML 直接 `v-html` 渲染，确需使用时必须做白名单过滤。

## 十六、前端代码输出规范

AI 生成前端代码时，必须按以下结构组织输出，不允许随意拼接零散代码：

### 1）需求理解

简述本次页面或模块要实现什么。

### 2）组件拆分说明

说明页面由哪些组件组成、各自职责是什么。

### 3）完整代码

按文件逐个输出，并标明文件路径。首次生成时必须包含：

- `.env.development`
- `.env.production`
- `vite.config.js`
- `src/utils/request.js`
- `src/api/xxx.js`

代码中涉及端口、路径、地址、超时时间等配置项时，必须自动添加带 **【可配置】** 标识的注释说明。

### 4）依赖说明

至少列出：

```bash
npm install axios element-plus @element-plus/icons-vue
```

### 5）使用说明

如有组件 Props、Emits、事件流转或使用方式，需一并说明。


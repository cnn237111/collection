# Vue 前端开发技术规范（整合版）

> 本文档整合自《技术规范》第 5 章、《Vue3 + Vite 前端开发规范》及《Vue 前端规范》中所有与 Vue 前端开发相关的规约，并补充了常见前端工程化规约，作为项目前端开发的统一执行标准。
> 如各来源文档存在冲突，以本文档为准。

## 一、总体技术栈与基本要求

- 前端统一使用 **Vue3 + Vite + Element Plus + JavaScript + Axios + Vue Router**。
- **严禁使用 TypeScript**：严禁出现 `.ts`、`.tsx` 文件，严禁使用 `interface`、`type`、类型注解等 TypeScript 语法。
- 必须使用 `<script setup>`，**禁止使用 Options API**。
- **职责划分**：页面（views）负责业务编排，组件（components）负责可复用交互，API 模块（api）负责网络通信。
- 前台访客页面无需登录；后台页面必须经过登录和路由权限校验。**前端权限只用于界面控制，真正的权限判定必须由后端完成。**
- 不在组件中写死服务器地址、密钥、角色权限和业务状态文字。
- 页面代码必须简洁、规范、可维护；前端命名应与后端接口、实体和字段保持一致。
- **不得擅自引入未约定技术栈**；不得省略注释、代码必须可直接交付使用。
- 修改前先确认组件是否仍被路由或其他组件使用；删除废弃页面、废弃接口和重复实现。
- 重构后必须执行 `npm run build`；涉及接口调整时，同时检查请求参数、响应结构和错误处理。

### 1.1 实现原则：简单可读优先

- 代码编写**以可读性为第一优先级**，用简单的风格写代码，避免炫技式写法。
- 不追求"巧妙"的写法，优先选择一眼能看懂的实现；能用简单逻辑表达清楚的，不引入抽象层。
- 优先使用简单、明确的实现，避免在项目早期引入不必要的抽象。
- 注意代码换行与留白：长表达式换行、逻辑块之间空行，遵循 ESLint 风格（见第六章）。
- V1 阶段优先规则化、直白的实现，**尽量少用高级特性**（如 Pinia、复杂组合式封装、自定义渲染函数等），确有必要使用时须先说明理由（见第七章状态管理）。
- 一个函数只负责一个明确的业务动作，函数体尽量短小。

## 二、项目目录结构规范

```text
src/
├── api/              # 按业务模块封装接口，与后端 Controller 对应
├── assets/           # 图片、字体和全局样式
├── components/       # 通用可复用组件
├── config/           # 【可配置】集中管理应用配置（app.js）、请求配置（request.js）等
├── constants/        # 常量与稳定枚举
├── hooks/            # 可复用组合式逻辑（use 前缀）
├── layouts/          # Public、Admin 等布局
├── router/           # 路由与守卫，路由组件懒加载
├── stores/           # Pinia 状态（仅在确有必要时使用）
├── utils/            # request、格式化等工具
│   └── request.js    # Axios 统一封装
├── views/
│   ├── public/       # 公开前台页面
│   └── admin/        # 登录后台页面
├── App.vue
└── main.js
```

- 页面级组件统一放在 `src/views/`；可复用组件统一放在 `src/components/`。
- **业务专用组件优先放在对应页面或模块目录下**，不全部堆进全局 `components`。
- 每个组件尽量只承担单一职责，**单文件组件超过 200 行时应评估是否混合了多个职责，按业务含义拆分**（不按行数机械拆分）。

## 三、命名规范

- Vue 组件文件使用 PascalCase（大驼峰），如 `TeamCard.vue`、`EventEditor.vue`。
- API 文件按业务模块命名，使用 camelCase，如 `event.js`、`company.js`。
- 工具函数、常量、hooks 文件使用 camelCase 或 kebab-case，如 `formatDate.js`、`statusMap.js`。
- JavaScript 变量和函数使用 camelCase，如 `teamList`、`loadTeams`。
- 常量使用全大写下划线，如 `DEFAULT_PAGE_SIZE`。
- 事件处理函数统一以 `handle` 开头，如 `handleSubmit`、`handlePageChange`。
- 获取数据函数使用明确动词，如 `fetchTeamList`、`loadEventDetail`。
- 组合式函数使用 `use` 前缀，如 `usePagination`。
- CSS 类名使用 kebab-case，如 `.team-card__title`；同一项目可采用 BEM 风格并保持一致。
- 路由路径使用 kebab-case，如 `/partner-list`。
- 文件名必须与内部组件/导出内容语义一致，禁止使用 `index2.vue`、`new.vue`、`temp.vue` 等无业务含义或含义模糊的命名。
- 避免 `data1`、`temp`、`info` 等含义模糊的命名。

## 四、Vue 单文件组件规范

### 4.1 顶级标签顺序

统一使用以下顺序（团队内保持一致）：

```vue
<script setup>
// imports、props/emits、state(ref)、computed、methods、lifecycle
</script>

<template>
  <!-- 页面结构 -->
</template>

<style scoped>
/* 组件样式 */
</style>
```

### 4.2 组件命名

- 自定义组件使用 **PascalCase** 命名。
- 组件名应使用两个或两个以上单词，避免与 HTML 原生元素冲突。
- 页面文件使用有业务含义的 PascalCase 名称，例如 `PartnerList.vue`。
- 模板中使用自定义组件时使用 PascalCase；第三方组件保持组件库约定写法。

### 4.3 响应式状态：统一使用 ref

- **统一使用 `ref` 声明所有响应式状态（包括基本类型和对象类型），避免使用 `reactive`**，便于团队风格统一。
- 理由：`ref` 对基本类型和对象类型均适用，访问时统一通过 `.value`（模板中自动解包），避免 `reactive` 带来的解构丢失响应性、深层嵌套难以追踪等问题。
- 对象类型使用 `ref` 时通过 `.value` 访问属性，如 `userInfo.value.name`。

```js
// ✅ 推荐：统一使用 ref
const count = ref(0)
const userInfo = ref({ name: '', age: 0 })
// 访问：userInfo.value.name

// ❌ 不推荐：使用 reactive
const state = reactive({ count: 0, user: { name: '', age: 0 } })
```

### 4.4 Composition API

- 使用 `<script setup>`，不混用 Options API。
- 计算属性使用 `computed`；**派生状态必须使用 `computed`，不要用 `watch` 重复维护可计算的数据**。
- `watch` 只用于明确的副作用，并及时清理定时器、监听器和订阅。
- 生命周期必须使用组合式 API，如 `onMounted`、`onUnmounted`，并放在逻辑结构靠后位置。
- **页面离开或组件卸载时（`onUnmounted`）取消不再需要的请求和副作用**。
- 少用高级特性：`defineExpose` 暴露的属性和方法必须有明确调用方。

### 4.5 Props 和 Emits

- 父传子使用 `defineProps`，子传父使用 `defineEmits`。
- Props 必须声明类型、是否必填和合理默认值；对象和数组默认值使用工厂函数；Props 应使用 camelCase。
- 对外事件名称应清晰表达业务动作；**子组件通过 `emit` 通知父组件，不直接修改 Props**。
- 双向绑定应有明确的数据所有者，避免多层组件同时修改同一对象。

### 4.6 模板规范

- 模板必须保持可读性，多个属性必须换行。
- 使用 `:`、`@`、`#` 指令缩写。
- 模板中只保留简单表达式，复杂逻辑放入计算属性或方法。
- `v-for` 必须设置稳定且唯一的 `key`，**禁止用数组下标代替业务 ID**。
- 不要在同一个元素上同时使用 `v-if` 和 `v-for`。
- 可自闭合的自定义组件使用自闭合写法。
- 页面模板中的搜索区、表格区、弹窗区等应添加中文注释。

## 五、JavaScript 代码规范

- 使用 2 个空格缩进。
- 常量命名使用 `UPPER_SNAKE_CASE`（例如 `API_BASE_URL`）；其余命名规则见第三章。
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

## 六、代码风格与 ESLint 规范

- 项目统一使用 **ESLint + Prettier** 约束代码风格，启用 ESLint 和 Prettier，并在提交前执行检查。`package.json` 中必须提供以下脚本：

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
- **生产环境禁止输出 `console.log`**，通过 ESLint 规则 `no-console` 拦截；开发环境使用 `console.debug` / `console.info` 输出调试信息，交付前清理。

## 七、状态管理规范（少用高级特性）

- **尽量少用 Pinia 等高级状态管理功能**，此类功能过于复杂，V1 阶段不默认引入。
- 状态流转优先顺序：
  1. **仅组件内部使用的状态**：保留在组件中（使用 `ref`）。
  2. **父子组件共享状态**：优先通过 Props 和 Emits。
  3. **跨层级组件共享状态**：使用 `provide/inject`，而非引入 Pinia。
  4. **登录用户、权限、跨页面筛选条件等全局状态**：在确有必要时才使用 Pinia；Store 不直接操作 DOM，也不保存不可序列化对象。
- **服务端数据不应在多个 Store 中重复保存**；明确唯一数据来源和刷新时机，不把所有接口结果长期缓存到全局状态，避免陈旧数据。
- 禁止在组件中随意挂载全局事件总线（如 `mitt`）、禁止用模块全局变量保存业务状态。
- 后续如因业务复杂度确实需要引入 Pinia，须先说明理由并统一在项目范围内引入，不允许页面私自引入。

## 八、可配置项标识规范

为方便搜索定位，项目中**所有常用配置项**（如端口、路径、地址、超时时间等）必须满足以下要求：

- 配置项本身必须通过环境变量（`.env.*`）或集中在 `src/config/` 中定义，**禁止散落在业务代码中写死**。
- 在配置文件、环境变量文件中为配置项添加注释说明，注释中必须带 **【可配置】** 标识，并说明默认值、用途和调整影响。
- AI 生成代码时，凡涉及端口、路径、地址、超时等配置，必须自动带上【可配置】标识注释。
- 全文搜索"【可配置】"应能快速定位所有可调参数。

### 8.1 环境变量（【可配置】最主要的入口）

```bash
# .env.development
# 【可配置】开发环境配置
# 【可配置】后端接口基础路径，开发环境走 Vite 代理，前缀需与 vite.config.js 中 proxy 的 key 一致
VITE_API_BASE_URL=/api
# 【可配置】应用标题，用于 <title> 或页面展示
VITE_APP_TITLE=My App (Dev)
# 【可配置】本地开发服务端口（如需修改默认 5173，在 package.json 的 dev 脚本中 --port）
# VITE_API_PORT=3000
# 【可配置】是否开启本地 mock，true/false
# VITE_MOCK=true
```

```bash
# .env.production
# 【可配置】生产环境配置
# 【可配置】生产环境接口基础地址，由网关/反向代理转发
VITE_API_BASE_URL=/api
# 【可配置】生产环境应用标题
VITE_APP_TITLE=My App
```

| 变量名 | 含义 | 默认值 | 取值范围 / 示例 | 使用方式 |
| --- | --- | --- | --- | --- |
| `VITE_API_BASE_URL` | 后端接口基础路径（上下文） | `/api` | 以 `/` 开头的相对路径（走同源代理），或完整域名 | `axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL })` |
| `VITE_APP_TITLE` | 应用标题 | `My App` | 任意字符串 | `document.title = import.meta.env.VITE_APP_TITLE` |
| `VITE_API_PORT`（可选） | 本地开发端口 | `5173` | 可用端口号 | `package.json` 的 `dev` 脚本中 `--port` |
| `VITE_MOCK`（可选） | 是否启用 mock | `false` | `true` / `false` | 入口处 `if (import.meta.env.VITE_MOCK === 'true')` 按需加载 |

- 所有自定义环境变量必须以 `VITE_` 开头，否则 Vite 不会注入到客户端代码中。
- **仅 `VITE_` 前缀的变量会暴露给客户端，变量中不得包含密钥、数据库密码等敏感信息**。
- 在代码中通过 `import.meta.env.VITE_XXX` 读取，**不要写死字符串**：

```js
// ✅ 正确：通过环境变量读取
const baseURL = import.meta.env.VITE_API_BASE_URL || '/api'

// ❌ 错误：写死地址，换环境就要改代码
const baseURL = 'http://192.168.1.100:8080/api'
```

- 环境差异通过配置解决，不在代码中使用域名字符串判断环境；不同环境文件：`.env.development`（开发）、`.env.production`（生产）、`.env.staging`（预发，按需）。

### 8.2 集中配置（src/config/）

```js
// src/config/app.js
// 【可配置】应用级配置集中管理，开发者搜索"可配置"即可定位
export const APP_CONFIG = {
  APP_NAME: import.meta.env.VITE_APP_TITLE || 'Vue App',      // 【可配置】应用名称（取自环境变量）
  API_BASE_URL: import.meta.env.VITE_API_BASE_URL || '/api',  // 【可配置】API 基础路径（取自环境变量）
  DEFAULT_PAGE_SIZE: 10,                                      // 【可配置】分页默认条数
  UPLOAD_MAX_SIZE: 10 * 1024 * 1024,                          // 【可配置】上传文件大小限制（10MB）
  TOKEN_KEY: 'auth_token',                                    // 【可配置】Token 存储 Key
  LOGIN_PATH: '/login',                                       // 【可配置】登录页路径
  HOME_PATH: '/dashboard'                                     // 【可配置】登录后跳转首页
}
```

```js
// src/config/request.js
// 【可配置】请求层配置
export const REQUEST_CONFIG = {
  TIMEOUT: 10000,        // 【可配置】默认超时（毫秒）
  UPLOAD_TIMEOUT: 60000, // 【可配置】上传接口超时
  RETRY_COUNT: 0,        // 【可配置】失败重试次数
  RETRY_DELAY: 1000      // 【可配置】重试间隔
}
```

## 九、接口与请求规范

### 9.1 HTTP 方法约定

- 本项目业务接口**默认统一优先使用 `POST`**，不强制区分 `GET`、`PUT`、`DELETE` 的语义。
- 查询、新增、更新、删除等业务动作原则上都可统一为 `POST`；只有当 `POST` 明显无法满足场景时，才单独评估其他 HTTP 方法。
- 接口语义通过路径和业务动作表达，而非依赖 HTTP 动词区分；**前端不得自行改用其他请求方式导致接口不一致**。

### 9.2 Axios 统一封装

- 所有 HTTP 请求必须通过 `src/utils/request.js` 中的 Axios 实例发送，**禁止在组件或业务 API 文件中直接 `import axios`**，页面组件不直接创建 Axios 实例。
- 统一封装要求：
  - `baseURL` 来自 `import.meta.env.VITE_API_BASE_URL`
  - `timeout` 统一为 `10000`；上传等长耗时接口可单独调整
  - 请求拦截器统一附加登录凭证（可在 `config.headers` 附加 `requestId` 用于链路追踪）
  - 响应拦截器统一处理标准响应、业务异常、网络错误和登录失效
  - 收到 401 时清理本地登录状态并跳转登录页，避免多个并发请求重复提示
  - **禁止在日志或错误弹窗中显示完整 Token、请求头或敏感数据**
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
    if (err.response?.status === 403) {
      ElMessage.error('无权限执行此操作')
    }
    ElMessage.error(err.message || '网络异常，请稍后重试')
    return Promise.reject(err)
  }
)

export default request
```

### 9.3 API 模块规范

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

### 9.4 请求交互规范

- 列表查询、表单提交等请求发起后，应给出 loading 反馈（表格 `v-loading` 或按钮 loading），请求结束必须关闭。
- 表单提交、删除等写操作必须**防止重复提交**：提交期间禁用提交按钮，或做重复点击拦截。
- 删除等危险操作必须先弹 `ElMessageBox` 确认框，确认后才发起请求。
- 分页查询必须携带分页参数，空数据场景展示 Element Plus 的空状态。

## 十、Element Plus 使用规范

- 组件必须**按需引入**，不允许全量引入，避免无必要的整库打包。
- 表单必须使用 `el-form` + `el-form-item`，表单校验必须配置 `rules`。
- 表格必须使用 `el-table` + `el-table-column`；分页统一使用 `el-pagination`，分页行为和参数命名保持统一。
- 弹窗统一使用 `el-dialog`（对话框用于短流程编辑或确认；复杂表单使用独立页面）。
- 消息提示统一使用 `ElMessage`；确认框统一使用 `ElMessageBox`（需要更强提示时可用 `ElNotification`）。
- 主题颜色、间距、圆角和字号集中为 CSS 变量或主题配置。
- **删除和不可逆操作必须二次确认，并清楚显示影响对象。**

### 10.1 图标规范（Font Awesome）

- 统一使用 **Font Awesome** 图标库，按需引入，**不与其他图标库混用**；不建议使用 `@element-plus/icons-vue`（图标数量较少且与 Font Awesome 有差异）。
- **直接在元素上写 FA 类名即可，不搞额外的配置文件**，保持简单：

```vue
<template>
  <!-- 菜单图标 -->
  <i class="fas fa-users"></i>
  <!-- 按钮图标 -->
  <el-button><i class="fas fa-plus"></i> 新增</el-button>
</template>
```

- 写法要点：
  - 免费版用 `fas`（solid）或 `fab`（brands）前缀，如 `fas fa-users`、`fab fa-github`。
  - 图标类名以字符串形式直接写在 `class` 上，需要动态切换时再用 `:class` 绑定。
  - 菜单、按钮等场景如需根据数据驱动，直接在数据里放 FA 类名字符串（如路由 `meta.icon`），渲染时 `<i :class="item.meta.icon"></i>` 直接使用。
  - 图标语义应与功能一致，不要为图省事全用 `fa-star`。
  - 不要在多个地方写死重复的图标逻辑；如有统一封装（如 `<icon-name>` 组件），组件内部仍是 FA 类名，不要再加一层配置层。

## 十一、表单规范

- 前端校验用于即时反馈，**后端校验才是最终依据**。
- 必填、长度、格式和数值范围规则应与后端保持一致。
- 提交时禁用提交按钮并显示加载状态，避免重复提交。
- 请求失败后保留用户已填写内容，除非业务明确要求清空。
- 服务端字段错误应映射到具体表单项；通用错误显示在表单顶部或消息区。
- 编辑页面加载完成前不得显示错误的默认值。
- 离开有未保存修改的复杂表单时应提示用户。

## 十二、路由与布局规范

- 公开前台和管理后台使用不同布局，例如 `PublicLayout`、`AdminLayout`。
- 路由组件使用**懒加载**，降低首次加载体积。
- 路径使用 kebab-case；路由名称使用 PascalCase 且唯一，并与页面组件名称保持一致。
- 路由元信息（meta）统一存放标题、菜单显示、图标、排序、权限等信息；后台路由通过 `meta.requiresAuth` 标记需要登录。
- 角色访问范围通过路由元信息和统一守卫控制，但**后端仍须再次校验**。
- 登录校验在全局路由守卫中处理；**登录后跳回原目标页面时，必须校验重定向地址，防止开放重定向**。
- 页面跳转参数优先使用路由参数，不依赖临时全局变量。
- 必须配置 404、无权限和通用错误页面；无权限访问应在守卫中拦截并给出提示。

### 12.1 后台导航栏数据从路由派生

后台左侧导航栏数据**应从路由配置中派生**，保持代码干练简洁，避免维护两份菜单数据：

```js
// layouts/AdminLayout.vue 中派生菜单
import { useRouter } from 'vue-router'

const router = useRouter()
// 从路由配置中过滤出需要展示在菜单中的路由
const menuRoutes = computed(() => {
  return router.getRoutes()
    .filter(route => route.meta?.showInMenu) // 只展示标记了 showInMenu 的路由
    .sort((a, b) => (a.meta?.order || 0) - (b.meta?.order || 0))
})
```

路由配置中通过 `meta` 提供菜单所需信息：

```js
// router/admin.js
{
  path: '/admin/teams',
  name: 'AdminTeams',
  component: () => import('@/views/admin/Teams.vue'),
  meta: {
    requiresAuth: true,
    showInMenu: true,          // 是否在菜单中展示
    title: '团队管理',           // 菜单显示文字
    icon: 'fas fa-users',      // 菜单图标（直接写 Font Awesome 类名，见 10.1 图标规范）
    order: 1,                  // 排序权重
    roles: ['admin', 'editor'] // 允许访问的角色
  }
}
```

## 十三、加载、空状态与错误处理

- **每个异步页面应明确处理加载中、成功、空数据和失败四种状态**。
- 首次加载使用骨架屏或区域加载效果（如表格 `v-loading`），避免整页反复闪烁。
- 操作按钮的加载状态应限制在对应操作范围。
- 可重试错误提供重试入口；权限错误不反复请求。
- 异常不能只在控制台输出而不给用户反馈。
- 多个请求并行时分别维护状态，避免一个请求结束后错误关闭全部加载状态。

## 十四、样式规范

- 所有样式统一写在 `<style scoped>` 中，禁止内联 `style`（值确实依赖运行时计算时才允许动态样式绑定）。
- 类名统一使用 **kebab-case**（可采用 BEM 风格并保持一致）。
- 布局优先使用 **Flexbox / Grid**，不依赖大量绝对定位，尽量减少无意义嵌套。
- 样式按页面结构分组，保持换行和缩进；避免过深的选择器嵌套；避免使用 `!important`（确有第三方组件覆盖需要时须说明原因）。
- 公共颜色、间距、字号、阴影和层级使用统一设计变量（优先复用 Element Plus 自带样式和 CSS 变量）；禁止各处散落重复的魔法数值。
- 公共样式放在全局样式文件，组件专属样式放在组件内；样式以简洁实用为主，不额外引入其他 CSS 框架。

### 14.1 响应式断点

页面至少适配桌面、平板和手机四档：

- 大屏：`>= 1280px`
- 中屏：`960px - 1279px`
- 小屏：`760px - 959px`
- 手机：`< 760px`

## 十五、手机端 H5 适配规范

本项目 Vue 页面默认需兼顾手机端 H5 浏览体验，不能只按桌面后台尺寸开发；AI 生成页面时，除桌面端效果外，必须同步补充手机端适配样式，不能把移动端适配留到后续补做。

- 页面布局必须使用响应式设计，优先使用 `flex`、`grid`、百分比宽度、`minmax`、`clamp` 等方式实现自适应，断点按 14.1 执行。
- 手机端下多列布局应自动收敛为单列或上下结构，避免内容被强行压缩。
- 表单在手机端应优先单列展示，输入框、选择器、按钮需便于触控操作，交互目标应有足够点击区域。
- 表格类页面必须考虑手机端可用性；列较多时允许使用横向滚动容器包裹表格，或卡片化、隐藏次要列，而不是强行压缩到不可读。
- 弹窗、抽屉、卡片等容器在手机端宽度不得写死，必须限制为视口内可完整展示。
- 顶部操作区、筛选区、按钮组在手机端应允许换行，避免按钮溢出。
- 不得把仅适合桌面端的 hover 交互作为唯一反馈方式，手机端点击态和默认态也要可理解。
- 文本、间距、圆角、卡片阴影在手机端应适当收敛，避免页面显得拥挤或过重。

## 十六、可访问性规范

- 图片必须提供有意义的 `alt`，装饰图片使用空 `alt`。
- 表单控件必须具有可识别标签，不能只依赖 placeholder。
- 键盘能够访问菜单、按钮、对话框和主要操作。
- 焦点状态清晰可见；打开和关闭对话框时正确管理焦点。
- 颜色不能作为状态的唯一表达方式，文字与背景应具有足够对比度。
- 使用原生语义元素，避免把 `div` 伪装成按钮。

## 十七、性能规范

- 路由页面和大型功能按需加载（懒加载）。
- 搜索输入使用防抖，滚动和缩放处理使用节流。
- 大列表使用服务端分页；确有需要时使用虚拟列表。
- 图片设置合理尺寸、格式和懒加载，避免上传原图直接展示。
- 避免在模板中调用高开销函数，使用 `computed` 缓存派生结果。
- 引入新依赖前评估体积、维护状态和是否可由现有能力完成。

## 十八、安全规范

- 任何密钥、Token、密码、reCAPTCHA 密钥、内部服务地址等**禁止硬编码在源码中**，只能通过环境变量注入；**前端环境变量会进入构建产物，不得存储任何真正的秘密**。
- **禁止用 `v-html` 渲染未经可信清洗的用户内容**；确需使用时必须做白名单过滤。
- 不在 `localStorage` 保存密码和长期敏感信息；Token 仅存于 `localStorage`，由请求拦截器统一携带；禁止把敏感信息写入 URL 参数或明文日志。
- 上传文件时前端校验文件类型、大小和数量（后端仍需重新校验）。
- 外部链接使用 `target="_blank"` 时增加 `rel="noopener noreferrer"`。
- 前端权限只用于界面控制，**不根据客户端传入的角色或用户 ID 判断最终权限**，真正的权限判定必须由后端完成。
- 代理转发日志仅在开发环境开启，不在生产环境输出敏感请求信息；控制台日志仅用于开发调试，交付前必须清理（代理转发日志除外）。

## 十九、代码质量与测试

- 启用 ESLint 和 Prettier，并在提交前执行检查（见第六章）。
- 禁止遗留无用途的变量、调试日志和注释代码。
- 复杂业务逻辑提取为纯函数或组合式函数，以便测试。
- 核心工具函数、权限判断和状态转换应有单元测试。
- 关键用户流程至少覆盖登录、公开浏览、列表查询、创建和编辑等集成测试。
- 修复缺陷时优先补充复现测试。
- **自动生成的页面必须人工检查响应式布局、空状态、权限和错误处理。**

### 19.1 注释规范

- 公共组件、API、复杂逻辑、工具函数、组合式函数必须添加中文注释/JSDoc，说明功能、参数、返回值。
- 注释说明"为什么这样做"，不要只重复代码含义；复杂的登录、权限、分页、批量操作逻辑必须添加说明。
- 所有【可配置】项必须附带注释说明参数含义、取值范围和默认值。
- 注释应及时维护，禁止保留与代码不一致的注释。

```js
/**
 * 格式化日期为指定格式
 * @param {Date|string|number} date - 日期对象、时间戳或日期字符串
 * @param {string} format - 输出格式，如 'YYYY-MM-DD HH:mm:ss'
 * @returns {string} 格式化后的日期字符串
 */
function formatDate(date, format = 'YYYY-MM-DD') {
  // 实现逻辑...
}
```

## 二十、工程化补充规范

### 20.1 Git 提交规范

- 使用约定式提交（Conventional Commits）：`type(scope): subject`。
- 常用 type：`feat`、`fix`、`docs`、`style`、`refactor`、`test`、`chore`。
- 示例：`feat(admin): 添加团队管理列表页`、`fix(api): 修复请求拦截器 401 处理`。

### 20.2 .gitignore 规范

- 必须忽略 `node_modules/`、`dist/`、`*.local`、`*.log`。
- 环境变量文件仅提交 `.env.example`，不提交 `.env.production` 等含敏感信息的文件。

### 20.3 依赖管理

- 生产依赖和开发依赖严格区分（`dependencies` vs `devDependencies`）。
- 锁定依赖版本（`package-lock.json` 或 `yarn.lock` 必须提交）。
- 定期审查依赖安全漏洞（`npm audit`）。

### 20.4 Code Review 检查项

- 是否符合本规范要求。
- 是否有硬编码的敏感信息或环境相关字符串。
- 是否正确处理了加载、空状态和错误。
- 是否有合理的注释和【可配置】标记。
- 是否在确有必要时才引入复杂技术（如 Pinia）。

### 20.5 国际化（i18n）预留

- 业务状态文字、提示信息不硬编码在组件中，提取到常量或语言包。
- 预留 `src/locales/` 目录结构，便于后续接入 vue-i18n。

### 20.6 日志规范

- 开发环境使用 `console.debug` / `console.info` 输出调试信息，生产环境禁止输出 `console.log`（ESLint `no-console` 拦截）。
- 错误日志统一通过封装的日志工具输出，便于后续接入日志收集服务。

### 20.7 CHANGELOG 维护

- 每次发版更新 `CHANGELOG.md`，按 `Added`、`Changed`、`Fixed`、`Removed` 分类记录。
- 结合 Git 提交记录自动生成或手动维护。

## 二十一、提交前检查清单

- [ ] 开发构建和生产构建均能成功完成（`npm run build`）。
- [ ] ESLint、格式化和自动化测试通过。
- [ ] 页面覆盖加载、空数据、错误和无权限状态。
- [ ] 后台路由需要登录，角色菜单和按钮显示正确。
- [ ] 所有接口通过统一请求实例调用，无写死域名或敏感信息。
- [ ] 表单可防止重复提交，并能展示后端错误。
- [ ] 桌面端和手机端关键页面均完成实际检查。
- [ ] 无明显控制台错误、未处理 Promise 或资源加载失败。

## 二十二、前端代码输出规范

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

## 二十三、当前项目执行要求

- 当前项目使用 Vue 3 + Vite + Element Plus + Axios。
- 当前页面统一使用 `<script setup>`。
- 管理员、区域合伙人、加盟商页面复用统一的组织管理组件。
- 登录页面根据身份调用对应 Controller 的登录接口。
- Google reCAPTCHA 相关密钥只能通过环境变量配置，不能写入源码。

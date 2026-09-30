# Vue3 + Vite 前端开发规范

## 目录

1. 适用范围
2. 基本原则
3. 推荐目录结构
4. 命名规范（4.1 通用命名、4.2 组件命名）
5. Vue 组件规范（5.1 基本约定、5.2 Props 和 Emits、5.3 模板规范）
6. 状态管理
7. 路由与布局（7.1 后台导航栏数据从路由派生）
8. Element Plus 使用规范（8.1 图标规范）
9. 表单规范
10. API 与 Axios（10.1 精简 Axios 封装示例）
11. 加载、空状态与错误处理
12. 样式与响应式设计
13. 可访问性
14. 安全规范
15. 环境与代理（15.1 环境变量配置、15.2 集中配置示例）
16. 性能规范
17. 代码质量与测试（17.1 JavaScript 代码规范、17.2 注释规范）
18. 提交前检查
19. 其他主流规范补充（19.1 Git 提交规范、19.2 .gitignore 规范、19.3 依赖管理、19.4 Code Review 检查项、19.5 国际化预留、19.6 日志规范、19.7 CHANGELOG 维护）

## 1. 适用范围

本规范适用于本项目公开前台和登录后台的 Web 页面开发。

统一技术栈：

- Vue 3
- Vite
- JavaScript（当前阶段不使用 TypeScript）
- Element Plus
- Axios
- Vue Router
- 状态管理优先使用 Props/Emits、provide/inject、组合式函数；确有跨页面全局共享状态时再引入 Pinia（见第 6 章）

## 2. 基本原则

- 使用 Vue 3 Composition API 和 `<script setup>`。
- 页面负责业务编排，组件负责可复用交互，API 模块负责网络通信。
- 前台访客无需登录；后台页面必须经过登录和路由权限校验。
- 前端权限只用于界面控制，真正的权限判定必须由后端完成。
- 优先使用简单、明确的实现，避免在项目早期引入不必要的抽象。
- 不在组件中写死服务器地址、密钥、角色权限和业务状态文字。
- **响应式状态统一使用 `ref` 声明、避免 `reactive`**（理由和示例见第 5.1 章）。
- 一个函数只负责一个明确的业务动作，不把多个业务揉在一个函数里。

## 3. 推荐目录结构

```text
src
├── api             # 按业务模块封装接口
├── assets          # 图片、字体和全局样式
├── components      # 通用组件
├── config          # 【可配置】集中管理应用配置（app.js）、请求配置（request.js）等
├── constants       # 常量与稳定枚举
├── hooks           # 可复用组合式逻辑
├── layouts         # Public、Admin 等布局
├── router          # 路由与守卫
├── stores          # Pinia 状态（仅在确有必要时使用）
├── utils           # request、格式化等工具
├── views
│   ├── public      # 公开前台页面
│   └── admin       # 登录后台页面
├── App.vue
└── main.js
```

- 业务专用组件优先放在对应页面或模块目录下，不全部堆进全局 `components`。
- 组件超过约 200 行时检查是否混合了多个职责；按业务含义拆分，不按行数机械拆分。

## 4. 命名规范

### 4.1 通用命名

- JavaScript 变量和函数使用小驼峰，如 `teamList`、`loadTeams`。
- 常量使用全大写下划线，如 `DEFAULT_PAGE_SIZE`、`API_BASE_URL`。
- 事件处理函数使用 `handle` 前缀，如 `handleSubmit`、`handlePageChange`。
- 获取数据函数使用明确动词，如 `fetchTeamList`、`loadEventDetail`。
- 组合式函数使用 `use` 前缀，如 `usePagination`。
- CSS 类名使用 kebab-case，如 `.team-card__title`；同一项目可采用 BEM 风格并保持一致。
- 避免 `data1`、`temp`、`info` 等含义模糊的命名；避免使用没有业务含义的缩写。

### 4.2 组件命名

- Vue 组件文件使用大驼峰（PascalCase），如 `TeamCard.vue`、`EventEditor.vue`。
- 自定义组件使用 PascalCase 命名，且组件名应使用两个或两个以上单词，避免与 HTML 原生元素冲突（例如用 `TeamCard`，不要用 `Card`）。
- 页面文件使用有业务含义的 PascalCase 名称，例如 `PartnerList.vue`。
- 组件模板中引用自定义组件时使用 PascalCase（如 `<TeamCard />`）；第三方组件保持组件库自身的约定写法（如 `<el-button>`）。

## 5. Vue 组件规范

### 5.1 基本约定

- 统一使用 `<script setup>`，不混用 Options API。
- **统一使用 `ref` 声明所有响应式状态（包括基本类型和对象类型），避免使用 `reactive`。**
  - 理由：`ref` 对基本类型和对象类型均适用，访问时统一通过 `.value`（模板中自动解包），避免 `reactive` 带来的解构丢失响应性、类型推断复杂、深层嵌套难以追踪等问题，便于团队风格统一。
  - 对象类型使用 `ref` 时通过 `.value` 访问属性，如 `state.value.xxx`。
  - 示例：

    ```js
    // ✅ 推荐：统一使用 ref
    const count = ref(0)
    const userInfo = ref({ name: '', age: 0 })
    // 访问：userInfo.value.name
    // ❌ 不推荐：使用 reactive
    const state = reactive({ count: 0, user: { name: '', age: 0 } })
    ```

- 派生状态、计算逻辑使用 `computed`，不要使用 `watch` 重复维护可计算的数据。
- `watch` / `watchEffect` 只用于明确的副作用（监听外部变化），并及时清理定时器、监听器和订阅。
- 生命周期钩子应放在 `<script setup>` 中逻辑结构的靠后位置，先写状态和函数，再挂生命周期。
- 页面离开或组件卸载时（`onUnmounted` 或路由离开守卫）取消不再需要的请求和副作用，如未完成的接口请求、定时器、事件监听。
- 复杂逻辑、工具函数、组合式函数中应加入适当注释，说明意图和参数含义（见第 17.2 章）。

### 5.2 Props 和 Emits

- Props 必须声明类型、是否必填和合理默认值；对象和数组默认值使用工厂函数。
- Props 声明使用 camelCase（如 `teamId`、`userInfo`），模板中传参时使用对应写法。
- 对外事件名称应清晰表达业务动作（如 `update`、`delete`、`page-change`），不用 `click1`、`do2` 这类模糊名字。
- 子组件通过 `emit` 通知父组件，不直接修改 Props。
- 双向绑定应有明确的数据所有者，避免多层组件同时修改同一对象。

### 5.3 模板规范

- 模板必须保持可读性，元素带多个属性时必须换行书写，一个属性一行。
- 统一使用指令缩写：`:`（`v-bind`）、`@`（`v-on`）、`#`（`v-slot`），不写完整指令名。
- 模板中只保留简单表达式，复杂逻辑放入 `computed` 或方法中。
- `v-for` 必须设置稳定且唯一的 `key`，禁止用数组下标代替业务 ID。
- 不要在同一个元素上同时使用 `v-if` 和 `v-for`，需要过滤请先 `computed` 处理数据。
- 可自闭合的自定义组件使用自闭合写法，如 `<TeamCard />`。
- 页面模板中的搜索区、表格区、弹窗区等应添加中文注释，方便快速定位结构。

推荐的单文件组件顺序：

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

## 6. 状态管理

- 仅组件内部使用的状态保留在组件中（使用 `ref`）。
- 父子组件共享状态优先通过 Props 和 Emits。
- 跨层级组件共享状态使用 `provide/inject`，而非引入 Pinia。
- 登录用户、权限、跨页面筛选条件等全局状态，在确有必要时才使用 Pinia；Store 不直接操作 DOM，也不保存不可序列化对象。
- 服务端数据不应在多个 Store 中重复保存；明确唯一数据来源和刷新时机。
- 不把所有接口结果长期缓存到全局状态，避免陈旧数据。

## 7. 路由与布局

- 公开前台和管理后台使用不同布局，例如 `PublicLayout`、`AdminLayout`。
- 路由名称唯一，路径使用小写短横线。
- 后台路由通过 `meta.requiresAuth` 标记需要登录。
- 角色访问范围通过路由元信息和统一守卫控制，但后端仍须再次校验。
- 路由组件采用懒加载，降低首次加载体积。
- 提供 404、无权限和通用错误页面。
- 登录后跳回原目标页面时，必须校验重定向地址，防止开放重定向。

登录与权限校验通过统一的全局前置守卫完成：

```js
// router/index.js
import { APP_CONFIG } from '@/config/app'

router.beforeEach((to) => {
  const token = localStorage.getItem(APP_CONFIG.TOKEN_KEY)
  // 未登录访问后台路由：跳转登录页，并带上原目标地址
  if (to.meta.requiresAuth && !token) {
    return { path: APP_CONFIG.LOGIN_PATH, query: { redirect: to.fullPath } }
  }
  // 已登录再访问登录页：跳回首页
  if (to.path === APP_CONFIG.LOGIN_PATH && token) {
    return { path: APP_CONFIG.HOME_PATH }
  }
})
```

登录成功后从 `route.query.redirect` 读取原目标地址回跳，**回跳前必须校验该地址是否为站内路径**（如以 `/` 开头且不属于外部域名），防止开放重定向。

### 7.1 后台导航栏数据从路由派生

后台管理系统如有左侧导航栏，**导航栏数据应从路由配置中派生**，保持代码干练简洁，避免维护两份菜单数据。

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
    showInMenu: true,        // 是否在菜单中展示
    title: '团队管理',         // 菜单显示文字
    icon: 'fas fa-users',    // 菜单图标（直接写 Font Awesome 类名，见 8.1 图标规范）
    order: 1,                // 排序权重
    roles: ['admin', 'editor'] // 允许访问的角色
  }
}
```

## 8. Element Plus 使用规范

- Element Plus 组件按需引入，避免无必要的整库打包。
- 表单统一使用 `el-form`、`el-form-item` 和 `rules` 完成前端校验。
- 列表使用 `el-table`，分页行为和参数命名保持统一。
- 对话框用于短流程编辑或确认；复杂表单使用独立页面。
- 成功、警告和失败反馈统一使用 `ElMessage` 或 `ElNotification`。
- 删除和不可逆操作必须二次确认，并清楚显示影响对象。
- 主题颜色、间距、圆角和字号集中为 CSS 变量或主题配置。

### 8.1 图标规范

- 统一使用 **Font Awesome** 图标库，不与其他图标库混用。
- **直接在元素上写 FA 类名即可，不搞额外的配置文件**，保持简单：

```vue
<template>
  <!-- 菜单图标 -->
  <i class="fas fa-users"></i>
  <!-- 普通按钮图标 -->
  <button><i class="fas fa-plus"></i> 新增</button>
  <!-- el-button 的 icon 属性接收的是“组件”，不是类名；
       用 FA 图标时应放在默认插槽里，而不是传给 :icon -->
  <el-button><i class="fas fa-search"></i> 查询</el-button>
</template>
```

- 写法要点：
  - 免费版用 `fas`（solid）或 `fab`（brands）前缀，如 `fas fa-users`、`fab fa-github`。
  - 图标类名以字符串形式直接写在 `class` 上，需要动态切换时再用 `:class` 绑定。
  - 菜单、按钮等场景如需根据数据驱动，直接在数据里放 FA 类名字符串：

    ```js
    // 例如路由 meta 或菜单数据中，直接存类名，无需额外映射文件
    {
      path: '/admin/teams',
      meta: { title: '团队管理', icon: 'fas fa-users', order: 1 }
    }
    ```

    ```vue
    <!-- 渲染时直接使用 -->
    <template>
      <i :class="item.meta.icon"></i>
      <span>{{ item.meta.title }}</span>
    </template>
    ```

- 注意：
  - 图标语义应与功能一致，不要为图省事全用 `fa-star`。
  - 不要在多个地方写死重复的图标逻辑；如有统一封装（如 `<icon-name>` 组件），组件内部仍是 FA 类名，不要再加一层配置层。

## 9. 表单规范

- 前端校验用于即时反馈，后端校验才是最终依据。
- 必填、长度、格式和数值范围规则应与后端一致。
- 提交时禁用提交按钮并显示加载状态，避免重复提交。
- 请求失败后保留用户已填写内容，除非业务明确要求清空。
- 服务端字段错误应映射到具体表单项；通用错误显示在表单顶部或消息区。
- 编辑页面加载完成前不得显示错误的默认值。
- 离开有未保存修改的复杂表单时应提示用户。

## 10. API 与 Axios

所有 HTTP 请求必须通过 `src/utils/request.js` 中的 Axios 实例发送，页面组件不得直接创建 Axios 实例，接口请求和页面展示逻辑分离。

基础配置：

- `baseURL` 来自 `import.meta.env.VITE_API_BASE_URL`。
- 默认超时时间 10 秒；上传等长耗时接口可单独调整。
- 请求拦截器统一附加登录凭证和 `requestId`，认证令牌由请求拦截器统一添加，不在各页面重复处理。
- 响应拦截器统一处理标准响应、网络错误和登录失效。
- 收到 401 时清理本地登录状态并跳转登录页，避免多个并发请求重复提示。
- 禁止在日志或错误弹窗中显示完整 Token、请求头或敏感数据。

接口按业务模块封装，API 方法名应与后端接口语义保持一致，并添加中文注释说明用途和参数：

```js
// src/api/team.js
import request from '@/utils/request'

// 分页查询团队列表
// @param {Object} data - 查询条件与分页参数，如 { pageNum, pageSize, keyword }
export function queryTeams(data) {
  return request.post('/team/query', data)
}

// 新增团队
// @param {Object} data - 团队表单数据
export function createTeam(data) {
  return request.post('/team/create', data)
}
```

本项目当前后端以 POST 为主要方法，查询、新增、修改、删除均按后端动作式路径调用。前端不得自行改用其他请求方式导致接口不一致。

### 10.1 精简 Axios 封装示例（带注释）

```js
// src/utils/request.js
import axios from 'axios'
import { ElMessage } from 'element-plus'
import { useAuthStore } from '@/stores/auth' // 仅在确有全局状态时引入
// 【可配置】创建 axios 实例，集中配置基础参数
const request = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL, // 【可配置】API 基础路径，通过环境变量注入
  timeout: 10000, // 【可配置】超时时间（毫秒），长耗时接口可单独覆盖
  headers: {
    'Content-Type': 'application/json'
  }
})
// 请求拦截器：统一附加凭证
request.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('auth_token') // 【可配置】Token 存储 key
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    // 【可配置】附加 requestId 用于链路追踪
    config.headers['X-Request-Id'] = crypto.randomUUID()
    return config
  },
  (error) => Promise.reject(error)
)
// 响应拦截器：统一处理业务响应和异常
request.interceptors.response.use(
  (response) => {
    const res = response.data
    // 【可配置】与后端约定统一的响应格式，此处假设 { code, data, message }
    if (res.code === 200 || res.code === 0) {
      return res.data // 直接返回业务数据，调用方无需再解包
    }
    // 业务错误：统一提示
    ElMessage.error(res.message || '操作失败')
    return Promise.reject(new Error(res.message || 'Business Error'))
  },
  (error) => {
    // HTTP 状态码异常
    if (error.response?.status === 401) {
      // 【可配置】登录失效处理：清理状态并跳转
      localStorage.removeItem('auth_token')
      window.location.href = '/login'
      ElMessage.error('登录已过期，请重新登录')
      return Promise.reject(error)
    }
    if (error.response?.status === 403) {
      ElMessage.error('无权限执行此操作')
      return Promise.reject(error)
    }
    // 网络错误或其他异常
    ElMessage.error(error.message || '网络异常，请稍后重试')
    return Promise.reject(error)
  }
)
export default request
```

## 11. 加载、空状态与错误处理

- 每个异步页面应明确处理加载中、成功、空数据和失败四种状态。
- 首次加载使用骨架屏或区域加载效果，避免整页反复闪烁。
- 操作按钮的加载状态应限制在对应操作范围。
- 可重试错误提供重试入口；权限错误不反复请求。
- 异常不能只在控制台输出而不给用户反馈。
- 多个请求并行时分别维护状态，避免一个请求结束后错误关闭全部加载状态。

## 12. 样式与响应式设计

- 业务组件样式默认使用 `<style scoped>`；公共样式放在全局样式文件，组件专属样式放在组件内。
- CSS 类名使用 kebab-case（见第 4.1 章）。
- 样式按页面结构分组，组与组之间保持换行和缩进，便于阅读。
- 禁止使用内联样式，只有值确实依赖运行时计算时才使用动态样式绑定。
- 布局优先使用 Flexbox 或 Grid，不依赖大量绝对定位。
- 避免使用过深的选择器嵌套，控制在必要的层级内。
- 公共颜色、间距、字号、阴影和层级使用统一设计变量。
- 不随意使用 `!important`；仅在确有第三方组件样式需要覆盖时使用，并说明原因。
- 页面至少适配桌面、平板和手机：
  - 大屏：`>= 1280px`
  - 中屏：`960px - 1279px`
  - 小屏：`760px - 959px`
  - 手机：`< 760px`
- 手机端表格应选择横向滚动、卡片化或隐藏次要列，不得简单挤压到不可读。
- 交互目标应有足够点击区域，避免仅依靠 hover 展示核心功能。

## 13. 可访问性

- 图片必须提供有意义的 `alt`，装饰图片使用空 `alt`。
- 表单控件必须具有可识别标签，不能只依赖 placeholder。
- 键盘能够访问菜单、按钮、对话框和主要操作。
- 焦点状态清晰可见；打开和关闭对话框时正确管理焦点。
- 颜色不能作为状态的唯一表达方式，文字与背景应具有足够对比度。
- 使用原生语义元素，避免把 `div` 伪装成按钮。

## 14. 安全规范

- 禁止使用 `v-html` 渲染未经可信清洗的用户内容。
- 不在 `localStorage` 保存密码和长期敏感信息。
- 前端环境变量会进入构建产物，不得存储任何真正的秘密。
- 上传文件时校验文件类型、大小和数量；后端仍需重新校验。
- 外部链接使用 `target="_blank"` 时增加 `rel="noopener noreferrer"`。
- 不根据客户端传入的角色或用户 ID 判断最终权限。

## 15. 环境与代理

环境变量是本项目最主要的**【可配置】**入口。开发者在代码中搜索「可配置」即可定位所有需要配置的地方；而环境变量本身集中写在 `.env.*` 文件中，无需在代码里散落配置值。

### 15.1 环境变量配置（【可配置】）

```dotenv
# .env.development                    【可配置】开发环境
VITE_API_BASE_URL=/api                # 【可配置】后端接口基础路径（开发环境走 Vite 代理，见 vite.config.js）
VITE_APP_TITLE=My App (Dev)           # 【可配置】应用标题，可用于 <title> 或页面展示
# VITE_API_PORT=3000                  # 【可配置】本地开发服务端口（如需修改默认 5173，在 package.json 的 dev 脚本中 --port）
# VITE_MOCK=true                      # 【可配置】是否开启本地 mock，true/false
```

```dotenv
# .env.production                     【可配置】生产环境
VITE_API_BASE_URL=/api                # 【可配置】生产环境接口基础路径（由网关/反向代理转发）
VITE_APP_TITLE=My App                 # 【可配置】生产环境应用标题
```

**用法与说明：**

| 变量名 | 含义 | 默认值 | 取值范围 / 示例 | 使用方式 |
| --- | --- | --- | --- | --- |
| `VITE_API_BASE_URL` | 后端接口基础路径（上下文） | `/api` | 以 `/` 开头的相对路径（走同源代理），或完整域名 `https://api.example.com` | `axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL })` |
| `VITE_APP_TITLE` | 应用标题 | `My App` | 任意字符串 | `document.title = import.meta.env.VITE_APP_TITLE` |
| `VITE_API_PORT`（可选） | 本地开发端口 | `5173` | 可用端口号 | 在 `package.json` 的 `dev` 脚本中 `--port` |
| `VITE_MOCK`（可选） | 是否启用 mock | `false` | `true` / `false` | 在入口处 `if (import.meta.env.VITE_MOCK === 'true')` 按需加载 |

- **仅 `VITE_` 前缀的变量会暴露给客户端**，变量中不得包含密钥、数据库密码等敏感信息。
- 在代码中通过 `import.meta.env.VITE_XXX` 读取，**不要写死字符串**：

  ```js
  // ✅ 正确：通过环境变量读取
  const baseURL = import.meta.env.VITE_API_BASE_URL || '/api'
  // ❌ 错误：写死地址，换环境就要改代码
  const baseURL = 'http://192.168.1.100:8080/api'
  ```

- 开发环境通过 Vite 代理访问后端，生产环境由网关或反向代理转发 `/api`。代理配置中通过 `configure` 钩子开启转发日志，方便调试时核对转发地址是否正确：

  ```js
  // vite.config.js（示例，说明代理如何消费该配置）
  import { fileURLToPath, URL } from 'node:url'
  import { defineConfig } from 'vite'
  import vue from '@vitejs/plugin-vue'

  export default defineConfig({
    plugins: [vue()],
    resolve: { alias: { '@': fileURLToPath(new URL('./src', import.meta.url)) } },
    server: {
      port: Number(import.meta.env.VITE_API_PORT) || 5173, // 【可配置】开发端口
      proxy: {
        '/api': { // 【可配置】代理上下文，对应 VITE_API_BASE_URL
          target: 'http://localhost:8041/laundryadmin', // 【可配置】本地后端地址（含后端上下文路径）
          changeOrigin: true,
          // 开启转发日志：调试时打印 源地址 -> 目标地址，核对代理是否正确
          configure: (proxy, options) => {
            proxy.on('proxyReq', (proxyReq, req) => {
              const targetUrl = `${options.target}${req.url}`
              console.log(`[Vite 代理转发] ${req.url} -> ${targetUrl}`)
            })
          }
        }
      }
    }
  })
  ```

  说明：后端配置了上下文路径（如 `/laundryadmin`）时，开发环境由代理在 `target` 中统一补齐，前端代码只面向 `/api`。
- 不同环境文件说明：`.env.development`（开发）、`.env.production`（生产）、`.env.staging`（预发，按需）；**环境差异通过配置解决，不在代码中使用域名字符串判断环境**。
- 代理日志仅在开发环境开启，不在生产环境输出敏感请求信息。

### 15.2 集中配置示例（config/）

除环境变量外，部分前端常量集中在 `src/config/` 下管理（同样是【可配置】项）：

```js
// src/config/app.js
// 【可配置】应用级配置集中管理，开发者搜索"可配置"即可定位
export const APP_CONFIG = {
  APP_NAME: import.meta.env.VITE_APP_TITLE || 'Vue App',     // 【可配置】应用名称（取自环境变量）
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
  TIMEOUT: 10000,                    // 【可配置】默认超时（毫秒）
  UPLOAD_TIMEOUT: 60000,             // 【可配置】上传接口超时
  RETRY_COUNT: 0,                    // 【可配置】失败重试次数
  RETRY_DELAY: 1000                  // 【可配置】重试间隔
}
```

## 16. 性能规范

- 路由页面和大型功能按需加载。
- 搜索输入使用防抖，滚动和缩放处理使用节流。
- 大列表使用服务端分页；确有需要时使用虚拟列表。
- 图片设置合理尺寸、格式和懒加载，避免上传原图直接展示。
- 避免在模板中调用高开销函数，使用 `computed` 缓存派生结果。
- 引入新依赖前评估体积、维护状态和是否可由现有能力完成。

## 17. 代码质量与测试

- 启用 ESLint 和 Prettier，并在提交前执行检查。
- 禁止遗留无用途的变量、调试日志和注释代码；删除无用变量、无用导入、调试代码和废弃代码。
- 复杂业务逻辑提取为纯函数或组合式函数，以便测试。
- 核心工具函数、权限判断和状态转换应有单元测试。
- 关键用户流程至少覆盖登录、公开浏览、列表查询、创建和编辑等集成测试。
- 修复缺陷时优先补充复现测试。
- 自动生成的页面必须人工检查响应式布局、空状态、权限和错误处理。

### 17.1 JavaScript 代码规范

- 使用 2 个空格缩进。
- 使用 camelCase 命名变量和函数；常量使用全大写加下划线（如 `API_BASE_URL`）。
- 函数之间保留空行，避免一行代码过长。
- 异步逻辑优先使用 `async/await`。
- 异常必须被处理或继续向上抛出，不能静默忽略。

### 17.2 注释规范

- 公共组件、API、关键业务方法必须添加中文注释，说明功能、参数、返回值（复杂工具函数和组合式函数使用 JSDoc）。
- 注释说明“为什么这样做”，不要只重复代码“做了什么”。
- 复杂的登录、权限、分页、批量操作逻辑必须添加说明。
- 关键业务逻辑在代码中加入行内注释，解释“为什么这样做”而非“做了什么”。
- 所有【可配置】项必须附带注释说明参数含义、取值范围和默认值。
- 注释应及时维护，禁止保留与代码不一致的过时注释。
- 示例：

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

## 18. 提交前检查

- 开发构建和生产构建均能成功完成。
- ESLint、格式化和自动化测试通过。
- 页面覆盖加载、空数据、错误和无权限状态。
- 后台路由需要登录，角色菜单和按钮显示正确。
- 所有接口通过统一请求实例调用，无写死域名或敏感信息。
- 表单可防止重复提交，并能展示后端错误。
- 桌面端和手机端关键页面均完成实际检查。
- 无明显控制台错误、未处理 Promise 或资源加载失败。

## 19. 其他主流规范补充

### 19.1 Git 提交规范

- 使用约定式提交（Conventional Commits）：`type(scope): subject`
- 常用 type：`feat`、`fix`、`docs`、`style`、`refactor`、`test`、`chore`
- 示例：`feat(admin): 添加团队管理列表页`、`fix(api): 修复请求拦截器 401 处理`

### 19.2 .gitignore 规范

- 必须忽略 `node_modules/`、`dist/`、`*.local`、`*.log`
- 环境变量文件仅提交 `.env.example`，不提交 `.env.production` 等含敏感信息的文件

### 19.3 依赖管理

- 生产依赖和开发依赖严格区分（`dependencies` vs `devDependencies`）
- 锁定依赖版本（`package-lock.json` 或 `yarn.lock` 必须提交）
- 定期审查依赖安全漏洞（`npm audit`）

### 19.4 Code Review 检查项

- 是否符合本规范要求
- 是否有硬编码的敏感信息或环境相关字符串
- 是否正确处理了加载、空状态和错误
- 是否有合理的注释和【可配置】标记
- 是否在确有必要时才引入复杂技术（如 Pinia）

### 19.5 国际化（i18n）预留

- 业务状态文字、提示信息不硬编码在组件中，提取到常量或语言包
- 预留 `src/locales/` 目录结构，便于后续接入 vue-i18n

### 19.6 日志规范

- 开发环境使用 `console.debug` / `console.info` 输出调试信息
- 生产环境禁止输出 `console.log`，通过 ESLint 规则 `no-console` 拦截
- 错误日志统一通过封装的日志工具输出，便于后续接入日志收集服务
- Vite 代理转发日志仅在开发环境开启（见第 15.1 章），用于核对转发地址

### 19.7 CHANGELOG 维护

- 每次发版更新 `CHANGELOG.md`，按 `Added`、`Changed`、`Fixed`、`Removed` 分类记录
- 结合 Git 提交记录自动生成或手动维护

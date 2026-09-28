# Vue3 + Vite 前端开发规范

适用范围
本规范适用于本项目公开前台和登录后台的 Web 页面开发。

统一技术栈，版本不追求使用最新版，只选择最新的稳定版本：

- Vue 3
- Vite
- JavaScript（当前阶段不使用 TypeScript）
- Element Plus
- Axios
- Vue Router

状态管理优先使用 Props/Emits、provide/inject、组合式函数；不到万不得已，不得使用 Pinia（见第 6 章）

## 基本原则

1. 使用 Vue 3 Composition API 和 `<script setup>`。
2. 页面负责业务编排，组件负责可复用交互，API 模块负责网络通信。
3. 如果是实现门户，新闻类网站，则前台访客无需登录；如果是后台管理系统，则后台页面必须经过登录和路由权限校验。
4. 前端权限只用于界面控制，真正的权限判定必须由后端完成。
5. 优先使用简单、明确的实现，避免在项目早期引入不必要的抽象。
6. 不在组件中写死服务器地址、密钥、角色权限和业务状态文字。
7. 统一使用 `ref` 声明响应式状态，避免使用 `reactive`，便于团队风格统一。

## 推荐目录结构

```
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

## 命名规范

1. Vue 组件文件使用大驼峰，如 `TeamCard.vue`、`EventEditor.vue`。
2. JavaScript 变量和函数使用小驼峰，如 `teamList`、`loadTeams`。
3. 常量使用全大写下划线，如 `DEFAULT_PAGE_SIZE`。
4. 事件处理函数使用 `handle` 前缀，如 `handleSubmit`、`handlePageChange`。
5. 获取数据函数使用明确动词，如 `fetchTeamList`、`loadEventDetail`。
6. 组合式函数使用 `use` 前缀，如 `usePagination`。
7. CSS 类名使用 kebab-case，如 `.team-card__title`；同一项目可采用 BEM 风格并保持一致。
8. 避免 `data1`、`temp`、`info` 等含义模糊的命名。

## Vue 组件规范

1. 统一使用 `<script setup>`，不混用 Options API。
2. 统一使用 `ref` 声明所有响应式状态（包括基本类型和对象类型），避免使用 `reactive`。> 
> 理由：ref 对基本类型和对象类型均适用，访问时统一通过 `.value`（模板中自动解包），避免 reactive 带来的解构丢失响应性、类型推断复杂、深层嵌套难以追踪等问题，便于团队风格统一。
> 对象类型使用 ref 时通过 `.value` 访问属性，如 `state.value.xxx`。

```
// ✅ 推荐：统一使用 ref
const count = ref(0)
const userInfo = ref({ name: '', age: 0 })
// 访问：userInfo.value.name

// ❌ 不推荐：使用 reactive
const state = reactive({ count: 0, user: { name: '', age: 0 } })
```

3. 派生状态使用 `computed`，不要使用 `watch` 重复维护可计算的数据。
4. `watch` 只用于明确的副作用，并及时清理定时器、监听器和订阅。
5. Props 必须声明类型、是否必填和合理默认值；对象和数组默认值使用工厂函数。
6. 子组件通过 `emit` 通知父组件，不直接修改 Props。
7. 双向绑定应有明确的数据所有者，避免多层组件同时修改同一对象。
8. 列表渲染使用稳定唯一的 key，禁止用数组下标代替业务 ID。
9. 页面离开或组件卸载时取消不再需要的请求和副作用。
10. 复杂逻辑、工具函数、组合式函数中应加入适当注释，说明意图和参数含义（见第 17.1 章）。

推荐的单文件组件顺序：

```
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

## 6 状态管理

1. 仅组件内部使用的状态保留在组件中（使用 ref）。
2. 父子组件共享状态优先通过 Props 和 Emits。
3. 跨层级组件共享状态使用 provide/inject，而非引入 Pinia。
4. 登录用户、权限、跨页面筛选条件等全局状态，在确有必要时才使用 Pinia；Store 不直接操作 DOM，也不保存不可序列化对象。
5. 服务端数据不应在多个 Store 中重复保存；明确唯一数据来源和刷新时机。
6. 不把所有接口结果长期缓存到全局状态，避免陈旧数据。

## 7 路由与布局

1. 公开前台和管理后台使用不同布局，例如 `PublicLayout`、`AdminLayout`。
2. 路由名称唯一，路径使用小写短横线。
3. 后台路由通过 `meta.requiresAuth` 标记需要登录。
4. 角色访问范围通过路由元信息和统一守卫控制，但后端仍须再次校验。
5. 路由组件采用懒加载，降低首次加载体积。
6. 提供 404、无权限和通用错误页面。
7. 登录后跳回原目标页面时，必须校验重定向地址，防止开放重定向。

### 7.1 后台导航栏数据从路由派生

后台管理系统如有左侧导航栏，导航栏数据应从路由配置中派生，保持代码干练简洁，避免维护两份菜单数据。

```
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

路由配置中通过 meta 提供菜单所需信息：

```
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

## 8 Element Plus 使用规范

1. Element Plus 组件按需引入，避免无必要的整库打包。
2. 表单统一使用 `el-form`、`el-form-item` 和 `rules` 完成前端校验。
3. 列表使用 `el-table`，分页行为和参数命名保持统一。
4. 对话框用于短流程编辑或确认；复杂表单使用独立页面。
5. 成功、警告和失败反馈统一使用 `ElMessage` 或 `ElNotification`。
6. 删除和不可逆操作必须二次确认，并清楚显示影响对象。
7. 主题颜色、间距、圆角和字号集中为 CSS 变量或主题配置。

### 8.1 图标规范

统一使用 Font Awesome 图标库，不与其他图标库混用。
直接在元素上写 FA 类名即可，不搞额外的配置文件，保持简单：

```
<template>
  <!-- 菜单图标 -->
  <i class="fas fa-users"></i>
  <!-- 按钮图标 -->
  <button><i class="fas fa-plus"></i> 新增</button>
  <!-- Element Plus 的 icon 插槽里也可用 -->
  <el-button :icon="Search">查询</el-button>
</template>
```

写法要点：

- 免费版用 `fas`（solid）或 `fab`（brands）前缀，如 `fas fa-users`、`fab fa-github`。
- 图标类名以字符串形式直接写在 class 上，需要动态切换时再用 `:class` 绑定。
- 菜单、按钮等场景如需根据数据驱动，直接在数据里放 FA 类名字符串：

```
// 例如路由 meta 或菜单数据中，直接存类名，无需额外映射文件
{
  path: '/admin/teams',
  meta: { title: '团队管理', icon: 'fas fa-users', order: 1 }
}
```

```
<!-- 渲染时直接使用 -->
<template>
  <i :class="item.meta.icon"></i>
  <span>{{ item.meta.title }}</span>
</template>
```

注意：

- 图标语义应与功能一致，不要为图省事全用 `fa-star`。
- 不要在多个地方写死重复的图标逻辑；如有统一封装（如 `<icon-name>` 组件），组件内部仍是 FA 类名，不要再加一层配置层。

## 9 表单规范

1. 前端校验用于即时反馈，后端校验才是最终依据。
2. 必填、长度、格式和数值范围规则应与后端一致。
3. 提交时禁用提交按钮并显示加载状态，避免重复提交。
4. 请求失败后保留用户已填写内容，除非业务明确要求清空。
5. 服务端字段错误应映射到具体表单项；通用错误显示在表单顶部或消息区。
6. 编辑页面加载完成前不得显示错误的默认值。
7. 离开有未保存修改的复杂表单时应提示用户。

## 10 API 与 Axios

1. 所有 HTTP 请求必须通过 `src/utils/request.js` 中的 Axios 实例发送，组件不得直接创建 Axios 实例。
2. 基础配置：
   - baseURL 来自 `import.meta.env.VITE_API_BASE_URL`。
   - 默认超时时间 10 秒；上传等长耗时接口可单独调整。
   - 请求拦截器统一附加登录凭证和 requestId。
   - 响应拦截器统一处理标准响应、网络错误和登录失效。
3. 收到 401 时清理本地登录状态并跳转登录页，避免多个并发请求重复提示。
4. 禁止在日志或错误弹窗中显示完整 Token、请求头或敏感数据。
5. 接口按业务模块封装：

```
// src/api/team.js
import request from '@/utils/request'

export function queryTeams(data) {
  return request.post('/team/query', data)
}

export function createTeam(data) {
  return request.post('/team/create', data)
}
```

> 
> 本项目当前后端以 POST 为主要方法，查询、新增、修改、删除均按后端动作式路径调用。前端不得自行改用其他请求方式导致接口不一致。

### 10.1 精简 Axios 封装示例（带注释）

```
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

## 11 加载、空状态与错误处理

1. 每个异步页面应明确处理加载中、成功、空数据和失败四种状态。
2. 首次加载使用骨架屏或区域加载效果，避免整页反复闪烁。
3. 操作按钮的加载状态应限制在对应操作范围。
4. 可重试错误提供重试入口；权限错误不反复请求。
5. 异常不能只在控制台输出而不给用户反馈。
6. 多个请求并行时分别维护状态，避免一个请求结束后错误关闭全部加载状态。

## 12 样式与响应式设计

1. 组件样式默认使用 `<style scoped>`。
2. 禁止使用内联样式，只有值确实依赖运行时计算时才使用动态样式绑定。
3. 布局优先使用 Flexbox 或 Grid，不依赖大量绝对定位。
4. 公共颜色、间距、字号、阴影和层级使用统一设计变量。
5. 不随意使用 `!important`；确需使用时说明原因。
6. 页面至少适配桌面、平板和手机：
   - 大屏：>= 1280px
   - 中屏：960px - 1279px
   - 小屏：760px - 959px
   - 手机：< 760px
7. 手机端表格应选择横向滚动、卡片化或隐藏次要列，不得简单挤压到不可读。
8. 交互目标应有足够点击区域，避免仅依靠 hover 展示核心功能。

## 13 可访问性

1. 图片必须提供有意义的 alt，装饰图片使用空 alt。
2. 表单控件必须具有可识别标签，不能只依赖 placeholder。
3. 键盘能够访问菜单、按钮、对话框和主要操作。
4. 焦点状态清晰可见；打开和关闭对话框时正确管理焦点。
5. 颜色不能作为状态的唯一表达方式，文字与背景应具有足够对比度。
6. 使用原生语义元素，避免把 div 伪装成按钮。

## 14 安全规范

1. 禁止使用 `v-html` 渲染未经可信清洗的用户内容。
2. 不在 localStorage 保存密码和长期敏感信息。
3. 前端环境变量会进入构建产物，不得存储任何真正的秘密。
4. 上传文件时校验文件类型、大小和数量；后端仍需重新校验。
5. 外部链接使用 `target="_blank"` 时增加 `rel="noopener noreferrer"`。
6. 不根据客户端传入的角色或用户 ID 判断最终权限。

## 15 环境与代理

环境变量是本项目最主要的**【可配置】**入口。开发者在代码中搜索「可配置」即可定位所有需要配置的地方；而环境变量本身集中写在 `.env.*` 文件中，无需在代码里散落配置值。

### 15.1 环境变量配置（【可配置】）

```
# .env.development                    【可配置】开发环境
VITE_API_BASE_URL=/api                # 【可配置】后端接口基础路径（开发环境走 Vite 代理，见 vite.config.js）
VITE_APP_TITLE=My App (Dev)           # 【可配置】应用标题，可用于 <title> 或页面展示
# VITE_API_PORT=3000                  # 【可配置】本地开发服务端口（如需修改默认 5173，在 package.json 的 dev 脚本中 --port）
# VITE_MOCK=true                      # 【可配置】是否开启本地 mock，true/false
```

```
# .env.production                     【可配置】生产环境
VITE_API_BASE_URL=/api                # 【可配置】生产环境接口基础路径（由网关/反向代理转发）
VITE_APP_TITLE=My App                 # 【可配置】生产环境应用标题
```

用法与说明：

| 变量名 | 含义 | 默认值 | 取值范围 / 示例 | 使用方式 |
| --- | --- | --- | --- | --- |
| VITE_API_BASE_URL | 后端接口基础路径（上下文） | /api | 以 / 开头的相对路径（走同源代理），或完整域名 [https://api.example.com](https://api.example.com) | axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL }) |
| VITE_APP_TITLE | 应用标题 | My App | 任意字符串 | document.title = import.meta.env.VITE_APP_TITLE |
| VITE_API_PORT（可选） | 本地开发端口 | 5173 | 可用端口号 | 在 package.json 的 dev 脚本中 --port |
| VITE_MOCK（可选） | 是否启用 mock | false | true / false | 在入口处 if (import.meta.env.VITE_MOCK === 'true') 按需加载 |

- 仅 `VITE_` 前缀的变量会暴露给客户端，变量中不得包含密钥、数据库密码等敏感信息。
- 在代码中通过 `import.meta.env.VITE_XXX` 读取，不要写死字符串：

```
// ✅ 正确：通过环境变量读取
const baseURL = import.meta.env.VITE_API_BASE_URL || '/api'

// ❌ 错误：写死地址，换环境就要改代码
const baseURL = '[http://192.168.1.100:8080/api](http://192.168.1.100:8080/api)'
```

开发环境通过 Vite 代理访问后端，生产环境由网关或反向代理转发 `/api`。

> 
> 开发调试阶段，可配置代理监听事件打印转发地址，方便调试观察；**该日志仅在开发环境生效，生产构建需关闭代理日志**。

```
// vite.config.js（示例，说明代理如何消费该配置，增加代理转发日志）
export default defineConfig({
  server: {
    port: Number(import.meta.env.VITE_API_PORT) || 5173, // 【可配置】开发端口
    proxy: {
      [import.meta.env.VITE_API_BASE_URL]: {  // 【可配置】代理上下文，对应 VITE_API_BASE_URL
        target: '[http://localhost:8080](http://localhost:8080)',       // 【可配置】本地后端地址
        changeOrigin: true,
        // 开发调试用：打印代理转发地址，仅开发环境启用
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

> 
> 代理日志仅在开发环境开启，不在生产环境输出敏感请求信息。

### 15.2 集中配置示例（config/）

除环境变量外，部分前端常量集中在 `src/config/` 下管理（同样是【可配置】项）：

```
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

```
// src/config/request.js
// 【可配置】请求层配置
export const REQUEST_CONFIG = {
  TIMEOUT: 10000,                    // 【可配置】默认超时（毫秒）
  UPLOAD_TIMEOUT: 60000,             // 【可配置】上传接口超时
  RETRY_COUNT: 0,                    // 【可配置】失败重试次数
  RETRY_DELAY: 1000                  // 【可配置】重试间隔
}
```

## 16 性能规范

1. 路由页面和大型功能按需加载。
2. 搜索输入使用防抖，滚动和缩放处理使用节流。
3. 大列表使用服务端分页；确有需要时使用虚拟列表。
4. 图片设置合理尺寸、格式和懒加载，避免上传原图直接展示。
5. 避免在模板中调用高开销函数，使用 computed 缓存派生结果。
6. 引入新依赖前评估体积、维护状态和是否可由现有能力完成。

## 17 代码质量与测试

1. 启用 ESLint 和 Prettier，并在提交前执行检查。
2. 禁止遗留无用途的变量、调试日志和注释代码。
3. 复杂业务逻辑提取为纯函数或组合式函数，以便测试。
4. 核心工具函数、权限判断和状态转换应有单元测试。
5. 关键用户流程至少覆盖登录、公开浏览、列表查询、创建和编辑等集成测试。
6. 修复缺陷时优先补充复现测试。
7. 自动生成的页面必须人工检查响应式布局、空状态、权限和错误处理。

### 17.1 注释规范

1. 复杂逻辑、工具函数、组合式函数必须加入 JSDoc 注释，说明功能、参数、返回值。
2. 关键业务逻辑在代码中加入行内注释，解释"为什么这样做"而非"做了什么"。
3. 所有【可配置】项必须附带注释说明参数含义、取值范围和默认值。

```
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

## 18 提交前检查

- 开发构建和生产构建均能成功完成。
- ESLint、格式化和自动化测试通过。
- 页面覆盖加载、空数据、错误和无权限状态。
- 后台路由需要登录，角色菜单和按钮显示正确。
- 所有接口通过统一请求实例调用，无写死域名或敏感信息。
- 表单可防止重复提交，并能展示后端错误。
- 桌面端和手机端关键页面均完成实际检查。
- 无明显控制台错误、未处理 Promise 或资源加载失败。

## 19 其他主流规范补充

### 19.1 Git 提交规范

使用约定式提交（Conventional Commits）：`type(scope): subject`
常用 type：`feat`、`fix`、`docs`、`style`、`refactor`、`test`、`chore`

示例：

- `feat(admin): 添加团队管理列表页`
- `fix(api): 修复请求拦截器 401 处理`

### 19.2 .gitignore 规范

必须忽略 `node_modules/`、`dist/`、`*.local`、`*.log`
环境变量文件仅提交 `.env.example`，不提交 `.env.production` 等含敏感信息的文件

### 19.3 依赖管理

1. 生产依赖和开发依赖严格区分（dependencies vs devDependencies）
2. 锁定依赖版本（package-lock.json 或 yarn.lock 必须提交）
3. 定期审查依赖安全漏洞（npm audit）

### 19.4 Code Review 检查项

- 是否符合本规范要求
- 是否有硬编码的敏感信息或环境相关字符串
- 是否正确处理了加载、空状态和错误
- 是否有合理的注释和【可配置】标记

### 19.5 国际化（i18n）预留

业务状态文字、提示信息不硬编码在组件中，提取到常量或语言包
预留 `src/locales/` 目录结构，便于后续接入 vue-i18n

### 19.6 日志规范

1. 开发环境使用 `console.debug` / `console.info` 输出调试信息
2. 生产环境禁止输出 `console.log`，通过 ESLint 规则 `no-console` 拦截
3. 错误日志统一通过封装的日志工具输出，便于后续接入日志收集服务

### 19.7 CHANGELOG 维护

每次发版更新 CHANGELOG.md，按 Added、Changed、Fixed、Removed 分类记录
结合 Git 提交记录自动生成或手动维护。

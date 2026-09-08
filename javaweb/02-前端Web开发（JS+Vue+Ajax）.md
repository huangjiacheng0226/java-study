# 02 前端 Web 开发（JavaScript + Vue + Ajax）

## 1. JavaScript 核心语法

### 1.1 引入方式

~~~html
<script src="app.js" defer></script>
~~~

外部 JS 文件只写 JavaScript，不要再写 script 标签。defer 让脚本在 HTML 解析完成后执行。

### 1.2 变量、类型和函数

推荐用 let 声明变量，用 const 声明常量，避免使用 var。

~~~javascript
const apiBase = '/api';
let count = 0;
count += 1;

function sum(a, b) {
  return a + b;
}
~~~

常见类型：string、number、boolean、undefined、null；数组和对象属于引用类型。

#### 1.2.1 输出和运算

~~~javascript
console.log('控制台输出');
document.write('<p>页面输出</p>');
alert('弹窗提示');

const total = 10 + 2 * 3;
const ok = total >= 10 && total < 20;
~~~

JavaScript 的常见运算符包括算术运算符、比较运算符、逻辑运算符和赋值运算符。比较值和类型时优先使用严格相等 `===`，避免隐式类型转换。

#### 1.2.2 类型和类型转换

| 类型 | 示例 | 说明 |
|---|---|---|
| string | `'Tom'` | 字符串 |
| number | `18`、`3.14` | 整数和小数统一为 number |
| boolean | `true`、`false` | 布尔值 |
| undefined | `let value` | 已声明但没有值 |
| null | `null` | 明确表示没有对象 |
| object | `{ id: 1 }` | 对象、数组、日期等 |

表单输入值通常是字符串，需要时使用 `Number(value)`、`String(value)` 或 `Boolean(value)` 转换。

变量名使用字母、数字、`_` 或 `$`，不能以数字开头；常量通常使用全大写加下划线。`typeof null` 的结果是 `"object"`，判断数组应使用 `Array.isArray(value)`。`==` 会先进行隐式转换，初学阶段优先使用 `===` 和 `!==`。

### 1.3 流程控制和函数

~~~javascript
function findMax(a, b) {
  if (a > b) {
    return a;
  }
  return b;
}

for (let i = 0; i < 3; i += 1) {
  console.log(i);
}

const names = ['Tom', 'Jerry'];
names.forEach(name => console.log(name));
~~~

函数可以使用声明式、表达式或箭头函数定义。参数是调用者传入的数据，`return` 把结果交给调用者。数组常用 `push`、`pop`、`slice`、`map`、`filter` 和 `find` 方法。

### 1.4 对象、数组、字符串和 JSON

~~~javascript
const user = { id: 1, name: 'Tom' };
user.name = 'Jerry';

const jsonText = JSON.stringify(user);
const copy = JSON.parse(jsonText);
const message = `用户：${copy.name}`;
~~~

对象使用键值对保存数据，数组保存有序集合。JSON 只能表示数据，不能保存函数；前后端传输 JSON 时，发送端使用 `JSON.stringify`，接收端使用 `response.json()` 或 `JSON.parse`。

### 1.5 DOM 与事件

~~~javascript
const button = document.querySelector('#loadBtn');

button.addEventListener('click', () => {
  document.querySelector('#message').textContent = '加载中...';
});
~~~

推荐使用 addEventListener，普通文本使用 textContent 更新，避免不可信输入直接写入 innerHTML。

BOM 表示浏览器窗口、地址和历史记录等对象；DOM 把 HTML 文档表示为内存中的树。事件通常经历捕获、目标、冒泡三个阶段，需要时可使用 event.stopPropagation() 阻止继续传播。

常用 DOM API：

| API | 作用 |
|---|---|
| `querySelector` | 获取第一个匹配元素 |
| `querySelectorAll` | 获取全部匹配元素 |
| `createElement` | 创建元素 |
| `append` | 添加子节点 |
| `classList.add` | 添加 CSS 类 |
| `addEventListener` | 注册事件监听器 |

事件传播方向是捕获阶段、目标阶段、冒泡阶段。父元素监听点击时，子元素点击可能触发父元素处理；需要阻止时调用 `event.stopPropagation()`。

### 1.6 BOM 常用对象

`window` 表示浏览器窗口，`location` 表示当前地址，`history` 管理浏览历史，`localStorage` 保存本地字符串数据。

~~~javascript
console.log(location.href);
location.assign('/login');
localStorage.setItem('token', 'demo-token');
const token = localStorage.getItem('token');
~~~

不要把密码等高敏感信息直接放入 localStorage；令牌保存位置还要结合 XSS、CSRF 和项目认证方案评估。

## 2. Vue 基础

### 2.1 响应式和模板

~~~html
<div id="app">
  <input v-model="keyword" placeholder="搜索部门">
  <ul>
    <li v-for="dept in filteredDepts" :key="dept.id">
      {{ dept.name }}
    </li>
  </ul>
</div>
<script>
Vue.createApp({
  data() {
    return {
      keyword: '',
      depts: [{ id: 1, name: '教研部' }]
    };
  },
  computed: {
    filteredDepts() {
      return this.depts.filter(d => d.name.includes(this.keyword));
    }
  }
}).mount('#app');
</script>
~~~

Vue 模板中的 `{{ }}` 是文本插值；表达式应保持简单，复杂计算放入 `computed`。修改响应式数据后，Vue 会在下一次更新周期刷新 DOM，可使用 `nextTick` 等待更新完成。

### 2.2 Vue 指令对比

| 指令 | 作用 | 示例 |
|---|---|---|
| v-bind 或冒号 | 绑定属性 | :disabled="loading" |
| v-model | 表单双向绑定 | v-model="form.name" |
| v-on 或 @ | 绑定事件 | @click="save" |
| v-if | 创建或销毁节点 | v-if="visible" |
| v-show | CSS 控制显示 | v-show="visible" |
| v-for | 列表渲染 | v-for="item in list" |

v-for 应使用稳定的 key，优先使用业务主键。

### 2.3 Vue 应用和组件

Vue 应用由根组件、数据、计算属性、方法和生命周期组成。计算属性适合根据已有数据计算新值，方法适合响应用户操作。

~~~javascript
const app = Vue.createApp({
  data() {
    return { count: 0 };
  },
  computed: {
    double() {
      return this.count * 2;
    }
  },
  methods: {
    increment() {
      this.count += 1;
    }
  }
});
app.mount('#app');
~~~

组件把页面拆成可复用的局部区域，父组件可以通过 props 向子组件传值，子组件可以通过事件通知父组件。

### 2.4 Vue 指令使用规范

| 指令 | 作用 | 注意事项 |
|---|---|---|
| `v-bind` / `:` | 绑定属性 | 动态属性不要写成普通字符串 |
| `v-model` | 表单双向绑定 | 适合 input、select、textarea |
| `v-on` / `@` | 绑定事件 | 事件方法写在 methods 中 |
| `v-if` | 条件创建或销毁 | 切换开销较大 |
| `v-show` | 使用 CSS 显示隐藏 | 频繁切换更合适 |
| `v-for` | 列表渲染 | 必须使用稳定的 `key` |
| `v-html` | 插入 HTML | 不要直接插入不可信内容 |

组件通信常见方式：父组件通过 `props` 传入数据，子组件通过自定义事件 `emit` 通知父组件；多个页面共享状态时再使用 Pinia 等状态管理工具。不要在子组件中直接修改 props。

## 3. Ajax 与 Axios

### 3.1 异步请求

同步代码会等待前一步执行完成，异步代码把任务交给浏览器，完成后通过回调、Promise 或 async/await 继续执行。网络请求应使用异步方式，避免阻塞页面。

原生 Ajax 的基本流程是创建 XMLHttpRequest、设置回调、打开连接、发送请求：

~~~javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/depts');
xhr.onload = () => {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.onerror = () => console.error('网络错误');
xhr.send();
~~~

~~~javascript
async function loadDepts() {
  try {
    const response = await fetch('/depts');
    // fetch 对 404/500 不一定自动抛异常
    if (!response.ok) throw new Error('HTTP ' + response.status);
    return await response.json();
  } catch (error) {
    console.error(error);
    return [];
  }
}
~~~

### 3.2 Axios 封装

~~~javascript
import axios from 'axios';

const http = axios.create({
  baseURL: '/api',
  timeout: 10000
});

async function queryDepts() {
  const { data } = await http.get('/depts');
  return data;
}

async function createDept(name) {
  await http.post('/depts', { name });
}
~~~

项目中建议统一配置 baseURL、超时、请求拦截器、响应拦截器和错误提示。

Axios 常用请求方法：

~~~javascript
http.get('/depts', { params: { name: '研发' } });
http.post('/depts', { name: '研发部' });
http.put('/depts/1', { name: '技术部' });
http.delete('/depts/1');
~~~

请求拦截器可统一添加 Token，响应拦截器可统一处理业务码和登录失效：

~~~javascript
http.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
~~~

前端校验用于即时反馈，但后端仍必须再次校验。使用 Fetch 时要检查 response.ok，因为 404/500 不一定让 Promise reject；长列表或页面切换时可用 AbortController 取消过期请求，避免旧响应覆盖新数据。跨域请求还会受到 CORS 策略限制。

请求失败应区分网络错误、HTTP 状态错误和业务错误：网络错误通常没有响应，HTTP 错误可通过 `error.response.status` 判断，业务错误则读取后端约定的 `code`。统一处理后再在页面显示用户能理解的消息。

## 4. 前端工程化

### 4.1 组件和生命周期

created/setup 阶段准备数据，mounted 阶段访问 DOM，beforeUnmount 阶段清理定时器和监听器。组件负责封装可复用页面区域。

### 4.2 Vue Router 和 Element Plus

Vue Router 管理页面路由；Element Plus 提供 Table、Pagination、Dialog、Form 等组件。

Vue 项目常见开发流程是：创建项目、安装依赖、划分组件、定义路由、封装请求、联调接口、执行构建。生产构建通常使用 `npm run build`，生成的静态文件再部署到 Web 服务器。

生命周期常见顺序为：创建组件、挂载 DOM、更新数据、卸载组件。定时器、事件监听和 WebSocket 应在卸载阶段清理，避免组件销毁后仍然执行回调。

### 4.3 Element Plus 常用组件

| 组件 | 用途 | 关键属性或事件 |
|---|---|---|
| Table | 展示列表 | `data`、`prop`、`label` |
| Pagination | 分页 | `current-page`、`page-size`、`current-change` |
| Dialog | 弹窗 | `v-model`、`title`、`width` |
| Form | 表单校验 | `model`、`rules`、`ref` |

列表页通常由查询表单、表格、分页和新增/编辑弹窗组成。分页组件变化时重新请求后端，删除成功后刷新当前页；表单提交前调用校验方法，失败时定位到具体字段。

### 4.4 路由与前端部署

路由把 URL 映射到页面组件。前端路由切换时应显示加载状态，登录后才能访问的页面要做路由守卫，但真正的权限校验仍由后端完成。

生产构建命令通常是 `npm run build`，构建结果位于 `dist`。部署时把 `dist` 放到 Nginx 等 Web 服务器，并配置历史路由回退到 `index.html`；API 代理或 CORS 要与后端地址保持一致。

## 5. 本章总结

- JavaScript 负责行为，DOM 负责文档操作。
- Vue 使用响应式数据驱动视图，Axios 负责 HTTP 请求。
- 组件、路由和组件库是前端工程化基础。

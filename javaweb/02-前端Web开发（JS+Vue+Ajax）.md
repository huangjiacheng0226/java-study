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

### 1.3 DOM 与事件

~~~javascript
const button = document.querySelector('#loadBtn');

button.addEventListener('click', () => {
  document.querySelector('#message').textContent = '加载中...';
});
~~~

推荐使用 addEventListener，普通文本使用 textContent 更新，避免不可信输入直接写入 innerHTML。

BOM 表示浏览器窗口、地址和历史记录等对象；DOM 把 HTML 文档表示为内存中的树。事件通常经历捕获、目标、冒泡三个阶段，需要时可使用 event.stopPropagation() 阻止继续传播。

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

## 3. Ajax 与 Axios

### 3.1 异步请求

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

前端校验用于即时反馈，但后端仍必须再次校验。使用 Fetch 时要检查 response.ok，因为 404/500 不一定让 Promise reject；长列表或页面切换时可用 AbortController 取消过期请求，避免旧响应覆盖新数据。跨域请求还会受到 CORS 策略限制。

## 4. 前端工程化

### 4.1 组件和生命周期

created/setup 阶段准备数据，mounted 阶段访问 DOM，beforeUnmount 阶段清理定时器和监听器。组件负责封装可复用页面区域。

### 4.2 Vue Router 和 Element Plus

Vue Router 管理页面路由；Element Plus 提供 Table、Pagination、Dialog、Form 等组件。

## 5. 本章总结

- JavaScript 负责行为，DOM 负责文档操作。
- Vue 使用响应式数据驱动视图，Axios 负责 HTTP 请求。
- 组件、路由和组件库是前端工程化基础。

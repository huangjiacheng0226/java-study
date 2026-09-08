# 01 前端 Web 开发（HTML + CSS）

## 1. Web 前端与 Web 标准

### 1.1 前端职责

前端负责把文字、图片、表格、表单等数据组织成用户可见的页面。浏览器会解析 HTML、CSS 和 JavaScript，再通过渲染引擎显示结果。

### 1.2 技术分工

| 技术 | 主要职责 | 示例 |
|---|---|---|
| HTML | 页面结构和内容 | 标题、表格、表单 |
| CSS | 页面外观和布局 | 颜色、间距、Flex |
| JavaScript | 页面行为和交互 | 点击、校验、异步请求 |

### 1.3 Web 标准和浏览器工作方式

Web 标准主要由结构、表现和行为三部分组成：HTML 描述内容结构，CSS 描述外观，JavaScript 负责交互行为。分离三者可以减少重复代码，便于维护和无障碍访问。

浏览器访问页面时会先解析 HTML 生成 DOM 树，再解析 CSS 生成 CSSOM，合并后计算布局并绘制页面；脚本可能修改 DOM，从而触发重新布局或重绘。因此脚本应放在合适位置，避免频繁读写布局属性。

## 2. HTML 基础

### 2.1 页面骨架

~~~html
<!doctype html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>部门管理</title>
</head>
<body>
  <h1>部门列表</h1>
</body>
</html>
~~~

head 保存标题、元数据和样式，body 保存用户可见内容。标签建议小写，属性值使用双引号。

### 2.2 常用标签

| 场景 | 标签 | 说明 |
|---|---|---|
| 标题 | h1 到 h6 | 数字越小级别越高 |
| 段落 | p | 表示一段文字 |
| 换行 | br | 插入换行 |
| 链接 | a | href 指定地址 |
| 图片 | img | src 指定路径，alt 提供替代文本 |
| 视频/音频 | video、audio | controls 显示控件 |
| 布局 | div、span | 块级和行内容器 |
| 表格 | table、tr、th、td | 表头使用 th |
| 表单 | form、input、select、button | name 影响提交参数 |

页面主内容应优先使用 header、nav、main、section、footer 等语义标签；表单控件配合 label，图片填写有意义的 alt，并保证交互控件可以通过键盘 Tab 访问。

表格应使用 caption 说明用途，表头使用 th 并通过 scope 标明行列关系：

~~~html
<table>
  <caption>部门列表</caption>
  <thead><tr><th scope="col">编号</th><th scope="col">名称</th></tr></thead>
  <tbody><tr><td>1</td><td>教研部</td></tr></tbody>
</table>
~~~

### 2.3 表单示例

~~~html
<form id="deptForm">
  <label for="name">部门名称</label>
  <input id="name" name="name" maxlength="10" required>
  <select name="status">
    <option value="1">启用</option>
    <option value="0">禁用</option>
  </select>
  <button type="submit">保存</button>
</form>
~~~

### 2.4 路径和常用属性

相对路径以当前 HTML 文件所在目录为基准，绝对路径以 `/` 或完整域名为基准。项目中图片、CSS 和脚本通常使用相对路径，避免把本机磁盘路径写入页面。

| 属性 | 适用标签 | 作用 |
|---|---|---|
| `id` | 任意元素 | 唯一标识元素 |
| `class` | 任意元素 | 关联一个或多个 CSS 类 |
| `href` | `a`、`link` | 指定链接或资源地址 |
| `src` | `img`、`script` | 指定外部资源地址 |
| `alt` | `img` | 图片无法显示时的替代文本 |
| `name` | 表单控件 | 作为提交参数名 |
| `value` | 表单控件 | 控件提交的值 |

### 2.5 表格和表单细节

表格通常由 `table`、`caption`、`thead`、`tbody`、`tr`、`th` 和 `td` 组成。`th` 表示表头，`scope="col"` 表示列标题，`scope="row"` 表示行标题。不要使用表格完成页面整体布局。

表单常见控件包括文本框、密码框、单选框、复选框、下拉框、文件框和按钮：

~~~html
<form action="/depts" method="post">
  <input name="name" type="text" required maxlength="10">
  <input name="status" type="radio" value="1" checked>启用
  <input name="status" type="radio" value="0">禁用
  <button type="submit">保存</button>
</form>
~~~

`label for` 应与控件的 `id` 对应；提交按钮使用 `type="submit"`，普通按钮使用 `type="button"`，避免误提交表单。

表单提交时只有具有 `name` 的控件会生成参数；单选框同名时只能选一个，复选框同名时可能提交多个值。`GET` 会把参数拼到 URL，`POST` 通常把参数放进请求体；前端校验不能代替后端校验。

## 3. CSS 样式

### 3.1 三种引入方式

| 方式 | 特点 | 建议 |
|---|---|---|
| 行内样式 | 写在 style 属性中 | 临时演示 |
| 内部样式 | 写在 style 标签中 | 单页面小项目 |
| 外部样式 | link 引入 CSS 文件 | 正式项目首选 |

### 3.2 选择器

~~~css
* { box-sizing: border-box; }

/* class 选择器可复用 */
.primary { color: #1677ff; }

/* id 选择器通常只使用一次 */
#app { max-width: 960px; margin: 0 auto; }
~~~

选择器优先级大致为：行内样式 > id > class/属性/伪类 > 元素。项目中优先使用 class。

### 3.3 常见选择器和伪类

~~~css
/* 元素、类、ID、后代、子元素选择器 */
p { line-height: 1.6; }
.card { padding: 16px; }
#app { min-height: 100vh; }
.card .title { font-weight: 600; }
.menu > li { list-style: none; }

/* 伪类 */
a:hover { color: #1677ff; }
input:focus { outline: 2px solid #1677ff; }
li:first-child { font-weight: bold; }
~~~

选择器越具体，优先级通常越高；项目中应避免层级过深和大量 `!important`，优先通过清晰的 class 设计样式。

### 3.4 盒子模型和 Flex

盒子由 content、padding、border、margin 组成。box-sizing: border-box 会把内边距和边框计入宽高。

~~~css
.layout {
  display: flex;
  gap: 16px;
  align-items: center;
  justify-content: space-between;
}
.panel {
  width: 320px;
  padding: 16px;
  border: 1px solid #ddd;
  margin: 8px;
}
~~~

Flex 适合一维布局，Grid 适合二维布局。移动端应使用相对单位和媒体查询。

### 3.5 常见布局和响应式

`display: block` 元素独占一行，`inline` 元素不能设置完整宽高，`inline-block` 兼具两者部分特征。Flex 常用属性如下：

| 属性 | 作用 |
|---|---|
| `display: flex` | 启用 Flex 布局 |
| `flex-direction` | 设置主轴方向 |
| `justify-content` | 设置主轴对齐 |
| `align-items` | 设置交叉轴对齐 |
| `flex-wrap` | 是否换行 |
| `gap` | 子元素间距 |

响应式页面可以使用媒体查询：

~~~css
.content { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }

@media (max-width: 768px) {
  .content { grid-template-columns: 1fr; }
}
~~~

### 3.6 CSS 单位和继承

`px` 是固定像素，`%` 相对父元素，`rem` 相对根元素字体大小，`vw` 和 `vh` 相对视口。字体、颜色等属性可以继承，盒模型尺寸通常不会自动继承。

### 3.7 定位、层叠和常见排错

`position: relative` 保留原位置并提供定位参照，`absolute` 脱离普通文档流，`fixed` 相对视口固定，`sticky` 在滚动到阈值后固定。多个元素重叠时可用 `z-index` 调整层叠顺序，但只有定位元素或 Flex/Grid 子项等场景才生效。

页面显示不符合预期时，优先在浏览器开发者工具中检查元素、Computed 样式、盒模型和 Network 请求；常见原因是选择器优先级、继承、路径错误、元素默认 margin 或宽度超出父容器。

## 4. 本章总结

- HTML 负责结构，CSS 负责表现。
- 重点掌握标签、路径、表单、选择器和盒子模型。
- 正式项目优先使用外部 CSS 和可复用 class。

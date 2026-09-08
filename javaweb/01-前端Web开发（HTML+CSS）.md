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

### 3.3 盒子模型和 Flex

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

## 4. 本章总结

- HTML 负责结构，CSS 负责表现。
- 重点掌握标签、路径、表单、选择器和盒子模型。
- 正式项目优先使用外部 CSS 和可复用 class。

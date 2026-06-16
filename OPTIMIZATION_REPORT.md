# 个人主页优化报告

## 一、修复问题：导航栏无法跳转

### 问题分析

经过代码审查，发现导航栏点击后无法跳转到对应区域的原因有 **3 个**：

| 问题 | 位置 | 原因 |
|------|------|------|
| **1. `<base target="_blank">` 标签** | `_includes/head.html` | 该标签导致所有导航链接（包括页面内锚点链接）默认在新窗口打开，破坏了单页内的平滑滚动体验，且重新加载页面时锚点可能失效 |
| **2. 锚点 ID 不匹配** | `_pages/about.md` + `_data/navigation.yml` | Publications 和 Honors and Awards 等 Section 没有显式定义锚点，依赖 Jekyll/Kramdown 自动生成的标题 ID。但导航中使用了 `/#-publications`（带 `-` 前缀），这与自动生成的 ID 可能不一致，导致找不到目标元素 |
| **3. `domain` 变量未定义** | `_includes/masthead.html` | 导航模板使用了 `{{ domain }}` 变量，但 `_config.yml` 中未定义，导致链接解析可能异常 |

### 修复措施

#### 修复 1：移除 `<base target="_blank">`，改用智能外部链接处理
**文件**：`_includes/head.html`

- 移除了 `<head><base target="_blank"></head>`（这本身是非法的嵌套 HTML）
- 添加了一段 JavaScript，在页面加载后自动为所有**外部链接**（非 `#`、非 `/` 开头）添加 `target="_blank"` 和 `rel="noopener noreferrer"`
- 这样站内导航（锚点跳转）可以正常工作，外部链接仍然会在新窗口打开

```javascript
document.addEventListener("DOMContentLoaded", function() {
  document.querySelectorAll('a').forEach(function(link) {
    var href = link.getAttribute('href');
    if (href && !href.startsWith('#') && !href.startsWith('/') && !href.startsWith('mailto:') && !href.startsWith('javascript:')) {
      link.setAttribute('target', '_blank');
      link.setAttribute('rel', 'noopener noreferrer');
    }
  });
});
```

#### 修复 2：为所有 Section 添加显式锚点
**文件**：`_pages/about.md`

为 Publications、Honors and Awards、Education、Work Experience 都添加了显式锚点：

```html
<span class='anchor' id='publications'></span>
# 📝 Publications

<span class='anchor' id='honors-and-awards'></span>
# 🎖 Honors and Awards

<span class='anchor' id='education'></span>
# 📖 Education

<span class='anchor' id='work-experience'></span>
# 📖 Work Experience
```

这样导航栏中的 `/#publications`、`/#honors-and-awards` 等链接可以**精确匹配**到目标位置，不再依赖 Jekyll 自动生成的标题 ID。

#### 修复 3：替换未定义的 `domain` 变量
**文件**：`_includes/masthead.html`

将 `{{ domain }}` 替换为 `{{ site.baseurl }}`：

```liquid
<li class="masthead__menu-item"><a href="{{ site.baseurl }}{{ link.url }}">{{ link.title }}</a></li>
```

这是 Jekyll 的标准变量，确保链接在任何环境下都能正确解析。

---

## 二、扩展性优化：展示更多项目

为了让你后续能够方便地展示技术博客、算法竞赛、项目集等内容，我为你创建了以下扩展页面和模板：

### 新增文件清单

| 文件 | 用途 | 说明 |
|------|------|------|
| `_layouts/post.html` | 博客文章布局 | 用于渲染单篇技术博客文章，包含标题、日期、标签、返回列表页链接 |
| `_pages/blog.md` | 技术博客列表页 | 展示所有博客文章摘要，URL: `/blog/` |
| `_pages/projects.md` | 项目展示页 | 展示研究项目、工程项目、开源贡献，URL: `/projects/` |
| `_pages/competitions.md` | 算法竞赛页 | 展示竞赛记录、奖项、题解，URL: `/competitions/` |
| `_posts/2025-01-01-sample-tech-blog.md` | 博客文章示例 | 演示如何撰写技术博客文章 |

### 导航栏更新

**文件**：`_data/navigation.yml`

新增了 3 个导航入口，同时修复了原有的锚点链接：

```yaml
main:
  - title: "About Me"
    url: "/#about-me"
  - title: "Publications"
    url: "/#publications"
  - title: "Honors and Awards"
    url: "/#honors-and-awards"
  - title: "Education"
    url: "/#education"
  - title: "Work Experience"
    url: "/#work-experience"
  - title: "Blog"          # 新增
    url: "/blog/"
  - title: "Projects"      # 新增
    url: "/projects/"
  - title: "Competitions"  # 新增
    url: "/competitions/"
```

### 使用指南

#### 1. 撰写技术博客

在 `_posts/` 目录下新建 Markdown 文件，文件名格式：`YYYY-MM-DD-title.md`

```yaml
---
layout: post
title: "你的文章标题"
date: 2025-06-16
tags: ["computer-vision", "tutorial"]
excerpt: "文章摘要，会显示在博客列表页"
---

## 正文内容

使用 Markdown 语法写作...
```

Jekyll 会自动将这些文章按时间倒序排列在 `/blog/` 页面中。

#### 2. 添加项目展示

编辑 `_pages/projects.md`，在对应 Section 下添加新的项目卡片：

```markdown
<div class="project-card">
  <h3>项目名称</h3>
  <p>项目描述...</p>
  <p><strong>Tech:</strong> Python, PyTorch</p>
  <a href="https://github.com/...">[Code]</a>
</div>
```

#### 3. 记录算法竞赛

编辑 `_pages/competitions.md`，添加新的竞赛记录：

```markdown
### 2025
- **比赛名称**: 题目描述和解题思路
- **Result**: 奖项 / 排名
- **Solution**: [代码链接]
```

---

## 三、进一步优化的建议

### 1. 样式美化（CSS 建议）

建议为新增页面添加自定义样式，可在 `_sass` 目录中新建 `_custom.scss`，然后在 `assets/css/main.css` 中引入，或在 `_includes/head/custom.html` 中添加内联样式。例如：

```scss
// 博客卡片样式
.blog-card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  transition: box-shadow 0.2s;
}
.blog-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}
.blog-card .tag {
  display: inline-block;
  background: #f0f0f0;
  padding: 0.2rem 0.6rem;
  border-radius: 4px;
  font-size: 0.85rem;
  margin-right: 0.5rem;
}

// 项目卡片样式
.project-card {
  border-left: 4px solid #2c7abe;
  padding-left: 1rem;
  margin-bottom: 2rem;
}
```

### 2. 首页锚点滚动偏移优化

当前 `assets/js/_main.js` 中的平滑滚动偏移是 `offset: -20`，但 sticky 导航栏（masthead）的高度可能大于 20px，导致跳转后 Section 标题被导航栏遮挡。建议修改为更大的偏移量：

```javascript
$("a").smoothScroll({offset: -80});  // 根据 masthead 实际高度调整
```

或更精确地，为 `.anchor` 添加 CSS 偏移：

```scss
.anchor {
  display: block;
  position: relative;
  top: -80px;  // 向上偏移导航栏高度
  visibility: hidden;
}
```

### 3. 添加 "回到顶部" 按钮

在长页面（如首页）添加一个浮动的回到顶部按钮，提升用户体验。可以在 `_includes/scripts.html` 或自定义 JS 中添加：

```javascript
// Back to top button
var backToTop = document.createElement('button');
backToTop.innerHTML = '↑';
backToTop.style.cssText = 'position:fixed;bottom:20px;right:20px;padding:10px 15px;font-size:18px;background:#2c7abe;color:white;border:none;border-radius:50%;cursor:pointer;display:none;z-index:1000;';
document.body.appendChild(backToTop);

window.addEventListener('scroll', function() {
  backToTop.style.display = window.scrollY > 300 ? 'block' : 'none';
});

backToTop.addEventListener('click', function() {
  window.scrollTo({top: 0, behavior: 'smooth'});
});
```

### 4. SEO 优化

- 在 `_config.yml` 中填写 `google_site_verification`、`bing_site_verification` 等 SEO 相关字段
- 确保每篇文章都有 `excerpt` 字段，有助于搜索引擎抓取摘要
- 为图片添加 `alt` 属性

### 5. 响应式导航优化

当前导航栏使用了 `greedy-nav` 插件，在小屏幕上会折叠。如果导航项过多（如现在的 8 个），建议：
- 将 "Education" 和 "Work Experience" 合并为 "About Me" 下的子页面或折叠区域
- 或将独立页面（Blog/Projects/Competitions）使用下拉菜单分组

### 6. 数据驱动管理

对于竞赛记录、项目列表等结构化数据，建议使用 Jekyll 的 `_data` 目录来管理：

**`_data/competitions.yml`**:
```yaml
- year: 2024
  name: "ICPC Asia Regional"
  award: "Gold Medal"
  rank: "12 / 200"
  link: "https://..."
  solution: "https://github.com/..."
```

然后在 `_pages/competitions.md` 中用 Liquid 循环渲染：
```liquid
{% for c in site.data.competitions %}
- **{{ c.name }}** ({{ c.year }}): {{ c.award }} — [Solution]({{ c.solution }})
{% endfor %}
```

这样数据与展示分离，更易于维护和扩展。

---

## 四、提交修改到 GitHub

所有修改已保存在本地仓库中。你可以按以下步骤推送到 GitHub：

```bash
cd /Users/weihong/Documents/kimi/workspace/huangwh.github.io
git add .
git commit -m "fix: navigation anchor links; add blog, projects, competitions pages"
git push origin main
```

GitHub Pages 会自动重新构建站点，约 1-2 分钟后生效。

---

**总结**：本次修复解决了导航栏无法跳转的核心问题，同时为你的主页增加了 Blog、Projects、Competitions 三个扩展页面，并提供了完整的文件模板和使用指南。你可以直接编辑这些 Markdown 文件来填充自己的内容。

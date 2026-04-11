# zhangliang.dev 博客用户手册

> 技术栈：Hugo v0.147.4 (extended) + 自定义主题 + GitHub Pages
> 仓库：LiangZhang0103/LiangZhang0103.github.io (master)
> 网址：https://liangzhang0103.github.io/
> 项目目录：~/claw/blog/

---

## 一、项目结构

```
~/claw/blog/
├── hugo.toml                          # 站点配置（标题、URL、菜单、参数）
├── .github/workflows/deploy.yml       # GitHub Actions 自动部署
├── .gitignore
├── archetypes/default.md              # 文章模板（hugo new 时使用）
├── content/
│   ├── posts/                         # 博客文章（核心内容）
│   │   ├── plm-selection-guide.md
│   │   ├── mbd-practice.md
│   │   └── python-catia-automation.md
│   └── pages/
│       └── about.md                   # 关于页
└── themes/zhangliang-dev/             # 自定义主题
    └── layouts/
        ├── index.html                 # 首页（Hero + 终端卡片 + 文章网格 + 关于）
        ├── posts/
        │   ├── list.html              # 文章列表页（日期 + 标题 + 标签）
        │   └── single.html            # 文章详情页
        └── _default/
            ├── baseof.html            # 基础布局（导航栏 + 页脚 + CSS）
            ├── single.html            # 通用页面 fallback
            ├── list.html              # 通用列表 fallback
            └── about.html             # 关于页（统计卡片 + 技能列表）
```

---

## 二、日常写作流程

### 1. 创建新文章

```bash
cd ~/claw/blog
hugo new content posts/my-new-post.md
```

生成的文件自动包含 front matter 模板：

```yaml
---
title: "My New Post"
date: 2026-04-11T16:40:00+08:00
emoji: "📝"            # 首页文章卡片上的图标
tags: []               # 标签，如 ["PLM实施", "CATIA"]
draft: true            # 草稿状态，发布前改为 false
summary: ""            # 文章摘要，显示在列表页和首页卡片上
---
```

### 2. 本地预览

```bash
hugo server -D         # -D 包含草稿，默认 http://localhost:1313
```

在浏览器打开 http://localhost:1313/ 实时预览，修改文件后自动刷新。

### 3. 编辑文章

用任意编辑器打开 markdown 文件编写内容。支持的 markdown 扩展：

- 代码高亮（Dracula 主题，带行号）
- 表格
- 数学公式（如需）
- HTML 内联（front matter 中 unsafe: true）

### 4. 发布文章

```bash
# 1. 将 draft: true 改为 draft: false（或删除该行）
# 2. 提交并推送
git add -A
git commit -m "post: 新文章标题"
git push
```

推送后 GitHub Actions 自动构建部署，约 30 秒生效。

### 5. 查看部署状态

```bash
gh run list --repo LiangZhang0103/LiangZhang0103.github.io --limit 1
```

---

## 三、Front Matter 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| title | 是 | 文章标题 |
| date | 是 | 发布日期，`hugo new` 自动生成 |
| emoji | 否 | 首页卡片图标，默认 📝 |
| tags | 否 | 标签数组，如 `["PLM", "CATIA"]`，显示在卡片和详情页 |
| draft | 否 | `true` 为草稿不发布，`false` 或缺省为正式发布 |
| summary | 否 | 摘要文字，留空则 Hugo 自动截取正文前 70 字 |

---

## 四、站点配置 (hugo.toml)

```toml
baseURL = 'https://liangzhang0103.github.io/'
languageCode = 'zh-CN'
title = 'zhangliang.dev'
theme = 'zhangliang-dev'

[params]
  description = 'PLM / CAD / MBD / 数字化转型。10年+ 一线实施经验...'
  author = '张亮'
  github = 'https://github.com/zhangliang862093'   # 导航栏 GitHub 按钮
  zhihu = 'https://www.zhihu.com/'                  # 页脚知乎链接
  email = 'zhangliang862093@163.com'                # 页脚邮箱链接

[menus]
  [[menus.main]]
    name = '首页'     # 导航栏菜单项
    url = '/'
    weight = 1
  [[menus.main]]
    name = '文章'
    url = '/posts/'
    weight = 2
  [[menus.main]]
    name = '关于'
    url = '/about/'
    weight = 3
```

修改配置后 `git push` 即可生效。

---

## 五、主题定制 (themes/zhangliang-dev/)

### 修改全局样式

编辑 `layouts/_default/baseof.html` 中的 `<style>` 标签。关键 CSS 变量：

```css
:root {
  --color-bg: #ffffff;           /* 页面背景 */
  --color-text: #171717;         /* 主文字色 */
  --color-accent: #0072f5;       /* 强调色（链接、标签） */
  --color-border: rgba(0,0,0,0.08);
  --max-width: 1200px;           /* 内容最大宽度 */
}
```

### 修改首页

编辑 `layouts/index.html`：
- Hero 区域（标题、描述、按钮）
- 终端卡片内容（profile.yaml 文本）
- 最新文章网格
- 关于作者区域

### 修改导航栏

编辑 `layouts/_default/baseof.html` 中的 `<nav>` 部分，或在 `hugo.toml` 的 `[menus]` 中增删菜单项。

### 修改页脚

编辑 `layouts/_default/baseof.html` 中的 `<footer>` 部分。

### 字体

使用 Geist Sans + Geist Mono，通过 jsdelivr CDN 加载。如需更换：
1. 修改 `baseof.html` 中的 `<link>` 标签
2. 修改 CSS 中的 `font-family`

---

## 六、关于页编辑

文件：`content/pages/about.md`

```yaml
---
title: "关于"
layout: about
url: /about/
---
```

正文使用标准 Markdown。页面自动渲染：
- 左侧：正文内容
- 右侧：技能列表（硬编码在 about.html 模板中）
- 底部：统计卡片（10+年、20+项目、5行业、PMP）

如需修改技能列表或统计数据，编辑 `layouts/_default/about.html`。

---

## 七、开发调试

### 本地完整构建（模拟生产环境）

```bash
cd ~/claw/blog
hugo --minify
# 输出到 public/ 目录，可用任意 HTTP 服务器预览
python3 -m http.server 8080 --directory public
```

### 清理缓存重新构建

```bash
rm -rf public/ resources/
hugo --minify
```

### 检查构建结果

```bash
hugo --minify 2>&1
# 输出类似：
# Total in 89 ms
# Pages            | 31
# Paginator pages   |  0
# Non-page files    |  0
# Static files      |  0
# Processed images  |  0
# Aliases           |  9
# Sitemaps          |  1
# Cleaned           |  0
```

---

## 八、常用 Git 操作

```bash
# 查看修改状态
git status

# 查看修改内容
git diff

# 提交所有修改
git add -A && git commit -m "描述"

# 推送到远程
git push

# 查看部署状态
gh run list --repo LiangZhang0103/LiangZhang0103.github.io --limit 3

# 查看某次部署详情
gh run view <run-id> --repo LiangZhang0103/LiangZhang0103.github.io

# 手动触发部署
gh workflow run deploy.yml --repo LiangZhang0103/LiangZhang0103.github.io --ref master
```

---

## 九、常见问题

### Q: 推送后网站没更新？

1. 检查 Actions 是否成功：`gh run list --limit 1`
2. 文章是否 `draft: false`
3. GitHub Pages 缓存，等待 1-2 分钟或清除浏览器缓存

### Q: 新文章没出现在首页？

- 确认 `draft: false`
- 确认文件在 `content/posts/` 目录下
- 首页只显示最新 6 篇

### Q: 本地预览正常，线上样式异常？

- 检查 `hugo.toml` 中 `baseURL` 是否正确
- 确认 `hugo --minify` 构建无报错

### Q: 如何添加自定义域名？

1. 在项目根目录创建 `static/CNAME` 文件，内容为域名（如 `blog.zhangliang.dev`）
2. 在域名 DNS 添加 CNAME 记录指向 `liangzhang0103.github.io`
3. GitHub 仓库 Settings > Pages > Custom domain 填入域名

### Q: 如何添加图片？

将图片放在 `static/images/` 下，文章中引用：

```markdown
![描述](/images/my-image.png)
```

---

## 十、快速参考卡片

```
新建文章    hugo new content posts/标题.md
本地预览    hugo server -D
生产构建    hugo --minify
提交发布    git add -A && git commit -m "msg" && git push
查看部署    gh run list --limit 1
编辑配置    vim hugo.toml
编辑首页    vim themes/zhangliang-dev/layouts/index.html
编辑关于    vim content/pages/about.md
```

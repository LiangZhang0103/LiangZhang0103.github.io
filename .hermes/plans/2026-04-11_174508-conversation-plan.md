# 计划评审：博客标签系统 — 基于标签查找相关文章

## 评审结论：原方案有 3 个问题需修正，修正后可执行

---

## 问题 1：相关文章算法 — 应使用 Hugo 内置 Related API（严重）

### 原方案

手动遍历所有文章，用 `intersect` 计算标签交集，再用 `dict` + `append` 构建列表排序：

```go
{{ $related := slice }}
{{ range where .Site.RegularPages "Section" "posts" }}
  {{ $common := intersect $tags .Params.tags }}
  {{ if gt (len $common) 0 }}
    {{ $related = $related | append (dict "page" . "weight" (len $common)) }}
  {{ end }}
{{ end }}
```

### 问题

1. **这段模板代码在 Hugo 中无法按预期工作** — Hugo Go 模板不支持对 slice of dicts 按某个 key 排序，`sort` 函数对复杂结构支持有限
2. **性能差** — O(n²) 全量遍历，文章多了会拖慢构建
3. **重复造轮子** — Hugo 从 v0.27 起内置了 `.Site.RegularPages.Related .` 方法，经过多年优化

### 修正方案

使用 Hugo 内置 Related API：

```go
{{ $related := .Site.RegularPages.Related . | first 3 }}
```

同时在 `hugo.toml` 中配置推荐策略：

```toml
[related]
  threshold = 0
  includeNewer = true
  toLower = false
  [[related.indices]]
    name = 'tags'
    weight = 80
  [[related.indices]]
    name = 'date'
    weight = 10
```

好处：
- 内置算法：标签匹配 + 时间衰减 + 可配置权重
- 零模板代码，一行调用
- Hugo 核心团队维护，性能有保障
- 可扩展：未来加 category、keyword 只需改配置

---

## 问题 2：标签云页过早引入（轻微）

### 现状

博客目前 3 篇文章，8 个标签。标签云页（`/tags/`）内容会非常单薄。

### 建议

标签云页可以建，但优先级放最后。先确保核心链路跑通：

**优先级排序**：
1. 启用 taxonomy（hugo.toml）
2. term.html（标签详情页）
3. 文章页标签可点击
4. 相关文章推荐（Related API）
5. 首页卡片标签可点击
6. 标签云页 terms.html（可后补）

---

## 问题 3：标签详情页缺少 RSS feed 链接（遗漏）

### 说明

Hugo 为每个 taxonomy term 自动生成 RSS feed（`/tags/catia/index.xml`），这是 Hugo 最佳实践的标准能力。

标签详情页 `term.html` 应该包含一个 RSS 订阅链接，让读者可以订阅特定标签的文章更新。

### 修正

在 term.html 中添加：

```html
<link rel="alternate" type="application/rss+xml" 
      title="{{ .Title }} - {{ .Site.Title }}" 
      href='{{ with .OutputFormats.Get "RSS" }}{{ .Permalink }}{{ end }}'>
```

同时在页面上加一个 "RSS 订阅此标签" 按钮。

---

## 其他评审通过的部分

| 项目 | 评审结果 |
|------|----------|
| 删除 disableKinds 启用 taxonomy | 正确 |
| term.html 使用分隔线布局 | 合理，风格一致 |
| single.html 标签 span 改 a 链接 | 正确 |
| index.html article-badge 改 a 链接 | 正确 |
| 中文标签 URL encoding 由 Hugo 处理 | 正确 |
| 最多显示 3 篇相关文章 | 合理 |
| 无相关文章时不显示区块 | 正确 |
| 导航栏加标签入口 | 可选，同意 |

---

## 修正后的实施计划

### 文件变更清单

| 文件 | 操作 | 变更内容 |
|------|------|----------|
| `hugo.toml` | 修改 | 删 disableKinds，加 [taxonomies] + [related] |
| `layouts/_default/term.html` | 新建 | 标签详情页 + RSS 链接 |
| `layouts/_default/terms.html` | 新建 | 标签云页（优先级最低） |
| `layouts/posts/single.html` | 修改 | 标签可点击 + 相关文章（Related API） |
| `layouts/index.html` | 修改 | 卡片标签可点击 |
| `layouts/posts/list.html` | 不改 | 已有标签 badge，后续可考虑可点击化 |

### 执行顺序

1. hugo.toml — 启用 taxonomy + 配置 related
2. single.html — 标签可点击 + Related API 相关文章
3. term.html — 标签详情页
4. index.html — 首页卡片标签可点击
5. 本地构建验证
6. terms.html — 标签云页（可后补）

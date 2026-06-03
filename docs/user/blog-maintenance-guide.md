---
doc_type: user-guide
slug: blog-maintenance-guide
component: astro-blog
status: current
summary: Astro 博客工程的日常使用、发文、更新、预览、构建、发布和维护指南。
tags:
  - blog
  - astro
  - maintenance
last_reviewed: 2026-06-02
---

# 博客维护使用指南

这份指南面向博客维护者，说明如何在当前工程里添加文章、更新内容、管理图片和 PDF、调整站点信息、预览构建、发布上线以及做日常维护。

当前项目是 **Astro + pnpm** 博客，不是 Hexo。文章是 Markdown 文件，主要放在 `src/content/posts/` 下；页面、主题样式、站点配置和资源入口由 `src/`、`public/` 和根目录配置文件共同维护。

## 1. 先认识这个工程

### 1.1 常用目录

| 路径 | 用途 | 平时会不会改 |
|---|---|---|
| `src/content/posts/` | 博客文章 Markdown | 最常改 |
| `src/content/spec/about.md` | 关于页正文 | 偶尔改 |
| `public/images/` | 公开图片资源，访问路径以 `/images/...` 开头 | 常改 |
| `public/pdfs/` | 公开 PDF 资源，访问路径以 `/pdfs/...` 开头 | 常改 |
| `src/config.ts` | 站点标题、导航、头像、个人简介、协议、目录等配置 | 偶尔改 |
| `src/pages/resources.astro` | 资源页 PDF 列表 | 添加资源时改 |
| `src/pages/notes.astro` | 笔记页，筛选 `category: "笔记"` 的文章 | 很少改 |
| `astro.config.mjs` | Astro、Markdown 插件、构建集成配置 | 谨慎改 |
| `package.json` | 命令脚本和依赖 | 谨慎改 |
| `dist/` | 构建输出目录 | 不手动改 |
| `.astro/` | Astro 缓存目录 | 不手动改 |
| `node_modules/` | 依赖目录 | 不手动改 |

### 1.2 常用命令

本项目指定包管理器是 `pnpm@9.14.4`，并且 `package.json` 里有 `preinstall` 限制。日常请使用 `pnpm`，不要用 `npm install` 或 `yarn install`。

| 命令 | 作用 | 什么时候用 |
|---|---|---|
| `pnpm install` | 安装依赖 | 第一次拉项目、依赖变更后 |
| `pnpm dev` | 启动本地开发服务器 | 写文章、改页面时 |
| `pnpm new-post -- my-post` | 创建新文章草稿 | 发新文章时 |
| `pnpm check` | Astro 项目检查 | 改代码、改配置后 |
| `pnpm type-check` | TypeScript 类型检查 | 改 TS/组件后 |
| `pnpm format` | 用 Biome 格式化 `src/` | 改代码后 |
| `pnpm lint` | 用 Biome 检查并自动修复 `src/` | 改代码后 |
| `pnpm build` | 构建静态站点，并生成 Pagefind 搜索索引 | 发布前必须跑 |
| `pnpm preview` | 本地预览构建后的 `dist/` | 发布前抽查 |

推荐的日常启动方式：

```powershell
cd C:\Blog
pnpm install
pnpm dev
```

开发服务器默认地址通常是：

```text
http://localhost:4321/
```

如果端口被占用，Astro 会在终端输出新的端口，以终端提示为准。

## 2. 添加一篇新文章

### 2.1 创建文章文件

推荐使用项目自带脚本：

```powershell
pnpm new-post -- dian-lu-zan-tai-fen-xi
```

这会创建：

```text
src/content/posts/dian-lu-zan-tai-fen-xi.md
```

文件名会成为文章 URL 的 slug。上面的文章上线后地址是：

```text
/posts/dian-lu-zan-tai-fen-xi/
```

命名建议：

- 使用英文、小写、数字、连字符，或拼音连字符，例如 `gao-shu-bi-ji.md`。
- 不建议使用空格。
- 不建议频繁改文件名，因为文件名一改，文章 URL 也会变。
- 中文文件名技术上可以存在，但 URL 会被编码，不利于复制、分享和长期维护。

脚本会自动补 `.md` 后缀。如果想用 MDX，也可以显式写 `.mdx`，但当前文章基本都是 `.md`，日常建议继续用 Markdown。

### 2.2 新文章的 frontmatter

新建后，脚本会生成类似下面的头部信息：

```yaml
---
title: dian-lu-zan-tai-fen-xi
published: 2026-06-02
description: ''
image: ''
tags: []
category: ''
draft: false
lang: ''
---
```

建议你马上改成更完整的版本：

```yaml
---
title: "电路暂态分析笔记"
published: 2026-06-02
updated: 2026-06-02
description: "整理 RC、RL 电路暂态响应的基本公式、求解步骤和例题。"
image: "/images/circuit-cover.png"
tags:
  - "电路"
  - "笔记"
  - "暂态分析"
category: "笔记"
draft: true
lang: "zh_CN"
---
```

字段说明：

| 字段 | 是否重要 | 说明 |
|---|---:|---|
| `title` | 必填 | 页面标题、文章卡片标题、SEO 标题。建议用正式中文标题。 |
| `published` | 必填 | 发布时间。首页和归档按它倒序排列。格式建议 `YYYY-MM-DD`。 |
| `updated` | 可选 | 更新时间。只有和 `published` 不同才会在文章页显示。 |
| `description` | 推荐 | 文章摘要。文章列表优先显示它；为空时会取正文第一个段落作为摘要。 |
| `image` | 可选 | 封面图。为空则没有封面。 |
| `tags` | 推荐 | 标签列表，会进入标签页和文章元信息。 |
| `category` | 推荐 | 单个分类，会进入分类页。写成 `"笔记"` 时会出现在 `/notes/` 页面。 |
| `draft` | 推荐 | `true` 表示草稿，线上构建不会发布；本地开发仍可看到。 |
| `lang` | 可选 | 文章语言。中文文章建议 `zh_CN`。 |

已有文章里可能有 `categoryPath` 字段，这是历史内容留下的字段。当前代码主要使用 `category`，日常新文章不需要再加 `categoryPath`。

### 2.3 草稿和发布

写文章时建议先设：

```yaml
draft: true
```

这样本地 `pnpm dev` 可以看到草稿，但正式构建时不会进入线上站点。

准备发布时改为：

```yaml
draft: false
```

注意：当前项目没有“定时发布”逻辑。只要 `draft: false`，即使 `published` 写的是未来日期，生产构建也会把文章发布出来，并且可能排在首页最前面。

### 2.4 写正文

frontmatter 后面就是 Markdown 正文：

```markdown
# 电路暂态分析笔记

这篇整理 RC、RL 电路暂态响应的基本思路。

## 一阶电路的三要素法

三要素包括初始值、稳态值和时间常数。
```

写作建议：

- 正文第一个普通段落会被当作自动摘要的来源。即使你填了 `description`，也建议第一段写得清楚。
- 标题层级从 `#` 或 `##` 开始，不要跳得太乱。
- 文章内目录来自标题，站点配置里当前启用了 TOC，深度是 `2`。
- 代码块要写语言名，方便高亮。

代码块示例：

````markdown
```c
int main(void) {
    return 0;
}
```
````

命令行示例：

````markdown
```shellsession
pnpm dev
pnpm build
```
````

当前工程用 Expressive Code 渲染代码块，`shellsession` 的行号会被关闭，更适合展示终端命令。

## 3. 文章里的图片和附件

### 3.1 使用公共图片

如果图片要被多篇文章复用，放到：

```text
public/images/
```

例如：

```text
public/images/circuit-cover.png
```

文章 frontmatter 里这样引用：

```yaml
image: "/images/circuit-cover.png"
```

正文里这样引用：

```markdown
![电路封面](/images/circuit-cover.png)
```

凡是放在 `public/` 里的文件，访问路径都从网站根路径开始，不需要写 `public`。

### 3.2 使用文章专属图片

如果图片只属于某一篇文章，推荐使用文章文件夹：

```text
src/content/posts/dian-lu-zan-tai-fen-xi/
  index.md
  cover.png
  figure-01.png
```

创建方式：

```powershell
pnpm new-post -- dian-lu-zan-tai-fen-xi/index.md
```

frontmatter：

```yaml
image: "cover.png"
```

正文：

```markdown
![三要素法示意图](figure-01.png)
```

这种方式的好处是文章和图片放在一起，迁移、删除、备份都更直观。

### 3.3 使用 PDF

公共 PDF 放到：

```text
public/pdfs/
```

例如：

```text
public/pdfs/电路原理.pdf
```

文章里链接：

```markdown
> PDF 资源：[打开 PDF：电路原理.pdf](/pdfs/电路原理.pdf)
```

如果要让 PDF 出现在资源页，还需要改 `src/pages/resources.astro`，在 `resources` 数组里加一项：

```ts
{
  title: "电路原理",
  description: "电路原理课程 PDF 笔记",
  href: "/pdfs/电路原理.pdf",
},
```

资源页只读这个数组，不会自动扫描 `public/pdfs/`。所以添加 PDF 后，如果希望资源页展示，一定要同步改 `resources.astro`。

## 4. 分类、标签和专题页

### 4.1 分类

文章分类是单个字符串：

```yaml
category: "笔记"
```

当前常见分类：

- `"笔记"`：课程笔记、学习笔记。会进入 `/notes/` 页面。
- `"杂物"`：杂项文章、建站记录、零散资料。

分类页和归档页会自动统计分类。分类名称要保持一致，不要一会儿写 `"笔记"`，一会儿写 `"学习笔记"`，否则会被当成两个分类。

特别注意：`src/pages/notes.astro` 目前只筛选：

```ts
post.data.category === "笔记"
```

所以如果一篇课程笔记没有出现在“笔记”页，优先检查 `category` 是否严格等于 `"笔记"`。

### 4.2 标签

标签是字符串数组：

```yaml
tags:
  - "C语言"
  - "指针"
  - "作业"
```

建议：

- 同一个概念使用同一个写法，例如统一用 `"C语言"`，不要混用 `"C"`、`"c-lang"`、`"C 语言"`。
- 标签数量控制在 2 到 5 个。
- 标签适合描述主题，分类适合描述文章归属。

### 4.3 首页、归档页、标签页如何更新

这些页面不需要手动改：

- 首页文章列表来自 `src/content/posts/`。
- 归档页按 `published` 排序。
- 分类页统计 `category`。
- 标签页统计 `tags`。
- 文章上一篇/下一篇链接按发布时间自动生成。

改完文章后，重新构建即可更新。

## 5. 更新已有文章

### 5.1 找到文章文件

如果知道 slug，直接找：

```powershell
Get-ChildItem src\content\posts
```

如果只记得标题或关键词：

```powershell
rg -n "高数笔记" src\content\posts
rg -n "三要素法" src\content\posts
```

### 5.2 修改正文

直接编辑对应的 `.md` 文件即可。推荐同步检查：

- 标题是否仍准确。
- `description` 是否需要更新。
- `tags` 是否需要新增或合并。
- `category` 是否仍属于正确分类。
- 是否需要新增或更新封面图。

### 5.3 修改更新时间

如果只是改错字，不一定需要加 `updated`。

如果内容有实质更新，建议写：

```yaml
updated: 2026-06-02
```

文章页会显示更新时间；文章列表目前隐藏更新时间，但文章详情页会显示。

### 5.4 不要轻易改文件名

文件名就是 URL。比如：

```text
src/content/posts/gpio.md
```

对应：

```text
/posts/gpio/
```

如果改成：

```text
src/content/posts/stm32-gpio.md
```

URL 会变成：

```text
/posts/stm32-gpio/
```

当前项目没有看到专门的重定向配置。改名后，旧链接可能失效。确实要改名时，请做三件事：

```powershell
rg -n "gpio" src public docs
```

检查站内链接并改掉旧 slug。

然后本地预览旧链接和新链接，确认没有误伤。

最后在提交信息里说明 URL 变更。

## 6. 删除文章或资源

### 6.1 删除文章前检查引用

假设要删除：

```text
src/content/posts/old-post.md
```

先搜索引用：

```powershell
rg -n "old-post|旧文章标题" src public docs
```

如果其他文章、资源页、关于页、导航或文档里有链接，先更新它们。

### 6.2 删除图片或 PDF 前检查引用

删除公共图片：

```text
public/images/example.png
```

先搜：

```powershell
rg -n "/images/example.png|example.png" src public docs
```

删除 PDF：

```text
public/pdfs/电路原理.pdf
```

先搜：

```powershell
rg -n "电路原理.pdf|/pdfs/电路原理.pdf" src public docs
```

尤其要检查 `src/pages/resources.astro`，因为资源页 PDF 列表是手写数组。

## 7. 本地预览与发布前检查

### 7.1 写作时预览

```powershell
pnpm dev
```

打开：

```text
http://localhost:4321/
```

建议检查：

- 首页有没有新文章。
- 文章页标题、摘要、分类、标签是否正确。
- 封面图是否显示。
- 正文图片是否显示。
- 代码块是否高亮。
- 数学公式是否渲染。
- 移动端宽度下有没有排版问题。

### 7.2 发布前完整检查

发布前建议按顺序跑：

```powershell
pnpm check
pnpm type-check
pnpm build
pnpm preview
```

`pnpm build` 会做两件事：

1. `astro build` 构建静态站点到 `dist/`。
2. `pagefind --site dist` 为构建结果生成搜索索引。

`pnpm preview` 用来预览构建后的 `dist/`，比 `pnpm dev` 更接近线上效果。

发布前至少手动点一遍：

- `/`
- `/posts/新文章-slug/`
- `/archive/`
- `/categories/`
- `/tags/`
- `/notes/`
- `/resources/`
- `/about/`
- 搜索框

### 7.3 Git 提交流程

内容改完、构建通过后：

```powershell
git status
git add src/content/posts public/images public/pdfs src/pages/resources.astro src/config.ts docs
git commit -m "Add circuit transient analysis post"
git push
```

如果只改了一篇文章，不要把无关文件一起提交。`dist/`、`.astro/`、`node_modules/` 都在 `.gitignore` 里，正常不需要提交。

## 8. 发布和部署

当前工程是静态站点。构建结果在：

```text
dist/
```

常见发布方式：

| 方式 | 做法 |
|---|---|
| 托管平台自动部署 | 推送 Git 仓库，平台执行 `pnpm install` 和 `pnpm build`，发布 `dist/`。 |
| 手动部署 | 本地运行 `pnpm build`，把 `dist/` 内容上传到静态服务器。 |
| 本地服务器预览 | 运行 `pnpm preview`，只用于本地检查，不是长期生产服务。 |

如果使用 Vercel、Netlify、GitHub Pages 或其他静态托管平台，构建设置通常是：

```text
Install command: pnpm install
Build command: pnpm build
Output directory: dist
```

`astro.config.mjs` 里当前有：

```ts
site: "https://stellatol.github.io/",
base: "/",
trailingSlash: "always",
```

如果将来换域名或换部署子路径，需要同步检查这几个字段。

## 9. 维护站点配置

### 9.1 修改站点标题、副标题、语言

文件：

```text
src/config.ts
```

位置：

```ts
export const siteConfig: SiteConfig = {
  title: "Stellato的星空",
  subtitle: "学习笔记、技术探索与灵感闪光",
  lang: "zh_CN",
}
```

改完后跑：

```powershell
pnpm check
pnpm build
```

### 9.2 修改导航栏

文件：

```text
src/config.ts
```

位置：

```ts
export const navBarConfig: NavBarConfig = {
  links: [
    LinkPreset.Home,
    {
      name: "笔记",
      url: "/notes/",
    },
    {
      name: "资源",
      url: "/resources/",
    },
    LinkPreset.Archive,
    {
      name: "分类",
      url: "/categories/",
    },
    {
      name: "标签",
      url: "/tags/",
    },
    LinkPreset.About,
  ],
}
```

添加站内链接：

```ts
{
  name: "项目",
  url: "/projects/",
}
```

添加外部链接：

```ts
{
  name: "GitHub",
  url: "https://github.com/StellatoL",
  external: true,
}
```

### 9.3 修改头像、简介、社交链接

文件：

```text
src/config.ts
```

位置：

```ts
export const profileConfig: ProfileConfig = {
  avatar: "/images/st_w.png",
  name: "Stellato",
  bio: "电信工程学生，记录学习笔记、技术探索和数字花园的长期生长。",
  links: [
    {
      name: "GitHub",
      icon: "fa6-brands:github",
      url: "https://github.com/StellatoL",
    },
  ],
}
```

头像文件通常放在 `public/images/`，路径写成 `/images/...`。

### 9.4 修改关于页

文件：

```text
src/content/spec/about.md
```

这里是普通 Markdown，可以直接更新个人介绍、网站日志、技能栈等内容。

改完后访问：

```text
/about/
```

### 9.5 修改资源页

文件：

```text
src/pages/resources.astro
```

新增 PDF 时：

1. 把 PDF 放进 `public/pdfs/`。
2. 在 `resources` 数组里新增一项。
3. 本地访问 `/resources/` 检查能否打开。

### 9.6 修改版权协议

文件：

```text
src/config.ts
```

位置：

```ts
export const licenseConfig: LicenseConfig = {
  enable: true,
  name: "CC BY-NC-SA 4.0",
  url: "https://creativecommons.org/licenses/by-nc-sa/4.0/",
}
```

如果不想显示文章底部版权信息，可以设置：

```ts
enable: false
```

## 10. Markdown 增强能力

当前项目在 `astro.config.mjs` 中启用了多种 Markdown 插件。

### 10.1 数学公式

支持 KaTeX。

行内公式：

```markdown
电容电压满足 $u_C(t)=U_\infty+(U_0-U_\infty)e^{-t/\tau}$。
```

块级公式：

```markdown
$$
i(t)=I_\infty+(I_0-I_\infty)e^{-t/\tau}
$$
```

### 10.2 提示块

支持 GitHub 风格提示块：

```markdown
> [!NOTE]
> 这是一个说明。
```

也支持 directive 形式：

```markdown
:::note[说明]
这是一个说明。
:::

:::tip[提示]
这里写一个技巧。
:::

:::warning[注意]
这里写注意事项。
:::
```

当前配置里的类型包括：

- `note`
- `tip`
- `important`
- `caution`
- `warning`

### 10.3 GitHub 仓库卡片

可以写：

```markdown
::github{repo="withastro/astro"}
```

页面会渲染成 GitHub 仓库卡片。这个卡片会在浏览器端请求 GitHub API，如果网络不可用，卡片可能显示加载失败，但不会影响文章主体。

### 10.4 搜索索引

搜索由 Pagefind 在构建后生成：

```powershell
pnpm build
```

`pagefind.yml` 当前排除了 KaTeX 和搜索面板自身，避免公式或搜索 UI 污染索引。

如果新文章在本地开发环境里能看到，但搜索搜不到，先运行一次 `pnpm build`，再用 `pnpm preview` 检查构建后的搜索。

## 11. 常见问题排查

### 11.1 新文章本地可见，线上不可见

优先检查：

```yaml
draft: true
```

生产构建会排除草稿。要发布请改成：

```yaml
draft: false
```

### 11.2 新文章没有出现在“笔记”页

检查分类是否严格等于：

```yaml
category: "笔记"
```

`"学习笔记"`、`"note"`、`"笔 记"` 都不会进入当前的 `/notes/` 页面。

### 11.3 构建时报 frontmatter 错误

检查 `src/content/config.ts` 里的 schema。当前文章集合要求：

- `title` 是字符串。
- `published` 是日期。
- `updated` 如果存在也是日期。
- `draft` 是布尔值。
- `tags` 是字符串数组。
- `category` 是字符串或空值。

常见错误：

```yaml
tags: "电路"
```

应该写成：

```yaml
tags:
  - "电路"
```

常见错误：

```yaml
draft: "false"
```

应该写成：

```yaml
draft: false
```

### 11.4 图片不显示

如果是公共图片：

```markdown
![图](/images/example.png)
```

确认文件存在：

```text
public/images/example.png
```

如果是文章本地图片：

```markdown
![图](figure-01.png)
```

确认文件和文章在同一个文章文件夹：

```text
src/content/posts/my-post/index.md
src/content/posts/my-post/figure-01.png
```

还要注意大小写。部署到 Linux 平台时，`Cover.png` 和 `cover.png` 是两个不同文件名。

### 11.5 PDF 链接 404

文章里写：

```markdown
[打开 PDF](/pdfs/电路原理.pdf)
```

就必须存在：

```text
public/pdfs/电路原理.pdf
```

路径不要写成 `public/pdfs/...`，浏览器访问时不带 `public`。

### 11.6 搜索结果没有新内容

搜索索引只在构建时生成。运行：

```powershell
pnpm build
pnpm preview
```

然后在 preview 站点里测试搜索。

### 11.7 首页排序不对

首页按 `published` 倒序排列。检查日期：

```yaml
published: 2026-06-02
```

不要把月份和日期写反，也不要给未发布文章写未来日期并设置 `draft: false`。

### 11.8 依赖安装失败

确认使用的是：

```powershell
pnpm install
```

不要用：

```powershell
npm install
yarn install
```

项目里有 `preinstall` 限制，会阻止错误的包管理器。

## 12. 日常维护清单

### 12.1 每次发文前

- 文件名使用稳定 slug。
- `title` 是正式标题。
- `published` 日期正确。
- `description` 清晰可读。
- `draft` 发布前改成 `false`。
- `category` 和 `tags` 没有拼写分裂。
- 图片、PDF 链接能打开。
- 本地 `pnpm dev` 预览正常。
- 发布前 `pnpm build` 通过。

### 12.2 每周或每次集中更新后

```powershell
pnpm check
pnpm type-check
pnpm build
```

然后用 `pnpm preview` 抽查首页、文章页、归档、资源页和搜索。

### 12.3 每月维护

- 检查是否有未提交文章：`git status`。
- 检查草稿：`rg -n "draft: true" src/content/posts`。
- 检查资源页 PDF 链接是否仍有效。
- 检查首页、归档页、分类页、标签页是否能正常打开。
- 检查 `public/images/` 和 `public/pdfs/` 是否有明显无用的大文件。
- 备份 `src/content/posts/`、`public/images/`、`public/pdfs/`、`src/config.ts`。

### 12.4 依赖更新

依赖更新不要和发文混在一个提交里。建议单独开一次维护：

```powershell
pnpm outdated
```

确认要更新后再执行更新命令。更新后至少跑：

```powershell
pnpm check
pnpm type-check
pnpm build
pnpm preview
```

如果更新 Astro、Svelte、Tailwind、Pagefind、Biome 或 Markdown 相关插件，要重点检查：

- 文章列表是否正常。
- 文章页 Markdown 是否正常。
- 代码块是否正常。
- 数学公式是否正常。
- 搜索是否正常。
- 图片优化是否正常。

## 13. 推荐工作流

### 13.1 写一篇普通文章

```powershell
cd C:\Blog
pnpm new-post -- my-new-post
pnpm dev
```

编辑：

```text
src/content/posts/my-new-post.md
```

写完后：

```powershell
pnpm build
pnpm preview
git status
git add src/content/posts/my-new-post.md
git commit -m "Add my new post"
git push
```

### 13.2 写一篇带专属图片的文章

```powershell
cd C:\Blog
pnpm new-post -- my-new-post/index.md
```

放图片：

```text
src/content/posts/my-new-post/cover.png
src/content/posts/my-new-post/figure-01.png
```

frontmatter：

```yaml
image: "cover.png"
```

正文：

```markdown
![图 1](figure-01.png)
```

检查：

```powershell
pnpm dev
pnpm build
```

### 13.3 添加一份 PDF 资源

1. 把 PDF 放到 `public/pdfs/`。
2. 新建或更新一篇文章，在正文里加 PDF 链接。
3. 如果要出现在资源页，编辑 `src/pages/resources.astro` 的 `resources` 数组。
4. 运行 `pnpm build`。
5. 用 `pnpm preview` 打开 `/resources/` 和对应文章检查链接。

### 13.4 更新站点个人信息

1. 改 `src/config.ts` 的 `siteConfig` 或 `profileConfig`。
2. 如需改关于页正文，改 `src/content/spec/about.md`。
3. 运行 `pnpm check` 和 `pnpm build`。
4. 本地检查首页侧栏、导航栏和 `/about/`。

## 14. 外部参考

- Astro CLI：<https://docs.astro.build/en/reference/cli-reference/>
- Astro Content Collections：<https://docs.astro.build/en/guides/content-collections/>
- pnpm run：<https://pnpm.io/cli/run>
- Pagefind：<https://pagefind.app/docs/>

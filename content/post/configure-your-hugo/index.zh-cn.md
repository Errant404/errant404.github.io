---
title: "配置你的 Hugo"
slug: configure-your-hugo
description: 
date: 2023-08-04T21:11:59+08:00
image: hugo-original-wordmark.svg
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - documents
tags:
    - 博客
---

本文为 Hugo 官方文档中 [configuration](https://gohugo.io/getting-started/configuration/) 部分的翻译，翻译时间为 2023/8/4，此时 Hugo 最新的 Release 版本为 v0.116.1，请注意文档的时效性。

## 关于配置文件

Hugo 使用 `hugo.toml`, `hugo.yaml`，或者是 `hugo.json`（如果在根目录下被发现的话）作为默认的站点配置文件。

用户可以选择使用命令行选项`--config`用一个或多个站点配置文件覆盖该默认值。

例如：

```bash
hugo --config debugconfig.toml
hugo --config a.toml,b.toml,c.toml
```

> 多个站点配置文件可以作为逗号分隔的字符串指定给 `--config` 选项。

## hugo.toml vs config.toml

在 Hugo 0.110.0 版本中，我们将默认的配置基本文件名更改为`hugo`，例如`hugo.toml`。我们仍然会查找`config.toml`等文件，但我们建议您最终将其重命名（但如果您想支持旧版本的 Hugo，则需要等待）。我们这样做的主要原因是让代码编辑器和构建工具更容易识别它作为 Hugo 配置文件和项目的标识。

## 配置目录

除了使用单个站点配置文件外，您还可以使用`configDir`目录（默认为`config/`）来更轻松地组织和管理环境特定的设置。

- 每个文件代表一个配置的根对象，例如`params.toml`用于`[Params]`，`menu(s).toml`用于`[Menu]`，`languages.toml`用于`[Languages]`等等...

- 每个文件的内容必须是顶级的，例如：

  `hugo.yaml`:

  ```yaml
  Params:
    foo: bar
  ```

  `params.yaml`:

  ```yaml
  foo: bar
  ```

- 每个目录保存了一组文件，其中包含特定于某个环境的设置。

- 文件可以进行本地化，以便成为特定语言的配置文件。

  ```txt
  ├── config
  │   ├── _default
  │   │   ├── hugo.toml
  │   │   ├── languages.toml
  │   │   ├── menus.en.toml
  │   │   ├── menus.zh.toml
  │   │   └── params.toml
  │   ├── production
  │   │   ├── hugo.toml
  │   │   └── params.toml
  │   └── staging
  │       ├── hugo.toml
  │       └── params.toml
  ```

考虑上面的结构，在运行`hugo --environment staging`命令时，Hugo 会使用`config/_default`目录下的所有设置，并将`staging`目录的设置合并在其之上。

我们通过一个例子来更好地理解。假设您正在为您的网站使用 Google Analytics。这需要在`hugo.toml`文件中指定`googleAnalytics = "G-XXXXXXXX"`。现在考虑以下情况：

- 您不希望在开发环境，即`localhost`中加载 Analytics 代码。
- 您想要为生产和演示环境使用不同的 Google Analytics ID，例如：
  - 生产环境使用`G-PPPPPPPP`
  - 演示环境使用`G-SSSSSSSS`

根据上述情况，您需要配置您的`hugo.toml`文件如下：

1. 在`_default/hugo.toml`中，您根本不需要提及`googleAnalytics`参数。这确保在开发服务器上运行`hugo server`时不会加载任何 Google Analytics 代码。这有效是因为，默认情况下，当您运行`hugo server`时，Hugo 会将`Environment=development`，这将使用`_default`文件夹中的配置文件。

2. 在`production/hugo.toml`中，您只需要一行：

   `googleAnalytics = "G-PPPPPPPP"`

   您无需在此配置文件中再次提及所有其他参数，如`title`、`baseURL`、`theme`等等。您只需要提及与生产环境不同或新的参数。这是因为 Hugo 将把这个配置与`_default/hugo.toml`进行**合并**。现在，当您运行`hugo`（构建命令）时，默认情况下 Hugo 会将`Environment=production`，因此生产网站中将包含`G-PPPPPPPP`的 Analytics 代码。

3. 类似地，在`staging/hugo.toml`中，您只需要一行：

   `googleAnalytics = "G-SSSSSSSS"`

   现在您需要告诉 Hugo 您正在使用演示环境。因此，您的构建命令应为`hugo --environment staging`，这将在您的演示网站中加载`G-SSSSSSSS`的Analytics代码。

> 当使用`hugo server`时，默认环境为**development**，当使用`hugo`命令时，默认环境为**production**。

## 合并主题配置

`_merge`配置项可以有以下值：

**none**

不进行合并。

**shallow**

只添加新键的值。

**deep**

添加新键的值，并合并现有的值。

请注意，您不需要像下面的默认设置那样冗长；如果未设置，较高层级的`_merge`值将被继承。

`hugo.yaml`：

```yaml
build:
  _merge: none
caches:
  _merge: none
cascade:
  _merge: none
frontmatter:
  _merge: none
imaging:
  _merge: none
languages:
  _merge: none
  en:
    _merge: none
    menus:
      _merge: shallow
    params:
      _merge: deep
markup:
  _merge: none
mediatypes:
  _merge: shallow
menus:
  _merge: shallow
minify:
  _merge: none
module:
  _merge: none
outputformats:
  _merge: shallow
params:
  _merge: deep
permalinks:
  _merge: none
privacy:
  _merge: none
related:
  _merge: none
security:
  _merge: none
sitemap:
  _merge: none
taxonomies:
  _merge: none
```

## 所有的配置项设置

以下是Hugo定义的变量的完整列表。用户可以选择覆盖其站点配置文件中的这些值。

### archetypeDir 

**默认值：** "archetypes"

Hugo 查找原型文件（内容模板）的目录。还可以参考[模块挂载配置](https://gohugo.io/hugo-modules/configuration/#module-configuration-mounts)，以另一种方式配置这个目录（从 Hugo 0.56 版本开始）。

### assetDir 

**默认值：** "assets"

Hugo 查找用于[Hugo Pipes](https://gohugo.io/hugo-pipes/) 的资源文件的目录。还可以参考[模块挂载配置](https://gohugo.io/hugo-modules/configuration/#module-configuration-mounts)，以另一种方式配置这个目录（从 Hugo 0.56 版本开始）。

### baseURL 

您发布的站点的绝对 URL（包括协议、主机、路径和斜杠），例如 `https://www.example.org/docs/`。

### build 

参见[配置构建](https://gohugo.io/getting-started/configuration/#configure-build)。

### buildDrafts (false) 

**默认值：** false

构建时是否包含草稿。

### buildExpired 

**默认值：** false

是否包含已过期的内容。

### buildFuture 

**默认值：** false

是否包含发布日期在未来的内容。

### caches 

参见[配置文件缓存](https://gohugo.io/getting-started/configuration/#configure-file-caches)。

### cascade 

将默认配置值（前置元数据）传递给内容树中的页面。站点配置中的选项与页面前置元数据中的选项相同，请参阅[前置元数据级联](https://gohugo.io/content-management/front-matter#front-matter-cascade)。

### canonifyURLs 

**默认值：** false

启用后，将相对 URL 转换为绝对 URL。

### cleanDestinationDir 

**默认值：** false

构建时，删除目标中在静态目录中找不到的文件。

### contentDir 

**默认值：** "content"

Hugo 读取内容文件的目录。还可以参考[模块挂载配置](https://gohugo.io/hugo-modules/configuration/#module-configuration-mounts)，以另一种方式配置这个目录（从 Hugo 0.56 版本开始）。

### copyright 

**默认值：** ""

版权声明，通常显示在网站的页脚。

### dataDir 

**默认值：** "data"

Hugo 读取数据文件的目录。还可以参考[模块挂载配置](https://gohugo.io/hugo-modules/configuration/#module-configuration-mounts)，以另一种方式配置这个目录（从 Hugo 0.56 版本开始）。

### defaultContentLanguage 

**默认值：** "en"

没有语言标识的内容将默认使用此语言。

### defaultContentLanguageInSubdir 

**默认值：** false

在子目录中呈现默认内容语言，例如 `content/en/`。网站根目录 `/` 将重定向到 `/en/`。

### disableAliases 

**默认值：** false

禁用别名重定向的生成。请注意，即使设置了 `disableAliases`，别名本身仍将保留在页面中。使用此选项可以在 `.htaccess`、Netlify 的 `_redirects` 文件或类似的情况下生成 301 重定向。

### disableHugoGeneratorInject 

**默认值：** false

Hugo 默认情况下会在*首页*的 HTML 头部插入生成器元标签。您可以关闭它，但我们真的希望您不要这样做，因为这是观察 Hugo 的受欢迎程度的好方法。

### disableKinds 

**默认值：** []

启用对指定的 *Kinds* 的所有页面进行禁用。允许在此列表中使用的值有："page"、"home"、"section"、"taxonomy"、"term"、"RSS"、"sitemap"、"robotsTXT"、"404"。

### disableLiveReload 

**默认值：** false

禁用浏览器窗口的自动实时重新加载。

### disablePathToLower 

**默认值：** false

不要将 URL/路径转换为小写。

### enableEmoji 

**默认值：** false

为页面内容启用 Emoji 表情符号支持；参见[Emoji 表情符号秘笈](https://www.webpagefx.com/tools/emoji-cheat-sheet/)。

### enableGitInfo 

**默认值：** false

为每个页面启用 `.GitInfo` 对象（如果 Hugo 站点已通过 Git 进行版本控制）。这将使用内容文件的最后一次 git 提交日期更新每个页面的 `Lastmod` 参数。

### enableInlineShortcodes 

**默认值：** false

启用行内短代码支持。参见[行内短代码](https://gohugo.io/templates/shortcode-templates/#inline-shortcodes)。

### enableMissingTranslationPlaceholders 

**默认值：** false

如果翻译缺失，显示一个占位符，而不是默认值或空字符串。

### enableRobotsTXT 

**默认值：** false

启用 `robots.txt` 文件的生成。

### frontmatter 

参见[前置元数据配置](https://gohugo.io/getting-started/configuration/#configure-front-matter)。

### googleAnalytics 

**默认值：** ""

Google Analytics 跟踪 ID。

### hasCJKLanguage 

**默认值：** false

如果为 true，则自动检测内容中的中文/日文/韩文语言。这将使 `.Summary` 和 `.WordCount` 对于 CJK 语言正确地工作。

### imaging 

参见[图像处理配置](https://gohugo.io/content-management/image-processing/#imaging-configuration)。

### languageCode 

**默认值：** ""

由[RFC 5646](https://datatracker.ietf.org/doc/html/rfc5646) 定义的语言标签。此值用于填充：

- 内部[RSS 模板](https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/_default/rss.xml) 中的 `<language>` 元素。
- 内部[别名模板](https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/alias.html) 中 `<html>` 元素的 `lang` 属性。

### languages 

参见[配置语言](https://gohugo.io/content-management/multilingual/#configure-languages)。

### disableLanguages 

参见[禁用语言](https://gohugo.io/content-management/multilingual/#disable-a-language)。

### markup 

参见[配置标记](https://gohugo.io/getting-started/configuration-markup)。

### mediaTypes 

参见[配置媒体类型](https://gohugo.io/templates/output-formats/#media-types)。

### menus 

参见[菜单](https://gohugo.io/content-management/menus/#define-in-site-configuration)。

### minify 

参见[配置压缩](https://gohugo.io/getting-started/configuration/#configure-minify)。

### module 

模块配置参见[模块配置](https://gohugo.io/hugo-modules/configuration/)。

### newContentEditor 

**默认值：** ""

创建新内容时使用的编辑器。

### noChmod 

**默认值：** false

不同步文件的权限模式。

### noTimes 

**默认值：** false

不同步文件的修改时间。

### outputFormats 

参见[配置输出格式](https://gohugo.io/getting-started/configuration/#configure-additional-output-formats)。

### paginate 

**默认值：** 10

默认的分页每页元素数目，参见[分页](https://gohugo.io/templates/pagination/)。

### paginatePath 

**默认值：** "page"

用于分页的路径元素（`https://example.com/page/2`）。

### permalinks 

参见[内容管理](https://gohugo.io/content-management/urls/#permalinks)。

### pluralizeListTitles 

**默认值：** true

在列表中使用复数形式的标题。

### publishDir 

**默认值：** "public"

Hugo 将写入最终静态站点的目录（包含 HTML 文件等）。

### related 

参见[相关内容](https://gohugo.io/content-management/related/#configure-related-content)。

### relativeURLs 

**默认值：** false

启用后，所有相对 URL 将相对于内容根目录。请注意，这不影响绝对 URL。

### refLinksErrorLevel 

**默认值：** "ERROR"

使用 `ref` 或 `relref` 解析页面链接时，如果无法解析链接，将使用此日志级别记录。有效值为 `ERROR`（默认值）或 `WARNING`。任何 `ERROR` 都会导致构建失败（`exit -1`）。

### refLinksNotFoundURL 

用于在 `ref` 或 `relref` 中找不到页面引用时使用的占位符 URL。使用原样。

### removePathAccents 

**默认值：** false

从内容路径中的[组合字符](https://en.wikipedia.org/wiki/Precomposed_character)中删除[非间隔标记](https://www.compart.com/en/unicode/category/Mn)。

```text
content/post/hügó.md --> https://example.org/post/hugo/
```

### rssLimit 

**默认值：** -1（无限制）

RSS 订阅中的最大项数。

### sectionPagesMenu 

参见[菜单](https://gohugo.io/content-management/menus/#define-in-site-configuration)。

### security 

参见[安全策略](https://gohugo.io/about/security-model/#security-policy)。

### sitemap 

默认的[站点地图配置](https://gohugo.io/templates/sitemap-template/#configuration)。

### summaryLength 

**默认值：** 70

`.Summary` 中显示的文本长度（按照单词计算），用于自动摘要拆分。

### taxonomies 

参见[配置分类法](https://gohugo.io/content-management/taxonomies#configure-taxonomies)。

### theme 

参见[模块配置](https://gohugo.io/hugo-modules/configuration/#module-configuration-imports) 如何导入主题。

### themesDir 

**默认值：** "themes"

Hugo 从中读取主题的目录。

### timeout 

**默认值：** "30s"

生成页面内容的超时时间，以[持续时间](https://pkg.go.dev/time#Duration) 或秒为单位指定。*注意：* 这用于递归内容生成的退出。如果您的页面生成缓慢（例如因为它们需要大型图像处理或依赖于远程内容），则可能需要增加此限制。

### timeZone 

时区（或地理位置），例如 `Europe/Oslo`，用于解析没有此类信息的前置元数据日期和[`time`函数](https://gohugo.io/functions/time/)。有效值列表可能与系统相关，但应包括 `UTC`、`Local` 和[IANA 时区数据库](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) 中的任何位置。

### title 

站点标题。

### titleCaseStyle 

**默认值：** "ap"

参见[配置标题大小写](https://gohugo.io/getting-started/configuration/#configure-title-case)。

### uglyURLs 

**默认值：** false

启用后，URL 将采用 `/filename.html` 的形式，而不是 `/filename/`。

### watch 

**默认值：** false

监视文件系统的更改，并根据需要重新创建。

> 如果您在 *nix 系统上开发您的网站，则以下命令行快捷方式很有用，可以找到配置选项：
>
> ```text
> cd ~/sites/yourhugosite
> hugo config | grep emoji
> ```
>
> 输出示例：
>
> ```text
> enableemoji: true
> ```

## 配置构建

`build` 配置部分包含全局构建相关的配置选项。

`hugo.yaml`:

```yaml
build:
  buildStats:
    disableClasses: false
    disableIDs: false
    disableTags: false
    enable: false
  cachebusters:
  - source: assets/.*\.(js|ts|jsx|tsx)
    target: (js|scripts|javascript)
  - source: assets/.*\.(css|sass|scss)$
    target: (css|styles|scss|sass)
  - source: (postcss|tailwind)\.config\.js
    target: (css|styles|scss|sass)
  - source: assets/.*\.(.*)$
    target: $1
  noJSConfigInAssets: false
  useResourceCacheWhen: fallback
```

### buildStats

[自v0.115.1新增](https://github.com/gohugoio/hugo/releases/tag/v0.115.1)

当启用时，在项目根目录下创建一个 `hugo_stats.json` 文件。该文件包含发布站点中每个HTML元素的 `class` 属性、`id` 属性和标签的数组。在从您的站点中[删除未使用的CSS](https://gohugo.io/hugo-pipes/postprocess/#css-purging-with-postcss)时，可以使用此文件作为数据源。这个过程也称为剪枝（pruning）、清理（purging）或摇树（tree shaking）。

通过 `disableClasses`、`disableIDs` 和 `disableTags` 键从 `hugo_stats.json` 中排除 `class` 属性、`id` 属性或标签。

> 在v0.115.0及更早版本中，通过将 `writeStats` 设置为 `true` 来启用此功能。虽然仍然可用，但 `writeStats` 键将在将来的版本中被弃用。
>
> 由于CSS清理通常仅限于生产构建，因此将 `buildStats` 对象放在 [config/production](https://gohugo.io/getting-started/configuration/#configuration-directory) 之后。
>
> 为了提高速度，解析发布站点时可能会出现“误报”（例如，不是HTML元素的HTML元素）。这些“误报”很少且无关紧要。
>
> 由于部分服务器构建的性质，新的HTML实体在服务器运行时添加，但旧值在您重新启动服务器或运行常规 `hugo` 构建之前不会被删除。

### cachebusters

参见[配置缓存破坏](https://gohugo.io/getting-started/configuration/#configure-cache-busters)

### noJSConfigInAssets

关闭将 `jsconfig.json` 写入 `/assets` 文件夹的功能，并映射从运行的 [js.Build](https://gohugo.io/hugo-pipes/js) 导入的内容。此文件旨在帮助代码编辑器（如[VS Code](https://code.visualstudio.com/)）中的智能感知/导航。请注意，如果您不使用 `js.Build`，则不会写入任何文件。

### useResourceCacheWhen

何时使用 `/resources/_gen` 中缓存的资源进行 PostCSS 和 ToCSS。有效值为 `never`、`always` 和 `fallback`。最后一个值意味着如果 PostCSS/扩展版本不可用，则尝试使用缓存。

## 配置缓存破坏

[自 v0.112.0 版新增](https://github.com/gohugoio/hugo/releases/tag/v0.112.0)

添加了 `build.cachebusters` 配置选项，以支持使用 Tailwind 3.x 的 JIT 编译器进行开发，其中 `build` 配置可能如下所示：

`hugo.yaml`:

```yaml
build:
  buildStats:
    enable: true
  cachebusters:
  - source: assets/watching/hugo_stats\.json
    target: styles\.css
  - source: (postcss|tailwind)\.config\.js
    target: css
  - source: assets/.*\.(js|ts|jsx|tsx)
    target: js
  - source: assets/.*\.(.*)$
    target: $1
```

上面的示例中的一些关键点是 `writeStats = true`，它会在每次构建时将一个 `hugo_stats.json` 文件与渲染输出中使用的 HTML 类等写入。对此文件的更改将触发 `styles.css` 文件的重建。您还需要将 `hugo_stats.json` 添加到 Hugo 的服务器监视器中。有关运行示例，请参见 [Hugo Starter Tailwind Basic](https://github.com/bep/hugo-starter-tailwind-basic)。

### source

一个匹配与 Hugo 中的一个虚拟组件目录（通常是 `assets/...`）相关联的文件的正则表达式。

### target

一个匹配资源缓存中应在 `source` 更改时过期的键的正则表达式。您可以在表达式中使用来自 `source` 的匹配正则表达式组，例如 `$1`。

## 配置服务器

这仅在运行 `hugo server` 时相关，它允许在开发过程中设置 HTTP 标头，以便您可以测试内容安全策略等。配置格式与 [Netlify](https://docs.netlify.com/routing/headers/#syntax-for-the-netlify-configuration-file) 相匹配，但带有稍微更强大的 [Glob 匹配](https://github.com/gobwas/glob)：

`hugo.yaml`:

```yaml
server:
  headers:
  - for: /**
    values:
      Content-Security-Policy: script-src localhost:1313
      Referrer-Policy: strict-origin-when-cross-origin
      X-Content-Type-Options: nosniff
      X-Frame-Options: DENY
      X-XSS-Protection: 1; mode=block
```

由于这仅适用于“开发”，因此将其放在 `development` 环境下可能是有意义的：

`config/development/server.yaml`:

```yaml
headers:
- for: /**
  values:
    Content-Security-Policy: script-src localhost:1313
    Referrer-Policy: strict-origin-when-cross-origin
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    X-XSS-Protection: 1; mode=block
```

您还可以为服务器指定简单的重定向规则。语法与 Netlify 类似。

请注意，`status` 为 200 的情况会触发 [URL 重写](https://docs.netlify.com/routing/redirects/rewrites-proxies/)，这在 SPA（单页面应用程序）情况下是您想要的，例如：

`config/development/server.yaml`:

```yaml
redirects:
- force: false
  from: /myspa/**
  status: 200
  to: /myspa/
```

将 `force=true` 设置为 `true` 将使重定向即使路径中存在现有内容也会生效。请注意，在 Hugo 0.76 之前，`force` 是默认行为，但与 Netlify 的做法保持一致。

## 404 服务器错误页面 

[自 v0.103.0 版新增](https://github.com/gohugoio/hugo/releases/tag/v0.103.0)

默认情况下，当运行 `hugo server` 时，Hugo 将使用 `404.html` 模板渲染所有 404 错误。请注意，如果您已经将一个或多个重定向添加到[服务器配置](https://gohugo.io/getting-started/configuration/#configure-server)，您需要显式添加 404 重定向，例如：

`config/development/server.yaml`:

```yaml
redirects:
- from: /**
  status: 404
  to: /404.html
```

## 配置标题大小写 

将 `titleCaseStyle` 设置为指定 [title](https://gohugo.io/functions/title/) 模板函数和 Hugo 中自动生成的章节标题使用的标题样式。

可以是以下之一：

- `ap`（默认），[美联社 (AP) 样式指南](https://www.apstylebook.com/)中的大写规则。
- `chicago`，[芝加哥手册](https://www.chicagomanualofstyle.org/home.html)中的规则。
- `go`，Go 语言中的每个单词都大写。
- `firstupper`，将第一个单词的首字母大写。
- `none`，不使用大写。

## 配置环境变量

### HUGO_NUMWORKERMULTIPLIER

可以设置此变量以增加或减少在 Hugo 中并行处理中使用的工作程序数量。如果未设置，将使用逻辑 CPU 的数量。

## 配置查找顺序

类似于模板的[查找顺序](https://gohugo.io/templates/lookup-order/)，Hugo 在网站源代码目录的根目录中有一组默认规则来搜索配置文件作为默认行为：

1. `./hugo.toml`
2. `./hugo.yaml`
3. `./hugo.json`

在配置文件中，您可以指示 Hugo 如何渲染您的网站，控制网站的菜单，并任意定义特定于项目的全局参数。

## 示例配置

以下是配置文件的典型示例。嵌套在 `params:` 下的值将填充 [`.Site.Params`](https://gohugo.io/variables/site/) 变量，供[模板](https://gohugo.io/templates/)使用：

`hugo.yaml`:

```yaml
baseURL: https://yoursite.example.com/
params:
  AuthorName: Jon Doe
  GitHubUser: spf13
  ListOfFoo:
  - foo1
  - foo2
  SidebarRecentLimit: 5
  Subtitle: Hugo is Absurdly Fast!
permalinks:
  posts: /:year/:month/:title/
title: My Hugo Site
```

## 使用环境变量进行配置

除了上述3个配置选项外，还可以通过操作系统环境变量定义配置键值对。

例如，以下命令将在类 Unix 系统上有效地设置网站的标题：

```txt
$ env HUGO_TITLE="Some Title" hugo
```

如果您使用类似 Netlify 的服务部署站点，这非常有用。可以查看 Hugo 文档的 [Netlify 配置文件](https://github.com/gohugoio/hugoDocs/blob/master/netlify.toml) 获取示例。

> 名称必须以 `HUGO_` 为前缀，并且在设置操作系统环境变量时，配置键必须使用大写。
>
> 要设置配置参数，请将名称以 `HUGO_PARAMS_` 为前缀

如果使用的是蛇形命名法的变量名，则上述方法将不起作用。Hugo通过 `HUGO` 后的第一个字符来确定分隔符的使用方式。这允许您使用任何允许的[分隔符](https://stackoverflow.com/questions/2821043/allowed-characters-in-linux-environment-variable-names#:~:text=So%20names%20may%20contain%20any%2C%20printable%20characters%2C%20but%20may%20not%20begin%20with%20a%20digit.)在环境变量中定义变量，例如 `HUGOxPARAMSxAPI_KEY=abcdefgh`。

## Ignore content and data files when rendering 

**Note:** This works, but we recommend you use the newer and more powerful [includeFiles and excludeFiles](https://gohugo.io/hugo-modules/configuration/#module-configuration-mounts) mount options.

To exclude specific files from the `content`, `data`, and `i18n` directories when rendering your site, set `ignoreFiles` to one or more regular expressions to match against the absolute file path.

To ignore files ending with `.foo` or `.boo`:

`hugo.yaml`:

```yaml
ignoreFiles:
- \.foo$
- \.boo$
```

To ignore a file using the absolute file path:

`hugo.yaml`:

```yaml
ignoreFiles:
- ^/home/user/project/content/test\.md$
```

## 配置前置元数据

### 配置日期

日期在 Hugo 中非常重要，您可以通过向 `hugo.toml` 添加 `frontmatter` 部分来配置 Hugo 如何为您的内容页面分配日期。

默认配置如下：

`hugo.yaml`:

```yaml
frontmatter:
  date:
  - date
  - publishDate
  - lastmod
  expiryDate:
  - expiryDate
  lastmod:
  - :git
  - lastmod
  - date
  - publishDate
  publishDate:
  - publishDate
  - date
```

例如，如果您的某些内容中有非标准的日期参数，您可以覆盖 `date` 的设置：

`hugo.yaml`:

```yaml
frontmatter:
  date:
  - myDate
  - :default
```

其中 `:default` 是默认设置的快捷方式。上述配置将将 `.Date` 设置为 `myDate` 中的日期值（如果存在），如果不存在，则会查找 `date`、`publishDate`、`lastmod`，并选择第一个有效日期。

在右侧的列表中，以“:”开头的值是具有特殊含义的日期处理程序（见下文）。其他值只是您前置元数据配置中的日期参数的名称（不区分大小写）。还请注意，Hugo 对上述内容有一些内置别名：`lastmod` => `modified`，`publishDate` => `pubdate`，`published` 和 `expiryDate` => `unpublishdate`。例如，使用 `pubDate` 作为前置元数据中的日期，在默认情况下，将分配给 `.PublishDate`。

特殊的日期处理程序有：

`:fileModTime`

从内容文件的最后修改时间戳中提取日期。

例如：

`hugo.yaml`:

```yaml
frontmatter:
  lastmod:
  - lastmod
  - :fileModTime
  - :default
```

上面的配置首先尝试从 `lastmod` 前置元数据参数中提取 `.Lastmod` 的值，然后从内容文件的修改时间戳中提取日期。最后，`:default` 在这里可能不需要，但 Hugo 最终将在 `:git`、`date` 和 `publishDate` 中查找有效日期。

`:filename`

从内容文件的文件名中提取日期。例如，`2018-02-22-mypage.md` 将提取日期 `2018-02-22`。另外，如果未设置 `slug`，将使用 `mypage` 作为 `.Slug` 的值。

例如：

`hugo.yaml`:

```yaml
frontmatter:
  date:
  - :filename
  - :default
```

上面的配置首先尝试从文件名中提取 `.Date` 的值，然后将查找前置元数据参数 `date`、`publishDate` 和最后是 `lastmod`。

`:git`

这是该内容文件的最后一次修订的 Git 作者日期。只有在设置了 `--enableGitInfo` 或在站点配置中设置 `enableGitInfo = true` 时才会设置这个值。

## 配置额外的输出格式

Hugo v0.20 引入了将内容渲染为多种输出格式（例如 JSON、AMP html 或 CSV）的能力。有关如何将这些值添加到 Hugo 项目的配置文件的信息，请参阅[输出格式](https://gohugo.io/templates/output-formats/)。

## 配置压缩

默认配置如下：

`hugo.yaml`:

```yaml
minify:
  disableCSS: false
  disableHTML: false
  disableJS: false
  disableJSON: false
  disableSVG: false
  disableXML: false
  minifyOutput: false
  tdewolff:
    css:
      keepCSS2: true
      precision: 0
    html:
      keepComments: false
      keepConditionalComments: true
      keepDefaultAttrVals: true
      keepDocumentTags: true
      keepEndTags: true
      keepQuotes: false
      keepWhitespace: false
    js:
      keepVarNames: false
      precision: 0
      version: 2022
    json:
      keepNumbers: false
      precision: 0
    svg:
      keepComments: false
      precision: 0
    xml:
      keepWhitespace: false
```

## 配置文件缓存

自 Hugo 0.52 版本开始，您可以配置更多内容，而不仅仅是 `cacheDir`。以下是默认配置：

`hugo.yaml`:

```yaml
caches:
  assets:
    dir: :resourceDir/_gen
    maxAge: -1
  getcsv:
    dir: :cacheDir/:project
    maxAge: -1
  getjson:
    dir: :cacheDir/:project
    maxAge: -1
  getresource:
    dir: :cacheDir/:project
    maxAge: -1
  images:
    dir: :resourceDir/_gen
    maxAge: -1
  modules:
    dir: :cacheDir/modules
    maxAge: -1
```

您可以在自己的 `hugo.yaml` 中覆盖任何这些缓存设置。

### 关键词解释

`:cacheDir`

请参阅[配置 cacheDir](https://gohugo.io/getting-started/configuration/#configure-cachedir)。

`:project`

当前 Hugo 项目的基本目录名称。这意味着，默认情况下，每个项目都有单独的文件缓存，这意味着当您运行 `hugo --gc` 时，不会触及其他运行在同一台计算机上的 Hugo 项目相关的文件。

`:resourceDir`

这是 `resourceDir` 配置选项的值。

**maxAge**

这是在缓存条目被驱逐之前的持续时间，-1 表示永久保存，0 有效地关闭了特定的缓存。使用 Go 的 `time.Duration`，所以有效的值是 `"10s"`（10 秒），`"10m"`（10 分钟）和 `"10h"`（10 小时）。

**dir**

缓存文件将存储在其中的绝对路径。允许的起始占位符是 `:cacheDir` 和 `:resourceDir`（参见上文）。

## 配置格式规范

- [TOML 规范](https://github.com/toml-lang/toml)
- [YAML 规范](https://yaml.org/spec/)
- [JSON 规范](https://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf)

## 配置 cacheDir

这是 Hugo 默认存储文件缓存的目录。详见 [配置文件缓存](https://gohugo.io/getting-started/configuration/#configure-file-caches)。

可以使用 `cacheDir` 配置选项或通过操作系统环境变量 `HUGO_CACHEDIR` 来设置它。

如果未设置该选项，Hugo 将按以下优先顺序使用：

1. 如果在 Netlify 上运行：`/opt/build/cache/hugo_cache/`。这意味着如果您在 Netlify 上运行构建，所有配置为 `:cacheDir` 的缓存将在下次构建时保存和恢复。对于其他 CI 供应商，请阅读其文档。有关 CircleCI 的示例，请参见 [此配置](https://github.com/bep/hugo-sass-test/blob/6c3960a8f4b90e8938228688bc49bdcdd6b2d99e/.circleci/config.yml)。
2. 在操作系统用户缓存目录下的 `hugo_cache` 目录，由 Go 的 [os.UserCacheDir](https://pkg.go.dev/os#UserCacheDir) 定义。在 Unix 系统上，这是 `$XDG_CACHE_HOME`，由 [basedir-spec-latest](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) 指定，如果非空，则为 `$HOME/.cache`。在 macOS 上，这是 `$HOME/Library/Caches`。在 Windows 上，这是 `%LocalAppData%`。在 Plan 9 上，这是 `$home/lib/cache`。[Hugo v0.116.0 中新增](https://github.com/gohugoio/hugo/releases/tag/v0.116.0)
3. 在操作系统临时目录下的 `hugo_cache_$USER` 目录。

如果想知道当前的 `cacheDir` 值，可以运行 `hugo config`，例如：`hugo config | grep cachedir`。

## 另请参阅

- [数据模板](https://gohugo.io/templates/data-templates/)
- [Front matter（前置元数据）](https://gohugo.io/content-management/front-matter/)
- [配置标记](https://gohugo.io/getting-started/configuration-markup/)
- [在 GitLab Pages 上托管](https://gohugo.io/hosting-and-deployment/hosting-on-gitlab/)
- [Hugo 和《通用数据保护条例》](https://gohugo.io/about/hugo-and-gdpr/)
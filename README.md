### 总体关系（从“输入”到“页面”）


- **内容文件**：`_pages/*.md`、`_posts/*.md`、`_news/*.md`、`_projects/*.md`、`_bibliography/papers.bib`、`_data/*.yml`  
  ↓（根据 front matter 中的 `layout`、`permalink` 等）
- **布局文件**：`_layouts/*.liquid`（如 `default.liquid`、`about.liquid`、`page.liquid`、`post.liquid`、`cv.liquid`、`bib.liquid`）  
  ↓（在布局内部通过 `{% include ... %}`）
- **片段文件**：`_includes/*.liquid`（头部、导航、页脚、新闻列表、出版物搜索、项目卡片、简历区块等）  
  ↓（片段内部通过 `site.*` 访问）
- **配置 & 数据 & 插件**：`_config.yml`、`_data/*.yml`、`_plugins/*.rb`、`assets/*`  
  ↓
- **Jekyll 构建**：`bundle exec jekyll build`（由 `bin/cibuild`、Docker、GitHub Pages 调用）  
  ↓
- **生成站点**：`_site/` 下最终 HTML/CSS/JS

---

### 1. `_config.yml` ↔ 其他所有部分

- **和布局 / 页面关系**
  - `title / first_name / last_name / description / footer_text / icon`：在 `_includes/head.liquid`、`_includes/header.liquid`、`_includes/footer.liquid` 中通过 `site.xxx` 使用。
  - `url / baseurl`：在 `robots.txt`、`sitemap`、Open Graph 等 meta 标签中使用。
- **和集合（Collections）关系**
  - `collections.news` ↔ `_news/*.md`：定义 `news` 是一个集合，`output: true` + `permalink: /news/:path/`。
  - `collections.projects` ↔ `_projects/*.md`：定义项目集合，在项目列表 include（`projects.liquid` 等）中通过 `site.projects` 使用。
  - `announcements` / `latest_posts`：控制 `_includes/news.liquid`、`_includes/latest_posts.liquid` 的显示数量与滚动方式。
- **和插件 / 数据关系**
  - `plugins` 列表 ↔ `_plugins/*.rb` + 模板中的标签：
    - `jekyll-scholar` ↔ `{% bibliography %}`、`{% cite %}`，使用 `_bibliography/papers.bib` 和 `_layouts/bib.liquid`。
    - `jekyll-get-json` ↔ `jekyll_get_json` 段落，加载 `assets/json/resume.json` 到 `site.data.resume`，供 CV 使用。
    - 其他插件（`jekyll-feed`、`jekyll-sitemap`、`jekyll-minifier`、`jekyll-terser` 等）在构建时生效，不直接出现在模板里。
- **和 Sass / CSS 关系**
  - `sass.style: compressed`：控制 Sass 编译输出的 CSS 是否压缩。
  - `max_width`：在 `assets/css/main.scss` 中通过 `@use "variables" with ( $max-content-width: {{ site.max_width }} )` 传入 `_sass/_variables.scss`，控制页面最大宽度。
- **和第三方库关系**
  - `third_party_libraries`：在 `_includes/scripts.liquid` / `_includes/head.liquid` 中，通过 `site.third_party_libraries.xxx` 拼接 CDN 链接和 `integrity`，加载 Chart.js、ECharts、Mermaid、MathJax 等。

---

### 2. 页面文件 `_pages/*.md` ↔ 布局 `_layouts/*.liquid` ↔ 片段 `_includes/*.liquid`

- **`_pages/about.md`（首页）**
  - front matter：`layout: about`, `permalink: /`
  - **关联布局**：`_layouts/about.liquid`
    - 内部通常会：
      - 继承 `default.liquid` 或直接结构化 HTML。
      - 使用 `page.profile`、`page.selected_papers`、`page.announcements` 等 front matter 字段控制显示。
      - include：
        - `header.liquid`（导航）
        - `footer.liquid`（页脚）
        - `selected_papers.liquid`（精选论文，依赖 jekyll-scholar 和 `_bibliography/papers.bib`）
        - `news.liquid`（公告，依赖 `site.news` 集合，即 `_news/*.md`）
        - `latest_posts.liquid`（如启用，依赖 `_posts/*.md`）
    - 进一步依赖 `_config.yml`（例如 `site.announcements` 配置）。
- **`_pages/publications.md`**
  - front matter：`layout: page`, `permalink: /publications/`
  - **关联布局**：`_layouts/page.liquid` → 再由 `page.liquid` 使用 `default.liquid` 作为基础。
  - **内部内容**：
    - `{% include bib_search.liquid %}` ↔ `_includes/bib_search.liquid`
      - bib_search 片段会渲染一个搜索框，利用 jekyll-scholar 的索引。
    - `{% bibliography %}` ↔ `jekyll-scholar` 插件
      - 读取 `_bibliography/papers.bib`；
      - 使用 `_layouts/bib.liquid` 渲染每条文献；
      - `bib.liquid` 中再使用 `_data/venues.yml`、`_data/coauthors.yml` 和 `_config.yml` 中的 `scholar` 配置决定样式（期刊颜色、作者高亮、合作者链接等）。
- **`_pages/cv.md`**
  - front matter：`layout: cv`, `permalink: /cv/`, `cv_pdf`, `cv_format` 等。
  - **关联布局**：`_layouts/cv.liquid`
    - cv 布局会：
      - include `_includes/cv/*.liquid`（`education.liquid`、`experience.liquid`、`skills.liquid` 等），每个片段代表简历的一个 section。
      - 使用：
        - `page.cv_pdf`（通常来自 `_pages/cv.md` front matter 或 `_data/socials.yml` 中的 `cv_pdf`）来生成“下载 PDF”按钮。
        - 若使用 `rendercv` / `jsonresume`，则通过 `site.data.resume`（由 `jekyll_get_json` 从 `assets/json/resume.json` 加载）填充各 section 内容。
- **`_pages/news.md`**
  - front matter：`layout: page`, `permalink: /news/`
  - **内容**：`{% include news.liquid %}` → `_includes/news.liquid`
    - `news.liquid` 遍历 `site.news`（collections.news），展示 `_news/*.md` 的简要内容，受 `_config.yml` 中 `announcements` 或其他参数控制。
- **`_pages/404.md`**
  - 使用 `layout: page` 或其他专门布局，最后还是通过 `default.liquid` 和 `head/header/footer/scripts` 渲染。
- **所有页面的共同关系：`_layouts/default.liquid`**
  - 绝大多数布局（`about.liquid`、`page.liquid`、`post.liquid`、`cv.liquid` 等）都会以 `default.liquid` 为基础：
    - `<head>` 部分：`{% include head.liquid %}`，用到 `_config.yml`（title、语言、第三方库等）。
    - `<body>` 顶部：`{% include header.liquid %}`，用到导航信息（page 上的 `nav: true`、`nav_order` 等）。
    - 主内容区域：`{{ content }}` 或带 TOC 的二维布局（读 `page.toc.sidebar`）。
    - 底部：`{% include footer.liquid %}`、`{% include scripts.liquid %}`。

---

### 3. 集合 `_news/`、`_posts/`、`_projects/` ↔ 布局 / 片段

- **`_news/*.md`**
  - front matter 多为 `layout: post`, `inline: true` 等。
  - **布局**：`_layouts/post.liquid`
  - **被引用**： 在 `news.liquid` 中通过 `site.news` 列出；在 about 页通过 `announcements` 区块（若 `about.md` 开启）。
- **`_posts/*.md`**
  - 示例博客文章，布局也是 `post.liquid`。
  - **被引用**：
    - `latest_posts.liquid`（若在首页或别的页面 include）；
    - `related_posts.liquid`（文章页底部的“相关文章”）；
    - 归档页（`archive.liquid` 和 jekyll-archives 插件）会按年/标签/分类列出这些 post。
- **`_projects/*.md`**
  - front matter：`layout: page`，`category`、`img`、`importance` 等。
  - **被引用**：
    - 若存在 `projects` 页面（例如 `_pages/projects.md`），会 include `projects.liquid` 或 `projects_horizontal.liquid`，遍历 `site.projects` 集合生成项目卡片。
    - about 页或其他页面也可以通过这些 include 展示项目。

---

### 4. `_data/*.yml` ↔ 布局 / 片段

- **`_data/socials.yml`**
  - 提供：`cv_pdf`、`email`、`scholar_userid`、`rss_icon`、自定义 social 链接等。
  - **被引用**：
    - 导航栏 / about 页底部的社交图标（通过 jekyll-socials 插件和 `_includes/header.liquid`、`_includes/footer.liquid`）。
    - 简历页下载按钮如果没有在 `cv.md` 中专门设置，也会回落到这里的 `cv_pdf`。
- **`_data/venues.yml`**
  - 映射期刊/会议缩写到 URL + 颜色（如 `"AJP"`, `"PhysRev"` 等）。
  - **被引用**：`_layouts/bib.liquid` 中：通过 `site.data.venues[entry.abbr]` 查找当前文献的 venue；决定按钮颜色、链接到期刊官网。
- **`_data/coauthors.yml`**
  - 为每个合作者（按姓的 key）提供 `firstname` 和 `url`。
  - **被引用**：`_layouts/bib.liquid` 中：通过 `site.data.coauthors[clean_last_name]` 查找作者是否在 coauthors 表里；若有，给该作者名字加上超链接。
- **`_data/repositories.yml`**
  - 包含 `github_users` 和 `github_repos`。
  - **被引用**：
    - `_includes/repository/repo.liquid` 中，用于生成 GitHub 仓库列表。
    - 原本由 `_pages/repositories.md` 页面 include，但你已经把该页面删除，所以这部分目前是“孤立”的。

---

### 5. `_bibliography/papers.bib` ↔ `jekyll-scholar` ↔ `bib.liquid`

- **`_bibliography/papers.bib`**
  - 存放所有 BibTeX 条目（论文、书、文章等），前两行 `---` 只是让 Jekyll 处理。
- **`_config.yml` 中的 `scholar` 段**
  - 指定：
    - `source: /_bibliography/`
    - `bibliography: papers.bib`
    - `bibliography_template: bib`（即 `_layouts/bib.liquid`）
    - 作者名、样式、分组方式、过滤字段等。
- **`_layouts/bib.liquid`**
  - 渲染单条文献：
    - 用 `entry.abbr` 与 `_data/venues.yml` 取得期刊信息；
    - 用 `entry.author_array` 与 `_data/coauthors.yml` + `_config.yml.scholar` 确定哪些作者是自己、哪些有链接；
    - 根据 `_config.yml` 中的 `enable_publication_badges` 决定是否展示 Altmetric、Dimensions、Google Scholar、InspireHEP 徽章。
- **调用链**
  - `publications.md` 中 `{% bibliography %}`  
    → jekyll-scholar 读取 `papers.bib`  
    → 使用 `_layouts/bib.liquid` 渲染每条  
    → 再通过 `sites.data.*` 和 `site.scholar` 加装额外信息。

---

### 6. `_plugins/*.rb` ↔ 模板中的自定义标签 / Filter

- **`google-scholar-citations.rb`**
  - 定义 Liquid 标签，例如 `{% google_scholar_citations scholar_id article_id %}`（实际名称可在文件里看到）。
  - 被 `_includes/citation.liquid` 或出版物相关模板使用，用来从 Google Scholar 拉取某篇文章的引用数，并缓存结果。
- **`inspirehep-citations.rb`**
  - 类似地定义一个 tag，用于从 InspireHEP 获取引用数，供物理/HEP 领域使用。
- **`external-posts.rb`**
  - 配合 `_config.yml` 中的 `external_sources` 段（如 Medium、Google Blog），把外部 RSS/文章作为“外部帖子”加入 `site.posts` 或专门列表。
- **`remove-accents.rb`**
  - 定义 `remove_accents` Filter（你在 `bib.liquid` 里已经看到：`| remove_accents`），用于统一处理作者姓氏 key。
- **`details.rb` / `file-exists.rb` / `hide-custom-bibtex.rb`**
  - 分别扩展 `<details>` 片段、提供文件存在性判断、隐藏 BibTeX 中不想渲染的字段等。
  - 在各自的 include 或 layout 中以 tag/filter 的形式被调用。

---

### 7. 样式与脚本：`assets/css/main.scss` ↔ `_sass/*` ↔ `head/scripts.liquid`

- **Sass 主入口：`assets/css/main.scss`**
  - `@use "variables" with ( $max-content-width: {{ site.max_width | default: "930px" }} );`
    - 将 `_config.yml` 中的 `max_width` 传给 `_sass/_variables.scss`，影响布局宽度。
  - `@use "themes"; @use "layout"; @use "typography"; @use "navbar"; ...`
    - 这些 `@use` 对应 `_sass/_themes.scss`、`_sass/_layout.scss`、`_sass/_typography.scss` 等文件，分别定义颜色、布局、排版、导航、页脚、博客、简历等部分的样式。
- **CSS 引入：`_includes/head.liquid`**
  - 在 `<head>` 中 `<link>` 引用编译后的 `assets/css/main.css`（由 main.scss 生成）。
  - 同时引用第三方 CSS（Bootstrap、MDB、Highlight.js、Lightbox2 等）——这些 URL 来自 `_config.yml.third_party_libraries`。
- **脚本引入：`_includes/scripts.liquid`**
  - 引入主题 JS 文件（搜索、图表、画廊、滚动进度条、暗黑模式切换等）。
  - 引入第三方 JS（Chart.js、D3、ECharts、Plotly、Mermaid、MathJax 等），同样根据 `_config.yml.third_party_libraries` 的 URL 和 version 生成 `<script>` 标签。

---

### 8. 构建 / 部署脚本 ↔ Jekyll / Docker

- **`Gemfile`**
  - 列出了所有 Ruby 依赖（jekyll、jekyll-scholar、各类 jekyll 插件等），`bundle install` 时会被安装。
- **`bin/cibuild`**
  - 内容是 `bundle exec jekyll build`，CI 或本地脚本直接调用它来构建 `_site/`。
- **`bin/deploy`**
  - 读取源分支（如 `main`）和部署分支（默认 `gh-pages`），在本地运行构建，然后把 `_site` 推送到指定分支。
- **`Dockerfile` / `.devcontainer/devcontainer.json`**
  - Dockerfile 构建一个包含 Ruby、Jekyll、Python nbconvert 等的镜像。
  - devcontainer.json 指定使用这个 Dockerfile 作为 VS Code/Cursor 的开发容器，attach 后通过 `./bin/entry_point.sh` 自动运行 `jekyll serve`。

---

```shell
docker compose pull
docker compose up
```

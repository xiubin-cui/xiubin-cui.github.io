这是一个基于 **Jekyll** 的静态个人学术网站/作品集项目，使用了流行的 [al-folio](https://github.com/alshedivat/al-folio) 主题。该项目主要用于展示个人学术简历、发表论文、新闻动态、博客文章和教学经历等，非常适合研究人员或开发者使用。

**主要特点**：

- **响应式设计**：基于 Bootstrap，适配移动端和桌面端。
- **学术专注**：内置论文引用（BibTeX）、Google Scholar 集成、数学公式支持（MathJax）。
- **高度可配置**：通过 `_config.yml` 和 `_data` 文件夹管理内容，无需深入修改代码。
- **自动化部署**：包含完善的 GitHub Actions 工作流（CI/CD）。

### 2. 技术栈

- **核心框架**：[Jekyll](https://jekyllrb.com/) (Ruby) - 静态站点生成器。
- **前端技术**：
  - HTML/Liquid 模板语言
  - Sass/SCSS (CSS 预处理器)
  - JavaScript (ES6+, 包含多个可视化库如 Chart.js, ECharts, Mermaid)
  - Bootstrap & MDB (UI 框架)
- **辅助工具**：
  - **Python**：用于处理引用更新 (`update_scholar_citations.py`) 和 CV 渲染 (`rendercv`)。
  - **Docker**：用于容器化开发和部署环境。
- **CI/CD**：GitHub Actions (自动化测试、构建和部署)。

### 3. 项目结构

```text
xiubin-cui.github.io/
├── _config.yml             # 核心配置文件（站点标题、作者信息、插件开关等）
├── Gemfile                 # Ruby 依赖定义（Jekyll 插件）
├── requirements.txt        # Python 依赖定义（辅助脚本）
├── docker-compose.yml      # Docker 编排文件
├── .github/workflows/      # CI/CD 自动化流程配置
│
├── _layouts/               # 页面布局模板 (如 default.liquid, post.liquid, bib.liquid)
├── _includes/              # 可复用的页面片段 (如 header.liquid, footer.liquid)
├── _pages/                 # 独立页面内容 (about.md, cv.md, publications.md)
├── _news/                  # 新闻动态内容
├── _bibliography/          # 论文引用数据 (papers.bib)
├── _data/                  # 数据文件 (socials.yml, venues.yml, coauthors.yml) - 数据驱动内容
├── _plugins/               # 自定义 Jekyll 插件 (Ruby 脚本)
├── _sass/                  # 样式源文件 (SCSS)
├── assets/                 # 静态资源 (CSS, JS, 图片, 字体, PDF)
│
└── bin/                    # 实用脚本 (构建 cibuild, 部署 deploy, Docker 入口 entry_point.sh)
```

### 4. 环境要求与依赖

#### 编程语言依赖

- **Ruby**：必须安装 Ruby 开发环境（推荐通过 rbenv 或 rvm 管理）。
  - 依赖包：见 `Gemfile` (如 `jekyll`, `jekyll-scholar`, `jekyll-sitemap` 等)。
- **Python**：用于特定的数据处理任务。
  - 依赖包：见 `requirements.txt` (如 `nbconvert`, `rendercv`, `scholarly`)。
- **Node.js** (可选)：虽然主要依赖 Ruby，但部分前端工具或插件可能依赖 Node 环境。

#### 系统依赖

- **ImageMagick**：如果使用 `jekyll-imagemagick` 插件处理图片，通常需要系统安装此工具。

### 5. 配置文件

- **`_config.yml`**：最重要的文件。控制几乎所有全局设置，包括：
  - 网站元数据（标题、描述、URL）。
  - 作者信息（姓名、邮箱、社交链接）。
  - 主题设置（颜色模式、代码高亮样式）。
  - 插件配置（Scholar 设置、统计分析 ID）。
- **`_data/*.yml`**：
  - `socials.yml`：配置页面底部的社交图标链接。
  - `venues.yml`：配置期刊/会议的显示样式。
  - `coauthors.yml`：配置合作者的链接和信息。

### 6. 当前状态说明

项目结构完整，是一个标准的 Jekyll 站点。已经配置了 Docker 开发环境，便于快速上手。包含完整的 GitHub Actions 配置，支持推送到 `main` 分支时自动部署。 主要内容通过 Markdown 和 YAML/BibTeX 文件维护，内容与样式分离做得很好。

### 7. 运行方式

#### 方案 A：Docker 运行（推荐，最简单）

无需本地安装 Ruby/Python 环境，直接使用 Docker 容器。

```bash
# 拉取镜像并启动服务
docker compose pull
docker compose up
```

启动后访问：`http://localhost:8080`

#### 方案 B：本地运行

需要先安装 Ruby 和 Bundler。

```bash
# 1. 安装依赖
bundle install

# 2. 启动本地服务器
bundle exec jekyll serve
```

启动后访问：`http://127.0.0.1:4000`

### 8. 主要功能模块

1.  **首页 (About)**：由 `_pages/about.md` 控制，展示个人简介、精选论文 (`selected_papers`) 和最新新闻。
2.  **简历 (CV)**：由 `_pages/cv.md` 控制，支持在线预览和 PDF 下载（可自动生成）。
3.  **出版物 (Publications)**：由 `_pages/publications.md` 和 `jekyll-scholar` 插件驱动，自动从 `_bibliography/papers.bib` 读取 BibTeX 格式的论文并渲染列表。
4.  **新闻 (News)**：在 `_news/` 目录下添加 Markdown 文件即可发布动态。
5.  **博客 (Blog)**：支持在 `_posts/` (目前根目录未见此文件夹，可手动创建) 发布技术文章，支持 Jupyter Notebook 转换。

### 9. 注意事项

- **内容修改**：大部分修改不需要动代码，只需修改 `_config.yml`、`_pages/` 下的 Markdown 文件或 `_data/` 下的 YAML 文件。
- **论文更新**：添加新论文只需更新 `_bibliography/papers.bib` 文件。
- **样式调整**：如果需要修改样式，请优先在 `assets/css/main.scss` 中覆盖，或者修改 `_sass/` 下的变量，避免直接修改编译后的 CSS。
- **部署**：推送到 GitHub 后，GitHub Actions (`.github/workflows/deploy.yml`) 会自动构建并发布到 GitHub Pages。

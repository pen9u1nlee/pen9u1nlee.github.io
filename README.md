# PenguinLee's Blog

在这里写点字打发时间。

站点地址：https://pen9u1nlee.github.io

## 内容目录

- `_posts/`：已发布文章，文件名使用 `YYYY-MM-DD-title.md`。
- `_manuscripts/`：写作草稿。
- `img/`：文章图片。
- `about.md`：关于页。
- `_config.yml`：站点配置。
- `_data/navigation.yml`：导航。

## 本地预览与构建

安装 Ruby 和 Bundler 后，在仓库目录执行：

```sh
bundle install
bundle exec jekyll serve
```

预览地址为 `http://localhost:4000`。修改 `_config.yml` 后需要重启预览服务。

生成静态站点：

```sh
bundle exec jekyll build
```

构建结果位于 `_site/`。如需使用 npm 命令，先执行 `npm install`；
`npm run serve` 用于预览，`npm run build` 用于生产构建。

## 发布

站点使用 GitHub Pages。实际发布分支和构建方式以仓库 Settings → Pages 的配置为准。
本仓库不再包含原主题文档站的 Travis 发布流程。

## 主题与许可

基于 Jekyll 和 TeXt 主题，保留本地主题运行代码及原始 `LICENSE`。
`Gemfile` 通过 `jekyll-text-theme.gemspec` 声明依赖，两者需要一并保留。
文章的许可设置见 `_config.yml`。

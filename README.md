
# leonharetd 的个人博客仓库

这是一个基于 Hexo 框架搭建的静态博客网站，托管在 GitHub Pages 服务上。包含以下核心功能：

### 🚀 主要特性
- 使用 Hexo 静态网站生成器构建
- 集成 GitHub Actions 自动化部署
- 支持 Markdown 文章写作
- 包含主题配置和自定义样式
- 自动化构建和持续集成

### 📂 目录结构说明
```
.
├── source/         # 博客文章和资源文件
│   └── _posts/     # 所有博客文章（Markdown格式）
├── themes/         # Hexo主题配置
├── scaffolds/      # 文章模板
├── _config.yml     # 全局配置文件
└── .github/        # GitHub Actions 工作流配置
```

### 🛠️ 本地开发
1. 安装依赖：
```bash
npm install
```

2. 启动本地服务器：
```bash
hexo server
```

3. 生成静态文件：
```bash
hexo generate
```

### 🌐 部署流程
- 推送代码到 `main` 分支时自动触发 GitHub Actions
- 构建后的静态文件将自动部署到 GitHub Pages
- 访问地址：https://leonharetd.github.io

### 📝 写作规范
- 所有文章使用 Markdown 格式
- 文章存放在 `source/_posts` 目录
- 使用 YAML front-matter 添加文章元数据

欢迎通过 issue 提交建议或参与项目贡献！

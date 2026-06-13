# XID Blog

[![Deploy to GitHub Pages](https://github.com/yyaf/xid-blog/actions/workflows/deploy.yml/badge.svg)](https://github.com/yyaf/xid-blog/actions/workflows/deploy.yml)

欢迎来到我的个人博客，这里记录我在前端、运维学习之路上的点滴探索。

内容涵盖前端开发、Python 脚本、GitHub 工作流、Linux 等项目实践。

🌐 **https://xid.pp.ua**

## 技术栈

- **[Astro](https://astro.build/)** — 静态站点生成器
- **[astro-theme-cactus](https://github.com/chrismwilliams/astro-theme-cactus)** — 博客主题
- **[Tailwind CSS](https://tailwindcss.com/)** — CSS 框架
- **[Pagefind](https://pagefind.app/)** — 静态搜索
- **[Expressive Code](https://expressive-code.com/)** — 代码高亮
- **GitHub Pages** — 托管与部署

## 本地开发

```bash
# 安装依赖（需要 pnpm）
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 构建搜索索引
pnpm postbuild

# 预览生产构建
pnpm preview
```

## 项目结构

```
src/
├── components/     # Astro 组件
├── content/
│   ├── post/       # 博客文章 (.md/.mdx)
│   ├── note/       # 简短笔记
│   └── tag/        # 标签页面
├── layouts/        # 页面布局
├── pages/          # 路由页面
├── plugins/        # Remark/Rehype 插件
├── styles/         # 全局样式
└── utils/          # 工具函数
```

## 协议

[MIT](LICENSE)

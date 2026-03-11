# Twitter 资讯日报 - 实施方案

## 当前状态
- ✅ 静态网站已部署到 Vercel: https://twitter-daily-news.vercel.app/
- ✅ GitHub 仓库: ransomzhang/twitter-daily-news
- ⏳ 每日自动抓取流程未完全自动化

## 部署架构
```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│  定时 Cron      │────▶│  抓取+生成    │────▶│  GitHub        │
│  (每天 7:10)   │     │  (手动/半自动) │     │  (代码托管)    │
└─────────────────┘     └──────────────┘     └────────┬────────┘
                                                     │
                      ┌──────────────────────────────┘
                      ▼
              ┌───────────────┐
              │   Vercel     │
              │  (自动部署)   │
              └───────────────┘
```

## 文件结构
```
~/.agents/skills/daily-news/
├── daily-workflow.sh          # 主流程脚本（未完全工作）
├── settings.yaml               # 配置文件
├── twitter-settings.yaml       # Twitter 账号列表
├── output/
│   ├── twitter-raw.json       # 原始推文数据
│   └── 2026-03-11.html       # 生成的日报
├── website/                   # 网站源码
│   ├── dist/                 # 构建输出
│   │   ├── index.html
│   │   └── 2026-03-11.html
│   └── vercel.json           # Vercel 配置
└── scripts/
    ├── build_website.py       # 网站构建
    ├── generate_report.py     # 报告生成
    └── push_github.py        # GitHub 推送
```

## 手动执行流程（当前使用）

### 1. 抓取推文
```bash
# 用浏览器打开 Twitter 列表，手动复制推文到文件
~/.agents/skills/daily-news/output/twitter-raw.json
```

### 2. 翻译 + 生成 HTML
```python
# 手动编辑 html 内容，翻译成中文
# 保存到 ~/.agents/skills/daily-news/output/2026-XX-XX.html
```

### 3. 构建 + 推送
```bash
cd ~/.agents/skills/daily-news
source .venv/bin/activate
python scripts/build_website.py
cd website
git add -A && git commit -m "Update" && git push origin main
```

## 待解决问题

### 1. ZHIPUAI_API_KEY 配置
需要在 ~/.zshrc 中添加：
```bash
export ZHIPUAI_API_KEY="your-key"
```

### 2. Browser MCP 自动抓取
- 当前脚本依赖 Claude Browser MCP
- 需要配置 ~/.claude.json 中的 browsermcp
- 或使用 OpenClaw 内置浏览器替代

### 3. 列表是私密的
- Financial News 列表是锁定列表
- 需要浏览器登录才能访问
- 可考虑改为公开列表

## Cron 配置
```yaml
ID: daily-news-001
Name: Twitter资讯日报
Schedule: 10 7 * * * (每天 7:10)
```

## 未来优化方向

1. **完全自动化**: 配置 API Key + Browser MCP 自动抓取翻译
2. **过滤规则**: 根据关键词(gold, silver, Fed, oil等)过滤非金融内容
3. **模板优化**: 翻译+分类全部自动生成

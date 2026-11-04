# DataShare · 开放数据下载与引用链接生成

一个**纯静态**、托管在 **GitHub Pages** 上的数据分享站点。它把数据文件放在仓库里，
自动生成可直接写进论文 / 博客 / README 的下载链接与引用格式。

- 数据集列表 + 搜索 + 标签筛选
- 每个数据集自动生成：raw 原始链接、jsDelivr CDN 链接、Release 附件链接、页面永久链接
- 一键复制：BibTeX、APA 7、GB/T 7714、Markdown、数据可用性声明（DAS）
- 附 Python `pandas.read_csv(url)` 与 `curl` 用法片段
- 无后端、无数据库，改一个 JS 文件即可新增数据集

## 目录结构

```
.
├── index.html                 # 单页应用：首页 / 详情 / 说明
├── assets/
│   ├── css/style.css
│   └── js/
│       ├── config.js          # 站点配置（owner / repo / ref / 许可）
│       └── app.js             # 渲染、路由、链接与引用生成
├── data/
│   ├── datasets.js            # ★ 数据集清单，新增数据只需改这里
│   └── files/                 # ★ 实际数据文件放这里
├── scripts/publish.ps1        # 一键创建仓库并发布到 GitHub Pages
├── .github/workflows/pages.yml# Pages 自动部署
└── .nojekyll                  # 让 Pages 直接发布静态文件
```

## 快速开始（本地预览）

```powershell
python -m http.server 8080
# 浏览器打开 http://127.0.0.1:8080
```

## 发布到你的 GitHub 账号

### 方式 A：一键脚本（推荐）

先确保本机有 `git`；装过 GitHub CLI（`gh`）会更省事：

```powershell
winget install --id Git.Git -e          # 安装 git（如未安装）
winget install --id GitHub.cli -e       # 可选：安装 gh
```

然后在仓库根目录执行：

```powershell
pwsh -File scripts/publish.ps1 -RepoName data-share -Visibility public
```

脚本会依次完成：检查工具 → `gh auth login` → 创建远程仓库 → 提交并推送 →
把 Pages 源设为 *GitHub Actions* → 打印站点地址。

### 方式 B：手动

```powershell
git init -b main
git add -A
git commit -m "init data share site"
git remote add origin https://github.com/<你的用户名>/data-share.git
git push -u origin main
```

然后在 GitHub 仓库页面：**Settings → Pages → Build and deployment → Source** 选择
**GitHub Actions**。等 Actions 里的 *Deploy to GitHub Pages* 跑完（约 1–2 分钟），
站点地址为：

```
https://<你的用户名>.github.io/data-share/
```

> 仓库名若为 `<你的用户名>.github.io`，站点地址则是 `https://<你的用户名>.github.io/`。
> 若使用自定义域名，把域名填到 `assets/js/config.js` 的 `siteUrl`。

## 新增一个数据集

1. 把文件放进 `data/files/`（建议小于 100 MB；更大的文件请用 Release 附件或 Git LFS）。
2. 在 `data/datasets.js` 的 `window.DATASETS` 数组里加一项：

```js
{
  id: "my-dataset",                 // 唯一 id，出现在 #/dataset/my-dataset
  title: "我的数据集",
  version: "v1.0.0",
  authors: ["张三", "李四"],
  year: 2026,
  license: "CC BY 4.0",
  tags: ["文本", "基准数据"],
  summary: "一句话说明。",
  description: ["第一段说明。", "第二段说明。"],
  files: [
    { path: "data/files/my_dataset.csv", format: "CSV", size: "12 MB", note: "字段说明" }
  ],
  keywords: ["关键词1", "关键词2"],
  doi: ""                            // 有 Zenodo DOI 时填写
}
```

3. 提交推送，Pages 会自动重新部署；详情页里的链接全部按 `owner/repo/ref` 实时拼装。

## 生成的链接长什么样

假设仓库是 `alice/data-share`，分支 `main`，文件 `data/files/mydata.csv`：

| 用途 | 链接 |
| --- | --- |
| 原始文件 | `https://raw.githubusercontent.com/alice/data-share/main/data/files/mydata.csv` |
| CDN 加速 | `https://cdn.jsdelivr.net/gh/alice/data-share@main/data/files/mydata.csv` |
| 页面永久链接 | `https://alice.github.io/data-share/#/dataset/my-dataset` |
| Release 附件 | `https://github.com/alice/data-share/releases/download/v1.0.0/mydata.csv` |

论文中建议引用 **tag 版本**（如 `@v1.0.0`）而不是分支名，保证可复现。

## 在文章中引用

打开任意数据集详情页，点「引用」区块的标签页可切换并复制：

- **BibTeX** —— LaTeX / Overleaf
- **APA 7** —— 英文期刊
- **GB/T 7714—2015** —— 中文期刊与学位论文
- **Markdown** —— 博客、README、GitHub Issue
- **数据可用性声明** —— 直接放进论文的 Data Availability 小节

## 配置项（`assets/js/config.js`）

| 字段 | 说明 |
| --- | --- |
| `owner` | GitHub 用户名；留空则从页面域名自动推断 |
| `repo` | 仓库名；留空则自动推断 |
| `ref` | 默认分支或 tag，建议正式发布时用 tag |
| `siteUrl` | 站点根地址，自定义域名时填写 |
| `license` / `licenseUrl` | 默认数据许可与链接 |

页面右上角「仓库设置」可在浏览器本地临时覆盖这些值（存 `localStorage`），方便未推送前先看效果。

## 许可

站点代码 MIT；示例数据为合成数据，仅作模板演示。真实数据请自行确认再分发权利。

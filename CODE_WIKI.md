# jkjoy/github-profile — Code Wiki 文档

> 一个基于 `github-profile-summary-cards` GitHub Action 自动生成 GitHub 个人资料摘要卡片的配置项目。
>
> 维护者：jkjoy (浪子) | 邮箱：jkjoy@live.cn | 仓库：[jkjoy/github-profile](https://github.com/jkjoy/github-profile)

---

## 目录

- [1. 项目概述](#1-项目概述)
- [2. 项目架构](#2-项目架构)
- [3. 目录结构](#3-目录结构)
- [4. 核心模块说明](#4-核心模块说明)
  - [4.1 GitHub Actions 工作流](#41-github-actions-工作流)
  - [4.2 卡片输出目录](#42-卡片输出目录)
  - [4.3 主题系统](#43-主题系统)
  - [4.4 主 README](#44-主-readme)
- [5. 卡片类型详解](#5-卡片类型详解)
- [6. 主题列表](#6-主题列表)
- [7. 依赖关系](#7-依赖关系)
- [8. 项目运行方式](#8-项目运行方式)
- [9. 如何自定义](#9-如何自定义)

---

## 1. 项目概述

### 1.1 项目定位

本项目是一个**GitHub Profile 自动化配置仓库**，本身不包含自定义编写的源代码。它通过配置 [vn7n24fzkq/github-profile-summary-cards](https://github.com/vn7n24fzkq/github-profile-summary-cards) 这一第三方 GitHub Action，实现**自动生成并更新 GitHub 个人资料展示卡片**（SVG 格式图片），并将这些卡片展示在 GitHub Profile README 中。

### 1.2 核心功能

- **自动生成** — 通过 GitHub Actions 定时任务，每 24 小时自动运行
- **多主题支持** — 内置 50+ 种主题配色，可自由切换
- **五项核心指标** — 生成 5 种不同维度的个人资料卡片
- **零编码维护** — 无需手动编写代码，克隆后修改配置即可使用

### 1.3 技术栈

| 技术 | 用途 |
|------|------|
| GitHub Actions | CI/CD 自动化工作流引擎 |
| YAML | 工作流配置文件（`.yml`） |
| SVG | 卡片输出图片格式 |
| Markdown | 文档与 README 展示 |

---

## 2. 项目架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                      GitHub 托管                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              GitHub Actions 自动触发                          │   │
│  │  ┌──────────┐  触发条件:                                    │   │
│  │  │ Workflow  │  - push (代码推送)                            │   │
│  │  │ 配置文件   │  - create (分支创建)                          │   │
│  │  │  (.yml)   │  - schedule (每24小时)                       │   │
│  │  └────┬─────┘  - workflow_dispatch (手动触发)               │   │
│  │       │                                                      │   │
│  │       ▼                                                      │   │
│  │  ┌──────────────────────────────────────────┐               │   │
│  │  │ vn7n24fzkq/github-profile-summary-cards   │              │   │
│  │  │ @release Action                           │              │   │
│  │  │  (第三方 Action)                           │              │   │
│  │  └──────────────────┬───────────────────────┘               │   │
│  │                     │                                        │   │
│  │                     ▼                                        │   │
│  │  ┌──────────────────────────────────────────┐               │   │
│  │  │    profile-summary-card-output/           │              │   │
│  │  │    └── <theme-name>/                      │              │   │
│  │  │        ├── 0-profile-details.svg          │              │   │
│  │  │        ├── 1-repos-per-language.svg       │              │   │
│  │  │        ├── 2-most-commit-language.svg     │              │   │
│  │  │        ├── 3-stats.svg                    │              │   │
│  │  │        ├── 4-productive-time.svg          │              │   │
│  │  │        └── README.md                      │              │   │
│  │  └──────────────────────────────────────────┘               │   │
│  │                     │                                        │   │
│  │                     ▼                                        │   │
│  │  ┌──────────────────────────────────────────┐               │   │
│  │  │    README.md (主页面)                      │              │   │
│  │  │    通过 Markdown 图片引用嵌入 SVG 卡片     │               │   │
│  │  └──────────────────────────────────────────┘               │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. 目录结构

```text
github-profile/
├── .github/
│   └── workflows/
│       └── profile-summary-cards.yml    # GitHub Actions 工作流配置文件
│
├── profile-summary-card-output/         # 自动生成的卡片输出目录
│   ├── 2077/                            # 各主题子目录（50+ 个主题）
│   │   ├── 0-profile-details.svg        # 个人资料详情卡片
│   │   ├── 1-repos-per-language.svg     # 仓库语言分布卡片
│   │   ├── 2-most-commit-language.svg   # 提交语言统计卡片
│   │   ├── 3-stats.svg                  # 综合统计卡片
│   │   ├── 4-productive-time.svg        # 高效时段分析卡片
│   │   └── README.md                    # 该主题的引用说明
│   ├── algolia/
│   ├── apprentice/
│   ├── aura/
│   ├── ...                              # 其他所有主题...
│   └── README.md                        # 主题预览总索引
│
├── README.md                            # 项目根 README（GitHub Profile 主页）
└── CODE_WIKI.md                         # 本文档
```

---

## 4. 核心模块说明

### 4.1 GitHub Actions 工作流

**文件路径：** [`.github/workflows/profile-summary-cards.yml`](file:///e:/kmoretti-github/github-profile/.github/workflows/profile-summary-cards.yml)

这是项目的**唯一可执行配置**，定义了自动化流程的全部行为。

```yaml
name: GitHub-Profile-Summary-Cards

on:
  create:              # 分支/标签创建时触发
  push:                # 代码推送时触发
  schedule:            # 定时触发（每24小时）
    - cron: "* */24 * * *"
  workflow_dispatch:   # 支持手动触发

jobs:
  build:
    runs-on: ubuntu-latest
    name: generate-github-cards
    permissions:
      contents: write    # 需要写入仓库内容的权限
    steps:
      - uses: actions/checkout@v2
      - uses: vn7n24fzkq/github-profile-summary-cards@release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          USERNAME: ${{ github.repository_owner }}
```

**关键参数说明：**

| 参数 | 值 | 说明 |
|------|-----|------|
| `GITHUB_TOKEN` | `${{ secrets.GITHUB_TOKEN }}` | GitHub 自动提供的认证令牌，用于访问仓库数据 |
| `USERNAME` | `${{ github.repository_owner }}` | 目标 GitHub 用户名（自动从仓库所有者获取） |
| `actions/checkout@v2` | — | 检出仓库代码 |
| `vn7n24fzkq/github-profile-summary-cards@release` | — | 核心 Action，生成摘要卡片 |

> **注意：** 当前的 cron 表达式 `* */24 * * *` 的语义是"每小时的第 0 分钟"（`*` 表示每分钟都匹配，而 `*/24` 表示每 24 小时一次），实际上它每 24 小时运行一次。更标准的写法应为 `0 0 * * *`（每天 0 点运行）。

### 4.2 卡片输出目录

**文件路径：** [`profile-summary-card-output/`](file:///e:/kmoretti-github/github-profile/profile-summary-card-output)

这是工作流的**输出产物目录**。当 GitHub Action 运行时，`github-profile-summary-cards` Action 会扫描指定 GitHub 用户的仓库和提交数据，生成 SVG 卡片并写入此目录。

每个主题子目录包含 **5 个 SVG 文件**（详见第 5 节）和 **1 个 README.md**（提供 Markdown 引用代码示例）。

根目录下的 `README.md` 是所有主题的**总预览页面**，按字母顺序列出全部主题的卡片样式预览。

### 4.3 主题系统

项目内置了 **50+ 种主题配色方案**，覆盖了主流的代码编辑器主题、框架主题和自定义配色：

| 类别 | 主题示例 |
|------|----------|
| **编辑器主题** | `dracula`, `monokai`, `nord_dark`, `one_dark`, `material_palenight`, `nightowl` |
| **框架主题** | `vue`, `react`, `github`, `discord_old_blurple` |
| **配色系列** | `dark`, `default`, `transparent`, `highcontrast` |
| **创意主题** | `synthwave`, `outrun`, `cobalt2`, `tokyonight`, `gruvbox` |
| **极简主题** | `graywhite`, `calm`, `date_night` |

主题名称即为子目录名。每个主题的配色方案由上游 Action 定义，用户只需在引用卡片时指定不同的目录路径即可切换。

### 4.4 主 README

**文件路径：** [`README.md`](file:///e:/kmoretti-github/github-profile/README.md)

这是 GitHub Profile 的主页文件，**直接展示在用户的 GitHub 个人主页上**。由于仓库名称与用户名一致（`jkjoy`），GitHub 会自动将此 README 显示在 `github.com/jkjoy` 页面。

当前配置示例使用 `vue` 主题展示 5 张卡片：

```markdown
[![](https://raw.githubusercontent.com/jkjoy/github-profile/master/profile-summary-card-output/vue/0-profile-details.svg)](https://github.com/vn7n24fzkq/github-profile-summary-cards)
[![](https://raw.githubusercontent.com/jkjoy/github-profile/master/profile-summary-card-output/vue/1-repos-per-language.svg)](https://github.com/vn7n24fzkq/github-profile-summary-cards)
[![](https://raw.githubusercontent.com/jkjoy/github-profile/master/profile-summary-card-output/vue/2-most-commit-language.svg)](https://github.com/vn7n24fzkq/github-profile-summary-cards)
[![](https://raw.githubusercontent.com/jkjoy/github-profile/master/profile-summary-card-output/vue/3-stats.svg)](https://github.com/vn7n24fzkq/github-profile-summary-cards)
[![](https://raw.githubusercontent.com/jkjoy/github-profile/master/profile-summary-card-output/vue/4-productive-time.svg)](https://github.com/vn7n24fzkq/github-profile-summary-cards)
```

---

## 5. 卡片类型详解

每套主题包含以下 5 种卡片：

| 序号 | 文件名 | 卡片名称 | 展示内容 |
|------|--------|----------|----------|
| `0` | `0-profile-details.svg` | **个人资料详情** | 用户名、总贡献数、公开仓库数、加入 GitHub 时长、邮箱、年度贡献趋势图 |
| `1` | `1-repos-per-language.svg` | **仓库语言分布** | 按代码仓库统计的各编程语言使用占比 |
| `2` | `2-most-commit-language.svg` | **提交语言统计** | 按提交记录统计的各编程语言使用占比 |
| `3` | `3-stats.svg` | **综合统计** | Stars 数、PR 数、Issue 数、贡献数等综合统计 |
| `4` | `4-productive-time.svg` | **高效时段分析** | 按小时统计的提交活跃度分布，展示最活跃编码时段 |

### 示例：0-profile-details.svg 内容结构

该 SVG 卡片包含以下几个视觉区域：

```
┌─────────────────────────────────────────────────────────────┐
│  jkjoy (浪子)                                                │
│                                                             │
│  ★  2.14k Contributions on GitHub                           │
│  📦  76 Public Repos                                        │
│  🕐  Joined GitHub 9 years ago                              │
│  ✉  jkjoy@live.cn                                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              年度贡献趋势折线图                       │    │
│  │    ▁▃▅█▇▆▄▃▅▇▆▅▃▄▅▆▇█▆▄▃▂                         │    │
│  │    25/06  25/08  25/10  25/12  26/02  26/04  26/06  │    │
│  └─────────────────────────────────────────────────────┘    │
│              contributions in the last year                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. 主题列表

以下是当前项目中包含的全部 50+ 个主题：

| # | 主题名 | 风格描述 |
|---|--------|----------|
| 1 | `2077` | 赛博朋克风（霓虹粉+黄） |
| 2 | `algolia` | Algolia 搜索蓝 |
| 3 | `apprentice` | 深色学徒风格 |
| 4 | `aura` | Aura 紫色主题 |
| 5 | `aura_dark` | Aura 深色变体 |
| 6 | `ayu_mirage` | Ayu Mirage 暖灰 |
| 7 | `bear` | Bear 笔记风格 |
| 8 | `blue_green` | 蓝绿色调 |
| 9 | `blueberry` | 蓝莓紫 |
| 10 | `buefy` | Buefy UI 风格 |
| 11 | `calm` | 浅色柔和风格 |
| 12 | `chartreuse_dark` | 黄绿色深色 |
| 13 | `city_lights` | 城市之光 |
| 14 | `cobalt` | Cobalt 蓝 |
| 15 | `cobalt2` | Cobalt2 增强版 |
| 16 | `codeSTACKr` | codeSTACKr 主题 |
| 17 | `darcula` | JetBrains Darcula |
| 18 | `dark` | 经典深色 |
| 19 | `date_night` | 约会之夜 |
| 20 | `default` | 默认浅色 |
| 21 | `discord_old_blurple` | Discord 旧版紫色 |
| 22 | `dracula` | Dracula 吸血鬼 |
| 23 | `flag_india` | 印度国旗配色 |
| 24 | `github` | GitHub 浅色 |
| 25 | `github_dark` | GitHub 深色 |
| 26 | `gotham` | 哥谭暗黑 |
| 27 | `graywhite` | 灰白极简 |
| 28 | `great_gatsby` | 了不起的盖茨比 |
| 29 | `gruvbox` | Gruvbox 复古 |
| 30 | `highcontrast` | 高对比度 |
| 31 | `jolly` | 欢乐彩色 |
| 32 | `kacho_ga` | 花鸟风月和风 |
| 33 | `maroongold` | 栗色+金色 |
| 34 | `material_palenight` | Material Palenight |
| 35 | `merko` | 墨绿色 |
| 36 | `midnight_purple` | 午夜紫 |
| 37 | `moltack` | Moltack 风格 |
| 38 | `monokai` | Monokai 经典 |
| 39 | `moonlight` | 月光蓝 |
| 40 | `nightowl` | Night Owl 猫头鹰 |
| 41 | `noctis_minimus` | Noctis 极简 |
| 42 | `nord_bright` | Nord 亮色 |
| 43 | `nord_dark` | Nord 深色 |
| 44 | `ocean_dark` | 海洋深色 |
| 45 | `omni` | Omni 多彩 |
| 46 | `onedark` | Atom One Dark |
| 47 | `outrun` | Outrun 霓虹 |
| 48 | `panda` | Panda 熊猫 |
| 49 | `prussian` | 普鲁士蓝 |
| 50 | `radical` | 激进红 |
| 51 | `react` | React 蓝色 |
| 52 | `rose_pine` | Rose Pine 松树 |
| 53 | `shades_of_purple` | 紫色渐变 |
| 54 | `slateorange` | 石板橙 |
| 55 | `solarized` | Solarized 浅色 |
| 56 | `solarized_dark` | Solarized 深色 |
| 57 | `swift` | Swift 橙色 |
| 58 | `synthwave` | 合成波霓虹 |
| 59 | `tokyonight` | 东京之夜 |
| 60 | `transparent` | 透明背景 |
| 61 | `vision_friendly_dark` | 视觉友好深色 |
| 62 | `vue` | Vue 绿色 |
| 63 | `yeblu` | 黄+蓝 |
| 64 | `zenburn` | Zenburn 柔和 |

---

## 7. 依赖关系

### 7.1 外部依赖

本项目**无传统包管理依赖**（无 `package.json`、`requirements.txt` 等），唯一的运行时依赖是 GitHub Actions 生态中的第三方 Action：

| 依赖 | 版本 | 来源 | 用途 |
|------|------|------|------|
| `actions/checkout` | `@v2` | GitHub 官方 | 检出仓库代码到运行环境 |
| `vn7n24fzkq/github-profile-summary-cards` | `@release` | 第三方社区 | 核心卡片生成引擎，扫描 GitHub API 生成 SVG 卡片 |

### 7.2 上游项目

- **项目名称：** [github-profile-summary-cards](https://github.com/vn7n24fzkq/github-profile-summary-cards)
- **作者：** [vn7n24fzkq](https://github.com/vn7n24fzkq)
- **说明：** 一个 GitHub Action，通过调用 GitHub API 获取用户数据，使用 D3.js 生成美观的 SVG 摘要卡片
- **许可协议：** MIT License（请参考上游项目确认）

### 7.3 内部依赖关系图

```
github-profile (本仓库)
├── [依赖] actions/checkout@v2          → 基础检出步骤
├── [依赖] vn7n24fzkq/...@release       → 核心卡片生成
│   ├── [调用] GitHub REST API          → 获取用户/仓库/提交数据
│   ├── [处理] 数据聚合与统计            → 计算各项指标
│   └── [输出] SVG 文件                  → 写入 profile-summary-card-output/
└── [引用] README.md                     → 通过 <img> 标签嵌入 SVG
```

### 7.4 权限需求

工作流需要 `contents: write` 权限，因为 Action 需要将生成的 SVG 文件写回仓库。

---

## 8. 项目运行方式

### 8.1 前置条件

- 一个 GitHub 账号
- 仓库名称必须为 `{你的用户名}/{你的用户名}`（例如 `jkjoy/github-profile`）
- GitHub Actions 功能已启用（默认开启）

> **重要提示：** 要让 GitHub 个人主页显示此 README，仓库名称必须与你的用户名一致。

### 8.2 使用模板创建（推荐）

前往上游项目模板：`vn7n24fzkq/github-profile-summary-cards-example`，点击 **"Use this template"** 按钮即可创建自己的仓库。

### 8.3 手动 Fork/Clone

```bash
git clone https://github.com/jkjoy/github-profile.git
cd github-profile
```

### 8.4 配置工作流

编辑 `.github/workflows/profile-summary-cards.yml`：

- **`USERNAME`** — 默认使用 `${{ github.repository_owner }}`，自动从仓库所有者获取
- **`GITHUB_TOKEN`** — 默认使用内置的 `secrets.GITHUB_TOKEN`，无需额外配置
- 如果希望避免 API 限流，可以创建 [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) 并添加到仓库 Secrets 中

### 8.5 触发运行

工作流在以下情况下自动触发：

1. **代码推送** (`push`) — 提交代码后自动运行
2. **分支/标签创建** (`create`) — 创建新分支或标签时
3. **定时调度** (`schedule`) — 每 24 小时自动运行一次
4. **手动触发** — 在 GitHub Actions 页面点击 "Run workflow" 按钮

### 8.6 查看结果

运行完成后：

1. 进入 `profile-summary-card-output/` 目录查看生成的 SVG 卡片
2. 编辑 `README.md`，选择喜欢的主题并插入卡片引用代码
3. 提交更改后，你的 GitHub 个人主页将自动显示这些卡片

### 8.7 切换主题

只需修改 `README.md` 中的图片路径，将主题名替换为想要的样式：

```markdown
# 将 vue 替换为其他主题名
![](https://raw.githubusercontent.com/jkjoy/github-profile/master/profile-summary-card-output/dracula/0-profile-details.svg)
```

---

## 9. 如何自定义

### 9.1 更换主题

在 `README.md` 中将 URL 中的主题目录名替换为第 6 节列表中的任意主题名。

### 9.2 调整更新频率

修改 `.github/workflows/profile-summary-cards.yml` 中的 `cron` 表达式：

```yaml
schedule:
  - cron: "0 0 * * *"   # 每天午夜运行
  - cron: "0 */6 * * *"  # 每 6 小时运行一次
```

### 9.3 添加更多统计

如果需要更多类型的统计卡片，可以参考上游项目 README 中提到的其他兼容工具，如：
- [github-readme-stats](https://github.com/anuraghazra/github-readme-stats) — 更多样化的统计卡片
- [github-readme-activity-graph](https://github.com/Ashutosh00710/github-readme-activity-graph) — 贡献活动图

---

## 附录

### A. 贡献代码

本项目是一个配置文件仓库，不包含自定义源代码。如需改进卡片生成逻辑，请向上游项目 [vn7n24fzkq/github-profile-summary-cards](https://github.com/vn7n24fzkq/github-profile-summary-cards) 提交 Issue 或 PR。

### B. 常见问题

**Q: 生成的卡片没有数据显示？**
A: 确保 GITHUB_TOKEN 有足够的权限。如果数据量较大，可以考虑创建 Personal Access Token 替代默认令牌。

**Q: 更新 README 后主页没有变化？**
A: GitHub 个人主页的 README 缓存可能需要几分钟才能刷新。尝试清空浏览器缓存或使用无痕模式查看。

**Q: 如何只在主页显示特定主题的卡片？**
A: 从 `profile-summary-card-output/` 中选择一个主题目录，在 `README.md` 中只引用该主题目录下的图片路径即可。

---

*文档生成日期：2026-06-30*
*本文档基于 [jkjoy/github-profile](https://github.com/jkjoy/github-profile) 仓库 v1.0 版本编写*

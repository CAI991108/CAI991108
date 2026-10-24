# 主页改造规格 v1 — @CAI991108

> 状态：待确认 · 2026-10-24
> 依据：`research/github-profile-readme.md`（所有约束均带 primary source）
> 一句话定位：**CUHK-Shenzhen 的 AI 博士生，做多模态/生成式模型，用可点的代码证明自己。**

---

## 1. 定位与三个证据支柱

主页要回答三个问题，顺序固定：**我研究什么 → 我做出过什么 → 怎么找到我。**

| 支柱 | 目前有什么 | 主页怎么用 |
|---|---|---|
| 研究品味 | UNSPOKEN（ACM MM 2025，音频语言模型隐喻推理） | Selected work 第一条，带会议标签 |
| 工程深度 | JD_search_agent、InstructPix2Pix、SD3M-LoRA、GPT-2/Llama 训练 | Selected work 其余条目，一行「为什么重要」 |
| 可信度 | CUHK-SZ 博士身份、个人站、HF | Hero 一行 + 链接行 |

**重要提醒**：你选了「不放学术内容」，但 `UNSPOKEN` 是 MM25 的论文代码 —— 这是你手上最强的一条证据。建议折中：**不开 Publications / Scholar 区块，但把它放进 Selected work 并标注会议**。这是「作品」叙事，不是「学术履历」叙事。

---

## 2. 已锁定的决策

| 决策 | 结论 |
|---|---|
| 读者 | 研究与合作者优先，招聘方其次 |
| 语言 | 英文正文 + 中文 `<details>` 折叠 |
| 风格 | 学术极简外壳 + **一个**克制的仪表盘岛 |
| 隐私 | 删除手机号，改邮箱 + LinkedIn |
| 证书 | 文末 `<details>`，图片压缩后放 |
| 自动化 | 卡片（图片 URL）+ 每周统计表（Action），cron 改周更 |
| 交付 | 我在本地提交，你在自己终端 `git push` |

---

## 3. 页面结构（自上而下）

| # | 区块 | 内容 | 组件 | 高度预算 |
|---|---|---|---|---|
| 1 | Hero | `# Zijin CAI` + 一行身份 + 链接行 | 纯 Markdown + 文本链接 | ≤ 120px |
| 2 | `## Now` | 3 条：在做什么 / 在哪个组 / 找什么机会 | 文本 | ≤ 100px |
| 3 | `## Focus` | 4–6 个研究方向 chip | shields 无 label chip | ≤ 40px |
| 4 | `## Selected work` | 4–5 个项目，每个一行「做了什么 + 为什么重要」 | `<table>` + 链接 | ≤ 260px |
| 5 | `## Toolbox` | 6–8 个技术 badge（替代 15 个 skillicons） | shields flat | ≤ 40px |
| 6 | `## Coding activity` | 仪表盘岛：卡片矩阵 + 每周表 | `<details>` 包表格 | ≤ 520px |
| 7 | `## Awards` | 证书，折叠 | `<details>` + 压缩图 | 0px（收起） |
| 8 | `## Contact` | 邮箱 / LinkedIn / 站点 / HF | 文本链接 | ≤ 60px |

**中文折叠**：放在 `## Now` 之后，`<details><summary>中文简介</summary>` 一段 3 行自我介绍。

**为什么砍掉 capsule-render + typing-svg**：两者合计约 350px 的动效出现在任何信息之前。手机上 360×640 的屏幕，这就是整整一屏。这不是「不好看」，是「挡住了内容」。

---

## 4. 组件清单（精确 URL）

### 4.1 Hero 链接行（文本，不用 badge）

```
[Website](https://cai991108.github.io/) · [Email](mailto:<待填>) · [LinkedIn](<待填>) · [Hugging Face](https://huggingface.co/jimchoi) · [GitHub](https://github.com/CAI991108)
```

### 4.2 Focus chips（无 label 的两段式，读起来像 tag）

```html
<img src="https://img.shields.io/badge/Multimodal_LLMs-0969DA?style=flat&logoColor=white" alt="Multimodal LLMs" />
```

每条一个方向；`_` 代替空格，`-` 需写成 `--`。

### 4.3 仪表盘岛卡片（保留全部，随时可减）

| 卡片 | URL 关键参数 |
|---|---|
| stats 总览 | `github-readme-stats.vercel.app/api?username=CAI991108&show_icons=true&hide_border=true&theme=transparent&cache_seconds=86400` |
| streak | `streak-stats.demolab.com?user=CAI991108&hide_border=true&theme=transparent` |
| top-langs | `…/api/top-langs/?username=CAI991108&layout=donut-vertical&theme=transparent&hide_border=true` |
| WakaTime | `…/api/wakatime?username=<WakaTime 用户名，待确认>&layout=compact&theme=transparent&hide_border=true` |
| activity | `github-readme-activity-graph.vercel.app/graph?username=CAI991108&theme=github-compact&hide_border=true&area=true` |
| trophy | `github-profile-trophy.vercel.app/?username=CAI991108&no-frame=true&theme=flat&column=7` |
| views | `komarev.com/ghpvc/?username=CAI991108&color=0969DA&abbreviated=true` |

**风险，写在这里免得踩**：
- 公共 Vercel 实例被官方 README 描述为 "best-effort, can be unreliable"。**所有 `.vercel.app` 域名在本机被 sinkhole，我无法替你验证实际渲染**，只能保证 URL 语法正确。
- WakaTime 卡要的是 **WakaTime 用户名**，不是 GitHub 句柄。你现在传的是 `CAI991108`，需要浏览器里确认它是不是你的 WakaTime 用户名；不是的话这张卡现在是空的或别人的。
- 需要 WakaTime 打开两个公开开关：code time 公开 + languages/editors/os 公开。

### 4.4 工具 badge（`flat`，20px 高，统一 logoColor）

Python · PyTorch · TensorFlow · scikit-learn · R · LaTeX · Linux · Docker

注意 simple-icons **没有** `linkedin` / `matlab` / `windows` / `vue`（`vue` 现为 `vuedotjs`）——别写进 badge。

---

## 5. 设计系统

**配色（三色，按角色分配）**

| 角色 | 值 | 用在哪 |
|---|---|---|
| accent | `#0969DA` | 所有 chip、链接强调 |
| accent-2 | `#8250DF` | 只用于「Now / 开放机会」一处 |
| muted | `#6E7781` | 次要说明、分隔 |

**硬规则**
1. **绝不写 `style=`** —— GitHub 会整条剥离（已有源码级证据）。布局只能用 `<p align>`、`<table>`、`<img width/align>`。
2. 所有 badge 统一 `style=flat`、20px 高；一排不超过 6 个，最多两排。
3. 每张图片必须有 `alt`。
4. 单图 ≤ 150KB，整页图片总量 ≤ 400KB（现在是 ~693KB）。
5. 宽度：桌面可用 700–1000px，手机可用 ~330px。含文字的图不超过 400px 宽。
6. 深色/浅色：自己控制的图用 `<picture>` + `prefers-color-scheme`（GitHub 官方博客有原文片段）；第三方卡片只能靠配色在两种底色上都有对比度。
7. 至少 2 个 `##` 标题，换取 GitHub 自动生成的 Outline 目录。

---

## 6. 仓库内容盘点（20 个）

| 仓库 | 判断 | 建议一句话 |
|---|---|---|
| UNSPOKEN | **主推** | ACM MM'25 — 音频语言模型的隐喻理解 |
| JD_search_agent | **主推** | LangGraph 有状态购物助手，文本+图片 OCR 多模态检索 |
| Robotic-Action-Frame-Prediction-… | **主推** | 用 InstructPix2Pix 预测机器人未来帧 |
| SD3M-SimpleTuner-lora | **主推** | SD3 Medium 的 LoRA 微调与生成艺术 |
| Machine-Learning-and-Language-Model | 可选 | GPT-2 / Llama 预训练、LoRA、BAdam、GSM8K |
| CUHKSZ-SAI-LaTeX_Template | 收录 | 给同学的 LaTeX 模板（唯一「对别人有用」的仓库） |
| *-Utils 六个课程仓库 | 合并成一行 | "Coursework: NLP / DL / ML / CV / Optimization" |
| CAI991108.github.io | Hero 链接 | 个人站与博客 |
| mattermost | 不展示 | 巨型 fork，与定位无关 |
| damai_ticket | **建议转私有** | 抢票脚本，法律与观感风险都高 |
| omarchy-NTE / dsh-studios / AIRI-Dev | 待定 | 无 description，要么补一句，要么不置顶 |

**共性缺口**：多数仓库 description 是空的、没有 topics。主页写得再漂亮，点进去是空白，叙事就断了。这属于「主页之外的工作」，但决定主页的可信度。

---

## 7. CI 改造

| 现在 | 改成 |
|---|---|
| `.github/workflows/waka_readme`（无 `.yml`，永不执行） | 删除 |
| `wakatime.yml` 每天空提交 | cron 改每周；补 `<!--START_SECTION:waka-->` / `<!--END_SECTION:waka-->` 标记；只留需要的 `SHOW_*`；提交信息改成 `chore(stats): weekly coding stats [skip ci]` |

**关于那 827 个空提交**：`git log --name-only` 显示最近 25 条改动文件数为 0，README 最后一次真实修改是 2025-04-27。历史要不要重写（orphan 分支重开）是可选动作，代价是所有贡献记录清零 —— 建议**保留历史、只止血**。

---

## 8. 验收标准

- [ ] 全文搜索 `style=` 结果为 0
- [ ] 每张 `<img>` 都有 `alt`
- [ ] 图片总量 ≤ 400KB
- [ ] 所有 badge 高 20px、样式统一、配色一致
- [ ] 手机 360px 宽无横向滚动
- [ ] 深浅色下文字与 chip 均可读
- [ ] 所有外链 200，无自指链接（GitHub badge 指向当前页）
- [ ] 至少 2 个 `##` 标题
- [ ] CI：删掉死文件，周更跑通一次
- [ ] 手机号彻底移除

---

## 9. 待你确认 / 待补材料

1. **邮箱、LinkedIn** —— 决定 Contact 区（等待简历）
2. **UNSPOKEN 是否作为 Selected work 主推**（我的建议：是）
3. **WakaTime 用户名** —— 需要你在浏览器里确认
4. **`damai_ticket` 是否转私有**
5. 中文折叠段落的措辞

---

## 10. 实施顺序

| 票 | 内容 | 依赖 |
|---|---|---|
| T1 | 删死 workflow、改 cron、加标记 | 无 |
| T2 | Hero + Now + 中文折叠 | 材料 |
| T3 | Selected work + Toolbox | 无 |
| T4 | 仪表盘岛 + 卡片参数 | WakaTime 用户名 |
| T5 | 证书压缩 + `<details>` | 无 |
| T6 | 验收清单逐项过 | T1–T5 |

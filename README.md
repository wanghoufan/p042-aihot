# AIHOT 热点日报站

> 把信源和精选标准换成你自己的，它每天筛一遍，出一份属于你行业的热点日报。

[English](README.en.md)

<p><img src="docs/assets/board-light.png" width="900" alt="AIHOT 本地示范站：模型榜页面"></p>

<p><img src="docs/assets/cluster-light.png" width="900" alt="事件归组与热度：多篇报道聚成一个事件，右侧是当前热点榜"></p>

<sub>模型榜为本地示范站的真实界面截图；第二张左半为归组示意、右半为实际热点榜。站名默认 MyHOT。</sub>

## 它解决什么问题

一个行业每天都有新消息，但真正值得看的只有几条。人工盯几十个信源不现实，直接转发又会被营销稿和重复报道淹没。

AIHOT 把这件事拆成可配置的流程：从信源采集，先判断是不是本行业的事，再用同一份评分标准独立打两次分，过门槛才入选；然后写中文标题和摘要，把不同来源讲的同一件事聚成一个事件，按独立来源数量算热度，定时出日报、周报和月报。

线上站点是 [aihot.news](https://aihot.news)，服务 AI 行业。这个仓库是它的完整框架，包含全部提示词和入选门槛；换掉 `industry/` 里的行业配置，就是另一个行业的站。

## 核心能力

- **六种信源**：RSS、网页列表、JSON 接口、X 账号、微信公众号，以及外部脚本推送。信源可分级（官方一手 / 媒体个人），抓取频率按产出自动调整。
- **可校准的精选**：预筛之后，同一份评分标准独立打两次分，两次之和达到 2 × 门槛才入选；门槛按信源分级不同。评分提示词和门槛都在 `industry/` 里，可用自己标注的样本跑 `scripts/eval-selection.ts`，结果导入后台 SelectBench 对比。
- **写作**：中文标题、答案先行的摘要、推荐理由、标签，外文内容可翻译；对原文没提到的公司名有防护。
- **事件归组**：先用标题摘要向量找候选，再让模型判断是同一件事、后续进展还是两件事；拿不准时合并后换一家模型确认。人工改过的归属不会被覆盖。
- **热度**：按事件计算，48 小时内每个独立来源只算一次，24 小时减半；与 6 小时前比较，上涨的标记为上升，新出现的标记为“新”。
- **日报 / 周报 / 月报**：每天 08:00 出日报，每周一 10:00 出周报，每月 1 日 10:30 出月报，按分类分节。
- **给人和 Agent 的出口**：网页、RSS（精选 / 全部 / 全文 / 日报）、公开 API `/api/v1/`、MCP `/api/mcp`、`llms.txt`、站点地图和分享图，共用同一套公开读取层。
- **后台**：信源管理与试抓、内容诊断、精选评测、每一步单独换模型、付费服务预算熔断、运行记录与告警。
- **可关掉的 AI 专属模块**：模型榜与 Codex 重置监控，在 `industry/features.ts` 里设为 `false` 即可关闭入口与定时任务。

## 快速开始

需要 Docker（含 Compose）、Node.js 24.11 或更高版本（仅用于生成 `.env`），以及一个 OpenAI 兼容的模型 API Key。

```bash
node scripts/init-env.ts --llm-key <你的模型 API Key>
docker compose up -d --build
```

`init-env.ts` 会生成 `.env`，填好随机密钥与管理员密码，并把密码打印一次（也写在 `.env` 的 `ADMIN_PASSWORD` 里）。机器上没有 Node 时，把 `.env.example` 复制成 `.env`，自行填写 `ADMIN_PASSWORD`（至少 12 位）、`SESSION_SECRET`、`IMG_PROXY_SIGN_SECRET`、`POSTGRES_PASSWORD` 和 `LLM_API_KEY`。

打开 <http://localhost:3000>，后台在 `/admin`。第一次启动会导入示范信源，一两分钟后开始有内容；首次导入的一百多条资料大约半小时处理完。

`docker compose` 会起五个容器：`db`（PostgreSQL 17）、`setup`（迁移与种子数据后退出）、`api`、`worker`、`web`。部署到服务器前，先把 `.env` 里的 `SITE_URL` 改成读者实际访问的地址，否则 RSS、站点地图和分享链接里仍是 localhost。

改成你的行业：主要改 [`industry/`](industry/) 里的站点文案、分类、示范信源、提示词和门槛，步骤见 [把它改成你的行业](docs/customize.md)。改完可运行：

```bash
npm run typecheck
npm run test
node scripts/smoke.ts
```

## 数据与隐私

- 内容、事件、日报保存在 PostgreSQL 17；上传文件与本地数据保存在 `AIHOT_DATA_DIR`（Docker 中为 `/data`）。
- 站点不要求读者注册。读者的收藏与已读状态存在浏览器本地，后台只允许管理员访问。
- 每条精选都链接原文。站内是否显示全文由信源的 `site_fulltext` 决定，默认只显示摘要。
- `.env` 含管理员密码、数据库密码和各家 API Key，已被 `.gitignore` 排除；`.data/` 同样不进 Git。
- 付费调用（模型、X、公众号、Jina）先记回执再取结果，重试时复用已付费的结果；每个服务可设置每分钟、每小时、每天预算上限，超限暂停。

## 已知限制

- 这是 `aihot.news` 线上代码的一份快照，不是打磨过的通用框架；上游更新不保证逐次同步。
- 仓库只带公开的海外 AI 资讯示范信源，不含 AIHOT 生产环境的信源名单和运营数据。
- X 信源需要 SocialData 的 key，公众号信源需要极致了（Dajiala）的 key，网页渲染可选用 Jina，均按次计费；模型也需要自己的 OpenAI 兼容 Key。
- 只提供简体中文文档与界面；仓库内未提供英文站点界面。
- 中国大陆服务器需自行完成 ICP 备案，并把备案号填入 `industry/site.ts`；抓不到海外信源时可在 `.env` 设置 `EGRESS_PROXY_URL`。
- 模型榜与 Codex 重置监控默认开启，非 AI 行业应在 `industry/features.ts` 关闭。

## 许可

代码使用 [MIT 许可证](LICENSE)。AIHOT 的名称与 Logo 不在许可范围内；字体、模型厂商和评测来源的标志各有自己的许可与商标归属，见 [NOTICE](NOTICE)。

# GEO 本地复审报告：Yak Project

**复审日期：** 2026-08-13
**复审地址：** `http://localhost:3000`
**预览方式：** `yarn serve --port 3000`（Docusaurus 生产构建静态预览）
**对比基线：** `GEO-AUDIT-REPORT.md`，原综合得分 52/100

> 此报告审计的是本地最新生产构建，而不是 `yarn start` 开发服务器。开发服务器会把静态文件请求回退到应用 HTML，不能用于判断 `robots.txt`、`sitemap.xml` 或安全响应头的线上表现。

## 结论

**本地复审综合 GEO 得分：67/100（Fair）**，相比基线 **+15 分**。

本次改动已经解决审计中的三个 Critical 缺口：AI 爬虫准入、`llms.txt` 和全站 Organization/WebSite 实体 Schema。128 篇 API 文档也已不再输出 `library-http}` 这一类无意义 description；英文首页现在实际渲染英文内容。站点已经从“内容可见但不易被稳定识别和引用”提升到“具备明确 AI 入口、实体身份和可抓取摘要”的水平。

仍未达到 Good 的主要原因是：生产层安全头、缓存、压缩与 HTTP/2/3 尚无法在本地静态预览中验证；英文页面 metadata 尚未本地化；博客作者与摘要体系仍然薄弱；品牌外部权威信号不是本仓库改动可直接提升的内容。

| 类别 | 基线 | 本地复审 | 权重 | 加权分 |
|---|---:|---:|---:|---:|
| AI 可引用性 | 55 | **70** | 25% | 17.50 |
| 品牌权威信号 | 55 | **55** | 20% | 11.00 |
| 内容 E-E-A-T | 60 | **68** | 20% | 13.60 |
| 技术地基 | 45 | **65** | 15% | 9.75 |
| 结构化数据 | 40 | **84** | 10% | 8.40 |
| 平台优化 | 45 | **65** | 10% | 6.50 |
| **综合得分** | **52** | **67** | | **66.75** |

### 评分边界

- 品牌权威维持 55 分：本地审计无法重新验证 GitHub、Wikipedia、Reddit、媒体等站外实体提及，故不把站内改动误计为站外权威提升。
- 技术地基的安全、缓存、压缩得分以本地静态服务器的真实响应为准；它不代表正式 CDN/nginx 的最终状态。部署后应重新审计线上 URL。
- 平台优化评分来自可抓取英文首页、`llms.txt`、SSR 与 Schema 的改进；未把尚未完成的英文文档/博客翻译计入。

## 已验证的改善

### 1. AI 爬虫发现与准入：已修复

- `GET /robots.txt` 返回 `200 text/plain`，同时明确放行 `GPTBot`、`ClaudeBot`、`PerplexityBot`、`Google-Extended`，并声明 `https://yaklang.com/sitemap.xml`。
- `GET /llms.txt` 返回 `200 text/plain`，含 Yaklang、Yakit、IRify、Memfit AI 的定义、核心入口、规范来源和引用指引。
- `GET /sitemap.xml` 返回 `200 application/xml`，XML 有效，包含 **727** 个 URL。

这消除了旧报告中“请求返回首页 fallback、无法发现爬虫协议文件”的 Critical 问题。

### 2. 实体与产品结构化数据：显著提升

首页静态 HTML 直接包含可解析 JSON-LD：

- `Organization`：统一名称为 `Yak Project`，`alternateName` 含 `Yaklang`、`Yakit`，并关联官网、Logo、GitHub Organization 与描述。
- `WebSite`：通过 `publisher` 指回 Organization，声明 `zh-CN` 与 `en`。
- 下载页新增 `SoftwareApplication`：描述 Yakit、操作系统及安全软件分类。
- 开源生态页仍保留 21 项 `SoftwareSourceCode` 的 `ItemList`，是样本页中最完整的页面级结构化数据。

首页、下载页、文档页均为初始 SSR HTML 直接输出 Schema 和 metadata，不依赖浏览器执行 JavaScript。

### 3. 分享与页面主题信号：已提升

全局 HTML 已包含 `og:site_name`、`og:image`、Twitter card 与 canonical/hreflang。重点页面标题已更具主题性：

- 首页：`Yak Project | 开源网络安全基础设施`
- 下载页：`下载资源：Yakit 白皮书、API 文档与离线包 | Yak Project`
- IRify：`IRify：SSA 驱动的静态代码安全分析 | Yak Project`
- 团队页：`关于 Yak Project：开源网络安全基础设施团队 | Yak Project`

### 4. API 文档摘要：128/128 通过

构建后检查了 `build/docs/api/` 下所有 **128** 个 API 页面：

- `meta[name="description"]` 与 `og:description` 均只有一份；
- 无页面包含 `library-<name>}` 形式的旧垃圾 description；
- 例如 `/docs/api/http` 已输出对 HTTP 客户端、安全测绘和相邻库边界的真实摘要。

这直接改善 AI 摘要、搜索结果片段和 API 页的主题识别。

### 5. 英文首页正文：已修复

`/en/` 实际返回 `lang="en"` 的 SSR HTML，H1 为：

> Widely Used Open-Source Cybersecurity Infrastructure

旧报告中“英文 URL 实际渲染中文正文”的核心问题在首页已被修复。

## 当前问题与优先级

### High：生产响应头、缓存与压缩尚未交付

本地生产预览的首页仅返回 `Content-Type`，未观察到以下头：

- `Strict-Transport-Security`
- `Content-Security-Policy`
- `X-Content-Type-Options`
- `X-Frame-Options` 或 `frame-ancestors`
- `Referrer-Policy`
- `Permissions-Policy`
- `Cache-Control`
- `Content-Encoding`

本地静态服务器不是正式 nginx/CDN，因此这不是源码回归结论；但它意味着这些策略尚未随仓库中的站点配置交付。生产 nginx/CDN 应补全安全头、Brotli/gzip、版本化静态资源长缓存和 HTTP/2/3，并上线后实测。

### High：英文 metadata 与英文正文仍不一致

英文首页已经是英文正文，但 `description` 与 `og:description` 仍为中文。英文页面应使用对应英文的 title、description、Open Graph 与 Twitter 文案；之后再优先翻译 `/en/docs/intro` 及 5 篇旗舰博客。

### High：博客的作者与摘要信号不足

- 198 篇博客没有显式 frontmatter description；当前摘要通常来自首段，近期文章容易只呈现“问题开场”，而不是结论与成果。
- 作者统一为 `Yak Project` 团队，虽然有 GitHub URL 和头像，但没有真人作者页、资历、专长或 `Person` Schema。

建议先为高价值博客补充 1–2 句结论型 description，并为核心作者建立资料页与可核验的公开身份链接。

### Medium：文档页面的引用友好结构可增强

- `/docs/intro` 的 description 仍是“我们要解决什么问题？”，不含 Yaklang 实体、能力或目标读者。
- 首页内容丰富，但缺少一段紧凑的 TL;DR 与问题式答案块。
- API 页面 `BreadcrumbList` 目前仅有当前页；应补全首页 → 文档 → API → 库名层级，并为文档添加 `TechArticle` 或 `WebPage` 语义。

### Medium：Schema 属性完整度

- 首页可增加 `WebPage`；若站内搜索 URL 经过验证，可给 `WebSite` 添加 `SearchAction`。
- Yakit `SoftwareApplication` 可在事实可核验后补齐 `@id`、功能列表、版本、截图、下载 URL 和免费 Offer；不要虚构版本、评分、价格或评价。
- 全站应统一补 `og:type`、`twitter:title`、`twitter:description`。目前开源生态页已经具备较完整的页面级分享字段，可作为模板。
- 使用专用 1200×630 Open Graph 图，替代通用方形 Logo，可提高分享卡片呈现质量。

### Medium：技术性能与 sitemap 信息

- 本地首页 HTML 约 347 KB，主要 JS 约 1.41 MB（未压缩观察值）；应测量并拆分非关键前端包。
- sitemap 目前没有 `<lastmod>`；内容更新流程可生成准确更新时间。
- 暂未获得 CrUX/Lighthouse 现场数据，不能对 Core Web Vitals 下强结论；正式部署后需运行 PageSpeed Insights 与真实用户监控。

## 复审证据摘要

| 项目 | 本地结果 |
|---|---|
| `/` | 200，347,062 B，SSR HTML |
| `/en/` | 200，349,335 B，英文 H1 已渲染 |
| `/robots.txt` | 200，200 B，`text/plain` |
| `/llms.txt` | 200，2,137 B，`text/plain` |
| `/sitemap.xml` | 200，93,042 B，`application/xml`，727 URL |
| `/download` | 200，含 `SoftwareApplication` JSON-LD |
| `/opensource` | 200，含 ItemList 与 SoftwareSourceCode Schema |
| `/docs/api/http` | 200，真实 description/og:description |
| API 页面回归检查 | 128 页，0 个垃圾或重复 description |

## 下一轮建议

1. 先将 nginx/CDN 的安全头、Brotli/gzip、缓存与 HTTP/2/3 部署到测试环境；针对测试域名复跑 GEO 技术审计。
2. 为 `/en/`、`/en/docs/intro` 和旗舰英文内容补齐英文 metadata 与翻译，确保 hreflang 的语义真实一致。
3. 批量生成并人工审核博客 description；为作者建立可验证的 Person 页面和 Schema。
4. 将首页 TL;DR、核心问答和 docs/API 的完整面包屑作为第二阶段的引用优化。
5. 生产部署后，对 `https://yaklang.com` 再执行一次完整 GEO 审计；届时可验证部署层得分和外部品牌信号。

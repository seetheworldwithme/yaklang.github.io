# GEO Content and Metadata Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use test-driven-development and verification-before-completion while implementing this plan.

**Goal:** Improve locally controllable GEO metadata, structured data, English consistency, blog citability, and editorial trust signals without requiring access to the yaklang.com deployment.

**Architecture:** Extend the existing post-build GEO plugin so generated static HTML receives consistent social metadata, blog abstracts, technical-document JSON-LD, and complete breadcrumbs. Keep truthful, high-value editorial information in source frontmatter and pages, while excluding untranslated English content from indexing instead of presenting Chinese pages as English.

**Tech Stack:** Docusaurus 3.9, React 19, Node.js post-build processing, Markdown frontmatter, JSON-LD.

---

### Task 1: Add regression coverage for metadata helpers

**Files:**
- Modify: `scripts/verify-geo-foundations.js`
- Modify: `plugins/geo-metadata-plugin.js`

1. Add failing assertions for generic Markdown summaries, social metadata completion, JSON-LD enhancement, and untranslated English noindex behavior.
2. Run `yarn test:geo` and verify the new assertions fail for missing APIs.
3. Implement the smallest reusable HTML/Markdown helpers.
4. Run `yarn test:geo` and verify all assertions pass.

### Task 2: Localize custom-page metadata

**Files:**
- Modify: `src/pages/index.js`
- Modify: `src/pages/download.tsx`
- Modify: `src/pages/team.js`
- Modify: `src/pages/irify.js`
- Modify: `src/locales/en.json`
- Modify: `src/locales/zh.json`

1. Add failing source assertions for locale-backed titles and descriptions.
2. Make page metadata use the SSR-selected react-i18next locale.
3. Add `og:type`, Twitter title, and Twitter description through shared page heads or post-build completion.

### Task 3: Enhance document and blog output

**Files:**
- Modify: `plugins/geo-metadata-plugin.js`
- Modify: `docusaurus.config.js`
- Modify: `docs/intro.md`

1. Generate a meaningful fallback description from Markdown paragraphs.
2. Add `TechArticle` JSON-LD and a full hierarchical `BreadcrumbList` to document pages.
3. Add or synchronize blog description, Open Graph, Twitter, and BlogPosting publisher data.
4. Enable sitemap `lastmod`.
5. Exclude untranslated `/en/docs`, `/en/products`, `/en/Yaklab`, and `/en/blog` routes from sitemap and add `noindex` to their HTML.

### Task 4: Add editorial trust signals and flagship abstracts

**Files:**
- Modify: `blog/authors.yml`
- Modify: ten newest `blog/*.md` files
- Create: `src/pages/editorial-policy.tsx`
- Modify: `docusaurus.config.js`

1. Add answer-led descriptions to the ten newest posts based only on their existing content.
2. Describe the existing Yak Project team author as an organization/team without inventing individuals or credentials.
3. Add an editorial policy covering sourcing, reproducibility, updates, corrections, and security boundaries.
4. Link the policy from the footer.

### Task 5: Verify generated output

1. Run `yarn test:geo`.
2. Run `yarn typecheck`.
3. Run `yarn build`.
4. Inspect representative generated pages for English metadata, blog summaries, TechArticle/Breadcrumb JSON-LD, noindex rules, and sitemap lastmod.
5. Review `git diff --check` and confirm unrelated audit files remain untouched.

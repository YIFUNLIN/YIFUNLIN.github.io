# 個人網站維護指南

這份文件整理之後修改網站內容時的固定流程。此專案是 Hugo 網站，請優先修改來源檔，再用 Hugo 產生靜態頁面。

## 一、專案結構

常用檔案位置：

| 目的 | 檔案或資料夾 |
| --- | --- |
| About 主要內容 | `content/english/about/_index.md` |
| About 左側 Experience 區塊 | `themes/liva-hugo/layouts/about/list.html` |
| About / 全站自訂樣式 | `themes/liva-hugo/assets/scss/templates/_main.scss` |
| Blog 文章 | `content/english/blog/*.md` |
| Blog 列表設定 | `content/english/blog/_index.md` |
| Contact 內容 | `content/english/contact/_index.md` |
| 導覽列與 footer 選單 | `config/_default/menus.en.toml` |
| 網站標題、baseURL | `config/_default/hugo.toml` |
| 作者、社群連結、全站參數 | `hugo.toml` |
| 圖片來源 | `assets/images/`、`images/` |
| Hugo 暫存產出資料夾 | `public/` 或 `/private/tmp/...` |
| 根目錄靜態輸出 | `about/`、`blog/`、`categories/`、`tags/`、`scss/`、`images/` 等 |

目前這個 repo 的 GitHub Pages 是從根目錄服務，因此修改後需要 commit「來源檔」以及「根目錄靜態輸出」。`public/` 是本機暫存產物，已放進 `.gitignore`，不要 commit。

## 二、修改前先做的事

先看目前有哪些變更：

```bash
git status --short
```

如果看到很多不相關檔案，commit 時不要使用：

```bash
git add .
```

請改用指定檔案加入，避免把草稿、暫存圖片或不相關產出一起推上去。

## 三、修改 About 頁

### 1. 修改 About 主要內容

編輯：

```text
content/english/about/_index.md
```

常見可改內容：

```yaml
title: "About Me"
image: "images/myself.jpg"
description: "..."
profile_name: "Evan Lin 林羿帆"
profile_company: "LINE Taiwan"
profile_role: "Back-end Engineer Intern"
profile_interests: "喜歡打籃球、量化交易研究"
```

`---` 下方是頁面正文，可修改工作經歷表格、競賽經歷、Product、論文、社團經歷等內容。

### 2. 修改左側 Experience 清單

編輯：

```text
themes/liva-hugo/layouts/about/list.html
```

找到：

```html
<div class="about-sidebar-experience">
```

每一筆 Experience 長這樣：

```html
<li>
  <div class="direction-l">
    <div class="flag-wrapper">
      <span class="hexa"></span>
      <span class="time-wrapper"><span class="time">2025 Mar - Now </span></span>
    </div>
    <div class="desc"><h5> LINE </h5><br> Backend Engineer Intern</div>
  </div>
</li>
```

修改時間、公司名稱、職稱即可。若要新增一筆，複製整段 `<li>...</li>` 後改文字。

### 3. 修改 About 左側樣式

編輯：

```text
themes/liva-hugo/assets/scss/templates/_main.scss
```

搜尋：

```scss
.about-sidebar-experience
```

這裡控制左側 Experience 的寬度、線條、圓點、文字間距等。若只是改內容，不需要改這個檔案。

## 四、修改 Blog

### 1. 修改既有文章

文章在：

```text
content/english/blog/
```

例如：

```text
content/english/blog/line_bot.md
```

修改 front matter 或正文：

```yaml
---
title: "LINE Bot 建立 - 完整教學"
date: 2024-08-06T13:49:23+06:00
draft: false
description: "LINE Bot 建立 - 手把手完整教學"
categories:
  - "技術分享"
tags:
  - "LINE"
type: "featured"
---
```

重點：

- `title`：文章標題
- `date`：文章日期
- `draft: false`：會顯示在網站上
- `draft: true`：草稿，不正式顯示
- `description`：SEO/摘要
- `categories`：分類
- `tags`：標籤
- `image`：文章縮圖，可選

### 2. 新增 Blog 文章

在 `content/english/blog/` 新增一個 `.md` 檔，例如：

```text
content/english/blog/my-new-post.md
```

範例內容：

```markdown
---
title: "新文章標題"
date: 2026-05-23T10:00:00+08:00
draft: false
description: "文章簡短描述"
categories:
  - "技術分享"
tags:
  - "Hugo"
  - "Website"
type: "featured"
---

## 前言

這裡開始寫文章內容。
```

新增後執行 `hugo`，Hugo 會產生文章頁、分類頁、標籤頁、列表頁與 RSS。

### 3. 修改 Blog 列表頁

編輯：

```text
content/english/blog/_index.md
```

這通常是 Blog 頁面的標題、描述或列表設定。

## 五、修改圖片

圖片使用方式有兩種。

### 1. Markdown 或 HTML 直接引用圖片

例如 About 內有：

```html
<img src="/images/autoqa.png" alt="AutoQA">
```

這種圖片需要確認檔案存在：

```text
images/autoqa.png
public/images/autoqa.png
```

若只新增在 `images/`，但 `public/images/` 沒有，部署後可能看不到圖片。

### 2. front matter 使用 `image`

例如：

```yaml
image: "images/myself.jpg"
```

這通常會經過 Hugo 的圖片處理，來源圖常放在：

```text
assets/images/
```

Hugo 可能產生 `images/*_hu_*` 這類壓縮或響應式圖片。若產出的 HTML 有引用這些新圖檔，部署時也需要一起 commit。

## 六、修改選單、社群連結、網站資訊

### 1. 修改導覽列或 footer

編輯：

```text
config/_default/menus.en.toml
```

範例：

```toml
[[main]]
name = "About"
URL = "about/"
weight = 1
```

`weight` 越小越前面。

### 2. 修改網站標題或 baseURL

編輯：

```text
config/_default/hugo.toml
```

常見欄位：

```toml
baseURL = "https://yifunlin.github.io/"
title = "林羿帆-個人網站"
```

### 3. 修改作者、Email、社群連結

編輯：

```text
hugo.toml
```

搜尋：

```toml
[params]
[[params.social]]
```

## 七、本機預覽與產生靜態檔

### 1. 本機預覽

```bash
hugo server -D
```

開啟：

```text
http://localhost:1313/
```

`-D` 會包含草稿文章。

### 2. 正式產生靜態檔

```bash
hugo
```

執行後 Hugo 預設會更新：

```text
public/
resources/_gen/assets/
```

`public/` 不需要 commit。若只是快速本機檢查可以使用 `hugo`；若要準備部署，建議用下一節的乾淨輸出流程，避免 `public/` 殘留舊頁。

若有修改 CSS，Hugo 也會更新：

```text
public/scss/style.min.css
```

## 八、同步根目錄靜態輸出

這個 repo 的正式部署檔案在根目錄，例如：

```text
about/index.html
scss/style.min.css
```

建議每次要部署前都用乾淨暫存目錄產生完整靜態輸出：

```bash
DEPLOY_DIR=$(mktemp -d /private/tmp/yifan-public.XXXXXX)
hugo -d "$DEPLOY_DIR"
rsync -a "$DEPLOY_DIR"/ ./
```

同步後檢查是否還有舊的破版片段或本機網址：

```bash
rg -n '<p><h2|localhost:1313' --glob '*.html' --glob '!public/**' --glob '!themes/**' .
```

如果你刪除文章、分類或標籤，根目錄可能殘留舊靜態頁。可用下列流程找出「根目錄有，但乾淨 build 已經沒有」的 HTML：

```bash
find "$DEPLOY_DIR" -name '*.html' -print | sed "s#$DEPLOY_DIR/##" | sort > /tmp/new-html-list
find . -path './.git' -prune -o -path './public' -prune -o -path './themes' -prune -o -name '*.html' -print | sed 's#^./##' | sort > /tmp/root-html-list
comm -23 /tmp/root-html-list /tmp/new-html-list
```

確認列表都是舊產物後，再移除：

```bash
comm -23 /tmp/root-html-list /tmp/new-html-list > /tmp/stale-generated-files
git rm --pathspec-from-file=/tmp/stale-generated-files
```

## 九、Commit 建議流程

### 1. 檢查變更

```bash
git status --short
```

### 2. 只加入這次需要的檔案

About 版面修改常見要加入：

```bash
git add \
  themes/liva-hugo/layouts/about/list.html \
  themes/liva-hugo/assets/scss/templates/_main.scss \
  public/about/index.html \
  about/index.html \
  public/scss/style.min.css \
  scss/style.min.css \
  resources/_gen/assets/scss/style.scss_8eb8c3e1c83e81a3be17b8e96be1a8e3.content
```

About 內容修改常見要加入：

```bash
git add \
  content/english/about/_index.md \
  public/about/index.html \
  about/index.html
```

Blog 新增或修改常見要加入：

```bash
git add \
  content/english/blog/文章檔名.md \
  blog \
  categories \
  tags \
  index.html \
  index.json \
  index.xml \
  sitemap.xml
```

若文章有新增圖片，也要加入對應圖片：

```bash
git add images/圖片檔名.png
```

### 3. Commit

```bash
git commit -m "Update about page"
```

或：

```bash
git commit -m "Add new blog post"
```

### 4. Push

```bash
git push
```

## 十、常見問題

### 1. 網頁看起來沒變

可能原因：

- 沒有執行 `hugo`
- 沒有把乾淨 build 同步到根目錄
- 瀏覽器快取
- GitHub Pages 還沒部署完成

可先強制重新整理瀏覽器，或等幾分鐘。

### 2. 圖片不見或 404

檢查：

```bash
rg -n "圖片檔名" public about blog categories tags
```

確認 HTML 引用的圖片檔有被 commit，例如：

```text
images/圖片檔名.png
```

### 3. CSS 改了但畫面沒變

檢查是否有：

```text
public/scss/style.min.css
scss/style.min.css
```

若只改了 SCSS，記得：

```bash
DEPLOY_DIR=$(mktemp -d /private/tmp/yifan-public.XXXXXX)
hugo -d "$DEPLOY_DIR"
rsync -a "$DEPLOY_DIR"/ ./
```

### 4. Git 出現很多不相關檔案

不要直接 `git add .`。

先用：

```bash
git status --short
```

再只加入這次確定要推的檔案。

## 十一、目前 `.gitignore` 規則

目前已忽略：

```gitignore
.DS_Store
.hugo_build.lock
public/
resources/_gen/images/
```

不要把 `images/`、`about/`、`blog/`、`categories/`、`tags/`、`scss/` 直接 ignore，因為 GitHub Pages 目前會從根目錄讀這些靜態輸出檔案。

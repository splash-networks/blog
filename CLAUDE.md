# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Engineering blog for [Splash Networks](https://splashnetworks.co) (guest Wi-Fi and network automation), published at `blog.splashnetworks.co`. Posts are step-by-step write-ups of projects we've done (MikroTik, FreeRADIUS, Passpoint, captive portals, VPNs, Google Workspace auth, Aruba/UniFi APIs). Goals: share knowledge and promote the brand via SEO and AEO (answer engine optimization), so posts should be written to be found and quoted by search engines and AI answer engines.

## Tech stack

- **Jekyll 4.2** static site (Ruby 3.2 in CI), with plugins `jekyll-feed`, `jekyll-seo-tag` and `jekyll-sitemap`. Gemfile.lock is gitignored.
- Hand-rolled layouts/includes and Sass. No JS build, no Node, no test suite, no linter.
- Hosted on **GitHub Pages** via Actions (custom domain in `CNAME`).
- KaTeX is vendored in `assets/katex/` and only loaded when a page sets `mathjax: true` (or `site.mathjax`). The flag name is `mathjax` even though the library is KaTeX.

## Commands

```
bundle install                  # first-time setup
bundle exec jekyll serve        # dev server at http://localhost:4000 (add --drafts / --future as needed)
bundle exec jekyll build        # output to _site/ (what CI runs)
```

Pushing to `master` triggers `.github/workflows/notification.yml`: `jekyll build`, then two jobs that both depend only on the build: deploy to GitHub Pages, and a Discord webhook notification (secret `DISCORD_WEBHOOK_URL`). Because `notify` doesn't wait for `deploy`, the "successfully deployed" message can be sent even when the deploy fails. Every push to `master` goes live, so there is no staging step; do work on a branch (e.g. `aeo`) and merge when ready.

## Architecture

- `_layouts/default.html` is the shell (header, nav, footer). `post.html` and `page.html` wrap it. `_includes/meta.html` renders the post title/date header and is reused by the home list. `_includes/menu.html` renders both the header nav (`navigation`) and the footer links (`external`); `index.html` is the home page and `404.html` the not-found page.
- `_includes/home.html` (excerpt list, used when `show_excerpts: true`) and `_includes/archive.html` (title/date list, used by `/archive.html`) render post lists from `site.posts`. `home.html` shows `post.excerpt`, which is everything before `<!--more-->`; `excerpt_separator` is set to that in `_config.yml`.
- There is no separate head include. `default.html` holds all `<head>` output inline: `{% seo %}` (title, canonical, meta description, Open Graph/Twitter, JSON-LD), favicons, feed link, and the conditional KaTeX and comment scripts.
- `about.md` is the site's About page (`permalink: /about/`, `layout: page`) and is linked from the nav via `_config.yml` `navigation` (nav entries are matched by page file name, so renaming a nav page means updating the config). It carries the Organization JSON-LD. There is deliberately no README.md.
- Site-wide settings (title, permalink pattern `/:title/`, nav, footer social links, frame/sidebar toggles) are all in `_config.yml`. `description` is plain text for SEO meta tags; the HTML footer text is the separate `footer_text` key. `author` (Nasir Hafeez) applies to every post. `CLAUDE.md` is in `exclude` so it is not published.
- `robots.txt` is a Jekyll-processed file (front matter + Liquid for the sitemap URL). It explicitly allows all crawlers including AI ones (GPTBot, ClaudeBot, PerplexityBot, etc.), which is intentional.
- Styles: `_sass/` partials are imported by `assets/css/frame.sass` (used when `show_frame: true`, currently on) and `index.sass`. `_includes/sidebar.html` and `assets/css/sidebar.sass` are only used when `show_sidebar: true` (currently off).
- Icons come from the SVG sprite `assets/fontawesome/icons.svg` (`<use xlink:href="...#<icon>">`), so an `icon:` in `_config.yml` must exist in that sprite. `_data/font-awesome/icons.json` is not referenced by any template.
- Comments (Disqus/Isso) are wired up in `default.html`/`post.html` but disabled (keys commented out in `_config.yml`).
- `post.html` renders the body through `_includes/anchor_headings.html`, a Liquid-only include that adds a hover `#` link (`.anchor`, styled in `_sass/classes.sass`) to every `h2`–`h4`, pointing at the `id` kramdown generates from the heading text (e.g. `## FreeRADIUS Setup` → `#freeradius-setup`). Pages, the home excerpts and the feed are not affected.
- `_includes/embed.html` embeds a URL in a responsive 16:9 iframe (`{% include embed.html url="..." %}`), for videos.

## Writing posts

- File: `_posts/YYYY-MM-DD-slug.md`. Front matter: `title`, `description` (150 characters or fewer; SEO metadata only, not rendered on the page), `layout: post`, `image` (only when the post has a banner; used for `og:image`, omit otherwise, no fallback), `last_modified_at` and `tags`. Author comes from `_config.yml` and appears only in the JSON-LD, not on the page (a visible byline was tried and removed).
- `last_modified_at` starts as the filename date. Update it by hand, only when a post is substantively revised. When it differs from the post date, `_includes/meta.html` shows an "Updated" line and the JSON-LD `dateModified` changes.
- `tags` are metadata only (nothing renders them yet). Use the existing vocabulary: `freeradius`, `mikrotik`, `google-workspace`, `wifi-authentication`, `captive-portal`, `vpn`, `api`, `passpoint`, `pfsense`.
- Post structure: intro paragraph(s), a banner image (when there is one), then `<!--more-->` (this ends the home-page excerpt), then the body with `##` section headings. Older posts still use `###` or no headings at all; only the Passpoint guide follows the `##` convention so far. The CoA, captive portal API and WireGuard posts have no banner and no `image`.
- Every image needs descriptive alt text that says what the screen shows (for example "MikroTik RADIUS Incoming dialog with Accept checked"), never a generic "screenshot". Use the official product spellings: MikroTik, WireGuard, UniFi, WinBox, Wi-Fi.
- Images go in `assets/images/<folder>/` and are referenced relatively as `../assets/images/<folder>/file.png` (Markdown `![alt](...)` or `<img src="...">`). The folder is a short name, not the full post slug (e.g. `pptp-vpn`, `coa-on-mikrotik`, `wireguard-vpn`); reuse the existing folder when editing a post. Banners are usually `title.png`, and the `image` front matter uses the absolute `/assets/images/...` path. To control width, use a centered `<div style="text-align: center;"><img ... width="70%" /></div>` (recent commits are mostly image-width tweaks).
- Permalinks are `/:title/`, where `:title` is the slug from the **filename** (not the `title` front matter). Renaming a published post's file changes its URL and breaks inbound links and SEO, but editing its `title` is safe.
- Heading anchors come from the heading text, so renaming a heading in a published post changes its `#anchor` and breaks links to that section. Case and punctuation don't affect the id; words do.
- Everything above `<!--more-->` is shown on the home page as the excerpt, so don't put section headings there.

## SEO/AEO notes

Phases 1 to 3 of the SEO/AEO plan are done. Phase 1: `jekyll-seo-tag` (meta description, Open Graph/Twitter, `BlogPosting` JSON-LD), `jekyll-sitemap`, `robots.txt`, and Organization JSON-LD on the About page. Phase 2: per-post front matter that `jekyll-seo-tag` reads (see Writing posts). Phase 3: descriptive image alt text and product spellings across all posts. Still to do: `##` headings in the older posts, FAQ sections, and internal links between related posts (there are none yet; e.g. the FreeRADIUS/Google Workspace, MikroTik and captive portal posts could cross-link). `archive.html` sets `sitemap: false`. Site-wide output changes should be checked with the user first.

Known quirk, accepted: `jekyll-seo-tag` fills the JSON-LD `publisher` name from the author, so it reads "Nasir Hafeez" (with the Splash Networks logo). This is intentional; do not work around it.

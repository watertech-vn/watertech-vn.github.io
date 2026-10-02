# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Bilingual (Vietnamese default, English) Jekyll site for Viện Công nghệ Tài nguyên nước và Môi trường (iWAT), deployed via GitHub Pages at `info.watertech.vn` (see `CNAME`). It is built on the remote theme `raviriley/agency-jekyll-theme` (`remote_theme` in `_config.yml`), so the theme's layouts, JS, and SCSS partials (`assets/js/*`, `base/variables.scss`, etc.) are **not in this repo**; local files override the theme. The README is just the upstream starter template text.

## Commands

- Install: `bundle install` (run `bundle update` after changing `Gemfile`)
- Serve locally: `bundle exec jekyll serve` (restart after editing `_config.yml`; it is not auto-reloaded)
- Build: `bundle exec jekyll build` (output in `_site/`, git-ignored)

There are no tests or linters. Verification is building the site and viewing it (the flow in GitHub's [testing locally with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) guide): run `bundle exec jekyll serve`, then open `http://localhost:4000/` (Vietnamese) and `http://localhost:4000/en/` (English) and check both. Building requires network access to fetch the remote theme.

### Verifying in the Claude Code cloud sandbox

The plain commands above fail there, so check UI changes like this (all temp files in the scratchpad, nothing in the repo):

1. `Gemfile` has no `jekyll` gem (`github-pages` is commented out), so `bundle exec jekyll` reports "command not found". Use a scratch Gemfile with `jekyll ~> 4.3`, `jekyll-remote-theme`, `jekyll-feed`, `webrick`, point `BUNDLE_GEMFILE` at it, and run Jekyll via `bundle exec ruby -e 'load Gem.bin_path("jekyll","jekyll")' build ...`.
2. The proxy blocks the remote theme download (`codeload.github.com` returns 403). Instead, `git clone --depth 1 https://github.com/raviriley/agency-jekyll-theme`, copy it into a scratch dir, copy this repo over it (excluding `.git`), and build with an override config that sets `remote_theme: ""` (`--config _config.yml,override.yml --destination <scratch>/_site`).
3. Serve `_site/` (`python3 -m http.server 4000`) and screenshot with `/opt/pw-browsers/chromium-*/chrome-linux/chrome --headless --no-sandbox --virtual-time-budget=5000 --screenshot=out.png <url>`.

Sass deprecation warnings from the theme during the build are expected.

## Multi-language architecture

Language is determined per page and drives everything:

- `_config.yml` defines `locale: "vi"` (default), `languages: ["vi", "en"]`, and `defaults` that set `lang` by path: everything is `vi` except `en/` and `_portfolio/en/` which are `en`. The default language is served at `/`, others at `/<code>/`.
- `_includes/i18n.html` must be included first by anything that needs language data. It sets shared variables (`lang`, `t`, `nav`, `lang_prefix`, `page_path`, `site_title`); Jekyll shares variables across includes, so other includes (`nav`, `services`, `team`, ...) just read `t.<section>`.
- All UI text lives in `_data/sitetext.yml` and the menu in `_data/navigation.yml`, each keyed by language code with parallel `vi:` / `en:` blocks. **Any content change (services, team, timeline, clients, ...) must be made in both blocks.** Menu `section` values must match the `section` values in `sitetext.yml`.
- `_includes/lang_switch.html` appends the language toggle to the nav, linking to the same page in the other language (`page_path`), or to `page.alt_path` if set (used by `404.html`, which links home since GitHub Pages serves one 404).
- Pages come in pairs: `index.md` / `en/index.md`, `legal.md` / `en/legal.md`. Adding a page means adding both.

## Portfolio (projects)

Projects are the `portfolio` collection (`_portfolio/projectNN.md`, Vietnamese = canonical). English translations live in `_portfolio/en/` **with the same filename**. `_includes/portfolio_list.html` builds the `projects` list from the Vietnamese files and swaps in the English file matched by `slug`; a project without a translation falls back to the Vietnamese version. So a new project needs `_portfolio/projectNN.md` plus `_portfolio/en/projectNN.md`, with images in `assets/img/portfolio/`. Rendering is in `portfolio_grid.html` (cards + modals via `modals.html`).

## Styling

`assets/css/agency.scss` is a Liquid-processed SCSS entry (empty front matter) that pulls colors/fonts/images from `_data/style.yml`; change brand colors there rather than in SCSS. `_layouts/default.html` loads the theme's JS from `assets/js/` (provided by the remote theme).

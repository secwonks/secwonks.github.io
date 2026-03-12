# Security Wonks — Jekyll Site

Dark-mode collegiate Jekyll site for GitHub Pages with custom domain.

## Quick Start

```bash
bundle install
bundle exec jekyll serve
# → http://localhost:4000
```

## Structure

```
├── _config.yml          # Site configuration
├── _layouts/
│   ├── default.html     # Base layout (header/footer)
│   └── post.html        # Blog post layout
├── _posts/              # Blog posts (YYYY-MM-DD-title.md)
├── assets/css/
│   └── main.scss        # All styles
├── index.html           # Landing page
├── blog/index.html      # Blog listing page
├── about.md             # About page
└── CNAME                # Custom domain (edit this!)
```

## Deploying to GitHub Pages

### 1. Create your GitHub repository
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/USERNAME/REPO.git
git push -u origin main
```

### 2. Enable GitHub Pages
- Go to repo **Settings → Pages**
- Source: **GitHub Actions**

### 3. Custom domain
- Edit `CNAME` — replace `yourdomain.com` with your actual domain
- Add DNS records at your registrar:
  ```
  A     @    185.199.108.153
  A     @    185.199.109.153
  A     @    185.199.110.153
  A     @    185.199.111.153
  CNAME www  USERNAME.github.io
  ```
- In repo Settings → Pages → Custom domain, enter your domain
- Check "Enforce HTTPS" (available after DNS propagates)

### 4. Update `_config.yml`
```yaml
url: "https://yourdomain.com"
```

## Writing Posts

Create files in `_posts/` with the format `YYYY-MM-DD-slug.md`:

```markdown
---
layout: post
title: "Your Post Title"
description: "One-sentence synopsis shown on the landing page and blog index."
date: 2024-12-01
author: Your Name
category: Analysis
---

Your content here...
```

The `description` field is used as the excerpt on the landing page — keep it to 1–2 sentences.

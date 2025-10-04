# Psychedelic Canvas Dreams

A Jekyll-based website featuring interactive psychedelic canvas art and digital visual experiences.

## Quick Start

1. Make sure you have Ruby and Jekyll installed
2. Install dependencies:
   ```bash
   bundle install
   ```
3. Build and serve the site:
   ```bash
   bundle exec jekyll serve
   ```
4. Visit `http://localhost:4000` in your browser

## Structure

- `_layouts/` - HTML templates for different page types
- `_posts/` - Blog posts in Markdown format
- `assets/css/` - Stylesheets
- `_config.yml` - Site configuration

## Features

- Responsive design with psychedelic gradient backgrounds
- Interactive canvas art posts
- Modern CSS with backdrop filters and animations
- Mobile-friendly navigation
- Tag-based post organization

## Adding New Posts

Create new files in `_posts/` following the naming convention:
`YYYY-MM-DD-post-title.md`

Include front matter at the top:
```yaml
---
layout: post
title: "Your Post Title"
tags:
  - graphics
  - interactive
date: YYYY-MM-DD
---
```
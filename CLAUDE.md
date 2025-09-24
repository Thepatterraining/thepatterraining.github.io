# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hexo-based static blog site with Chinese technical content. The site is deployed to GitHub Pages at https://thepatterraining.github.io/.

## Key Commands

### Development
- `npm run server` - Start local development server
- `npm run build` - Generate static files (alias: `hexo generate`)
- `npm run clean` - Clean generated files and cache
- `npm run deploy` - Deploy to GitHub Pages (master branch)

### Content Management
- `hexo new post "title"` - Create new blog post
- `hexo new page "title"` - Create new page
- `hexo new draft "title"` - Create new draft

## Architecture

### Directory Structure
- `source/_posts/` - Blog posts in Markdown format (200+ posts)
- `source/images/` - Image assets for posts
- `themes/next/` - NexT theme for Hexo
- `scaffolds/` - Templates for new posts/pages
- `public/` - Generated static files (not in git)

### Configuration
- `_config.yml` - Main Hexo configuration
- `themes/next/_config.yml` - Theme-specific configuration
- Uses NexT theme with Chinese language settings
- Configured for GitHub Pages deployment
- Includes search, feed generation, and backup features

### Content Management
- Posts use front matter with title, date, and tags
- Chinese language content with technical focus (programming, systems)
- Images stored in `/source/images/` directory
- Posts range from small tutorials to comprehensive technical guides

### Deployment
- Primary deployment: `git@github.com:Thepatterraining/thepatterraining.github.io.git` (master branch)
- Backup branch: backup branch with hexo-git-backup plugin
- Uses hexo-deployer-git for automated deployment

## Important Notes
- Site language is Chinese (zh-CN)
- Timezone: Asia/Shanghai
- Posts use `.html` permalinks instead of default format
- Search functionality enabled via hexo-generator-searchdb
- RSS feed configured at `/atom.xml`
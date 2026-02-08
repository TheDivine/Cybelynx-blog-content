# CyberLynx Blog Content

This repository stores the blog post content that the Astro site loads at runtime.

## Structure
- `index.json` lists post metadata (slug, title, description, pubDate, tags).
- `posts/` contains Markdown files named `{slug}.md`.

## Adding a Post
1. Create a new Markdown file in `posts/` named `{slug}.md`.
2. Add a new entry to `index.json` with the same `slug`.
3. Keep `pubDate` in `YYYY-MM-DD` format.

## Images
Images should be stored in this repo under `images/`.
Reference them in Markdown as `images/your-image.ext`.


## Regenerating index.json
If you're rebuilding from frontmatter-based posts, you can generate `index.json`
using the restore script in the main site repo.

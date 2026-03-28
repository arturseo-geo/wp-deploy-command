# /wp-deploy — Claude Code Command

Deploy HTML posts to WordPress with full compliance — strip DOCTYPE, add JSON-LD schema, set categories/tags/RankMath, clear cache, verify.

## What it does

One command replaces a 15-minute manual deployment process:

1. Strips DOCTYPE/html/head/body from HTML file
2. Preserves JSON-LD schema blocks and CSS
3. Wraps in `<!-- wp:html -->` block
4. Removes h1 tags (WordPress title handles it)
5. Creates/updates WordPress post via REST API
6. Sets categories, tags, and RankMath SEO fields
7. Clears Nginx FastCGI cache
8. Runs post-publish verification (schema, h1 count, CSS, HTTP status)

## Install

Copy `wp-deploy.md` to your Claude Code commands directory:

```bash
cp wp-deploy.md ~/.claude/commands/wp-deploy.md
```

## Usage

In Claude Code:
```
/wp-deploy path/to/post.html
```

Or to update an existing post:
```
/wp-deploy post-id:1234
```

## Configuration

The command uses WordPress REST API with Basic Auth. Update the credentials in the command file to match your site.

## Requirements

- WordPress with Application Passwords enabled
- SSH access to VPS (for cache clearing)
- Nginx with FastCGI cache

## Licence

[MIT](LICENSE)

---

Created by [Artur Ferreira](https://thegeolab.net/about/) at [The GEO Lab](https://thegeolab.net) · [GitHub](https://github.com/arturseo-geo)

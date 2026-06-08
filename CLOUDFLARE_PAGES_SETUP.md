# PCjs on Cloudflare Pages Deployment Guide

## Quick Setup

### Via Cloudflare Dashboard

1. **Connect your GitHub repo** to Cloudflare Pages
2. **Build settings** (should auto-detect):
   - Framework: None (Jekyll)
   - Build command: (empty - uses `wrangler.toml`)
   - Build output directory: `_site`
3. **Environment** (production):
   - Set `JEKYLL_ENV = production`
4. **Deploy**

### Via Wrangler CLI

```bash
npm install -g @cloudflare/wrangler
wrangler pages deploy _site
```

## Configuration

The `wrangler.toml` file handles:
- Build command execution
- Output directory (`_site`)
- Environment variables
- URL redirects for clean URLs

### For Cloudflare Pages subdomain (e.g., `pcjs.pages.dev`):

No additional setup needed. Cloudflare auto-generates the URL.

### For custom domain:

1. **Add domain to Cloudflare zone** (nameservers or CNAME)
2. **In Cloudflare Dashboard**, go to **Pages → pcjs → Custom domains**
3. **Add your custom domain** (e.g., `pcjs.org`)
4. **Update `wrangler.toml`**:
   ```toml
   [env.production]
   routes = [
     { pattern = "pcjs.org", zone_name = "pcjs.org" },
     { pattern = "www.pcjs.org", zone_name = "pcjs.org" },
   ]
   ```
5. **Optionally update `_cloudflare.yml`**:
   ```yaml
   domain: pcjs.org
   url: https://www.pcjs.org
   ```

## Build Process

Same as Netlify:
```bash
bundle install && npm install && npm run build && bundle exec jekyll build
```

Generates static site in `_site/` which Cloudflare Pages publishes.

## Environment Variables (Optional)

Set via **Cloudflare Dashboard → Pages → Settings → Environment variables**:

- `JEKYLL_ENV` = `production` (auto-set for production builds)
- `SITE_URL` = your full URL
- `SITE_DOMAIN` = your domain only

Then use in `_config.yml`:
```yaml
url: ${SITE_URL}
domain: ${SITE_DOMAIN}
```

## Caching & Performance

Cloudflare Pages includes:
- ✅ Global CDN caching
- ✅ Automatic gzip/brotli compression
- ✅ HTTP/2 push
- ✅ Cache purge on every deploy

No additional cache headers needed, but can be customized via `_headers` file (optional).

## Advanced: Custom Headers File

Create `public/_headers` for advanced caching:

```
/*
  Cache-Control: public, max-age=3600

/assets/*
  Cache-Control: public, max-age=31536000, immutable

/js/*
  Cache-Control: public, max-age=31536000, immutable

/css/*
  Cache-Control: public, max-age=31536000, immutable
```

Then in `wrangler.toml`:
```toml
[build]
  command = "..."
  cwd = "/"
  root_dir = "_site"
```

## Local Testing for Cloudflare Pages

```bash
# Standard development
npm run build && bundle exec jekyll serve --config _config.yml,_developer.yml

# Production simulation
npm run build && bundle exec jekyll build

# Test with Cloudflare locally (if using Functions)
wrangler pages dev _site
```

## Troubleshooting

**Build fails on Cloudflare?**
- Check Cloudflare Pages build logs
- Verify Ruby/Node versions match local setup
- Ensure all gems are in Gemfile

**Site looks broken?**
- Check if CSS/JS paths are correct (especially for subdirectories)
- Verify `baseurl` is empty for root domain
- Clear Cloudflare cache: **Caching → Purge Everything**

**Custom domain not working?**
- Verify DNS is pointing to Cloudflare
- Wait 5 minutes for DNS propagation
- Check **Pages → Custom domains** shows as "Active"

## Switching Between Hosts

**GitHub Pages** (original):
- Relies on CNAME file (`www.pcjs.org`)
- Enabled via repository settings

**Netlify**:
- Uses `netlify.toml`
- Connect repo to Netlify dashboard

**Cloudflare Pages** (new):
- Uses `wrangler.toml`
- Connect repo to Cloudflare Pages
- Both can coexist; last deployment wins

## Recommended Setup

For maximum flexibility, keep all three configs:
1. **GitHub Pages** (CNAME for pcjs.org)
2. **Netlify** (backup, alternative URL)
3. **Cloudflare Pages** (primary, global CDN, fast)

Switch between them by changing DNS or GitHub Pages settings.

## Next Steps

1. Push `wrangler.toml` and `_cloudflare.yml` to repo
2. Go to Cloudflare Pages dashboard
3. Connect your GitHub repo
4. Watch the first build log
5. Set custom domain if needed
6. Update DNS if switching from GitHub Pages

# PCjs on Netlify Deployment Guide

## Quick Setup

1. **Connect your repository** to Netlify via GitHub
2. **Default settings** should work:
   - Build command: (leave empty - uses `netlify.toml`)
   - Publish directory: `_site`
3. **Deploy**

The `netlify.toml` file automatically configures the build process.

## Configuration

### For a Netlify subdomain (e.g., `your-site.netlify.app`):
No additional changes needed. The site will auto-detect and work correctly.

### For a custom domain:

1. **Add your domain in Netlify dashboard** under Site settings → Domain management
2. **Update `_config.yml`** (production config):
   ```yaml
   domain: your-custom-domain.com
   url: https://your-custom-domain.com
   ```
3. **Redeploy** on Netlify

### Using environment variables (recommended):

Instead of hardcoding the domain, set Netlify environment variables:

1. Go to **Netlify Dashboard → Site settings → Build & deploy → Environment**
2. Add variables:
   ```
   SITE_URL=https://your-site.netlify.app
   SITE_DOMAIN=your-site.netlify.app
   ```

3. Update `_config.yml`:
   ```yaml
   domain: ${SITE_DOMAIN}
   url: ${SITE_URL}
   ```

## Build Process

The `netlify.toml` runs:
```bash
bundle install && npm install && npm run build && bundle exec jekyll build
```

This:
1. Installs Ruby dependencies (Gems)
2. Installs npm dependencies
3. Runs `npm run build` (gulp tasks for assets)
4. Builds the Jekyll static site

Output goes to `_site/` which Netlify publishes.

## Local Testing for Netlify

To test locally as Netlify would build it:

```bash
# Standard development
npm run build && bundle exec jekyll serve --config _config.yml,_developer.yml

# Production simulation (no _developer.yml)
npm run build && bundle exec jekyll build && bundle exec jekyll serve --config _config.yml
```

## Troubleshooting

**Build fails?**
- Check Netlify deploy logs
- Ensure `Gemfile` and `package.json` are up to date
- Verify Ruby version (netlify.toml specifies 3.2.0)

**Site looks broken after deploy?**
- Clear browser cache (Netlify caches assets)
- Check baseurl is empty (for root domain)
- Verify CSS/JS paths in rendered HTML

**Static assets 404?**
- Run `npm run build` locally to generate assets
- Check `_site/assets/` was generated correctly
- Verify no trailing slashes breaking asset paths

## Rollback to pcjs.org

To restore original GitHub Pages hosting:
1. Remove custom domain from Netlify
2. Revert any changes to `_config.yml`
3. The `CNAME` file (currently `www.pcjs.org`) handles GitHub Pages

## Next Steps

1. Push these files to your repo
2. Connect repo to Netlify
3. Watch the deploy logs
4. Update DNS if using a custom domain

# sct-hub: Deployment Guide

**Purpose:** Detailed guide for deploying sct-hub to GitHub Pages and other platforms

---

## Deployment Options

```
┌─────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT TARGETS                        │
├─────────────────────────────────────────────────────────────┤
│  GitHub Pages    │  Free, custom domain, HTTPS              │
│  Netlify         │  Auto-deploy, previews, forms            │
│  Cloudflare Pages│  Fast CDN, workers integration           │
│  IPFS           │  Decentralized, censorship-resistant     │
│  Self-hosted    │  Full control, any server                │
└─────────────────────────────────────────────────────────────┘

           All deploy from same generated static files
```

---

## GitHub Pages (Recommended)

### Why GitHub Pages?

**SCT-aligned benefits:**
- ✅ **Sovereignty:** Git repo is source of truth, can move anywhere
- ✅ **Locality:** Generate locally, push when ready
- ✅ **Portability:** Static files work on any host
- ✅ **Openness:** Git is open protocol, not proprietary
- ✅ **Free:** Custom domain, HTTPS, global CDN

**Limitations:**
- 1GB size limit (plenty for personal sites)
- Public repos only (unless GitHub Pro)
- GitHub's infrastructure (not fully decentralized)

**Trade-off:** GitHub Pages is pragmatic middle-ground - better than Linktree, stepping stone to full decentralization.

---

## Setup: GitHub Pages

### Option 1: Manual Deployment

**Step 1: Create GitHub repository**

```bash
# Create repo on GitHub (web UI or gh CLI)
gh repo create EverythingSings.github.io --public

# Or use custom repo name
gh repo create my-hub --public
```

**Step 2: Configure sct-hub**

```bash
# Initialize sct-hub
sct-hub init \
  --title "EverythingSings.Art" \
  --url "https://everythingsings.art" \
  --author "EverythingSings"

# Set deployment config
sct-hub config set github_repo "EverythingSings/EverythingSings.github.io"
sct-hub config set github_branch "gh-pages"
```

**Step 3: Add content**

```bash
# Add links
sct-hub add link \
  --title "Plurcast" \
  --url "https://github.com/EverythingSings/plurcast" \
  --category "Projects" \
  --icon "📡"

sct-hub add link \
  --title "RSS WASM" \
  --url "https://github.com/EverythingSings/rss-wasm" \
  --category "Projects" \
  --icon "📰"

# Import blog posts
rss-wasm parse https://blog.example.com/feed.xml --format json | \
  sct-hub import json - --category "Blog"
```

**Step 4: Generate site**

```bash
# Generate to output directory
sct-hub generate \
  --output ./public \
  --base-url "https://everythingsings.art" \
  --theme minimal \
  --minify
```

**Step 5: Deploy**

```bash
# Initialize git in output directory
cd ./public
git init
git checkout -b gh-pages
git add .
git commit -m "Initial site deployment"

# Add remote and push
git remote add origin git@github.com:EverythingSings/EverythingSings.github.io.git
git push -u origin gh-pages
```

**Step 6: Configure GitHub Pages**

1. Go to repo settings → Pages
2. Source: Deploy from branch `gh-pages`
3. Custom domain: `everythingsings.art`
4. Enforce HTTPS: ✅

**Step 7: Configure DNS**

For `everythingsings.art`:
```
# A records
@  A  185.199.108.153
@  A  185.199.109.153
@  A  185.199.110.153
@  A  185.199.111.153

# OR CNAME record (if using subdomain)
www  CNAME  everythingsings.github.io.
```

### Option 2: Automated Deployment (Recommended)

**Create GitHub Actions workflow:**

`.github/workflows/deploy-hub.yml`:

```yaml
name: Deploy SCT Hub

on:
  push:
    branches: [main]
  # Auto-update from RSS feeds every 6 hours
  schedule:
    - cron: '0 */6 * * *'
  # Manual trigger
  workflow_dispatch:

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          profile: minimal

      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: |
            ~/.cargo
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}

      - name: Install sct-hub
        run: |
          # TODO: Replace with release binary once published
          cargo install --git https://github.com/EverythingSings/sct-hub --force

      - name: Install rss-wasm
        run: |
          cargo install --git https://github.com/EverythingSings/rss-wasm --force

      - name: Initialize database (if needed)
        run: |
          if [ ! -f ~/.local/share/sct/sct.db ]; then
            sct-hub init \
              --title "EverythingSings.Art" \
              --url "https://everythingsings.art" \
              --author "EverythingSings"
          fi

      - name: Import RSS feeds
        run: |
          # Import from feeds.opml or individual feeds
          if [ -f feeds.opml ]; then
            rss-wasm parse feeds.opml --format json | \
              sct-hub import json - --category "Blog"
          fi

      - name: Add manual links (if not in DB)
        run: |
          # Add links from links.json config file
          if [ -f links.json ]; then
            cat links.json | \
              jq -r '.[] | @json' | \
              while read link; do
                sct-hub add link \
                  --title "$(echo $link | jq -r '.title')" \
                  --url "$(echo $link | jq -r '.url')" \
                  --category "$(echo $link | jq -r '.category')" \
                  --icon "$(echo $link | jq -r '.icon')"
              done
          fi

      - name: Generate static site
        run: |
          sct-hub generate \
            --output ./public \
            --base-url "https://everythingsings.art" \
            --theme minimal \
            --minify \
            --clean

      - name: Add CNAME
        run: |
          echo "everythingsings.art" > ./public/CNAME

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          commit_message: 'Deploy site: ${{ github.event.head_commit.message }}'

      - name: Publish update to Nostr (optional)
        if: github.event_name == 'push'
        run: |
          # TODO: Integrate with Plurcast
          # echo "Updated site: ${{ github.event.head_commit.message }}" | \
          #   plurcast publish --platform nostr --pow 18
```

**Workflow features:**
- ✅ Auto-deploy on push to main
- ✅ Auto-sync RSS feeds every 6 hours
- ✅ Manual trigger via GitHub UI
- ✅ Cache dependencies for faster builds
- ✅ Generate + deploy in one step

---

## Alternative 1: Netlify

**Benefits:**
- Deploy previews for PRs
- Form handling
- Functions (serverless)
- Automatic HTTPS

**Setup:**

1. **Connect repo to Netlify:**
   - Link GitHub repo
   - Set build command: `sct-hub generate --output ./public`
   - Set publish directory: `public`

2. **netlify.toml** (optional):
```toml
[build]
  command = "sct-hub generate --output ./public --minify"
  publish = "public"

[build.environment]
  RUST_VERSION = "1.82"

[[redirects]]
  from = "/posts/*"
  to = "/posts/:splat"
  status = 200
```

---

## Alternative 2: IPFS (Decentralized)

**Benefits:**
- Fully decentralized
- Censorship-resistant
- Content-addressed (CID)
- IPNS for mutable links

**Setup:**

```bash
# Generate site
sct-hub generate --output ./public

# Install IPFS (ipfs-desktop or kubo)
# https://docs.ipfs.io/install/

# Add to IPFS
ipfs add -r ./public

# Output: added QmXXX... public
# Site available at: https://ipfs.io/ipfs/QmXXX...

# Create IPNS name (mutable pointer)
ipfs name publish QmXXX...

# Output: Published to k51... (your IPNS key)
# Site available at: https://ipfs.io/ipns/k51...
```

**Update workflow:**
```bash
# Regenerate site
sct-hub generate --output ./public

# Update IPFS
NEW_CID=$(ipfs add -r -Q ./public)
ipfs name publish $NEW_CID

# Optional: Pin to Pinata/Infura for reliability
```

---

## Alternative 3: Self-Hosted

**Benefits:**
- Full control
- Any server (VPS, home server)
- Custom server features

**Setup (nginx example):**

```bash
# Generate site
sct-hub generate --output ./public

# Copy to server
rsync -avz ./public/ user@server:/var/www/hub/

# nginx config
server {
    listen 80;
    server_name everythingsings.art;
    root /var/www/hub;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# SSL with Let's Encrypt
certbot --nginx -d everythingsings.art
```

---

## Deployment Comparison

| Feature | GitHub Pages | Netlify | IPFS | Self-Hosted |
|---------|-------------|---------|------|-------------|
| **SCT Sovereignty** | ⚠️ Medium | ⚠️ Medium | ✅ High | ✅ High |
| **Cost** | Free | Free tier | Free* | $5-50/mo |
| **HTTPS** | ✅ Auto | ✅ Auto | ⚠️ Gateway | Manual |
| **Custom Domain** | ✅ Yes | ✅ Yes | ✅ DNSLink | ✅ Yes |
| **Build Time** | ~2 min | ~1 min | Local only | Local only |
| **CDN** | ✅ GitHub | ✅ Netlify | ✅ IPFS | Optional |
| **Censorship Resistance** | ❌ Low | ❌ Low | ✅ High | ⚠️ Medium |

*IPFS pinning services may have costs

**Recommendation for starting:**
1. **Start:** GitHub Pages (easy, free, good enough)
2. **Upgrade:** IPFS + GitHub Pages (both)
3. **Ultimate:** Self-hosted + IPFS (full sovereignty)

---

## Multi-Target Deployment

**Deploy to multiple platforms simultaneously:**

```bash
#!/bin/bash
# deploy.sh - Deploy to all targets

# Generate once
sct-hub generate \
  --output ./public \
  --base-url "https://everythingsings.art" \
  --minify

# GitHub Pages
cd ./public
git add .
git commit -m "Update site $(date +%Y-%m-%d)"
git push origin gh-pages
cd ..

# IPFS
NEW_CID=$(ipfs add -r -Q ./public)
ipfs name publish $NEW_CID
echo "IPFS: https://ipfs.io/ipns/$(ipfs key list -l | grep self | awk '{print $1}')"

# Netlify (if using CLI)
netlify deploy --prod --dir=./public

# Announce update
echo "Site deployed to GitHub Pages, IPFS, and Netlify 🚀" | \
  plurcast publish --platform nostr,mastodon --pow 18
```

---

## Continuous Integration

### Daily Content Sync

**Cron job for auto-importing blog posts:**

```bash
# crontab -e
0 */6 * * * /usr/local/bin/sync-hub.sh
```

**sync-hub.sh:**
```bash
#!/bin/bash
set -e

# Import RSS feeds
rss-wasm parse ~/feeds.opml --format json | \
  sct-hub import json - --category "Blog"

# Check if there are new items
NEW_COUNT=$(sct-hub list posts --format json | \
  jq 'map(select(.source == "rss" and .created_at > (now - 21600))) | length')

if [ "$NEW_COUNT" -gt 0 ]; then
  echo "Found $NEW_COUNT new posts, regenerating site..."

  # Regenerate site
  sct-hub generate --output ./public --minify

  # Deploy (assuming git setup)
  cd ./public
  git add .
  git commit -m "Auto-update: $NEW_COUNT new posts"
  git push origin gh-pages

  # Announce
  echo "Site updated with $NEW_COUNT new blog posts! Check them out at https://everythingsings.art" | \
    plurcast publish --platform nostr --pow 15
fi
```

---

## Monitoring & Analytics

### Privacy-Respecting Analytics

**Option 1: No analytics** (most SCT-aligned)
- Respect user privacy
- No tracking
- Pure sovereignty

**Option 2: Self-hosted analytics**
- Plausible (self-hosted)
- Umami Analytics
- GoatCounter

**Implementation:**
```bash
# Add to template (optional, user choice)
sct-hub config set analytics_provider "plausible"
sct-hub config set analytics_domain "stats.everythingsings.art"

# Template includes script only if configured
# No tracking cookies, privacy-respecting
```

### Site Health

**Check deployment:**
```bash
# Test generated site
sct-hub generate --output /tmp/test-site
cd /tmp/test-site
python3 -m http.server 8000
# Open http://localhost:8000 to verify

# Test links
wget --spider -r -nd -nv http://localhost:8000 2>&1 | \
  grep -B1 "broken link"
```

---

## Troubleshooting

### Common Issues

**1. GitHub Pages not updating:**
```bash
# Check Actions status
gh run list --workflow=deploy-hub.yml

# View logs
gh run view <run-id> --log

# Manual trigger
gh workflow run deploy-hub.yml
```

**2. Custom domain not working:**
- DNS propagation takes up to 24-48 hours
- Check CNAME record: `dig everythingsings.art`
- Verify CNAME file in gh-pages branch
- Check GitHub Pages settings

**3. Site generation fails:**
```bash
# Test locally first
sct-hub generate --output /tmp/test --verbose

# Check database
sqlite3 ~/.local/share/sct/sct.db "SELECT COUNT(*) FROM hub_links;"

# Verify templates
sct-hub themes list
```

**4. RSS import not working:**
```bash
# Test RSS parsing separately
rss-wasm parse feed.xml --format json

# Check import command
rss-wasm parse feed.xml | \
  sct-hub import json - --dry-run --verbose
```

---

## Best Practices

### Version Control

**What to commit:**
- ✅ Configuration files (`links.json`, `feeds.opml`)
- ✅ Custom themes/templates
- ✅ GitHub Actions workflows
- ✅ Documentation

**What NOT to commit:**
- ❌ Generated `./public` directory (unless using subtree deploy)
- ❌ SQLite database (local-first, regenerate from config)
- ❌ Temporary files

**Example `.gitignore`:**
```
# Generated files
public/
site/
*.db
*.db-shm
*.db-wal

# Build artifacts
target/
```

### Backup Strategy

**1. SQLite database:**
```bash
# Export to JSON
sct-hub export all --format json > hub-backup.json

# Backup database file
cp ~/.local/share/sct/sct.db ~/backups/sct-$(date +%Y%m%d).db
```

**2. Configuration:**
```bash
# Export config
sct-hub config export > config-backup.json
```

**3. Regeneration:**
```bash
# Restore from backup
sct-hub import json hub-backup.json
sct-hub config import config-backup.json
sct-hub generate --output ./public
```

---

## Next Steps

After deploying:

1. **Test your site:** Visit your custom domain, verify all links work
2. **Set up automation:** GitHub Actions for auto-deploy
3. **Import content:** RSS feeds, Nostr posts, manual links
4. **Customize theme:** Edit templates to match your style
5. **Announce:** Share on Nostr/Mastodon via Plurcast
6. **Monitor:** Check deployment logs, site uptime
7. **Iterate:** Add more content, refine design

---

## Resources

- **GitHub Pages Docs:** https://docs.github.com/en/pages
- **GitHub Actions:** https://docs.github.com/en/actions
- **IPFS Docs:** https://docs.ipfs.io
- **Netlify Docs:** https://docs.netlify.com
- **DNS Setup:** https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site

---

**Ready to replace Linktree?** Start with GitHub Pages, iterate from there. The beauty of SCT is you can always migrate to a more sovereign solution - your data stays with you.

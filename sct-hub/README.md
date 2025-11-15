# sct-hub: Personal Content Hub & Site Generator

**Status:** 🎯 Design Phase
**Version:** 0.1.0-alpha
**Purpose:** SCT-compliant replacement for Linktree

---

## What is sct-hub?

A personal content hub and static site generator that puts **you** in control of your online presence.

**Replace this:**
- ❌ Linktree - your links on their servers
- ❌ About.me - platform lock-in
- ❌ Carrd - proprietary builder
- ❌ Beacons - tracked, centralized

**With this:**
- ✅ Your links in your local SQLite database
- ✅ Static HTML/CSS you control
- ✅ Deploy anywhere (GitHub Pages, IPFS, self-hosted)
- ✅ Compose with other SCT tools
- ✅ Publish to Nostr/Mastodon

---

## The Vision

### Before: Linktree (Platform-Dependent)

```
You
 └─> Linktree Server (their control)
      └─> Your links (trapped)
           └─> Visitors (tracked)
```

**Problems:**
- Links live on their servers
- Platform can change/shut down anytime
- No customization without paying
- Tracking and analytics they control
- Can't integrate with other tools

### After: sct-hub (Sovereign)

```
You
 ├─> Local SQLite DB (your control)
 │    ├─> Links, posts, projects
 │    └─> Content from RSS/Nostr
 │
 ├─> Static Site Generator
 │    ├─> Generate offline
 │    └─> Deploy anywhere
 │
 └─> Integrations
      ├─> rss-wasm (import blog posts)
      ├─> sct-npow (import Nostr content)
      └─> Plurcast (publish updates)
```

**Benefits:**
- ✅ **Sovereignty:** Your data, your keys, your execution
- ✅ **Composability:** Works with other SCT tools via JSON/SQLite
- ✅ **Locality:** Generate completely offline
- ✅ **Portability:** Deploy to any static host
- ✅ **Openness:** Publish to decentralized platforms (Nostr)

---

## Quick Start (When Implemented)

```bash
# 1. Initialize hub
sct-hub init \
  --title "My Personal Hub" \
  --url "https://mysite.com"

# 2. Add links
sct-hub add link \
  --title "My Blog" \
  --url "https://blog.mysite.com" \
  --category "Writing" \
  --icon "📝"

sct-hub add link \
  --title "GitHub" \
  --url "https://github.com/username" \
  --category "Code" \
  --icon "💻"

# 3. Import blog posts (via rss-wasm)
rss-wasm parse https://blog.mysite.com/feed.xml | \
  sct-hub import json - --category "Blog"

# 4. Generate static site
sct-hub generate \
  --output ./public \
  --theme minimal \
  --minify

# 5. Deploy to GitHub Pages
cd ./public
git add .
git commit -m "Deploy site"
git push origin gh-pages

# 6. Announce on Nostr (via Plurcast)
echo "Check out my new personal hub! https://mysite.com" | \
  plurcast publish --platform nostr --pow 18
```

---

## Features

### Core Features

- **Link Management:** Add, edit, remove, organize links with categories
- **Content Import:** Auto-import blog posts from RSS feeds
- **Static Generation:** Generate fast, accessible HTML/CSS sites
- **Themes:** Multiple built-in themes, custom theme support
- **Templates:** Handlebars-based templates for customization
- **Preview Server:** Local preview before deploying
- **Offline-First:** Everything works without internet

### SCT Integration

- **rss-wasm:** Auto-import your blog posts as content
- **sct-npow:** Import high-quality Nostr essays
- **Plurcast:** Publish site updates to Nostr/Mastodon
- **sct-vec (future):** Semantic search and related content

### Deployment

- **GitHub Pages:** One-command deployment with custom domain
- **GitHub Actions:** Automated builds and deploys
- **IPFS:** Decentralized, censorship-resistant hosting
- **Self-Hosted:** Deploy to any static web server
- **Multi-Target:** Deploy to multiple platforms simultaneously

---

## Architecture

Following the SCT 4-layer specification:

### Layer 1: Core (Pure WASM, Zero I/O)
- Link/content validation
- Markdown rendering
- URL normalization
- no_std compatible

### Layer 2: Protocol (Optional Network)
- Metadata fetching (Open Graph)
- Nostr publishing
- WASI 0.2 networking

### Layer 3: Storage (Required)
- SQLite database (`~/.local/share/sct/sct.db`)
- Shared with other SCT tools
- Links, content, configuration

### Layer 4: Application (CLI + Generator)
- Command-line interface
- Static site generator
- Template engine (Handlebars)
- Preview server

---

## Documentation

### Design Documents

- **[sct-hub-spec.md](../docs/sct-hub-spec.md)** - Complete technical specification
- **[DEPLOYMENT.md](./DEPLOYMENT.md)** - Deployment guide (GitHub Pages, IPFS, etc.)
- **[INTEGRATION.md](./INTEGRATION.md)** - SCT ecosystem integration patterns

### Key Sections

**From sct-hub-spec.md:**
- Complete CLI command reference
- SQLite schema design
- Template engine details
- Feature flags and build options

**From DEPLOYMENT.md:**
- GitHub Pages setup (manual + automated)
- GitHub Actions workflow
- IPFS deployment
- Custom domain configuration
- Multi-target deployment

**From INTEGRATION.md:**
- Integration with rss-wasm (blog import)
- Integration with sct-npow (Nostr content)
- Integration with Plurcast (publishing)
- Complete workflow examples
- Multi-agent patterns

---

## Use Cases

### 1. Personal Landing Page

Replace Linktree with sovereign alternative:

```bash
sct-hub add link --title "Blog" --url "..." --icon "📝"
sct-hub add link --title "Twitter" --url "..." --icon "🐦"
sct-hub add link --title "GitHub" --url "..." --icon "💻"
sct-hub add link --title "Nostr" --url "..." --icon "⚡"
sct-hub generate --theme cards --output ./public
```

### 2. Developer Portfolio

Showcase projects with blog posts:

```bash
# Add project links
sct-hub add project --title "Plurcast" --url "..." --category "Projects"
sct-hub add project --title "rss-wasm" --url "..." --category "Projects"

# Import blog posts
rss-wasm parse ~/feeds.opml | sct-hub import json - --category "Blog"

# Generate with blog theme
sct-hub generate --theme blog --output ./public
```

### 3. Content Creator Hub

Aggregate content from multiple sources:

```bash
# Import from blog
rss-wasm parse blog-feed.xml | sct-hub import json -

# Import from Nostr
sct-npow fetch --author <npub> --kind 30023 | sct-hub import nostr -

# Generate and publish
sct-hub generate --output ./public
sct-hub publish github-pages
echo "New content!" | plurcast publish --platform nostr,mastodon
```

### 4. SCT Dogfooding

Demonstrate SCT principles in practice:

> "My website is built with sct-hub, pulls blog posts via rss-wasm, publishes updates via Plurcast, and deploys to GitHub Pages. The entire site generates offline from my local SQLite database. No Linktree. No platform lock-in. This is sovereign computing."

---

## Why sct-hub?

### The Problem with Platforms

**Linktree:**
- Your links live on their servers
- Platform can change pricing/features anytime
- No data portability
- Tracking and analytics they control
- Limited customization
- Can't integrate with other tools

**Carrd, About.me, Beacons, etc.:**
- Same problems: platform lock-in
- Proprietary builders
- Subscription fees
- Limited control

### The SCT Alternative

**sct-hub gives you:**
- ✅ **Data ownership:** Links in your SQLite database
- ✅ **Offline generation:** Works without internet
- ✅ **Deploy anywhere:** GitHub, IPFS, self-hosted
- ✅ **Full customization:** Edit templates, CSS, HTML
- ✅ **Integration:** Compose with other SCT tools
- ✅ **Decentralized publishing:** Nostr, Mastodon via Plurcast
- ✅ **Future-proof:** Can move to any host, any platform

### The Dogfooding Argument

sct-hub is being built to replace the developer's own Linktree:

- **Before:** everythingsings.art → Linktree (doesn't follow SCT)
- **After:** everythingsings.art → sct-hub (demonstrates all 5 axioms)

**The message:**
> "I'm asking you to use sovereign tools. Here's my website built with those same tools. I'm dogfooding the vision."

---

## Roadmap

### Phase 1: Core + Storage (Q4 2024 - Current)
- ✅ Design specification
- ⏳ Core data model (Layer 1)
- ⏳ SQLite schema (Layer 3)
- ⏳ Basic CLI (`init`, `add`, `list`)
- ⏳ Simple static generation

**Deliverable:** Can add links and generate basic HTML site

### Phase 2: Site Generation (Q1 2025)
- ⏳ Template engine (Handlebars)
- ⏳ Multiple themes (minimal, cards, blog)
- ⏳ RSS/JSON feed generation
- ⏳ Preview server
- ⏳ Markdown rendering for posts

**Deliverable:** Full-featured static site generator

### Phase 3: Integration (Q1 2025)
- ⏳ RSS import via rss-wasm
- ⏳ Nostr publishing via Plurcast
- ⏳ GitHub Pages deployment
- ⏳ Auto-sync workflows

**Deliverable:** Complete SCT ecosystem integration

### Phase 4: Advanced Features (Q2 2025)
- ⏳ Metadata fetching (Open Graph)
- ⏳ Custom themes API
- ⏳ IPFS publishing
- ⏳ Analytics (privacy-respecting, local)

**Deliverable:** Feature-complete 1.0 release

---

## Contributing

**Status:** Not yet open source (design phase)

Once we reach a stable implementation, contributions will be welcome for:
- Theme designs
- Template improvements
- Integration examples
- Documentation
- Deployment guides
- Bug fixes

---

## Philosophy

sct-hub embodies the five SCT axioms:

1. **Sovereignty** - Users own their data, keys, and execution
   - Links stored locally in SQLite
   - No platform dependency
   - Can regenerate/migrate anytime

2. **Composability** - Tools compose via text streams
   - Import from rss-wasm
   - Export to Plurcast
   - JSON I/O for piping

3. **Locality** - Offline-first, local storage is truth
   - Generate completely offline
   - Preview locally before deploy
   - No cloud dependency

4. **Portability** - Deploy anywhere
   - WASM-capable, cross-platform
   - Static output works on any host
   - GitHub Pages, IPFS, self-hosted

5. **Openness** - Decentralized protocols only
   - Publish to Nostr (open protocol)
   - No proprietary APIs
   - Git for version control

---

## Comparison

| Feature | Linktree | Carrd | sct-hub |
|---------|----------|-------|---------|
| **Data Ownership** | ❌ Platform | ❌ Platform | ✅ Local SQLite |
| **Offline Generation** | ❌ No | ❌ No | ✅ Yes |
| **Deploy Anywhere** | ❌ No | ⚠️ Limited | ✅ Yes |
| **Custom Domain** | 💰 Paid | ✅ Yes | ✅ Yes |
| **Customization** | ⚠️ Limited | ⚠️ Builder | ✅ Full control |
| **Integration** | ❌ No | ❌ No | ✅ SCT tools |
| **Nostr Publishing** | ❌ No | ❌ No | ✅ Via Plurcast |
| **Open Source** | ❌ No | ❌ No | ✅ Yes (future) |
| **Privacy** | ⚠️ Tracked | ⚠️ Tracked | ✅ No tracking |
| **Cost** | Free/Paid | Paid | ✅ Free |
| **Platform Risk** | ⚠️ High | ⚠️ High | ✅ None |

---

## FAQ

**Q: Why not just use Linktree?**
A: Linktree violates all five SCT axioms. Your links live on their servers, you can't use it offline, you're locked to their platform, and you can't integrate it with other tools. sct-hub puts you in control.

**Q: How is this different from a static site generator like Hugo?**
A: sct-hub is designed specifically for personal hubs (links + content), integrates with other SCT tools (rss-wasm, Plurcast, sct-npow), and follows the SCT 4-layer architecture for composability.

**Q: Do I need to know how to code?**
A: No! The CLI makes it easy: `sct-hub add link --title "..." --url "..."`. But if you want custom themes, you can edit templates.

**Q: Can I import my existing Linktree?**
A: Yes! You can manually add links or import from JSON. We may add a Linktree scraper in the future.

**Q: What about analytics?**
A: sct-hub doesn't track visitors. If you want analytics, use privacy-respecting self-hosted options like Plausible or GoatCounter.

**Q: Can I use this for my business?**
A: Absolutely! sct-hub works for personal sites, portfolios, small business landing pages, etc.

**Q: What if GitHub Pages shuts down?**
A: That's the beauty of static sites - your content is just HTML/CSS files. Deploy to Netlify, IPFS, self-hosted, etc. No lock-in.

---

## License

TBD (will be open source upon stable release)

---

## Links

- **Main Project:** [EverythingSings](https://github.com/EverythingSings/EverythingSings)
- **SCT Philosophy:** [README.md](../README.md)
- **Specification:** [sct-hub-spec.md](../docs/sct-hub-spec.md)
- **Deployment Guide:** [DEPLOYMENT.md](./DEPLOYMENT.md)
- **Integration Guide:** [INTEGRATION.md](./INTEGRATION.md)

---

**sct-hub** - Your content, your data, your site. No platform required.

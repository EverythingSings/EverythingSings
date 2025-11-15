# sct-hub: Personal Content Hub & Site Generator

**Status:** 🎯 DESIGN PHASE
**Version:** 0.1.0-alpha
**Layer:** APPLICATION
**Repository:** TBD

---

## Overview

`sct-hub` (Sovereign Composable Tools - Hub) is a personal content hub and static site generator that replaces centralized link aggregators like Linktree with a sovereign, composable alternative.

Unlike platform-based solutions, `sct-hub` treats **local data as the source of truth** - your content lives in your SQLite database, generates static sites you control, and publishes metadata to decentralized platforms.

### Design Philosophy

Following the SCT specification:
- **Sovereignty:** Your links/content stored locally, site generation offline
- **Composability:** Composes with rss-wasm (import blog posts), Plurcast (publish updates)
- **Locality:** Local-first SQLite storage, static HTML works offline
- **Portability:** WASM-first (wasm32-wasip2), deploy anywhere (GitHub Pages, IPFS, etc.)
- **Openness:** Publish metadata to Nostr, no platform lock-in

---

## Why sct-hub?

### The Problem

**Linktree and similar services violate SCT principles:**
- Your links live on their servers (no sovereignty)
- Requires their service to work (no locality)
- Proprietary, centralized platform (no openness)
- Isolated silo (no composability)
- Platform lock-in (no portability)

### The Solution

**Local-first personal hub with static site generation:**
- Links stored in **your local SQLite database**
- Generate **static HTML/CSS** you control
- Deploy **anywhere** (GitHub Pages, Netlify, IPFS)
- Compose with **other SCT tools** (rss-wasm auto-imports blog posts)
- Publish metadata to **Nostr** for decentralized discovery

### Use Cases

**For Personal Websites:**
- Replace Linktree with sovereign alternative
- Showcase projects, blog posts, social links
- Customize design with templates
- Deploy to custom domain

**For Content Creators:**
- Auto-import blog posts from RSS feeds via rss-wasm
- Publish site updates to Nostr/Mastodon via Plurcast
- Archive all your content locally
- Regenerate site anytime from local data

**For Developers:**
- Dogfood your own SCT tools
- Demonstrate SCT principles in practice
- Compose with other tools in your stack
- Full control over presentation and data

---

## Architecture

Following the 4-layer SCT specification:

### Layer 1: Core (Pure WASM, Zero I/O)

**Purpose:** Content model, validation, transformation

```rust
// Core functionality (no I/O)
pub struct Link {
    pub id: String,
    pub title: String,
    pub url: String,
    pub description: Option<String>,
    pub category: Option<String>,
    pub icon: Option<String>,
    pub priority: i32,
}

pub struct ContentItem {
    pub id: String,
    pub title: String,
    pub content: String,
    pub content_type: ContentType,
    pub published_at: i64,
    pub source: Source,
}

pub enum Source {
    Manual,
    RSSImport { feed_url: String },
    NostrImport { event_id: String },
}

// URL validation, markdown rendering, etc.
impl Link {
    pub fn validate(&self) -> Result<(), ValidationError> {
        // Pure validation logic
    }
}
```

**Features:**
- Link/content validation
- Markdown rendering
- URL parsing and normalization
- Category/tag management
- no_std compatible

### Layer 2: Protocol (Optional Network)

**Purpose:** Fetch metadata, publish to decentralized protocols

**Feature flag:** `network`

```rust
#[cfg(feature = "network")]
pub struct MetadataFetcher {
    // Fetch Open Graph metadata for links
    pub async fn fetch_metadata(&self, url: &str) -> LinkMetadata {
        // HTTP request for og:title, og:description, og:image
    }
}

#[cfg(feature = "network")]
pub struct NostrPublisher {
    // Publish site updates to Nostr
    pub async fn publish_update(&self, site: &Site) -> Result<EventId> {
        // Uses Plurcast internally or direct Nostr publishing
    }
}
```

**WASI 0.2 networking:**
- HTTP requests for metadata fetching
- Optional Nostr publishing
- Offline-first (optional feature)

### Layer 3: Storage (Required Persistence)

**Purpose:** Local content storage, site configuration

**Feature flag:** `storage` (default)

**Database:** SQLite (shared with other SCT tools)

**Schema:**
```sql
-- Links table
CREATE TABLE hub_links (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  url TEXT NOT NULL,
  description TEXT,
  category TEXT,
  icon TEXT,                      -- URL or emoji
  priority INTEGER DEFAULT 0,     -- Display order
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  metadata JSON,                  -- og:tags, custom fields
  source TEXT DEFAULT 'manual',   -- manual, rss, nostr
  source_id TEXT,                 -- feed_url or event_id
  published BOOLEAN DEFAULT 1,    -- Show on site
  archived BOOLEAN DEFAULT 0
);

CREATE INDEX idx_priority ON hub_links(priority DESC, created_at DESC);
CREATE INDEX idx_category ON hub_links(category);
CREATE INDEX idx_published ON hub_links(published, archived);

-- Content items (blog posts, projects, etc.)
CREATE TABLE hub_content (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,      -- URL-friendly identifier
  content TEXT NOT NULL,           -- Markdown or HTML
  content_type TEXT DEFAULT 'markdown',
  summary TEXT,
  published_at INTEGER NOT NULL,
  updated_at INTEGER,
  category TEXT,
  tags JSON,                       -- ["rust", "wasm"]
  source TEXT DEFAULT 'manual',
  source_id TEXT,                  -- RSS item ID or Nostr event ID
  featured BOOLEAN DEFAULT 0,
  published BOOLEAN DEFAULT 1,
  metadata JSON
);

CREATE INDEX idx_published_at ON hub_content(published_at DESC);
CREATE INDEX idx_slug ON hub_content(slug);
CREATE INDEX idx_featured ON hub_content(featured, published_at DESC);

-- Site configuration
CREATE TABLE hub_config (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  updated_at INTEGER NOT NULL
);

-- Site metadata
INSERT INTO hub_config (key, value, updated_at) VALUES
  ('site_title', 'My Personal Hub', strftime('%s', 'now')),
  ('site_description', 'Links, projects, and blog posts', strftime('%s', 'now')),
  ('site_url', 'https://example.com', strftime('%s', 'now')),
  ('author_name', '', strftime('%s', 'now')),
  ('author_bio', '', strftime('%s', 'now')),
  ('theme', 'default', strftime('%s', 'now')),
  ('social_nostr', '', strftime('%s', 'now')),
  ('social_mastodon', '', strftime('%s', 'now')),
  ('social_github', '', strftime('%s', 'now'));

-- Categories/sections
CREATE TABLE hub_categories (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  priority INTEGER DEFAULT 0,
  created_at INTEGER NOT NULL
);

CREATE INDEX idx_cat_priority ON hub_categories(priority DESC);
```

**Location:** `~/.local/share/sct/sct.db` (shared with rss-wasm, plurcast, sct-npow)

### Layer 4: Application (CLI + Site Generator)

**Purpose:** User interface, site generation, deployment

**Binary:** `sct-hub`

**Commands:**
```bash
sct-hub init          # Initialize hub database
sct-hub add           # Add link or content
sct-hub list          # List links/content
sct-hub edit          # Edit existing item
sct-hub remove        # Remove item
sct-hub import        # Import from RSS/JSON
sct-hub generate      # Generate static site
sct-hub preview       # Local preview server
sct-hub publish       # Publish to Nostr/deployment
sct-hub config        # Manage site configuration
sct-hub themes        # List/install themes
```

---

## CLI Interface

### Command: `init`

Initialize hub database and configuration.

```bash
sct-hub init [OPTIONS]

Options:
  --title <TITLE>           Site title
  --url <URL>               Site URL (for canonical links)
  --author <NAME>           Author name
  --theme <THEME>           Theme name (default: minimal)
  --template <PATH>         Custom template directory
  --force                   Overwrite existing config

Examples:
  # Interactive setup
  sct-hub init

  # Non-interactive
  sct-hub init \
    --title "EverythingSings.Art" \
    --url "https://everythingsings.art" \
    --author "EverythingSings" \
    --theme minimal
```

### Command: `add`

Add a link or content item.

```bash
sct-hub add [TYPE] [OPTIONS]

Types:
  link                      Add a link
  post                      Add a blog post/content
  project                   Add a project

Options (link):
  --title <TITLE>           Link title (required)
  --url <URL>               Link URL (required)
  --description <TEXT>      Description
  --category <CAT>          Category/section
  --icon <EMOJI|URL>        Icon (emoji or image URL)
  --priority <N>            Display order (higher = top)
  --fetch-metadata          Fetch Open Graph metadata

Options (post):
  --title <TITLE>           Post title (required)
  --file <PATH>             Markdown file to import
  --content <TEXT>          Content (if not using --file)
  --slug <SLUG>             URL slug (auto-generated if omitted)
  --published-at <TIME>     Publication date (default: now)
  --category <CAT>          Category
  --tags <TAGS>             Comma-separated tags
  --featured                Mark as featured
  --draft                   Save as draft (not published)

Examples:
  # Add link
  sct-hub add link \
    --title "My Blog" \
    --url "https://blog.example.com" \
    --category "Writing" \
    --icon "📝" \
    --fetch-metadata

  # Add blog post from file
  sct-hub add post \
    --file ~/blog/my-post.md \
    --category "Essays" \
    --tags "rust,wasm,sct" \
    --featured

  # Add project
  sct-hub add project \
    --title "Plurcast" \
    --url "https://github.com/user/plurcast" \
    --description "Multi-platform publishing tool" \
    --category "Projects"
```

### Command: `list`

List links and content.

```bash
sct-hub list [TYPE] [OPTIONS]

Types:
  links                     List links
  posts                     List content/posts
  all                       List everything (default)

Options:
  --category <CAT>          Filter by category
  --published               Only published items
  --drafts                  Only drafts
  --archived                Include archived items
  --format <FMT>            Output format: table|json|jsonl (default: table)
  --limit <N>               Limit results

Examples:
  # List all links
  sct-hub list links

  # List published posts as JSON
  sct-hub list posts --published --format json

  # List by category
  sct-hub list --category "Projects"
```

### Command: `edit`

Edit existing item.

```bash
sct-hub edit [ID] [OPTIONS]

Options:
  --title <TITLE>           Update title
  --url <URL>               Update URL (links only)
  --description <TEXT>      Update description
  --category <CAT>          Update category
  --content <TEXT>          Update content (posts only)
  --file <PATH>             Update from file (posts only)
  --publish                 Publish draft
  --unpublish               Unpublish (make draft)
  --archive                 Archive item
  --unarchive               Unarchive item

Examples:
  # Edit link title
  sct-hub edit abc123 --title "New Title"

  # Publish draft post
  sct-hub edit post456 --publish

  # Update post from file
  sct-hub edit post789 --file ~/updated-post.md
```

### Command: `remove`

Remove item (soft delete - archives).

```bash
sct-hub remove [ID] [OPTIONS]

Options:
  --force                   Hard delete (permanent)

Examples:
  # Archive link
  sct-hub remove abc123

  # Permanently delete
  sct-hub remove abc123 --force
```

### Command: `import`

Import content from external sources.

```bash
sct-hub import [SOURCE] [OPTIONS]

Sources:
  rss <URL|FILE>            Import from RSS feed
  opml <FILE>               Import feeds from OPML
  json <FILE>               Import from JSON file
  nostr <EVENT_ID>          Import Nostr event

Options:
  --category <CAT>          Assign category to imports
  --auto-sync               Set up auto-sync (RSS only)
  --limit <N>               Limit imported items
  --dry-run                 Preview without importing

Examples:
  # Import blog posts from RSS
  sct-hub import rss https://blog.example.com/feed.xml \
    --category "Blog" \
    --limit 10

  # Import using rss-wasm (composition)
  rss-wasm parse feed.xml --format json | \
    sct-hub import json - --category "Imported"

  # Auto-sync RSS feed
  sct-hub import rss https://blog.example.com/feed.xml \
    --auto-sync
```

### Command: `generate`

Generate static site.

```bash
sct-hub generate [OPTIONS]

Options:
  --output <DIR>            Output directory (default: ./site)
  --theme <THEME>           Theme to use
  --template <PATH>         Custom template directory
  --base-url <URL>          Base URL for links
  --minify                  Minify HTML/CSS/JS
  --clean                   Clean output directory first

Examples:
  # Generate site
  sct-hub generate --output ./public

  # Generate with custom theme
  sct-hub generate \
    --output ./site \
    --theme dark \
    --base-url "https://everythingsings.art" \
    --minify

  # Generate to temp for preview
  sct-hub generate --output /tmp/hub-preview
```

### Command: `preview`

Start local preview server.

```bash
sct-hub preview [OPTIONS]

Options:
  --port <PORT>             Port to serve on (default: 8080)
  --host <HOST>             Host to bind (default: 127.0.0.1)
  --watch                   Auto-regenerate on changes
  --open                    Open browser

Examples:
  # Start preview server
  sct-hub preview --port 3000 --open

  # Watch mode for development
  sct-hub preview --watch --port 8080
```

### Command: `publish`

Publish site and/or metadata.

```bash
sct-hub publish [TARGET] [OPTIONS]

Targets:
  nostr                     Publish metadata to Nostr
  github-pages              Deploy to GitHub Pages
  ipfs                      Publish to IPFS

Options:
  --message <MSG>           Commit message (GitHub Pages)
  --branch <BRANCH>         Target branch (default: gh-pages)
  --pow <DIFFICULTY>        Add PoW to Nostr event

Examples:
  # Publish metadata to Nostr
  sct-hub publish nostr --pow 20

  # Deploy to GitHub Pages
  sct-hub generate --output ./public && \
    sct-hub publish github-pages --message "Update site"

  # Using Plurcast for multi-platform
  sct-hub export metadata | \
    plurcast publish --platform nostr,mastodon
```

### Command: `config`

Manage site configuration.

```bash
sct-hub config [SUBCOMMAND]

Subcommands:
  get <KEY>                 Get config value
  set <KEY> <VALUE>         Set config value
  list                      List all config
  export                    Export config as JSON
  import <FILE>             Import config from JSON

Examples:
  # Set site title
  sct-hub config set site_title "My Hub"

  # Get site URL
  sct-hub config get site_url

  # List all config
  sct-hub config list
```

### Command: `themes`

Manage themes.

```bash
sct-hub themes [SUBCOMMAND]

Subcommands:
  list                      List installed themes
  install <NAME|URL>        Install theme
  preview <THEME>           Preview theme
  create <NAME>             Create custom theme

Examples:
  # List themes
  sct-hub themes list

  # Install theme
  sct-hub themes install minimal-dark

  # Preview theme
  sct-hub themes preview minimal-dark --open
```

---

## Static Site Generation

### Template Engine

**Handlebars-based templates** with partials and helpers.

**Template structure:**
```
templates/
  themes/
    minimal/
      layout.hbs           # Base layout
      index.hbs            # Homepage
      post.hbs             # Blog post page
      partials/
        header.hbs
        footer.hbs
        link-card.hbs
      static/
        style.css
        script.js
```

**Template data:**
```json
{
  "site": {
    "title": "Site Title",
    "description": "Site description",
    "url": "https://example.com",
    "author": {
      "name": "Author Name",
      "bio": "Bio text"
    },
    "social": {
      "nostr": "npub...",
      "mastodon": "@user@instance",
      "github": "username"
    }
  },
  "links": [
    {
      "id": "abc",
      "title": "Link Title",
      "url": "https://...",
      "description": "...",
      "category": "Projects",
      "icon": "🚀"
    }
  ],
  "posts": [
    {
      "id": "xyz",
      "title": "Post Title",
      "slug": "post-title",
      "content_html": "...",
      "published_at": 1731629200,
      "tags": ["rust", "wasm"]
    }
  ],
  "categories": {
    "Projects": [...],
    "Writing": [...]
  }
}
```

### Generated Structure

```
site/
  index.html              # Homepage with links
  posts/
    post-slug-1.html
    post-slug-2.html
  static/
    style.css
    script.js
    images/
  feed.xml                # RSS feed of your content
  feed.json               # JSON feed
  sitemap.xml
  robots.txt
```

### Themes

**Included themes:**
- `minimal` - Clean, simple, fast (default)
- `cards` - Card-based layout (like Linktree)
- `blog` - Blog-focused with featured posts
- `dark` - Dark mode variant

**Custom themes:** Users can create custom themes with their own Handlebars templates.

---

## Integration Patterns

### With rss-wasm (Auto-Import Blog Posts)

Import your blog posts as content automatically:

```bash
# One-time import
rss-wasm parse https://blog.example.com/feed.xml --format json | \
  sct-hub import json - --category "Blog"

# Set up auto-sync (cron job)
0 */6 * * * rss-wasm parse ~/feeds.opml | sct-hub import json - --category "Blog"

# Generate site with imported posts
sct-hub generate --output ./public
```

### With Plurcast (Publish Updates)

Announce site updates to Nostr/Mastodon:

```bash
# Generate site and publish announcement
sct-hub generate --output ./public

# Export latest post and publish
sct-hub list posts --limit 1 --format json | \
  jq -r '.[0] | "New post: \(.title)\n\(.summary)\n\nhttps://site.com/posts/\(.slug)"' | \
  plurcast publish --platform nostr,mastodon --pow 20
```

### With sct-npow (Nostr Content Import)

Import high-quality Nostr content:

```bash
# Import high-PoW Nostr events as posts
sct-npow fetch \
  --author <your_pubkey> \
  --min-pow 20 \
  --kind 30023 \
  --format json | \
  sct-hub import nostr - --category "Nostr Posts"
```

### Complete Workflow

```bash
#!/bin/bash
# Complete content hub workflow

# 1. Import blog posts from RSS
rss-wasm parse ~/feeds.opml --format json | \
  sct-hub import json - --category "Blog"

# 2. Import your own Nostr long-form content
sct-npow fetch \
  --author <your_npub> \
  --kind 30023 \
  --format json | \
  sct-hub import nostr - --category "Essays"

# 3. Generate static site
sct-hub generate \
  --output ./public \
  --theme minimal \
  --minify

# 4. Deploy to GitHub Pages
cd ./public && \
  git add . && \
  git commit -m "Update site $(date +%Y-%m-%d)" && \
  git push origin gh-pages

# 5. Announce update to Nostr
echo "Site updated! Check out my latest posts at https://everythingsings.art" | \
  plurcast publish --platform nostr --pow 18
```

---

## Configuration

### Config File Location

`~/.config/sct/sct-hub.toml`

### Example Configuration

```toml
[site]
title = "EverythingSings.Art"
description = "Sovereign Composable Tools & Personal Projects"
url = "https://everythingsings.art"
theme = "minimal"

[author]
name = "EverythingSings"
bio = "Building sovereign composable tools for humans and agents"

[social]
nostr = "npub..."
mastodon = "@everythingsings@mastodon.social"
github = "EverythingSings"

[storage]
database = "~/.local/share/sct/sct.db"

[generation]
output_dir = "./public"
minify = true
base_url = "https://everythingsings.art"

[imports]
# Auto-sync RSS feeds
auto_sync_enabled = true
sync_interval = "6h"

[[imports.rss_feeds]]
url = "https://blog.example.com/feed.xml"
category = "Blog"

[publishing]
# Nostr publishing settings
nostr_relay = "wss://relay.damus.io"
nostr_pow = 18

# GitHub Pages
github_repo = "EverythingSings/EverythingSings.github.io"
github_branch = "gh-pages"
```

---

## Implementation Details

### Dependencies

**Core (Layer 1):**
- `pulldown-cmark` - Markdown parsing
- `url` - URL parsing and validation
- `serde` - Serialization

**Network (Layer 2):**
- HTTP client (WASI-compatible) for metadata fetching
- Optional Nostr SDK integration

**Storage (Layer 3):**
- `rusqlite` - SQLite bindings
- `serde_json` - JSON serialization

**Application (Layer 4):**
- `clap` - CLI argument parsing
- `handlebars` - Template engine
- `minify-html` - HTML minification
- `chrono` - Date/time handling

### WASM Target

**Primary target:** `wasm32-wasip2`
- WASI 0.2 (Preview 2)
- WebAssembly Component Model
- Requires Rust 1.82+ (Tier 2 support)

**Build command:**
```bash
cargo build --release --target wasm32-wasip2
```

### Feature Flags

```toml
[features]
default = ["storage", "cli", "templates"]

# Layer 2: Network features
network = ["dep:reqwest", "metadata-fetch"]
metadata-fetch = ["network"]

# Layer 3: Storage
storage = ["dep:rusqlite"]

# Layer 4: CLI & generation
cli = ["dep:clap"]
templates = ["dep:handlebars"]
preview-server = ["dep:tiny-http"]

# All features
full = ["network", "storage", "cli", "templates", "preview-server"]

# Minimal (just core + storage)
minimal = ["storage"]
```

---

## GitHub Pages Deployment

### Setup

1. **Create GitHub repository** for your site
2. **Configure sct-hub** with repo details
3. **Generate and push** to gh-pages branch

### Automated Deployment

**GitHub Actions workflow** (`.github/workflows/deploy.yml`):

```yaml
name: Deploy SCT Hub

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 */6 * * *'  # Auto-sync every 6 hours
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install sct-hub
        run: |
          # TODO: Install from release or build from source
          cargo install --git https://github.com/EverythingSings/sct-hub

      - name: Import RSS feeds
        run: |
          rss-wasm parse feeds.opml --format json | \
            sct-hub import json - --category "Blog"

      - name: Generate site
        run: |
          sct-hub generate \
            --output ./public \
            --base-url "https://everythingsings.art" \
            --minify

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          cname: everythingsings.art
```

### Custom Domain

1. Add `CNAME` file to output: `echo "everythingsings.art" > public/CNAME`
2. Configure DNS: `A` record to GitHub Pages IPs or `CNAME` to `<username>.github.io`
3. Enable HTTPS in GitHub repo settings

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

## Success Metrics

### For Personal Use

- Replaces Linktree with sovereign alternative
- Easy to add/manage links and content
- Beautiful, customizable static sites
- Quick deployment (< 1 minute to update)

### For SCT Ecosystem

- Demonstrates all five axioms in practice
- Composes seamlessly with existing tools
- Provides real-world value (not just demo)
- Attracts users to SCT philosophy

### Technical

- Static site generation < 1 second
- WASM binary < 3MB
- Works completely offline
- Zero dependencies for generated sites

---

## Open Questions

### Design

1. **Theme system:** Custom themes vs theme marketplace?
   - **Decision:** Start with bundled themes, document custom theme API

2. **Import strategy:** Auto-sync vs manual import?
   - **Decision:** Support both, auto-sync opt-in

3. **Content model:** Flexible schema vs strict types?
   - **Decision:** Start strict (links, posts), extend later

### Technical

1. **Template engine:** Handlebars vs Tera vs custom?
   - **Leading:** Handlebars (popular, good WASM support)

2. **Preview server:** Built-in vs external tool?
   - **Decision:** Built-in for convenience

3. **Deployment:** GitHub Actions vs manual vs both?
   - **Decision:** Both - provide Actions template, support manual

---

## Contributing

**Status:** Not yet open source (design phase)

Once stable, we'll welcome contributions for:
- Theme designs
- Integration examples
- Template improvements
- Documentation
- Deployment guides

---

## License

TBD (will be open source upon stable release)

---

## References

- **SCT Philosophy:** /README.md
- **sct-npow Architecture:** /docs/sct-npow-spec.md
- **Integration Patterns:** /sct-npow/research-integration.md
- **GitHub Pages:** https://pages.github.com
- **Handlebars:** https://handlebarsjs.com

---

**sct-hub** - Your content, your data, your site. No platform required.

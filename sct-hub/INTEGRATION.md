# sct-hub: SCT Ecosystem Integration

**Date:** 2025-11-15
**Purpose:** Design how sct-hub composes with existing SCT infrastructure

---

## The Complete SCT Ecosystem (Updated)

```
┌─────────────────────────────────────────────────────────────────┐
│                         INPUT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  rss-wasm          │  Parse RSS/Atom/JSON feeds                 │
│  sct-npow          │  Read Nostr events (PoW-focused)           │
├─────────────────────────────────────────────────────────────────┤
│                      INTELLIGENCE LAYER                          │
├─────────────────────────────────────────────────────────────────┤
│  sct-vec           │  Vector search, semantic RAG (Q1 2026)     │
├─────────────────────────────────────────────────────────────────┤
│                      APPLICATION LAYER                           │
├─────────────────────────────────────────────────────────────────┤
│  sct-hub           │  Personal content hub + site generator     │
├─────────────────────────────────────────────────────────────────┤
│                        OUTPUT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  Plurcast          │  Post to Nostr/Mastodon/SSB                │
└─────────────────────────────────────────────────────────────────┘

              All tools share: SQLite storage + JSON I/O
```

**sct-hub's unique role:**
- **Aggregates** content from INPUT layer (rss-wasm, sct-npow)
- **Presents** content via static sites (APPLICATION layer)
- **Publishes** updates via OUTPUT layer (Plurcast)
- **Demonstrates** complete SCT workflow (dogfooding)

---

## Integration Pattern 1: INPUT → sct-hub

### rss-wasm → sct-hub (Blog Aggregation)

**Use Case:** Auto-import your blog posts as content on your hub

```bash
# One-time import of recent posts
rss-wasm parse https://blog.example.com/feed.xml \
  --format json | \
  sct-hub import json - \
    --category "Blog" \
    --limit 10

# Import from OPML (multiple feeds)
rss-wasm parse ~/feeds.opml --format json | \
  sct-hub import json - --category "Writing"

# Scheduled sync (cron)
0 */6 * * * rss-wasm parse ~/feeds.opml | \
  sct-hub import json - && \
  sct-hub generate --output ./public
```

**Data Flow:**
1. rss-wasm fetches and parses RSS/Atom feeds
2. Outputs standardized JSON format
3. sct-hub imports as `hub_content` table entries
4. Site generation includes imported posts
5. Posts appear alongside manual links

**Schema Mapping:**
```json
// rss-wasm output
{
  "id": "item-guid",
  "title": "Post Title",
  "content": "<p>HTML content</p>",
  "link": "https://blog.example.com/post",
  "published": 1731629200,
  "author": "Author Name",
  "categories": ["tech", "rust"]
}

// sct-hub import creates
hub_content {
  id: "item-guid",
  title: "Post Title",
  slug: "post-title",
  content: "HTML content (or converted to markdown)",
  published_at: 1731629200,
  category: "Blog",
  tags: ["tech", "rust"],
  source: "rss",
  source_id: "https://blog.example.com/feed.xml"
}
```

### sct-npow → sct-hub (Nostr Content)

**Use Case:** Import your own high-quality Nostr posts to your site

```bash
# Import your long-form Nostr articles
sct-npow fetch \
  --author <your_npub> \
  --kind 30023 \
  --min-pow 15 \
  --format json | \
  sct-hub import nostr - --category "Essays"

# Import curated high-PoW content
sct-npow fetch \
  --relay wss://relay.damus.io \
  --min-pow 22 \
  --kind 1,30023 \
  --since 7d \
  --format json | \
  sct-hub import nostr - \
    --category "Curated" \
    --tag "high-quality"
```

**Use Cases:**
- **Personal archive:** Your Nostr posts → your website
- **Content curation:** High-PoW content → featured section
- **Cross-platform presence:** Nostr + website = wider reach

**Schema Mapping:**
```json
// sct-npow output
{
  "id": "event_id_hex",
  "pubkey": "author_pubkey",
  "created_at": 1731629200,
  "kind": 30023,
  "content": "# Long-form article\n\nMarkdown content...",
  "pow_difficulty": 22,
  "tags": [
    ["d", "article-slug"],
    ["title", "Article Title"],
    ["summary", "Brief summary"],
    ["t", "rust"],
    ["t", "nostr"]
  ]
}

// sct-hub import creates
hub_content {
  id: "event_id_hex",
  title: "Article Title",
  slug: "article-slug",
  content: "# Long-form article...",
  summary: "Brief summary",
  published_at: 1731629200,
  category: "Essays",
  tags: ["rust", "nostr"],
  source: "nostr",
  source_id: "event_id_hex",
  metadata: {"pow_difficulty": 22, "pubkey": "..."}
}
```

---

## Integration Pattern 2: sct-hub → OUTPUT

### sct-hub → Plurcast (Site Updates)

**Use Case:** Announce site updates to Nostr/Mastodon

```bash
# After generating site, announce update
sct-hub generate --output ./public && \
  echo "Just updated my site! Check out my latest projects and posts at https://everythingsings.art 🚀" | \
  plurcast publish --platform nostr,mastodon --pow 18

# Announce new blog post import
NEW_POST=$(sct-hub list posts --limit 1 --format json | \
  jq -r '.[0] | "New post: \(.title)\n\n\(.summary)\n\nRead more: https://everythingsings.art/posts/\(.slug)"')

echo "$NEW_POST" | \
  plurcast publish --platform nostr --pow 20

# Auto-announcement workflow
sct-hub import rss feed.xml && \
  sct-hub generate --output ./public && \
  sct-hub publish nostr  # Built-in Plurcast integration
```

**Announcement Templates:**

**New link added:**
```
🔗 New link in my hub: [Title]

[Description]

Check it out: https://everythingsings.art#[category]
```

**New blog post:**
```
📝 New post: [Title]

[Summary]

Read: https://everythingsings.art/posts/[slug]

#blog #[tags]
```

**Site redesign:**
```
✨ Just redesigned my personal hub!

New theme, better organization, and fresh content.

Built with sct-hub - sovereign, composable, no Linktree needed.

https://everythingsings.art
```

### sct-hub → Nostr (Direct Publishing)

**Use Case:** Publish site metadata as Nostr events

```bash
# Publish site profile (NIP-05 style metadata)
sct-hub export profile --format nostr | \
  plurcast publish --platform nostr --kind 0

# Publish link collection as structured event
sct-hub export links --format nostr --category "Projects" | \
  plurcast publish --platform nostr --kind 30001

# Future: Native Nostr publishing in sct-hub
sct-hub publish nostr \
  --type profile \
  --pow 20
```

**Nostr Event Types:**
- **Kind 0:** Profile metadata (name, bio, picture, website)
- **Kind 30001:** Categorized bookmark list (your links)
- **Kind 30023:** Long-form content (your blog posts)

---

## Integration Pattern 3: Complete Workflow

### Personal Content Pipeline

**Scenario:** Content creator with blog, Nostr presence, and personal site

```bash
#!/bin/bash
# personal-content-pipeline.sh

set -e

echo "=== Personal Content Pipeline ==="

# 1. INPUT: Gather content from sources
echo "[1/5] Importing content..."

# Import blog posts via RSS
rss-wasm parse ~/blog-feeds.opml --format json | \
  sct-hub import json - --category "Blog"

# Import your own Nostr long-form posts
sct-npow fetch \
  --author $MY_NOSTR_PUBKEY \
  --kind 30023 \
  --since 30d \
  --format json | \
  sct-hub import nostr - --category "Nostr Essays"

# 2. APPLICATION: Generate site
echo "[2/5] Generating static site..."
sct-hub generate \
  --output ./public \
  --base-url "https://everythingsings.art" \
  --theme minimal \
  --minify

# 3. OUTPUT: Deploy site
echo "[3/5] Deploying to GitHub Pages..."
cd ./public
git add .
git commit -m "Auto-update: $(date +%Y-%m-%d)" || echo "No changes"
git push origin gh-pages
cd ..

# 4. OUTPUT: Announce updates
echo "[4/5] Announcing updates..."

NEW_POSTS=$(sct-hub list posts \
  --since 24h \
  --format json | \
  jq 'length')

if [ "$NEW_POSTS" -gt 0 ]; then
  ANNOUNCEMENT="Site updated with $NEW_POSTS new posts! Check them out at https://everythingsings.art"
  echo "$ANNOUNCEMENT" | \
    plurcast publish --platform nostr,mastodon --pow 18
fi

# 5. BACKUP: Archive to local storage
echo "[5/5] Backing up data..."
sct-hub export all --format json > ~/backups/hub-$(date +%Y%m%d).json

echo "=== Pipeline complete! ==="
```

**Cron schedule:**
```cron
# Run every 6 hours
0 */6 * * * ~/scripts/personal-content-pipeline.sh >> ~/logs/content-pipeline.log 2>&1
```

---

## Integration Pattern 4: Shared Storage

### SQLite as Common Infrastructure

All SCT tools write to `~/.local/share/sct/sct.db`:

**Cross-tool queries:**

```sql
-- All content sources in one view
CREATE VIEW all_content AS
SELECT
  'rss' as source,
  id,
  title,
  link as url,
  published as created_at,
  NULL as pow_difficulty
FROM rss_items
UNION ALL
SELECT
  'nostr' as source,
  id,
  content as title,
  NULL as url,
  created_at,
  pow_difficulty
FROM nostr_events
WHERE kind IN (1, 30023)
UNION ALL
SELECT
  'hub' as source,
  id,
  title,
  NULL as url,
  published_at as created_at,
  NULL as pow_difficulty
FROM hub_content;

-- Quality metrics across sources
SELECT
  source,
  COUNT(*) as count,
  AVG(CASE WHEN pow_difficulty IS NOT NULL THEN pow_difficulty ELSE 0 END) as avg_pow
FROM all_content
GROUP BY source;
```

**Cross-tool automation:**

```bash
# Find high-quality Nostr content and feature on hub
sqlite3 ~/.local/share/sct/sct.db "
  SELECT id, content FROM nostr_events
  WHERE pow_difficulty >= 22
    AND kind = 1
    AND created_at > strftime('%s', 'now', '-7 days')
  ORDER BY pow_difficulty DESC
  LIMIT 5
" | while read event_id content; do
  sct-hub add link \
    --title "$content" \
    --url "https://njump.me/$event_id" \
    --category "High-PoW Nostr" \
    --icon "⚡"
done
```

---

## Integration Pattern 5: Theme as Data Consumer

### Templates Pull From Multiple Sources

**Handlebars template data includes all sources:**

```handlebars
{{!-- index.hbs --}}
<main>
  {{!-- Manual links --}}
  <section class="links">
    <h2>Projects & Links</h2>
    {{#each links}}
      {{> link-card this}}
    {{/each}}
  </section>

  {{!-- Imported blog posts (via rss-wasm) --}}
  <section class="blog">
    <h2>Latest Blog Posts</h2>
    {{#each (filter posts 'source' 'rss')}}
      {{> post-card this}}
    {{/each}}
  </section>

  {{!-- Nostr essays (via sct-npow) --}}
  <section class="essays">
    <h2>Essays on Nostr</h2>
    {{#each (filter posts 'source' 'nostr')}}
      <article>
        <h3>{{this.title}}</h3>
        <span class="pow-badge">⚡ PoW: {{this.metadata.pow_difficulty}}</span>
        <p>{{this.summary}}</p>
        <a href="/posts/{{this.slug}}">Read more →</a>
      </article>
    {{/each}}
  </section>

  {{!-- Featured content from sct-vec (future) --}}
  <section class="featured">
    <h2>Recommended Reading</h2>
    {{!-- Query sct-vec for semantically similar content --}}
  </section>
</main>
```

**Template data structure:**

```json
{
  "site": {...},
  "links": [
    {"id": "...", "title": "...", "source": "manual"},
    {"id": "...", "title": "...", "source": "manual"}
  ],
  "posts": [
    {
      "id": "...",
      "title": "Blog Post",
      "source": "rss",
      "source_id": "https://blog.example.com/feed.xml"
    },
    {
      "id": "...",
      "title": "Nostr Essay",
      "source": "nostr",
      "source_id": "event_hex_id",
      "metadata": {"pow_difficulty": 22}
    }
  ],
  "categories": {...}
}
```

---

## Integration Pattern 6: Multi-Agent Workflows

### Agent-Assisted Content Curation

**Use Case:** Agent monitors sources, curates quality content, updates hub

```bash
#!/bin/bash
# agent-content-curator.sh

# 1. Monitor multiple sources
echo "Gathering content from sources..."

# Fetch high-PoW Nostr content
NOSTR_CONTENT=$(sct-npow fetch \
  --min-pow 20 \
  --kind 1,30023 \
  --since 24h \
  --search "rust|wasm|sct" \
  --format json)

# Fetch RSS updates
RSS_CONTENT=$(rss-wasm parse ~/curated-feeds.opml \
  --since 24h \
  --format json)

# 2. Analyze with sct-vec (future)
# Find content relevant to your interests
echo "$NOSTR_CONTENT" | \
  sct-vec index --collection nostr-temp

echo "$RSS_CONTENT" | \
  sct-vec index --collection rss-temp

# Query for relevant content
RELEVANT=$(echo "sovereign computing wasm rust" | \
  sct-vec query \
    --collections nostr-temp,rss-temp \
    --k 10 \
    --min-score 0.8 \
    --format json)

# 3. Agent decision: Import to hub?
echo "$RELEVANT" | \
  agent-decide-import | \
  while read -r item; do
    SOURCE=$(echo "$item" | jq -r '.source')

    if [ "$SOURCE" = "nostr" ]; then
      sct-hub add link \
        --title "$(echo "$item" | jq -r '.title')" \
        --url "$(echo "$item" | jq -r '.url')" \
        --category "Curated" \
        --icon "⚡"
    else
      sct-hub import json - --category "Curated" <<< "$item"
    fi
  done

# 4. Regenerate and deploy
sct-hub generate --output ./public
sct-hub publish github-pages

# 5. Announce curation
echo "I just updated my curated links with interesting content on sovereign computing and WASM. Check it out! https://everythingsings.art" | \
  plurcast publish --platform nostr --pow 18
```

**Agent capabilities:**
- Monitor INPUT sources (rss-wasm, sct-npow)
- Analyze with INTELLIGENCE layer (sct-vec)
- Curate to APPLICATION layer (sct-hub)
- Publish via OUTPUT layer (Plurcast)

---

## Integration Pattern 7: Dogfooding Example

### EverythingSings.Art - Complete SCT Stack

**Real-world deployment demonstrating all SCT principles:**

**Content sources:**
1. **Manual links:** Projects, tools, social profiles
2. **Blog posts:** Via rss-wasm from personal blog
3. **Nostr essays:** Via sct-npow (long-form NIP-23)
4. **Curated links:** High-PoW Nostr content

**Site structure:**
```
https://everythingsings.art/
├── /                        # Homepage: links + featured content
├── /projects/               # SCT tools (Plurcast, rss-wasm, sct-npow)
├── /writing/                # Blog posts + Nostr essays
├── /about/                  # Bio, philosophy, contact
└── /feed.xml                # RSS feed of all content
```

**Deployment workflow:**
```yaml
# .github/workflows/deploy-hub.yml
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours

jobs:
  deploy:
    - Import blog posts (rss-wasm)
    - Import Nostr essays (sct-npow)
    - Generate site (sct-hub)
    - Deploy to GitHub Pages
    - Announce updates (Plurcast)
```

**Message to visitors:**
> "This site is built with the same SCT tools I'm developing. No Linktree. No platform lock-in. Just sovereign, composable tools that work together. The links live in my local SQLite database, the site generates offline, and I can deploy anywhere. This is what intentional, user-owned computing looks like."

---

## Future Integration: sct-vec

### Semantic Search for Your Hub

**When sct-vec is released (Q1 2026):**

```bash
# Index all your content
sct-hub export all --format json | \
  sct-vec index --collection my-hub

# Semantic search across everything
echo "rust wasm tools" | \
  sct-vec query --collection my-hub --k 5

# Related content suggestions
sct-hub generate \
  --output ./public \
  --enable-related-content \
  --sct-vec-collection my-hub
```

**Template integration:**
```handlebars
{{!-- post.hbs --}}
<article>
  <h1>{{post.title}}</h1>
  <div>{{post.content_html}}</div>

  {{!-- Related posts via sct-vec --}}
  <section class="related">
    <h2>Related Content</h2>
    {{#each (sct_vec_similar post.id 5)}}
      <a href="{{this.url}}">{{this.title}}</a>
    {{/each}}
  </section>
</article>
```

---

## Output Format Standardization

### JSON Export for Composition

**sct-hub exports in SCT-standard format:**

```bash
# Export all content as JSON
sct-hub export all --format json

# Output format (compatible with other SCT tools)
{
  "version": "sct-hub-v1",
  "site": {
    "title": "...",
    "url": "...",
    "author": {...}
  },
  "items": [
    {
      "type": "link",
      "id": "...",
      "title": "...",
      "url": "...",
      "category": "...",
      "source": "manual",
      "created_at": 1731629200
    },
    {
      "type": "post",
      "id": "...",
      "title": "...",
      "content": "...",
      "slug": "...",
      "source": "rss",
      "source_id": "feed_url",
      "published_at": 1731629200,
      "tags": ["..."]
    }
  ]
}
```

**Composability:**
```bash
# Export and transform
sct-hub export all | \
  jq '.items[] | select(.source == "nostr")' | \
  other-tool process

# Merge multiple hubs
cat hub1.json hub2.json | \
  jq -s 'add' | \
  sct-hub import json -
```

---

## Summary: Why Integration Matters

### The SCT Value Proposition

**Without integration (traditional tools):**
- Linktree: Links isolated in their platform
- Blog: Posts isolated in WordPress
- Social: Content isolated in Twitter/Instagram
- No composition, no local control, no sovereignty

**With SCT integration:**
- **INPUT:** rss-wasm, sct-npow gather from open sources
- **INTELLIGENCE:** sct-vec analyzes and connects (future)
- **APPLICATION:** sct-hub aggregates and presents
- **OUTPUT:** Plurcast publishes to decentralized platforms

**Result:**
- ✅ All content in local SQLite database
- ✅ Tools compose via JSON pipes
- ✅ Site generation completely offline
- ✅ Deploy anywhere (GitHub, IPFS, self-hosted)
- ✅ Publish to Nostr/Mastodon with one command
- ✅ Regenerate entire site from local data anytime

**The dogfooding story:**
> "I replaced Linktree with sct-hub, which pulls my blog posts via rss-wasm, imports my Nostr essays via sct-npow, generates a static site I control, and announces updates via Plurcast. Every tool composes through SQLite and JSON. No platforms. No lock-in. Just sovereign computing."

---

## Next Steps

1. **Build sct-hub Phase 1:** Core + storage + basic generation
2. **Test integration:** rss-wasm → sct-hub workflow
3. **Deploy EverythingSings.Art:** Real-world dogfooding
4. **Document patterns:** Blog posts showing integration
5. **Iterate:** Add sct-npow integration when ready
6. **Plan sct-vec:** Semantic layer for future

**The goal:** Prove the SCT vision works in practice, not just theory. When people ask "why should I care about sovereign tools?", you point to your site and say "because I built this, and you can too."

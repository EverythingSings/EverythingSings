# SCT Tools - Usage Examples

Practical examples for composing Sovereign Composable Tools.

## Plurcast Examples

### Basic Posting
```bash
# Post to multiple platforms
echo "Hello decentralized world!" | plur-post

# Pipe composition
cat draft.txt | sed 's/foo/bar/g' | plur-post --platform nostr,mastodon

# Query history with semantic filtering
plur-history --since=7d --format=json | jq '.[] | select(.platform == "nostr")'
```

## rss-wasm Examples

### Basic Parsing
```bash
# Parse any feed format (human or agent)
curl https://blog.com/feed.xml | rss-wasm parse

# Filter and query
rss-wasm parse feed.xml | rss-wasm filter --since=24h | jq '.items[].title'
```

### Multi-Agent Workflows
```bash
# Monitor AI announcements
rss-wasm parse openai-feed.xml | agent analyze-impact | agent draft-response

# Archive for historical context
rss-wasm parse feed.xml | vectordb index --collection=ai-news
```

## sct-npow Examples (Experimental)

### PoW-Based Filtering
```bash
# Fetch high-PoW events from Nostr relays
sct-npow fetch --relay wss://relay.damus.io --min-pow 20 --since 24h

# Pipe to vector search
sct-npow fetch --min-pow 20 | sct-vec index --collection nostr-pow
```

### Cross-Platform Workflows
```bash
# Merge with RSS for unified intentional feed
(rss-wasm parse feeds.opml && sct-npow fetch --min-pow 20) | \
  jq -s 'sort_by(.created_at) | reverse'

# Amplify quality content cross-platform
sct-npow fetch --min-pow 25 --limit 1 | plurcast publish --platform mastodon
```

## sct-vec Examples (Planned)

### Multi-Source Intelligence
```bash
# Monitor and respond to AI announcements (multi-agent workflow)
# Combine RSS (official blogs) + Nostr (community commentary)
(rss-wasm parse openai-feed.xml anthropic-feed.xml && \
 sct-npow fetch --min-pow 20 --search "llm|ai") | \
  sct-vec index --collection=ai-news

# Query historical context and draft response
sct-vec query "previous model releases" --collection=ai-news --k=20 | \
  agent analyze-trends | \
  agent draft-commentary | \
  plurcast publish --platform nostr,mastodon

# Semantic search across all archived content
echo "rust wasm performance" | sct-vec query --collection=feeds --k=10
```

## Backup and Redundancy

### Local Backups
```bash
# Backup SQLite databases with rsync
rsync -av ~/.local/share/sct/ /mnt/external-drive/sct-backup/

# Or to local NAS you control
rsync -av ~/.local/share/sct/ nas.local:/backups/sct/
```

### Encrypted Cloud Backup
```bash
# Encrypt locally, upload anywhere
tar czf - ~/.local/share/sct/ | \
  gpg --encrypt --recipient you@domain.com | \
  aws s3 cp - s3://bucket/backup.tar.gz.gpg

# Using restic
restic backup ~/.local/share/sct/

# Using borg
borg create backup-repo::snapshot ~/.local/share/sct/
```

### Export and Import
```bash
# Export from SCT storage to standard formats
plur-history --format=json > all-posts.json
plur-history --format=csv > all-posts.csv
rss-wasm export feeds.opml

# Import from anywhere
cat external-posts.json | plur-import

# Direct SQLite access
sqlite3 ~/.local/share/sct/posts.db "SELECT * FROM posts"
```

## Cross-Tool Composition

### Agent Monitoring Pipeline
```bash
# Token-efficient monitoring (2K vs 50K+ tokens for HTML)
rss-wasm parse openai-feed.xml anthropic-feed.xml | \
  agent analyze-for-breaking-changes | \
  plur-post --platform nostr
```

### Quality Content Amplification
```bash
# Find high-effort Nostr content, republish to Mastodon
sct-npow fetch --min-pow 25 --limit 5 | \
  jq '.[] | .content' | \
  plur-post --platform mastodon
```

### Unified Intentional Feed
```bash
# Merge all intentional sources
(rss-wasm parse blogs.opml && \
 rss-wasm parse podcasts.opml && \
 sct-npow fetch --min-pow 20) | \
  jq -s 'sort_by(.published) | reverse | .[0:50]'
```

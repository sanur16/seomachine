# Batch Articles Command

Process the next N articles from the programmatic SEO queue. Runs the full research → write → optimize → finalize pipeline for each article.

## Usage
`/batch-articles [count]`

**Examples:**
- `/batch-articles 3` - Process next 3 queued articles
- `/batch-articles 1` - Process the next queued article
- `/batch-articles` - Defaults to processing the next 1 article

## What This Command Does
1. Reads the article queue from `seo/data/article-queue.json`
2. Selects the next N articles with status `queued` (ordered by priority, then ID)
3. For each article, runs the full `/create` pipeline
4. Updates the queue file with status changes and metadata

## Process

### Step 1: Read the Queue

Read `seo/data/article-queue.json` and identify the next N articles where `status` is `queued`, ordered by:
1. `priority` (ascending — priority 1 first)
2. `id` (ascending — earliest added first)

Display the batch plan:
```
Batch Processing: [N] articles

1. [title] (Priority [X], ID [Y])
   Keyword: [primary_keyword]
   Cluster: [cluster]
   Target: [target_word_count] words

2. [title] ...
```

### Step 2: Process Each Article

For each article in the batch, execute the full `/create` command workflow:

1. **Research Phase**
   - Check if a research brief already exists in `seo/research/` for this topic
   - If not, run the `/research` workflow for the article's `primary_keyword`
   - Update queue entry: `status` → `researched`, add `researched_date`

2. **Write Phase**
   - Run the full `/create` workflow (write → optimize → finalize)
   - The article will end up in either `seo/output/` (if quality score ≥ 70) or `seo/review-required/` (if below threshold)
   - Update queue entry: `status` → `drafted`, add `drafted_date`, `file`, `quality_score`

3. **Archive Supporting Files**
   - Move agent reports and analysis files from `seo/drafts/` to `seo/archive/[slug]/`
   - Keep `seo/drafts/` clean for in-progress work only

### Step 3: Update the Queue

After each article is processed, update `seo/data/article-queue.json`:

```json
{
  "id": 1,
  "status": "drafted",
  "researched_date": "YYYY-MM-DD",
  "drafted_date": "YYYY-MM-DD",
  "file": "seo/output/[slug]-[YYYY-MM-DD].md",
  "quality_score": 85,
  ...existing fields...
}
```

If the article fails the quality gate:
```json
{
  "status": "drafted",
  "file": "seo/review-required/[slug]-[YYYY-MM-DD].md",
  "quality_score": 62,
  "review_notes": "Failed quality threshold after 2 revision attempts"
}
```

### Step 4: Batch Summary

After all articles are processed, display a summary:

```
Batch Complete: [N] articles processed

✓ [title] → seo/output/[filename].md (Score: 85)
✓ [title] → seo/output/[filename].md (Score: 91)
✗ [title] → seo/review-required/[filename].md (Score: 62, needs review)

Queue Status:
  Queued: [remaining]
  Researched: [count]
  Drafted: [count]
  Published: [count]
```

## Queue File Structure

The queue file at `seo/data/article-queue.json` contains:

```json
{
  "metadata": {
    "description": "...",
    "status_values": ["queued", "researched", "drafted", "published", "skipped"]
  },
  "published": [...],
  "queue": [
    {
      "id": 1,
      "priority": 1,
      "status": "queued",
      "slug": "article-slug",
      "title": "Article Title",
      "primary_keyword": "target keyword",
      "secondary_keywords": ["keyword 2", "keyword 3"],
      "cluster": "topic-cluster",
      "article_type": "how-to-guide",
      "target_word_count": 2500,
      "search_intent": "informational",
      "competition": "medium",
      "campwatch_fit": "high",
      "notes": "Context for writing"
    }
  ]
}
```

## Status Lifecycle

```
queued → researched → drafted → published
                ↘ skipped (manual)
```

- **queued**: Ready to process
- **researched**: Research brief completed, not yet written
- **drafted**: Article written and in `seo/output/` or `seo/review-required/`
- **published**: Live on WordPress (updated via `/publish-draft`)
- **skipped**: Manually removed from pipeline

## Moving to Published

After using `/publish-draft` on an article from `seo/output/`:
1. Move the article from `seo/output/` to `seo/published/`
2. Update the queue entry: `status` → `published`, add `published_date`
3. Move the entry from `queue` array to `published` array in the JSON

## Important Notes

- Each article in the batch gets the full `/create` treatment — no shortcuts
- If an article fails the quality gate, it still counts as processed but goes to `seo/review-required/`
- The queue file is the single source of truth for pipeline status
- Articles use the `slug`, `primary_keyword`, `secondary_keywords`, and `notes` from their queue entry to guide research and writing
- Pass queue metadata (article_type, cluster, campwatch_fit, etc.) through to the article frontmatter

## Example

```
/batch-articles 3
```

This will:
1. Read the queue and find the next 3 `queued` articles by priority
2. For each: research → write → optimize → scrub → score → finalize
3. Save finished articles to `seo/output/`
4. Update `seo/data/article-queue.json` with results
5. Display batch summary

# Create Command

Use this command to write, optimize, and finalize an article in one step. Combines `/write` and `/optimize` into a single end-to-end workflow, then moves the finished article to `seo/output/`.

## Usage
`/create [topic or research brief]`

## What This Command Does
1. Writes a complete SEO-optimized article (full `/write` workflow)
2. Applies optimization fixes from `/optimize` directly to the article
3. Scrubs the final version for AI watermarks
4. Moves the finished article to `seo/output/`

## Process

### Phase 1: Write the Article

Execute the full `/write` command workflow for the given topic:

1. **Pre-Writing Review** — Read all context files (brand voice, style guide, SEO guidelines, target keywords, writing examples, internal links map, features)
2. **Draft the Article** — Create the full article following all `/write` content structure rules (hook, APP formula, mini-stories, contextual CTAs, keyword placement, internal/external links, meta elements)
3. **Save to Drafts** — Save as `seo/drafts/[topic-slug]-[YYYY-MM-DD].md`
4. **Scrub** — Run `/scrub` on the saved file to remove AI watermarks and em-dash overuse
5. **Quality Score Loop** — Run `python3 seo/data_sources/modules/content_scorer.py` on the draft. If score < 70, apply priority fixes and re-score (up to 2 iterations). If still < 70 after 2 iterations, save to `seo/review-required/` with review notes and STOP (do not continue to Phase 2)
6. **Run Agents** — Execute the 5 optimization agents (content-analyzer, seo-optimizer, meta-creator, internal-linker, keyword-mapper) and save their reports to `seo/drafts/`

### Phase 2: Optimize the Article

Using the agent reports generated in Phase 1, perform the full `/optimize` workflow:

1. **Keyword Audit** — Check keyword density (1-2%), placement in H1, first 100 words, H2s, meta elements, and URL slug. Fix any gaps
2. **Heading Structure** — Validate H1/H2/H3 hierarchy, ensure 2-3 H2s contain keyword variations. Fix any issues
3. **Content Quality** — Check word count (2000+ min), paragraph length (2-4 sentences max), sentence variety, readability (8th-10th grade), active voice. Fix any issues
4. **Link Optimization** — Verify 3-5+ internal links (cross-check against seo/context/internal-links-map.md) and 2-3+ external authority links. Add any missing links
5. **Meta Optimization** — Refine meta title (50-60 chars), meta description (150-160 chars), and URL slug. Apply the best options directly to the article frontmatter using the snake_case YAML fields `meta_title`, `meta_description`, and `url_slug`
6. **Featured Snippet Optimization** — Structure content to capture featured snippets where applicable (definition boxes, numbered lists, tables)
7. **Brand & Voice Check** — Verify alignment with seo/context/brand-voice.md and seo/context/style-guide.md

**IMPORTANT**: In Phase 2, do NOT just generate a report. Apply all fixes and improvements directly to the article file. The goal is a publish-ready article, not a list of suggestions.

### Phase 3: Finalize and Move to Output

1. **Final Scrub** — Run `/scrub` one more time on the optimized article to clean any new AI patterns introduced during optimization
2. **Generate Optimization Summary** — Create a brief optimization summary at the bottom of the article as an HTML comment:
   ```
   <!-- Optimization Summary
   SEO Score: [score]/100
   Word Count: [count]
   Primary Keyword: [keyword]
   Keyword Density: [X.X%]
   Internal Links: [count]
   External Links: [count]
   Readability: Grade [X]
   -->
   ```
3. **Move to Output** — Move the final article from `seo/drafts/` to `seo/output/[topic-slug]-[YYYY-MM-DD].md`
4. **Archive Supporting Files** — Move all supporting files for this article from `seo/drafts/` into `seo/archive/[topic-slug]/`. This includes content-analysis, seo-report, meta-options, link-suggestions, and keyword-analysis files. Create the archive directory if it doesn't exist. This keeps `seo/drafts/` clean for in-progress work only.

## Output

### Primary Output
A single, publish-ready article in `seo/output/[topic-slug]-[YYYY-MM-DD].md` containing:
- Complete article with all optimizations applied
- Snake_case YAML frontmatter (`title`, `meta_title`, `meta_description`, `primary_keyword`, `secondary_keywords`, `url_slug`, and queue metadata such as `article_type` and `cluster`)
- Proper keyword placement and density
- All internal and external links
- Featured snippet-optimized sections
- Optimization summary as HTML comment

### Supporting Files (archived)
- `seo/archive/[topic-slug]/content-analysis-[topic-slug]-[YYYY-MM-DD].md`
- `seo/archive/[topic-slug]/seo-report-[topic-slug]-[YYYY-MM-DD].md`
- `seo/archive/[topic-slug]/meta-options-[topic-slug]-[YYYY-MM-DD].md`
- `seo/archive/[topic-slug]/link-suggestions-[topic-slug]-[YYYY-MM-DD].md`
- `seo/archive/[topic-slug]/keyword-analysis-[topic-slug]-[YYYY-MM-DD].md`

### Completion Message
When finished, display:
```
Article created and optimized:
  Output: seo/output/[filename].md
  Word Count: [count]
  Quality Score: [score]/100
  Reports: seo/archive/[topic-slug]/
```

## Quality Gate

This command will NOT move an article to `seo/output/` if:
- Quality score is below 70 after revision attempts (article goes to `seo/review-required/` instead)
- The article fails to meet minimum requirements (2000+ words, proper heading hierarchy, required links)

If the quality gate fails, the article stays in `seo/review-required/` with review notes explaining what needs human attention.

## Example

```
/create podcast advertising ROI for small businesses
```

This will:
1. Research and write a full article on the topic
2. Run all agents and apply their recommendations
3. Optimize keywords, meta, links, and structure
4. Scrub for AI patterns
5. Save to `seo/output/podcast-advertising-roi-small-businesses-2026-03-14.md`

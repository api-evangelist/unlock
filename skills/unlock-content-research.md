---
name: unlock-content-research
description: >-
  Research Unlock's published position on a home-equity topic by searching its editorial archive —
  251 blog posts, 9 Learn articles, 13 homeowner stories and the Learn curriculum — through the
  WordPress core content API, with correct pagination and attribution.
api: Unlock Editorial API
base_url: https://www.unlock.com/wp-json
operations:
  - getWpV2Search
  - getWpV2Posts
  - getWpV2PostsById
  - getWpV2Articles
  - getWpV2Stories
  - getWpV2Categories
  - getWpV2Authors
generated: '2026-09-02'
method: generated
source: openapi/unlock-editorial-api-openapi.yml, data-model/unlock-data-model.yml, conventions/unlock-conventions.yml
---

# Research the Unlock editorial archive

Unlock's Learn and blog content is readable as JSON. Use it when you need what **Unlock itself** has
said about a topic — refinancing, HELOC comparisons, settlement, appraisals — rather than what the
wider web says about Unlock.

## Steps

1. **Cast a wide net first.**
   `GET /wp/v2/search?search=<terms>&per_page=20` (`getWpV2Search`)
   Searches across every registered content type at once and returns `{id, title, url, type,
   subtype}`. The `subtype` tells you which collection to go to next: `post`, `articles`, `stories`,
   `lessons`, `topics`.

2. **Pull the full record from the right collection.**
   - Blog posts: `GET /wp/v2/posts/{id}` (`getWpV2PostsById`)
   - Learn articles: `GET /wp/v2/articles` (`getWpV2Articles`)
   - Homeowner stories: `GET /wp/v2/stories` (`getWpV2Stories`)
   Search results carry a title and a URL, not the body; the collection route carries
   `content.rendered`.

3. **Filter by category when you want a themed sweep.**
   `GET /wp/v2/categories` (`getWpV2Categories`) to resolve ids, then
   `GET /wp/v2/posts?categories=<id>&orderby=date&order=desc` (`getWpV2Posts`).

4. **Attribute correctly.**
   Authors are their own content type here, not WordPress user accounts:
   `GET /wp/v2/authors` (`getWpV2Authors`). Eleven bylines at last capture, including in-house
   staff and outside contributors. Attribute a post to the byline it carries, and say plainly that
   it is Unlock's own marketing content.

5. **Page properly.**
   `per_page` is bounded 1..100. Read `X-WP-Total` (251 posts at capture) and `X-WP-TotalPages`, or
   follow `Link; rel="next"`. Prefer date filters — `after`, `before`, `modified_after` — over
   walking every page when you only need recent material.

## Rules

- **`post_tag` is empty on this deployment.** `GET /wp/v2/tags` returns `[]`. Filter with
  `categories`, not `tags`, or you will silently get nothing.
- **Content is marketing.** Every post here is published by a company selling a financial product.
  Report it as Unlock's stated position, never as neutral analysis or as advice.
- **Anonymous and read-only.** No credential. `context=edit` returns 401 `rest_forbidden_context`,
  and the write methods advertised in the route index require a WordPress application password
  Unlock does not issue publicly.
- **Respect the cache.** The edge serves `Cache-Control: public, max-age=604800`. Re-fetching the
  same collection repeatedly gains you nothing.
- **Do not touch administration routes.** The route index also advertises `/wp/v2/users`,
  `/wp/v2/plugins`, `/wp/v2/settings` and application-password routes. They are WordPress
  administration surface, they are not anonymously readable, and they are out of scope.

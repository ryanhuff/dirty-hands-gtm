# Publish to HubSpot

Save a finished article to HubSpot as a blog post draft. This skill does NOT author content. It takes an existing article file, fetches configuration from HubSpot, asks any needed clarifying questions, and creates the blog post via the HubSpot V3 API.

## Trigger

User invokes `/author-hubspot` with an optional article path as argument.

## Context Consumed

- The article file to publish (from `outputs/articles/`)
- The SEO assets YAML for the article (from `outputs/seo-assets/`)
- HubSpot configuration cache at `.claude/hubspot-config.json`

Does NOT read: `strategy/`, `customer-intelligence/`, `proof-library/`

## Input

- `$ARGUMENTS` -- path to a finished article markdown file (e.g., `outputs/articles/dealhub-alternative-hubspot_final.md`)
- If no argument provided, list all `_final.md` and `_linked.md` files in `outputs/articles/` and ask the user to select one.
- If no articles exist, tell the user: "No finished articles found. Run /seo-pipeline first to produce a publishable article."

## Authentication

This skill requires the environment variable `HUBSPOT_PERSONAL_ACCESS_TOKEN` to be set.

1. Check that `HUBSPOT_PERSONAL_ACCESS_TOKEN` is available by running: `echo $HUBSPOT_PERSONAL_ACCESS_TOKEN | head -c 4`
2. If the variable is empty or missing, tell the user:
   > Set your HubSpot Personal Access Token as `HUBSPOT_PERSONAL_ACCESS_TOKEN` in your `.env` file. The token needs the `content` scope. You can create one at Settings > Integrations > Private Apps in HubSpot.
3. Stop execution if the token is not available.

## Configuration

Configuration is cached at `.claude/hubspot-config.json` to avoid re-fetching on every run.

### Loading config

1. Check if `.claude/hubspot-config.json` exists.
2. If it exists, read it and use the values as defaults. The schema:

```json
{
  "portalId": "string — HubSpot portal/account ID, used to construct editor URLs",
  "blogId": "string — contentGroupId of the target blog",
  "blogName": "string — human-readable blog name",
  "blogAuthorId": "string — default blog author ID",
  "blogAuthorName": "string — human-readable author name",
  "defaultTagIds": ["array of tag ID strings commonly used"],
  "defaultTagNames": ["array of tag names for display"],
  "lastUpdated": "ISO 8601 timestamp"
}
```

3. If the config file does not exist, or if the user passes `--reconfigure`, run the full configuration flow (see below).

### Configuration flow

Run this when no config exists or the user requests reconfiguration.

**Step 1: Discover blogs**

```
GET https://api.hubapi.com/cms/v3/blogs/posts?limit=1
```

Use the `Authorization: Bearer $HUBSPOT_PERSONAL_ACCESS_TOKEN` header for all API calls.

If this returns a 401, tell the user their token is invalid or missing the `content` scope and stop.

Then fetch the list of blogs:

```
GET https://api.hubapi.com/content/api/v2/blogs?limit=100
```

Present the list of blogs to the user (name + ID) and ask them to select one. If only one blog exists, confirm it as the default.

**Step 2: Discover authors**

```
GET https://api.hubapi.com/cms/v3/blogs/authors?limit=100
```

Present the list of authors (displayName + ID) and ask the user to select a default author. If only one author exists, confirm it as the default.

**Step 3: Discover tags**

```
GET https://api.hubapi.com/cms/v3/blogs/tags?limit=100
```

Present the available tags and ask the user if they want to set any as defaults. This is optional. The user can skip this step.

**Step 4: Get portal ID**

Ask the user for their HubSpot portal ID (also called account ID). This is needed to construct editor URLs. It can be found in the URL bar when logged into HubSpot (e.g., `app.hubspot.com/blog/{portalId}/...`).

**Step 5: Save config**

Write the configuration to `.claude/hubspot-config.json` with all selected values, the portal ID, and a `lastUpdated` timestamp.

Tell the user: "HubSpot configuration saved. These defaults will be used for future posts. Run `/author-hubspot --reconfigure` to change them."

---

## Workflow

### Step 1: Parse the article

1. Read the article markdown file.
2. Look for a matching SEO assets file at `outputs/seo-assets/{slug}_seo-assets.yaml`. If it exists, extract:
   - `title_tag` --> use as `htmlTitle`
   - `meta_description` --> use as `metaDescription`
   - `slug` --> use as the URL `slug`
3. Parse the article markdown:
   - The first H1 becomes `name` (the blog post title).
   - Everything after the H1 is the post body. Convert the markdown body to HTML for `postBody`.
   - If an H1 is not found, ask the user for a title.

### Step 2: Confirm post details

Present the user with a summary of what will be created:

| Field | Value |
|-------|-------|
| Title | (from article H1 or SEO title_tag) |
| Slug | (from SEO assets or derived from title) |
| Blog | (from config -- blogName) |
| Author | (from config -- blogAuthorName) |
| Meta description | (from SEO assets or ask user) |
| Tags | (from config defaults, if any) |
| State | DRAFT |

Ask the user:
- "Does this look correct? Reply with any changes, or confirm to proceed."
- If the user wants to change tags, present the full tag list from configuration and let them pick.
- If the user wants to add new tags, note that tags must be created in HubSpot first (or offer to create them via POST to `/cms/v3/blogs/tags`).
- If no meta description is available (no SEO assets file), ask the user to provide one. A meta description is required for future publishing.

### Step 3: Convert markdown to HTML

Convert the article body from markdown to HTML. Use a Bash command with a tool that handles markdown-to-HTML conversion. Options in order of preference:

1. If `pandoc` is available: `pandoc -f markdown -t html`
2. If `npx` is available: `npx marked`
3. As a last resort, do a basic manual conversion in-skill:
   - `## heading` --> `<h2>heading</h2>`
   - `**bold**` --> `<strong>bold</strong>`
   - `*italic*` --> `<em>italic</em>`
   - `- list item` --> `<ul><li>list item</li></ul>`
   - `[text](url)` --> `<a href="url">text</a>`
   - Paragraphs separated by blank lines --> `<p>...</p>`

### Step 4: Create the blog post

Make the API call:

```
POST https://api.hubapi.com/cms/v3/blogs/posts
Authorization: Bearer $HUBSPOT_PERSONAL_ACCESS_TOKEN
Content-Type: application/json

{
  "name": "<title>",
  "contentGroupId": "<blogId from config>",
  "slug": "<slug>",
  "blogAuthorId": "<blogAuthorId from config>",
  "metaDescription": "<meta description>",
  "htmlTitle": "<SEO title tag or article title>",
  "postBody": "<HTML content>",
  "postSummary": "<meta description or first paragraph>",
  "useFeaturedImage": false,
  "tagIds": [<tag IDs if any>]
}
```

### Step 5: Handle the response

**On success (201):**
1. Extract the post `id` and `url` from the response.
2. Always construct and display the editor URL using the portal ID from config: `https://app.hubspot.com/blog/{portalId}/editor/{postId}`
3. Tell the user:
   > Blog post created as DRAFT in HubSpot.
   > - **Title:** {name}
   > - **Post ID:** {id}
   > - **Review draft:** https://app.hubspot.com/blog/{portalId}/editor/{id}
   >
   > The post is saved as a draft. Review it in HubSpot and publish when ready. To publish via API, I can set the state to PUBLISHED -- just ask.

**Important:** The editor link is mandatory. Always include it in the success output. Use the `portalId` from `.claude/hubspot-config.json` to construct it.

**On error:**
- **401:** Token expired or invalid. Ask the user to regenerate their Personal Access Token.
- **403:** Token missing the `content` scope. Tell the user which scope is needed.
- **400:** Parse the error message from the response body and present it. Common issues:
  - Missing required field: identify which field and ask the user to provide it.
  - Invalid `contentGroupId`: suggest running `--reconfigure`.
  - Duplicate slug: suggest an alternative slug and ask the user to confirm.
- **429:** Rate limited. Wait and retry once after 10 seconds.

### Step 6: Offer next steps

After successful creation, ask the user:

> What would you like to do next?
> - **Publish now** -- I will set the post state to PUBLISHED (requires slug, author, and meta description to be set)
> - **Schedule** -- Provide a date/time and I will schedule the post
> - **Done** -- Leave as draft for review in HubSpot

If the user chooses to publish:

```
PATCH https://api.hubapi.com/cms/v3/blogs/posts/{postId}
Authorization: Bearer $HUBSPOT_PERSONAL_ACCESS_TOKEN
Content-Type: application/json

{
  "state": "PUBLISHED"
}
```

If the user chooses to schedule:

```
POST https://api.hubapi.com/cms/v3/blogs/posts/schedule
Authorization: Bearer $HUBSPOT_PERSONAL_ACCESS_TOKEN
Content-Type: application/json

{
  "id": "{postId}",
  "publishDate": "<ISO 8601 datetime from user>"
}
```

---

## API Reference

All API calls use `Authorization: Bearer $HUBSPOT_PERSONAL_ACCESS_TOKEN` header.

| Action | Method | Endpoint |
|--------|--------|----------|
| List blogs | GET | `/content/api/v2/blogs?limit=100` |
| List authors | GET | `/cms/v3/blogs/authors?limit=100` |
| List tags | GET | `/cms/v3/blogs/tags?limit=100` |
| Create tag | POST | `/cms/v3/blogs/tags` with `{"name": "tag name"}` |
| Create post | POST | `/cms/v3/blogs/posts` |
| Update post | PATCH | `/cms/v3/blogs/posts/{postId}` |
| Publish post | PATCH | `/cms/v3/blogs/posts/{postId}` with `{"state": "PUBLISHED"}` |
| Schedule post | POST | `/cms/v3/blogs/posts/schedule` |

## Notes

- This skill creates posts as DRAFT by default. Publishing is always an explicit user action.
- The `content` scope is required on the Personal Access Token.
- Configuration is saved to `.claude/hubspot-config.json` so that blog, author, and tag defaults persist across runs.
- If the user says `--reconfigure`, re-run the full configuration flow even if a config file exists.
- Markdown-to-HTML conversion should preserve the semantic structure of the article. Do not strip headings, lists, or links.

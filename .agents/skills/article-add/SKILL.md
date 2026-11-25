---
name: article-add
description: 'Add one or more articles, releases, or announcements as new entries to ITDigest.md (IT), AINews.md (AI), JavaNews.md (Java), or NuxtNews.md (Nuxt) in it-news. Use when the user provides URLs to add to any of the curated news or digest lists.'
argument-hint: '<IT|AI|JAVA|NUXT> <url> [<url> ...]'
---

# Add Articles to it-news Lists

Input: a list selector followed by one or more URLs (`/article-add <IT|AI|JAVA|NUXT> <url1> <url2> ...`).
Each URL becomes one table row in the selected file.

| First argument | Target file | Grouping |
| --- | --- | --- |
| `IT` | [it/ITDigest.md](../../../it/ITDigest.md) | Yearly (`## YYYY`) |
| `AI` | [ai/AINews.md](../../../ai/AINews.md) | Yearly (`## YYYY`) |
| `JAVA` | [java/JavaNews.md](../../../java/JavaNews.md) | Monthly (`## YYYY - Month`) |
| `NUXT` | [nuxt/NuxtNews.md](../../../nuxt/NuxtNews.md) | Monthly (`## YYYY - Month`) |

- The first argument is case-insensitive. If it is missing or is not one of `IT`, `AI`, `JAVA`, or `NUXT` (for example, the user passes only URLs), ask which list to use. Do not guess.
- Only edit the selected file. Do not create other files or notes.
- All four files share the same row format and workflow. File-specific placement rules are noted in the Placement section.

## Entry format

```markdown
| **`YYYY-MM-DD`** | [Article Title](URL) - short description with **bold** key terms |
```

- Title: the article's own title. Shorten only if it is very long or clickbaity.
- Description: one sentence, after ` - `, in a neutral informative tone. No emojis, no marketing wording,
  no opinions of your own. Say what the article announces or explains.
- Bold (`**...**`) the key product, project, release, or version — e.g. `**Rust**`, `**React**`, `**Maven**` (IT/AI),
  `**JDK 28**`, `**Quarkus**`, `**WildFly 42**` (Java), `**Nuxt v4**`, `**Vue**`, `**Vite 8**` (Nuxt).
  Use backticks for code identifiers, e.g. `` `HttpClient` ``, `` `ref` ``.
- Use the original article URL exactly as the user gave it (do not add tracking parameters or rewrite it).

## Workflow

1. Fetch all URLs in parallel. Ask for the title, publication date, author, and a content summary.
2. Determine the date of each article (see below). Never guess it.
3. Read the current sections of the selected file **after** fetching (current year sections for `it/ITDigest.md` and `ai/AINews.md`; current month sections at the top for `java/JavaNews.md` and `nuxt/NuxtNews.md`). The user edits these files by hand (shortening descriptions, fixing typos), so always work from the current content, not from memory.
4. Insert the rows (see Placement below) with a single edit per file where possible.
5. Report briefly: one line per article with its date and the section it went to, plus any caveat.

## Determining the date

Use the real publication date, in this order of preference:

- The byline or date shown on the page (e.g. InfoQ, JavaPro, Foojay, inside.java, WildFly, Quarkus, TornadoVM, InfoWorld, ...).
  The date in the URL may be wrong or only approximate (for example, a Foojay URL can differ from its byline date).
- GitHub release pages show relative dates ("2 weeks ago"). Fetch
  `https://api.github.com/repos/<owner>/<repo>/releases/tags/<tag>` and use `published_at`.
- Mailing-list announcements (mail-archive.com): use the date in the message header.
- Anthropic launch pages (e.g. `anthropic.com/claude-...`): use the dated entry on <https://www.anthropic.com/news>. Research posts (`/research/...`) show `Published <date>` in the text.
- OpenAI pages (`openai.com/index/...`) that show no date in the text: use the dated card on <https://openai.com/news/>.
- If the date cannot be determined, ask the user. Do not invent one. If the user supplies it later, add the entry then.

## Placement

### Common rules

- Every section uses the same table header:

  ```markdown
  | Date&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Article |
  | --- | :-- |
  ```

- Sections are ordered newest first. Put each entry into the section of its own period (year or month), which is not necessarily the current one.
- If the target section does not exist yet, create it directly above the newest existing section, copying the table header above.
- Within a section, order rows by date, newest first. For equal dates, add the new row next to the existing rows of that date.
- The file may already contain a few out-of-order rows. Do not reorder or reformat existing rows.

### Yearly lists (`it/ITDigest.md`, `ai/AINews.md`)

- Entries are grouped in `## YYYY` sections (e.g. `## 2026`). A late December article added in January goes to the previous year.
- There are no yearly archive files (e.g. `ITDigest2025.md`) at this time.
- **`it/ITDigest.md` only**: everything older than 2024 is collected in a single `## < 2024` section at the very bottom, above the closing `---` and the "this list is being actively maintained since" footer line. Do not create per-year sections below 2024.
- **`ai/AINews.md` only**: the list is maintained since 11/2026, so there is no `## < 2024` section. Leave the `## Sources` section above the year sections untouched and keep year sections below it and above the closing `---` footer. If an article is older than the maintained-since date, still add it to its year section, but mention it in the report. A year section may exist with an empty table (header only); add rows directly below the header.

### Monthly lists (`java/JavaNews.md`, `nuxt/NuxtNews.md`)

- Entries are grouped in `## YYYY - Month` sections (e.g. `## 2026 - October`). A late August article added in September goes to August.
- Leave introductory sections above the month sections untouched (e.g. `## State of JDK`, `## Sources`, `## Events`, `## Upcoming events`).
- If the date belongs to a year other than the one in the file's sections, ask the user before touching archive files (`java/JavaNews2025.md`, `nuxt/NuxtNews2025.md`, etc.).

## Safe editing

- Match the existing text exactly in the old-string of the edit, and copy every unchanged row verbatim into the new string.
  After the edit, check that no existing row changed (for example, by grepping a neighbouring row). Past mistake:
  rewording `**Java 28**` to `**JDK 28**` in an unrelated row. Fix such slips immediately.
- Do not add a row for a URL that is already in the selected file. Tell the user instead.

## When a page cannot be read

- Cookie walls and long boilerplate are normal. Look for the article body, byline, and date in the fetched text.
- If a fetch redirects to an unrelated (ad or tracking) domain, do not follow it. Retry once, then try one alternative
  source (the project's own site or API). If the content is still unavailable, say so.
- Without readable content, write the description only from what is verifiable (for example, the title) and tell the user.
  If the date is also unknown, ask for it before adding the row.

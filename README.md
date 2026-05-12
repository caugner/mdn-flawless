# MDN Flawless

A single-file static viewer for `issues.json` build output from [Rari](https://github.com/mdn/rari) (when run with `--issues`). No build step — open `index.html` from a local HTTP server and browse, filter, group, and export the issue set.

The deployment at <https://caugner.github.io/mdn-flawless/> shows the `issues.json` from the last production build of <https://developer.mozilla.org/>.

## Running

```sh
python3 -m http.server
# or
npx http-server .
```

Then open <http://localhost:8000/> (or <http://localhost:8080/>). The page streams `issues.json` with a live progress bar and flattens it into a searchable table.

## Features

### Filtering

- Full-text search across message, file, URL, redirect, source, slug, basepath, macro, and sidebar (space-separated tokens, AND semantics).
- Faceted dropdowns with live counts: **Category**, **Repository**, **Locale**, **Macro (templ)**, **Source position** (with/without line info).
- "File path contains" substring filter.
- Click any pill (repo, locale, macro, category) in a row to filter by that value.
- **Reset** clears all filters; the button highlights whenever any filter is active.

### Grouping

- Group by **URL**, **Slug**, or **File path**.
- Group rows are sortable by key or count, and expand inline to reveal their issues.

### Table

- Sortable columns (File, Category, Message); keyboard-accessible headers.
- Search-term highlighting via `<mark>`.
- File cell links to the GitHub blob with `?plain=1` and precise `L#C#-L#C#` anchors (1-based, derived from the 0-based source spans).
- Slug links to the live MDN page using normalized locale casing (e.g. `pt-br` → `pt-BR`).
- Pagination (50 / 200 / 500 / 2000 per page) with prev/next and a page indicator.

### Export

- **Copy MD** — Markdown table of the filtered list (or grouped summary), with an active-filter summary and a link back to the live view.
- **Copy CSV** — same data as CSV (grouped or flat).

### UX

- Sticky header, dark mode via `prefers-color-scheme`.
- URL hash mirrors all filters — links are shareable, and back/forward updates the view.
- `aria-live` stats line shows filtered/total issue count plus file and slug counts.

## Fields

Each row in `issues.json` is flattened into a single record with the following fields. See `flatten()` in [index.html](index.html).

### Location

| Field | Description |
|---|---|
| `repo` | Derived from the path: `content`, `translated-content`, `translated-content-de` (when locale is `de`), or `curriculum`. |
| `filePath` | Path to the source `.md` file, normalized to start at `content/`, `translated-content/`, or `curriculum/curriculum/` (absolute prefix from the linter run is stripped). |
| `file` | The file the linter actually parsed. Usually equal to `filePath`. |
| `line` / `col` / `endLine` / `endCol` | 0-based source position. Converted to 1-based GitHub anchors (`L11C96-L11C155`) for blob links. |

### Issue content

| Field | Description |
|---|---|
| `source` | The linter check that produced the issue (e.g. `templ-broken-link`, `templ-redirected-link`, `broken-link`). |
| `message` | Human-readable error text. May be empty for some sources (e.g. `broken-link`), in which case `source` acts as the category. |
| `category` | Coarse bucket extracted from the start of `message`: longest matching known prefix, else the part before `:`, else the first 4 words. |
| `sourceCategory` | What the Category filter uses: `source: category` when both exist, otherwise whichever is present. Keeps e.g. `templ-broken-link` and `broken-link` distinguishable even when their messages categorize the same. |

### Document context (from `spans`)

| Field | Description |
|---|---|
| `locale` | MDN locale (`en-US`, `de`, `pt-BR`, …). Normalized to MDN URL casing when linking out. |
| `slug` | Document slug, e.g. `Glossary/Leading`. Linked to `developer.mozilla.org/{locale}/docs/{slug}`. |
| `basepath` | Base path span reported by the linter. Searchable, not displayed as a column. |
| `templ` | Rari template (macro) being expanded when the issue was found (e.g. `glossary`, `cssxref`). |
| `sidebar` | Sidebar context, if any. Searchable only. |

### Issue payload (from `fields`)

| Field | Description |
|---|---|
| `url` | The URL the linter was checking (e.g. the target of a broken link). |
| `redirect` | For redirect issues, the URL it actually redirects to. |

## License

See [LICENSE](LICENSE).

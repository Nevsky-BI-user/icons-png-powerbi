---
name: icon-set-manager
description: Use whenever user asks to create, search, find, list, or manage PNG icons for Power BI reports stored in a GitHub icon library. Unified find-or-fetch flow - if requested icon exists in manifest.json, return raw URL and DAX snippet; if not, fetch from external sources (Iconify aggregator with 200+ sets, SVG Repo, optional Flaticon/Icons8), commit to GitHub, return URL. Triggers - 'іконка', 'icon', 'набір іконок', 'знайди іконку', 'find icon', 'create icons', 'PNG для звіту', 'Power BI icon', 'додай іконки', 'потрібні іконки', 'icon library'. Defaults - transparent PNG, color 063E61, size 64px (128px for hero/header), naming category_name.png snake_case ASCII. Repo - icons-png-powerbi. Prefers permissive licenses (MIT/Apache/CC0). Do NOT trigger for DAX measures (use dax-measures), SVG measures (use dax-svg), or Deneb specs (use deneb-vegalite).
---

# Icon Set Manager

Unified workflow for Power BI icon library: searches existing icons by semantic query, fetches missing icons from free sources, stores everything in one GitHub repository organized by category.

## Configuration

These values are substituted during setup (see `setup-instructions.md`):

- **Nevsky-BI-user**: `Nevsky-BI-user`
- **REPO_NAME**: `icons-png-powerbi`
- **BRANCH**: `main`
- **LOCAL_CLONE**: `~/projects/icons-png-powerbi`
- **MANIFEST_PATH**: `manifest.json` (at repo root)
- **ICON_PATH_PATTERN**: `icons/{category}/{category}_{name}.png`
- **RAW_URL_PATTERN**: `https://raw.githubusercontent.com/{Nevsky-BI-user}/{REPO_NAME}/{BRANCH}/{path}`
- **CDN_URL_PATTERN**: `https://cdn.jsdelivr.net/gh/{Nevsky-BI-user}/{REPO_NAME}@{BRANCH}/{path}` (jsDelivr proxy for production Power BI)

API keys (optional, stored in user config):

- **FLATICON_KEY**: `~/.config/icon-set-manager/flaticon.key` (chmod 600)
- **ICONS8_KEY**: `~/.config/icon-set-manager/icons8.key` (chmod 600)

If a key file is absent, that source is silently skipped in the fallback chain.

## Defaults

| Parameter | Default | Override trigger |
|---|---|---|
| Color | `063e61` | "колір X", "color X", "білий", "чорний", "червоний", brand-color names |
| Size | `64px` | "128", "256", "великий", "hero", "header", "заголовок", "banner" |
| Background | transparent | (never overridden, always transparent) |
| Style family | `material-symbols-outlined` | "lucide", "mdi", "tabler", "phosphor", "game", "carbon", "fluent", "heroicons" |
| Naming | `{category}_{name}` snake_case ASCII | (fixed convention) |
| License preference | permissive (MIT, Apache 2.0, CC0, CC-BY-SA, ISC) | "будь-яка ліцензія", "any license" |

Hero/header sizing: if user requests size ≥128 OR query contains hero-marker words, fetch 128px.

## 20 Categories

```
navigation       — home, back, forward, menu, search, filter, breadcrumb, sidebar
actions          — refresh, export, import, edit, delete, add, drill, share, save, copy
kpi              — trend-up, trend-down, target, threshold, alert, gauge, score, goal
status           — ok, warning, error, info, pending, locked, blocked, completed
time             — calendar, clock, period, history, schedule, deadline, timer
data             — table, chart, database, file, dataset, query, pivot, csv, excel
users            — person, team, role, manager, group, profile, avatar
communication    — email, chat, notification, phone, video, message, bell
security         — lock, shield, key, password, audit, access, permission
documents        — contract, certificate, report, invoice-doc, presentation, archive
geography        — map, pin, region, country, globe, address, route-map
analytics        — funnel, segment, cohort, dashboard, insight, distribution
finance          — money, currency, bank, invoice, payment, budget, profit, expense
ecommerce        — cart, order, product, shipping, package, discount, store
hr               — employee, hiring, training, onboarding, performance, leave, payroll
agro             — crop, livestock, tractor, field, harvest, weather, seed, irrigation
logistics        — truck, warehouse, container, delivery, route, fleet
production       — factory, machine, quality, conveyor, gear, assembly, robot
marketing        — campaign, megaphone, lead, conversion, brand, ad
sales            — deal, pipeline, contract, handshake, quote, opportunity, won, lost
```

If user requests a category not on this list, ask for confirmation before creating a new folder.

## Unified Workflow: Find or Fetch

Primary entry point. Activates on any icon-related query.

### Step 1 — Parse query

Extract:
- **Intent**: explicit lookup ("знайди іконку для X") vs explicit create ("створи іконку X") vs ambiguous ("потрібна іконка X")
- **Category**: from explicit mention OR inferred from concept (e.g., "tractor" → `agro`); if ambiguous, ask
- **Concept/name**: actual icon meaning
- **Style override**: e.g., "в стилі lucide"
- **Color override**: e.g., "колір білий"
- **Size override**: e.g., "128px", "hero"

### Step 2 — Search existing icons in manifest

```bash
cd ~/projects/icons-png-powerbi
git pull --quiet origin main  # ensure manifest is current
```

Read `manifest.json`. Match query semantically:

1. **Exact name match** within category → confidence HIGH
2. **Tag exact match** (any tag field equals query term) → confidence HIGH
3. **Substring/fuzzy name match** within category → confidence MEDIUM
4. **Cross-category tag match** → confidence MEDIUM
5. **Semantic match** (e.g., "growth" → finds icon tagged `trend-up`) → confidence MEDIUM
6. **No match** → confidence ZERO

Decision matrix:

| Confidence | Style override matches stored style? | Action |
|---|---|---|
| HIGH | yes | Return existing icon, skip fetch |
| HIGH | no | Return existing + offer to fetch alternative in requested style |
| MEDIUM | — | Show top 3 existing candidates + offer to fetch new |
| ZERO | — | Proceed to Step 3 (fetch) |

If user explicitly says "створи новий" / "force new" — skip search, go to Step 3.

### Step 3 — Fetch from external sources (only if Step 2 = ZERO or user confirms)

Fallback chain executed in order. First success wins.

```
3.1  Iconify direct lookup   → api.iconify.design/{set}/{name}.png?color=063e61&width=64
     Sets tried in order: material-symbols-outlined, mdi, lucide, tabler,
                          phosphor, game-icons, carbon, fluent, healthicons

3.2  Iconify search API      → api.iconify.design/search?query={term}&limit=20
     Pick first match from preferred sets above; respect license preference.

3.3  SVG Repo direct/search  → www.svgrepo.com/api/v1/search?term={term}&type=svg
     Download SVG → recolor (sed) → rsvg-convert to PNG.

3.4  Flaticon API (if key)   → api.flaticon.com/v3/search/icons?q={term}
     Requires FLATICON_KEY. Free tier: attribution required.

3.5  Icons8 API (if key)     → api.icons8.com/api/iconsets/v3/icons?term={term}
     Requires ICONS8_KEY. Free tier: 100 downloads/day, attribution required.

3.6  Manual selection        → Iconify search returns candidates →
                              present 5 to user → user picks.
```

#### 3.1 Iconify direct (server-side render)

```bash
SET="material-symbols-outlined"
NAME="agriculture"
CATEGORY="agro"
COLOR="063e61"
SIZE=64

curl -sfL --max-time 10 \
  "https://api.iconify.design/${SET}/${NAME}.png?color=${COLOR}&width=${SIZE}" \
  -o "icons/${CATEGORY}/${CATEGORY}_${NAME}.png"

# Verify it's actually a PNG, not 404 HTML
if file "icons/${CATEGORY}/${CATEGORY}_${NAME}.png" | grep -q "PNG image"; then
  SOURCE="iconify:${SET}"
  LICENSE=$(curl -s "https://api.iconify.design/collection?prefix=${SET}" | jq -r '.license.title // "unknown"')
else
  rm "icons/${CATEGORY}/${CATEGORY}_${NAME}.png"
  # Try next set in chain
fi
```

#### 3.2 Iconify search (fuzzy across all 200+ sets)

```bash
QUERY="growth chart"
ENCODED=$(printf %s "$QUERY" | jq -sRr @uri)
RESULTS=$(curl -s "https://api.iconify.design/search?query=${ENCODED}&limit=20" | jq -r '.icons[]')

# Returns list like:
#   material-symbols:trending-up
#   lucide:trending-up
#   mdi:trending-up
# Pick first match from preferred sets; verify license is permissive.
```

#### 3.3 SVG Repo (vector source, requires local conversion)

```bash
QUERY="tractor"
CATEGORY="agro"
NAME="tractor"
COLOR="063e61"
SIZE=64

# Search SVG Repo
RESPONSE=$(curl -s "https://www.svgrepo.com/api/v1/search?term=${QUERY}&type=svg")
SVG_URL=$(echo "$RESPONSE" | jq -r '.icons[0].svg_url // empty')
LICENSE=$(echo "$RESPONSE" | jq -r '.icons[0].license // "unknown"')
AUTHOR=$(echo "$RESPONSE" | jq -r '.icons[0].author // empty')

if [ -z "$SVG_URL" ]; then
  echo "SVG Repo: no results for $QUERY"
  exit 1
fi

# Download SVG
curl -sfL "$SVG_URL" -o /tmp/icon.svg

# Recolor if monotone — replace any fill hex or currentColor with target color
sed -i.bak -E \
  -e 's/fill="#[0-9A-Fa-f]{3,8}"/fill="#'"$COLOR"'"/g' \
  -e 's/currentColor/#'"$COLOR"'/g' \
  /tmp/icon.svg

# Rasterize to transparent PNG of target size
rsvg-convert -w "$SIZE" -h "$SIZE" -b none /tmp/icon.svg \
  -o "icons/${CATEGORY}/${CATEGORY}_${NAME}.png"

# Cleanup
rm -f /tmp/icon.svg /tmp/icon.svg.bak
```

#### 3.4 Flaticon (requires API key, attribution required)

```bash
KEY=$(cat ~/.config/icon-set-manager/flaticon.key 2>/dev/null)
[ -z "$KEY" ] && { echo "SKIP Flaticon: no key"; exit 1; }

# Authenticate (Flaticon uses Bearer token after exchange)
TOKEN=$(curl -s -X POST "https://api.flaticon.com/v3/app/authentication" \
  -H "Accept: application/json" \
  -d "apikey=${KEY}" | jq -r '.data.token')

# Search
QUERY="employee"
RESPONSE=$(curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://api.flaticon.com/v3/search/icons/priority?q=${QUERY}&styleColor=black&limit=10")

# Pick first result
ICON_ID=$(echo "$RESPONSE" | jq -r '.data[0].id')
ICON_AUTHOR=$(echo "$RESPONSE" | jq -r '.data[0].author.name')
ICON_PACK=$(echo "$RESPONSE" | jq -r '.data[0].pack.name')
ICON_URL=$(echo "$RESPONSE" | jq -r '.data[0].images.png."64"')

curl -sfL -o "icons/${CATEGORY}/${CATEGORY}_${NAME}.png" "$ICON_URL"

# Attribution required for free tier — stored in manifest + ATTRIBUTIONS.md
```

#### 3.5 Icons8 (requires API key, attribution required)

```bash
KEY=$(cat ~/.config/icon-set-manager/icons8.key 2>/dev/null)
[ -z "$KEY" ] && { echo "SKIP Icons8: no key"; exit 1; }

QUERY="department"
RESPONSE=$(curl -s -H "Authorization: ${KEY}" \
  "https://api.icons8.com/api/iconsets/v3/icons?term=${QUERY}&platform=color&amount=10")

ICON_ID=$(echo "$RESPONSE" | jq -r '.icons[0].id')
ICON_URL="https://img.icons8.com/${ICON_ID}/${SIZE}/${COLOR}.png"

curl -sfL -o "icons/${CATEGORY}/${CATEGORY}_${NAME}.png" "$ICON_URL"
```

### Step 4 — Update manifest.json

Append entry to `icons` array. Schema:

```json
{
  "path": "icons/agro/agro_tractor.png",
  "url_raw": "https://raw.githubusercontent.com/Nevsky-BI-user/icons-png-powerbi/main/icons/agro/agro_tractor.png",
  "url_cdn": "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/agro/agro_tractor.png",
  "category": "agro",
  "name": "tractor",
  "tags": ["tractor", "farm machinery", "agriculture", "трактор", "трактор", "agro", "vehicle"],
  "source": "iconify:material-symbols-outlined",
  "source_icon_id": "material-symbols:agriculture",
  "style": "outlined",
  "size_px": 64,
  "color": "063e61",
  "license": "Apache-2.0",
  "attribution": null,
  "added": "2026-05-11"
}
```

**Tag generation**:
- Original icon name (split kebab/camel: `trend-up` → `trend up`)
- English synonyms (2-4)
- Ukrainian translation (always include)
- Russian translation
- Category name
- Industry-specific synonyms (e.g., for `kpi_target` add `goal`, `objective`, `okr`)

**License handling**:
- Permissive non-attribution (MIT, Apache-2.0, CC0, ISC) → `"attribution": null`
- Attribution-required (CC-BY, CC-BY-SA, Flaticon free, Icons8 free) → fill `"attribution": "Icon by {author} from {pack} via {source}"`
- Append unique attribution to `ATTRIBUTIONS.md` in repo root (deduplicate by `source_icon_id`)

### Step 5 — Atomic JSON update

```bash
ENTRY='{ "path": "icons/agro/agro_tractor.png", ... }'

jq --argjson entry "$ENTRY" \
   --arg date "$(date +%Y-%m-%d)" \
   '.icons += [$entry] | .updated = $date | .count = (.icons | length)' \
   manifest.json > manifest.tmp && mv manifest.tmp manifest.json

# Validate
jq empty manifest.json || { echo "manifest.json corrupt"; exit 1; }
```

### Step 6 — Commit and push

```bash
git add icons/${CATEGORY}/${CATEGORY}_${NAME}.png manifest.json ATTRIBUTIONS.md 2>/dev/null
git add icons/ manifest.json
git commit -m "Add ${CATEGORY}/${NAME} from ${SOURCE}"
git push --quiet origin main
```

**For batch operations** (multiple icons in one request): collect all changes, ONE commit + push at the end.

### Step 7 — Return result

Output format:

```
Іконка знайдена/створена.

Шлях:     icons/agro/agro_tractor.png
Raw:      https://raw.githubusercontent.com/Nevsky-BI-user/icons-png-powerbi/main/icons/agro/agro_tractor.png
CDN:      https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/agro/agro_tractor.png
Джерело:  Iconify / material-symbols-outlined
Ліцензія: Apache-2.0 (без attribution)

DAX:
Icon Tractor = "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/agro/agro_tractor.png"
```

Always remind user: set the measure's **Data Category → Image URL** in Power BI model view, then use in Image visual or Table column.

Default to `url_cdn` (jsDelivr) in DAX — faster, not rate-limited. Use `url_raw` only for debugging.

## Search-Only Mode

User says "тільки пошук" / "не створюй" / "search only" — execute Steps 1-2, skip Step 3 if no match. Return: "В репо нема. Хочете створити з зовнішніх джерел?"

## Create-Only Mode

User says "створи новий" / "force new" / "не шукай в репо" — skip Step 2, go straight to Step 3. Useful when:
- Stored icon is in wrong style/color and user wants alternative
- User wants duplicate with different name

Append `_v2` / `_alt` / user-supplied suffix to filename to avoid collision.

## Batch Mode

User requests multiple icons (e.g., "створи 6 іконок для hr"):

1. Resolve full list of icon names (explicit list OR category seed names)
2. Show proposed list, wait for confirmation IF count > 5
3. For each icon: Steps 1-5 (with `git push` deferred)
4. ONE commit + push at end: `Add hr icons: employee, hiring, training, onboarding, performance, leave (6)`
5. Return all results in single response (one block per icon)

## Error Handling

| Symptom | Cause | Resolution |
|---|---|---|
| `curl` returns HTML 404 instead of PNG | Icon name not in Iconify set | Fall through to next set in chain automatically |
| `rsvg-convert: command not found` | `librsvg2-bin` not installed | `sudo apt install librsvg2-bin` or `brew install librsvg` |
| `jq: command not found` | `jq` not installed | `sudo apt install jq` or `brew install jq` |
| `git push` rejected (non-fast-forward) | Remote ahead of local | `git pull --rebase origin main`, retry |
| `git push` auth failure | `gh auth` token expired | `gh auth refresh` |
| Iconify search returns nothing | Term too specific | Broaden with synonyms; try SVG Repo |
| SVG Repo returns nothing | Term not indexed | Try Flaticon/Icons8 if keys configured |
| All sources return nothing | Concept too niche | Present user with 3-5 closest matches from Iconify search |
| Duplicate name in manifest | Same `path` already exists | Skip fetch, return existing; if override → ask for `_v2` suffix |
| `manifest.json` parse error | Malformed from interrupted write | `git show HEAD:manifest.json > manifest.json` |
| jsDelivr 404 immediately after push | Cache not yet updated | Wait 5-10 minutes; use `url_raw` meanwhile |
| Recolor didn't apply (SVG Repo) | Multi-color SVG, has multiple fills | Skip recolor, use original colors; warn user |

## Verification After Each Operation

```bash
# 1. File is valid PNG
file icons/${CATEGORY}/${CATEGORY}_${NAME}.png
# Expected: PNG image data, 64 x 64, 8-bit/color RGBA

# 2. Manifest valid JSON
jq empty manifest.json

# 3. Entry exists in manifest
jq -r --arg path "icons/${CATEGORY}/${CATEGORY}_${NAME}.png" \
   '.icons[] | select(.path == $path) | .name' manifest.json

# 4. Commit pushed
git log origin/main..HEAD  # should be empty

# 5. Raw URL responds 200 (test 30s after push)
curl -sI "https://raw.githubusercontent.com/Nevsky-BI-user/icons-png-powerbi/main/icons/${CATEGORY}/${CATEGORY}_${NAME}.png" | head -1
```

## DAX Generation Patterns

### Single icon constant

```dax
Icon Tractor = "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/agro/agro_tractor.png"
```

### Dynamic switch by dimension

```dax
Icon URL =
SWITCH(
    SELECTEDVALUE(DimCategory[Category]),
    "Sales",     "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/sales/sales_pipeline.png",
    "HR",        "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/hr/hr_employee.png",
    "Agro",      "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/agro/agro_tractor.png",
    "Finance",   "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/finance/finance_money.png",
    BLANK()
)
```

### Conditional KPI icon

```dax
Trend Icon =
VAR _yoy = [Sales YoY %]
RETURN
SWITCH(
    TRUE(),
    _yoy >= 0.05,  "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/kpi/kpi_trend_up.png",
    _yoy <= -0.05, "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/kpi/kpi_trend_down.png",
                   "https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/kpi/kpi_neutral.png"
)
```

Always remind: set **Data Category → Image URL** on the measure in model view.

## Performance and Scale

Soft limit triggers (skill warns user when approaching):

- **Category folder >800 files** → suggest subcategories (e.g., `agro/livestock/`, `agro/machinery/`)
- **Total icons in manifest >5,000** → suggest sharding manifest by category
- **Manifest file size >2 MB** → same suggestion
- **Commits in last hour >100** → batch more aggressively

GitHub absolute limits (will be enforced):
- Single file: 100 MiB (PNG icons ≈5-10 KB, irrelevant)
- Repo size recommended: 1 GB (≈500,000 icons at 2 KB avg)
- Repo soft cap: 5 GB
- Directory width: 3,000 entries
- Push rate: 6/min recommended

## Output Discipline

- Never fabricate icon names — always search source APIs first
- For batch >5: present proposed list, wait for confirmation
- Always output `url_cdn` in DAX snippets (faster than raw)
- Preserve manifest entry order — append only, never reorder
- Never overwrite existing PNG without explicit user request (use `_v2`)
- Always validate file is PNG (use `file` command) before committing
- Always commit + push atomically — one logical operation = one commit
- For attribution-required sources, ALWAYS update `ATTRIBUTIONS.md` and `manifest.attribution` field

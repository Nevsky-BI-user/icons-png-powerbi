# icons-png-powerbi

PNG icon library for Power BI reports. Organized by business domain.

## Structure

`icons/` contains 20 category folders:
navigation, actions, kpi, status, time, data, users, communication,
security, documents, geography, analytics, finance, ecommerce, hr,
agro, logistics, production, marketing, sales.

Top-level files:
- `manifest.json` - index of all icons with metadata
- `ATTRIBUTIONS.md` - required attribution for non-permissive licenses

## Defaults

- Format: PNG, transparent background
- Color: `#063E61` (mono)
- Size: 64x64 px (128x128 for hero/header use)
- Style: `material-symbols-outlined` (Apache 2.0, no attribution required)

## Naming Convention

`{category}_{name}.png` lowercase snake_case ASCII.
Example: `icons/kpi/kpi_trend_up.png`

## Usage in Power BI

1. Find icon via Claude Code: "Знайди іконку для X у моєму репо"
2. Copy `url_cdn` from response (jsDelivr URL)
3. Create DAX measure returning the URL string
4. Set measure Data Category -> "Image URL" in model view
5. Use in Image visual or Table column

## CDN

Production Power BI should use jsDelivr URLs:

```
https://cdn.jsdelivr.net/gh/Nevsky-BI-user/icons-png-powerbi@main/icons/{cat}/{file}.png
```

Raw GitHub URLs also work but are slower:

```
https://raw.githubusercontent.com/Nevsky-BI-user/icons-png-powerbi/main/icons/{cat}/{file}.png
```

## Sources

1. Iconify aggregator (200+ icon sets) - free, no API key
2. SVG Repo (currently disabled - requires `rsvg-convert` not available on Windows native)
3. Flaticon (optional, disabled by default) - free API key, attribution required
4. Icons8 (optional, disabled by default) - free API key, attribution required

Permissive licenses (MIT, Apache 2.0, CC0) preferred over attribution-required.

## License

Repository structure and tooling: CC0 (public domain).
Individual icons: see `manifest.json` (per-icon license field) and `ATTRIBUTIONS.md`.

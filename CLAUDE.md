# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT INSTRUCTIONS FOR CLAUDE - DO NOT IGNORE OR REMOVE
- Claude must never work directly on main branch. Always use a feature branch.
- Claude must always update the CLAUDE.md and README.md files before committing changes.
    - CLAUDE.md must be kept concise, it can never be more than 750 words maximum.
    - README.md is meant for humans. It should focus on project description, usage, and essential info for humans to understand, work with, and contribute to the project. It must be no longer than 750 words.
- Claude must bump the package version in package.json before submitting a PR.

## Commands

```bash
npm run dev      # Start Vite dev server (http://localhost:5173)
npm run build    # Production build → dist/
npm run preview  # Serve production build locally
```

No test suite exists.

## CI/CD

Deployment to GitHub Pages is automated via `.github/workflows/deploy.yml`. Every push to `main` triggers build and deploy using `actions/deploy-pages@v4`. Pages source must be set to "GitHub Actions" in Settings → Pages.

## Architecture

Vanilla JS + Three.js app, no framework. `src/main.js` (`TheTrueSize3DApp` class) orchestrates everything. `src/styles.css` contains all app styles.

**Data pipeline:**
- `CountryLoader.js` fetches TopoJSON (50m, sourced from world-atlas@2 dataset) from `public/data/countries-50m.json` (bundled statically, not an npm dep) and converts to GeoJSON via custom arc-stitching parser (delta-decode → stitch → GeoJSON features).
- `SubdivisionLoader.js` fetches Natural Earth 50m admin1 GeoJSON (~2.3 MB) from `public/data/ne_50m_admin_1_states_provinces.geojson`. Both loaders fetched in parallel; subdivision failure is non-fatal.
- Data in `public/data/`, URLs use `import.meta.env.BASE_URL` for dev/GitHub Pages compatibility.
- `public/CNAME` contains custom domain (`thetruesize3d.com`); Vite copies it to `dist/` on build.
- `CountrySelector.js` holds a unified `items` list of `{ id, name, displayName, getFeature }` — countries and subdivisions share one search field.

**Rendering (radius layering: overlay fills 5.005 → country fills 5.01 → country borders 5.011 → subdivision borders 5.012 → subdivision hit meshes 5.015):**
- `Globe.js` manages the Three.js scene (camera, renderer, OrbitControls, lighting). Globe sphere radius **5.0**. Country fills use deterministic colors (keyed by `properties.name` in `countryColorMap`). Polygons triangulated via tangent-plane projection at centroid, then subdivided for sphere curvature (handles polar polygons like Antarctica). `countryFillMeshes` and `subdivisionHitMeshes` raycasted on `mousemove` for hover tooltips; raycasting checks globe sphere to suppress far-side hits.
- `CountryOverlay.js` manages **3 overlay slots** (red/blue/green), each a `THREE.Group`. Each slot stores `originalCentroidDir`, `displayNDC` (screen NDC), and `userRotation` (radians). `update()` re-projects `displayNDC` via ray–sphere intersection — overlays are fixed to screen position, globe rotates underneath. Rotation applied as negated angle around camera direction (`makeRotationAxis(cameraDir, -slot.userRotation)`). Fill meshes carry `userData.slotIndex` for drag raycasting.
- Controls panel (top-left): title/help header, per-slot search rows, attribution footer. On mobile (`max-width: 768px`) starts expanded; `#mobile-toggle` ("▲ Hide" / "▼ Menu") toggles `.mobile-collapsed` class.

**Per-slot compass dials (main.js):**
- 36×36px inline SVG compass per slot (hidden until region selected). Rotating `<g>` with tick marks and "N" label. Dragging (mouse or touch) calls `overlay.setRotation(slotIndex, radians)`.

**Coordinate math** (`src/utils/geoUtils.js`):
- `latLonToVector3(lat, lon, radius)` — spherical → Cartesian. GeoJSON coordinates are WGS84 `[lon, lat]`.

## Key Patterns

- Geo data classes: `load()` (async fetch + parse), `getXList()` (sorted `{id, name, displayName}`), `getXById(id)`.
- `CountryOverlay._localFrame(dir)` computes `{right, up, forward}` tangent frame on unit sphere for camera alignment.
- Overlay drag: `mousedown`/`touchstart` on fill mesh → sets `draggingSlot`, disables OrbitControls. `mousemove`/`touchmove` → `setDisplayNDC()`. `mouseup`/`touchend` → re-enables OrbitControls.
- Known limitation: antimeridian-crossing regions (±180°, e.g. Russia) may render with gaps.

## SEO Files

Static files in `public/` for search engine visibility:
- `public/robots.txt` — allows all crawlers, references sitemap.
- `public/sitemap.xml` — canonical URL with monthly update frequency.
- `public/og-image.png` — 1200×630 social preview image for OG/Twitter Card tags.

`index.html` `<head>`: meta description, keywords, canonical URL, Open Graph, Twitter Card, theme-color, `WebApplication` JSON-LD. `<noscript>` block provides SEO fallback HTML.

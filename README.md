# TheTrueSize3D - Interactive 3D Globe

An interactive 3D globe that lets you compare the true sizes of countries and regions by overlaying them on each other. Inspired by [thetruesize.com](https://thetruesize.com), it addresses the common misconception about country sizes caused by Mercator projection distortion.

**Live site: [thetruesize3d.com](https://thetruesize3d.com)**

## Features

- **Interactive 3D Globe**: Rotate and zoom with mouse or touch controls
- **Country & Subdivision Visualization**: Countries rendered with deterministic colors and black outlines; admin1 subdivision borders drawn on top
- **Hover Tooltips**: Hover over any country or subdivision to see its name
- **Multiple Overlays**: Compare up to 5 regions simultaneously in distinct colors
- **Screen-Fixed Overlays**: Overlays stay at their screen position while the globe rotates freely underneath
- **Center On**: Search any country or region to animate the globe to face that location
- **Drag to Reposition**: Click/tap and drag any overlay to move it on screen
- **Per-Slot Compass Dials**: Each overlay has its own compass — drag to rotate, double-click to reset to north-up
- **Search**: Per-slot search covering both countries and admin1 subdivisions; supports arrow key + Enter keyboard navigation
- **Mobile-Optimized**: Collapsible controls panel; pinch-to-zoom works via OrbitControls
- **Help Panel**: Click `?` to reveal an in-app usage guide

## Technology Stack

- **Three.js** — 3D rendering with WebGL
- **Vite** — Development server and build tool
- **Vanilla JavaScript** — No framework overhead
- **world-atlas** — TopoJSON country boundaries (bundled locally, not fetched from CDN)
- **Natural Earth 50m admin1** — Subdivision boundaries (bundled locally)

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- npm

### Installation

1. Install dependencies:
```bash
npm install
```

2. Start the development server:
```bash
npm run dev
```

Open your browser to `http://localhost:5173`.

### Deploy to GitHub Pages

Deployment is automated — every push to `main` triggers a GitHub Actions workflow (`.github/workflows/deploy.yml`) that builds and deploys to GitHub Pages.

## Usage

1. **Center On**: Type a country or region in the "Center on" field to animate the globe to face that location (border turns green while animating; manually dragging clears it)
2. **Select a Region**: Use the slot search field to find any country or admin1 subdivision and overlay it
3. **Add More Overlays**: Click **+ Add Overlay** to add more regions (up to 5)
4. **Drag Overlays**: Click/tap and drag any overlay to reposition it; the globe spins freely underneath
5. **Compare Sizes**: Rotate the globe while overlays stay screen-fixed to compare across regions
6. **Rotate Overlay**: Drag a slot's compass dial to rotate that overlay independently
7. **Remove a Slot**: Click `×` next to a search field to clear that overlay
8. **Mobile**: On mobile the controls panel starts expanded — tap **▲ Hide** to collapse, **▼ Menu** to expand
9. **Help**: Click `?` at the top of the controls panel for the usage guide
10. **Explore**: Try overlaying Greenland on Africa to see how Mercator projection distorts sizes!

## Project Structure

```
thetruesize3d/
├── package.json
├── vite.config.js            # Vite config (base path for GitHub Pages)
├── index.html                # HTML structure, UI, and SEO meta tags
├── public/
│   ├── CNAME                 # Custom domain for GitHub Pages
│   ├── robots.txt            # Search engine crawler instructions
│   ├── sitemap.xml           # XML sitemap for search engines
│   ├── og-image.png          # Social preview image (1200×630)
│   └── data/
│       ├── countries-50m.json                      # world-atlas TopoJSON
│       └── ne_50m_admin_1_states_provinces.geojson  # Natural Earth admin1
├── src/
│   ├── main.js               # App entry and orchestration
│   ├── Globe.js              # Three.js scene and globe rendering
│   ├── CountryLoader.js      # TopoJSON fetch and GeoJSON conversion
│   ├── SubdivisionLoader.js  # Admin1 GeoJSON fetch and parse
│   ├── CountryOverlay.js     # Overlay rendering and positioning
│   ├── CountrySelector.js    # Unified search and selection
│   ├── styles.css            # UI styling
│   └── utils/
│       └── geoUtils.js       # Coordinate transformation utilities
├── .github/workflows/
│   └── deploy.yml            # Auto-deploy on push to main
├── CLAUDE.md                 # Claude Code guidance
└── README.md
```

## Data Sources

- Country boundaries: [world-atlas](https://github.com/topojson/world-atlas) (TopoJSON, 50m resolution) — bundled in `public/data/`
- Subdivision boundaries: [Natural Earth 50m admin1](https://www.naturalearthdata.com/) — bundled in `public/data/`

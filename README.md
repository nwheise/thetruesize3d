# TheTrueSize3D - Interactive 3D Globe

An interactive 3D globe web application that allows users to compare the true sizes of countries and regions by overlaying them on each other. Inspired by [thetruesize.com](https://thetruesize.com), it addresses the common misconception about country sizes caused by traditional 2D map projections (Mercator distortion).

## Features

- **Mobile-Optimized UI**: Collapsible controls panel on mobile so the globe fills the screen; pinch-to-zoom works correctly via OrbitControls
- **Interactive 3D Globe**: Rotate and zoom with mouse/touch controls
- **Country & Subdivision Visualization**: Countries rendered with deterministic colors and flush black outlines; admin1 subdivision borders drawn on top, color-matched to their country
- **Hover Tooltips**: Hover over any country or subdivision on the globe to see its name in a tooltip
- **High-Resolution Borders**: 50m-resolution country borders for detailed coastlines
- **Multiple Overlays**: Compare up to 3 regions simultaneously — each shown in a distinct color (red, blue, green)
- **Screen-Fixed Overlays**: Overlays stay at their screen position while the globe rotates freely underneath
- **Drag to Reposition**: Click and drag any overlay to a new position on the screen
- **Search**: Per-slot search field covering both countries and admin1 subdivisions
- **Per-Slot Compass Dials**: Each overlay slot has its own mini compass — drag it to rotate that overlay independently
- **Help Panel**: Click `?` in the controls panel to reveal a usage guide
- **Real Geographic Data**: world-atlas TopoJSON (countries) + Natural Earth 50m admin1 GeoJSON (subdivisions)

## Technology Stack

- **Vite** - Fast development server and build tool
- **three.js** - 3D rendering with WebGL
- **OrbitControls** - Smooth mouse/touch rotation
- **world-atlas** - TopoJSON country boundaries (bundled locally)
- **Natural Earth 50m admin1** - Subdivision boundaries (bundled locally)
- **Vanilla JavaScript** - No framework overhead

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

3. Open your browser to `http://localhost:5173`

### Build for Production

```bash
npm run build
```

The production files will be in the `dist/` directory.

### Deploy to GitHub Pages

The live site is hosted at **https://thetruesize3d.com**.

Deployment is automated — every push or merge to `main` triggers a GitHub Actions workflow (`.github/workflows/deploy.yml`) that builds the project and deploys to GitHub Pages.

**First-time setup** (only needed once):

1. Push the repository to GitHub.
2. Go to **Settings → Pages** in the GitHub repo.
3. Under "Build and deployment", set **Source** to **"GitHub Actions"**.
4. Click **Save**.

**Manual deploy (legacy):**

```bash
npm run deploy
```

This builds the project and publishes the `dist/` folder to the `gh-pages` branch. This is no longer needed with the GitHub Actions workflow in place.

> **Note:** Geographic data files are bundled in `public/data/` and served from the same origin — no external CDN fetches at runtime.

## Usage

1. **Select a Region**: Use the search field in the controls panel to find any country or admin1 subdivision
2. **Add More Overlays**: Click **+ Add Overlay** to add a second or third region (up to 3, in red/blue/green)
3. **Drag Overlays**: Click and drag any overlay to reposition it on the screen; the globe spins freely underneath
4. **Compare Sizes**: Rotate the globe while the overlays stay screen-fixed to compare sizes across different regions
5. **Rotate Overlay**: Drag a slot's compass dial to rotate that overlay independently around the view axis
6. **Remove a Slot**: Click `×` next to a search field to clear that overlay
7. **Mobile**: On mobile devices the controls panel starts collapsed — tap **▼** to expand it; tap **▲** to collapse and return the globe to full-screen view
8. **Help**: Click the `?` button at the top of the controls panel to show/hide the usage guide
8. **Explore**: Try overlaying Greenland on Africa to see how Mercator projection distorts sizes!

## Project Structure

```
thetruesize3d/
├── package.json
├── vite.config.js          # Vite config (sets base path for GitHub Pages)
├── index.html              # HTML structure and UI
├── public/
│   ├── CNAME                                         # Custom domain for GitHub Pages
│   └── data/
│       ├── countries-50m.json                      # world-atlas TopoJSON (50m)
│       └── ne_50m_admin_1_states_provinces.geojson # Natural Earth admin1 GeoJSON
├── src/
│   ├── main.js            # Application entry and orchestration (TheTrueSize3DApp)
│   ├── Globe.js           # Three.js scene and globe rendering (radius 5.0)
│   ├── CountryLoader.js   # TopoJSON fetch, arc-stitch, and GeoJSON conversion
│   ├── SubdivisionLoader.js # Natural Earth admin1 GeoJSON fetch and parse
│   ├── CountryOverlay.js  # Overlay mesh at radius 5.005, camera-locked orientation
│   ├── CountrySelector.js # Unified country+subdivision search and selection
│   ├── utils/
│   │   └── geoUtils.js    # Coordinate transformation utilities
│   └── styles.css         # UI styling
├── .github/
│   └── workflows/
│       └── deploy.yml     # Auto-deploy to GitHub Pages on push to main
├── CLAUDE.md              # Claude Code guidance
└── README.md
```

## Key Implementation Details

### Coordinate Transformation

Geographic coordinates (latitude, longitude) to 3D Cartesian coordinates on a sphere:

```javascript
function latLonToVector3(lat, lon, radius) {
  const phi = (90 - lat) * (Math.PI / 180);      // Polar angle
  const theta = (lon + 180) * (Math.PI / 180);   // Azimuthal angle

  const x = -radius * Math.sin(phi) * Math.cos(theta);
  const y = radius * Math.cos(phi);
  const z = radius * Math.sin(phi) * Math.sin(theta);

  return new THREE.Vector3(x, y, z);
}
```

### Overlay System

- Globe mesh: radius **5.0**; camera orbits around it via OrbitControls
- Up to 3 overlay slots, each a `THREE.Group` at radius **5.005**
- Each slot stores `originalCentroidDir` (fixed world-space centroid) and `displayNDC` (screen-space `THREE.Vector2`)
- Every frame, `displayNDC` is re-projected to a world-space direction via analytic ray–sphere intersection against the globe sphere (radius 5.0) using the current camera — this is what keeps overlays fixed on screen while the globe rotates
- Rotation matrix maps `originalCentroidDir → targetDir`; each slot's `userRotation` applied independently as `makeRotationAxis(cameraDir, -slot.userRotation)`
- Overlay colors: red (`0xff3333`), blue (`0x4488ff`), green (`0x33cc66`)

### Data Pipeline

Both data sources are fetched in parallel on startup. Subdivision failure is non-fatal — the app continues with countries only.

## Known Limitations

- Countries/regions crossing the antimeridian (±180° longitude) like Russia may have rendering gaps

## Credits

- Inspired by [thetruesize.com](https://thetruesize.com)

## Data Sources

- Country boundaries: [world-atlas](https://github.com/topojson/world-atlas) (TopoJSON, 50m resolution) — bundled in `public/data/`
- Subdivision boundaries: [Natural Earth 50m admin1](https://www.naturalearthdata.com/) — bundled in `public/data/`

## License

ISC

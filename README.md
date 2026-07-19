# NYC Hydrant Density Map

Live map: [https://pamelaagreen.github.io/nyc-hydrant-map/](https://pamelaagreen.github.io/nyc-hydrant-map/)

A web map showing NYC fire hydrant density by neighborhood, built with a fully open-source, serverless stack: MapLibre GL JS + PMTiles + GitHub Pages.

## The question

Where is hydrant coverage densest in NYC, and which neighborhoods are relatively underserved compared with their area?

## Repository contents

This repo contains two main pieces:

1. `generate_pmtiles.sh`  
   A shell script that converts the neighborhood-density GeoJSON into a PMTiles archive for efficient web delivery.

2. `index.html`  
   A static MapLibre map that loads the PMTiles file directly from GitHub Pages and styles neighborhood polygons by hydrants per km².

## The data

- NYC Neighborhoods: 262 polygons (NYC Open Data)
- NYC Fire Hydrants: 109,725 points (NYC Open Data)
- Density metric: hydrants per km²
- Density processing was completed upstream in PP2 using PostGIS and GeoPandas.

## Why this stack

- **MapLibre GL JS** for rendering interactive vector maps in the browser, with no vendor lock-in.
- **PMTiles** for the thematic layer, a single-file tiled archive that can be hosted on static storage without a tile server.
- **tippecanoe** for vector tile generation, including explicit layer naming and optimization for browser rendering.
- **GitHub Pages** for hosting, which works well for static apps and PMTiles range requests when files are publicly accessible.

Total monthly cost: $0  
Total servers running: 0

## Workflow

### 1. Generate the PMTiles archive

The shell script takes the precomputed neighborhood density GeoJSON and creates a `.pmtiles` file ready for MapLibre.

Example (from the repo root):

```bash
./generate_pmtiles.sh
```

Expected output:

```text
nyc_neighborhood_hydrant_density.pmtiles
```

### 2. Serve or publish the map

For local testing:

```bash
python3 -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

For production, publish the repo with GitHub Pages and host both:

- `index.html`
- `nyc_neighborhood_hydrant_density.pmtiles`

in the same site/repo path.

## Map configuration notes

The MapLibre app depends on three values being correct:

1. The PMTiles URL must point to the publicly hosted `.pmtiles` file.
2. The MapLibre layer `"source-layer"` must match the internal vector layer name inside the PMTiles archive.
3. The style expression field names must match the actual feature attributes inside the archive.

For this tileset:

- PMTiles file: `nyc_neighborhood_hydrant_density.pmtiles`
- Internal vector layer: `neighborhood_density`
- Density field used for styling: `hydrants_per_km2`

## Lessons learned

### PMTiles generation

- The PMTiles file can be valid and load correctly, but the browser map will still render nothing if the internal vector layer name is not entered exactly as `"source-layer"` in MapLibre.
- The PMTiles viewer (pmtiles.io) is the fastest way to verify both the internal layer name and the feature attribute names before debugging the web map.
- If the basemap loads but the thematic layer does not, the problem is often not hosting; it’s usually a mismatch in `"source-layer"` or in the property name used in the paint expression.
- Browser DevTools help confirm that the `.pmtiles` file is loading successfully via HTTP range requests, which narrows debugging to style configuration rather than file access.

## Debugging checklist

If the PMTiles layer does not appear:
1. Open browser DevTools and check the **Network** tab for successful `.pmtiles` requests.
2. Confirm the file is publicly reachable from its GitHub Pages URL.
3. Open the same URL in the [PMTiles viewer](https://pmtiles.io/).
4. Check:
   - `vector_layers[].id` → use this as `"source-layer"`.
   - Feature properties → use these exact names in `["get", "..."]`.
5. If needed, add a temporary solid-color debug layer to confirm geometry is drawing before applying data‑driven styling.

## Stack

- MapLibre GL JS
- PMTiles JS
- tippecanoe
- GitHub Pages

## Acknowledgments

- NYC Open Data for the source datasets.
- The MapLibre and PMTiles open-source projects for making static, low-cost geospatial publishing practical.
# Postal Code to Commuting Zone Assignment

This project contains the notebooks and data for mapping postal codes to Meta (Facebook) Commuting Zones (FBCZs) for the UK and Germany.

## Assignment Logic

Both country notebooks follow the same core algorithm:

1. **Load postal code polygons** from shapefiles (one polygon per postal code area).
2. **Load commuting zone definitions** from `meta_commuting_zones.csv`, which contains WKT geometries for Meta's Facebook Commuting Zones. For the UK, multiple rows per zone are consolidated into a single geometry via `unary_union`.
3. **Compute postal code centroids** by projecting to a local meter-based CRS (UK: EPSG:27700 British National Grid; DE: EPSG:25832 UTM zone 32N), computing the centroid, then projecting back to WGS84.
4. **Spatial join** (`sjoin`, predicate `within`): assign each postal code centroid to the commuting zone polygon it falls inside.
5. **Nearest-neighbor fallback**: any centroid that doesn't land inside a commuting zone polygon is assigned to the nearest zone by haversine (great-circle) distance between the unmatched centroid and commuting zone centroids, using `sklearn.neighbors.NearestNeighbors` with a ball tree. Typically this happens to coastal postal codes. The shapefiles tend to have some overlap with water which can drag the centroid out into the ocean.
6. **Export** a CSV mapping (`PostCode -> NumericID, CommutingZoneID, CommutingZoneName`) and a GeoJSON of the commuting zones with postal code counts.

## Country-Specific Notes

| | UK | Germany |
|---|---|---|
| Notebook | `uk_commuting_zones.ipynb` | `de_commuting_zones.ipynb` |
| Postal code count | ~2,700 | ~8,170 |
| Commuting zones | 85 | 168 |
| Projection CRS | EPSG:27700 | EPSG:25832 |
| Zone consolidation | Yes (multi-row per zone) | No (single row per zone) |

## Shapefile Sources

- **UK postal code polygons** (`uk.geojson`): [missinglink/uk-postcode-polygons](https://github.com/missinglink/uk-postcode-polygons/tree/master) on GitHub (FYI in the latest shapefiles the BT postal area is missing, which encompasses Northern Ireland). . 
- **Germany postal code polygons** (`plz-5stellig.*`): [Germany PLZ dataset](https://www.kaggle.com/datasets/jonaslneri/germany-plz/data) on Kaggle. Five-digit PLZ boundaries.
- **Commuting zone definitions** (`meta_commuting_zones.csv`): [Meta's Facebook Commuting Zones (FBCZs)](https://ai.meta.com/ai-for-good/datasets/commuting-zones/), containing zone IDs, names, population, area, and WKT geometries.

## Outputs

| File | Description |
|---|---|
| `uk_postal_code_to_commuting_zone.csv` | UK postcode-to-zone mapping |
| `de_postal_code_to_commuting_zone.csv` | German PLZ-to-zone mapping |
| `uk_commuting_zones.geojson` | UK zone geometries with postal code counts |
| `de_commuting_zones.geojson` | German zone geometries with postal code counts |

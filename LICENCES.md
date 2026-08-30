# Licensing

This dataset is **mixed-licence** and cannot be deposited under a single blanket
licence such as CC0 or CC BY. Most of it derives from OpenStreetMap, whose
licence carries a share-alike condition that CC0 and CC BY do not satisfy.

Every feature also carries its own `licence` property, so a layer lifted out of
this folder still states its terms.

## By file

| Files | Derived from | Licence |
| --- | --- | --- |
| `rivers`, `lhende-khola`, `tributaries`, `downstream`, `settlements`, `facilities`, `bridges`, `roads`, `hydropower`, `waypoints` | OpenStreetMap via Overpass | **ODbL 1.0**, © OpenStreetMap contributors |
| `reach-bands`, `source-rings`, `source-to-station`, `ganga-confluence`, `channel-composition.json` | OpenStreetMap and USGS | **ODbL 1.0** (derivative database of OSM) |
| `usgs-event` | USGS FDSN event service | Public domain (U.S. Geological Survey) |
| `station`, `seismic-summary.json` | EarthScope FDSN | Open data, attribution requested |
| `cross-sections`, `valley-profile.json` | Terrarium terrain tiles, AWS Open Data (SRTM-derived over Nepal) | Public domain / ODbL by source tile. Over Nepal the source is SRTM, public domain. |
| `exposure.json` | WorldPop, plus the OSM-derived channel | **CC BY 4.0** (WorldPop) and **ODbL 1.0** (channel) |
| `arrival-times.json` | OSM-derived chainage, USGS origin time, two quoted reports | **ODbL 1.0**, with quoted figures attributed |
| `gauges` | ICIMOD media advisory | Figures quoted with attribution |

## Attribution

- Contains information from **OpenStreetMap**, © OpenStreetMap contributors,
  available under the Open Database Licence, <https://www.openstreetmap.org/copyright>.
- Seismic event solution from the **U.S. Geological Survey**, event `us7000tbwb`.
- Waveform and station metadata from **EarthScope** FDSN services,
  <https://service.earthscope.org/>.
- Elevation from **Terrarium terrain tiles** on AWS Open Data,
  <https://registry.opendata.aws/terrain-tiles/>.
- Population from **WorldPop**: Bondarenko M., Priyatikanto R.,
  Tejedor-Garavito N., Zhang W., McKeen T., Cunningham A., Woods T., Hilton J.,
  Cihan D., Nosatiuk B., Brinkhoff T., Tatem A., Sorichetta A. 2025,
  *Constrained estimates of 2015-2030 population*, WorldPop, University of
  Southampton. CC BY 4.0.
- River-level figures quoted from an **ICIMOD** media advisory, 26 August 2026,
  and from **Al Jazeera**, 27 August 2026.

## Depositing this

Choose per-file or "Other (Open)" licensing rather than a single repository
default. Zenodo, Dryad and PANGAEA all support this. Selecting CC0 or CC BY for
the whole deposit would misstate the terms of the OpenStreetMap-derived files.

`README.md`, `METHODS.md` and this file are the authors' own text, licensed
**CC BY 4.0**. The ODbL terms above govern the data, not the documentation.

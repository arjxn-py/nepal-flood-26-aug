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

## The review dataset, `data/nepal-flood-2026-review/`

These layers come from the assessments published after the first map was built.
They add two licences the first dataset did not carry, one of them a share-alike
that is **not** the same share-alike as ODbL, so they cannot simply be merged.

| Files | Published by | Licence |
| --- | --- | --- |
| `extent-hot`, `destroyed-features`, `bridges`, `schools`, `mapping-projects` | Humanitarian OpenStreetMap Team, rebuilt from OpenStreetMap | **ODbL 1.0**, © OpenStreetMap contributors |
| `copernicus-grading`, `not-analysed` | Copernicus Emergency Management Service, EMSR927 | **CC BY**, © 2026 European Union |
| `extent-unosat`, `barrier-lakes`, `detachment-zone` | United Nations Satellite Centre (UNOSAT), FL20260826NPL | **CC BY-SA** |
| `comparison.json`, the `km`, `off_m`, `band` and `area_*` fields | Measured here, on the channel from `data/nepal-flood-2026/` | **ODbL 1.0** (derivative of the OSM-derived channel) |

**On versions.** HDX states the Copernicus and UNOSAT terms as "CC BY" and
"CC BY-SA" without a version number, and neither publisher's dataset page gives
one. The `licence` property on each feature repeats that ambiguity rather than
resolving it, because guessing a version would be inventing terms on a
publisher's behalf. Check with the publisher before relying on a specific
version.

**On combining them.** ODbL and CC BY-SA both impose share-alike, and they are
not compatible with each other. A derived layer that merges an OSM-derived file
with a UNOSAT file has no clean licence, so nothing here does that: the layers
are kept separate and the comparisons between them live in `comparison.json` as
numbers rather than as merged geometry.

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
- Flood extent, damage mapping and Tasking Manager boundaries from the
  **Humanitarian OpenStreetMap Team**, rebuilt from OpenStreetMap,
  <https://data.humdata.org/dataset/hot_flood_npl>.
- Damage grading from the **Copernicus Emergency Management Service**,
  activation EMSR927, © 2026 European Union.
- Flood extent, detachment zone and barrier lakes from the **United Nations
  Satellite Centre (UNOSAT)**, code FL20260826NPL. UNOSAT state their barrier
  lake analysis is preliminary and not validated in the field.
- Radar imagery in `s1_pair.py` is **Copernicus Sentinel data 2026**, processed
  to radiometric terrain correction and served by the Microsoft Planetary
  Computer. The script downloads it; none of it is redistributed here.

## Depositing this

Choose per-file or "Other (Open)" licensing rather than a single repository
default. Zenodo, Dryad and PANGAEA all support this. Selecting CC0 or CC BY for
the whole deposit would misstate the terms of the OpenStreetMap-derived files.

`README.md`, `METHODS.md` and this file are the authors' own text, licensed
**CC BY 4.0**. The ODbL terms above govern the data, not the documentation.

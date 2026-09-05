# Methods

How the derived files in `data/nepal-flood-2026/` were made: the thresholds,
sampling intervals and assumptions behind each one.

The generating code is not distributed with this dataset, so this document
stands in for it. It is written to be enough to judge the numbers, to see what
each one does and does not claim, and to reimplement the derivations closely.
It is not a substitute for the code, and working from the same inputs will not
reproduce these files exactly.

Each feature also carries `source`, `source_url`, `licence` and `retrieved` in
its own properties, so a single layer lifted out of the folder still says where
it came from. Fetched layers were retrieved on **2026-08-28**.

## Inputs

| Input | What was taken |
| --- | --- |
| USGS FDSN event service, event `us7000tbwb` | Origin time, location, magnitude. First issued as an earthquake, re-typed **landslide** after review. M 5.2 `ms_vx`, a surface-wave magnitude. |
| EarthScope FDSN station and dataselect | Station `NK.KKN..BHZ` at Kakani, and its waveform for the event window. |
| OpenStreetMap, via the Overpass API | Rivers, roads, bridges, settlements, schools and health facilities, hydropower. |
| Terrarium terrain tiles, AWS Open Data | Elevation. Zoom 13, about 17 m per pixel at this latitude. SRTM-derived over Nepal. |
| WorldPop, Nepal 2026, constrained 100 m counts (R2025A) | Population, for the exposure totals only. The raster itself is not redistributed here. |
| ICIMOD media advisory, 26 Aug 2026; Al Jazeera, 27 Aug 2026 | Two reported river-level times, quoted as facts. |

## The channel

Everything with a chainage is measured along one line. It is walked from the
head of the Lhende Khola, the tributary the collapse came down, through the
Bhote Koshi, the Trishuli and the Narayani, to where the Gandak reaches the
Ganga.

OpenStreetMap splits a river into ways that meet end to end but arrive in no
order and point in either direction. The ways were chained greedily: from the
current end, take whichever remaining way starts or ends nearest, flipping it if
needed, and stop when the nearest is further than **0.3 km** away, which is
wider than a way junction and narrower than a different river.

Ways were matched on `name` and `name:en` against the spellings
`Lhende`, `लेन्डे`, `Bhote`, `Bhotekoshi`, `Trishuli`, `Trisuli`, `Narayani`.
OpenStreetMap spells the Trishuli three ways along this river; the aliases fold
them together.

Chainage is the cumulative haversine distance between consecutive vertices, one
value per vertex. The chained channel is **210.0 km** and every `km_*` property
in the dataset is kilometres along it.

`ganga-confluence.geojson` is not a coordinate typed off a map: it is the end of
the Gandak that comes closest to the Ganga in OpenStreetMap.

`source-to-station.geojson` is the great-circle line between the USGS solution
and station NK.KKN, 57.1 km.

## Terrain, and the gradients the bands are coloured by

`valley-profile.json`, `valley-profile.png`, `cross-sections.geojson`

The long profile steps down the channel every **200 m**. At each step the bed is
taken as the **lowest of 7 samples** across a transect **±60 m** perpendicular to
the local channel bearing, because a 30 m DEM rarely puts a pixel centre in the
bottom of a gorge; the single nearest pixel would sit on a valley side and the
profile would be too high and too rough.

`valley-profile.json` carries three gradient figures measured over **different
stretches**, which is easy to misread, so `gradient_definitions` in that file
names each one:

| Key | Measured over | Value |
| --- | --- | --- |
| `upper_gradient_m_per_km` | channel head to 20 km | 42.5 |
| `lower_gradient_m_per_km` | the last 100 km | 1.9 |
| `band_gradients_m_per_km` | one per reach band | upper 42.5, middle 24.3, lower 2.6 |

The band gradients are the ones the map uses. Each is the drop between the
band's start and end chainage, interpolated on the profile, divided by the
band's length.

Cross-sections run **±900 m** either side of the channel at **15 m** spacing.
The reported floor width is how wide the valley floor is within **10 m** of the
bed elevation. That is a terrain measure available at every section, reported or
not: **it is not an observed flood width**, and the two width figures the story
quotes are captioned as computed rather than observed.

## Reach bands

`reach-bands.geojson`

The channel is cut at **20 km** and **60 km** into three bands. Colour stands
for the **measured gradient of the riverbed** and nothing else. It is not a
damage or severity layer: no verified flood footprint was published for this
event, and none is included here.

The breaks are where the bed gradient changes character, which is also what
decides whether the water arrives as a wall in minutes or a rise over hours.

## Distance rings

`source-rings.geojson`

Discs and annuli at **10, 25 and 50 km** around the USGS solution, drawn as
240-point geodesic circles. A hole is wound opposite to the ring containing it.
Rendered in one neutral hue at three opacities, deliberately: distance is not an
intensity and must not be read as one.

These are straight-line distance from the seismic solution. **They are not an
uncertainty estimate.** The USGS origin product for `us7000tbwb` publishes
`horizontal-error: 0`, `latitude-error: 0.0000` and `longitude-error: 0.0000`,
which are placeholders rather than measurements, alongside a standard error of
11.91 s. There is no radius that could honestly be drawn, so none is.

## Things near the channel

- `tributaries.geojson` — a river counts as reaching the channel if it comes
  within **1.0 km** of it. Each carries `km_confluence`, the chainage of its
  junction, and a note on backwater: a main-stem surge backs up into tributary
  mouths as well as tributaries adding water going down.
- `settlements.geojson`, `facilities.geojson`, `bridges.geojson`,
  `roads.geojson`, `hydropower.geojson` — within a **3 km** corridor of the
  channel. Presence in the corridor is proximity, not damage.
- `waypoints.geojson` — named places on the channel with their chainage.
- `downstream.geojson` — the Narayani, Gandak and Ganga, fetched by name within
  a bounding box below the corridor and drawn as OpenStreetMap has them. These
  are not chained into the channel and carry no chainage; they are there to show
  where the water goes after the mapped reach ends.
- `gauges.geojson` — the two places ICIMOD reported a river level for. The
  figures are quoted from the advisory with attribution; they are not a
  measurement made here.

## Channel composition

`channel-composition.json`

For each segment of the chained channel, the midpoint is assigned to whichever
named river way is nearest, sampling every third vertex of each candidate way.
Segment lengths are summed per name. This is what supports the statement that
the event is named for the Bhote Koshi but most of the mapped channel is
Trishuli.

## Arrival times

`arrival-times.json`

**This is the most model-dependent file in the dataset. Treat it as an
illustration of warning time, not as a measurement.**

Origin is **2026-08-26T02:52:10Z**, 08:37 Nepal time. Reporting places the water
at Timure between **08:50 and 09:10** local. Dividing Timure's chainage by those
two elapsed times gives a fastest and slowest celerity over the first reach.
That leg, and only that leg, is anchored on reported observation.

Lower reaches are scaled from the anchored one by the **square root of the ratio
of bed gradients**. Travel time to any chainage is then accumulated reach by
reach.

The `method` field in the file states the limitation and it is repeated here:
the scaling is crude, and it is crudest on the flattest reach, where a flood
wave is governed by channel shape and storage as much as by bed slope. The
measured leg is rounded to **5 minutes** and everything worked out from it to
**15 minutes**, so the rounding shows which numbers are which.

## Exposure

`exposure.json`

Population within **500 m, 1 km and 3 km** of the channel. Buffers are built in
**EPSG:32645**; the raster is read as **EPSG:4326** with nodata **-99999.0**,
both recorded in the file as assumptions rather than as things read from the
file header.

These are **modelled people, not counted people**. WorldPop estimates where
people live from census totals and observed built-up area, and the 2026 layer is
a projection within a 2015-2030 series. Each person is counted once, in the
smallest buffer that contains them.

Population within a distance of the river is **not** a casualty figure and not a
count of people affected. This dataset carries no casualty or damage geometry at
all; Nepal's NDRRMA publishes those.

## Seismic summary

`seismic-summary.json`, `kkn-seismogram.png`

Station `NK.KKN..BHZ`, Kakani, 57.1 km from the solution, 50 Hz. The window runs
from 13 minutes before the origin to 79 minutes after; the instrument response
is removed to velocity with a pre-filter of (0.01, 0.02, 20.0, 24.0) Hz, and the
window is trimmed after response removal so the taper's decay at each edge is
not mistaken for signal.

The envelope is smoothed and expressed in dB above the pre-event noise floor,
taken as the median of the quiet portion. Onset, peak and tail are read off that
envelope: the envelope takes about **40 s** to reach its peak and has not
returned to background **75 minutes** later. An earthquake of this magnitude
rings down in minutes. That difference is the evidence for the event being
re-typed.

## The review layers

`data/nepal-flood-2026-review/` holds the assessments published in the week after
this map was built, put onto the same channel. As with the rest of this deposit
the generating code is not distributed, so this section stands in for it.

### Where each layer comes from

Every file is a published product, fetched from the Humanitarian Data Exchange on
**2026-09-05** and reprojected to EPSG:4326, with the vertices of the polygon
layers thinned to a **5 m** tolerance for drawing. Areas quoted in the attributes
and in `comparison.json` were measured on the unthinned geometry in EPSG:32645.

| File | Published by | What it is |
| --- | --- | --- |
| `extent-hot` | Humanitarian OpenStreetMap Team | The water line interpreted from drone, Landsat, PlanetScope and Sentinel imagery for 27 August. 31.7 km². |
| `extent-unosat` | UNOSAT | The same flood as UNOSAT mapped it, multisensor, 26-28 August. 64.9 km². |
| `destroyed-features`, `bridges`, `schools` | Humanitarian OpenStreetMap Team | Status set by OpenStreetMap volunteers from imagery, rebuilt from OSM. |
| `copernicus-grading` | Copernicus EMS, EMSR927 | Damage graded by photo-interpretation, four areas of interest, plus the AOI 03 re-flight. |
| `not-analysed` | Copernicus EMS, EMSR927 | Ground the analysts could not assess. |
| `mapping-projects` | Humanitarian OpenStreetMap Team | Tasking Manager project areas, with the chainage each spans. |
| `barrier-lakes`, `detachment-zone` | UNOSAT | Two impoundments on CARTOSAT-3 imagery of 28 August, and the source area on Landsat 9. |

### Chainage

`km`, `off_m` and `band` on every feature are measured against the same channel
as the rest of this deposit: the point is projected onto the chained channel in
EPSG:32645, `km` is the distance along it and `off_m` the distance from it. The
reconstructed channel is **209.9 km** against the 210.0 km measured for the first
map, which is the check that the two datasets are on one ruler.

### The tier field

Each feature carries `tier`, which is the strongest claim the data supports:
**observed** for something a person or sensor recorded on a stated date,
**predicted** for a model output not checked in the field, **reported** for a
figure an organisation stated and we quote. No layer here is `reported`; the
reported figures live in prose, attributed.

### The comparisons

`comparison.json` holds the figures the story quotes. Each is a count or a ratio
taken directly from the layers above, with no modelling:

- Bridges destroyed per reach, as a share of bridges present in that reach.
- Destroyed features per kilometre of each reach, and their distance from the
  channel as a median, a 90th percentile and a maximum.
- The Copernicus AOI 03 counts for the initial product and the monitoring round.
- The chainage at which the damage record ends, against the chainage at which the
  Tasking Manager projects change over.

### Two things these layers cannot tell you

**The damage record ends at km 78.0 and the mapping projects change over at
km 78.9.** The lower projects report 100% mapped and validated for buildings,
roads and land use, so this is not simply unmapped ground, and the bridge losses
do fall away downstream on their own. But a record that stops within a kilometre
of a boundary in the mapping campaign cannot settle whether the damage stopped
there too. `mapping-projects.geojson` is included so the question stays visible.

**Four organisations counted destroyed buildings and got four numbers**: 2,813
from Copernicus over four areas of interest, 5,048 affected from UNOSAT over a
much larger analysis extent, 1,641 from OpenStreetMap volunteers, and 677 scored
by HOT's fAIr model inside a single satellite tile. They cover different ground
by different methods with different definitions. None of them is the number, and
none should be divided by another.

## What is not here

- **No flood extent.** Copernicus EMS was activated for this event as
  **EMSR927** but had published no products; NESRA FloodWatch offered a
  dashboard with no machine-readable download. No flood outline is included and
  none should be inferred from the reach bands.
- **No casualty, damage or response geometry.**
- **No uncertainty polygon for the seismic solution**, for the reason given
  under Distance rings.
- **The WorldPop raster itself.** Only derived totals are included. The raster
  is at the URL recorded in `exposure.json`.
- **Any satellite imagery.** The radar check behind the review layers reads a
  Sentinel-1 pair from the Planetary Computer at run time and redistributes
  none of it. No optical before-and-after exists for the damaged reach: every
  Sentinel-2 pass over it between 26 August and 5 September is 70-90% cloud.

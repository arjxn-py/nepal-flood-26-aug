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

Every file but one is a published product, fetched from the Humanitarian Data
Exchange on **2026-09-08** and reprojected to EPSG:4326, with the vertices of the
polygon layers thinned to a **5 m** tolerance for drawing. Areas quoted in the
attributes and in `comparison.json` were measured on the unthinned geometry in
EPSG:32645. The exception is `timed-places`, which is transcribed from a
situation report rather than fetched; it is described under the table.

| File | Published by | What it is |
| --- | --- | --- |
| `extent-hot` | Humanitarian OpenStreetMap Team | The water line interpreted from drone, Landsat, PlanetScope and Sentinel imagery for 27 August. 31.7 km². |
| `extent-unosat` | UNOSAT | The same flood as UNOSAT mapped it, multisensor, 26-28 August. 64.9 km². |
| `destroyed-features`, `bridges`, `schools` | Humanitarian OpenStreetMap Team | Status set by OpenStreetMap volunteers from imagery, rebuilt from OSM. |
| `copernicus-grading` | Copernicus EMS, EMSR927 | Damage graded by photo-interpretation, four areas of interest, plus the AOI 03 re-flight. |
| `not-analysed` | Copernicus EMS, EMSR927 | Ground the analysts could not assess. |
| `mapping-projects` | Humanitarian OpenStreetMap Team | Tasking Manager project areas, with the chainage each spans. |
| `barrier-lakes`, `detachment-zone` | UNOSAT | Two impoundments on CARTOSAT-3 imagery of 28 August, and the source area on Landsat 9. |
| `timed-places` | NDRRMA, on OpenStreetMap locations | The four places situation report 01 times the wave past, with the chainage of each. |

### The timed places

`timed-places.geojson` is four points, and it exists because the story quotes four
arrival times in prose and the reader had nothing on the map to attach them to.

The times are NDRRMA's, transcribed from situation report 01 as published:
Galchhi 10:28, Malekhu 11:50, Mugling 13:00, Devghat 15:20. NDRRMA give no
uncertainty on any of them, and none is a gauge trace.

The coordinates are OpenStreetMap's, not NDRRMA's. Galchhi, Malekhu and Devghat
are on the same OSM nodes the first map's `waypoints.geojson` used, retrieved
2026-08-28. Mugling is not in that file and was looked up separately, on OSM node
`567088223` (`place=hamlet`, `name:en=Mugling`), retrieved 2026-09-07. Each
feature names both, in `source` and in `located_from`, and carries a `licence` for
the times and a `location_licence` for the coordinates.

These four times are **not** the modelled arrival windows in the first map's
`arrival-times.json`, and the two disagree by hours on the lower reach: that file
scales celerity from a single anchored leg by bed gradient, and says so. Where
NDRRMA state an hour, this layer carries NDRRMA's hour.

`label` on each feature is `"Galchhi 10:28"` and so on. It is a field rather than
a caption because a JupyterGIS vector layer has no text channel: the story map
colours the four dots from it and the layer panel builds a legend, but nothing
draws the name on the map.

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
figure an organisation stated and we quote. `timed-places` is the only
`reported` layer here, and it is one because NDRRMA state those four times and
nothing has re-derived them. The other reported figures live in prose,
attributed.

### The comparisons

`comparison.json` holds the figures the story quotes. Each is a count or a ratio
taken directly from the layers above, with no modelling:

- Bridges destroyed per reach, as a share of bridges present in that reach.
- Destroyed features per kilometre of each reach, and their distance from the
  channel as a median, a 90th percentile and a maximum.
- The Copernicus AOI 03 counts for the initial product and the monitoring round.
- The chainage at which the damage record ends, against the chainage at which the
  Tasking Manager projects change over.

### The before and after imagery

`fig-before-after-red/orange/yellow.png`, `fig-detachment-before-after.png`,
`fig-barrier-lakes.png`

Sentinel-2 L2A, 10 m, true colour, from the Element 84 STAC index of the AWS
open data. No single pass after the flood is usable: every one over this
corridor between 26 August and 6 September is **70-90% cloud**. The cloud sits
in a different place on each, so each pass is masked to the pixels its own
scene classification calls clear (classes 2, 4, 5, 6, 7 and 11, minus 3, 8, 9
and 10), and the median is taken across the window. Ground that no pass saw
clearly is drawn as a gap rather than filled in.

The before side is the same operation over passes to 24 August with less than
60% scene cloud, where one nearly clear pass on 12 August does most of the work.
The after window runs from 26 August: to 5 September for the three reach
figures, and to 6 September for the two upper ones, which picks up one further
pass.

Six boxes in five figures:

| Figure | Box | Gap after |
| --- | --- | --- |
| `red` | Timure and Rasuwagadhi, 6 km | 22% |
| `orange` | Syabru Besi, 6 km | 19% |
| `yellow` | below Bidur, 6 km | 18% |
| `detachment` | the source area, 4.5 km | 38% |
| `barrier-lakes` | the two impoundments, 2.8 km each | 40% and 51% |

The two upper figures use a longer brightness stretch than the three reach
figures, because at 4,000 m and above the snow saturates the reach figures'
stretch and takes the rock with it. Neither stretch is radiometric; both sides
of every figure share one, so the two panels can be read against each other.

**What does not improve the two upper boxes.** Their gaps are large and the
cloud that survives the mask is thin cloud lying over snow, which the scene
classification calls snow. Five variants were tried against the six-pass median:
growing the cloud classes by one and two pixels (gap 38% → 40% and 42%, fringes
tidied, nothing recovered); choosing the least hazy clear observation per pixel
instead of the median (introduces dark blotches where the darkest observation is
terrain or cloud shadow); dropping the snow class from the clear set (gap → 59%,
and real snow becomes a hole); the single 27 August pass alone (gap 56%);
and 27 August with 1 September only (gap 39%, indistinguishable from the
six-pass median). The six-pass median is kept. The limit here is the weather in
the window rather than the compositing, and no arrangement of these passes
recovers ground that none of them saw.

Areas and distances quoted alongside them are measured on the UNOSAT polygons in
EPSG:32645: the upper lake is **159 m** from the edge of the detachment polygon,
the lower one **5.1 km** from it and **4.6 km** from the upper.

### Is there water inside the lake outlines

True colour cannot settle it. Fresh rock flour, dry sediment and a silt-laden
lake all read pale. Near infrared can: water absorbs it and the rest of this
ground does not. NDWI, (green - NIR) / (green + NIR), was sampled inside each
UNOSAT lake outline on the same masked-median composites, before and after:

| Outline | Median NDWI before | after | Pixels above zero | Clear after |
| --- | --- | --- | --- | --- |
| upper, 19.5 ha | -0.006 | +0.051 | 39% → 99% | 1,947 of 1,960 |
| lower, 11.9 ha | -0.059 | +0.014 | 38% → 61% | 423 of 1,183 |

Both move the way water would move them, and neither moves far past the zero
threshold usually taken for open water. A silty impoundment in mixed 10 m pixels
in a shadowed valley looks like this, and so does wet sediment. The shift is
evidence of standing water; it is not a measurement of depth, volume or extent,
and the lower outline had a clear look at barely a third of its pixels.

### The radar pair

`fig-radar-change.png`

Sentinel-1 RTC gamma0 from the Microsoft Planetary Computer, VV, **relative
orbit 85 ascending**, 16 and 28 August, both acquired at 12:21 UTC. The matched
orbit is what makes the difference readable. In a gorge this steep a single
radar image is largely a picture of the slope, with shadow where the terrain
faces away and layover where it faces into the beam. Two passes of identical
geometry subtract that away, and what is left is change on the ground. Multilooked by 3, so a displayed pixel is
about 30 m and speckle is settled.

Terrain correction matters a great deal in this valley, which is why the RTC
collection is used rather than GRD.

Smooth surfaces reflect away from the sensor and read dark, and standing water,
wet mud and fresh sand are all smoother than the ground that was there before,
so new water or new deposit shows as a drop. **A drop in backscatter measures
how the surface reflects, and nothing else.** It is quoted here only against the
outlines other people mapped, and it should not be read as water, as sediment or
as damage.

Change from 16 to 28 August, as a share of pixels dropping more than 3 dB:

| Where | Pixels | Median change | Below -3 dB |
| --- | --- | --- | --- |
| the whole box | 72,675 | +0.30 dB | 4.1% |
| UNOSAT detachment zone | 2,168 | -0.33 dB | 29.2% |
| upper barrier lake | 218 | +0.41 dB | 22.5% |
| lower barrier lake | 131 | -4.49 dB | 57.3% |

The detachment zone and the lower lake darken far more than the box around them.
The upper lake stays close to its surroundings, and it is the one where the
optical water index moved most. The two sensors disagree about it, on 218 and
1,960 pixels respectively, and nothing in this dataset settles which of them is
right, so the story reports the disagreement and leaves it open.

28 August is also the day UNOSAT mapped the lakes from CARTOSAT-3, so the
outlines and the second radar pass are the same day.

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
- **Any satellite imagery.** The radar figures and checks read Sentinel-1 from
  the Planetary Computer at run time and redistribute none of it. The optical before-and-afters are rendered figures, not data: the
  Sentinel-2 scenes they composite are read at run time from the AWS open data
  and none is redistributed here.
- **The commercial imagery the agencies worked from.** Pléiades Neo and
  WorldView-3 at 0.3 m, CARTOSAT-3 and SkySat are reserved. The outlines drawn
  from them can be shared; the images cannot, which is why every picture here is
  10 m and cloudy.

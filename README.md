# Bhote Koshi flood, 26 August 2026

A JupyterGIS story map about the flash flood on the Bhote Koshi and Trishuli of
26 August 2026, and the data behind it.

The event was first issued by the USGS as an earthquake and re-typed a
**landslide** after review. A seismometer 57 km away recorded a signal that took
about 40 s to reach its peak and had not returned to background 75 minutes
later, which is not how an earthquake of that magnitude behaves. The story
follows that signal, then follows the water 210 km down four rivers, from the
Lhende Khola to where the Gandak reaches the Ganga, falling 2,215 m.

## Contents

```
nepal-flood-2026.jGIS      the story map, 22 segments
data/nepal-flood-2026/     18 GeoJSON layers, 5 JSON sidecars
METHODS.md                 how the derived files were made, and what is absent
LICENCES.md                per-file licensing and attribution
```

Open `nepal-flood-2026.jGIS` in JupyterGIS. The figures are embedded in the
document; the layers are read from `data/nepal-flood-2026/`, so keep the two
together and do not rename that folder.

## The data

The 18 GeoJSON layers are what the map draws. The 5 JSON sidecars carry the
numbers the story quotes in prose: `valley-profile` (bed gradients),
`seismic-summary` (onset, peak and tail), `arrival-times` (when the water
reached each place), `exposure` (modelled population near the channel) and
`channel-composition` (how much of the 210 km each named river is).

Every feature carries `source`, `source_url`, `licence` and `retrieved` in its
own properties, so a layer lifted out of the folder still says where it came
from. Fetched layers were retrieved on 2026-08-28.

## Before you reuse it

- **The reach colours are bed gradient, not damage**: 42.5, 24.3 and 2.6 m per
  km. This dataset contains no flood footprint, and none was published for the
  event.
- **The arrival times are modelled.** Only the first leg is anchored on
  reported observation; the rest is scaled from it.
- **The exposure figures are modelled people living near a river**, not
  casualties and not people affected.

There is no casualty, damage or response geometry here at all. Nepal's
[NDRRMA](https://bipadportal.gov.np/) publishes those. `METHODS.md` gives the
method behind each number and lists what is deliberately absent, and why.

## About this copy

This is a standalone deposit. The same story map ships as an example inside
[JupyterGIS](https://github.com/geojupyter/jupytergis), alongside the seven
Python scripts that fetch the inputs and write these files.

Those scripts are not distributed here, so `METHODS.md` stands in for them: it
describes each derivation rather than letting you re-run it. Three consequences,
all cosmetic:

- Two passages of the story and one segment title that sent a reader to the
  scripts now point at `METHODS.md`.
- Eight data files cite `METHODS.md` in `source_url` where they cited the script
  that wrote them.
- The final segment is titled "Sources" rather than "Sources, and how to
  rebuild this".

No geometry, measurement or attribution differs from the example's.

## Licensing

Mixed, and it cannot be released under a single blanket licence: most layers are
derivative databases of OpenStreetMap and carry ODbL 1.0's share-alike
condition. See `LICENCES.md` for the per-file breakdown and the attribution
text.

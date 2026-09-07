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
nepal-flood-2026.jGIS           the story map, 22 segments
data/nepal-flood-2026/          18 GeoJSON layers, 5 JSON sidecars
data/nepal-flood-2026-review/   11 layers from the assessments published later,
                                8 figures, and per-layer notes
METHODS.md                      how the derived files were made, and what is absent
LICENCES.md                     per-file licensing and attribution
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

## What was published afterwards

`data/nepal-flood-2026-review/` holds the assessments that appeared in the week
after the first map was built, put onto the same channel so the two can be held
against each other. It is a check on the first map rather than a correction of
it.

Two results, both in `comparison.json`:

- **Bed gradient predicted damage well.** Bridges destroyed run 15 of 16 in the
  steep gorge, 22 of 47 in the middle valley and 6 of 108 on the open valley,
  against measured bed gradients of 42.5, 24.3 and 2.6 m per km.
- **Distance from the channel predicted it badly.** Observed damage sits a
  median 108 m from the water, against the 500 m, 1 km and 3 km buffers the
  first map counted people in, and those counts pointed downstream while the
  damage fell upstream.

Two things to know before quoting any figure out of it. The damage record ends
at km 78.0, and the volunteer mapping projects change over at km 78.9, so
`mapping-projects.geojson` is included to stop the end of the record being read
as the end of the damage. And four organisations counted destroyed buildings
over different ground by different methods, arriving at 2,813, 5,048, 1,641 and
677; none of those is the number, and `layer-notes.json` says what each one
covers.

As with the first dataset, the scripts that fetch the sources and write these
files are not distributed here. `METHODS.md` describes each derivation instead,
and `layer-notes.json` carries the caveat for each layer.

## About this copy

This is a standalone deposit. The first story map also ships as an example
inside [JupyterGIS](https://github.com/geojupyter/jupytergis), alongside the
seven Python scripts that fetch its inputs and write its files.

Those seven are not distributed here, so `METHODS.md` stands in for them: it
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

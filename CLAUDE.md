# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a static data repository — no build system, no tests, no server. All files are raw data resources consumed by external WME (Waze Map Editor) userscripts hosted on greasyfork.org. Files are fetched directly via GitHub raw URLs by those scripts.

## Directory Overview

Each subdirectory serves a specific WME userscript:

- **wme-live-camera-il/** — Live traffic camera data for Israel. `liveCameras.json` is an array of camera objects; `liveCameras.md` is a human-readable table of cameras with WME editor links.
- **wme-sabbath-closures/** — Road closure data for Jewish Sabbath/holidays. Multiple JSON files: `WME_sabbath.Segments.json` (road segments to close), `WME_sabbath.Cities.json` (city Sabbath timing offsets in minutes from Jerusalem), `WME_sabbath.JerusalemTimePerWeek.json`, `WME_sabbath.HolidaysSegments.json`, `WME_sabbath.RevertSegments.json`. Files use UTF-8 with BOM encoding.
- **wme-create-venues/** — Templates/defaults for venue creation actions: `venue.json`, `venue_update.json`, `residential.json`, `deleteVenue.json`, `entryExitPoints.json`, `streets_il.json`.
- **wme-objects-info/** — Reference data for WME objects: `managedAreas.json` (area manager polygons keyed by area ID with GeoJSON geometry), `cities.json`, `streets.json`.
- **wme-gis-il/** — GIS layer mappings: `cities2Gis.json` maps Waze city IDs to GIS server URLs.
- **wme-reply-mur/** — `gad_m.txt` is a Hebrew template text for replying to map update requests.
- **wme-simplex/** — `cities.json` for the wme-simplex script.

## Data Formats

- Most JSON files are plain UTF-8. Sabbath closure files use **UTF-8 with BOM** (`utf-8-sig` in Python).
- `WME_sabbath.Segments.json` — array of objects with `permalink`, `likeCity`, `cityName`, `streetName` fields.
- `WME_sabbath.Cities.json` — dict keyed by city name with `offset` (minutes offset from Jerusalem Sabbath time).
- `managedAreas.json` — dict keyed by area ID string; each value has `id`, `userID`, `name`, `geometry` (GeoJSON Polygon).
- `cities2Gis.json` — `{ "cities": { "<waze_city_id>": { "name": ..., "gis": "<url>" } } }`.
- `liveCameras.json` — array of camera objects (encoding issue: not valid UTF-8, may be latin-1/windows-1252).

## Making Updates

Since there is no build or validation step, edits are made directly to the JSON/text files. After editing:

1. Validate JSON manually: `python3 -m json.tool <file>` (use `python3 -c "import json,codecs; json.load(codecs.open('<file>','r','utf-8-sig'))"` for BOM files).
2. Commit and push — the scripts fetch data live from the GitHub raw URL, so changes take effect immediately after pushing to master.

The consuming userscripts are in the sibling repo under `greasyfork/`.

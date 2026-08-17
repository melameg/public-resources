# CLAUDE.md — wme-gis-il

## Updating cities2Gis.json

### Finding city IDs

Look up Hebrew city names in a `cities.json` file (ask the developer for the path) by matching the `name` field. If an exact match fails, try a partial/fuzzy match (e.g. `חצור אשדוד` is stored as `חצור-אשדוד`).

### Sorting / insertion order

Entries in `cities2Gis.json` are **grouped by `gis` value** — all cities sharing the same GIS server appear together. Within a group, entries are in numeric key order. New entries for a council go at the end of the file as a new group (or appended to an existing group if the `gis` key already exists), sorted numerically by city ID.

## City entry attributes

Each entry under `cities` is keyed by the Waze city ID and may have the following attributes:

- **`gis`** *(required)* — key that identifies the GIS server for this city. Must match a key in the `gis` section of the same file, which holds the base URL and the URL-building method. If no matching key exists in the `gis` section, the value itself is used directly as the URL.
  Example: `"gis": "up.intertown.co.il"`

- **`name`** *(required)* — the city's identifier as used by the GIS server, inserted into the URL path.
  Example: `"name": "btv"` → URL becomes `https://up.intertown.co.il/btv/public/`

- **`displayName`** *(optional)* — English display name shown in the WME interface. Use unencoded apostrophes (e.g. `Giv'Ati`, not `Giv%27Ati`).

- **`suffix`** *(optional)* — appended after the city name in the URL. Used by intertown and Taldor servers. Default for intertown is `"public"` when omitted.
  Example: `"suffix": "public"` → URL becomes `https://up.intertown.co.il/btv/public/`

- **`prefix`** *(optional)* — used by servers whose base URL contains a `__prefix__` placeholder, which gets replaced by this value. Also used by ArcGIS servers to override the default `https://www.` prefix.
  Example: `"prefix": "mg1"` → `__prefix__.gis-net.co.il` becomes `mg1.gis-net.co.il`

- **`isEPSG`** *(optional)* — set to `"true"` for ArcGIS servers that require EPSG:900913 coordinates instead of the default ITM coordinates.

- **`queryParams`** *(optional)* — extra query string parameters appended to the URL, used by v5GisNet servers.
  Example: `"queryParams": "foo=bar"`

## Updating a GIS for a council

Cities belonging to the same regional council share the same `gis` and `name` values. If asked to update the GIS or URL for one city, it almost certainly applies to all cities in that council. Before making the change, find all entries with the same `gis`+`name` combination and confirm with the developer that all of them should be updated.

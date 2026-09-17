# wme-sabbath-closures

Static JSON data consumed by the `wme-sabbath-closures` userscript on greasyfork.org. All files use **UTF-8 with BOM** encoding (`utf-8-sig` in Python).

## WME_sabbath.Segments.json

Array of road segments to close on Sabbath/holidays. 535 entries (as of 2026-07).

### Entry format

```json
{
  "permalink": "&lat=32.07710&lon=34.82772&segments=75086",
  "likeCity": "TelAviv",
  "cityName": "בני ברק",
  "streetName": "אלוף שמחוני"
}
```

**Fields:**
- `permalink` — WME deep-link fragment. Format: `&lat=<lat>&lon=<lon>&segments=<segmentID>`. May include optional `&layers=4`. One segment ID per entry (no comma-separated IDs).
- `likeCity` — English key referencing `WME_sabbath.Cities.json` to determine Sabbath timing zone. Currently all entries use `"TelAviv"`.
- `cityName` — Hebrew city name (display only).
- `streetName` — Hebrew street name (display only).

### Adding new segments

**Required inputs per segment:**
- Waze segment ID
- `lat` and `lon` (WME map coordinates of the segment)
- Hebrew street name
- Hebrew city name

**One entry per segment ID.** If a user requests multiple segments on the same street, add a separate entry for each.

**Insertion:** The file is sorted by `cityName` then `streetName` (Hebrew lexicographic order). Append new entries and then re-sort, or insert in the correct sorted position manually.

**To re-sort after adding:**
```bash
python3 -c "
import json,codecs
path='WME_sabbath.Segments.json'
data=json.load(codecs.open(path,'r','utf-8-sig'))
data.sort(key=lambda e: (e['cityName'], e['streetName']))
with codecs.open(path,'w','utf-8-sig') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
"
```

**To validate no data was lost after editing:**
```bash
python3 -c "
import json,codecs,subprocess
path='WME_sabbath.Segments.json'
current=json.load(codecs.open(path,'r','utf-8-sig'))
raw=subprocess.check_output(['git','show','HEAD:wme-sabbath-closures/WME_sabbath.Segments.json'])
original=json.loads(raw.decode('utf-8-sig'))
before=set((e['permalink'],e['likeCity'],e['cityName'],e['streetName']) for e in original)
after=set((e['permalink'],e['likeCity'],e['cityName'],e['streetName']) for e in current)
print('OK' if before==after else 'MISMATCH', '| before:', len(before), 'after:', len(after))
"
```

**To validate JSON after editing:**
```bash
python3 -c "import json,codecs; json.load(codecs.open('WME_sabbath.Segments.json','r','utf-8-sig')); print('valid')"
```

### Keeping segments.md in sync

`segments.md` is a human-readable RTL Markdown version of `WME_sabbath.Segments.json`. **It must be updated every time `WME_sabbath.Segments.json` changes.** Regenerate it by running:

```bash
python3 -c "
import re, json, codecs

path = 'WME_sabbath.Segments.json'
data = json.load(codecs.open(path, 'r', 'utf-8-sig'))

lines = ['<div dir=\"rtl\">', '', '# רשימת המקטעים שסגורים בשבתות וחגי ישראל', '']
current_city = None
for entry in sorted(data, key=lambda e: (e['cityName'], e['streetName'])):
    if entry['cityName'] != current_city:
        current_city = entry['cityName']
        lines.append(f'### {current_city}')
        lines.append('')
    seg_id = entry['permalink'].split('segments=')[1]
    lat = entry['permalink'].split('lat=')[1].split('&')[0]
    lon = entry['permalink'].split('lon=')[1].split('&')[0]
    url = f'https://www.waze.com/he/editor/?env=il&lon={lon}&lat={lat}&s=3638528&zoom=8&segments={seg_id}'
    lines.append(f'- [{entry[\"streetName\"]}]({url})')
lines.append('')
lines.append('</div>')

with open('segments.md', 'w', encoding='utf-8') as f:
    f.write('\n'.join(lines))
print('segments.md updated')
"
```

### Commit message convention

Include the Waze forum thread URL when a request originates from a forum post, e.g.:
```
Add Sabbath closure segments in <city> (<street>)

Per request: https://www.waze.com/discuss/t/topic/...
```

## Other files

- **WME_sabbath.Cities.json** — dict keyed by Hebrew city name; each value has an `offset` (integer, minutes offset from Jerusalem Sabbath time). The `likeCity` field in Segments.json references this via the English key to determine actual closure times.
- **WME_sabbath.JerusalemTimePerWeek.json** — weekly Sabbath start/end times for Jerusalem.
- **WME_sabbath.HolidaysSegments.json** — segments to close on Jewish holidays (same format as Segments.json).
- **WME_sabbath.RevertSegments.json** — segments to revert/reopen after Sabbath ends.
- **WME_sabbath.Junction_Boxes.json** — big junctions (תיבות צומת) to handle on Sabbath. Dict keyed by junction ID string; each value has `url` (WME permalink with `bigJunctions=<id>`) and `segments` (array of segment IDs, may be empty if unknown).
- **WME_sabbath.Roundabouts.json** — roundabout segments to handle on Sabbath. Array of objects, each with `url` (WME permalink with `segments=<ids>`) and `segments` (array of segment IDs).

## Analysing a WME URL to determine the target file

When the user provides a WME editor URL, determine the target file by the URL parameters:

| URL contains | Target file |
|---|---|
| `bigJunctions=<id>` | `WME_sabbath.Junction_Boxes.json` — add entry keyed by the junction ID with `url` and `segments` (empty array if no segment IDs in the URL) |
| `segments=<ids>` and it is a **roundabout** | `WME_sabbath.Roundabouts.json` — append entry with `url` and `segments` array |
| `segments=<ids>` and it is a regular road segment | `WME_sabbath.Segments.json` — add one entry per segment ID with `permalink`, `likeCity`, `cityName`, `streetName` |

The user will indicate whether a `segments=` URL is a roundabout or a regular segment. Never assume — ask if unclear.

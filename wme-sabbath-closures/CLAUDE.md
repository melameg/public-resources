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

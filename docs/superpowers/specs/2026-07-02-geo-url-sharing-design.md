# Geo Viewer — URL Sharing

**Date:** 2026-07-02

## Goal

Share a geo viewer state (map viewport + all layers) via a URL that can be sent to others.

## URL Format

```
/geo#v=1&s=<lzstring-compressed-base64>
```

- Hash fragment (`#`) — state is never sent to the server.
- `v=1` version prefix — safe path for future format changes.
- `s=` — LZ-string compressed, URL-safe base64 of the JSON state blob.

### State blob schema

```json
{
  "center": [lat, lng],
  "zoom": number,
  "layers": [
    { "id": number, "name": string, "type": "geojson"|"h3", "color": string, "opacity": number, "data": object|array }
  ]
}
```

This is the same shape already used by `saveLayers()` / localStorage.

## Dependencies

- **LZ-string** — loaded via CDN `<script>` tag (same pattern as h3-js). Provides `LZString.compressToEncodedURIComponent` / `decompressFromEncodedURIComponent`.

## State Sync (encode)

- `syncUrl()` is called at the end of the existing `saveLayers()` — no new call sites needed.
- `syncUrl()` also fires on `map.on('moveend', syncUrl)` to capture viewport changes.
- Uses `history.replaceState` (not `pushState`) — avoids polluting browser history on every pan/zoom.
- No debounce required — `replaceState` is cheap; `moveend` already fires after gestures settle.
- After writing to the hash, the same serialized state is written back to localStorage so refreshes without the hash still restore correctly.

## Load Flow (decode)

On `DOMContentLoaded`, before `restoreLayers()`:

1. Check for `#v=1&s=...` in `location.hash`.
2. If absent or version mismatch → fall through to normal `restoreLayers()`.
3. Decompress and parse the blob. If malformed → silently fall through to `restoreLayers()`.
4. If localStorage has existing layers → show `confirm()` dialog:
   > "This link contains a saved map. Replace your current layers?"
   - **Yes** → load URL state, overwrite localStorage, skip `restoreLayers()`.
   - **No** → discard URL state, proceed with `restoreLayers()` as normal.
5. If localStorage is empty → load URL state silently, no dialog.

## Edge Cases

- **Corrupt hash** — silently falls back to localStorage.
- **Empty localStorage + valid hash** — loads silently, no dialog.
- **LZ-string fails to load** — both `syncUrl()` and hash decode become no-ops (guard on `window.LZString` in both paths); existing localStorage persistence is unaffected.
- **URL too long for some clients** — no hard limit is enforced; LZ-string compression handles large GeoJSON gracefully, but very large datasets may still produce long URLs. Out of scope to truncate or warn.

## What Does Not Change

- localStorage persistence is unchanged and still the primary restore mechanism for non-shared sessions.
- Layer add/remove/rename/recolor/reorder/opacity flows are unchanged; `syncUrl()` piggybacks on the existing `saveLayers()` call.
- "Clear all layers" clears localStorage and also clears the hash.

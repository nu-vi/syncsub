# syncsub

`syncsub` is a small command-line tool to fix .srt subtitles that start in sync and slowly drift out of sync later in the movie.

You give it two reference points:
- the real video timestamp when a given line is spoken, and
- the current timestamp of that same line in the .srt

(one near the start, one near the end)

It then calculates the timing offset and speed drift and rewrites the whole `.srt` so that subtitles stay aligned from start to finish.

The original file is backed up automatically as `.bak`.

---

## Why

A simple delay (e.g. “shift everything by +16 seconds”) only helps at the beginning.

But sometimes the subs were synced to a slightly different source (different frame rate / intro / encode), so by 1h20m they’re several seconds off. That’s not a constant offset, that’s a drift.

`syncsub` fixes that case. It does a linear retime:
  
```text
new_time = A + B * old_time
based on two sync pairs you provide.

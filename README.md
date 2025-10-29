# syncsub

`syncsub` is a small command-line tool to fix `.srt` subtitles that start in sync and slowly drift out of sync later in the movie.

You give it two reference points:
- the real video timestamp when a specific line is spoken, and
- the current timestamp of that same line in the `.srt`

(one pair near the start, one pair near the end)

`syncsub` calculates both the global delay and the playback speed drift, then rewrites the entire `.srt` so subtitles stay aligned from start to finish.

The original file is backed up automatically as `yourfile.srt.bak`.

---

## Why

A simple delay (for example “shift everything by +16 seconds”) only fixes the beginning.

But often the subtitle file was synced to a slightly different encode or source (different intro, different frame pacing, etc.), so by 1h20m the lines are several seconds off. That’s drift.

`syncsub` fixes that by applying a linear correction:

```text
new_time = A + B * old_time
```

where:
- `A` is the offset
- `B` is the speed factor

`A` and `B` are computed from the two sync points you give it.

---

## Requirements

- macOS or Linux shell
- Python 3 available as `python3`
- A plain `.srt` subtitle file

No `pip install` needed.  
No extra modules.  
This does not edit subs embedded inside `.mkv` or `.mp4` containers — it edits standalone `.srt` files.

---

## Install

Clone the repo and make the script executable:

```bash
git clone https://github.com/nu-vi/syncsub
cd syncsub
chmod +x syncsub
```

Optionally put the syncsub on your PATH so you can call `syncsub` from anywhere.

---

## Usage

### Show help

```bash
syncsub -h
syncsub --help
```

### Show version

```bash
syncsub -v
syncsub --version
```

### Fix a subtitle file

```bash
syncsub SUBFILE V1 S1 V2 S2
```

Where:

- `SUBFILE`  
  Path to the `.srt` you want to fix.  
  The script will:
  - create `SUBFILE.bak`
  - then overwrite `SUBFILE` in-place with corrected timestamps

- `V1`  
  Correct **video** timestamp (what you see in the player) for a line near the start.  
  Example: `00:01:14,777`  
  This is when the actor actually says the line in the movie.

- `S1`  
  Current **subtitle** timestamp for that exact same spoken line, taken from the `.srt` *before* fixing.  
  Example: `00:00:58,777`

- `V2`  
  Correct **video** timestamp of a line near the end of the movie.  
  Example: `01:25:30,500`

- `S2`  
  Current **subtitle** timestamp for that same late line in the `.srt`.  
  Example: `01:25:20,214`

Timestamps can use comma or dot for milliseconds (`00:01:14,777` or `00:01:14.777`).  
Hour can be `0X:` or `X:` — both work.

#### Example real command

```bash
syncsub "example.srt" 00:01:14,777 00:00:58,777 01:25:30,500 01:25:20,214
```

What happens:
1. `syncsub` calculates a linear correction so that:
   - `S1` → `V1`
   - `S2` → `V2`
2. It rewrites every timestamp in the `.srt` using that mapping.
3. It saves a backup `...srt.bak` before writing.

---

## How to get V1 / S1 / V2 / S2

You only need to collect these four timestamps once per movie:

1. Play the movie in your player.
2. Pick a clear spoken line near the start.
   - Pause exactly when the actor says it.
   - Note the movie time on screen → this is `V1`.
3. Open the `.srt` in a text editor, find that exact line, and note its timestamp → this is `S1`.

4. Do the same thing again for a line near the end of the movie.
   - Movie time → `V2`
   - Subtitle time in the file → `S2`

Then pass those four timestamps to `syncsub`.

If the subtitle file is from the same cut (no missing scenes or extra scenes), this usually makes the whole movie sync properly from start to end.

---

## Limitations

- This assumes **linear drift**.  
  If your subtitle is actually from a different cut with added/removed scenes, then the drift is not smooth — there’s a jump.  
  One single transform can’t fix a jump in the middle. In that case you’d split the `.srt` around that jump and run `syncsub` twice (start→jump, jump→end).

- Only `.srt`.  
  Formats like ASS/SSA aren’t supported yet. Embedded subs inside .mkv are not processed directly.

- It overwrites the file in place.  
  It does create `file.srt.bak`, but after that, you’re responsible for keeping backups.

---

## Status

Version: 1.0.0  

`syncsub` is a tiny CLI utility meant for personal use.  
Pull requests are welcome if they keep it simple and keep the zero-dependency approach.
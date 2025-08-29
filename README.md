

Make a YouTube-ready video from a **WAV + cover image** in the current folder.  
Default = **no visualizer** (static cover). Opt-in **waveform**, **spectrum** (scrolling), or **bars**.

- 🖼️ Auto-finds cover (`cover.*` / `folder.*`, else newest `*.jpg|*.jpeg|*.png`)
- 🎵 Auto-finds audio (`*.wav`, picks newest if multiple)
- 🧭 Auto output size from **cover aspect**
  - 16:9 → `1920×1080`
  - 4:3 → `1440×1080` (no cropping)
  - Other ratios → full image shown with padding
- 🏷️ Sets MP4 **title** metadata from the WAV filename
- 🔊 Exports AAC @ 48 kHz (uses `libfdk_aac` if available, else native AAC)
- 🎚️ Optional bottom/top/center **waveform**, **scrolling spectrum**, or **bar analyzer**
- 🎨 Palette-driven auto color for waveform & bars (`--auto-color[=auto|accent|complement|on|dual]`)

---

## Requirements

- **FFmpeg** (includes `ffprobe` and video filters like `showwaves`, `showspectrum`, `showfreqs`)
- **ImageMagick** *(optional; only for `--auto-color` to pick a high-contrast waveform color)*

### macOS

**Homebrew (recommended):**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install ffmpeg imagemagick
```
MacPorts (alternative):
```bash
sudo port install ffmpeg ImageMagick
```
### Linux
#### Debian / Ubuntu:
```bash
sudo apt update
sudo apt install -y ffmpeg imagemagick
```
#### Fedora:
```bash
sudo dnf install -y ffmpeg ImageMagick
```
#### Arch / Manjaro:
```bash
sudo pacman -S ffmpeg imagemagick
```
### Windows
`mkyt` is a Bash script. Use WSL (Ubuntu) or Git Bash.
#### WSL (Ubuntu):
```powershell
wsl --install -d Ubuntu
# inside Ubuntu:
sudo apt update
sudo apt install -y ffmpeg imagemagick
```
#### Git Bash + FFmpeg binaries:
Install a static FFmpeg build and ensure ffmpeg.exe / ffprobe.exe are on PATH. ImageMagick is optional.

### Verify installs
```bash
ffmpeg -version
ffprobe -version
ffmpeg -hide_banner -filters | grep -E 'showwaves|showspectrum|showfreqs'   # should list all 3
ffmpeg -hide_banner -encoders | grep -i aac                                # see if libfdk_aac is present (optional)
convert -version                                                            # ImageMagick (optional)
```
---

## Install the script
Place the `mkyt` file wherever you like, make it executable, and put it on your PATH.

Example (user bin + symlink):
```bash
chmod +x /path/to/mkyt
mkdir -p ~/bin
ln -s /path/to/mkyt ~/bin/mkyt
# add once to ~/.zshrc or ~/.bashrc:
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc  # or: source ~/.bashrc
```
Check it’s visible:
```bash
which mkyt
mkyt -h
```

---

## Quick start
From a folder that contains one WAV and a cover image:

```bash
mkyt                    # static cover + audio, auto-detect, outputs "<wavname>.mp4"
mkyt waveform           # add a waveform strip (bottom by default)
mkyt spectrum           # add a scrolling spectrum strip (bottom)
mkyt bars               # add a bar analyzer strip (bottom)
```

Output resolution auto-matches the cover aspect (16:9 → 1920×1080, 4:3 → 1440×1080; others padded).

---

## All options (at a glance)
```pgsql
mkyt [none|waveform|spectrum|bars] [options]

Positional (optional):
  none|waveform|spectrum|bars   Visualizer mode (default: none)

Options (all optional):
  -i <cover>       Cover image (jpg/jpeg/png)
  -a <audio>       Audio file (wav)
  -o <output>      Output MP4 (default: <audio_basename>.mp4)
  -w <width>       Output width  (auto by cover AR if not set)
  -h <height>      Output height (auto by cover AR if not set)
  -H <vizHeight>   Visualizer height px (default: 300)
  -A <align>       Viz alignment: bottom|top|center (default: bottom)
  -M <margin>      Margin from chosen edge in px (default: 0)
  -B               Black strip behind viz (height = vizHeight, color black@1)
  -R <rectHeight>  Custom contrast-bar height px (0=off; default: 0)
  -C <rectColor>   Contrast-bar color (default: black@0.45)
  -m <mode>        Waveform mode: line|point|p2p (default: line)
  -f <fps>         Visualizer FPS (default: 25)
  -K <palette>     Spectrum palette (e.g. intensity|rainbow) (default: intensity)
  -b <abr>         Audio bitrate for native AAC (default: 320k)
  -s <sr>          Audio sample rate (default: 48000)
  -Q <crf>         H.264 CRF (lower=better; default: 18)
  -P <preset>      x264 preset (ultrafast..veryslow; default: medium)
  --auto-color[=auto|accent|complement|on|dual]  Palette auto-color (waveform & bars). Default: auto
  --debug-palette  Print extracted palette JSON and exit
  --spectro-scroll <scroll|rscroll|fullframe|replace>  Spectrum slide mode (default: scroll)
  --spectro-center  Center-out spectrum (new data at center)
  --spectro-vertical Vertical spectrum layout (rotate 90° CCW)
  --viz-below       Stack visualizer under the cover (extends canvas; alias --spectro-below)
```

---

## Examples (covering every feature)
### 1) No visualizer (default)
```bash
mkyt
mkyt -o "My Track (Official Audio).mp4"
```
### 2) Waveform strip
```bash
# default line waveform at bottom
mkyt waveform

# Palette auto-color (needs ImageMagick):
# - auto picks complement over artwork, or accent over a solid black strip (-B)
mkyt waveform --auto-color

# Prefer accent on a black strip for pop
mkyt waveform --auto-color -B

# Force complement or accent explicitly
mkyt waveform --auto-color=complement
mkyt waveform --auto-color=accent

# Use on-accent (black/white chosen for contrast over the accent)
mkyt waveform --auto-color=on -B

# Dual color: primary|secondary per channel (auto swaps on black strip)
# - Over artwork: complement|accent
# - On black strip: accent|complement
mkyt waveform --auto-color=dual

# different waveform look + smaller height + margin from edge
mkyt waveform -m p2p -H 240 -M 12
# stack waveform below the cover image
mkyt waveform --viz-below -B
```
### 3) Scrolling spectrum strip
```bash
# readability boost: black strip = viz height
mkyt spectrum -B

# try a different palette (if your FFmpeg supports it)
mkyt spectrum -K rainbow -H 280 -M 16 -B

# place spectrum at the top
mkyt spectrum -A top -B
# stack spectrum below the cover image
mkyt spectrum --viz-below -B
```
### 4) Bar analyzer (“dance” look)
```bash
mkyt bars                        # default bars at bottom
mkyt bars -H 260 -M 12 -B        # shorter with margin and black strip
mkyt bars -A center              # centered bar block over image
mkyt bars --viz-below -B         # stack bars below the cover image

# Palette auto-color tints bars (grayscale → tint):
mkyt bars --auto-color           # complement tint over artwork
mkyt bars --auto-color -B        # accent tint on black strip
mkyt bars --auto-color=complement
mkyt bars --auto-color=accent

# Dual tint: blend of primary and secondary tints (screen blend)
mkyt bars --auto-color=dual      # complement+accent over artwork (accent+complement on black strip)
```
### 5) Placement, margins, and contrast bar
```bash
# top placement with 16 px margin, translucent contrast bar
mkyt waveform -A top -M 16 -R 200 -C black@0.5

# centered visualizer (margin ignored in center mode)
mkyt spectrum -A center -H 320 -B
```
### 6) Explicit files (skip auto-detect)
```bash
mkyt -i art.png -a mixdown.wav -o release.mp4
```
### 7) Override output size (ignore auto aspect)
```bash
mkyt waveform -w 2560 -h 1440 -H 340 -B
```
### 8) Tuning performance / quality
```bash
# Faster encode (bigger files)
mkyt -Q 20 -P fast

# Slower, smaller (same visual quality)
mkyt -Q 18 -P slow
```
### 9) Audio settings
```bash
# Defaults: AAC-LC 320k @ 48kHz (uses libfdk_aac VBR 5 if available)
mkyt

# Force different bitrate or sample rate
mkyt -b 256k -s 48000
```
---

## How it works (internals in one breath)
- Cover is scaled to show the entire image and padded to the chosen output size.
- Visualizers use FFmpeg filters:
    - `showwaves` (waveform; supports custom color, modes line|point|p2p)
    - `showspectrum` (scrolling spectrogram with slide=scroll, Hann window, overlap 0.8)
    - `showfreqs` (bar analyzer with log frequency scaling)
 
- Palette extraction (Monet-like): the cover is quantized (ImageMagick), near-neutrals/extremes are filtered out, and colors are scored
  by presence and saturation with a mid-lightness bias. We return:
  - `accent_hex`: expressive primary from the cover
  - `complement_hex`: hue-rotated 180° of accent (L clamped to [0.4, 0.7])
  - `on_accent_hex`: black/white chosen by relative luminance of the accent
  - `gradient`: [accent → on_accent]

- Auto-color modes:
  - `auto`: complement over artwork; accent if a solid black strip is active (`-B` or `-R`=viz height with `-C black@1`).
  - `accent|complement|on`: explicit selection.
  - `dual`: uses primary|secondary (complement|accent over artwork; accent|complement on black strip). Waveform colors L|R; bars blend both tints.

- Audio is encoded with **libfdk_aac** VBR 5 if present; otherwise FFmpeg’s native AAC-LC at your chosen bitrate.

---

## Tips for YouTube audio
- Master/export your track to WAV/AIFF 48 kHz.
- Upload H.264 + AAC-LC 320 kbps, 48 kHz. YouTube will re-encode anyway; this avoids extra losses.
- Keep `-pix_fmt yuv420p` for universal playback.

---

## Troubleshooting
- `ffmpeg: command not found` → install FFmpeg (see above).
- `ffprobe: command not found` → install FFmpeg (ffprobe ships with it).
- No WAV / No cover found → place files in the folder, or pass -a / -i.
- `--auto-color` not working → install ImageMagick (`convert`/`magick`).
- Palette seems off → run `mkyt --debug-palette`.
- See what mkyt decided → set `MKYT_DEBUG=1` to log palette values, chosen modes, and filter strings.
- Need more color clusters → set `MKYT_PALETTE_COLORS=16` when debugging palette.
- Palette not recognized → your FFmpeg may not support that palette; use default -K intensity.
- Performance → raise -Q (e.g., 20–22) or use faster -P fast.

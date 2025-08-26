# mkyt

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
- 🎨 Optional **auto-contrast waveform color** vs cover (`--auto-color`, needs ImageMagick)

---

## Requirements

- **FFmpeg** (includes `ffprobe` and video filters like `showwaves`, `showspectrum`, `showfreqs`)
- **ImageMagick** *(optional; only for `--auto-color` to pick a high-contrast waveform color)*

### macOS

**Homebrew (recommended):**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install ffmpeg imagemagick
MacPorts (alternative):

bash
Copy
Edit
sudo port install ffmpeg ImageMagick
Linux
Debian / Ubuntu:

bash
Copy
Edit
sudo apt update
sudo apt install -y ffmpeg imagemagick
Fedora:

bash
Copy
Edit
sudo dnf install -y ffmpeg ImageMagick
Arch / Manjaro:

bash
Copy
Edit
sudo pacman -S ffmpeg imagemagick
Windows
mkyt is a Bash script. Use WSL (Ubuntu) or Git Bash.

WSL (Ubuntu):

powershell
Copy
Edit
wsl --install -d Ubuntu
# inside Ubuntu:
sudo apt update
sudo apt install -y ffmpeg imagemagick
Git Bash + FFmpeg binaries:
Install a static FFmpeg build and ensure ffmpeg.exe / ffprobe.exe are on PATH. ImageMagick is optional.

Verify installs
bash
Copy
Edit
ffmpeg -version
ffprobe -version
ffmpeg -hide_banner -filters | grep -E 'showwaves|showspectrum|showfreqs'   # should list all 3
ffmpeg -hide_banner -encoders | grep -i aac                                # see if libfdk_aac is present (optional)
convert -version                                                            # ImageMagick (optional)
Install the script
Place the mkyt file wherever you like, make it executable, and put it on your PATH.

Example (user bin + symlink):

bash
Copy
Edit
chmod +x /path/to/mkyt
mkdir -p ~/bin
ln -s /path/to/mkyt ~/bin/mkyt
# add once to ~/.zshrc or ~/.bashrc:
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc  # or: source ~/.bashrc
Check it’s visible:

bash
Copy
Edit
which mkyt
mkyt -h
Quick start
From a folder that contains one WAV and a cover image:

bash
Copy
Edit
mkyt                    # static cover + audio, auto-detect, outputs "<wavname>.mp4"
mkyt waveform           # add a waveform strip (bottom by default)
mkyt spectrum           # add a scrolling spectrum strip (bottom)
mkyt bars               # add a bar analyzer strip (bottom)
Output resolution auto-matches the cover aspect (16:9 → 1920×1080, 4:3 → 1440×1080; others padded).

All options (at a glance)
pgsql
Copy
Edit
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
  --auto-color     (waveform only) Auto-pick high-contrast color (needs ImageMagick)
Examples (covering every feature)
1) No visualizer (default)
bash
Copy
Edit
mkyt
mkyt -o "My Track (Official Audio).mp4"
2) Waveform strip
bash
Copy
Edit
# default line waveform at bottom
mkyt waveform

# auto-contrast color vs cover (needs ImageMagick), with a solid black strip behind
mkyt waveform --auto-color -B

# different waveform look + smaller height + margin from edge
mkyt waveform -m p2p -H 240 -M 12
3) Scrolling spectrum strip
bash
Copy
Edit
# readability boost: black strip = viz height
mkyt spectrum -B

# try a different palette (if your FFmpeg supports it)
mkyt spectrum -K rainbow -H 280 -M 16 -B

# place spectrum at the top
mkyt spectrum -A top -B
4) Bar analyzer (“dance” look)
bash
Copy
Edit
mkyt bars                        # default bars at bottom
mkyt bars -H 260 -M 12 -B        # shorter with margin and black strip
mkyt bars -A center              # centered bar block over image
5) Placement, margins, and contrast bar
bash
Copy
Edit
# top placement with 16 px margin, translucent contrast bar
mkyt waveform -A top -M 16 -R 200 -C black@0.5

# centered visualizer (margin ignored in center mode)
mkyt spectrum -A center -H 320 -B
6) Explicit files (skip auto-detect)
bash
Copy
Edit
mkyt -i art.png -a mixdown.wav -o release.mp4
7) Override output size (ignore auto aspect)
bash
Copy
Edit
mkyt waveform -w 2560 -h 1440 -H 340 -B
8) Tuning performance / quality
bash
Copy
Edit
# Faster encode (bigger files)
mkyt -Q 20 -P fast

# Slower, smaller (same visual quality)
mkyt -Q 18 -P slow
9) Audio settings
bash
Copy
Edit
# Defaults: AAC-LC 320k @ 48kHz (uses libfdk_aac VBR 5 if available)
mkyt

# Force different bitrate or sample rate
mkyt -b 256k -s 48000
How it works (internals in one breath)
Cover is scaled to show the entire image and padded to the chosen output size.

Visualizers use FFmpeg filters:

showwaves (waveform; supports custom color, modes line|point|p2p)

showspectrum (scrolling spectrogram with slide=scroll, Hann window, overlap 0.8)

showfreqs (bar analyzer with log frequency scaling)

--auto-color samples the average cover color (ImageMagick) and chooses black/white for best contrast.

Audio is encoded with libfdk_aac VBR 5 if present; otherwise FFmpeg’s native AAC-LC at your chosen bitrate.

Tips for YouTube audio
Master/export your track to WAV/AIFF 48 kHz.

Upload H.264 + AAC-LC 320 kbps, 48 kHz. YouTube will re-encode anyway; this avoids extra losses.

Keep -pix_fmt yuv420p for universal playback.

Troubleshooting
ffmpeg: command not found → install FFmpeg (see above).

ffprobe: command not found → install FFmpeg (ffprobe ships with it).

No WAV / No cover found → place files in the folder, or pass -a / -i.

--auto-color not working → install ImageMagick (convert cmd).

Palette not recognized → your FFmpeg may not support that palette; use default -K intensity.

Performance → raise -Q (e.g., 20–22) or use faster -P fast.

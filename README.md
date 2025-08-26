mkyt

Make a YouTube-ready video from a WAV + cover image in the current folder.
Default is no visualizer (static cover). Optionally add a waveform or spectrum bar.

🖼️ Auto-finds the cover image (cover.* / folder.*, else newest *.jpg|*.png)

🎵 Auto-finds the audio (*.wav, picks the newest if multiple)

🏷️ Sets MP4 title metadata from your WAV filename

🔊 Exports AAC at 48 kHz (uses libfdk_aac if available, else native AAC)

🟪 Optional bottom/top/center waveform or spectrum bar

Requirements

macOS (or Linux) with FFmpeg

brew install ffmpeg


No extra plugins needed — showwaves and showspectrum are built in.

Install

Put the script file mkyt somewhere in your repo (or ~/scripts/mkyt/mkyt).

Make it executable and add to your PATH (directly or via a symlink):

chmod +x /path/to/mkyt
# Symlink into a personal bin:
mkdir -p ~/bin
ln -s /path/to/mkyt ~/bin/mkyt
# Add to PATH (in ~/.zshrc once):
export PATH="$HOME/bin:$PATH"
source ~/.zshrc

Quick start

From a folder that contains one WAV and a cover image:

mkyt                 # static cover + audio, auto-detected, outputs "<wavname>.mp4"
mkyt waveform        # add a waveform bar (bottom by default)
mkyt spectrum        # add a spectrum bar (bottom by default)

Examples
# Custom output name (no visualizer)
mkyt -o "My Track (Official Audio).mp4"

# Waveform at bottom, smaller height, with subtle contrast strip + margin
mkyt waveform -H 280 -R 120 -M 16

# Spectrum placed at the top
mkyt spectrum -A top -H 320

# Explicitly pick files if you don’t want auto-detect
mkyt -i art.png -a mixdown.wav -o release.mp4

Options
mkyt [none|waveform|spectrum] [options]

Positional:
  none|waveform|spectrum   Visualizer mode (default: none)

Options:
  -i <cover>       Cover image (jpg/jpeg/png)
  -a <audio>       Audio file (wav)
  -o <output>      Output MP4 (default: <audio_basename>.mp4)
  -w <width>       Video width  (default: 1920)
  -h <height>      Video height (default: 1080)
  -H <vizHeight>   Visualizer height in px (default: 300)
  -A <align>       Visualizer alignment: bottom|top|center (default: bottom)
  -M <margin>      Margin from edge in px (default: 0)
  -m <mode>        Waveform mode: line|point|p2p (default: line)
  -f <fps>         Visualizer FPS (default: 25)
  -R <rectHeight>  Contrast bar behind viz in px (0=off; default: 0)
  -C <rectColor>   Bar color (e.g. black@0.45; default: black@0.45)
  -b <abr>         Audio bitrate for native AAC (default: 320k)
  -s <sr>          Audio sample rate (default: 48000)
  -Q <crf>         H.264 CRF (lower=better; default: 18)
  -P <preset>      x264 preset (ultrafast..veryslow; default: medium)

Audio quality tips

Keep your master WAV/AIFF at 48 kHz.

YouTube will re-encode your upload; supplying AAC-LC 320 kbps, 48 kHz avoids extra losses on your end.

If your FFmpeg includes libfdk_aac, mkyt uses it (VBR 5). Otherwise it uses the built-in AAC encoder.

Troubleshooting

“ffmpeg: command not found” → brew install ffmpeg

“No WAV found” / “No cover image found” → put files in the current folder or pass -a / -i

Audio crackles → ensure the source and export are 48 kHz end-to-end (-s 48000)

Colors look washed out on upload → that’s YouTube’s pipeline; use standard 8-bit H.264 (yuv420p) as mkyt does

License

MIT

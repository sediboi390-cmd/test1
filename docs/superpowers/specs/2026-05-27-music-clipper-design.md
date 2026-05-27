# Music Clipper — Design Spec
*2026-05-27*

## Goal

A fully browser-based audio editing tool that lets users load audio files, visualize them as waveforms, cut/trim sections, merge multiple clips, choose an output format, and download the result — with no server or backend required.

## Architecture

A single self-contained file: `musicclipper/index.html`

All processing happens client-side using:
- **WaveSurfer.js** — waveform rendering and region selection
- **FFmpeg.wasm** — audio cutting, merging, and format conversion in the browser

No build step. No framework. Consistent with all other apps in the `flow` repo.

## Components

### 1. File Loader
- Drag-and-drop zone + file input button
- Accepts: MP3, WAV, OGG, M4A, FLAC, AAC
- Displays file name and duration after load
- Multiple files can be loaded for merging

### 2. Waveform Visualizer
- Renders audio waveform using WaveSurfer.js
- Playback controls: play, pause, stop
- Clickable/draggable region selection for cut/trim
- Displays current playhead time

### 3. Time Input Controls
- Start time input (mm:ss.ms)
- End time input (mm:ss.ms)
- Syncs bidirectionally with waveform region selection
- "Select All" button

### 4. Edit Actions
- **Trim** — keep only the selected region
- **Cut** — remove the selected region, join remaining parts
- **Merge** — combine all loaded clips in order

### 5. Output & Download
- Format selector: MP3, WAV, OGG
- "Export & Download" button
- Shows progress indicator during FFmpeg processing
- Triggers browser download on completion

## Data Flow

```
User loads file(s)
  → FileReader reads as ArrayBuffer
  → WaveSurfer renders waveform
  → User selects region (waveform drag or time inputs)
  → User clicks Trim / Cut / Merge
  → FFmpeg.wasm processes audio in browser
  → Output blob → download link triggered
```

## Error Handling

- Invalid file type → toast message
- FFmpeg load failure → clear error message with reload suggestion
- No region selected when trimming → toast warning
- Empty file list when merging → toast warning

## UI Design

- Light, minimal, clean
- Single-page layout, no routing
- Sections: Header → File Loader → Waveform → Controls → Export
- Responsive (works on tablet and desktop)
- Color palette: white background, light grey panels, blue accents

## Tech Stack

| Library | Version | Purpose |
|---------|---------|---------|
| WaveSurfer.js | 7.x (CDN) | Waveform + region selection |
| FFmpeg.wasm | 0.12.x (CDN) | Audio processing in browser |
| Vanilla JS | ES2020 | App logic |
| CSS | Custom | Styling |

## Constraints

- No server, no backend
- No npm/build step
- Single HTML file
- Must work in Chrome, Firefox, Safari (modern versions)

## Success Criteria

- [ ] User can load an audio file and see its waveform
- [ ] User can select a region by dragging or typing times
- [ ] User can trim to keep only the selection
- [ ] User can cut to remove the selection
- [ ] User can load multiple files and merge them
- [ ] User can choose MP3/WAV/OGG output format
- [ ] User can download the processed file
- [ ] All actions show clear feedback (progress, errors, success)

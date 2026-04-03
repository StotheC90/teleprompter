# Teleprompter

Single-file HTML teleprompter for podcast recordings and video production. Runs locally in the browser — no server, no install, no dependencies.

## Features

- **Voice Sync** — speech-driven scrolling via Web Speech API. Adapts to your speaking pace in real-time (Chrome/Edge)
- **Auto-Scroll** — smooth scrolling with adjustable speed (0.5x–8x)
- **Podcast Cues** — `## Section`, `[PAUSE]`, `[LANGSAM]`, `[BETONEN]text[/BETONEN]`, `[TIMER 5s]`
- **Focus Zone** — spotlight effect highlights the current reading area
- **Keyboard Shortcuts** — Space, Arrow Keys, +/-, R, F, V, Esc
- **Remote Control** — control from a second browser tab via BroadcastChannel
- **Templates** — built-in podcast and video script templates
- **File Upload** — drag & drop or file picker for .txt/.md files
- **Customizable** — font size, font family, line width, scroll speed
- **Persistent** — settings, text, and scroll position saved in LocalStorage
- **Fullscreen** — native fullscreen support
- **Mobile** — responsive layout with touch controls

## Usage

1. Open `index.html` in your browser
2. Enter or paste your script (or load a file / pick a template)
3. Click **Start**
4. Use **Space** to play/pause or **V** for Voice Sync

## Voice Sync

Press **V** or click the mic button to activate. The teleprompter listens to your microphone and scrolls based on what you say. Speaks faster → scrolls faster. Pause → text waits.

Requires Chrome or Edge and an internet connection (speech recognition runs via browser services).

## Cue Syntax

| Syntax | Effect |
|--------|--------|
| `## Title` | Section divider |
| `[PAUSE]` | Pause marker |
| `[LANGSAM]` | "Speak slower" warning |
| `[BETONEN]text[/BETONEN]` | Highlighted emphasis |
| `[TIMER 5s]` | Inline countdown timer |

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Space` | Play / Pause |
| `V` | Toggle Voice Sync |
| `↑` / `↓` | Speed +/- |
| `+` / `-` | Font size +/- |
| `R` | Reset to start |
| `F` | Fullscreen |
| `Esc` | Back / Close |
| `?` | Show shortcuts |

## License

MIT

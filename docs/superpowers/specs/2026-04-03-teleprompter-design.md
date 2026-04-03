# Teleprompter — Design Spec

## Ueberblick

Eine selbststaendige HTML-Datei (Single File, kein Build-System), die als Teleprompter funktioniert. Schwarzer Hintergrund, weisse Schrift, scrollender Text mit Podcast-spezifischen Features. Laeuft lokal im Browser, kein Server noetig.

**Use Case:** Monitor neben/unter der Kamera, von dem Text abgelesen wird. Keine Spezial-Hardware.

## Texteingabe

- **Textfeld-Editor** — Text direkt eingeben oder einfuegen
- **Datei-Upload** — .txt und .md Dateien laden per Drag & Drop oder File-Picker
- **Text-Vorlagen** — vorinstallierte Podcast-Templates (Intro/Hauptteil/Outro Struktur)

## Darstellung

- **Schwarzer Hintergrund, weisse Schrift** — klassischer Teleprompter-Look
- **Schriftgroesse einstellbar** — Slider, von klein bis sehr gross, Default gross (Monitor-Ablesung)
- **Fokus-Zone** — aktuelle Lesezeile hell hervorgehoben, Text darueber/darunter abgedunkelt (Spotlight-Effekt)
- **Schriftwahl** — System Sans-Serif oder Monospace, umschaltbar
- **Keine Spiegelung** — nicht benoetigt (normaler Monitor, kein Glas-Setup)

## Podcast-Features

### Abschnittsmarkierungen
Markdown-Syntax im Text (`## Intro`, `## Hauptteil`, `## Outro`) wird als farbige Trennbalken dargestellt.

### Visuelle Cues
Spezielle Tags im Text werden als visuelle Hinweise gerendert:
- `[PAUSE]` — gelbes Pause-Symbol
- `[LANGSAM]` — gelber Hinweis "Langsamer sprechen"
- `[BETONEN]` — Text wird farbig hervorgehoben
- `[TIMER 5s]` — Countdown-Timer inline (z.B. fuer Pausen zwischen Abschnitten)

### Fortschritt
- Fortschrittsbalken mit Prozentanzeige
- Geschaetzte Restzeit basierend auf aktueller Scroll-Geschwindigkeit

## Steuerung

### Basis-Controls
- Play/Pause/Reset Buttons
- Scrollgeschwindigkeit stufenlos einstellbar (Slider)
- Countdown vor Start (3-2-1) damit man sich positionieren kann

### Tastatur-Shortcuts
| Taste | Aktion |
|-------|--------|
| `Space` | Play/Pause |
| `ArrowUp/ArrowDown` | Geschwindigkeit anpassen |
| `+/-` | Schriftgroesse |
| `R` | Reset zum Anfang |
| `F` | Fullscreen |
| `Escape` | Fullscreen verlassen / zurueck zum Editor |

### Fernsteuerung
- Teleprompter zeigt QR-Code mit lokaler URL + Session-ID
- Handy scannt QR-Code, oeffnet Mini-Remote mit Play/Pause/Speed Buttons
- Technologie: `BroadcastChannel` (gleicher Browser/Geraet), WebRTC fuer andere Geraete im WLAN
- Fallback: optionales kleines Node/Python Script fuer lokalen WebSocket-Server

## Extras

- **Estimated Read Time** — berechnet Lesezeit basierend auf Geschwindigkeit
- **Position merken** — per LocalStorage, Browser-Refresh setzt nicht zurueck
- **Mobile-kompatibel** — Touch-Gesten fuer Play/Pause, responsive Layout

## Architektur

```
projects/teleprompter/
└── index.html          ← alles drin: HTML, CSS, JS
```

Eine einzige Datei. Kein Framework, kein Build-Step, kein Server. Per Doppelklick im Browser oeffnen.

### Technische Details
- Vanilla HTML/CSS/JS
- CSS Custom Properties fuer Theme-Werte
- LocalStorage fuer Einstellungen + letzte Position
- requestAnimationFrame fuer smoothes Scrolling
- BroadcastChannel API fuer Same-Browser Fernsteuerung
- WebRTC (mit manuellem Signaling) fuer Cross-Device Fernsteuerung
- FileReader API fuer Datei-Upload
- Fullscreen API

## Nicht im Scope
- Backend / Server (ausser optionalem Signaling-Script)
- Benutzerkonten / Login
- Cloud-Sync
- Text-to-Speech
- Spiegelung (kein Glas-Setup)

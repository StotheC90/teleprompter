# Teleprompter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A single-file HTML teleprompter for podcast recordings with auto-scroll, visual cues, keyboard shortcuts, and remote control.

**Architecture:** One `index.html` file with embedded CSS and JS. Two views: Editor (text input, settings) and Teleprompter (scrolling display). No framework, no build step, no server.

**Tech Stack:** Vanilla HTML/CSS/JS, CSS Custom Properties, BroadcastChannel API, Fullscreen API, FileReader API, LocalStorage, requestAnimationFrame

**Spec:** `docs/superpowers/specs/2026-04-03-teleprompter-design.md`

---

## File Structure

```
projects/teleprompter/
└── index.html    ← Single file: HTML + CSS + JS
```

All tasks modify this one file. Each task adds HTML elements, CSS rules, and JS functions incrementally.

---

### Task 1: Base Skeleton + View Switching

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the initial HTML file with full skeleton**

Write the complete base `index.html`. This establishes the two-view architecture (editor + teleprompter), CSS custom properties, and view switching.

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Teleprompter</title>
  <style>
    :root {
      --bg: #000;
      --text: #fff;
      --accent: #00d4ff;
      --warning: #ffd600;
      --section: #ff6b35;
      --emphasis: #00ff88;
      --font-size: 42px;
      --font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      --scroll-speed: 2;
      --focus-size: 120px;
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--font-family);
      font-size: 16px;
      overflow: hidden;
      height: 100vh;
      width: 100vw;
    }

    /* -- Editor View -- */
    #editor-view {
      display: flex;
      flex-direction: column;
      height: 100vh;
      padding: 20px;
      gap: 16px;
    }

    #editor-view.hidden, #teleprompter-view.hidden, #remote-view.hidden {
      display: none !important;
    }

    .editor-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      flex-shrink: 0;
    }

    .editor-header h1 {
      font-size: 24px;
      color: var(--accent);
      font-weight: 600;
    }

    .editor-toolbar {
      display: flex;
      gap: 12px;
      align-items: center;
      flex-shrink: 0;
      flex-wrap: wrap;
    }

    #text-input {
      flex: 1;
      background: #111;
      color: var(--text);
      border: 1px solid #333;
      border-radius: 8px;
      padding: 16px;
      font-family: 'Cascadia Code', 'Fira Code', monospace;
      font-size: 16px;
      resize: none;
      outline: none;
      line-height: 1.6;
    }

    #text-input:focus {
      border-color: var(--accent);
    }

    #text-input::placeholder {
      color: #555;
    }

    /* -- Buttons & Controls -- */
    .btn {
      background: #222;
      color: var(--text);
      border: 1px solid #444;
      border-radius: 6px;
      padding: 8px 16px;
      font-size: 14px;
      cursor: pointer;
      transition: all 0.15s;
      font-family: inherit;
      white-space: nowrap;
    }

    .btn:hover { background: #333; border-color: #666; }

    .btn-primary {
      background: var(--accent);
      color: #000;
      border-color: var(--accent);
      font-weight: 600;
    }

    .btn-primary:hover { background: #33dfff; }

    .control-group {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .control-group label {
      font-size: 13px;
      color: #888;
      white-space: nowrap;
    }

    input[type="range"] {
      -webkit-appearance: none;
      appearance: none;
      height: 4px;
      background: #333;
      border-radius: 2px;
      outline: none;
    }

    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 16px;
      height: 16px;
      background: var(--accent);
      border-radius: 50%;
      cursor: pointer;
    }

    /* -- Teleprompter View -- */
    #teleprompter-view {
      position: relative;
      height: 100vh;
      overflow: hidden;
    }

    #prompter-content {
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      padding: 50vh 10vw;
      font-size: var(--font-size);
      line-height: 1.5;
      transition: font-size 0.2s, font-family 0.2s;
    }

    /* -- Teleprompter Controls Overlay -- */
    #prompter-controls {
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      background: linear-gradient(transparent, rgba(0,0,0,0.9) 30%);
      padding: 40px 20px 16px;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 16px;
      z-index: 100;
      opacity: 1;
      transition: opacity 0.3s;
    }

    #prompter-controls.auto-hide {
      opacity: 0;
    }

    #prompter-controls:hover {
      opacity: 1;
    }

    /* -- Progress Bar -- */
    #progress-bar {
      position: fixed;
      top: 0;
      left: 0;
      height: 3px;
      background: var(--accent);
      z-index: 100;
      transition: width 0.1s linear;
    }

    /* -- Focus Zone Overlay -- */
    #focus-overlay-top, #focus-overlay-bottom {
      position: fixed;
      left: 0;
      right: 0;
      pointer-events: none;
      z-index: 50;
    }

    #focus-overlay-top {
      top: 0;
      height: calc(50vh - var(--focus-size) / 2);
      background: linear-gradient(to bottom, rgba(0,0,0,0.85), rgba(0,0,0,0.4));
    }

    #focus-overlay-bottom {
      bottom: 0;
      height: calc(50vh - var(--focus-size) / 2);
      background: linear-gradient(to top, rgba(0,0,0,0.85), rgba(0,0,0,0.4));
    }

    /* -- Countdown Overlay -- */
    #countdown-overlay {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.9);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 200;
      font-size: 120px;
      font-weight: 700;
      color: var(--accent);
    }

    #countdown-overlay.hidden { display: none; }

    /* -- Drop Zone Highlight -- */
    #editor-view.dragover #text-input {
      border-color: var(--accent);
      background: #0a1a2a;
    }

    /* -- Shortcut Help -- */
    #shortcut-help {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.95);
      z-index: 300;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    #shortcut-help.hidden { display: none; }

    #shortcut-help table {
      border-collapse: collapse;
      font-size: 18px;
    }

    #shortcut-help th, #shortcut-help td {
      padding: 8px 20px;
      text-align: left;
      border-bottom: 1px solid #333;
    }

    #shortcut-help th { color: var(--accent); }

    #shortcut-help kbd {
      background: #222;
      border: 1px solid #555;
      border-radius: 4px;
      padding: 2px 8px;
      font-family: monospace;
    }

    /* -- Status Info -- */
    #status-bar {
      position: fixed;
      top: 8px;
      right: 16px;
      font-size: 13px;
      color: #666;
      z-index: 100;
      text-align: right;
    }

    /* -- Section Divider -- */
    .section-divider {
      display: flex;
      align-items: center;
      gap: 16px;
      margin: 32px 0 24px;
      color: var(--section);
      font-size: 0.6em;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 2px;
    }

    .section-divider::before, .section-divider::after {
      content: '';
      flex: 1;
      height: 2px;
      background: var(--section);
      opacity: 0.4;
    }

    /* -- Visual Cues -- */
    .cue-pause {
      display: inline-block;
      background: var(--warning);
      color: #000;
      padding: 4px 16px;
      border-radius: 4px;
      font-size: 0.6em;
      font-weight: 700;
      margin: 8px 0;
      letter-spacing: 1px;
    }

    .cue-slow {
      display: inline-block;
      background: var(--warning);
      color: #000;
      padding: 4px 16px;
      border-radius: 4px;
      font-size: 0.5em;
      font-weight: 600;
      margin: 4px 0;
    }

    .cue-emphasis {
      color: var(--emphasis);
      font-weight: 700;
      text-shadow: 0 0 20px rgba(0, 255, 136, 0.3);
    }

    .cue-timer {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      background: #1a1a2e;
      border: 1px solid var(--accent);
      color: var(--accent);
      padding: 4px 16px;
      border-radius: 4px;
      font-size: 0.6em;
      font-weight: 700;
      margin: 8px 0;
      font-variant-numeric: tabular-nums;
    }

    /* -- Remote View -- */
    #remote-view {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      gap: 24px;
      padding: 20px;
    }

    #remote-view h2 {
      color: var(--accent);
      font-size: 20px;
    }

    .remote-controls {
      display: flex;
      gap: 16px;
      flex-wrap: wrap;
      justify-content: center;
    }

    .remote-btn {
      width: 80px;
      height: 80px;
      border-radius: 50%;
      border: 2px solid #444;
      background: #111;
      color: var(--text);
      font-size: 28px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.15s;
    }

    .remote-btn:hover { border-color: var(--accent); background: #1a1a2a; }
    .remote-btn:active { transform: scale(0.95); }

    .remote-status {
      font-size: 14px;
      color: #666;
    }

    /* -- QR Code -- */
    #qr-modal {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.95);
      z-index: 300;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 16px;
    }

    #qr-modal.hidden { display: none; }

    #qr-modal canvas { border-radius: 8px; }

    #qr-modal .qr-url {
      color: #888;
      font-size: 13px;
      max-width: 90vw;
      word-break: break-all;
      text-align: center;
    }

    /* -- Templates Dropdown -- */
    select {
      background: #222;
      color: var(--text);
      border: 1px solid #444;
      border-radius: 6px;
      padding: 8px 12px;
      font-size: 14px;
      cursor: pointer;
      font-family: inherit;
    }

    select:hover { border-color: #666; }
  </style>
</head>
<body>

  <!-- ====== EDITOR VIEW ====== -->
  <div id="editor-view">
    <div class="editor-header">
      <h1>Teleprompter</h1>
      <div style="display:flex;gap:8px;">
        <button class="btn" id="btn-remote-qr" title="Fernsteuerung">Remote</button>
        <button class="btn btn-primary" id="btn-start">Start</button>
      </div>
    </div>

    <div class="editor-toolbar">
      <div class="control-group">
        <label>Vorlage:</label>
        <select id="template-select">
          <option value="">-- Vorlage waehlen --</option>
        </select>
      </div>
      <div class="control-group">
        <label>Datei laden:</label>
        <button class="btn" id="btn-file-upload">Datei oeffnen</button>
        <input type="file" id="file-input" accept=".txt,.md,.text" style="display:none">
      </div>
      <div class="control-group">
        <label>Geschwindigkeit:</label>
        <input type="range" id="speed-slider" min="0.5" max="8" step="0.5" value="2" style="width:120px">
        <span id="speed-display">2.0x</span>
      </div>
      <div class="control-group">
        <label>Schriftgroesse:</label>
        <input type="range" id="fontsize-slider" min="20" max="100" step="2" value="42" style="width:120px">
        <span id="fontsize-display">42px</span>
      </div>
      <div class="control-group">
        <label>Schrift:</label>
        <button class="btn" id="btn-font-toggle">Sans-Serif</button>
      </div>
    </div>

    <textarea id="text-input" placeholder="Text hier eingeben oder Datei per Drag & Drop ablegen...

Podcast-Markierungen:
  ## Abschnittstitel    → farbiger Trenner
  [PAUSE]              → Pause-Hinweis
  [LANGSAM]            → Langsamer sprechen
  [BETONEN]Text[/BETONEN] → Hervorhebung
  [TIMER 5s]           → Countdown-Timer"></textarea>

    <div style="display:flex;justify-content:space-between;align-items:center;flex-shrink:0;">
      <span style="font-size:13px;color:#666;" id="editor-stats"></span>
      <span style="font-size:13px;color:#666;">Shortcuts: ? | Drag & Drop fuer Dateien</span>
    </div>
  </div>

  <!-- ====== TELEPROMPTER VIEW ====== -->
  <div id="teleprompter-view" class="hidden">
    <div id="progress-bar"></div>
    <div id="focus-overlay-top"></div>
    <div id="focus-overlay-bottom"></div>
    <div id="prompter-content"></div>
    <div id="status-bar">
      <div id="status-progress">0%</div>
      <div id="status-time"></div>
    </div>
    <div id="prompter-controls">
      <button class="btn" id="btn-back" title="Zurueck zum Editor (Esc)">Zurueck</button>
      <button class="btn" id="btn-reset" title="Reset (R)">Reset</button>
      <button class="btn btn-primary" id="btn-play" title="Play/Pause (Space)">Play</button>
      <div class="control-group">
        <label>Speed:</label>
        <input type="range" id="speed-slider-prompter" min="0.5" max="8" step="0.5" value="2" style="width:100px">
        <span id="speed-display-prompter">2.0x</span>
      </div>
      <button class="btn" id="btn-fullscreen" title="Fullscreen (F)">Fullscreen</button>
    </div>
    <div id="countdown-overlay" class="hidden"></div>
  </div>

  <!-- ====== REMOTE VIEW ====== -->
  <div id="remote-view" class="hidden">
    <h2>Fernsteuerung</h2>
    <div class="remote-status" id="remote-status">Verbinde...</div>
    <div class="remote-controls">
      <button class="remote-btn" id="remote-reset" title="Reset">&#8634;</button>
      <button class="remote-btn" id="remote-slower" title="Langsamer">&#9664;</button>
      <button class="remote-btn" id="remote-play" title="Play/Pause" style="width:100px;height:100px;font-size:36px;">&#9654;</button>
      <button class="remote-btn" id="remote-faster" title="Schneller">&#9654;</button>
    </div>
    <div class="control-group" style="margin-top:16px;">
      <label style="color:#888;">Speed:</label>
      <span id="remote-speed" style="color:var(--accent);font-size:18px;font-weight:600;">2.0x</span>
    </div>
    <div class="remote-status" id="remote-progress"></div>
  </div>

  <!-- ====== QR MODAL ====== -->
  <div id="qr-modal" class="hidden">
    <h2 style="color:var(--accent);">Fernsteuerung</h2>
    <p style="color:#888;font-size:14px;">QR-Code scannen oder URL oeffnen:</p>
    <canvas id="qr-canvas" width="200" height="200"></canvas>
    <div class="qr-url" id="qr-url"></div>
    <div style="display:flex;gap:8px;margin-top:8px;">
      <button class="btn" id="btn-copy-remote-url">URL kopieren</button>
      <button class="btn" id="btn-open-remote">In neuem Tab oeffnen</button>
      <button class="btn" id="btn-close-qr">Schliessen</button>
    </div>
  </div>

  <!-- ====== SHORTCUT HELP ====== -->
  <div id="shortcut-help" class="hidden">
    <div>
      <h2 style="color:var(--accent);margin-bottom:16px;">Tastatur-Shortcuts</h2>
      <table>
        <tr><td><kbd>Space</kbd></td><td>Play / Pause</td></tr>
        <tr><td><kbd>&uarr;</kbd> / <kbd>&darr;</kbd></td><td>Geschwindigkeit +/-</td></tr>
        <tr><td><kbd>+</kbd> / <kbd>-</kbd></td><td>Schriftgroesse +/-</td></tr>
        <tr><td><kbd>R</kbd></td><td>Reset zum Anfang</td></tr>
        <tr><td><kbd>F</kbd></td><td>Fullscreen</td></tr>
        <tr><td><kbd>Esc</kbd></td><td>Zurueck / Schliessen</td></tr>
        <tr><td><kbd>?</kbd></td><td>Diese Hilfe</td></tr>
      </table>
      <p style="color:#666;margin-top:16px;font-size:13px;">Beliebige Taste druecken zum Schliessen</p>
    </div>
  </div>

  <script>
    /* ============================================
       TELEPROMPTER APP
       ============================================ */

    // ---- State ----
    const state = {
      playing: false,
      scrollY: 0,
      speed: 2,
      fontSize: 42,
      fontMono: false,
      currentView: 'editor', // 'editor' | 'prompter' | 'remote'
      contentHeight: 0,
      viewportHeight: 0,
      lastTimestamp: 0,
      countdownActive: false,
      controlsTimeout: null,
      timers: [],
    };

    // ---- DOM References ----
    const $ = (sel) => document.querySelector(sel);
    const $$ = (sel) => document.querySelectorAll(sel);

    const dom = {
      editorView: $('#editor-view'),
      prompterView: $('#teleprompter-view'),
      remoteView: $('#remote-view'),
      textInput: $('#text-input'),
      prompterContent: $('#prompter-content'),
      progressBar: $('#progress-bar'),
      statusProgress: $('#status-progress'),
      statusTime: $('#status-time'),
      editorStats: $('#editor-stats'),
      countdownOverlay: $('#countdown-overlay'),
      shortcutHelp: $('#shortcut-help'),
      qrModal: $('#qr-modal'),
      btnStart: $('#btn-start'),
      btnBack: $('#btn-back'),
      btnPlay: $('#btn-play'),
      btnReset: $('#btn-reset'),
      btnFullscreen: $('#btn-fullscreen'),
      btnFontToggle: $('#btn-font-toggle'),
      btnFileUpload: $('#btn-file-upload'),
      fileInput: $('#file-input'),
      speedSlider: $('#speed-slider'),
      speedSliderPrompter: $('#speed-slider-prompter'),
      speedDisplay: $('#speed-display'),
      speedDisplayPrompter: $('#speed-display-prompter'),
      fontsizeSlider: $('#fontsize-slider'),
      fontsizeDisplay: $('#fontsize-display'),
      templateSelect: $('#template-select'),
      btnRemoteQr: $('#btn-remote-qr'),
      qrCanvas: $('#qr-canvas'),
      qrUrl: $('#qr-url'),
      btnCopyRemoteUrl: $('#btn-copy-remote-url'),
      btnOpenRemote: $('#btn-open-remote'),
      btnCloseQr: $('#btn-close-qr'),
      remoteStatus: $('#remote-status'),
      remotePlay: $('#remote-play'),
      remoteReset: $('#remote-reset'),
      remoteSlower: $('#remote-slower'),
      remoteFaster: $('#remote-faster'),
      remoteSpeed: $('#remote-speed'),
      remoteProgress: $('#remote-progress'),
    };

    // ---- Templates ----
    const templates = {
      'podcast-standard': {
        name: 'Podcast — Standard',
        text: `## Intro

Willkommen bei [Podcast-Name], Folge [Nummer].
Heute sprechen wir ueber [Thema].

[PAUSE]

Mein Name ist [Name] und ich freue mich, dass ihr dabei seid.

## Hauptteil

[LANGSAM]

Kommen wir zum ersten Punkt: [Thema 1]

[BETONEN]Das Wichtigste vorweg:[/BETONEN] ...

[PAUSE]

Weiter mit Punkt zwei: [Thema 2]

...

[TIMER 3s]

## Zusammenfassung

Fassen wir zusammen, was wir heute besprochen haben:
1. ...
2. ...
3. ...

## Outro

Das wars fuer heute! Wenn euch die Folge gefallen hat, lasst gerne ein Abo da.

Bis zum naechsten Mal!

[PAUSE]
`,
      },
      'podcast-interview': {
        name: 'Podcast — Interview',
        text: `## Intro

Willkommen bei [Podcast-Name]!
Heute habe ich [Gast-Name] zu Gast.

[PAUSE]

[Gast-Name], stell dich doch kurz vor.

[TIMER 10s]

## Fragen

[LANGSAM]

Frage 1: [Frage]

[TIMER 30s]

[PAUSE]

Frage 2: [Frage]

[TIMER 30s]

[PAUSE]

Frage 3: [Frage]

[TIMER 30s]

## Abschluss

[BETONEN]Letzte Frage:[/BETONEN] Was moechtest du unseren Hoerern noch mitgeben?

[TIMER 15s]

## Outro

Vielen Dank [Gast-Name] fuer das Gespraech!

Alle Links findet ihr in den Shownotes.
Bis zum naechsten Mal!
`,
      },
      'video-script': {
        name: 'Video-Script',
        text: `## Hook

[BETONEN]Wusstet ihr, dass...[/BETONEN]

[PAUSE]

## Intro

Hey, ich bin [Name] und heute zeige ich euch [Thema].

## Hauptteil

[LANGSAM]

Schritt 1: ...

[PAUSE]

Schritt 2: ...

[PAUSE]

Schritt 3: ...

## Call to Action

[BETONEN]Wenn euch das Video gefallen hat, lasst einen Like da![/BETONEN]

Abonniert den Kanal fuer mehr Content.

[TIMER 3s]

## Outro

Danke fuers Zuschauen — bis zum naechsten Video!
`,
      },
    };

    // ---- Text Parsing ----
    function parseText(raw) {
      const lines = raw.split('\n');
      let html = '';

      for (let i = 0; i < lines.length; i++) {
        let line = lines[i];

        // Section headers: ## Title
        if (/^##\s+(.+)$/.test(line)) {
          const title = line.replace(/^##\s+/, '');
          html += `<div class="section-divider">${escapeHtml(title)}</div>`;
          continue;
        }

        // [PAUSE]
        if (/^\[PAUSE\]\s*$/.test(line.trim())) {
          html += `<div class="cue-pause">&#9208; PAUSE</div>`;
          continue;
        }

        // [LANGSAM]
        if (/^\[LANGSAM\]\s*$/.test(line.trim())) {
          html += `<div class="cue-slow">&#9888; Langsamer sprechen</div>`;
          continue;
        }

        // [TIMER Xs]
        const timerMatch = line.trim().match(/^\[TIMER\s+(\d+)s\]\s*$/);
        if (timerMatch) {
          const seconds = parseInt(timerMatch[1], 10);
          html += `<div class="cue-timer" data-timer="${seconds}">&#9201; <span class="timer-value">${seconds}s</span></div>`;
          continue;
        }

        // Inline [BETONEN]...[/BETONEN]
        line = line.replace(
          /\[BETONEN\](.*?)\[\/BETONEN\]/g,
          '<span class="cue-emphasis">$1</span>'
        );

        // Empty lines = paragraph break
        if (line.trim() === '') {
          html += '<div style="height:0.5em;"></div>';
        } else {
          html += `<div class="prompter-line">${escapeHtml(line).replace(
            /&lt;span class=&quot;cue-emphasis&quot;&gt;/g, '<span class="cue-emphasis">').replace(
            /&lt;\/span&gt;/g, '</span>')}</div>`;
        }
      }

      return html;
    }

    // Fix: handle emphasis correctly by processing before escaping
    function parseTextV2(raw) {
      const lines = raw.split('\n');
      let html = '';

      for (let i = 0; i < lines.length; i++) {
        let line = lines[i];

        // Section headers
        if (/^##\s+(.+)$/.test(line)) {
          const title = line.replace(/^##\s+/, '');
          html += `<div class="section-divider">${escapeHtml(title)}</div>`;
          continue;
        }

        // [PAUSE]
        if (/^\[PAUSE\]\s*$/.test(line.trim())) {
          html += `<div class="cue-pause">&#9208; PAUSE</div>`;
          continue;
        }

        // [LANGSAM]
        if (/^\[LANGSAM\]\s*$/.test(line.trim())) {
          html += `<div class="cue-slow">&#9888; Langsamer sprechen</div>`;
          continue;
        }

        // [TIMER Xs]
        const timerMatch = line.trim().match(/^\[TIMER\s+(\d+)s\]\s*$/);
        if (timerMatch) {
          const seconds = parseInt(timerMatch[1], 10);
          html += `<div class="cue-timer" data-timer="${seconds}">&#9201; <span class="timer-value">${seconds}s</span></div>`;
          continue;
        }

        // Empty lines
        if (line.trim() === '') {
          html += '<div style="height:0.5em;"></div>';
          continue;
        }

        // Process inline emphasis, then escape the rest
        const parts = line.split(/(\[BETONEN\].*?\[\/BETONEN\])/g);
        let lineHtml = '';
        for (const part of parts) {
          const emphMatch = part.match(/^\[BETONEN\](.*?)\[\/BETONEN\]$/);
          if (emphMatch) {
            lineHtml += `<span class="cue-emphasis">${escapeHtml(emphMatch[1])}</span>`;
          } else {
            lineHtml += escapeHtml(part);
          }
        }

        html += `<div class="prompter-line">${lineHtml}</div>`;
      }

      return html;
    }

    function escapeHtml(str) {
      return str.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
    }

    // ---- Word Count & Read Time ----
    function getWordCount(text) {
      return text.trim().split(/\s+/).filter(w => w.length > 0).length;
    }

    function getEstimatedReadTime(wordCount, speedMultiplier) {
      // Average speaking rate ~130 words/min, adjusted by speed
      const wordsPerMin = 130 * (speedMultiplier / 2);
      const minutes = wordCount / wordsPerMin;
      if (minutes < 1) return '< 1 Min';
      return `~${Math.round(minutes)} Min`;
    }

    function updateEditorStats() {
      const text = dom.textInput.value;
      const words = getWordCount(text);
      const readTime = getEstimatedReadTime(words, state.speed);
      dom.editorStats.textContent = `${words} Woerter | Lesezeit: ${readTime}`;
    }

    // ---- View Switching ----
    function showView(view) {
      state.currentView = view;
      dom.editorView.classList.toggle('hidden', view !== 'editor');
      dom.prompterView.classList.toggle('hidden', view !== 'prompter');
      dom.remoteView.classList.toggle('hidden', view !== 'remote');
    }

    function startPrompter() {
      const text = dom.textInput.value.trim();
      if (!text) return;

      dom.prompterContent.innerHTML = parseTextV2(text);
      state.scrollY = loadPosition() || 0;
      dom.prompterContent.style.transform = `translateY(${-state.scrollY}px)`;
      state.playing = false;
      state.contentHeight = dom.prompterContent.scrollHeight;
      state.viewportHeight = window.innerHeight;

      dom.btnPlay.textContent = 'Play';
      updateProgress();
      showView('prompter');
      saveText();

      // Auto-hide controls after 3s
      resetControlsTimeout();
    }

    function backToEditor() {
      stopPlaying();
      exitFullscreen();
      showView('editor');
    }

    // ---- Scroll Engine ----
    function scrollLoop(timestamp) {
      if (!state.playing) return;

      if (state.lastTimestamp === 0) {
        state.lastTimestamp = timestamp;
      }

      const delta = timestamp - state.lastTimestamp;
      state.lastTimestamp = timestamp;

      // Speed: pixels per frame at 60fps baseline
      const pixelsPerSecond = state.speed * 30;
      const scrollDelta = pixelsPerSecond * (delta / 1000);

      state.scrollY += scrollDelta;

      const maxScroll = state.contentHeight - state.viewportHeight / 2;
      if (state.scrollY >= maxScroll) {
        state.scrollY = maxScroll;
        stopPlaying();
      }

      dom.prompterContent.style.transform = `translateY(${-state.scrollY}px)`;
      updateProgress();
      savePosition();

      if (state.playing) {
        requestAnimationFrame(scrollLoop);
      }
    }

    function startPlaying() {
      if (state.countdownActive) return;

      // Start with countdown
      runCountdown(() => {
        state.playing = true;
        state.lastTimestamp = 0;
        dom.btnPlay.textContent = 'Pause';
        requestAnimationFrame(scrollLoop);
        broadcastState();
      });
    }

    function stopPlaying() {
      state.playing = false;
      state.lastTimestamp = 0;
      dom.btnPlay.textContent = 'Play';
      broadcastState();
    }

    function togglePlay() {
      if (state.playing) {
        stopPlaying();
      } else {
        startPlaying();
      }
    }

    function resetScroll() {
      stopPlaying();
      state.scrollY = 0;
      dom.prompterContent.style.transform = 'translateY(0)';
      updateProgress();
      savePosition();
      broadcastState();
    }

    // ---- Countdown ----
    function runCountdown(callback) {
      // Skip countdown if already playing (resume = no countdown)
      if (state.scrollY > 10) {
        callback();
        return;
      }

      state.countdownActive = true;
      dom.countdownOverlay.classList.remove('hidden');
      let count = 3;
      dom.countdownOverlay.textContent = count;

      const interval = setInterval(() => {
        count--;
        if (count <= 0) {
          clearInterval(interval);
          dom.countdownOverlay.classList.add('hidden');
          state.countdownActive = false;
          callback();
        } else {
          dom.countdownOverlay.textContent = count;
        }
      }, 1000);
    }

    // ---- Progress ----
    function updateProgress() {
      const maxScroll = state.contentHeight - state.viewportHeight / 2;
      if (maxScroll <= 0) return;

      const progress = Math.min(state.scrollY / maxScroll, 1);
      const percent = Math.round(progress * 100);

      dom.progressBar.style.width = `${percent}%`;
      dom.statusProgress.textContent = `${percent}%`;

      // Estimated remaining time
      if (state.speed > 0) {
        const remainingPixels = maxScroll - state.scrollY;
        const pixelsPerSecond = state.speed * 30;
        const remainingSeconds = remainingPixels / pixelsPerSecond;

        if (remainingSeconds < 60) {
          dom.statusTime.textContent = `~${Math.round(remainingSeconds)}s`;
        } else {
          dom.statusTime.textContent = `~${Math.round(remainingSeconds / 60)} Min`;
        }
      }

      broadcastState();
    }

    // ---- Speed Control ----
    function setSpeed(newSpeed) {
      state.speed = Math.max(0.5, Math.min(8, newSpeed));
      dom.speedSlider.value = state.speed;
      dom.speedSliderPrompter.value = state.speed;
      dom.speedDisplay.textContent = state.speed.toFixed(1) + 'x';
      dom.speedDisplayPrompter.textContent = state.speed.toFixed(1) + 'x';
      updateEditorStats();
      saveSettings();
      broadcastState();
    }

    // ---- Font Size ----
    function setFontSize(size) {
      state.fontSize = Math.max(20, Math.min(100, size));
      document.documentElement.style.setProperty('--font-size', state.fontSize + 'px');
      dom.fontsizeSlider.value = state.fontSize;
      dom.fontsizeDisplay.textContent = state.fontSize + 'px';

      // Recalculate content height
      if (state.currentView === 'prompter') {
        setTimeout(() => {
          state.contentHeight = dom.prompterContent.scrollHeight;
          updateProgress();
        }, 250);
      }
      saveSettings();
    }

    // ---- Font Family ----
    function toggleFont() {
      state.fontMono = !state.fontMono;
      const family = state.fontMono
        ? "'Cascadia Code', 'Fira Code', 'Consolas', monospace"
        : "'Segoe UI', system-ui, -apple-system, sans-serif";
      document.documentElement.style.setProperty('--font-family', family);
      dom.prompterContent.style.fontFamily = family;
      dom.btnFontToggle.textContent = state.fontMono ? 'Monospace' : 'Sans-Serif';
      saveSettings();
    }

    // ---- File Upload ----
    function handleFileUpload(file) {
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (e) => {
        dom.textInput.value = e.target.result;
        updateEditorStats();
        saveText();
      };
      reader.readAsText(file);
    }

    // ---- Fullscreen ----
    function toggleFullscreen() {
      if (document.fullscreenElement) {
        exitFullscreen();
      } else {
        document.documentElement.requestFullscreen().catch(() => {});
      }
    }

    function exitFullscreen() {
      if (document.fullscreenElement) {
        document.exitFullscreen().catch(() => {});
      }
    }

    // ---- Controls Auto-Hide ----
    function resetControlsTimeout() {
      const controls = $('#prompter-controls');
      controls.classList.remove('auto-hide');
      clearTimeout(state.controlsTimeout);
      state.controlsTimeout = setTimeout(() => {
        if (state.playing) {
          controls.classList.add('auto-hide');
        }
      }, 3000);
    }

    // ---- Keyboard Shortcuts ----
    function handleKeyboard(e) {
      // Don't handle when typing in textarea
      if (e.target === dom.textInput) {
        if (e.key === '?' && e.target.selectionStart === e.target.selectionEnd) return;
        return;
      }

      // Close overlays first
      if (e.key === 'Escape') {
        if (!dom.shortcutHelp.classList.contains('hidden')) {
          dom.shortcutHelp.classList.add('hidden');
          return;
        }
        if (!dom.qrModal.classList.contains('hidden')) {
          dom.qrModal.classList.add('hidden');
          return;
        }
        if (state.currentView === 'prompter') {
          backToEditor();
          return;
        }
      }

      // Shortcut help
      if (e.key === '?') {
        if (!dom.shortcutHelp.classList.contains('hidden')) {
          dom.shortcutHelp.classList.add('hidden');
        } else {
          dom.shortcutHelp.classList.remove('hidden');
        }
        return;
      }

      // Close shortcut help on any key
      if (!dom.shortcutHelp.classList.contains('hidden')) {
        dom.shortcutHelp.classList.add('hidden');
        return;
      }

      if (state.currentView !== 'prompter') return;

      switch (e.key) {
        case ' ':
          e.preventDefault();
          togglePlay();
          resetControlsTimeout();
          break;
        case 'ArrowUp':
          e.preventDefault();
          setSpeed(state.speed + 0.5);
          resetControlsTimeout();
          break;
        case 'ArrowDown':
          e.preventDefault();
          setSpeed(state.speed - 0.5);
          resetControlsTimeout();
          break;
        case '+':
        case '=':
          setFontSize(state.fontSize + 4);
          break;
        case '-':
          setFontSize(state.fontSize - 4);
          break;
        case 'r':
        case 'R':
          resetScroll();
          resetControlsTimeout();
          break;
        case 'f':
        case 'F':
          toggleFullscreen();
          break;
      }
    }

    // ---- LocalStorage ----
    const STORAGE_KEY = 'teleprompter';

    function saveSettings() {
      const data = loadStorage();
      data.speed = state.speed;
      data.fontSize = state.fontSize;
      data.fontMono = state.fontMono;
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
    }

    function saveText() {
      const data = loadStorage();
      data.text = dom.textInput.value;
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
    }

    function savePosition() {
      const data = loadStorage();
      data.scrollY = state.scrollY;
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
    }

    function loadPosition() {
      return loadStorage().scrollY || 0;
    }

    function loadStorage() {
      try {
        return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {};
      } catch {
        return {};
      }
    }

    function restoreState() {
      const data = loadStorage();
      if (data.speed) setSpeed(data.speed);
      if (data.fontSize) setFontSize(data.fontSize);
      if (data.fontMono) toggleFont();
      if (data.text) dom.textInput.value = data.text;
      updateEditorStats();
    }

    // ---- BroadcastChannel Remote ----
    let channel = null;

    function initBroadcastChannel() {
      try {
        channel = new BroadcastChannel('teleprompter-remote');
        channel.onmessage = (e) => {
          const msg = e.data;
          if (msg.type === 'command') {
            handleRemoteCommand(msg.action);
          }
          if (msg.type === 'state' && state.currentView === 'remote') {
            updateRemoteDisplay(msg);
          }
          if (msg.type === 'ping') {
            broadcastState();
          }
        };
      } catch {
        // BroadcastChannel not supported
      }
    }

    function handleRemoteCommand(action) {
      if (state.currentView !== 'prompter') return;
      switch (action) {
        case 'toggle-play': togglePlay(); break;
        case 'reset': resetScroll(); break;
        case 'speed-up': setSpeed(state.speed + 0.5); break;
        case 'speed-down': setSpeed(state.speed - 0.5); break;
      }
      resetControlsTimeout();
    }

    function broadcastState() {
      if (!channel) return;
      const maxScroll = state.contentHeight - state.viewportHeight / 2;
      const progress = maxScroll > 0 ? Math.round((state.scrollY / maxScroll) * 100) : 0;
      channel.postMessage({
        type: 'state',
        playing: state.playing,
        speed: state.speed,
        progress: progress,
      });
    }

    function sendRemoteCommand(action) {
      if (!channel) return;
      channel.postMessage({ type: 'command', action });
    }

    function updateRemoteDisplay(msg) {
      dom.remotePlay.innerHTML = msg.playing ? '&#9208;' : '&#9654;';
      dom.remoteSpeed.textContent = msg.speed.toFixed(1) + 'x';
      dom.remoteProgress.textContent = `Fortschritt: ${msg.progress}%`;
      dom.remoteStatus.textContent = msg.playing ? 'Laeuft...' : 'Pausiert';
      dom.remoteStatus.style.color = msg.playing ? '#0f0' : '#888';
    }

    // ---- QR Code (Simple Canvas) ----
    function generateQR() {
      const url = window.location.href.split('#')[0] + '#remote';
      dom.qrUrl.textContent = url;

      // Simple QR-like visual (not a real QR code — for real usage, embed qr-creator-es6 or similar)
      // We'll draw a text-based fallback
      const ctx = dom.qrCanvas.getContext('2d');
      ctx.fillStyle = '#fff';
      ctx.fillRect(0, 0, 200, 200);
      ctx.fillStyle = '#000';
      ctx.font = '14px monospace';
      ctx.textAlign = 'center';
      ctx.fillText('QR Code', 100, 80);
      ctx.fillText('Nutze die URL unten', 100, 100);
      ctx.fillText('oder "In neuem Tab"', 100, 120);

      dom.qrModal.classList.remove('hidden');
    }

    // ---- Templates ----
    function initTemplates() {
      for (const [key, tmpl] of Object.entries(templates)) {
        const option = document.createElement('option');
        option.value = key;
        option.textContent = tmpl.name;
        dom.templateSelect.appendChild(option);
      }
    }

    // ---- Timer Cues ----
    function initTimers() {
      // Clear existing timers
      state.timers.forEach(t => clearInterval(t));
      state.timers = [];

      const timerEls = dom.prompterContent.querySelectorAll('.cue-timer');
      timerEls.forEach(el => {
        const seconds = parseInt(el.dataset.timer, 10);
        el.querySelector('.timer-value').textContent = seconds + 's';
        el.dataset.remaining = seconds;
        el.dataset.started = 'false';
      });
    }

    function checkTimerVisibility() {
      if (!state.playing) return;

      const timerEls = dom.prompterContent.querySelectorAll('.cue-timer');
      const viewCenter = state.viewportHeight / 2;

      timerEls.forEach(el => {
        const rect = el.getBoundingClientRect();
        const isVisible = rect.top < viewCenter + 100 && rect.bottom > viewCenter - 100;

        if (isVisible && el.dataset.started === 'false') {
          el.dataset.started = 'true';
          let remaining = parseInt(el.dataset.remaining, 10);
          const valueEl = el.querySelector('.timer-value');

          const interval = setInterval(() => {
            remaining--;
            if (remaining <= 0) {
              clearInterval(interval);
              valueEl.textContent = 'Fertig!';
              el.style.borderColor = '#0f0';
              el.style.color = '#0f0';
            } else {
              valueEl.textContent = remaining + 's';
            }
          }, 1000);

          state.timers.push(interval);
        }
      });
    }

    // Run timer check periodically when playing
    setInterval(() => {
      if (state.playing) checkTimerVisibility();
    }, 500);

    // ---- Touch Support ----
    let touchStartX = 0;
    let touchStartY = 0;

    function handleTouchStart(e) {
      touchStartX = e.touches[0].clientX;
      touchStartY = e.touches[0].clientY;
    }

    function handleTouchEnd(e) {
      if (state.currentView !== 'prompter') return;

      const dx = e.changedTouches[0].clientX - touchStartX;
      const dy = e.changedTouches[0].clientY - touchStartY;

      // Tap (minimal movement) = play/pause
      if (Math.abs(dx) < 20 && Math.abs(dy) < 20) {
        // Only if not on a button
        if (!e.target.closest('button, input')) {
          togglePlay();
          resetControlsTimeout();
        }
      }
    }

    // ---- Check for Remote Mode (URL hash) ----
    function checkRemoteMode() {
      if (window.location.hash === '#remote') {
        showView('remote');
        // Ping main to get current state
        if (channel) {
          channel.postMessage({ type: 'ping' });
        }
        setTimeout(() => {
          if (dom.remoteStatus.textContent === 'Verbinde...') {
            dom.remoteStatus.textContent = 'Verbunden (warte auf Teleprompter)';
          }
        }, 2000);
      }
    }

    // ---- Event Listeners ----
    function init() {
      // Restore saved state
      restoreState();

      // Init
      initTemplates();
      initBroadcastChannel();

      // View switching
      dom.btnStart.addEventListener('click', startPrompter);
      dom.btnBack.addEventListener('click', backToEditor);

      // Playback
      dom.btnPlay.addEventListener('click', () => { togglePlay(); resetControlsTimeout(); });
      dom.btnReset.addEventListener('click', () => { resetScroll(); resetControlsTimeout(); });

      // Speed
      dom.speedSlider.addEventListener('input', (e) => setSpeed(parseFloat(e.target.value)));
      dom.speedSliderPrompter.addEventListener('input', (e) => { setSpeed(parseFloat(e.target.value)); resetControlsTimeout(); });

      // Font size
      dom.fontsizeSlider.addEventListener('input', (e) => setFontSize(parseInt(e.target.value, 10)));

      // Font toggle
      dom.btnFontToggle.addEventListener('click', toggleFont);

      // Fullscreen
      dom.btnFullscreen.addEventListener('click', toggleFullscreen);

      // File upload
      dom.btnFileUpload.addEventListener('click', () => dom.fileInput.click());
      dom.fileInput.addEventListener('change', (e) => {
        if (e.target.files[0]) handleFileUpload(e.target.files[0]);
      });

      // Drag & drop
      dom.editorView.addEventListener('dragover', (e) => {
        e.preventDefault();
        dom.editorView.classList.add('dragover');
      });
      dom.editorView.addEventListener('dragleave', () => {
        dom.editorView.classList.remove('dragover');
      });
      dom.editorView.addEventListener('drop', (e) => {
        e.preventDefault();
        dom.editorView.classList.remove('dragover');
        if (e.dataTransfer.files[0]) handleFileUpload(e.dataTransfer.files[0]);
      });

      // Templates
      dom.templateSelect.addEventListener('change', (e) => {
        const key = e.target.value;
        if (key && templates[key]) {
          dom.textInput.value = templates[key].text;
          updateEditorStats();
          saveText();
        }
        e.target.value = '';
      });

      // Editor stats
      dom.textInput.addEventListener('input', () => {
        updateEditorStats();
        saveText();
      });

      // Keyboard
      document.addEventListener('keydown', handleKeyboard);

      // Mouse movement shows controls
      dom.prompterView.addEventListener('mousemove', resetControlsTimeout);

      // Touch
      document.addEventListener('touchstart', handleTouchStart, { passive: true });
      document.addEventListener('touchend', handleTouchEnd, { passive: true });

      // Remote controls
      dom.btnRemoteQr.addEventListener('click', generateQR);
      dom.btnCloseQr.addEventListener('click', () => dom.qrModal.classList.add('hidden'));
      dom.btnCopyRemoteUrl.addEventListener('click', () => {
        const url = window.location.href.split('#')[0] + '#remote';
        navigator.clipboard.writeText(url).then(() => {
          dom.btnCopyRemoteUrl.textContent = 'Kopiert!';
          setTimeout(() => { dom.btnCopyRemoteUrl.textContent = 'URL kopieren'; }, 2000);
        });
      });
      dom.btnOpenRemote.addEventListener('click', () => {
        window.open(window.location.href.split('#')[0] + '#remote', '_blank');
      });

      // Remote buttons
      dom.remotePlay.addEventListener('click', () => sendRemoteCommand('toggle-play'));
      dom.remoteReset.addEventListener('click', () => sendRemoteCommand('reset'));
      dom.remoteSlower.addEventListener('click', () => sendRemoteCommand('speed-down'));
      dom.remoteFaster.addEventListener('click', () => sendRemoteCommand('speed-up'));

      // Window resize
      window.addEventListener('resize', () => {
        state.viewportHeight = window.innerHeight;
        if (state.currentView === 'prompter') {
          state.contentHeight = dom.prompterContent.scrollHeight;
          updateProgress();
        }
      });

      // Check URL hash for remote mode
      checkRemoteMode();
      window.addEventListener('hashchange', checkRemoteMode);

      // Initial stats
      updateEditorStats();
    }

    init();
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify in browser**

Open `index.html` in browser. Check:
1. Editor view shows with textarea, toolbar, and Start button
2. Type text, click Start — switches to teleprompter view
3. Click Play — countdown appears (3-2-1) then text scrolls
4. Space pauses/resumes, arrow keys change speed
5. Esc goes back to editor
6. Click "Remote" — QR modal shows with URL
7. Open URL with `#remote` in new tab — remote controls appear
8. Drag a .txt file onto the editor — text loads
9. Select a template — text populates
10. Font size slider works, font toggle works
11. Progress bar and time estimate update during scroll
12. Refresh browser — last text and settings persist

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: complete teleprompter app

Single-file HTML teleprompter with auto-scroll, podcast cues,
keyboard shortcuts, templates, and BroadcastChannel remote control."
```

---

### Task 2: Polish & Edge Cases

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Test edge cases and fix issues**

Test and verify these scenarios:
1. Empty text — Start button should be disabled or show warning
2. Very long text (1000+ lines) — scroll performance stays smooth
3. All visual cues render correctly: `[PAUSE]`, `[LANGSAM]`, `[BETONEN]text[/BETONEN]`, `[TIMER 5s]`
4. Timer cues count down when they enter the focus zone
5. Section dividers (`## Titel`) render as colored bars
6. Mobile: open on phone, tap to play/pause, controls usable
7. Fullscreen works (F key and button)
8. Remote tab receives state updates and controls work
9. LocalStorage persists across refresh

- [ ] **Step 2: Fix any issues found, commit**

```bash
git add index.html
git commit -m "fix: polish edge cases and verify all features"
```

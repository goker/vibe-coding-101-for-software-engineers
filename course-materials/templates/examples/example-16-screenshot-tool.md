# PRD Example 16: Screenshot Annotation Tool

> **Difficulty:** Intermediate | **Project Type:** Desktop App | **Time:** 6-8 hours

---

## Overview

| | |
|---|---|
| **What** | A desktop app for capturing screenshots and adding simple annotations |
| **Who** | Developers and designers who need quick annotated screenshots for docs or bug reports |
| **Why** | Faster than opening full image editors for simple annotations |

---

## Core Features (MVP)

1. **Capture Screen:** Hotkey triggers full-screen or region capture
2. **Draw Arrows:** Click and drag to add red arrows
3. **Add Text:** Click to place text labels
4. **Draw Rectangles:** Highlight areas with colored rectangles
5. **Save/Copy:** Save to file or copy to clipboard

---

## Non-Goals

**Will NOT build:**
- Screen recording or video
- Blur/pixelate tool
- Freehand drawing
- Multiple pages or layers
- Cloud upload or sharing
- Undo history (only single undo)
- Image editing (crop, resize, filters)
- Automatic OCR or text extraction

**Will NOT use:**
- Cloud APIs or storage
- Complex image processing libraries
- Database

---

## Technical Constraints

| | |
|---|---|
| **Language** | TypeScript |
| **Framework** | Electron |
| **Drawing** | HTML Canvas |
| **Dependencies** | electron, electron-builder |
| **Testing** | Playwright for E2E |
| **Platform** | macOS and Windows |

---

## Success Criteria

- [ ] Global hotkey (Cmd+Shift+4 / Ctrl+Shift+4) triggers capture
- [ ] Region selection works with click and drag
- [ ] Can draw arrows, rectangles, and text
- [ ] Save exports PNG to user-selected location
- [ ] Copy puts image in system clipboard
- [ ] App runs on macOS and Windows
- [ ] Basic E2E tests pass

---

## Implementation Phases

### Phase 1: Electron Setup & Screen Capture
**Goal:** Set up Electron app with screen capture

**Tasks:**
1. Create Electron project with TypeScript
2. Register global hotkey for capture
3. Implement full-screen capture using Electron desktopCapturer
4. Implement region selection overlay

**Verification:**
```bash
npm start
# App runs in system tray

# Press Cmd+Shift+4:
# Screen dims, drag to select region
# Screenshot appears in editor window
```

**Deliverables:** Electron app with capture working

---

### Phase 2: Canvas Editor & Annotations
**Goal:** Implement drawing tools

**Tasks:**
1. Create canvas-based editor window
2. Implement arrow tool (click start, drag to end)
3. Implement rectangle tool (click corner, drag to opposite)
4. Implement text tool (click to place, type to add)
5. Add color picker (red, blue, green, yellow, black)

**Verification:**
```
1. Capture screenshot
2. Select arrow tool → draw arrow on image
3. Select rectangle → draw highlight box
4. Select text → click and type label
5. Annotations appear on canvas
```

**Deliverables:** Working annotation tools

---

### Phase 3: Save, Copy & Testing
**Goal:** Export options and tests

**Tasks:**
1. Implement "Save" with file dialog
2. Implement "Copy to Clipboard"
3. Add simple undo (revert last action)
4. Write Playwright E2E tests
5. Build for macOS and Windows

**Verification:**
```bash
# Click "Save":
# File dialog opens, save as PNG

# Click "Copy":
# Image available in clipboard, paste works

npm run build
# Creates installers for macOS and Windows
```

**Deliverables:** Complete app with builds

---

## UI Layout

```
┌─────────────────────────────────────────┐
│  [Arrow] [Rect] [Text] | Color: [●●●●] │  ← Toolbar
├─────────────────────────────────────────┤
│                                         │
│           Screenshot Canvas             │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│  [Undo]              [Copy] [Save]      │  ← Action bar
└─────────────────────────────────────────┘
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Save with transparency support | Before adding new tools | Upload to cloud |
| Show visual feedback for selected tool | Before changing hotkeys | Access files outside project |
| Confirm before closing unsaved work | Before changing canvas size | Record screen/video |
| Support both PNG and clipboard output | | Add complex image editing |

# CloudKick — EXP-B: Browser WebAssembly PSP

**Experiment Type:** Path B — Browser-Native PSP via WebAssembly  
**Status:** 🟡 QUEUED — Awaiting VPS SSH  
**Owner:** Neo Stark (R&D) — GINPLAY / Quonord  

---

## What This Experiment Proves

Can PPSSPP compiled to WebAssembly run a PSP football game (PES / Winning Eleven) directly inside a user's browser — with the VPS handling only multiplayer sync, not rendering?

This approach eliminates video streaming latency entirely. The game engine runs client-side.

---

## Architecture

```
[Browser Client — WebAssembly]
  ├── ppsspp-wasm (PPSSPP compiled to WASM via Emscripten)
  ├── PSP ISO (loaded from Firebase Storage or direct upload)
  ├── Gamepad API (controller input — local, zero latency)
  └── WebSocket / WebRTC Data Channel
         ↓ (game state sync only — no video stream)
[VPS / Firebase Server]
  ├── Multiplayer sync server (AdHoc relay)
  ├── Match state (scores, events)
  ├── Firebase Auth (user identity)
  └── Firebase Realtime DB (match data, bets)
```

---

## Key Difference vs EXP-A

| | EXP-A (Server Stream) | EXP-B (Browser WASM) |
|---|---|---|
| Game runs on | VPS server | User's browser |
| Network carries | Full video stream | Game state only |
| Latency risk | HIGH (video RTT) | LOW (no rendering lag) |
| Server cost | High (GPU eventually) | Low (sync only) |
| Offline play | ❌ No | ✅ Possible |
| Mobile support | ✅ Browser stream | ✅ WASM in mobile browser |

---

## Tech Stack

| Component | Tool |
|---|---|
| PSP Emulator (browser) | [ppsspp-wasm](https://github.com/root-hunter/ppsspp-wasm) (WebAssembly/Emscripten) |
| Build System | Emscripten SDK |
| Multiplayer Relay | PPSSPP AdHoc relay server (VPS) |
| Auth | Firebase Authentication |
| Game Session DB | Firebase Realtime Database |
| Game Files | Firebase Storage (ISO hosting) |
| Frontend | React / Vanilla JS + WASM loader |
| Hosting | Firebase Hosting (frontend) + VPS (relay) |

---

## Success Metrics

| Metric | Target | Result |
|---|---|---|
| WASM loads in browser | ✅ | — |
| PSP game boots in browser | ✅ | — |
| Controller input works | Responsive | — |
| Two players sync via relay | Match completes | — |
| Load time | < 30 seconds | — |
| Mobile browser support | Chrome Android | — |

---

## Experiment Steps

### Step 1 — Clone ppsspp-wasm
```bash
git clone https://github.com/root-hunter/ppsspp-wasm.git
cd ppsspp-wasm
```

### Step 2 — Build with Emscripten
```bash
# Install Emscripten
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk && ./emsdk install latest && ./emsdk activate latest
source ./emsdk_env.sh

# Build PPSSPP for WASM
cd ../ppsspp-wasm
emcmake cmake . && make -j4
```

### Step 3 — Deploy Frontend
```html
<!-- index.html — minimal launcher -->
<canvas id="canvas"></canvas>
<input type="file" id="iso-loader" accept=".iso,.cso">
<script src="ppsspp.js"></script>
```

### Step 4 — Setup AdHoc Relay on VPS
```bash
# PPSSPP includes a built-in relay server
# Point both browser instances to this IP for multiplayer
./PPSSPPHeadless --adhocserver
```

### Step 5 — Firebase Integration
```javascript
// firebase-config.js — keys injected here for Phase Zero
// Security hardening done in Phase One
const firebaseConfig = {
  apiKey: "FIREBASE_API_KEY",
  authDomain: "cloudkick.firebaseapp.com",
  databaseURL: "https://cloudkick-default-rtdb.firebaseio.com",
  projectId: "cloudkick",
  storageBucket: "cloudkick.appspot.com"
};
```

---

## Latency Log

| Date | Player 1 Location | Player 2 Location | Sync Lag (ms) | Pass/Fail |
|---|---|---|---|---|
| — | — | — | — | — |

---

## Notes

- ISO file can be hosted on Firebase Storage and streamed into WASM memory
- Phase Zero: keys hardcoded for speed. Phase One: env vars + secret manager
- This path may become the PRIMARY architecture if latency results beat EXP-A
- Future: package this as an Android/iOS app using Capacitor or native build


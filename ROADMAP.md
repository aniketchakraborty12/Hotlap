# 🗺️ Hotlap: Implementation Roadmap & Architecture Blueprint

This document serves as your offline, step-by-step master guide for building **Hotlap**. Refer to this blueprint to maintain a strict separation of concerns:
* **Rust (The Pit Crew):** Single source of truth for time, OS threads, multi-window layout, file tracking, and secure economy logic.
* **React & Tailwind (The Dashboard):** Stateless UI, fluid hardware-accelerated animations, and responsive state synchronization.

---

## 📁 Directory Structure Blueprint

To keep the codebase modular, the frontend isolates its dashboard tabs, and the Rust backend avoids cluttering `src/` by organizing code into specialized module folders using `mod.rs`.

```text
hotlap/
├── public/                     # Static assets accessible directly by the webview
│   └── circuits/               # Acquired F1 Circuit Repositories
│       └── detailed/
│           └── white-outline/  # 500x500 pre-optimized high-contrast track maps
├── src/                        # React Frontend (Vite + Tailwind v4)
│   ├── assets/                 # SVGs, sound clips (F1 lights, wheel guns)
│   ├── components/             # Reusable UI widgets
│   │   ├── TrackTelemetryStage.tsx # The active racing canvas & GPU car dot
│   │   ├── StagingGrid.tsx        # The full-screen pre-race selection carousels
│   │   ├── ChequeredFlagOverlay.tsx # Phase 3: The race completion & points reward overlay
│   │   └── GachaShowcaseModal.tsx   # Phase 4: The 3D pack ripping and card flip UI
│   ├── hooks/                  # Global hooks & event listeners (useTauriEvents)
│   ├── store/                  # Global frontend state & circuit configuration data
│   │   ├── useRaceStore.ts         # Central race lifecycle, dynamic timer state & gacha triggers
│   │   └── circuitRegistry.ts      # Static track metadata, asset paths & sector math
│   └── views/                  # Primary Dashboard layouts / Application tabs
│       ├── Dashboard/          # The Pit Wall / Active Racing HUD
│       ├── Garage/             # The Driver/Constructor Gacha gallery (Pending UI)
│       └── TrophyCabinet/      # Historical performance metrics & graphs
│
├── src-tauri/                  # Native Rust Backend
│   ├── src/
│   │   ├── timer/              # Thread-safe countdown mechanics
│   │   │   ├── mod.rs          # Exposes timer commands, StartRaceRequest struct & event emitters
│   │   │   └── ticker.rs       # Active thread loops (Tokio intervals) & zero-intercept completion logic
│   │   ├── storage/            # Local user profile management & secure economy
│   │   │   ├── mod.rs          # Exposes profile reading/writing, atomic saves & race-completion payload struct
│   │   │   └── gacha.rs        # Native RNG logic (rand v0.10), card pool, variant definitions & unbox command
│   │   └── main.rs             # Application bootstrapper, window manager & command registry
│   ├── tauri.conf.json         # Desktop permissions, window setups, and build targets
│   └── Cargo.toml              # Rust dependency manager (uuid, rand v0.10)
│
└── ROADMAP.md                  # This master reference document
```

---

### Phase 1: The Engine Block (Core Timer & Bridge Setup) — [COMPLETED ✅]
*Goal: Establish a drift-free, native countdown loop that drives the React UI.*

* **1. Scaffold & Theme:** Project initialized via Vite + React + TS + Tailwind v4.
* **2. The Native Rust Ticker:** Built a native Tauri command system inside `src-tauri/src/timer/`. Spawns a background thread utilizing `tokio::time::interval` running at **10ms intervals** to broadcast high-frequency `timer-tick` updates down to the client.
* **3. Global UI State Management:** Implemented a centralized Zustand store (`useRaceStore`) utilizing strict `RaceStatus` typings (`'Idle' | 'Racing' | 'PitStop'`). Decouples frontend mutations and synchronizes state updates.
* **4. The Telemetry HUD:** Built a minimalist high-contrast pit-wall HUD component that binds directly to the Zustand store, preventing React render thrashing during 10ms ticks.

#### 📄 Core Files & Architecture (Phase 1)

**`src/store/useRaceStore.ts`**
The central nervous system of the frontend. Acts as a state container that defines strict type unions for race statuses to prevent illegal state transitions. It is engineered to handle high-frequency 10ms tick events by locking in the total session duration on the very first tick, preventing division-by-zero jitter. It then computes and securely clamps the progress percentage between 0 and 100, exposing actions to control the entire race lifecycle.

**`src-tauri/src/timer/mod.rs` & `ticker.rs`**
The native backend modules responsible for exact timekeeping. These files spawn an OS-level background thread via Tokio that runs a strict 10ms interval loop, completely decoupled from the webview's rendering cycle. It broadcasts system events carrying the remaining millisecond payload directly to the frontend, guaranteeing zero timer drift even if the UI thread stutters.

---

### Phase 2: The Aerodynamics (Track Paths, Live Telemetry & Pre-Race Configuration) — [COMPLETED ✅]
*Goal: Bring the F1 aesthetic to life using high-performance, GPU-friendly asset mapping, and build a cinematic, full-screen pre-race staging environment.*

* **1. Track Asset Registry Setup [ASSETS ACQUIRED 📦]:** Ingested 24 pre-optimized `500x500` Detailed Style (White Outline) SVGs into the public directory. All inner telemetry rail paths explicitly defined with unique IDs. Built a centralized TypeScript registry mapping circuit keys to filenames, lengths, dynamic asset paths, and decimal-based sector breakpoints.
* **2. Immersive Pre-Race Staging View:** Developed a dedicated full-screen configuration interface that entirely replaces the standard layout when idle. Features a "lurking chrono"—a massive, low-opacity background timer that dynamically morphs to reflect the user's selected session duration, setting an atmospheric baseline before the race begins.
* **3. Dynamic Session Strategy Selection:** Replaced hardcoded timer durations with a selectable array of F1-flavored time presets (e.g., Qualifying, Sprint Stint, Grand Prix). Selections instantly update both the global state and the background lurking chrono.
* **4. High-Performance Paginated Carousels:** Eliminated standard browser scrollbars in favor of a custom windowed-slider mechanism. Utilizes a dynamic measurement hook to calculate the exact pixel width of carousel items and gaps on the fly. Navigation is handled via UI arrows that trigger frictionless, GPU-accelerated `translateX` sliding animations, driven by a custom cubic-bezier curve to replicate the heavy, mechanical clunk of physical pit board equipment.
* **5. Inner-Zoom Selection Mechanic:** Solved carousel viewport clipping issues by implementing an inner-zoom architecture. Instead of scaling the selection cards themselves (which breaks parent overflow bounds), the cards maintain rigid dimensions while an inner wrapper scales the contents, providing a punchy, focused zoom effect without breaking the slider math.
* **6. The Dynamic UI Engine Wrapper:** Built a highly reusable component that dynamically positions selected circuit data frames via an HTML object tag, eliminating static hardcoding for individual layouts.
* **7. The GPU-Accelerated Car Dot:** Superimposes a stylized vector marker over the canvas layout layer. Harnesses native hardware acceleration by mapping position via modern CSS offset properties and automatic rotation.
* **8. High-Frequency Fluid Translation:** Streams countdown parameters directly from the Zustand store into the style runtime layout. Omits standard CSS transition latencies to match the exact resolution of the 10ms Rust ticker.
* **9. Sector Breakpoint Splitting:** Segments the session completion percentage into 3 chronologically distributed timeline clusters. Triggers reactive UI accent updates as milestones are reached.
* **10. Conditional Layout Switching:** The main Dashboard view seamlessly toggles between the full-screen staging grid (when Idle) and the strict side-by-side Pitwall HUD (when Racing/Pit Stop), ensuring maximum screen real estate is always dedicated to the current task.

#### 📄 Core Files & Architecture (Phase 2)

**`public/circuits/detailed/white-outline/*.svg`**
A standardized 24-piece asset library locked to a strict 4-layer architectural spec. Layer 1 is the thick visual track outline. Layer 2 is the filled start line marker (which can be arbitrarily rotated via matrix transforms). Layer 3 is the thin stroked direction arrow. Layer 4 is the hidden black center-line rail, equipped with a unique ID, serving purely as the mathematical track geometry for the car dot.

**`src/store/circuitRegistry.ts`**
A static configuration dictionary mapping string keys to track metadata. Contains human-readable names, geographic locations, relative asset paths for optimized rendering, physical track lengths, and decimal-based sector breakpoint fractions used by the UI to trigger visual color shifts.

**`src/store/useRaceStore.ts` (Phase 2 Additions)**
Expanded to hold dynamic pre-race configuration state. Now securely stores the currently selected session duration, which is passed directly to the Rust backend on race initialization, bridging the gap between the UI selection and the native OS thread.

**`src/components/StagingGrid.tsx`**
The visual centerpiece of the pre-race experience. A purely presentational component that orchestrates the track and time carousels. It manages localized pagination indices, interfaces with the pixel-width measurement hook to calculate precise sliding transforms, and renders the atmospheric background elements.

**`src/components/TrackTelemetryStage.tsx`**
The heaviest rendering component during an active race. Uses an HTML object tag to load the SVG cross-origin, allowing secure DOM traversal of the internal SVG document. Features a three-tier fallback algorithm to locate the start line. It uses a 300-point ray-cast sampling algorithm to map the start line's 2D coordinates to a precise percentage offset along the hidden rail path. Finally, it utilizes a ResizeObserver to dynamically calculate scaling transforms, ensuring the absolutely-positioned overlay dot remains perfectly anchored to the vector path regardless of window size, driven purely by GPU-accelerated CSS offset properties with zero transition latency.

**`src/views/Dashboard/index.tsx`**
The primary application shell. Functions as the bridge between the Tauri event bus and the Zustand store. Subscribes to native backend emissions and delegates those payloads to the store. Now acts as a layout router, conditionally rendering either the full-screen Staging Grid or the split-screen Race HUD based on the strict state machine, passing the derived progress percentage and active circuit configuration down to the telemetry stage.

---

### Phase 3: The Factory (Lightweight Storage & Progression) — [COMPLETED ✅]
*Goal: Securely persist user achievements without introducing complex database overhead.*

* **1. Atomic Profile Storage:** Bypassed heavy SQL and plugin stores in favor of raw `serde_json` crate implementations. Built a robust read/write layer in `storage/mod.rs` that utilizes a temporary file write-and-rename strategy to guarantee zero file corruption even if the application crashes mid-save.
* **2. Dynamic Point Allocation Logic:** When the native Tokio ticker hits zero milliseconds, it intercepts the loop, halts the thread, and fires a completion function. This function calculates F1-scaled Championship Points (CP) dynamically based on the completed session duration (Sprint, Qualifying, Grand Prix scales), appends a uniquely generated UUID race record to the user's history log, and saves the state.
* **3. Structured Event Broadcasting:** Replaced simple scalar emissions with a strongly-typed `RaceCompletePayload` struct that is emitted directly to the webview pipeline upon completion, containing points awarded, new totals, and full race metadata.
* **4. The Chequered Flag View:** Built an immersive, F1 broadcast-style overlay component (`ChequeredFlagOverlay.tsx`). Triggered by the frontend store state change, it renders a geometric data card featuring a dynamic checkered background pattern, formatted focus time, CP payout allocation, and an updated career balance.
* **5. State Hydration Loop:** Implemented a bootstrapping invoke command in the Dashboard view that queries the backend for the user's `profile.json` on mount, ensuring the frontend UI accurately reflects historical point balances across app restarts.

#### 📄 Core Files & Architecture (Phase 3)

**`src-tauri/src/storage/mod.rs`**
Handles the physical mapping to the OS-specific App Data directory. Contains the `UserProfile` and `RaceRecord` serializable structs. Implements the atomic `save_profile` logic and the `process_race_completion` interception point that bridges the timer module and the storage module.

**`src/components/ChequeredFlagOverlay.tsx`**
A purely presentational modal component driven by Zustand boolean flags. Features a dynamic scale-and-skew entry animation and parses the incoming payload to display contextual track names and session formats.

**`src/views/Dashboard/index.tsx` (Phase 3 Additions)**
Expanded to include hydration effects on mount, a secondary Tauri event listener specifically mapped to the `race-complete` event, and a header UI element displaying the live `championshipPoints` balance pulled from the store.

---

### Phase 4: The Garage (Gacha & Driver Collections) — [IN PROGRESS 🏗️]
*Goal: Build a compelling visual dopamine loop that encourages repeat focus sessions.*

* **1. Expandable Card Vault Architecture:** Designed a highly scalable static pool in Rust supporting both Drivers and Teams. Introduced a `CardCategory` enum and an optional `subtitle` string to natively support "Chase Variants" (e.g., Standard Verstappen vs. 2023 Championship Edition Verstappen) without frontend logic bloat.
* **2. Secure Native RNG Engine:** Built the `unbox_pack` logic in `storage/gacha.rs` utilizing the `rand` v0.10 crate and its `RngExt` trait. Implements a strict 60/30/10 weighted probability matrix for Common/Rare/Legendary drops. Securely handles point deduction, duplicate detection (yielding an `is_new` boolean flag), and atomic inventory array updates.
* **3. The 3D Unboxing Card Flip:** Developed a high-fidelity CSS 3D transform engine in `GachaShowcaseModal.tsx`. Utilizes Tailwind v4 `@layer utilities` to enforce `preserve-3d` and `perspective-1000` contexts. Features a two-stage physical interaction: a pack seal rupture state followed by a timed 180-degree Y-axis flip.
* **4. Telemetry Shimmer Effects:** Integrated a continuous diagonal sweep animation (`telemetry-shimmer`) that dynamically renders over the card face strictly for Rare and Legendary tier pulls, visually reinforcing the RNG outcome.
* **5. Grid Gallery Showcase:** [PENDING] Create a secondary grid screen in React displaying the player's collection. Unlocked items shine in full vibrant colors, while missing items remain locked as dark silhouettes, urging them to finish their next stint.

#### 📄 Core Files & Architecture (Phase 4)

**`src-tauri/src/storage/gacha.rs`**
The secure economy engine. Houses the `PoolEntry` constant array, the `GachaCard` serialization struct, and the unboxing business logic. Completely decoupled from the UI, ensuring gacha rates and inventory states cannot be manipulated via browser DevTools.

**`src/store/useRaceStore.ts` (Phase 4 Additions)**
Extended to include the `GachaCard`, `CardCategory`, `UnboxResult` TypeScript interfaces (perfectly mirroring the Rust structs). Added state hooks for `currentUnboxResult` and `isUnboxingModalOpen` to drive the UI flip animation lifecycle.

**`src/components/GachaShowcaseModal.tsx`**
The visual execution layer. Reads the strongly-typed payload to dynamically apply rarity-specific border glows (Teal for Rare, Orange for Legendary), render Category badges, and prominently display Chase Card subtitles. Includes a header-mounted "Buy Pack" trigger button with conditional disabling when funds are below the 100 CP threshold.

---

### Phase 5: The Pit Wall (The Multi-Window Mini HUD) — [PENDING ⏸️]
*Goal: Exploit Tauri's unique native windowing mechanics to build a floating productivity widget.*

* **1. The Minimalist HUD Canvas:** Create a highly isolated React view that renders nothing but a stripped-back countdown timer and a linear micro-progress bar representing the car's current lap position.
* **2. Decoupled Native Window Configuration:** In your `tauri.conf.json`, declare a secondary window labeled `pitwall`. Give it native custom flags: `visible: false` (hidden by default), `decorations: false` (removes OS window borders and title bars), `always_on_top: true` (keeps it floating over code editors/browsers), and `transparent: true` (allows clean, rounded UI corners).
* **3. Native Window Intercept & Drag:** Add a custom button inside the main layout that invokes Tauri's window toggles to transition between the main dashboard and the HUD overlay. Assign the `data-tauri-drag-region` attribute to the mini HUD container so users can drag it anywhere across multiple monitors.
* **4. Multi-Window Synced Broadcast:** Because both the main dashboard and the mini HUD listen directly to the uniform Rust `timer-tick` system events, both windows will remain perfectly synchronized down to the exact millisecond.

---

### Phase 6: Race Strategy (Theme Swap & Extra Juice) — [PENDING ⏸️]
*Goal: Differentiate breaks from work sessions and add sensory feedback.*

* **1. Pit Stop Shift:** When transitioning into a short break, morph the interface into a completely new theme. Replace the racing track with an interactive grid of tire-change progress bars, symbolizing a rapid, frantic pit stop reset.
* **2. Audio Feedback:** Integrate lightweight sound triggers. Play the classic F1 5-light sequence alarm when a session initializes, pneumatic wheel gun noises for break starts, and a grand team-radio clip when crossing the chequered flag.
* **3. Native Desktop Notifications:** Wire up Tauri's system notification API to push an OS-level alert when a session or pit stop finishes, ensuring the user stays on pace even when the application is out of focus.
* **4. The Trophy Cabinet:** Synthesize historical logs from your storage module to render personalized telemetry stat cards: "Total Stints Handled," "Favored Circuit," and an interactive performance graph measuring focus stamina across days.
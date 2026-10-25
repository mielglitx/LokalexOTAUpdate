# Lokalex App Release Notes

## Version 1.0.55 (2026-10-25)

### 1. Store Menu Camera & Photo Quality Overhaul
* **Auto-Orientation Detection:** Camera detects device orientation (portrait vs. landscape) in real time via gyroscope and screen orientation APIs, automatically orienting captured photos before preview and upload.
* **Pinch-to-Zoom & Hardware Zoom Slider:** Added interactive zoom gestures and a hardware zoom slider using native MediaStream track capabilities.
* **Sharpness & Speed Optimization:** Eliminated photo blurriness by capturing at the camera sensor's maximum supported native resolution before compression, and accelerated image processing to eliminate capture lag.

### 2. Live Chat Real-Time Presence, Typing & Seen Indicators
* **In-Thread Presence Banner:** Displays a real-time status banner at the top of the chat thread showing who is currently viewing/opening the chat.
* **Typing Indicator Bubble:** Shows real-time typing indicators both inside the message stream ("typing...") and on chat feed cards.
* **"Seen" Tracking:** Tracks and displays who has opened and viewed messages in the chat thread.

### 3. Alarm Chimes Filtering (Catered Order Sound Isolation)
* **Stopped Non-Catered Chimes:** Riders now only receive acoustic notification alarm chimes for customers they are actively catering (`isMyCustomerThread === true`).
* **Removed Admin Chime Bleed:** Removed an override that previously caused team leaders/admins to hear chimes for every customer catered by other riders.
* **Live Roster Caterer Resolution:** Added `getActiveRosterCaterer` before chime evaluation so orders catered in the active roster never falsely fall into the general inbox chime.
* **Inbox Chime Silencing:** Silenced unassigned inbox/applicant/done chimes on active rider sessions.

### 4. Rider Profile GCash Persistence
* **Instant Profile Credential Sync:** Fixed the bug where saving GCash details in profile settings didn't reflect when opening the rider's profile modal.
* **Multi-Node Sync:** Unified syncing across Firebase RTDB (`riders/`, `roster/`, and `gcash/`) and local storage.
* **Profile Display:** Added GCash Name and Number fields directly to the Rider Profile view.

### 5. Customer Directory Linking & Folder Preservation
* **Catering Folder Preservation:** Fixed the issue where linking a catered customer to an existing directory account with a slightly different name (e.g. *Zyra Mae Manalon* vs. *Zyra Mae Manalon Villegas*) kicked the thread into the inbox.

### 6. Stability & Startup Fixes
* **White Screen Fix:** Resolved the runtime `ReferenceError: startChatTypingListener is not defined` in `riderChat.js` by properly importing presence/typing listeners.
* **Syntax Cleanups:** Fixed parser diagnostics in `profileSave.js`.

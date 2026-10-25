# Lokalex App Release Notes

## Version 1.0.56 (2026-10-25)

### 1. Restaurant Menu Directory & Quick Send in Customer Chat
* **In-Chat Quick Menu Directory Picker:** Added dedicated header shortcut pill (`#rider-chat-menu-pill`) and action bar button (`#rider-chat-send-menu-btn`) inside active customer chats to browse and dispatch physical store menus directly with 1 tap.
* **Store Deduplication & Unified Card Listing:** Fixed duplicate store cards (e.g. Chowking x2, Chooks To Go x2). Directory records and Firebase metadata entries are now unified into a single card per store with accurate page counts.
* **Special Character & Apostrophe Crash Fix (Cindy's):** Eliminated the syntax crash caused by decoded HTML entity apostrophes in inline onclick handlers. Switched to HTML5 data attributes (`data-store-name`, `data-composite-key`, `data-meta-key`) and robust event delegation.
* **Multi-Candidate Key Resolution ("No Saved Photos" False Alarm):** Fixed first-click failure on Chowking by automatically resolving multiple candidate key variants (`stores` + clean name, stripped municipality prefixes, metadata keys) across IndexedDB and Firebase RTDB.

### 2. Pre-Send & Post-Send In-App Confirmation Modals
* **Custom Pre-Send Confirmation Modal (`#menu-send-confirm-modal`):** Replaced native browser `confirm()` with a styled in-app modal showing Store badge, Customer badge, and exact page count (e.g., *"Send all 4 menu pages of Chowking to Amiel Juralbal Yalung?"*).
* **Custom Post-Send Confirmation Modal (`#menu-sent-confirmation-modal`):** Triggers immediately upon dispatch with recipient confirmation, a 1-tap **"View in Chat"** button that focuses and scrolls to the sent pages, and a **"Done"** dismiss button.
* **Consistent Across All Flows:** Fully integrated into Chat Quick Send, Store Directory Menu Gallery (Send All / Send Selected), and the Customer Recipient Picker.

### 3. "Seen by" Badges Placement (Accurate Last-Seen Anchor)
* **Customer Seen Badge Anchor:** Customer seen badge (`Seen by [Customer Name]` + avatar) now strictly attaches **underneath the last message sent by the rider that the customer has actually seen** (`isExplicitSeen || msgTs <= customerLastSeenTs || msgTs <= latestCustomerReplyTs`), never floating at the bottom when newer unread messages exist.
* **Rider Seen Badge Anchor:** Rider seen badge (`Seen by [Rider Name]` + badge) now strictly attaches **underneath the exact message that was on screen when the rider viewed the thread** (`msgTs <= seenAt`), never forced to the bottom if newer incoming customer messages have arrived.
* **Independent Multi-Participant Positioning:** Customer and Rider seen indicators are positioned independently at their respective last-seen milestones.

### 4. Facebook Messenger Image Dispatch & Cloudflare Worker Optimization
* **Base64 Payload Compression:** Added client-side canvas compression for large base64 menu photos before forwarding to Facebook Messenger via Cloudflare Worker. Guarantees payloads remain under 45KB, preventing HTTP 413 (Payload Too Large) and 502 Bad Gateway errors.
* **Instant Optimistic Chat UI Update:** Dispatched menu pages immediately reflect in the active chat bubble view without waiting for round-trip webhook echo.

### 5. App-Wide English Language Standardization
* **Complete English Localization:** Standardized notifications, toasts, modals, confirmation dialogues, action sheets, and status prompts across the entire app to clean, professional English.

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


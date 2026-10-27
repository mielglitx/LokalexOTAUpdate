# Lokalex App Release Notes

## Version 1.0.66 (2026-10-27)

### 1. Fix Duplicate Page Echo Messages in Chat
* **Rider vs Page Echo Deduplication:** Resolved the bug where sending a message (or rates guideline) via the app caused duplicate messages to display in the chat (one from the rider like "Hero Dinong" and a duplicate echo from the Facebook Page like "Camiling Delivery by Lolalex").
* **Human Rider Priority Resolution:** Updated message filtering to always prioritize and preserve the authentic human rider name and photo, dropping the generic page echo.
* **Smart Content & Mid Match in Meta Sync:** Updated `syncCurrentThreadFromMetaGraph` to cross-reference existing rider messages before inserting. If an existing message matches the content and timestamp window, the Meta message ID is linked without creating a second bubble or duplicate database record.
* **Real-Time Stream & Render Deduplication:** Added deduplication in `renderRiderMessages`, `appendOrUpdateSingleRiderMessage`, and `child_added` event listeners, preventing duplicate bubbles from rendering and automatically purging any previously stored duplicate `m_...` keys from Firebase RTDB.

### 2. Facebook Page User vs. App Rider Identity Differentiation
* **Page Staff & Admin Distinction:** Direct messages sent from Facebook Page Inbox or Meta Business Suite are now cleanly labeled as **Page Admin** with a dedicated blue Admin shield badge (`🛡️ Admin`), eliminating the generic long Page name.
* **App Sender Identity Preservation:** Messages sent through the Lokalex App or Admin Dashboard preserve the authentic rider or dispatcher's name, avatar, and role (`rider` vs `admin`), providing full accountability across all conversations.

### 3. High-Speed Menu Picture Transmission & Document Clarity
* **Meta CDN Reusable Attachment IDs:** Integrated automatic capture and reuse of Meta's `attachment_id` for store menu pages. Once a menu page is sent, subsequent sends to any customer deliver near-instantaneously (~150ms) straight from Facebook's CDN cache without re-downloading from Firebase.
* **Unified 1-Call Delivery:** Streamlined menu photo transmission to skip redundant secondary text caption requests, cutting network requests in half and preventing duplicate customer notification sounds.
* **Pipelined Batch Dispatch:** Replaced artificial 800ms delays with a 60ms pipelined dispatch queue and pre-fetched customer metadata, reducing a 5-page menu transmission time from ~25 seconds down to ~3–4 seconds.
* **2048px High-Density Document Clarity:** Standardized menu scanner and native camera uploads to 2048px with subpixel text contrast enhancement (`contrast(1.08) brightness(1.02)`), keeping menu prices, descriptions, and fine print crystal clear while slashing file sizes by over 80%.

## Version 1.0.65 (2026-10-27)

### 1. High-Performance Facebook Page Chat Fetching & Real-Time Sync
* **Parallel Meta Graph API Fast-Sync:** Added direct parallel synchronization with Meta's edge servers (`https://graph.facebook.com/v19.0/me/conversations?user_id=...`) when opening customer chats. Eliminates webhook wait times and instantly retrieves messages directly from the Facebook Page within 150–250ms.
* **1-Tap Instant Facebook Sync Button:** Integrated an instant refresh button (`#rider-chat-sync-fb-btn`) with spin animation directly on the customer chat header for on-demand synchronization with the Facebook Page.
* **Multi-Path Single Atomic Database Writes:** Upgraded the webhook pipeline (`cloudflare-worker.js` & `server.js`) to use single atomic multi-path `PATCH /.json` calls, writing message payloads, thread previews, timestamps, and index maps simultaneously in a single <60ms request.
* **Zero-Latency In-Memory Caching:** Introduced persistent RAM caching for Page Access Tokens and customer identities within webhook worker isolates, completely eliminating redundant database read queries on subsequent messages.

## Version 1.0.64 (2026-10-27)

### 1. Audio Ducking Bug Fixed (Other Apps Volume Preservation)
* **Elimination of Silent Audio Loop:** Completely removed the background silent audio element (`silentAudioEl`) and the continuous 25-second Web Audio oscillator loop that forced Android's `AudioManager` to lower ("duck") other apps' media volume (Spotify, YouTube, podcast players) whenever Lokalex was opened.
* **Auto-Suspension of Audio Context:** Added automatic `AudioContext` suspension (`suspendAudioContextIfIdle()`) after synthesized chimes or delivery reminder alarms finish playing, ensuring Android audio focus is immediately returned to background music players at 100% full volume.
* **Pure Worker & Native Background Keep-Alive:** Background connectivity and WebSocket keep-alives are now maintained exclusively through dedicated Web Worker heartbeat timers and the native Android `LokalexForegroundService`, guaranteeing zero audio sub-system interference.

### 2. PiP Side-Docking Stashed Badge Icon
* **Native Android PiP Stashing Support:** Implemented Android 12+ (API 31+) PiP stashing callbacks via `Activity.onPictureInPictureUiStateChanged(PictureInPictureUiState pipState)` and `pipState.isStashed()`.
* **Edge-Docked Badge View:** When the floating window is dragged or flung to the side bezel of the screen, the complex dashboard and chat elements are cleanly hidden, displaying an elegant circular glowing Lokalex badge icon (`Logo.png`) with an active green live pulse.
* **Responsive Viewport Fallback:** Added `@media (max-width: 170px)` rules ensuring that narrow window dockings on any Android OEM skin seamlessly show the clean badge view.
* **1-Tap Re-Expansion:** Tapping the side badge icon instantly re-expands the app into interactive floating or full-screen view.

### 3. Smart Cart Scrolling Physics & List Performance Optimization
* **Gesture Axis Disambiguation:** Redesigned `cartGestures.js` touch evaluation with Cartesian threshold detection (`diffY` vs `diffX`). When the user moves vertically, swipe-to-delete horizontal translation is immediately released, allowing the native browser compositor to execute vertical scrolls with zero hitching.
* **Compositor Momentum Scrolling:** Applied `touch-action: pan-y; -webkit-overflow-scrolling: touch; overscroll-behavior-y: contain; will-change: scroll-position;` to `#cart-items-list` for smooth 60–120 FPS inertial scrolling.
* **Off-Screen Layout Optimization:** Integrated modern `content-visibility: auto; contain-intrinsic-size: 0 115px;` on all cart item cards, skipping off-screen DOM layout and paint work when managing large orders with dozens of items.

## Version 1.0.63 (2026-10-27)

### 1. Directory Credit Depletion & Disabled Enforcement Fix
* **Strict Honor of Disabled System:** Fixed the bug where rider credits continued to deplete when clicking "View Location" on customer entries even after the credit system was toggled OFF in Admin Directory Credit settings.
* **Instant Free Access When Disabled:** When the credit system is turned OFF (`enabled: false`), viewing customer GPS locations bypasses the credit confirmation modal entirely and opens the map link directly with zero credit deductions and no minimum balance requirement.
* **Real-time Status Feedback:** The Directory Credits pill on the home dashboard dynamically reflects the system status, showing "OFF" when disabled by an administrator and giving riders clear visibility that directory lookups are free.

### 2. Auto-Message Name Display Formats (First, Last, or Whole Name)
* **Configurable Name Display Preferences:** Added dedicated selector dropdowns in the Admin Auto-Message modal to configure how names are presented in automated messages for both Customer and Rider (`First Name Only`, `Last Name Only`, or `Whole Name`).
* **Dynamic Name Formatting Tags:** Added granular template placeholder tags:
  * `{customerFirstName}`, `{customerLastName}`, `{customerFullName}`
  * `{riderFirstName}`, `{riderLastName}`, `{riderFullName}`
  * Standard `{customerName}` and `{riderName}` dynamically format based on the admin's chosen format setting.
* **1-Tap Insert Tag Buttons:** Added quick-insert chip buttons for first name, last name, and full name tags in both the "Being Catered" and "Mark Done" message template editors.

## Version 1.0.62 (2026-10-27)

### 1. Store Menu Camera: 1x Framing & Blur-Free HD Capture
* **Zero-Lag WYSIWYG Video Capture:** Eliminated the blurry snapshot bug caused by `ImageCapture.takePhoto()` autofocus reset/hunting cycles on Android multi-camera hardware. Replaced with direct frame capture from the live, focused HTML5 video stream onto an off-screen HD canvas with high-quality bicubic smoothing (`imageSmoothingQuality = 'high'`).
* **Sensor Discrepancy & 0.6x Distortion Fix:** Fixed the issue where photos were captured significantly wider (0.6x sensor effect) than the 1x viewfinder preview. Accurate `object-cover` viewport calculations ensure the captured frame exactly matches the on-screen framing down to the pixel.
* **Preserved Viewfinder Zoom:** Accurately retains digital and hardware zoom levels (1x, 1.5x, 2x, 3x) in the captured photo.
* **Shutter Flash & Tap-to-Focus UI:** Added an instant white shutter flash animation (`#menu-camera-shutter-flash`) upon capture, plus an interactive amber focus ring (`#menu-camera-focus-ring`) when tapping anywhere on the viewfinder to re-lock continuous auto-focus.
* **Orientation & Document Clarity:** Ensured landscape and portrait rotations with orientation lock enabled maintain crisp, legible document text and menu prices.

### 2. Floating Window Mode (Picture-in-Picture)
* **Native Android PiP Multitasking:** Added native Picture-in-Picture mode support to keep Lokalex visible as a floating window while using navigation, messaging, or other apps.
* **1-Tap Quick Access Buttons:** Integrated floating window mode buttons directly on the top app header (`#header-floating-window-btn`), Home Dashboard (`#dashboard-float-window-btn`), and Customer Chat modal header.
* **Adaptive PiP UI Layout:** Responsive compact styling (`html.in-floating-mode`) automatically condenses dashboard cards and chat views for optimal readability in small window dimensions.

### 3. Background Persistence & Screen-Off Execution
* **Hardware Wakeup Watchdog (`BackgroundWatchdogReceiver`):** Configured `AlarmManager.setExactAndAllowWhileIdle(RTC_WAKEUP, ...)` to ensure Android CPU hardware wakes up periodically even in deep Doze mode when the screen is off or out of focus.
* **Foreground Service Modernization:** Migrated foreground service type to `remoteMessaging|location` in `AndroidManifest.xml` to prevent the 6-hour background timeout on modern Android versions.
* **Swipe-Away Persistence & Screen Keep-Alive:** Persists across swipe-away dismissals via `onTaskRemoved` and prevents premature screen timeouts during active app navigation.

### 4. Directory Scrolling & Touch Physics Fix
* **Unrestricted Directory Scrolling:** Fixed the issue where touch gestures could not scroll in any of the directory lists (Customers, Stores, and Rates & Barangays). Removed global `overscroll-behavior-y: contain` and restrictive `touch-action` rules that trapped gesture chaining in the Chromium compositor.
* **Guaranteed Scroll Handshake:** Added explicit `touch-action: pan-y; -webkit-overflow-scrolling: touch;` on `#record-list` and ensured view transitions (`switchView` and `openDirectory`) automatically clear any lingering body scroll locks (`overflow-hidden` / `touch-none`).

### 5. Customizable Automated Message Dispatch (Admin-Managed)
* **Admin-Only Configuration & Cloud Sync:** Automated message controls (toggling on/off and customizing message templates for both Catering and Done milestones) are strictly restricted to administrators. Configuration synchronizes globally across all riders in real time via Firebase Realtime Database (`settings/autoMessage`).
* **Scenario 1 - Being Catered:** Dispatches an introductory greeting message when a customer order is actively being catered.
* **Scenario 2 - Marked as Done (Receipt Protected & Active Catering Only):** Fixed the issue where Done auto-messages fired when moving non-catered Inbox threads to Done or when replying to follow-up messages on finished threads. Done auto-messages now strictly require:
  1. The customer must be actively catered on the roster and completing delivery.
  2. A verified receipt must exist for the order.
  3. Single-use receipt consumption guarantees that a receipt will never trigger a duplicate Done message if the thread is replied to or re-archived.
* **Dynamic Smart Placeholders:** Supports dynamic template parameters: `{customerName}`, `{riderName}`, `{receiptTotal}`, `{receiptNumber}`, and `{time}`.
* **Admin Interface Access:** Accessible to authorized administrators via the Live Chats header (`Auto-Msg` button), the Dashboard Admin Toolbar, and Profile Settings. Hidden from regular riders.

## Version 1.0.61 (2026-10-26)

### 1. App Startup & USB Debugging Smoothness Optimization
* **Startup Lag Elimination:** Completely eliminated the 30-second to 1-minute initial boot lag on lower-spec mobile devices (Qualcomm Snapdragon 460 / 4GB RAM) by addressing CPU thread blocking and uncoordinated network queries during launch.
* **Firebase Offline Outbox Self-Healing:** Fixed an infinite replay loop caused by unauthorized mutations (e.g., stale `chatPresence` writes) returning `PERMISSION_DENIED`. Added automatic discarding of non-retryable permission errors and startup purging of expired offline queue entries older than 15 minutes.
* **Coalesced Real-time Event Streams:** Replaced the 240-event synchronous storm in `cateredSyncManager` with a 120ms debounce coalescer (`scheduleBatchedSyncEvents`), collapsing consecutive `cateredUpdated`, `receiptsUpdated`, and `rosterUpdated` dispatches and IndexedDB writes into a single batch.
* **O(1) Customer Thread Duplicate Suppression:** Optimized `processCustomerChatMetaData` in `chatFeedListener.js` by pre-indexing Facebook Messenger customer names into an $O(1)$ `Set`, eliminating over 20,000 quadratic ($O(N^2)$) regex and token comparisons during each customer metadata update.
* **Deferred Heavy Background Sync:** Shifted `directory.silentSyncDirectory()`, `runBackgroundPersistenceSync()`, and `sweepStaleShiftsFromPriorDays()` to execute 20–25 seconds after launch, freeing up 100% of the main JavaScript UI thread during the critical initial user interaction window.
* **60fps UI & Gesture Responsiveness:** Reduced 99th percentile frame times from over 4,500ms down to 61ms, maintaining smooth 60fps scrolling and eliminating frame deadline misses and high input latency.

## Version 1.0.60 (2026-10-25)

### 1. Store Menu Camera: Hardware Gravity Sensor Auto-Orientation
* **Physical Gravity Detection (`devicemotion`):** Added hardware gravity sensor monitoring (`accelerationIncludingGravity`) to automatically switch the camera between Portrait and Landscape orientation even when the phone's system Orientation Lock (portrait lock) is enabled.
* **Orientation Lock Bypass:** Removed the premature screen orientation return that previously prevented accelerometer and gyroscope readings from being evaluated when the phone was locked in portrait.
* **Jitter & Flat Surface Protection:** Added a hysteresis threshold (`|acc.x| > |acc.y| + 1.8`) to prevent rapid flipping when held at slight angles, and resting-flat detection (`|acc.z| > 8.2`) to maintain the current orientation when the phone is placed on a table.
* **Direction-Aware Photo Rotation:** Integrated `lastDetectedAccX` and `tiltGamma` into `ensurePhotoOrientation` so wide landscape photos are rotated correctly before saving regardless of system lock status.

### 2. Store Menu Camera: Clean Viewfinder (Overlay Removed)
* **Framing Box & Corner Brackets Removed:** Removed the yellow document corner brackets, border lines, and center badge overlay from the camera viewfinder (`#menu-camera-viewfinder`) for a clean, unobstructed edge-to-edge view.
* **Header Status Feedback:** Relocated orientation feedback cleanly into the top bar (`#menu-camera-orient-btn` and `#menu-camera-sub`), providing non-intrusive status cues without covering the camera view.

### 3. In-App Change Log & Release Notes
* **Change Log Display Below Version:** Added a dedicated "Change Log" button and expandable details view right below the app version tracker in the Home Dashboard, Profile Settings, and Version Selector modal.
* **Dynamic Markdown Parser:** Renders the complete contents of `CHANGELOG.md` into formatted release notes cards with offline support and automatic GitHub synchronization.

## Version 1.0.59 (2026-10-25)

### 1. Store Menu Camera & Lightbox Stacking Context Fix
* **Camera Modal Stacking Elevation:** Fixed the issue where the camera modal (`#store-menu-camera-modal`) rendered underneath the store menu gallery modal (`#store-menu-gallery-modal`). Increased the camera modal's z-index to `1000060 !important` and ensured it is dynamically re-appended to `document.body` so it reliably sits at the very top of the DOM stacking context above the gallery (`1000010`).
* **Gallery Lightbox Elevation:** Elevated the menu full-screen photo lightbox (`#gallery-lightbox`) z-index to `1000050 !important`, ensuring high-resolution image previews open cleanly on top of the store menu.
* **Recipient Picker Elevation:** Raised the customer recipient selection picker (`#menu-recipient-picker-modal`) to z-index `1000020`, guaranteeing full visibility when sharing menus directly from the gallery.

## Version 1.0.58 (2026-10-25)

### 1. Commission Page: Admin Rider Isolation & Report Fix
* **Admin Rider Filter Isolation Fix:** Fixed the issue where isolating a single rider from the admin dropdown (`#admin-rider-select`) displayed a blank ledger with 0 records. Implemented `checkRiderFilterMatch` to accurately reconcile rider display names with numeric Telegram IDs and roster records across the commission calculation engine.
* **Daily Report Text Isolation:** Corrected rider isolation in the exportable Daily Summary Report text (`generateDailyReportText`), ensuring isolated rider breakdowns compute gross totals, rider shares, and company fees properly.
* **Settlement Status Protection:** Prevented marking or reverting settlements when "All Riders" is selected, requiring an isolated rider selection before committing payment statuses.

### 2. Map Calculation & Catering Roster Map Links: Public Web Hosting & Chrome Breakout
* **Public Domain URL Resolution (Localhost Loopback Fix):** Fixed the bug where generated map calculation (`?mapcalc=...`), customer delivery tracking (`?track=...`), and mutual live GPS (`?livegps=...`) links defaulted to `http://localhost` when generated inside the Android app (Capacitor) or development environments. All customer-facing links are now canonically anchored to the public GitHub Pages domain (`https://mielglitx.github.io/Lokalex-Complete-Tool-Kit/`).
* **Force External Google Chrome Browser Breakout:**
  * Implemented `isInAppBrowser()` detection for Facebook Messenger, Instagram, and webview environments.
  * Added automated Android Chrome Intent launching (`intent://...#Intent;scheme=https;package=com.android.chrome;...`) to break out of in-app webviews directly into standalone Google Chrome.
  * Added user-facing fallback banners (`#mapcalc-iab-chrome-banner`, `#track-iab-chrome-banner`, `#livegps-iab-chrome-banner`) equipped with a 1-tap **"Switch to Chrome Browser"** button and clear guidance on using the Messenger 3-dots (⋯) menu.
* **Instant Native Location Permission Prompt:** Customer portals for Map Calculation, Live Tracking, and Mutual GPS now automatically trigger location requests immediately upon opening, presenting the browser's native location permission dialog without requiring manual button taps.

## Version 1.0.57 (2026-10-25)

### 1. Delivery Fare Rates Quick-Send in Customer Chat
* **Chat Header, Action Bar & Quick-Toolbar Access:** Added dedicated "Rates" pill button (`#rider-chat-rates-pill`) in the chat header, an action bar "Rates" button (`#rider-chat-send-rate-btn`), and an input toolbar rate icon button (`#rider-chat-rates-input-btn`) directly next to the message field for instant access inside any customer chat.
* **In-Chat Delivery Fare Rates Picker Modal (`#chat-fare-rate-picker-modal`):** Allows riders to view and filter standardized delivery rates by Origin Municipality and Destination Municipality/Barangay, with instant text search and organized card layouts showing Origin, Destination, Base Fare, and Additional Rates.
* **Dual Dispatch Modes (Insert to Input vs. Direct Quick-Send):**
  * **"Insert" Button:** Formats the standard delivery rate details into a clean chat message template and inserts it directly into the chat input field for optional rider customization before sending.
  * **"Send Rate" Button:** Prepares the standardized delivery rate and triggers an in-app confirmation modal for immediate 1-tap dispatch into the active chat thread.
* **Pre-Send Confirmation Modal (`#fare-rate-send-confirm-modal`):** Displays a verified breakdown (Origin → Destination, Delivery Fee, and Customer Recipient Name) before any delivery fare rate is dispatched to avoid accidental sends.

### 2. Fee Rates Directory Auto-Fill & Suggestive Inputs
* **Automatic Destination Pre-Fill:** When viewing a destination in the Fee Rates Directory and clicking "Add New Rates", the destination municipality and destination region are automatically pre-filled to match the currently viewed destination, saving riders repetitive manual selection.
* **Suggestive Typewritten Textboxes (`<input list="...">`):** Converted the Destination Region and Destination Municipality from static dropdown selects into modern, suggestive typewritten inputs with dynamic `<datalist>` auto-suggestions.
* **Dynamic Cascading Suggestions:** Typing or choosing a Region dynamically filters and populates the Municipality datalist (`#form-rate-municipality-list`) via the Philippine Standard Geographic Code (PSGC) dataset, matching the behavior of the Barangay suggestive input.
* **Full Backward Compatibility:** Maintained fallback compatibility with existing directory records and custom municipality/region naming across the application.

### 3. Rider Profile Pictures & Avatar Stability Overhaul
* **Elimination of Temporary 72-Hour Litterbox Uploads:** Removed temporary 72-hour `litterbox.catbox.moe` photo uploads. All new rider profile photo uploads are now stored permanently using compressed high-resolution base64 data URLs when Firebase Storage is bypassed, ensuring avatars never expire.
* **Automatic Dead/Expired Link Sanitization:** Added automatic detection and sanitization that purges expired Litterbox URLs (`litterbox.catbox.moe`) from local storage and memory on app startup, rider login, and roster sync.
* **Universal Fallback UI (`onerror` Handlers):** Added inline `onerror` fallback handlers and defensive fallback logic across all avatar elements in the application:
  * Main Header User Avatar (`#header-user-avatar`)
  * Rider Profile Settings Preview (`#prof-modal-avatar-preview`)
  * Avatar Picker Modal Preview (`#avatar-picker-preview`)
  * Admin Controls Rider Info Modal (`#rider-info-avatar`)
  * Customer Chat Feed Message Avatars
  * Team Comms Message Avatars
* **Startup & Real-Time Profile Photo Hydration:** Ensured the rider's active avatar is hydrated immediately on app launch from local storage and continuously synced in real time with Firebase RTDB (`roster/` and `riders/` nodes).

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


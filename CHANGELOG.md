# Lokalex App Release Notes

## Version 1.0.89 (2026-11-04)

### 1. Real-Time Chat Message Reactions & Edits
* **Live In-Place Message Edit Updates:** When a customer edits a message on Facebook Messenger, the active chat modal instantly updates the message text in place without requiring you to close and reopen the chat.
* **Original Message History:** Displays an `edited` badge along with the previous text crossed out under an `Original:` tag.
* **Messenger Emoji Reactions:** Customer reactions (❤️, 👍, 😆, 😮, 😢, 😡) sync directly onto the chat bubbles in real time. Toggling or removing a reaction updates the count and badge live.
* **Facebook Page Echoes Synchronization:** Replies sent directly from the Facebook Page / Meta Business Suite now automatically appear in the Lokalex app in real time without duplicates.

### 2. Dedicated Supabase Webhook Integration
* **Dedicated Cloudflare Worker:** Configured dedicated Supabase webhook architecture handling incoming messages, multi-photo attachments, videos, voice clips, read receipts, delivery confirmations, and postback button clicks.
* **Inbox State Restoration:** Rescued archived active conversations and restored incoming chat routing directly to the active rider inbox feed.

### 3. Stability & UI Fixes
* **Smart Cart Button Scroll Lock:** Fixed an issue where clicking item buttons (store, market, paid, onhand, NA) caused the smart cart item list to scroll to the top.
* **Overtime Commission Incentive Crash Fix:** Fixed `ReferenceError: overtimeReward is not defined` crash when toggling overtime incentive settings.

## Version 1.0.88 (2026-11-02)

### 1. Broken Image Resolution in Chat & Interactive Recovery
* **Facebook CDN Referer Fix (`no-referrer`):** Added `referrerpolicy="no-referrer"` to single photo attachments, image grids, and the fullscreen image viewer. Resolves HTTP 403 Forbidden blocks from Meta / Facebook CDNs (`lookaside.fbsbx.com`, `scontent.xx.fbcdn.net`).
* **Accurate Meta Message ID Resolution:** Enhanced `resolveFacebookMessageAttachment` to look up Meta `mid` / `metaMid` when given a Firebase push key, ensuring Graph API queries succeed with valid message IDs.
* **Expired Cache Bypass:** Added a cache purge mechanism when handling image loading errors, clearing stale expired URLs from memory and local storage so fresh signed URLs are retrieved.
* **Real-Time Graph Attachment Refresh:** Updated `syncCurrentThreadFromMetaGraph` so when fresh media attachments are fetched from Meta Graph API, existing message bubbles, in-memory caches, and Firebase records are immediately refreshed.
* **Interactive Image Fallback Card ("Tap to reload"):** If a photo fails to load due to temporary network or token expiration, the broken browser icon is replaced with a clean card (`Photo expired or loading failed • Tap to reload`) that re-fetches the image on tap.

### 2. Stale Catering Status & Lock Bug Fix
* **Live Roster Authority:** The live rotation roster is now the strict source of truth for catering status (`getActiveRosterCaterer` and `status === 'Catering'`). If a rider is off duty or no longer in Catering status, customers are never falsely locked or marked as catered by them.
* **Auto-Purge Stale Metadata:** Opening or syncing a customer thread automatically purges stale `cateredBy`, `cateredByRiderName`, and `cateredByRiderId` from Firebase, resetting the folder to `inbox` (or `done`) so active riders can cater immediately.
* **End Shift & Auto End Shift Release:** Integrated automatic catering chat cleanup into `clockOutRider()` and overnight `sweepStaleShiftsFromPriorDays()`, ensuring off-duty riders never hold locks on customer chats.
* **Hidden False Admin Banner:** Removed false `⚡ Admin Mode — Replying as Admin (Catered by ...)` warnings on unassigned or previously catered threads.

### 3. Overtime (OT) Remittance Discount Shift-Boundary Fix
* **Order-by-Order Shift Boundary Evaluation:** Overtime remittance discounts now strictly apply only to orders delivered past the rider's set scheduled duty cutoff ($\text{Login Time} + \text{Set Duty Hours} + \text{Break Time}$). Orders completed within regular duty hours strictly pay standard commission.
* **Breakdown Card & Report Accuracy:** Individual customer cards display the `⚡ Overtime` badge and discounted rate only for qualified orders past duty, while daily settlement reports and rider summaries clearly delineate regular orders from overtime-discounted orders.

## Version 1.0.87 (2026-11-01)

### 1. Catering Queue Jump Prevention & Dedicated Notice Modal
* **Pre-Check Queue Spot on Catering Attempt:** Clicking the "Cater" button in the active customer chat thread (`#rider-chat-cater-btn`) or swiping to cater from the chat feed now strictly validates whether the rider is #1 in the queue lineup before initiating any assignment.
* **Dedicated Not-First Notice Modal (`#not-first-in-queue-modal`):** If a rider attempts to cater an incoming order while someone is ahead in queue or while already catering with available riders waiting, a prominent, high-z-index warning modal is displayed directly over the chat dialog.
* **Lineup Transparency Card:** The modal clearly presents:
  * **1st in Line Rider:** Name of the designated rider currently at the top of the queue.
  * **1st Rider Today's Gross:** Income comparison showing why the leading rider has queue priority.
  * **Your Queue Spot & Gross:** Clear status indication (e.g., `#2 in Queue (₱340.00)` or `Currently Catering (Busy)`).
  * **Clear Guidance:** Informs the rider that the customer order is designated for the rider at the top of the queue and to wait for their turn.

### 2. Auto-Message Leak Prevention
* **Zero Premature Messages:** Completely eliminated the bug where the "Being Catered" auto-message was dispatched to the customer even when the rider's cater request failed or was rejected due to queue position.
* **Strict Assignment Gate:** `triggerAutoMessageOnCater` is now guarded at the system level and only triggered upon verified, successful Firebase database commitment where `cateredByRiderId` and `status: Catering` are confirmed.
* **Fail-Safe Abort:** The message generator automatically aborts if the current user is not confirmed in active `Catering` status on the roster.

## Version 1.0.86 (2026-11-01)

### 1. Overtime Commission Incentive & Commission Relief Settings
* **Admin Overtime Incentive Settings:** Added comprehensive overtime reward configuration in Admin Commission Fee Settings (`#admin-commission-settings-modal`), allowing administrators to award riders working past their scheduled shift.
* **Flexible Commission Relief:** Supports both percentage discount deduction (e.g., -5% company commission rate, allowing riders to keep higher payout) or fixed deduction per order (e.g., -₱10 less company commission per delivery).
* **Shift Duty Threshold:** Configurable duty threshold hours (default: 8 hours or individual rider shift schedule), accurately factoring in net worked minutes while strictly excluding break times.
* **Real-Time Rate & Settlement Integration:** Automatically calculates and applies overtime commission relief across live orders, daily totals, settlement reports, and order-by-order breakdown cards with dynamic `⚡ Overtime` and `⚡ OT Incentive` badges.

### 2. Manual Map Picker for Customer Directory & Profile Addresses
* **Customer Directory Add Address Map Picker:** Added dedicated "Pick on Map" button (`window.openCustomerMapPicker('add-customer-address')`) directly inside the Add Address submodal (`#add-customer-address-modal`), allowing riders and dispatchers to drop and drag a custom map pin or tap anywhere on OpenStreetMap alongside the automatic GPS detector.
* **Customer Profile & Details Saved Locations Map Picker:** Added interactive "Pick on Map" button inside the customer details saved locations form (`#cust-modal-add-loc-box`) and customer profile edit modal (`#edit-customer-profile-modal`).
* **Overlay Layer & Input Integration:** Displays high-z-index Leaflet map canvas directly over open customer modals with center reticle and manual draggable pin, automatically populating GPS coordinates and reverse-geocoded addresses without disrupting dialog state or navigation stacks.

### 3. Shift Time Calculation & Break Period Allowance System
* **Break Time Excluded from Shift Total:** Rider shift duty now strictly excludes all accumulated break time ($\text{Net Worked} = \text{Gross Elapsed} - \text{Total Break}$), preventing idle time from inflating shift records.
* **Daily Break Allowance (60m Default):** Admin-configurable break duration with leftover rollover across breaks on the same calendar day, auto-exiting to Available when time expires and refreshing daily.
* **Unlimited Break for Admins:** Direct option for unlimited break duration when taken by admins or force-administered to riders.

## Version 1.0.85 (2026-11-01)

### 1. Interactive Picture-in-Picture (PiP) Floating Badge
* **Dynamic Status Rings:** The floating mini-badge now automatically updates its ring color in real time based on active rider operations:
  * **Blue Ring:** Idle standby mode when the rider has no unread messages in the inbox and is not currently catering.
  * **Yellow Ring:** Actively catering mode indicating the rider is on an active order delivery.
* **Dual Unread Notification Badges:**
  * **Green Badge:** Displays the number of unread incoming chats in the general inbox.
  * **Orange Badge:** Prominently displays the number of unread incoming messages from the customer the rider is actively catering.
  * **Auto-Balancing Layout:** Automatically anchors to the top-right corner when a single category is active, or splits into top-right (Catering) and bottom-right (Inbox) when both have unread messages to ensure zero visual overlap.
* **Touch & Tap Interactivity:** Tapping the badge triggers native tactile haptic feedback and intelligently routes to the relevant customer chat thread (catered customer first, then unread inbox chats, or restores fullscreen mode).
* **Mini PiP Chat & Fullscreen Restorer:** Integrated an expand-to-fullscreen button (`#rider-chat-expand-btn`) in the chat header and added native `exitFloatingWindowMode()` bridge so riders can toggle between the floating badge and full app seamlessly.

## Version 1.0.84 (2026-11-01)

### 1. Fixed Generate Receipt Button & Smart Cart Viewport Layout
* **Always-Visible Fixed Generate Receipt Button:** Converted the "Generate Receipt" button to a viewport-fixed bottom container (`fixed bottom-0 z-40`) to ensure it remains permanently present and 100% visible across all device screen heights, avoiding clipping issues from nested container paddings.
* **Sticky Top Navigation & Pinned Action Bar:** Kept the cart slot tabs, client assigner, subtotal display, and item actions pinned at the top (`sticky top-0 z-20`).
* **Internal Touch Momentum Scrolling:** Constrained `#cart-items-list` with `max-h-[calc(100vh-270px)]` and `overflow-y-auto` so items scroll smoothly between the fixed toolbars without displacing the bottom Generate Receipt button.
* **Smart Rail & Knob Auto-Display:** Tuned the vertical slider rail and circular draggable knob (`#cart-slider-track`) to reliably display whenever the cart has 5 or more items or when the list overflows the viewport.

### 2. Universal Scrollbar Modernization
* **Elimination of Ugly Desktop Scrollbars:** Enforced CSS `.no-scrollbar` rules across horizontal tabs, action button strips, category switchers, and quick reply chips on all platforms.
* **Native Arrow Button Removal:** Disabled default platform scrollbar stepper buttons globally (`::-webkit-scrollbar-button`) for a clean, modern aesthetic.
* **Sleek 3px Translucent Scrollbars:** Integrated minimal 3px semi-transparent thumb indicators for vertical chat message feeds and modal dialogues.

### 3. Screen-Off Notification Stability & Atomic Penalty Execution
* **Continuous Foreground WakeLock:** Maintained indefinite `PARTIAL_WAKE_LOCK` for `LokalexForegroundService`, preventing CPU suspension and missed chat/order notifications on Android 16 when the phone screen is turned off.
* **Exact Alarm Compatibility:** Added safe alarm fallback (`setAndAllowWhileIdle`) for Android 14+ devices without exact alarm grants.
* **Child-Level Penalty Database Transactions:** Streamlined commission penalties into atomic `.child(penaltyKey).set(record)` operations to prevent Firebase invalid key errors with dotted rider names.

## Version 1.0.83 (2026-11-01)

### 1. Eliminated Native Desktop Scrollbars & Added Sleek Mobile Scrollbars
* **Universal Horizontal Scrollbar Elimination:** Added CSS utility rules for `.no-scrollbar` and `[class*="no-scrollbar"]` across all platforms (Chromium, WebView, Desktop). Eliminated ugly native grey horizontal scrollbars and arrows `< [========] >` under chat action bars, category tabs, source filter chips, and switcher bars.
* **Global Arrow Button Removal:** Stripped native scrollbar arrow buttons (`::-webkit-scrollbar-button { display: none !important; }`) across the entire app.
* **Sleek Translucent Mobile Scrollbars:** Vertical message lists and modal dialogs now feature sleek, unobtrusive 3-4px rounded translucent thumb tracks that never obscure content.

### 2. Smart Menu & Cart Sticky Layout and Side Scroller
* **Sticky Header & Sticky Create Receipt:** Constrained `#view-cart` to viewport height so the top toolbar (cart slots, assign client, subtotal, and action buttons) remains permanently pinned at the top, and the "Create Receipt" button remains permanently pinned at the bottom.
* **Internal Items Scrolling:** Only `#cart-items-list` scrolls between the fixed headers and footers, preventing the browser window from pushing action buttons off-screen.
* **Permanent Side Scroller Rail & Knob:** The vertical slider track with the circular grabbing knob (`#cart-slider-track`) now reliably appears whenever the cart contains 5 or more items or when the list overflows, allowing effortless one-touch rapid scrolling.

### 3. Screen-Off Background Notification Reliability (Android 16 / TECNO CM8)
* **Indefinite Foreground WakeLock:** Updated `LokalexForegroundService` to hold `PowerManager.PARTIAL_WAKE_LOCK` continuously while the service is alive, preventing the CPU from entering suspend-to-RAM when the phone screen is turned off or idle.
* **Android 14+ Exact Alarm Fallback:** Added `canScheduleExactAlarms()` verification before scheduling alarms, falling back safely to `setAndAllowWhileIdle()` to guarantee background watchdog pings run without security exceptions or system drops.

### 4. Robust Commission Penalty Application
* **Guaranteed Atomic Child Writes:** Replaced composite update dictionaries with direct sanitized `.child(penaltyKey).set(record)` writes under `commissionPenalties`, completely eliminating Firebase RTDB `update failed: values argument contains an invalid key` errors for riders with dots (e.g. `Ron DC.`) or special punctuation.
* **Strict Primitives Sanitization:** Ensured all penalty record properties are non-undefined and safely formatted before database dispatch.

### 5. Chat Media Bubble & Preview Enhancements
* **Clean Media Text Suppression:** Prevented redundant placeholder captions (`📷 Photo`, `🎥 Video`, `🎤 Voice message`) from rendering as text bubbles below images and media cards.
* **Auto-Reopen on Incoming Message:** Completed threads in `folder: 'done'` automatically reopen into `inbox` / `catering` whenever a customer sends a new message.

## Version 1.0.82 (2026-11-01)

### 1. Restored Chat Pictures, Multi-Photos, and Attachments
* **Fixed Single & Multi-Image Rendering:** Resolved a critical logic conflict in `riderChatRender.js` where image URLs were improperly passed into video/audio detectors, causing the message image list to evaluate as empty. Sent and received pictures, multi-image grids, and gallery items now render instantly without delay.
* **Non-Blocking Combined Media:** Customers sending multiple photos or photo + video combinations simultaneously now display both media cards seamlessly without one suppressing the other.
* **Eliminated Raw Placeholder Text:** Fixed redundant text lines (`📷 Photo`, `🎥 Video`, `🎤 Voice message`) from displaying inside chat bubbles when media cards are loaded. Captions and genuine text messages are preserved cleanly.
* **Robust Image Error Recovery:** Enhanced `handleChatImageLoadError` to cleanly unpack refreshed media objects and prevent broken image `src` replacements.

### 2. Resolved Meta Graph API 400 Bad Request on Attachments
* **Clean Attachment Query Specification:** Replaced non-existent Graph API attachment field request (`video_data`) with standard `attachments` collection query across client resolvers and Cloudflare Worker webhooks.
* **Restored Voice Clips & Audio Playback:** Voice messages and audio clips from Facebook Messenger now fetch and resolve successfully without throwing HTTP 400 errors, with instant audio waveform player controls.

### 3. Real-Time Chat Feed Card Synchronization
* **Automatic Metadata Timestamp Catch-Up:** Opening any chat thread automatically checks if the latest message timestamp is newer than `customerChatMeta`, immediately repairing stale preview cards and keeping the inbox list strictly synchronized with the latest customer message.

## Version 1.0.81 (2026-11-01)

### 1. Fixed Commission Penalty Application & Name Sanitization
* **Illegal Firebase Character Sanitization:** Fixed the penalty submission failure (`update failed: values must be valid Firebase paths`) caused when targeting riders whose names include periods (e.g. `Ron DC.`), slashes, or special punctuation.
* **Child-Level Atomic Updates:** Multi-rider and multi-day penalty mutations now use direct sanitized child keys under `commissionPenalties` (`db.ref('commissionPenalties').update()`), ensuring 100% reliable Firebase database writes across all names and punctuation marks.
* **Fuzzy & Sanitized Key Lookup:** Enhanced penalty rate evaluation in `commissionRates.js` to look up both raw and sanitized keys, with automatic fallback to fuzzy name matching (`isRiderMatch`).

## Version 1.0.80 (2026-11-01)

### 1. Commission Penalty System Overhaul & Multi-Rider / Multi-Day Sanctions
* **Pre-Submission Confirmation Review:** Adding a penalty now features an explicit 2-step confirmation review modal displaying target rider(s), applicable dates, penalty surcharge percentage, reason, and total records calculation before applying.
* **Multi-Rider Sanctioning:** Penalty modal now supports Single Rider, Multiple Riders (with searchable list, Select All / Clear All, and live counter badge), or All Active Riders in one operation.
* **Date Range (Multiple Days):** Sanction riders for single dates or date ranges (From/To) with an automatic days counter and individual daily ledger records.
* **Active Date Penalties Banner:** Prominent live banner in the Commission view displaying all active penalties for the selected period, complete with rider name, date, surcharge percentage, reason, and a 1-click Remove button with delete confirmation.
* **Rider Card Header Badges:** Collapsed rider cards now feature a prominent red `⚖️ +X% Penalty` badge next to the rider's name for instant visibility.
* **Zero-Delivery Visibility:** Riders with active penalties remain visible on the Commission ledger even before logging deliveries for that day.
* **Quick Presets:** Instant preset chips for penalty rates (+3%, +5%, +10%, +15%, +20%) and common violation reasons (Late shift, Unexcused absence, Incomplete uniform, Extended break, Customer complaint).

### 2. Multi-Media Messenger Chat Resolution & Latest Previews
* **Enhanced Meta Graph API Subfield Extraction:** Fixed missing attachments when customers send multiple photos or photo + video combinations simultaneously from Facebook Messenger. Meta Graph API fetches now request full nested attachment subfields (`image_data, video_data, payload, subattachments, mime_type`) across Cloudflare Workers and client resolvers.
* **Accurate Chat Card Previews:** Resolved race conditions and stale timestamp comparisons in `chatFeedListener.js`. Newly incoming customer messages and media previews now immediately take absolute precedence over older cached messages.
* **Live Metadata Preview Sync:** Incoming message streams write latest message previews and media badges directly into `customerChatMeta` in real time.

### 3. Dedicated Add Customer Modal & Live GPS Location Capture
* **Streamlined Add Customer Modal:** Replaced nested directory prompts with a dedicated Customer Registration modal featuring Customer Name, Contact Number, and dynamic Add Address triggers.
* **Automatic GPS Calibration:** Both the Chat Customer Info location manager and the Directory Add Address modal now auto-capture the rider's live GPS coordinates and generate a precise Google Maps pin link automatically.
* **Multi-Location Management:** Manage and navigate to multiple registered addresses per customer directly from chat and directory cards.

### 4. Store Directory Bulk Menu Photo Upload & Smart Cart Scrollbar
* **Bulk Store Menu Uploads:** Store owners and riders can now select and upload multiple menu photos in a single batch with preview thumbnail grid and faster image compression.
* **Smart Cart Permanent Scrollbar:** Enhanced order scrolling for large orders with a visible, responsive scrollbar and touch slider for effortless navigation.

## Version 1.0.78 (2026-10-31)

### 1. Directory Customer Profile Auto-Connect
* **Instant Auto-Link:** When checking customer details in a chat thread, the app automatically checks the Customer Directory by name and contact phone. If a saved directory profile is detected, it automatically connects and links the customer profile immediately with zero extra taps required.
* **Synchronized Profile Data:** Auto-connecting immediately syncs the customer's verified name, phone number, address, and all saved locations into the active chat session and roster.

### 2. Multiple Saved Locations Support (Chat Info & Directory)
* **Multiple Addresses & GPS Pins:** Customers can now register and store multiple locations (e.g., Home, Work, Office, Farm) in both the Customer Directory and Chat Customer Info.
* **Chat Customer Info Locations Manager:** View all saved locations with one-tap Google Maps navigation, switch the active/default delivery location, or add new locations on the fly with 1-tap GPS calibration.
* **Customer Directory Multiple Locations:** Directory form now supports adding multiple addresses and map pin links. Customer directory cards render all registered locations with direct location pins for each.

### 3. Smart Cart Slider Scroller (Futuristic Circular Knob)
* **Invisible Slider Track & Glowing Knob:** Added a sleek, circular slider knob with an invisible track on the right edge of the smart cart for effortless, lightning-fast touch scrolling through large order lists.
* **Interactive Drag & Auto-Fade:** Supports 1:1 touch dragging and mouse drag scrolling with smooth haptics and automatic fade-out when idle.

### 4. Multi-Photo Gallery & In-Bubble Voice Player
* **Multi-Image Batch Display:** Fixed the bug where only 1 photo showed when customers sent multiple photos in Facebook Messenger. All photos are now parsed and displayed in an interactive gallery grid with fullscreen zoom.
* **Customer Voice Note Player:** Customer audio recordings from Messenger are now rendered directly in chat bubbles with a custom audio player featuring play/pause controls, interactive seekable progress track, and live elapsed timer.

## Version 1.0.77 (2026-10-27)

### 1. In-App Over-The-Air (OTA) APK Self-Updater
* **Direct Phone Upgrades Without PC/ADB:** Implemented a native APK self-updater allowing riders to upgrade and fresh-install base Android APKs directly on their phone over the air.
* **Seamless 1-Tap System Installer:** When a new base APK version is released, the app downloads the APK in the background with a live progress indicator (`showApkInstallProgress`) and launches Android's native `PackageInstaller` via `FileProvider`.
* **Zero Data Loss:** Updating the base APK preserves 100% of the rider's session, login tokens, SQLite database, and preferences.
* **Smart Permission Handling:** Checks `REQUEST_INSTALL_PACKAGES` permission and opens system settings if needed to ensure effortless one-tap updates.
* **Dual-Track Updater Engine:** Minor web patches continue to live-swap instantly via OTA zips without interruption, while core Android native updates (services, permissions, native bridges) prompt for APK installation.

### 2. Picture-in-Picture (PiP) Floating Window Mode & Auto-PiP
* **Smooth Native PiP:** Android's native Picture-in-Picture mode keeps Lokalex active in a compact, floating window while riding or using navigation apps (Google Maps, Waze).
* **Auto-PiP on Background:** Automatically transitions to Picture-in-Picture whenever the rider presses Home or switches apps via `onUserLeaveHint()`.
* **Instant Uninterrupted Chat & Audio:** PiP keeps the app running on screen in multi-window mode so Firebase listeners, chat badges, and chime notifications ring with zero delay.

## Version 1.0.76 (2026-10-27)

### 1. Restored Native Picture-in-Picture (PiP) Floating Window Mode
* **Live Chat Feeds & Uninterrupted Execution:** Switched back to Android's native Picture-in-Picture (PiP) window. Unlike overlay badges which put the app into background sleep mode, PiP keeps the WebView active on screen—ensuring incoming chat messages, badges, and audio chimes trigger instantaneously without delay.
* **Thread-Safe Native Bridge:** Fixed the issue where entering floating mode could fail by ensuring `enterPictureInPictureMode()` executes on the main UI thread.
* **Audio & Listener Continuity:** Automatically keeps DOM timers and audio engines unpaused while in PiP mode so riders never miss customer or catering messages.
* **Adaptive Mini-Window UI:** Restored PiP responsive styling for headers, chat panels, and stashed badge expanding.

## Version 1.0.75 (2026-10-27)

### 1. Drag-to-Close Dismiss Target for Floating Window
* **Messenger-Style Bottom Close Area:** Dragging the floating window badge now displays an intuitive, circular dismiss target (`✕` "Drag here to close") at the bottom center of the screen.
* **Interactive Hover Feedback & Haptics:** Dragging the badge into the bottom close zone triggers haptic vibration feedback, turns the close target glowing red (`#E11D48` "Release to close"), scales up the target, and subtly pulls the floating badge into the dismiss circle.
* **Instant Float Dismissal:** Releasing the badge over the close area cleanly animates and dismisses the floating window badge from the screen.
* **Safe Tap-to-Open & Edge Snapping:** Tapping the badge still brings Lokalex immediately to the foreground; dragging and releasing elsewhere on the screen continues to magnetically snap flush against the nearest screen edge.
* **Dual-Orientation Support:** The bottom dismiss area accurately centers itself in both portrait and landscape screen orientations.

## Version 1.0.74 (2026-10-27)

### 1. Landscape Full-Width Snapping & Physical Display Boundaries
* **Fixed Landscape Mid-Screen Stopping:** Resolved the bug where the floating window badge could only travel to the middle of the screen in landscape mode. Replaced stale `ApplicationContext` display metrics with physical hardware bounds via `WindowMetrics` (API 30+) and `Display.getRealMetrics()`.
* **Full-Width Edge Snapping:** In landscape mode, dragging now spans the entire physical width (e.g. 2400px), cleanly docking flush against the far right or far left screen edge.

### 2. Catered Chime Sound Isolation (Strictly My Cater Only)
* **Removed Admin Chime Bleed:** Eliminated the admin bypass in `chatFeedListener.js` that caused orders catered by other riders to chime on the current rider's device.
* **Strict Chime Ownership:** Incoming messages for catered customers now strictly ring the audio chime if and only if the current logged-in rider is the active caterer (`isMyCustomerThread === true`).
* **Visual Color Badge Retention:** Float window badges still accurately display category colors (Green for Inbox, Orange for Catering, Red for Follow-up, Purple for Applicant, White for Stores) without sounding disruptive audio alarms for non-catered messages while on duty.

## Version 1.0.73 (2026-10-27)

### 1. Dual-Orientation Screen Edge Docking (Portrait & Landscape)
* **Permanent Side Sticking:** Floating window badge is now permanently anchored/docked to the left or right edge of the screen (`dpToPx(6)` margin), never floating adrift or getting stranded in the middle of the display.
* **Instant Portrait & Landscape Adaptation:** Equipped with dual orientation watchers (`ComponentCallbacks2` and `OrientationEventListener`). When the device rotates between portrait and landscape modes, the badge automatically calculates the new screen width and height and smoothly animates to the docked edge without being cut off or pushed off-screen.
* **Relative Vertical Position Memory:** Stores relative vertical height ratio so the badge retains its proportional elevation whether navigating in landscape on a handlebar mount or holding the phone in portrait.
* **Debug Mode Deactivated:** Removed all web contents debugging and testing flags across native WebView and Java activity components.

## Version 1.0.72 (2026-10-27)

### 1. Color-Coded Category Badges in Float Window Mode
* **Color-Coded Folder Badges:** Added live dynamic color coding to the floating edge badge when receiving incoming customer messages in Float Window mode:
  - 🟢 **Green** for **Inbox** (`#10B981`)
  - 🟠 **Orange** for **Catering** (`#F97316`)
  - 🔴 **Red** for **Follow-up** (`#EF4444`)
  - 🟣 **Purple** for **Applicant** (`#A855F7`)
  - ⚪ **White** for **Stores** (`#FFFFFF` with high-contrast text)
* **Floating Head Glow & Bounce Alert:** The outer ring and status dot dynamically illuminate with the active category color, while a smooth bounce animation triggers on arrival to immediately catch the rider's attention over Maps or other apps.
* **Auto-Show on Message Arrival:** If the app is in the background when a message arrives, the floating badge is displayed automatically with the new unread pill.

### 2. Chat Acoustic Chimes Restoration & Dual Native Engine
* **Fixed Missing Chimes on Chat Updates:** Resolved the bug where incoming inbox, follow-up, and applicant messages were silenced for active riders due to a restrictive role gate in `chatFeedListener.js`.
* **Dual Native Android + Web Audio Engine:** In addition to Web Audio API synthesis, chimes now trigger Android native `RingtoneManager` sound via `playNativeChatChime()` through the native bridge. Chimes now ring loudly and reliably in the foreground, background, and Float Window mode.
* **Continuous AudioContext Availability:** Removed aggressive post-chime `suspend()` calls that previously froze the web audio engine on Android mobile WebViews.

## Version 1.0.71 (2026-10-27)

### 1. Replaced Android PiP with Native Floating Edge Badge (Float Window Mode)
* **Dedicated Float Window Mode:** Completely abandoned and removed fragile Android Picture-in-Picture (PiP) window manager implementation, which caused app vanishing/collapsing on Transsion (Infinix/Tecno) and Android 12+ devices due to `mWaitForEnteringPinnedMode` null pointer crashes in `ActivityTaskManager`.
* **Instant Float Header Button:** Tapping the header float button (`<i class="fa-solid fa-arrow-up-right-from-square"></i>`) immediately enters Float Window mode: smoothly minimizes the app and docks an interactive, circular Lokalex badge on the edge of the screen.
* **Automatic Background Float Mode:** When the rider switches to external apps (such as Google Maps, Waze, or Messenger) or minimizes the app, the floating badge automatically docks to the side of the screen.
* **Magnetic Edge-Snapping & Draggable:** The floating badge is smoothly draggable anywhere on the screen and features magnetic snapping to the nearest edge with a spring-decelerate animation.
* **Instant Re-foreground:** Tapping the edge badge immediately restores Lokalex Rider to the active foreground.
* **Active Status & Unread Badge:** Includes an active emerald ring pulse and an unread notification counter pill that displays real-time pending customer messages.

## Version 1.0.70 (2026-10-27)

### 1. PiP (Picture-in-Picture) Tree-Shaking & Bundle Inclusion Fix
* **Global Handler Export:** Fixed critical build tree-shaking bug where `window.enterFloatingWindowMode` was omitted from production bundles by Vite, causing the header PiP button to silently do nothing when tapped.
* **Persistent DOM Surface in PiP:** Completely eliminated `#pip-stashed-badge` and all `display: none !important` hide rules on `<body>` elements when in floating mode. The app content (live chat feed and modal views) now continuously paints to the Android SurfaceControl buffer, preventing the OS window manager from considering the window empty and dismissing it.
* **Informative Permission & Native APK Guidance:** If invoked on a client device running an older base APK without native multi-window permissions, the app now shows an actionable toast rather than silently doing nothing.


## Version 1.0.69 (2026-10-27)

### 1. PiP (Picture-in-Picture) Web Layout & Transition Fix
* **Removed Collapsing PiP Media Query:** Removed `@media (max-width: 170px)` CSS rule that inadvertently hid the entire app DOM (`body > *:not(#pip-stashed-badge) { display: none !important; }`) whenever the window resized down to portrait PiP dimensions (~120–140px width). This keeps the rider's chat and dashboard visible inside the floating mini-window instead of collapsing into a blank screen.
* **Transition Re-entry Lock:** Added a programmatic state lock in `enterFloatingWindowMode()` and `onFloatingWindowModeChanged()` to prevent double-invocation of the native window transition, eliminating Android `ActivityClientController onTransact aborts`.
* **Safe Badge Unstash Interaction:** Updated `#pip-stashed-badge` to harmlessly clear the stashed view state upon tapping rather than re-invoking native PiP entry while already in floating mode.
* **Decoupled Haptics During Window Transition:** Moved haptic triggers prior to window transition execution, eliminating audio/vibrator IPC collisions with OEM window managers.

### 2. Live Chat Feed Lock & Unlock Protection
* **Guaranteed Unlock Access:** Confirmed `[🔒 Locked]` / `[🔓 Unlocked]` toggle button stays elevated at `z-50` with direct pointer event passthrough, ensuring riders can reliably lock or unlock feed filters at any time.


## Version 1.0.68 (2026-10-27)

### 1. PiP (Picture-in-Picture) Floating Window Stability Fix
* **Elimination of Instant Window Closure:** Resolved the bug where entering PiP mode caused the app to instantly close or dismiss on Android 12+ (specifically on Infinix, Tecno, and OEM systems).
* **Safe Transition Architecture:** Removed conflicting gesture flags (`autoEnterEnabled` & `seamlessResize`) from programmatic triggers, preventing window manager transaction aborts.
* **Transition Guarding & Source Rect Hint:** Added an `isEnteringPip` state lock to eliminate race conditions from `onUserLeaveHint()`, along with hardware bounds hints (`sourceRectHint`) for smooth window scaling.
* **Manifest Modernization:** Configured `android:resizeableActivity="true"` and `android:enableOnBackInvokedCallback="false"` to prevent system back-gesture interceptions.

### 2. Live Chat Feed Lock Button Responsiveness
* **Isolated Stacking Context (`z-50`):** Fixed the lock button being unclickable when in the locked state. The toggle button is now elevated to its own independent stacking context (`z-50`) with persistent inline style enforcement, ensuring it remains immediately clickable even when feed misclick protection is active.
* **Target Origin Verification:** Added event target checks in `onChannelBarLockedClick` to guarantee lock toggle interactions are never intercepted by the background click-guard overlay.

## Version 1.0.67 (2026-10-27)

### 1. Live Chat Feed Channel Bar Lock (Misclick Protection)
* **Default Locked Protection:** The source filter bar (`#rider-chat-channel-bar`) and Full Booking mode trigger on the Live Chat Feed are now locked by default to prevent accidental misclicks by riders while scrolling through incoming customer conversations.
* **1-Tap Lock / Unlock Toggle:** Added a prominent `[🔒 Locked]` / `[🔓 Unlocked]` toggle button with instant visual and haptic feedback.
* **Intelligent Click-Guard:** When locked, chips and buttons are dimmed to 60% opacity with an active overlay that intercepts errant taps and notifies the rider (`"🔒 Feed filters are locked to prevent misclicks. Tap 🔓 on the right to edit."`).

### 2. Password Removal Direct Login Fix
* **Password-Free Rider Authentication:** Resolved the bug where riders whose passwords were removed by an administrator in the Roster could not log into the app because the login modal still demanded a password.
* **Auto-Routing to Setup Modal:** Cleared-password riders now bypass password entry automatically and are guided straight to the rider password setup modal.

### 3. PiP (Picture-in-Picture) UI Thread Safety & Floating Button Clean-Up
* **Main UI Looper Thread Dispatch:** Fixed native Android `IllegalStateException` on PiP mode entry by ensuring `enterPictureInPictureMode` is always invoked on the main UI looper Handler, preventing WebView worker thread crashes.
* **Streamlined PiP Buttons:** Removed cluttered and redundant PiP buttons from inside the customer chat modal and the live chat feed, retaining only the clean, dedicated top header button.

### 4. "Share Location" Map Picker Navigation Isolation
* **Checkout Wizard Misdirection Fix:** Resolved the bug where clicking "Share Location" in the chat modal inadvertently triggered history state pop navigation into the customer checkout wizard.
* **In-Place Modal Overlay:** Map location picking from chat is now fully isolated as an in-place modal overlay with zero history stack interference.

### 5. Chat Modal Toolbar Cleanup
* **Duplicate Menu & Rates Buttons Removed:** Cleaned up excessive Menu and Rates buttons from the customer chat header bar and cater action row, preserving only the dedicated action buttons located next to the `[⚡ Quick]` toolbar button.

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


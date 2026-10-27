# SDE-1 Android Interview Master Dossier & Tri-Company Strategy Report
**Candidate:** Bhavuk Rajput | **Experience:** 3+ Years | **Current:** RapiPay Fintech (NYE Android App)  
**Target Roles:** Software Engineer / SDE-1 (Android Developer)  
**Target Companies & Upcoming Schedules:**  
1. **Entertainment Network India Limited (ENIL / Radio Mirchi / Times Group)** — Sector 16A Film City, Noida  
2. **Smart Energy Water (SEW.AI)** — NSL TechZone, Sector 144, Noida — **F2F Interview: 28th October 2026 at 2:30 PM IST**  
3. **OLX India** — Gurugram / Noida (Source of the official *SE1 - Android Interview Preparation Guide*)

---

## Executive Summary & Official Evaluation Framework

This dossier synthesizes ground-truth data from the candidates' email invitations, official attachments, the **Times Group / ENIL Job Description (`Android.docx`)**, and the **OLX India SDE-1 Interview Preparation Guide**.

### The Core Evaluation Principles (From the OLX India SE1 Guide)
1. **Strength of Core Android & Kotlin/Java Fundamentals:** Mental models over memorized syntax.
2. **Ability to Read, Analyze, and Improve Code:** Diagnosing edge cases, race conditions, memory leaks, and lifecycle anti-patterns.
3. **Structured Problem-Solving:** Thinking out loud, asking clarifying questions, breaking complex problems into modular steps.
4. **Runtime Awareness:** Threading models, memory allocations, garbage collection pauses, and Android OS lifecycle boundaries.

---

## 🏢 Tri-Company Technical Dossiers & Strategic Alignment

### 1. Entertainment Network India Limited (ENIL / Times Group / Radio Mirchi)
- **Venue:** Plot No 6, FC - 6, Maharaja Agrasen Marg, Film City, Sector 16A, Noida, UP 201301
- **Domain:** Mass Media, Digital Audio & Video Streaming, Live FM 98.3, Mirchi Plus, Podcasts, OTT Platform
- **Key JD Requirements (Directly from `Android.docx`):**
  - **Video Streaming, Picture-in-Picture (PiP), Pay-Per-Use billing, and OTT media applications.**
  - **Inter-Process Communication (IPC)**, App Modularization, and Third-Party SDK integration.
  - **Android App Vitals** (Crash rate, ANR threshold, slow rendering frames, frozen frames) & Android Profiling Tools (Memory Profiler, Network Inspector, Systrace/Perfetto).
  - High screen-density scaling, backward compatibility (> Android 5.0), and modern Android Architecture Components.

#### Top ENIL / Times Group Technical Questions
1. **How do you implement Picture-in-Picture (PiP) mode in an Android media streaming application?**  
   *Model Answer:*  
   - Declare `android:supportsPictureInPicture="true"` and `android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation"` in the Activity manifest to prevent activity destruction when entering PiP.
   - When the user taps Home or an in-app minimize button, call `enterPictureInPictureMode(PictureInPictureParams.Builder().setAspectRatio(aspectRatio).setActions(remoteActions).build())`.
   - Override `onPictureInPictureModeChanged(isInPictureInPictureMode: Boolean)`: if `true`, hide non-essential UI (playback controls, chat, recommendation lists) and keep only the video surface visible. When exiting PiP, restore the controls.
   - Manage `RemoteAction` intents to allow users to play/pause or skip directly from the OS PiP window.

2. **What are the primary Inter-Process Communication (IPC) mechanisms in Android, and when do you choose which?**  
   *Model Answer:*  
   - **AIDL (Android Interface Definition Language):** Best for multi-threaded IPC across different applications where client-server RPC is required with custom Parcelable data.
   - **Messenger:** Simpler IPC built on top of AIDL and `Handler`. Serializes requests on a single thread (Queue of `Message` objects), avoiding thread-synchronization headaches.
   - **BroadcastReceiver:** For one-to-many asynchronous event distribution (e.g. system battery warnings, network connectivity changes).
   - **ContentProvider:** Best for structured data sharing (e.g. sharing media files, user contacts, or database records) across processes with URI-based permissions.

3. **How do you monitor and optimize Android App Vitals (ANRs, Slow Rendering, Frozen Frames)?**  
   *Model Answer:*  
   - **ANR (Application Not Responding):** Occurs when the main thread is blocked for $>5\text{ seconds}$ (or $>10\text{s}$ in a BroadcastReceiver). Prevent by running all disk I/O, database queries, and cryptographic operations on `Dispatchers.IO`. Diagnose using StrictMode and Google Play Console vitals reports.
   - **Slow Rendering & Frozen Frames:** A frame is slow if rendering exceeds $16.6\text{ms}$ ($60\text{fps}$) or $8.3\text{ms}$ ($120\text{fps}$); frozen if $>700\text{ms}$. Optimize layout hierarchies, avoid nested weights in XML, eliminate unnecessary recomposition in Jetpack Compose by deferring state reads, and inspect render passes using **Android GPU Rendering / Systrace**.

---

### 2. Smart Energy Water (SEW.AI)
- **Interview Schedule:** **28th October 2026 (Wednesday) at 2:30 PM IST (F2F)**
- **Venue:** SEW.AI Office, NSL TechZone, Sector 144 / Sector 142, Noida – 1st Floor (Nr. Sector 143 Metro)
- **Domain:** AI-powered Vertical Cloud Platforms for Energy, Water, and Gas Utilities (Smart Customer Mobile - SCM & Smart Work Force - SMW)
- **Core Mobile Challenges:**
  - **Offline-First Data Architecture:** Field technicians operate in remote utility sites, basements, and rural zones with zero network connectivity. Work orders, meter readings, and safety forms must be saved locally and synced seamlessly.
  - **Bi-directional Synchronization & Conflict Resolution:** Single Source of Truth (SSOT), optimistic local writes, version vectors, and resolving `409 Conflict` between dispatchers and field technicians.
  - **WorkManager Under the Hood:** SQLite persistence, `JobScheduler` delegation, constraints (`NetworkType.UNMETERED`, `RequiresCharging`), and exponential backoff retry policies.
  - **Battery & Location Optimization:** Fused Location Provider with displacement filtering (`setMinUpdateDistanceMeters`), batched updates, and hardware Geofencing API for job sites.
  - **OOM Crash Prevention:** Two-pass bitmap downsampling (`inSampleSize` + `RGB_565`) for high-resolution meter damage photos and streaming uploads via OkHttp direct from disk.

#### Top SEW.AI Technical Questions
1. **How do you architect an offline-first mobile app where technicians operate in zero-connectivity environments?**  
   *Model Answer:* Follow the Single Source of Truth (SSOT) pattern where the UI exclusively observes Room database tables (via Flow). All user interactions write synchronously to Room with `syncStatus = PENDING`. A background `WorkManager` worker queries pending orders, posts them to the API when network is detected, and flips `syncStatus = SYNCED`.
2. **How does WorkManager guarantee execution across device restarts and battery saver restrictions?**  
   *Model Answer:* WorkManager persists all tasks in an internal SQLite database. On modern Android versions, it delegates to `JobScheduler`; on older APIs, it uses `AlarmManager` + `BroadcastReceiver`. It adheres to OS Doze mode constraints and executes guaranteed background work with exponential backoff on retry.
3. **How do you prevent Out-Of-Memory (OOM) crashes during photo capture and inspection uploads?**  
   *Model Answer:* Decode file dimensions first using `BitmapFactory.Options.inJustDecodeBounds = true`, compute optimal `inSampleSize` to scale down to target display dimensions, set `inPreferredConfig = Bitmap.Config.RGB_565` (halving memory footprint to 2 bytes/pixel), and stream file uploads straight from disk using OkHttp `RequestBody`.

---

### 3. OLX India (Classifieds Marketplace)
- **Context:** Recruiter Srija Rai sent the official **SE1 - Android Interview Preparation Guide (13).pdf**.
- **Domain:** Global Classifieds Marketplace (P2P buying/selling cars, electronics, real estate, services)
- **Core Mobile Engineering Challenges:**
  - **Classifieds Feed with Paging 3:** Infinite scrolling of millions of listings with multi-category facets, price filters, and local geo-radius search.
  - **Real-Time Buyer-Seller Chat:** Instant messaging using WebSockets / MQTT with local message caching in Room, push notification wakeups, and offline message queueing.
  - **Ad Posting Flow:** Multi-photo picker, camera EXIF rotation correction, image compression, auto-category classification, and address geocoding.
  - **Deep Linking & App Indexing:** Deferred deep links for product sharing on social platforms (WhatsApp, SMS) via AppsFlyer/Branch with backstack restoration.

#### Top OLX India Technical Questions
1. **System Design: How would you architect the OLX Android Application?**  
   *Model Answer:*  
   - **UI Layer:** Jetpack Compose with MVI unidirectional data flow. Classifieds listing feed implemented with `LazyVerticalGrid` powered by the **Android Paging 3** library.
   - **Data Layer:** `RemoteMediator` coordinates between the remote REST/GraphQL API and the local Room database cache. The UI observes the cached database stream, enabling instantaneous offline feed browsing.
   - **Image Pipeline:** Coil/Glide configured with custom memory and disk caches, disk downsampling, and decoding WebP images.
   - **Real-Time Chat Engine:** A persistent WebSocket connection managed inside a bound Service. Outgoing messages write to Room with `MessageStatus.SENDING`; once acknowledged by the server, status updates to `DELIVERED` or `READ`.
2. **How do you handle deep linking in Android, and what is the difference between Deep Links and Android App Links?**  
   *Model Answer:*  
   - **Standard Deep Links:** Custom URL schemes (e.g. `olx://item/12345`) or generic HTTP URLs. If multiple apps claim the same scheme, Android shows the disambiguation dialog ("Open with...").
   - **Android App Links:** Verified HTTP/HTTPS URLs (e.g. `https://www.olx.in/item/12345`). By hosting a Digital Asset Links JSON file (`assetlinks.json`) at `https://www.olx.in/.well-known/assetlinks.json` containing the app's SHA-256 fingerprint, Android verifies domain ownership and opens your app directly without prompting the user.
3. **How do you optimize an ad-posting photo upload flow where sellers upload 10+ high-res images?**  
   *Model Answer:*  
   - Read image metadata via `ExifInterface` to fix orientation bugs (e.g. portrait photos rotating 90° landscape).
   - Compress images asynchronously on `Dispatchers.Default` using WebP/JPEG compression, capping file size to $\sim 400\text{ KB}$ per image.
   - Queue uploads sequentially or with bounded concurrency (e.g. 2 parallel uploads via coroutines) to avoid starving UI thread network bandwidth.

---

## 🎯 Bhavuk Rajput's Resume Deep-Dive & STAR Model Answers

### 1. EPA Employee-Assisted Full-KYC SDK Journey (RapiPay NYE)
- **Situation:** Users and field agents in tier-2/3 cities suffered severe drop-offs when unstable network connections dropped mid-way through multi-step KYC verification, forcing them to restart.
- **Task:** Design and own the client Android SDK integration for the EPA Full-KYC journey, providing deterministic state progression, journey resumption across app restarts, and automatic validation fallbacks.
- **Action:** Implemented a unidirectional MVI state machine using Kotlin `StateFlow` and sealed interfaces (`DocumentCaptured`, `BiometricPending`, `OtpVerifying`, `FallbackRequired`). Each step saved state snapshots into `DataStore` with a unique `journeyId`. Automated fallbacks allowed manual document upload when biometric/OCR failed without wiping previously validated fields. On app restart, the SDK called the resumption endpoint, reconciled local checkpoints, and restored the exact step.
- **Result:** Reduced onboarding drop-offs, eliminated redundant user data entry, and ensured 100% RBI compliance on wallet activation.

### 2. R8 Full Mode Optimization & Lending SDK Hilt `ClassCastException`
- **Situation:** Enabling R8 Full Mode (`android.enableR8.fullMode=true`) in AGP caused a runtime crash in release builds with `java.lang.ClassCastException` in the Lending SDK module.
- **Task:** Fix the runtime crash in production release builds without disabling R8 Full Mode.
- **Action:** In R8 Full Mode, the compiler aggressively inlines single-implementation interfaces, removes unused synthetic wrappers, and strips generic signatures (`Signature`). The Lending SDK used dynamic Dagger/Hilt entry points: `EntryPoints.get(context, LendingEntryPoint::class.java)`. Because the entry point was accessed dynamically and had only synthetic generated implementations, R8 Full Mode merged or altered the interface hierarchy, causing the generated component to fail runtime casting. Diagnosed using `-whyareyoukeeping`, isolated the stripped interface, and authored precise keep rules:
  ```pro
  -keepattributes *Annotation*,Signature,InnerClasses,EnclosingMethod
  -keep,allowobfuscation interface * extends dagger.hilt.internal.GeneratedComponent
  -keep,allowobfuscation @dagger.hilt.EntryPoint interface * { *; }
  -keep class com.rapipay.lending.di.** { *; }
  ```
- **Result:** Fixed the crash permanently in release builds while preserving R8 Full Mode optimizations, reducing DEX size by **~282.9 KB** and cutting the production APK from **44.4 MB to 44.1 MB**.

### 3. Android 16 / SDK 36 Edge-to-Edge & System Insets
- **Mandatory Policy:** Android 15/16 enforces edge-to-edge drawing by default; `windowOptOutEdgeToEdgeEnforcement` is deprecated/ignored.
- **Implementation:** Used `enableEdgeToEdge()` in `onCreate()`. In Compose, consumed `WindowInsets.safeDrawing` and `WindowInsets.ime`. In XML, used `ViewCompat.setOnApplyWindowInsetsListener` and set `clipToPadding="false"` on `RecyclerView` so scrollable items glide cleanly behind the transparent navigation bar. Handled foldables via `WindowWidthSizeClass` (Compact vs Expanded layouts).

### 4. VAPT Remediation & Security Hardening
- **SSL Pinning:** SHA-256 public key hash pinning via OkHttp `CertificatePinner` with backup keys.
- **Root & Frida Detection:** Scanned `/system/xbin/su`, `/proc/self/maps` for `frida-agent.so`, TCP port 27042, and `TracerPid > 0` for debugger attachment.
- **Secure Storage:** Replaced plain SharedPreferences with `EncryptedSharedPreferences` backed by Android Keystore (AES-256 GCM).

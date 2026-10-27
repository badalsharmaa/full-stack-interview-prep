# SDE-1 Android Interview Master Dossier & Strategy Report
**Candidate:** Bhavuk Rajput | **Experience:** 3+ Years | **Current:** RapiPay Fintech (NYE Android App)  
**Target Roles:** Software Engineer / SDE-1 (Android)  
**Upcoming Interviews:**  
1. **Entertainment Network India Limited (ENIL / Radio Mirchi / Mirchi Plus)** — Noida Sector 16A Film City  
2. **Smart Energy Water (SEW.AI)** — NSL TechZone, Sector 144, Noida  

---

## Executive Summary & Preparation Strategy

This master dossier synthesizes deep technical analysis across candidate experience, target company architectures, real-world interview question matrices, and the evaluation rubric outlined in the **SE1 - Android Interview Preparation Guide**.

### The Core Evaluation Philosophy (From the SE1 Guide)
The interviewing panel evaluates:
1. **Strength of Core Android & Kotlin Fundamentals:** Deep mental models over superficial syntax.
2. **Ability to Read, Analyze, and Improve Code:** Diagnosing edge cases, race conditions, memory leaks, and lifecycle traps.
3. **Structured Problem Solving & Clear Communication:** Thinking out loud, asking clarifying questions, breaking down problem spaces into modular components.
4. **Runtime Awareness:** Threading models, memory allocations, garbage collection, and OS lifecycle boundaries.

---

## 🏢 Target Company Intelligence & Architectural Analysis

### 1. Entertainment Network India Limited (ENIL / Radio Mirchi / Mirchi Plus)
- **Domain:** Digital Audio Entertainment, Live Radio (FM 98.3), Podcasts, Original Audio Series, Music Streaming (Mirchi Plus, Gaana platform ecosystem).
- **Core Mobile Engineering Challenges:**
  - **ExoPlayer & Media3 Architecture:** Media session management, audio decoding pipelines, continuous stream buffering over unstable cellular networks.
  - **Background Playback & Android 14/15/16 Foreground Services:** Strict OS background execution limits, Doze mode evasion, persistent media notifications with playback action handlers.
  - **Audio Focus Management:** Managing transient and permanent audio focus changes (phone calls, turn-by-turn navigation, notifications) using `AudioFocusRequestCompat`.
  - **Offline Caching & DRM:** Chunked audio caching (`CacheDataSourceFactory`), LRU eviction policies, Widevine DRM license acquisition for premium audiobooks and music.
  - **UI/UX Performance:** Smooth 60fps horizontal and vertical carousel scrolling in large content catalogs using Jetpack Compose `LazyRow` / `LazyColumn` with optimized image loading (Coil/Glide).

#### Key ENIL Interview Questions
1. **How do you architect a rock-solid background audio playback service in Android 14/15/16?**  
   *What they look for:* Proper usage of `MediaSessionService` / `MediaLibraryService` from AndroidX Media3, declaring `foregroundServiceType="mediaPlayback"`, handling runtime notifications, and binding with notification managers.
2. **Explain Audio Focus handling in Android. What happens during an incoming call vs. a GPS voice prompt?**  
   *What they look for:* Requesting `AUDIOFOCUS_GAIN`, handling `AUDIOFOCUS_LOSS` (stop & abandon focus), `AUDIOFOCUS_LOSS_TRANSIENT` (pause playback), and `AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK` (lower playback volume).
3. **How do you implement offline audio caching using ExoPlayer / Media3?**  
   *What they look for:* Configuring `SimpleCache` with `LeastRecentlyUsedCacheEvictor`, wrapping upstream `HttpDataSource` with `CacheDataSource`, and handling encrypted audio storage.
4. **How do you optimize image and media feed scrolling performance in Jetpack Compose?**  
   *What they look for:* Stable composable keys (`key = { song.id }`), `@Immutable` data models, preventing recomposition using `derivedStateOf`, memory cache sizing in image loaders.

---

### 2. Smart Energy Water (SEW.AI)
- **Domain:** AI-powered Cloud Platforms for Global Energy, Water, and Gas Utilities. Core mobile products:
  - **Smart Customer Mobile (SCM):** Customer self-service, bill payments, smart meter consumption analytics, outage tracking.
  - **Smart Work Force (SMW):** Enterprise field mobility for utility technicians (meter inspection, leak detection, work order dispatch, asset maintenance).
- **Core Mobile Engineering Challenges:**
  - **Offline-First Data Architecture:** Field technicians work in remote utility sites, basements, and rural zones with zero connectivity. The app must allow full work order creation, data entry, and inspection sign-offs locally in Room, syncing whenever connectivity returns.
  - **Bi-directional Synchronization & Conflict Resolution:** Handling optimistic local writes, conflict resolution strategies (Client Wins vs. Server Wins vs. Timestamp/Version vector merge), and queueing pending transactions in `WorkManager`.
  - **Background Location & Geofencing:** Tracking technician dispatch, geofenced job sites, battery-efficient polling with Google Fused Location Provider and Geofencing API.
  - **Heavy Dynamic Form Rendering:** Handling large multi-step inspection forms with dynamic schema validations, high-res photo capture, and signature pads without memory bloat.
  - **Enterprise Mobile Security:** Encrypted local databases (SQLCipher/Encrypted Room), biometric authentication (`BiometricPrompt`), secure token refresh via OkHttp Authenticator.

#### Key SEW.AI Interview Questions
1. **How do you architect an offline-first mobile app where technicians operate in zero-connectivity environments?**  
   *What they look for:* Single Source of Truth (SSOT) pattern where UI observes Room DB (via Flow), user interactions write exclusively to local DB with `SyncStatus = PENDING`, and background workers sync changes with the backend.
2. **How does WorkManager guarantee execution across device restarts and battery saver restrictions?**  
   *What they look for:* Persistent job storage in SQLite internal database, JobScheduler / AlarmManager fallbacks, `Constraints` (NetworkType.CONNECTED, RequiresCharging), and `ExistingWorkPolicy.APPEND_OR_REPLACE`.
3. **How do you prevent Out-Of-Memory (OOM) errors when handling camera capture and document uploads in field inspection forms?**  
   *What they look for:* Downsampling bitmaps using `BitmapFactory.Options.inSampleSize`, reading image EXIF rotation, compressing to WebP/JPEG, streaming file uploads via OkHttp `RequestBody` instead of reading entire files into memory.
4. **How do you secure local client databases and user credentials in an enterprise utility app?**  
   *What they look for:* Android Keystore system, AES-256 GCM encrypted keys, `EncryptedSharedPreferences`, Room with SQLCipher database encryption.

---

## 🎯 Bhavuk Rajput's Resume Deep-Dive & STAR Model Answers

### 1. EPA Employee-Assisted Full-KYC SDK Journey (RapiPay NYE)
- **The Challenge:** NYE wallet full KYC required an agent-assisted journey with Aadhaar OTP, biometric verification, Customer Application Form (CAF) data validation, and fallback mechanisms. High drop-off occurred when connections broke mid-journey.
- **STAR Framing:**
  - **S (Situation):** RapiPay needed a unified, secure Full-KYC SDK journey within the NYE consumer app that field agents and users could execute reliably across varying network conditions.
  - **T (Task):** Architect and own the Android SDK integration for the EPA Full-KYC flow, ensuring state persistence across multi-stage forms, automated fallback mechanisms, and seamless journey resumption.
  - **A (Action):** Built a unidirectional data flow (MVI) architecture using Kotlin Coroutines and StateFlow. Decoupled the KYC flow into discrete stage states represented by sealed interfaces. Implemented local journey checkpoints in DataStore with cryptographic session tokens. When a validation failure or network drop occurred during biometric/CAF verification, the engine gracefully routed the user to verified fallback paths (e.g., manual document upload + manual OTP verification). On relaunch, the SDK queried the journey resumption endpoint, reconciled local checkpoints, and restored the exact step without re-prompting previously validated inputs.
  - **R (Result):** Drastically reduced onboarding drop-offs, improved completion rates, and eliminated duplicate verification requests.

### 2. R8 Full Mode Optimization & Lending SDK Hilt `ClassCastException`
- **The Challenge:** Upgraded AGP and enabled R8 Full Mode to cut app size. Release builds crashed with `ClassCastException` in the Lending SDK module.
- **STAR Framing:**
  - **S (Situation):** In optimizing the NYE production app, we migrated to modern AGP and turned on `android.enableR8.fullMode=true` to achieve aggressive tree shaking and DEX optimization.
  - **T (Task):** Resolve a critical release-only runtime crash: `ClassCastException` occurring inside the Lending SDK's dependency injection layer, without turning off R8 Full Mode.
  - **A (Action):** Analyzed the stack trace and R8 `mapping.txt` and `usage.txt`. In R8 Full Mode, the compiler aggressively optimizes interfaces that have single implementers (interface merging), inlines synthetic classes, and strips generic signatures (`Signature` attribute) and Kotlin metadata. The Lending SDK utilized dynamic Dagger/Hilt entry points (`EntryPoints.get(context, LendingEntryPoint::class.java)`). Because the entry point interface was referenced dynamically and had only synthetic generated implementations, R8 merged or altered the interface hierarchy, causing the generated component to fail runtime casting.
    I diagnosed the issue using `-whyareyoukeeping`, isolated the stripped interface, and authored precise ProGuard rules:
    ```pro
    -keepattributes *Annotation*,Signature,InnerClasses,EnclosingMethod
    -keep,allowobfuscation interface * extends dagger.hilt.internal.GeneratedComponent
    -keep,allowobfuscation @dagger.hilt.EntryPoint interface * { *; }
    -keep class com.rapipay.lending.di.** { *; }
    ```
  - **R (Result):** Fixed the crash permanently in release builds while preserving R8 Full Mode optimizations, reducing DEX size by ~282.9 KB and cutting the production APK from 44.4 MB to 44.1 MB.

### 3. Android 16 / SDK 36 Edge-to-Edge & System Insets
- **Core Explanation:** Android 15/16 makes edge-to-edge rendering mandatory. Apps cannot opt out with `windowOptOutEdgeToEdgeEnforcement`.
- **Implementation:**
  - Called `enableEdgeToEdge()` in Activity `onCreate()`.
  - In Jetpack Compose: Consumed insets using `Modifier.windowInsetsPadding(WindowInsets.safeDrawing)` and `WindowInsets.ime`.
  - In XML views: Applied `ViewCompat.setOnApplyWindowInsetsListener` to consume system bar and display cutout insets, updating padding dynamically and setting `clipToPadding="false"` on `RecyclerView`.
  - Addressed foldable and large-screen layouts using `WindowWidthSizeClass` (Compact, Medium, Expanded) to provide responsive two-pane interfaces.

### 4. VAPT Remediation & Application Hardening
- **Root Detection:** Multi-tiered inspection checking for binaries (`/system/bin/su`, `/system/xbin/su`), dangerous properties (`ro.debuggable == 1`), test-keys build tags, and Google Play Integrity API attestation.
- **Anti-Hooking & Frida Detection:** Inspecting `/proc/self/maps` for injected libraries (`frida-agent.so`), checking TCP port 27042, and scanning memory maps for Frida hook signatures.
- **SSL Pinning:** Implemented SHA-256 public key hash pinning via OkHttp `CertificatePinner` with backup keys to prevent MITM proxy interception.
- **Debugger Detection:** Reading `/proc/self/status` for `TracerPid != 0` and monitoring `android.os.Debug.isDebuggerConnected()`.

### 5. Accessibility (WCAG 2.2 AA) & TalkBack Optimization
- **Compose Semantics:** Leveraged `Modifier.semantics(mergeDescendants = true)` so complex items (e.g., wallet balance, transaction time, and status tag) are read as one continuous announcement.
- **Touch Target Sizing:** Guaranteed $48 \times 48\text{ dp}$ minimum touch targets via `minimumInteractiveComponentSize()`.
- **Traversal Ordering:** Controlled accessibility focus order using `Modifier.semantics { traversalIndex = ... }`.

---

## 🧠 Master Technical Question Matrix

### Module 1: Kotlin Core & Concurrency
1. **Explain the difference between `launch` and `async`. How do they handle unhandled exceptions?**
   - `launch` returns a `Job` and represents "fire-and-forget". Exceptions thrown inside `launch` are treated as unhandled and immediately propagate up the job hierarchy to the parent, potentially crashing the app unless handled by a `CoroutineExceptionHandler`.
   - `async` returns a `Deferred<T>` and represents a computation that returns a value. Exceptions thrown inside `async` are captured inside the `Deferred` and are only thrown when `await()` is invoked. However, if using structured concurrency inside a `coroutineScope`, an unhandled failure in an `async` child will still cancel the parent scope immediately unless wrapped in a `supervisorScope`.

2. **What is the difference between `Job` and `SupervisorJob`?**
   - With a standard `Job`, failure of any child cancels the parent and all sibling coroutines.
   - With a `SupervisorJob`, failure of a child does not propagate upward to cancel the supervisor or its other children. Each child must handle its own exceptions. (Crucial inside `viewModelScope`, which uses a `SupervisorJob`).

3. **`StateFlow` vs `SharedFlow` vs `LiveData`?**
   - `LiveData` is lifecycle-aware and tied to the Android UI thread. It only dispatches on the main thread and has no built-in coroutine operators.
   - `SharedFlow` is a hot stream that can emit values to multiple subscribers. It has configurable replay cache and buffer overflow strategies (`BufferOverflow.DROP_OLDEST`, etc.). Ideal for events (navigation, toast, analytics).
   - `StateFlow` is a specialized `SharedFlow` with `replay = 1` that always holds a current value. It conflates values: emitting the same value consecutively will not trigger downstream collectors.

4. **What are `inline`, `noinline`, and `crossinline` functions in Kotlin?**
   - `inline`: Tells the compiler to copy the function body and lambda bytecode directly into the call site, eliminating object allocation overhead for function objects.
   - `noinline`: Used when an inlined function accepts multiple lambdas, but you want to prevent inlining for a specific lambda (e.g., to store it in a variable or pass it to another non-inlined function).
   - `crossinline`: Inlined lambdas normally allow non-local returns (`return` exiting the enclosing named function). `crossinline` disallows non-local returns from the lambda, ensuring the lambda cannot exit the calling context when executed inside an execution boundary (such as a runnable or nested lambda).

5. **What is `reified` in Kotlin? Why can it only be used with inline functions?**
   - Due to Java type erasure, generic type parameters `T` are erased at runtime (`List<String>` becomes `List`).
   - Declaring an inline function with `<reified T>` allows the compiler to inline the actual concrete class type into the call site bytecode, making expressions like `T::class.java` or `value is T` valid at runtime. It can only be used with inline functions because the compiler requires inlining to substitute the actual type.

---

### Module 2: Jetpack Compose & UI Architecture
1. **Explain the 3 phases of Jetpack Compose.**
   - **Composition:** What to show. Runs composable functions and builds the UI tree (SlotTable).
   - **Layout:** Where to place it. Measures each node and determines coordinates (Measure & Layout pass).
   - **Draw:** How it renders. Draws UI nodes onto the canvas (Draw pass).
   - *Optimization rule:* Defer state reads to the latest possible phase (e.g., passing a lambda `Modifier.offset { IntOffset(x.value, 0) }` reads state during layout/draw rather than composition, preventing unnecessary recomposition).

2. **What causes recomposition, and how does Compose determine if a parameter is Stable?**
   - Recomposition is triggered when state read during composition changes.
   - Compose classifies types as **Immutable** (all public primitives/val, cannot change), **Stable** (notifies Compose when public properties mutate), or **Unstable** (classes with `var`, or collections like `List` from standard library whose mutability cannot be guaranteed by the compiler).
   - Passing an unstable parameter causes the composable to be treated as non-skippable, triggering recomposition even if its data didn't change. Fixed using `@Immutable`, `@Stable`, or Kotlinx Immutable Collections.

3. **`remember` vs `rememberSaveable`?**
   - `remember`: Preserves value across recompositions as long as the composable remains in the composition tree. Lost on configuration changes (screen rotation) and process death.
   - `rememberSaveable`: Preserves value across recompositions AND configuration changes, as well as OS process death, by saving the value into the Android `SavedState` bundle.

4. **Explain Compose Side Effects: `LaunchedEffect`, `rememberCoroutineScope`, `DisposableEffect`, and `SideEffect`.**
   - `LaunchedEffect(key)`: Launches a coroutine when entering composition; restarts if `key` changes; cancels on leave.
   - `rememberCoroutineScope()`: Bound to composition lifecycle; provides a CoroutineScope to launch coroutines in response to user callbacks (button click).
   - `DisposableEffect(key)`: For effects requiring cleanup (registering/unregistering listeners, observers); must provide an `onDispose { }` block.
   - `SideEffect`: Runs after every successful recomposition to publish Compose state to non-Compose objects.

---

### Module 3: System Architecture, Memory & Lifecycle
1. **How does a ViewModel survive configuration changes under the hood?**
   - The Activity lifecycle is managed by `ComponentActivity`. During configuration changes (rotation), the system calls `onRetainNonConfigurationInstance()`.
   - The activity preserves its internal `ViewModelStore` in `NonConfigurationInstances`. When the new Activity instance is instantiated, `ComponentActivity.onCreate()` retrieves the retained `ViewModelStore`.
   - ViewModels are only cleared (`onCleared()`) when the Activity finishes permanently (`isFinishing == true`).

2. **What is Process Death? How does it differ from configuration changes?**
   - Configuration change (rotation) retains the process memory space, so ViewModels in memory survive.
   - Process Death occurs when the app is in the background and Android kills the entire application process to reclaim RAM for foreground apps.
   - Memory is completely wiped; ViewModels are destroyed. When the user returns, Android recreates the Activity stack and restores the `savedInstanceState` Bundle.
   - Solution: Use `SavedStateHandle` injected into ViewModels to persist minimal IDs/state keys.

3. **How do you diagnose and resolve Memory Leaks in Android?**
   - A memory leak occurs when an unused object is kept from being garbage-collected because an active GC root maintains a reference to it.
   - Common leaks: Static context/view references, non-static inner classes (Handler/AsyncTask/Thread holding Activity outer reference), listeners registered with singletons without unregistering, coroutines running on non-lifecycle scopes.
   - Tools: LeakCanary (automatically detects retained Activity/Fragment references and generates a leak trace path), Android Studio Memory Profiler (heap dump inspection, finding dominant instances).

---

### Module 4: Practical Coding & Problem Solving Scenarios
1. **Debounced Search Implementation using Kotlin Flow:**
   ```kotlin
   class SearchViewModel(private val repository: SearchRepository) : ViewModel() {
       private val _searchQuery = MutableStateFlow("")
       val searchQuery = _searchQuery.asStateFlow()

       @OptIn(FlowPreview::class, ExperimentalCoroutinesApi::class)
       val searchResults: StateFlow<List<SearchResult>> = _searchQuery
           .debounce(300L) // Wait 300ms after user stops typing
           .filter { it.trim().length >= 2 } // Ignore trivial queries
           .distinctUntilChanged() // Ignore if same text
           .flatMapLatest { query ->
               repository.search(query).catch { emit(emptyList()) }
           }
           .stateIn(
               scope = viewModelScope,
               started = SharingStarted.WhileSubscribed(5000),
               initialValue = emptyList()
           )

       fun onQueryChanged(newQuery: String) {
           _searchQuery.value = newQuery
       }
   }
   ```

2. **Offline-First Synchronization Worker:**
   ```kotlin
   class SyncWorkOrderWorker(
       context: Context,
       workerParams: WorkerParameters,
       private val localDao: WorkOrderDao,
       private val apiService: UtilityApiService
   ) : CoroutineWorker(context, workerParams) {

       override suspend fun doWork(): Result = withContext(Dispatchers.IO) {
           try {
               val pendingOrders = localDao.getPendingSyncOrders()
               for (order in pendingOrders) {
                   val response = apiService.syncOrder(order.toDto())
                   if (response.isSuccessful) {
                       localDao.markSynced(order.id, response.body()?.updatedAt)
                   }
               }
               Result.success()
           } catch (e: Exception) {
               if (runAttemptCount < 3) Result.retry() else Result.failure()
           }
       }
   }
   ```

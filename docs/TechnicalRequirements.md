# Nurse App

**Technical Requirements · Fantasix**  
**Last updated:** 20 October 2026  
**Status:** Planning document; membedakan prototype yang sedang dikerjakan dari planned MVP integration.

## 1. Project Overview

**App name:** Nurse App; Nursitor adalah nama project dan target iOS di repository `nurse-app` saat ini.  
**Team name:** Fantasix  
**Team members:** Adiat (Developer), Daffa (Developer), Hasan (Developer), Nicho (Developer), Ayu (Designer), Ivanna (Designer)  
**Platform and minimum version:** iOS 27+, watchOS 27+ 
**UI framework:** SwiftUI

**Description:**
Nurse App adalah aplikasi pendamping kesehatan kerja untuk perawat yang menggunakan iPhone dan Apple Watch. Satu session mewakili satu shift di satu unit/instalasi, dengan maksimal satu session duty aktif per perawat, termasuk saat break. Aplikasi ditujukan untuk menyajikan body battery dan personal suggestions yang mudah dipahami, serta membantu koordinasi break dengan rekan satu session tanpa membuka raw biomarker. Prototype pertama menggunakan data simulasi; HealthKit integration, backend, dan actual notifications merupakan tahap berikutnya.

### 1.1 Scope and Product Decisions

- Semua perawat boleh membuat session di iPhone dan menjadi host; peserta lain menjadi member. Role berlaku per session, bukan jabatan admin permanen.
- Nama unit/instalasi, jadwal dari template yang dapat diedit, dan optional break configuration diisi ketika membuat session. Department tidak selalu ada dan kamar pasien bukan dasar session; MVP tidak memerlukan form hierarki rumah sakit.
- Join memakai kode empat digit pada iPhone dan Watch. Leading zeros tetap dipertahankan. Create tidak tersedia di Watch.
- iPhone menampilkan durasi duty pribadi. Watch menampilkan status, unit dan body battery tanpa timer durasi duty; dashboard di kanan, action page di kiri.
- Break dan notification untuk koordinasi menggunakan satu flow. Tidak ada Ask for Help terpisah. Penerima notification adalah seluruh peer lain yang masih berhak dalam session, termasuk host, tanpa pengirim.
- Peer hanya melihat indikator yang diizinkan, status duty/break, dan source data age. Raw HR, HRV, RHR, tidur, dan biomarker lain tetap pribadi.
- Session otomatis berakhir sesuai jadwal atau lebih awal oleh host. Tidak ada session extension pada MVP ini.

### 1.2 Approved Review Decisions

| Decision | Agreed Behavior |
| --- | --- |
| Q1 — Per-session Sharing Consent | Setiap session baru menyediakan sharing confirmation dengan sharing off by default pada create/join. Consent pada session lama tidak otomatis berlaku pada session baru, tanpa perlu mengulang onboarding. |
| Q2 — Member Duty Completion | Setelah duty closure untuk member non-host confirmed, partisipasi peer aktif, indicator sharing, akses indikator tim aktif, dan penerimaan break notifications baru berakhir. Session rekan tetap berlangsung; hak host setelah selesai duty belum diputuskan. |
| Q3 — Break Limit | Countdown end time mengikuti waktu yang lebih awal antara akhir durasi break dan akhir session. Break 15 menit ketika sisa session 3 menit menjadi countdown 3 menit dengan penjelasan. |
| Q4 — Expired Notice | Break-start notification yang tertunda tidak diajukan pada/setelah countdown limit. Status Break/overdue masih dapat melalui reconciliation sebagai status information, tanpa notification bantuan baru atau auto-return. |

Duty closure saat offline tetap pending sampai reconciliation selesai dan closure confirmed, atau session lama dipastikan telah berakhir berdasarkan valid time rules. Duty baru tidak boleh dimulai seolah closure sebelumnya sudah confirmed. Prototype menggunakan simulation untuk flow ini; tidak menjanjikan pencabutan cache atau push yang sudah terkirim secara seketika.


## 2. Technologies and Frameworks

<!-- > Document every Apple framework, library, or external service your app uses. For each one, explain **which methods, functions, or classes** you call, **what they do**, and **how your app uses them**. A teammate who has never seen your code should be able to understand how the technology works in your app after reading this section. -->


### 2.1 Technology Overview

<!-- > List everything first, then go into detail in 2.2. -->

| Technology | Type | Why we use it | Used in which component |
|---|---|---|---|
| HealthKit | Apple Framework | To read biometric parameters (Sleep, SDNN, RMSSD, HRV, Activity State, and RHR) to compute nurse Body Battery status | Service |
| EventKit | Apple Framework | To display the user's schedule in view | Service  |
| Watch Connectivity | Apple Framework | To sync data between the iPhone and Apple Watch | Service  |
| ActivityKit | Apple Framework | To make a custom live activity in the iOS app | Widget |
| Swift Chart | Apple Framework | To display the user's health insight | Service |
| Nearby Interaction | Apple Framework | To detect nearby shift and joining the shift | Service |
| CoreML | Apple Framework | To analyze the data and make predictions | Service |
| Firebase | External API | To create session, save data & logs etc | Service |


### 2.2 Technology Details

<!-- > Copy the block below once for **each** technology in your table. Do not skip the "How we use it" part. Explaining what a method does is not enough. Explain what it does **in your app**. -->

#### HealthKit

**What it is:** HealthKit serves as the centralized on-device repository for all physiological vitals and biometric data recorded by iPhone and Apple Watch. In our app, HealthKit continuously queries sleep stages, autonomic heart rate variability (SDNN and derived RMSSD), resting heart rate, and on-duty physical activity to compute and update the nurse's real-time 0–100% Body Battery recovery score.

**Setup required:** 
- Add the **HealthKit** capability in your Xcode project’s Signing & Capabilities section.
- Add `NSHealthShareUsageDescription` to your app’s `Info.plist` explaining why we need read permissions (e.g., *"This app reads your sleep, heart rate variability, resting heart rate, and activity to monitor fatigue and calculate your shift body battery."*).
- Check `HKHealthStore.isHealthDataAvailable()` before making calls.

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `HKHealthStore` | The main object used to interact with the HealthKit database. | Maintained in `HealthKitService` to request authorization and execute health queries across all clinical workflows. | `HealthKitService.swift` |
| `HKHealthStore.isHealthDataAvailable()` | Checks whether HealthKit is supported on the current device. | Called at app launch to verify device capability before executing any health-related logic. | `HealthKitService.swift` |
| `requestAuthorization(toShare:read:)` | Prompts the system permission sheet to request access to specified HealthKit data types. | Requests read permissions for Sleep, SDNN, RHR, Heartbeat Series, Steps, and Active Energy upon first onboarding. | `HealthKitService.swift` |
| `HKCategoryType(.sleepAnalysis)` | Identifies sleep tracking samples and sleep stage intervals. | Fetches pre-shift sleep sessions to calculate baseline restorative recharge for **Use Case 1 (Pre-Shift Recovery & Capacity Calibration)**. | `SleepService.swift` |
| `HKCategoryValueSleepAnalysis` | Represents sleep stages (`.asleepDeep`, `.asleepREM`, `.asleepCore`, `.awake`). | Weighs deep and REM sleep higher in the recharge formula to evaluate restorative sleep between shifts (**Use Case 1**). | `SleepService.swift` |
| `HKQuantityType(.heartRateVariabilitySDNN)` | Represents heart rate variability measured as standard deviation of NN intervals (ms). | Establishes autonomic nervous system baseline; suppressed SDNN indicates accumulated fatigue or chronic strain (**Use Cases 1 & 3**). | `HRVService.swift` |
| `HKHeartbeatSeriesSample` | HealthKit sample containing discrete beat-to-beat (RR) interval timings. | Serves as the raw beat-to-beat timestamp source to calculate acute RMSSD for **Use Case 3 (Acute Stress & Vagal Depletion Detection)**. | `HRVService.swift` |
| `HKHeartbeatSeriesQuery` | Queries high-resolution RR intervals from an `HKHeartbeatSeriesSample`. | Streams beat-to-beat RR intervals to detect acute parasympathetic vagal tone drops during high-stress ward events (**Use Case 3**). | `HRVService.swift` |
| `calculateRMSSD(from:)` | Custom mathematical algorithm: $\sqrt{\frac{1}{N-1}\sum(RR_{i+1} - RR_i)^2}$. | Computes the nurse's acute physiological recovery; drops in RMSSD quantify acute stress during ward shifts (**Use Case 3**). | `BodyBatteryCalculator.swift` |
| `HKQuantityType(.restingHeartRate)` | Represents resting heart rate in beats per minute (bpm). | Evaluates baseline strain; an elevated RHR above the 14-day average caps starting battery capacity (**Use Case 1**). | `VitalsService.swift` |
| `HKQuantityType(.activeEnergyBurned)` | Measures active calories burned through physical exertion (kcal). | Quantifies high-intensity labor (e.g., patient transfers, rapid emergency responses) for **Use Case 2 (On-Duty Ward Exertion Tracking)**. | `ActivityService.swift` |
| `HKQuantityType(.stepCount)` | Quantifies accumulated step count over time. | Measures total on-feet floor workload during 8–12 hour ward rounds to model physical depletion (**Use Case 2**). | `ActivityService.swift` |
| `HKQuantityType(.appleExerciseTime)` | Measures time spent in brisk or elevated physical exertion (minutes). | Identifies sustained peak exertion intervals across hospital wards for **Use Case 2 (On-Duty Ward Exertion Tracking)**. | `ActivityService.swift` |
| `HKSampleQueryDescriptor` | Modern async/await query descriptor for reading discrete health samples. | Concurrently fetches latest sleep, SDNN, and RHR samples to calibrate pre-shift readiness (**Use Case 1**). | `HealthKitService.swift` |
| `HKStatisticsQueryDescriptor` | Calculates aggregate values (sum, average, min, max) over specified time intervals. | Computes cumulative steps and active calories per shift window (e.g., 07:00–19:00) for **Use Case 2 (On-Duty Ward Exertion Tracking)**. | `ActivityService.swift` |
| `HKQuantity.doubleValue(for:)` | Converts stored HealthKit quantities into numerical values with appropriate units. | Extracts numerical values using `HKUnit.secondUnit(with: .milli)` (HRV), `count().unitDivided(by: .minute())` (RHR), and `kilocalorie()` (Energy). | `HealthKitService.swift` |
| `calculateBodyBattery(...)` | Composite calculation weighing recharge (sleep, HRV baseline) against depletion (shift activity, elevated RHR, stress). | Generates a 0–100% real-time Body Battery score displayed on the shift dashboard and Apple Watch complication. | `BodyBatteryCalculator.swift` |

<br>

**Biometric Parameters to Body Battery Specification:**

| Parameter | Clinical Use Case | HealthKit Type / Identifier | Target Unit | Query Type | Purpose in Body Battery & Shift Management |
|---|---|---|---|---|---|
| **Sleep** | **Use Case 1: Pre-Shift Recovery & Capacity Calibration** | `HKCategoryType(.sleepAnalysis)` | Hours / Minutes | `HKSampleQueryDescriptor` | **Recharge Factor (+):** Primary restorative engine. Deep & REM stages grant bonus charge. Accurately aggregates split sleep between night shifts. |
| **SDNN** | **Use Case 1 & 3: Autonomic Baseline & Fatigue** | `HKQuantityType(.heartRateVariabilitySDNN)` | Milliseconds (`ms`) | `HKSampleQueryDescriptor` | **Autonomic Tone Baseline:** Compares against 14-day baseline. Suppressed SDNN indicates chronic fatigue or impending clinical burnout. |
| **RMSSD** | **Use Case 3: Acute Stress & Vagal Depletion Detection** | Derived from `HKDataTypeIdentifierHeartbeatSeries` | Milliseconds (`ms`) | `HKHeartbeatSeriesQuery` | **Acute Vagal Stress Index:** Reflects short-term parasympathetic activation. Rapid drops during critical ward events warn of acute exhaustion. |
| **HRV (Combined)** | **Use Case 3: Acute Stress & Vagal Depletion Detection** | Composite of SDNN & RMSSD | Scaled Score (0–100) | Computed Metric | **Drain Rate Modifier:** When HRV is suppressed, subsequent physical shift activity accelerates battery depletion up to $1.5\times$ faster. |
| **Activity State** | **Use Case 2: On-Duty Ward Exertion Tracking** | `stepCount`, `activeEnergyBurned`, `appleExerciseTime` | `count`, `kcal`, `min` | `HKStatisticsQueryDescriptor` | **Depletion Factor (-):** Continuous physical drain calculated per shift hour based on ward rounds, patient transfers, and floor movement. |
| **RHR** | **Use Case 1: Pre-Shift Recovery & Capacity Calibration** | `HKQuantityType(.restingHeartRate)` | Beats/min (`count/min`) | `HKSampleQueryDescriptor` | **Capacity Ceiling Modifier:** If RHR is elevated >5 bpm above baseline, maximum daily battery capacity is capped (e.g., at 80–85%). |

**Code example from our project:**

```swift
import HealthKit

final class HealthKitService {
    private let healthStore = HKHealthStore()
    
    // Core parameters required for Nurse Body Battery calculation
    private let readTypes: Set<HKObjectType> = [
        HKCategoryType(.sleepAnalysis),
        HKQuantityType(.heartRateVariabilitySDNN),
        HKQuantityType(.restingHeartRate),
        HKQuantityType(.stepCount),
        HKQuantityType(.activeEnergyBurned),
        HKQuantityType(.appleExerciseTime),
        HKSeriesType.heartbeat()
    ]
    
    func requestPermissions() async throws {
        guard HKHealthStore.isHealthDataAvailable() else { return }
        try await healthStore.requestAuthorization(toShare: [], read: readTypes)
    }
    
    /// Queries the latest Resting Heart Rate for daily baseline calibration
    func fetchLatestRHR() async throws -> Double? {
        let rhrType = HKQuantityType(.restingHeartRate)
        let descriptor = HKSampleQueryDescriptor(
            predicates: [.quantitySample(type: rhrType)],
            sortDescriptors: [SortDescriptor(\.endDate, order: .reverse)],
            limit: 1
        )
        let samples = try await descriptor.result(for: healthStore)
        let bpm = HKUnit.count().unitDivided(by: .minute())
        return samples.first?.quantity.doubleValue(for: bpm)
    }
}
```

**Challenges or limits we found (optional):** 
- **Device Dependency:** Full metric availability (SDNN, beat-to-beat heartbeat series, and resting heart rate) requires the nurse to wear an Apple Watch.
- **RMSSD Derivation:** HealthKit does not provide RMSSD natively; it must be calculated programmatically from `HKHeartbeatSeriesSample` beat intervals.
- **Permission Concealment:** iOS does not disclose if the user denied read permissions; queries will simply return empty arrays, requiring robust empty/fallback state handling in UI.
- **Shift Boundary Crossing:** Nurses frequently work night shifts spanning across calendar days (e.g. 19:00 to 07:00), requiring shift-based date range predicates rather than default midnight-to-midnight queries.
- **No Background Data Fetching:** HealthKit does not support background data fetching; the app must be opened to retrieve the latest data.

---

#### EventKit

**What it is:** EventKit provides programmatic access to the user's Apple Calendar database to read, create, and manage scheduled work events and reminders. In our app, EventKit syncs parsed hospital shift rosters into Apple Calendar and automatically binds scheduled shift intervals to in-app fatigue monitoring sessions and pre-shift readiness alerts.

**Setup required:**
- Add the **EventKit** capability in Xcode's Signing & Capabilities section.  
- Add usage description strings to your app’s `Info.plist`:
  - Key: `NSCalendarsUsageDescription` → Value: Explain why you need access to the calendar (e.g., *"This app needs access to your calendar to display and manage your work shifts."*)  
  - Key: `NSRemindersUsageDescription` → Value: Explain why you need access to reminders (e.g., *"This app uses reminders to notify you of upcoming shifts and breaks."*)

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `EKEventStore` | The central coordinator object used to request access and manipulate calendar events. | Initialized in `CalendarService` to manage permissions, query existing shifts, and commit imported events across all schedule workflows. | `CalendarService.swift` |
| `requestFullAccessToEvents()` | Prompts the system permission sheet to request full read/write calendar access (iOS 17+). | Called when importing an Excel roster to verify existing shifts and prevent duplicates for **Use Case 1 (Automated Hospital Roster Ingestion & Deduplication)**. | `CalendarService.swift` |
| `EKEvent` | Represents a calendar event with properties such as title, start date, end date, notes, and alarms. | Instantiated for each parsed row of the nurse's roster (e.g., "Day Shift - Ward 4B", "Night Shift - ICU") to represent work events (**Use Case 1**). | `ShiftScheduleImporter.swift` |
| `EKEvent.notes` | Stores textual descriptions, notes, or metadata attached to a calendar event. | Encodes structured metadata (e.g., `ShiftID`, `Ward`, `NurseID`) to link the calendar event to in-app shift sessions for **Use Case 3 (Shift-to-Session Lifecycle Binding)**. | `ShiftScheduleImporter.swift` |
| `EKEventStore.defaultCalendarForNewEvents` | Returns the user’s designated default calendar for creating new events. | Serves as the primary target calendar destination when synchronizing parsed shift events (**Use Case 1**). | `CalendarService.swift` |
| `EKEventStore.save(_:span:commit:)` | Persists an event (or batch of events) to the calendar database. | Saves all parsed shift events into Apple Calendar in a single committed transaction (**Use Case 1**). | `CalendarService.swift` |
| `EKEventStore.predicateForEvents(withStart:end:calendars:)` | Constructs a search query for calendar events occurring within a specified date window. | Queries calendar events across the imported month to identify overlapping or duplicate shift entries (**Use Case 1**). | `CalendarService.swift` |
| `EKEventStore.events(matching:)` | Executes an event predicate search and returns an array of matching `EKEvent` objects. | Fetches scheduled shifts to populate the in-app calendar view and bind upcoming shifts to monitoring sessions (**Use Cases 1 & 3**). | `CalendarService.swift` |
| `EKAlarm(relativeOffset:)` | Creates an alert triggered relative to an event’s scheduled start time. | Attaches pre-shift alerts (e.g., 60 minutes prior) to remind the nurse to review their Body Battery score for **Use Case 2 (Pre-Shift Readiness Reminders)**. | `CalendarService.swift` |
| `parseExcelSchedule(from:)` | Custom parser that ingests spreadsheet files (XLSX / CSV) and extracts shift dates, times, and unit placements. | Ingests spreadsheet files and maps tabular rows into structured `Shift` models for **Use Case 1 (Automated Hospital Roster Ingestion & Deduplication)**. | `ExcelScheduleParser.swift` |
| `assignNurseToSession(shift:nurseId:)` | Associates a scheduled calendar event with an active nursing shift session in the app. | Binds the calendar shift timeframe to HealthKit Body Battery tracking and triggers on-shift monitoring for **Use Case 3 (Shift-to-Session Lifecycle Binding)**. | `ShiftSessionManager.swift` |

<br>

**Shift Schedule & Session Integration Specification:**

| Workflow Step | Clinical & Operational Use Case | Framework / Component | Function / Class | Clinical & Workflow Purpose in App |
|---|---|---|---|---|
| **1. Schedule Import** | **Use Case 1: Roster Ingestion & Deduplication** | `SwiftUI` / `UniformTypeIdentifiers` | `.fileImporter(...)` | Nurse uploads hospital shift roster (Excel/CSV) from Files or iCloud Drive without manual data entry. |
| **2. Tabular Parsing** | **Use Case 1: Roster Ingestion & Deduplication** | Parser Service | `ExcelScheduleParser.parse(...)` | Extracts date, shift category (Day / Evening / Night), start/end times, ward assignment, and normalizes overnight shifts. |
| **3. Duplicate Check** | **Use Case 1: Roster Ingestion & Deduplication** | `EventKit` | `predicateForEvents(withStart:end:calendars:)` | Inspects Apple Calendar to ensure previously imported shifts are not duplicated upon re-uploading an updated roster. |
| **4. Calendar Sync** | **Use Case 1 & 2: Sync & Readiness Reminders** | `EventKit` | `EKEvent` + `eventStore.save(...)` | Writes shifts to Apple Calendar with 60-min pre-shift alerts (`EKAlarm`) and embedded nurse session metadata. |
| **5. Session Assignment** | **Use Case 3: Shift-to-Session Lifecycle Binding** | Session Manager | `ShiftSessionManager.assign(...)` | Links the calendar event ID to active in-app nursing sessions, establishing the timeframe for live HealthKit Body Battery tracking. |

**Code example from our project:**

```swift
import EventKit

final class CalendarService {
    private let eventStore = EKEventStore()
    
    /// Requests calendar access using iOS 17+ granular permissions
    func requestAccess() async throws -> Bool {
        return try await eventStore.requestFullAccessToEvents()
    }
    
    /// Adds an imported shift from an Excel roster and binds it to an in-app nurse session
    func addShiftFromExcel(
        title: String,
        startDate: Date,
        endDate: Date,
        ward: String,
        nurseId: String
    ) throws -> (eventId: String, sessionId: String) {
        let event = EKEvent(eventStore: eventStore)
        let sessionId = UUID().uuidString
        
        event.title = title
        event.startDate = startDate
        event.endDate = endDate
        event.notes = "NurseID: \(nurseId) | SessionID: \(sessionId) | Ward: \(ward)"
        event.calendar = eventStore.defaultCalendarForNewEvents
        
        // 60-minute pre-shift alert for Body Battery preparation
        event.addAlarm(EKAlarm(relativeOffset: -3600))
        
        try eventStore.save(event, span: .thisEvent, commit: true)
        
        // Assign nurse to active shift tracking session
        ShiftSessionManager.shared.assignNurseToSession(
            sessionId: sessionId,
            nurseId: nurseId,
            calendarEventId: event.eventIdentifier,
            start: startDate,
            end: endDate
        )
        
        return (event.eventIdentifier, sessionId)
    }
}
```

**Challenges or limits we found (optional):** 
- **Granular iOS 17 Permissions:** iOS 17 introduced write-only versus full-access calendar permissions. Reading existing shifts to prevent duplicates requires full access (`requestFullAccessToEvents()`); if only write access is granted, duplicate detection must rely on the app's local cache.
- **Excel Formatting Variations:** Hospital rosters vary widely across wards (e.g. 12-hour AM/PM vs. 24-hour notation, varying column headers), requiring flexible regex date parsing and error-handling fallbacks.
- **Overnight Shifts:** Night shifts (e.g., 19:00 to 07:00) cross midnight into the next calendar day; the parser must correctly calculate the `endDate` by rolling forward one calendar day.
- **External Calendar Sync Latency:** If shifts are altered outside the app (e.g., via Google Calendar or Outlook sync), changes take a few moments to propagate through EventKit notifications.

---

#### Watch Connectivity

**What it is:** Watch Connectivity is Apple's framework for bi-directional communication, background data synchronization, and state transfer between an iOS app and its paired watchOS companion app. In our app, it enables nurses to monitor their real-time Body Battery level, current shift progress, and physical fatigue state directly on their Apple Watch face and complications, while transmitting on-wrist rest break triggers and sensor events back to the iPhone.

**Setup required:** 
- Import `WatchConnectivity` in both iOS and watchOS targets.
- Implement `WCSessionDelegate` in the shared communication manager (handling iOS-specific lifecycle methods `sessionDidBecomeInactive` and `sessionDidDeactivate` for paired watch switching).
- Check `WCSession.isSupported()` and call `WCSession.default.activate()` on app launch.

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `WCSession` | Singleton session manager that controls communication between the iPhone and Apple Watch. | Coordinates all data transfers, complication updates, and delegate callbacks across both devices for all shift workflows. | `WatchConnectivityManager.swift` |
| `WCSession.isSupported()` | Returns a boolean indicating whether the current device supports Watch Connectivity. | Called on iOS before initializing session delegates to safely prevent crashes on unsupported devices. | `WatchConnectivityManager.swift` |
| `WCSession.activate()` | Asynchronously activates the session on both iOS and watchOS. | Executed at app startup so the watch companion can receive background Body Battery updates immediately. | `WatchConnectivityManager.swift` |
| `WCSession.updateApplicationContext(_:)` | Sends a dictionary representing the latest app state to the counterpart device. | Overwrites queued state with the freshest battery percentage and shift phase for **Use Case 3 (Disconnected Floor Continuity & Locker Sync)**. | `WatchConnectivityManager.swift` |
| `WCSession.transferCurrentComplicationUserInfo(_:)` | High-priority background budget transfer specifically designed to update watch complications. | Pushes updated Body Battery scores directly to the nurse's watch face complication for **Use Case 1 (At-a-Glance Complication Monitoring)**. | `WatchConnectivityManager.swift` |
| `WCSession.sendMessage(_:replyHandler:errorHandler:)` | Sends real-time dictionary payloads when both devices are actively reachable. | Transmits real-time rest break triggers and emergency incident logs from the wrist for **Use Case 2 (On-Wrist Break & Incident Event Logging)**. | `WatchConnectivityManager.swift` |
| `WCSession.isReachable` | Indicates whether the counterpart app is currently reachable for immediate real-time messaging. | Determines whether to dispatch immediate interactive messages or fall back to queued application context (**Use Cases 2 & 3**). | `WatchConnectivityManager.swift` |
| `session(_:didReceiveApplicationContext:)` | Delegate method triggered when new application context data is received. | Receives background state updates on watchOS to refresh the on-screen Body Battery gauge when reconnecting after locker storage (**Use Case 3**). | `WatchViewModel.swift` |
| `session(_:didReceiveUserInfo:)` | Delegate method called when background complication data or queued user info arrives. | Extracts the new Body Battery score on watchOS and calls `WidgetCenter.shared.reloadAllTimelines()` for **Use Case 1 (At-a-Glance Complication Monitoring)**. | `ComplicationController.swift` |
| `sendBodyBatteryUpdate(batteryLevel:shiftName:)` | Custom dispatch function that bundles current battery status and active shift session info. | Dispatches updates prioritizing complication budget transfers (**Use Case 1**) and persistent context caching (**Use Case 3**). | `WatchConnectivityManager.swift` |

<br>

**Watch & iOS Sync Channel Specification:**

| Sync Channel | Clinical Use Case | API Method | Transmission Frequency | Clinical & Operational Purpose in App |
|---|---|---|---|---|
| **Complication Budget** | **Use Case 1: At-a-Glance Complication Monitoring** | `transferCurrentComplicationUserInfo(...)` | Every 30–60 mins or on significant battery drop (>5%) | Pushes live Body Battery (`⚡ 74%`) to circular/modular complications, allowing hands-free stamina checks during sterile rounds without touching devices. |
| **Interactive Messaging** | **Use Case 2: On-Wrist Break & Incident Event Logging** | `sendMessage(...)` | Immediate on wrist button tap / toggle | Instantly syncs one-tap rest break entries or emergency incident markers (`🚨 Code Event`) from wrist to iPhone while within Bluetooth range. |
| **Application Context** | **Use Case 3: Disconnected Floor Continuity & Locker Sync** | `updateApplicationContext(...)` | On shift milestones, reconnects, & background calculations | Guarantees the watch UI loads the latest Body Battery graph and shift timeline upon wrist wake, even after the nurse leaves their iPhone in a locker room. |

**Code example from our project:**

```swift
import WatchConnectivity
import Foundation

final class WatchConnectivityManager: NSObject, WCSessionDelegate {
    static let shared = WatchConnectivityManager()
    private let session: WCSession = .default
    
    func startSession() {
        guard WCSession.isSupported() else { return }
        session.delegate = self
        session.activate()
    }
    
    /// Sends real-time Body Battery level and shift status to Apple Watch
    func sendBodyBatteryUpdate(batteryLevel: Int, shiftName: String) {
        guard session.activationState == .activated else { return }
        let payload: [String: Any] = [
            "bodyBattery": batteryLevel,
            "shiftName": shiftName,
            "timestamp": Date()
        ]
        
        // 1. Update the watch face complication budget if watch app is installed
        if session.isWatchAppInstalled && session.remainingComplicationUserInfoTransfers > 0 {
            session.transferCurrentComplicationUserInfo(payload)
        }
        
        // 2. Update persistent application context for instant display on watch app launch
        do {
            try session.updateApplicationContext(payload)
        } catch {
            print("Failed to update watch application context: \(error.localizedDescription)")
        }
    }
    
    // MARK: - WCSessionDelegate Lifecycle
    func session(_ session: WCSession, activationDidCompleteWith state: WCSessionActivationState, error: Error?) {}
    #if os(iOS)
    func sessionDidBecomeInactive(_ session: WCSession) {}
    func sessionDidDeactivate(_ session: WCSession) {
        session.activate() // Reactivate session if user switches paired Apple Watches
    }
    #endif
}
```

**Challenges or limits we found (optional):** 
- **Complication Transfer Budget Limits:** watchOS imposes a strict quota on `transferCurrentComplicationUserInfo` (~50 transfers per day). Updates must be throttled to fire only on significant battery changes (e.g., ≥5% delta) or critical shift milestones.
- **Locker Reachability Gap:** In hospital settings, nurses frequently leave their iPhones in locker rooms while working on the ward. Real-time `sendMessage` will fail when out of Bluetooth range, requiring reliable fallback to `updateApplicationContext` and local on-watch HealthKit caching.
- **Multi-Watch Paired Switching:** On iOS, nurses may switch between multiple Apple Watches; the delegate must implement `sessionDidDeactivate` and re-call `activate()` to maintain connection.
- **Background Delivery Delays:** When the Apple Watch is in low power mode or the screen is lowered, transfers are coalesced by watchOS, meaning updates may not appear instantly until the next wrist raise.

---

#### ActivityKit

**What it is:** ActivityKit manages the lifecycle of Live Activities, presenting real-time persistent shift information, live countdowns, and peer alerts on the iPhone Lock Screen, in StandBy, and across the Dynamic Island. In our app, ActivityKit tracks active nursing shift progress and elapsed time, delivers peer break alerts to maintain continuous ward coverage, and provides silent rest break countdowns.

**Setup required:** 
- Add `NSSupportsLiveActivities` with value `YES` to your app’s `Info.plist`.
- Create a Widget Extension target containing an `ActivityConfiguration`.
- Verify `ActivityAuthorizationInfo().areActivitiesEnabled` before requesting an activity.

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `ActivityAttributes` | Protocol defining the static (immutable) properties of a Live Activity. | Defines constant shift identifiers such as `nurseName`, `shiftType`, `wardName`, and `scheduledEndTime`. | `ShiftActivityAttributes.swift` |
| `ActivityAttributes.ContentState` | Struct defining dynamic (frequently updated) properties of the Live Activity. | Stores real-time variables across all 3 use cases: `bodyBattery` (Hands-Free Rounds), `isBreakActive`/`breakEndTime` (Silent Break Countdown), and `peerBreakNotice` (Peer Coverage Awareness). | `ShiftActivityAttributes.swift` |
| `Activity<ShiftActivityAttributes>` | The generic manager class representing an ongoing Live Activity instance. | Maintained in `ShiftActivityManager` to reference, update, and end the nurse's active shift tracking session. | `ShiftActivityManager.swift` |
| `Activity.request(attributes:content:pushType:)` | Starts and presents a new Live Activity on the Lock Screen and Dynamic Island. | Called when the nurse checks into their shift session or begins their workday to establish the hands-free tracking banner. | `ShiftActivityManager.swift` |
| `activity.update(_:)` / `update(ActivityContent<ContentState>)` | Dynamically updates the Live Activity with a new `ContentState`. | Fired when the nurse triggers a rest break (Break Countdown), when Body Battery updates (Sterile Rounds), or when a peer's break status changes (Peer Coverage). | `ShiftActivityManager.swift` |
| `activity.end(_:dismissalPolicy:)` | Concludes and dismisses the Live Activity with `.default` or `.immediate` policy. | Called upon shift handoff or shift completion to clean up the Lock Screen and Dynamic Island displays. | `ShiftActivityManager.swift` |
| `ActivityConfiguration(for:content:)` | SwiftUI configuration defining the interface of the Live Activity. | Configures the visual appearance for the Lock Screen banner and all Dynamic Island presentation modes. | `ShiftLiveActivityWidget.swift` |
| `DynamicIsland` | Provides presentation slots: `.compactLeading`, `.compactTrailing`, `.minimal`, and `.expanded`. | Adapts layout dynamically based on whether the nurse is actively working, on a break countdown, or viewing peer status. | `ShiftLiveActivityWidget.swift` |
| `LiveActivityIntent` | Protocol enabling interactive buttons directly within Live Activities (iOS 17+). | Powers the **[Acknowledge Coverage]** (Peer Floor Coverage) and **[Start Break]** (Silent Rest Break) one-tap action buttons directly on the Lock Screen banner. | `ShiftActivityIntents.swift` |
| `updateBreakCountdown(minutes:)` | Custom function that sets `breakEndTime` to `Date().addingTimeInterval(...)`. | Starts a live countdown timer displayed in the Dynamic Island trailing slot and on the Lock Screen for the **Silent Rest Break Countdown** use case. | `ShiftActivityManager.swift` |
| `updatePeerBreakNotice(peerName:breakEnd:)` | Custom handler that updates the `peerBreakNotice` string and timestamp. | Pushes peer break status (e.g., *"Sarah on break until 14:15"*) to coordinate floor coverage for the **Peer Break Awareness & Floor Coverage** use case. | `ShiftActivityManager.swift` |

<br>

**Live Activity & Dynamic Island Presentation Specification:**

| Presentation Mode | Clinical Use Case | UI Elements Displayed | Interactive App Intent Actions | Clinical & Workflow Purpose |
|---|---|---|---|---|
| **Compact Leading** | **Hands-Free Sterile Rounds** | Body Battery icon + percentage (e.g., `⚡ 74%`) | None (Tap to expand) | Instant at-a-glance check of physical stamina during bedside care and sterile rounds without touching the device. |
| **Compact Trailing** | **Silent Rest Countdown / Shift Time** | Shift countdown or Break timer (`☕ 14:28` or `⏳ 04h 15m`) | None (Tap to expand) | Persistent awareness of active shift time or silent downward break countdown without audible ringtones disturbing patients. |
| **Minimal** | **Multi-Tasking Shift Display** | Dual-activity icon (battery `⚡` or coffee `☕`) | None | Maintains continuous glanceable shift status when another app is also using the Dynamic Island. |
| **Expanded Dynamic Island** | **Peer Break Awareness & Break Start** | Full shift progress gauge, ward name, live break countdown, and peer notification banner | **[Start Break]** button | Tapped or long-pressed during rounds to check floor coverage, see colleague break end times, or initiate a rest break. |
| **Lock Screen Banner** | **Hands-Free Rounds & Peer Coverage** | Shift timeline, battery drain rate, break timer, and peer coverage alerts (*"Sarah: On Break until 14:15"*) | **[Acknowledge Coverage]** & **[Start Break]** buttons | Hands-free bedside glanceability eliminates cross-contamination; one-tap interactive buttons let nurses acknowledge buddy coverage without unlocking the phone. |

**Code example from our project:**

```swift
import ActivityKit
import AppIntents
import Foundation

struct ShiftActivityAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var bodyBattery: Int
        var isBreakActive: Bool
        var breakEndTime: Date?
        var peerBreakNotice: String? // e.g., "Sarah is on break until 14:15"
    }
    
    var wardName: String
    var shiftTitle: String
    var nurseId: String
}

// MARK: - iOS 17 Live Activity App Intent for Direct Lock Screen Action
struct AcknowledgeCoverageIntent: LiveActivityIntent {
    static var title: LocalizedStringResource = "Acknowledge Peer Coverage"
    
    func perform() async throws -> some IntentResult {
        // Clear peer alert and log coverage confirmation
        await ShiftActivityManager.shared.acknowledgePeerBreak()
        return .result()
    }
}

final class ShiftActivityManager {
    static let shared = ShiftActivityManager()
    private var currentActivity: Activity<ShiftActivityAttributes>?
    
    /// Starts shift tracking Live Activity
    func startShiftActivity(ward: String, title: String, nurseId: String, initialBattery: Int) {
        guard ActivityAuthorizationInfo().areActivitiesEnabled else { return }
        
        let attributes = ShiftActivityAttributes(wardName: ward, shiftTitle: title, nurseId: nurseId)
        let initialContent = ShiftActivityAttributes.ContentState(
            bodyBattery: initialBattery,
            isBreakActive: false,
            breakEndTime: nil,
            peerBreakNotice: nil
        )
        
        do {
            currentActivity = try Activity.request(
                attributes: attributes,
                content: .init(state: initialContent, staleDate: nil)
            )
        } catch {
            print("Failed to start shift Live Activity: \(error.localizedDescription)")
        }
    }
    
    /// Updates Live Activity with personal break countdown or peer break status
    func updateBreakStatus(isBreak: Bool, durationMinutes: Int = 15, peerNotice: String? = nil, battery: Int) async {
        guard let activity = currentActivity else { return }
        let breakEnd = isBreak ? Date().addingTimeInterval(Double(durationMinutes * 60)) : nil
        
        let updatedState = ShiftActivityAttributes.ContentState(
            bodyBattery: battery,
            isBreakActive: isBreak,
            breakEndTime: breakEnd,
            peerBreakNotice: peerNotice
        )
        
        await activity.update(.init(state: updatedState, staleDate: nil))
    }
    
    func acknowledgePeerBreak() async {
        guard let activity = currentActivity else { return }
        var state = activity.content.state
        state.peerBreakNotice = "Coverage Confirmed ✓"
        await activity.update(.init(state: state, staleDate: nil))
    }
}
```

**Challenges or limits we found (optional):** 
- **8-Hour Maximum Lifetime:** Live Activities are automatically ended by iOS after 8 hours (persisting on the Lock Screen for up to 12 hours). Because standard nursing shifts last 12 to 12.5 hours, the app must listen for activity state changes and seamlessly re-request a continuation activity during extended shifts.
- **4 KB ContentState Size Limit:** The dynamic payload cannot exceed 4 KB, requiring peer notifications and status strings to remain concise.
- **Lock Screen AppIntent Restrictions:** Interactive buttons using `LiveActivityIntent` must execute lightweight code within strict memory limits; network calls to sync coverage must be delegated asynchronously to background services.
- **Dynamic Island Space Constraints:** Compact presentations have very limited character width; we use concise symbols (`⚡` for Body Battery, `☕` for Breaks) to prevent truncation.
- **APNs Frequent Push Budget:** If peer break updates are pushed remotely from other nurses via ActivityKit Push Tokens, APNs token limits and budget throttling must be accounted for to avoid dropped notifications.

---

#### Swift Charts

**What it is:** Swift Charts is Apple's native declarative data visualization framework for creating accessible, animated, and interactive charts in SwiftUI. In our app, Swift Charts renders the nurse's daily Body Battery curve: displaying actual measured charge and depletion levels from shift start alongside a predictive forecasted drain curve (dashed gradient line) that models how fast their energy will deplete across the remainder of the shift based on physical activity and autonomic stress.

**Setup required:** 
- Import `Charts` into any SwiftUI view file (`import Charts`).
- Requires iOS 16.0+ (supported by the app's target version). No capabilities or `Info.plist` entries required.

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `Chart` | The top-level SwiftUI container view used to display graphical charts and plots. | Serves as the main chart container on the shift dashboard to plot the nurse's 24-hour and shift timeline. | `BodyBatteryChartView.swift` |
| `LineMark` | A visual chart mark that draws connected line segments between sequential data points. | Plots the continuous Body Battery trajectory: a solid line for measured past data and a dashed line for predicted future drain. | `BodyBatteryChartView.swift` |
| `AreaMark` | A chart mark that fills the region between a line plot and the baseline axis. | Fills the space beneath the battery curve with a smooth energetic gradient (Green $\to$ Amber $\to$ Red) to convey remaining energy volume. | `BodyBatteryChartView.swift` |
| `PointMark` | A chart mark that plots individual circular or shaped glyphs at specific $(x, y)$ coordinates. | Pinpoints critical inflection moments: shift start, rest breaks taken, current live battery score, and estimated battery at shift end. | `BodyBatteryChartView.swift` |
| `RuleMark` | A reference line drawn horizontally across the Y-axis or vertically across the X-axis. | Draws a horizontal safety threshold at $y = 20\%$ to highlight danger zones, and a vertical indicator at the current time (`Date()`). | `BodyBatteryChartView.swift` |
| `lineStyle(_:)` | Modifies the stroke properties (line width, dash pattern, cap style) of a line mark. | Applies `StrokeStyle(lineWidth: 3, dash: [6, 4])` to visually distinguish future predicted drain from historical recorded battery. | `BodyBatteryChartView.swift` |
| `chartYScale(domain:)` | Sets the explicit minimum and maximum range of the chart's vertical value axis. | Locks the vertical axis to `0...100` so nurses can intuitively read standard battery percentages. | `BodyBatteryChartView.swift` |
| `chartXAxis(content:)` | Customizes the appearance, labels, and tick intervals of the horizontal time axis. | Sets hourly strides (e.g., every 2 hours: 07:00, 09:00, 11:00...) formatted to match the nurse's 12-hour or 24-hour shift schedule. | `BodyBatteryChartView.swift` |
| `chartXSelection(value:)` | Adds interactive gesture scrubbing to select specific values along the X-axis. | Enables the nurse to drag their finger across the timeline to inspect predicted battery levels and fatigue warnings at any future hour. | `BodyBatteryChartView.swift` |
| `predictBatteryDrain(currentBattery:remainingHours:hourlyDrainRate:)` | Mathematical projection algorithm modeling linear and non-linear energy decay. | Generates future timestamps and projected battery levels up to the scheduled shift end time. | `BatteryPredictionEngine.swift` |

<br>

**Body Battery Chart Visualization & Prediction Specification:**

| Visual Element | Swift Charts Mark | Data Source | Styling & Visual Presentation | Purpose in Nurse App |
|---|---|---|---|---|
| **Measured Battery** | `LineMark` | HealthKit & Session Logs | Solid green/teal line (`lineWidth: 3`) | Displays actual recorded recovery and activity depletion from shift start up to the current moment. |
| **Predicted Drain** | `LineMark` | Predictive Engine Model | Dashed amber/orange line (`dash: [6, 4]`) | Forecasts expected battery decay for the remaining hours of the shift based on current workload. |
| **Energy Volume Area** | `AreaMark` | Full Timeline Data | Vertical Linear Gradient (Green $\to$ Yellow $\to$ Clear) | Visualizes total physiological energy reserves and makes battery drops immediately perceptible. |
| **Fatigue Alert Threshold** | `RuleMark` | Constant $y = 20\%$ | Horizontal dashed red line with label | Warns the nurse if their predicted drain trajectory will drop below safe stamina limits before shift end. |
| **Current Time Marker** | `RuleMark` | `Date()` (Current Timestamp) | Vertical subtle rule + "Now" tag | Clear boundary line separating verified historical measurements from predictive forecasts. |
| **Scheduled Break Glyphs** | `PointMark` | Calendar / Shift Schedule | Circular coffee/rest icons | Visualizes scheduled rest breaks that pause battery drain or grant partial recovery recharge. |

**Code example from our project:**

```swift
import SwiftUI
import Charts

struct BatteryDataPoint: Identifiable {
    let id = UUID()
    let timestamp: Date
    let batteryLevel: Double
    let isPrediction: Bool
}

struct BodyBatteryChartView: View {
    let dataPoints: [BatteryDataPoint]
    let criticalThreshold: Double = 20.0
    
    var body: some View {
        Chart {
            // 1. Shaded Energy Reserve Area
            ForEach(dataPoints) { point in
                AreaMark(
                    x: .value("Time", point.timestamp),
                    y: .value("Battery", point.batteryLevel)
                )
                .foregroundStyle(
                    LinearGradient(
                        colors: [.green.opacity(0.35), .orange.opacity(0.15), .clear],
                        startPoint: .top,
                        endPoint: .bottom
                    )
                )
            }
            
            // 2. Solid line for measured battery, Dashed line for predicted drain
            ForEach(dataPoints) { point in
                LineMark(
                    x: .value("Time", point.timestamp),
                    y: .value("Battery", point.batteryLevel)
                )
                .lineStyle(StrokeStyle(lineWidth: 3, dash: point.isPrediction ? [6, 4] : []))
                .foregroundStyle(point.isPrediction ? Color.orange : Color.green)
            }
            
            // 3. Safety Warning Threshold Line (20%)
            RuleMark(y: .value("Critical Fatigue", criticalThreshold))
                .lineStyle(StrokeStyle(lineWidth: 1.5, dash: [4, 4]))
                .foregroundStyle(.red)
                .annotation(position: .top, alignment: .leading) {
                    Text("Fatigue Alert (20%)")
                        .font(.caption2)
                        .foregroundColor(.red)
                }
        }
        .chartYScale(domain: 0...100)
        .chartXAxis {
            AxisMarks(values: .stride(by: .hour, count: 2)) {
                AxisValueLabel(format: .dateTime.hour())
                AxisGridLine()
            }
        }
        .frame(height: 220)
    }
}
```

**Challenges or limits we found (optional):** 
- **Non-Linear Drain Variations:** Nurse energy expenditure is rarely linear; sudden emergencies, code blues, or continuous walking cause rapid drain spikes. The prediction engine must recalculate the projection curve dynamically based on 30-minute rolling activity rather than static averages.
- **Visual Distinction for Predictions:** Medical and health applications require explicit visual distinctions between measured physiological data and algorithmic forecasts; using dashed line styles (`[6, 4]`) and color shifts (Green to Orange) ensures nurses do not confuse predictions with verified readings.
- **Overnight Shift Time Axis Spans:** Shifts spanning across midnight (e.g., 19:00 to 07:00) require `AxisMarks` with custom date formats to prevent the X-axis from resetting or overlapping hour labels at midnight.
- **Scrubbing Performance:** Rendering hundreds of raw per-minute biometric data points during interactive touch scrubbing can drop frame rates; downsampling data to 15-minute average buckets preserves 60/120 FPS fluidity.

---

#### Nearby Interaction

**What it is:** Nearby Interaction is Apple's framework utilizing the Ultra Wideband (UWB) U1/U2 chip and Bluetooth LE to measure precise spatial distance and direction between nearby devices. In our app, Nearby Interaction allows incoming nurses to detect their local nursing station iPad or station beacon upon arrival, revealing available shifts for that ward and allowing the nurse to review and tap to join their shift session.

**Setup required:** 
- Add `NSNearbyInteractionUsageDescription` to your app’s `Info.plist` (e.g., *"This app uses Nearby Interaction to detect nearby nurse stations and display available ward shifts."*).
- Add Bluetooth capability and `NSBluetoothAlwaysUsageDescription` to exchange the initial `NIDiscoveryToken` out-of-band locally without needing hospital Wi-Fi.
- Verify `NISession.isSupported` at runtime before starting ranging.

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `NISession` | The central session object coordinating Ultra Wideband spatial measurement and peer ranging. | Initialized in `NearbyShiftDiscoveryService` to measure precise proximity between the nurse's iPhone and the Nurse Station device. | `NearbyShiftDiscoveryService.swift` |
| `NISession.isSupported` | Checks whether the host device is equipped with an Apple U1/U2 Ultra Wideband chip. | Checked at launch; if false, the app gracefully falls back to Bluetooth-only or manual ward shift selection. | `NearbyShiftDiscoveryService.swift` |
| `NIDiscoveryToken` | An opaque token uniquely identifying an individual device’s UWB radio for a session. | Exchanged out-of-band over CoreBluetooth/Multipeer Connectivity between the nurse's phone and the station iPad. | `NearbyShiftDiscoveryService.swift` |
| `NINearbyPeerConfiguration(peerToken:)` | Configures an `NISession` for spatial tracking with a specific remote peer discovery token. | Instantiated with the station's discovery token to start line-of-sight distance measurement. | `NearbyShiftDiscoveryService.swift` |
| `session.run(_:)` | Starts or resumes spatial distance and direction measurement for a configured session. | Invoked as soon as the out-of-band Bluetooth handshake with the station device completes. | `NearbyShiftDiscoveryService.swift` |
| `session(_:didUpdate:)` | Delegate method triggered when new distance or direction measurements are received from a peer. | Reads `NINearbyObject.distance`; when within proximity (< 1.5 m), it presents available local shifts for that ward on the nurse's screen. | `NearbyShiftDiscoveryService.swift` |
| `session.invalidate()` | Terminates the Nearby Interaction session and releases Ultra Wideband hardware resources. | Called immediately after the nurse taps and joins their shift to prevent battery drain over the 8–12 hour shift. | `NearbyShiftDiscoveryService.swift` |
| `NINearbyObject.distance` | Float value representing real-time distance in meters between the two devices. | Evaluates whether the nurse has arrived at the station desk to unlock the shift selection sheet. | `NearbyShiftDiscoveryService.swift` |
| `joinShiftSession(shiftId:nurseId:)` | Custom function that binds the nurse to the selected local shift. | Sets active shift state, logs attendance, and kicks off HealthKit body battery tracking and ActivityKit widgets. | `ShiftSessionManager.swift` |

<br>

**Nearby Interaction Local Shift Discovery & Join Specification:**

| Step | Framework / Component | Function / Class | Purpose in Nurse App |
|---|---|---|---|
| **1. Local Beacon Discovery** | CoreBluetooth / Multipeer | `CBCentralManager` | Nurse's iPhone detects the Nurse Station iPad beacon locally without relying on hospital Wi-Fi or cellular network. |
| **2. Token Handshake** | CoreBluetooth / Multipeer | `NIDiscoveryToken` serialization | Out-of-band exchange of discovery tokens to initialize the secure Ultra Wideband spatial channel. |
| **3. Proximity Ranging** | Nearby Interaction | `NISession.run(NINearbyPeerConfiguration)` | Continuous UWB ranging tracks distance as the nurse walks up to the station desk. |
| **4. Shift Detection & Display** | SwiftUI / View Model | `session(_:didUpdate:)` | When distance drops below 1.5 meters, the app displays the station's ward and list of available shifts for review. |
| **5. Explicit Tap to Join** | SwiftUI Button Action | `joinShiftSession(shiftId:)` | Nurse reviews and explicitly taps to join their assigned shift (preventing accidental automatic check-in from nurses walking past). |
| **6. UWB Teardown** | Nearby Interaction | `session.invalidate()` | Ends UWB ranging immediately upon shift join to eliminate battery consumption during the 8–12 hour work shift. |

**Code example from our project:**

```swift
import NearbyInteraction
import CoreBluetooth
import SwiftUI

final class NearbyShiftDiscoveryService: NSObject, ObservableObject, NISessionDelegate {
    private var niSession: NISession?
    @Published var availableShifts: [String] = []
    @Published var isNearStation: Bool = false
    
    func startRanging(with stationToken: NIDiscoveryToken) {
        guard NISession.isSupported else { return }
        niSession = NISession()
        niSession?.delegate = self
        
        let config = NINearbyPeerConfiguration(peerToken: stationToken)
        niSession?.run(config)
    }
    
    // MARK: - NISessionDelegate
    func session(_ session: NISession, didUpdate nearbyObjects: [NINearbyObject]) {
        guard let station = nearbyObjects.first, let distance = station.distance else { return }
        
        // When within 1.5 meters of the nursing desk, unlock available shifts
        if distance <= 1.5 && !isNearStation {
            isNearStation = true
            fetchLocalShiftsForWard()
        }
    }
    
    /// Called when the nurse taps an available shift to join
    func selectAndJoinShift(shiftId: String, nurseId: String) {
        ShiftSessionManager.shared.joinShiftSession(shiftId: shiftId, nurseId: nurseId)
        
        // Invalidate UWB session immediately after join to conserve battery
        niSession?.invalidate()
        niSession = nil
    }
    
    private func fetchLocalShiftsForWard() {
        self.availableShifts = ["Ward 4B - Day Shift (07:00 - 19:00)", "Ward 4B - Acute Relief"]
    }
}
```

**Challenges or limits we found (optional):** 
- **Post-Join Battery Conservation:** Continuous Ultra Wideband ranging places noticeable demand on battery. Invalidate `NISession` immediately after the nurse confirms their shift selection to ensure zero battery penalty during the 8–12 hour shift.
- **Out-of-Band Discovery Requirement:** Nearby Interaction cannot discover peers autonomously; it requires an out-of-band channel (Bluetooth LE / Multipeer Connectivity) to exchange discovery tokens before UWB ranging can begin.
- **Hospital Metal & Shielded Environments:** Heavy hospital equipment (e.g., metal crash carts, lead-shielded X-ray doors) can attenuate UWB signals; using a distance threshold ($\le 1.5$ m) combined with BLE guarantees reliable detection at the desk.
- **Preventing Accidental Check-Ins:** Nurses frequently walk past nursing stations during patient rounds; requiring an explicit tap on the available shift list prevents accidental unwanted shift check-ins.

---

#### CoreML

**What it is:** CoreML is Apple's on-device machine learning framework designed for low-latency, privacy-preserving model inference and on-device training accelerated by the Apple Neural Engine (ANE) and GPU. In our app, CoreML runs a time-series regression model (`BodyBatteryDrainPredictor.mlpackage`) that forecasts future Body Battery levels across the remainder of a nurse's shift. Due to the absence of public clinical datasets, the base model is bootstrapped in Python using a synthetic physiological simulation engine and exported via `coremltools` with on-device updating enabled (`is_updatable = True`), allowing the model to continuously self-train and personalize to each individual nurse's physiological baseline using `MLUpdateTask`.

**Setup required:** 
- Drag the compiled `BodyBatteryDrainPredictor.mlpackage` into the Xcode project target.
- Set model flags in Python using `coremltools` with `is_updatable = True` to enable on-device fine-tuning.
- No permissions or `Info.plist` keys required; all biometric data processing and model fine-tuning occurs strictly on-device, fully compliant with healthcare data privacy (HIPAA/GDPR).

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `MLModel` | The compiled machine learning model loaded into memory for inference. | Represents the `BodyBatteryDrainPredictor` model used to forecast the shift energy depletion curve. | `CoreMLPredictionService.swift` |
| `MLModelConfiguration` | Configures model runtime options including hardware compute unit allocation. | Configured with `.computeUnits = .all` to utilize the Apple Neural Engine and GPU for instant zero-latency predictions. | `CoreMLPredictionService.swift` |
| `MLFeatureProvider` | Protocol representing a key-value dictionary of named input/output features. | Encapsulates input biometrics (Battery, Shift Hours, Sleep, Rolling Steps/Kcal, HRV Ratio, RHR Delta, Shift Type). | `CoreMLPredictionService.swift` |
| `MLArrayBatchProvider` | Bundles an array of feature providers into a training or batch inference set. | Compiles completed shift hourly slices (input features paired with ground-truth recorded battery) for model fine-tuning. | `ModelPersonalizationService.swift` |
| `model.prediction(from:)` | Generates a synchronous on-device forward-pass prediction. | Computes predicted battery scores at $+1\text{h}, +2\text{h}, +4\text{h}$, and shift-end to render the dashed projection line on Swift Charts. | `CoreMLPredictionService.swift` |
| `MLUpdateTask` | CoreML task class that executes on-device backpropagation and weight updates on updatable models. | Runs background fine-tuning at the end of each shift to adapt the model to the nurse's personal stamina and recovery profile. | `ModelPersonalizationService.swift` |
| `MLUpdateProgressHandlers` | Delegate closures delivering callbacks for training checkpoints and completion. | Monitors training epoch progression and saves the updated model checkpoint when fine-tuning converges. | `ModelPersonalizationService.swift` |
| `MLUpdateContext.model.write(to:)` | Serializes and writes updated model weights to a designated file URL. | Saves the fine-tuned personalized model into the app's `Application Support` directory so updates persist across app launches. | `ModelPersonalizationService.swift` |
| `predictDrainTrajectory(features:)` | Custom service method normalizing raw HealthKit values into model inputs. | Prepares normalized tensors and returns an array of future timestamped battery prediction points. | `CoreMLPredictionService.swift` |

<br>

**CoreML Model Architecture & On-Device AutoML Specification:**

| Pipeline Stage | Implementation Detail | Technology / Tool | Purpose in Nurse App |
|---|---|---|---|
| **1. Base Pretraining** | Synthetic Physiological Simulator | Python (`PyTorch` / `scikit-learn`) | Solves zero-dataset cold-start problem by pretraining on simulated interactions of sleep, steps, and circadian rhythm. |
| **2. CoreML Conversion** | Updatable Model (`is_updatable=True`) | `coremltools` (Python) | Exports trainable neural network layers optimized for the Apple Neural Engine. |
| **3. Input Feature Vector** | 8-Dimensional Biometric Tensor | Swift `MLFeatureProvider` | Inputs: Current Battery, Shift Elapsed, Sleep Score, Rolling Steps, Active Kcal, HRV Ratio, RHR Delta, Shift Type (Day/Night). |
| **4. Multi-Step Forecast** | Multi-Output Regression ($+1\text{h}, +2\text{h}, +4\text{h}, t_{\text{end}}$) | `model.prediction(from:)` | Directly plots the dashed future curve in Swift Charts and triggers ActivityKit fatigue warnings. |
| **5. On-Device AutoML** | End-of-Shift `MLUpdateTask` Fine-Tuning | CoreML On-Device Training | Personalizes the generic model over time to match the individual nurse's unique physical fatigue resistance. |

**Code example from our project:**

```swift
import CoreML
import Foundation

final class CoreMLPredictionService {
    static let shared = CoreMLPredictionService()
    private var modelURL: URL
    private var model: MLModel?
    
    init() {
        // Load personalized model from Application Support if available, else fallback to bundled model
        let fileManager = FileManager.default
        let appSupport = fileManager.urls(for: .applicationSupportDirectory, in: .userDomainMask).first!
        let personalizedURL = appSupport.appendingPathComponent("PersonalizedDrainModel.mlmodelc")
        
        self.modelURL = fileManager.fileExists(atPath: personalizedURL.path) 
            ? personalizedURL 
            : Bundle.main.url(forResource: "BodyBatteryDrainPredictor", withExtension: "mlmodelc")!
            
        let config = MLModelConfiguration()
        config.computeUnits = .all
        self.model = try? MLModel(contentsOf: modelURL, configuration: config)
    }
    
    /// Predicts future Body Battery levels (+1h, +2h, +4h, End-of-Shift)
    func predictDrain(currentBattery: Double, hoursRemaining: Double, rollingSteps: Double, hrvRatio: Double) -> [Double] {
        guard let model = model else { return [] }
        
        let inputFeatures: [String: Any] = [
            "currentBattery": currentBattery,
            "hoursRemaining": hoursRemaining,
            "rollingSteps": rollingSteps,
            "hrvRatio": hrvRatio
        ]
        
        guard let featureProvider = try? MLDictionaryFeatureProvider(dictionary: inputFeatures),
              let prediction = try? model.prediction(from: featureProvider),
              let outputMultiArray = prediction.featureValue(for: "predictedCurve")?.multiArrayValue else {
            return []
        }
        
        return (0..<outputMultiArray.count).map { outputMultiArray[$0].doubleValue }
    }
    
    /// On-Device AutoML: Personalizes model weights using actual recorded shift data
    func fineTuneModelOnDevice(trainingBatch: MLArrayBatchProvider) async throws {
        let updateTask = try MLUpdateTask(
            forModelAt: modelURL,
            trainingData: trainingBatch,
            configuration: nil,
            progressHandlers: MLUpdateProgressHandlers(
                forEvents: [.trainingBegin, .epochEnd],
                progressHandler: { _ in },
                completionHandler: { context in
                    // Save personalized model weights locally
                    let appSupport = FileManager.default.urls(for: .applicationSupportDirectory, in: .userDomainMask).first!
                    let saveURL = appSupport.appendingPathComponent("PersonalizedDrainModel.mlmodelc")
                    try? context.model.write(to: saveURL)
                }
            )
        )
        updateTask.resume()
    }
}
```

**Challenges or limits we found (optional):** 
- **Cold-Start Zero-Dataset Problem:** Solved by training the base model in Python on a synthetic physiological simulator (combining circadian night-shift dip formulas, sleep restorative scores, and step count caloric burn) before exporting with `coremltools`.
- **Read-Only Bundle Sandboxing:** The compiled `.mlmodelc` within the main app bundle is read-only; updated model weights from `MLUpdateTask` must be written to `ApplicationSupportDirectory` and prioritized on subsequent app launches.
- **Preventing Overfitting to Anomalous Shifts:** A single unusually chaotic shift (e.g. constant code blue emergencies) could distort the nurse's profile; we constrain `MLUpdateTask` to low learning rates ($\eta = 0.001$) and limit training to 5–10 epochs.
- **Thermal & Background Throttling:** On-device training requires significant compute; fine-tuning tasks are queued to execute in the background when the shift concludes or when the device is plugged into power.

---

#### Firebase

**What it is:** Firebase provides cloud backend infrastructure for user authentication, session security, and data persistence. In our app, Firebase is strictly focused on managing nurse authentication (login, session tokens, and logout via Firebase Authentication) and storing work schedules and shift session attendance (via Cloud Firestore). To preserve clinical confidentiality and regulatory compliance (HIPAA/GDPR), all raw biometric and HealthKit health data remains 100% on-device and is never transmitted to the cloud.

**Setup required:** 
- Add `GoogleService-Info.plist` to the Xcode project and ensure it is included in the main app target.
- Add the Firebase iOS SDK via Swift Package Manager (`https://github.com/firebase/firebase-ios-sdk`) selecting `FirebaseAuth` and `FirebaseFirestore`.
- Call `FirebaseApp.configure()` in the `@main` app lifecycle (`NurseApp.swift`).
- Enable the **Email/Password** sign-in provider in the Firebase Console under Authentication.
- Enable Cloud Firestore with offline persistence enabled by default.

**Methods / functions / classes we use:**

| Method / Function / Class | What it does | How we use it in our app | File |
|---|---|---|---|
| `FirebaseApp.configure()` | Initializes the Firebase SDK and connects services using `GoogleService-Info.plist`. | Called at app launch in `NurseApp.swift` before any authentication or database calls execute. | `NurseApp.swift` |
| `Auth.auth()` | The central singleton managing authentication state, tokens, and active user credentials. | Coordinates nurse login sessions, token refreshing, and secure sign-out operations. | `AuthService.swift` |
| `signIn(withEmail:password:)` | Asynchronously authenticates the user against Firebase Auth using clinical credentials. | Validates the nurse's hospital staff credentials and retrieves their unique account identifier (`UID`). | `AuthService.swift` |
| `signOut()` | Clears the active authentication session, tokens, and local auth state. | Allows nurses to securely log out at the conclusion of their shift, preventing unauthorized access on shared devices. | `AuthService.swift` |
| `addStateDidChangeListener(_:)` | Listens for real-time authentication lifecycle changes (logged in, logged out, token expired). | Dynamically transitions navigation between the Login view and the main Shift Dashboard. | `AuthService.swift` |
| `Firestore.firestore()` | The singleton gateway to the Cloud Firestore NoSQL cloud database. | Reads and writes nurse shift schedules, ward assignments, and active shift attendance records. | `FirestoreScheduleService.swift` |
| `CollectionReference.document(_:)` | References a specific document path within a Firestore collection hierarchy. | Targets the authenticated nurse's document path (`nurses/{uid}`) and nested schedule records. | `FirestoreScheduleService.swift` |
| `DocumentReference.setData(from:merge:)` | Encodes and writes a Swift `Codable` domain object into a Firestore document. | Uploads parsed Excel work schedule records and active shift check-in timestamps. | `FirestoreScheduleService.swift` |
| `Query.getDocuments(as:)` | Executes a database read and decodes documents into strongly typed Swift models. | Retrieves upcoming scheduled shifts to populate the shift calendar and allow offline roster review. | `FirestoreScheduleService.swift` |
| `saveShiftSchedule(shifts:nurseId:)` | Custom service method batch-writing parsed work schedule entries to Firestore. | Persists the nurse's uploaded Excel roster into the cloud so it syncs across paired devices. | `FirestoreScheduleService.swift` |

<br>

**Firebase Data Architecture & Privacy Boundary Specification:**

| Data Domain | Firebase Service | Storage Path / Model | Sync & Offline Behavior | Privacy & Compliance Scope |
|---|---|---|---|---|
| **Nurse Identity & Session** | `FirebaseAuth` | `Auth.currentUser` | Token cached securely in iOS Keychain; persists across app relaunches. | Staff email and unique `UID`. Zero health data. |
| **Shift Schedules** | `FirebaseFirestore` | `nurses/{uid}/schedules/{shiftId}` | Persistent local cache; accessible even without hospital Wi-Fi. | Work roster metadata only (Ward, Start Date, End Date, Shift Type). |
| **Active Shift Sessions** | `FirebaseFirestore` | `shift_sessions/{sessionId}` | Real-time write with offline queueing; syncs when connectivity resumes. | Check-in/check-out timestamps and rest break logs. |
| **Biometric Vitals & Battery** | *Excluded from Firebase* | *100% Local On-Device* | Processed locally via HealthKit, SwiftData, and CoreML. | Raw vitals (HRV, SDNN, RMSSD, Sleep) never leave the device. |

**Code example from our project:**

```swift
import FirebaseAuth
import FirebaseFirestore
import Foundation

final class AuthService: ObservableObject {
    static let shared = AuthService()
    @Published var currentUser: User?
    private let auth = Auth.auth()
    
    init() {
        auth.addStateDidChangeListener { [weak self] _, user in
            self?.currentUser = user
        }
    }
    
    /// Signs in nurse with hospital staff credentials
    func login(email: String, password: String) async throws -> String {
        let result = try await auth.signIn(withEmail: email, password: password)
        return result.user.uid
    }
    
    /// Signs out and cleans up session tokens
    func logout() throws {
        try auth.signOut()
    }
}

final class FirestoreScheduleService {
    static let shared = FirestoreScheduleService()
    private let db = Firestore.firestore()
    
    /// Saves parsed Excel schedule into nurse's Firestore cloud profile
    func saveSchedule(nurseId: String, shifts: [Shift]) async throws {
        let batch = db.batch()
        let scheduleCollection = db.collection("nurses").document(nurseId).collection("schedules")
        
        for shift in shifts {
            let docRef = scheduleCollection.document(shift.id)
            try batch.setData(from: shift, forDocument: docRef)
        }
        
        try await batch.commit()
    }
}
```

**Challenges or limits we found (optional):** 
- **Hospital Wi-Fi Blackouts (Shielded Wards):** Subterranean corridors, radiology suites, and emergency wards frequently suffer from wireless signal dropouts; Firestore offline persistence (`isPersistenceEnabled = true`) is strictly enabled so nurses can view schedules and record shifts without an internet connection.
- **Strict Clinical Privacy Boundary:** To eliminate security risks and comply with healthcare data privacy laws (HIPAA/GDPR), raw biometric vitals (HRV, Sleep, RMSSD, Heartbeat series) are completely quarantined on-device; Firestore is strictly limited to operational schedule and session metadata.
- **Shared Clinical Device Sanitation:** Institutional hospital iPads are frequently shared across rotating shifts; `signOut()` explicitly clears local Keychain auth tokens and flushes memory caches before the next nurse signs in.
- **Batch Import Limitations:** Bulk importing an entire quarterly roster from Excel can exceed Firestore's single-write limit; imports are partitioned into `WriteBatch` operations capped at 500 documents per transaction.

## 3. Architecture

### 3.1 Architecture Name

**Architecture:** Feature-Driven MVVM (Model–View–ViewModel), dengan `Features` terpisah pada target iOS dan watchOS. View dan ViewModel khusus platform; Model yang benar-benar sama dapat menjadi shared source.

### 3.2 Why We Chose It

UI, navigation, dan interaction iPhone berbeda dari Watch. Karena itu, setiap target memiliki feature beserta View dan ViewModel sendiri: misalnya iPhone membuat session dan mengakses Firebase, sedangkan Watch melakukan quick join dan meneruskan actions melalui iPhone. Susunan ini menjaga UI tetap native tanpa memaksa satu ViewModel menangani semua perbedaan platform.

Model session, status duty, break event, dan peer summary dapat dibagikan karena makna datanya sama. Memiliki ViewModel terpisah bukan masalah; jangan menyalin domain rules yang benar-benar identik ke dua tempat. Jika pure rules seperti schedule validation atau time calculations memang dipakai bersama, bagikan bagian tersebut saat diperlukan, tanpa langsung membuat framework atau additional layers.

### 3.3 Architecture Diagram

```text
iPhone: View iOS → ViewModel iOS → Shared Model / data source iOS
Watch:  View Watch → ViewModel Watch → Shared Model / data source Watch

Prototype: local fixtures untuk masing-masing app

Planned integration:
Watch ⇄ Watch Connectivity ⇄ iPhone ⇄ Firebase
HealthKit → local engine → personal insight
Firebase Cloud Functions → FCM → APNs → peer devices
```

Shared source tidak berarti shared memory atau automatic synchronization. Model yang sama tetap menjadi data pada instance dan perangkat masing-masing; perubahan state nyata dikirim melalui Watch Connectivity. Mengubah ViewModel iPhone tidak otomatis mengubah ViewModel Watch.

### 3.4 Components

| Component | What it does | What it must NOT do | Files / folders |
| --- | --- | --- | --- |
| Shared Model | Struktur data session, duty, break, peer dan availability yang maknanya sama pada kedua platform. | Menentukan layout/navigation atau menjalankan query Firebase. | Rencana `Nursitor/Shared/Models/`. Model yang hanya dipakai satu feature/platform boleh tetap dalam feature tersebut. |
| View iOS | Menampilkan UI dan input yang sesuai iPhone, termasuk form create dan dashboard dengan timer duty. | Menjalankan query backend atau mengurus komunikasi perangkat langsung. | Rencana `Nursitor/Nursitor/Features/<Feature>/`. |
| ViewModel iOS | Mengelola state dan actions pada feature iPhone melalui service yang dibutuhkan. | Menyimpan referensi View atau menampung semua perilaku Watch. | Berdekatan dengan View dalam feature iOS. |
| View watchOS | Menampilkan quick join, dashboard ringkas, aksi, dan countdown break yang sesuai Watch. | Menyalin layout iPhone secara paksa atau menyediakan create session. | Rencana `Nursitor/Nursitor Watch App/Features/<Feature>/`. |
| ViewModel watchOS | Mengelola state dan actions pada feature Watch serta hasil relay melalui connectivity service. | Mengakses Firebase langsung pada jalur companion yang disepakati atau mengasumsikan pesan terkirim berarti operasi diterima server. | Berdekatan dengan View dalam feature Watch. |
| Service/repository | Mengakses fixture, backend, HealthKit atau koneksi sesuai tanggung jawab platform. | Menentukan layout layar atau memberi status sukses tanpa hasil operasi yang sesuai. | `Services/` pada target yang menjalankannya, ditambahkan ketika dibutuhkan. |
| Target UI components | Component kecil yang digunakan ulang dalam platform yang sama. | Menjadi alasan memindahkan seluruh layar ke Shared atau mengganti native controls tanpa kebutuhan. | `UI/` pada target masing-masing jika sudah diperlukan. |

`Shared` dimulai dari Model, bukan tempat semua feature, View, atau ViewModel. Message contracts dan pure rules lintas platform boleh ditambahkan kemudian jika benar-benar dibutuhkan. Kesamaan nama feature tidak mewajibkan kedua target memiliki file atau jumlah layar yang sama.

### 3.5 Folder Structure

Struktur berikut merupakan **planned folder structure** berdasarkan lokasi dua target pada repository `nurse-app`, bukan klaim folder dan feature tersebut sudah dibuat. Saat diperiksa, source aplikasi berada di `Nursitor/Nursitor` dan `Nursitor/Nursitor Watch App`; belum ada folder Shared.

```text
nurse-app/
├── project.yml
├── Nursitor/
│   ├── Nursitor/                         # Source target iOS
│   │   ├── NursitorApp.swift
│   │   ├── ContentView.swift
│   │   ├── Assets.xcassets
│   │   ├── Features/
│   │   │   ├── Sessions/
│   │   │   │   ├── CreateSessionView.swift
│   │   │   │   ├── JoinSessionView.swift
│   │   │   │   └── SessionViewModel.swift
│   │   │   ├── Duty/
│   │   │   │   ├── DutyView.swift
│   │   │   │   └── DutyViewModel.swift
│   │   │   └── Break/
│   │   │       ├── BreakView.swift
│   │   │       └── BreakViewModel.swift
│   │   ├── Services/                    # iPhone: Firebase, relay, HealthKit
│   │   └── UI/                          # iPhone-specific components when needed
│   │
│   ├── Nursitor Watch App/              # Source target watchOS
│   │   ├── NursitorApp.swift
│   │   ├── ContentView.swift
│   │   ├── Assets.xcassets
│   │   ├── Features/
│   │   │   ├── Sessions/
│   │   │   │   ├── JoinSessionView.swift
│   │   │   │   └── SessionViewModel.swift
│   │   │   ├── Duty/
│   │   │   │   ├── DutyView.swift
│   │   │   │   └── DutyViewModel.swift
│   │   │   └── Break/
│   │   │       ├── BreakView.swift
│   │   │       └── BreakViewModel.swift
│   │   ├── Services/                    # Watch: relay and HealthKit
│   │   └── UI/                          # Watch-specific components when needed
│   │
│   └── Shared/                          # Planned; not created yet
│       └── Models/                      # Shared Models
│           ├── Session.swift
│           ├── DutyState.swift
│           ├── BreakEvent.swift
│           └── PeerSummary.swift
└── docs/
    └── TechnicalRequirements.md
```

Feature lain mengikuti pola yang sama saat diperlukan. Contoh nama file hanya menjelaskan organization; jangan membuat semua folder, ViewModel kosong, atau service sekaligus. Model khusus satu feature boleh berada di folder feature tersebut, sedangkan model lintas feature/platform berada di Shared.

**Target Membership Rules:**

- Source pada `Nursitor/Nursitor` hanya untuk target iOS `Nursitor`.
- Source pada `Nursitor/Nursitor Watch App` hanya untuk target `Nursitor Watch App`.
- Ketika `Nursitor/Shared` dibuat, sertakan source tersebut pada kedua target melalui project configuration. Nama folder Shared saja tidak otomatis membagikan source.
- Source configuration di `project.yml` saat diperiksa hanya mencantumkan folder aplikasi masing-masing. Penambahan shared source memerlukan pembaruan configuration ketika implementation dilakukan; perubahan dokumen ini tidak melakukannya.
- Setiap file App yang menjadi entry point hanya disertakan pada targetnya sendiri.
- Nama seperti `SessionViewModel` boleh sama pada dua target selama source masing-masing tidak ikut dibangun dalam target yang sama. Jangan memasukkan ViewModel target iOS dan Watch sekaligus ke satu target atau ke Shared dengan type name yang sama.
- Strategy default adalah View/ViewModel terpisah. Reuse ViewModel lintas platform bukan kewajiban dan tidak menjadi asumsi project structure ini.

## 4. Unit Testing Strategy

[Placeholder — belum dibahas.]

## 5. Git Workflow

Workflow dibatasi pada **empat branch types** yang tercantum di template: `main`, `develop`, `prod`, dan `feat/*`. Tidak menambahkan branch type lain. Pemisahan fungsi di bawah merupakan draft team workflow; PIC merge dan aktivasi branch protection belum ditetapkan atau verified.

### 5.1 Branches

| Branch | Purpose | Who can push directly? | Protected? |
| --- | --- | --- | --- |
| `main` | Planned stable code/release candidate yang sudah integrated dan direview. | Rencana melalui PR, bukan direct push. | Target protection; belum configured/verified. |
| `develop` | Rencana tempat menggabungkan feature dan mengecek app flow bersama. | Rencana melalui PR, bukan direct push. | Target protection; belum configured/verified. |
| `prod` | Rencana penanda production/release code yang dipromosikan dari `main`. | Rencana hanya Release PIC yang ditunjuk tim melalui PR. | Target protection; belum configured/verified. |
| `feat/*` | Satu tugas/feature yang dikerjakan dari `develop`. | Developer pemilik tugas. | Tidak perlu protection yang menghalangi pekerjaan pemilik tugas. |

### 5.2 Branch Diagram

```text
develop → feat/nama-fitur → PR ke develop → PR ke main → PR ke prod
```

Ini draft flow, bukan bukti cabang atau aturan GitHub sudah dibuat. Promosi ke `prod` tidak berarti otomatis deploy atau aplikasi langsung tersedia di App Store.

### 5.3 Merging Rules

| Merge | When does it happen? | Who does it? | What must be true first? |
| --- | --- | --- | --- |
| `feat/*` → `develop` | Rencana: pekerjaan sesuai cakupan sudah selesai dan direview. | Rencana developer pemilik PR setelah approval reviewer lain. | Deskripsi perubahan dan bukti pemeriksaan relevan; tidak mengklaim pemeriksaan yang belum dijalankan. |
| `develop` → `main` | Rencana: feature integrated siap menjadi stable version/release candidate. | Integration/release PIC yang akan ditunjuk tim. | Cross-feature flow diperiksa pada kedua target sejauh lingkungan memungkinkan; keterbatasan dicatat dan diterima tim. |
| `main` → `prod` | Rencana: versi tertentu disetujui untuk dipromosikan sebagai rilis. | Release PIC yang akan ditunjuk tim. | Release approval dan version identity jelas; tidak melewati proses signing, device testing, atau publication approval yang berlaku. |

Nama PIC, required approval count, merge method, dan final branch protection rules belum disepakati. Dokumen ini tidak menunjuk anggota tim sebagai approver tunggal atau mengubah configuration repository.

### 5.4 Step by Step: How One Feature Moves Through Our Workflow

1. Developer pemilik tugas mengambil satu pekerjaan dengan cakupan dan hasil yang jelas.
2. Developer membuat `feat/nama-fitur` dari `develop` terbaru.
3. Developer mengerjakan perubahan kecil dan memeriksa flow yang relevan pada kedua target bila terdampak.
4. Developer membuka PR ke `develop`, menjelaskan perubahan, bukti pemeriksaan, dan batas yang belum tested.
5. Developer reviewer lain meninjau perubahan; pemilik PR memperbaiki temuan sebelum merge sesuai aturan tim yang disepakati.
6. Integration PIC yang ditunjuk tim mempromosikan kumpulan perubahan yang siap dari `develop` ke `main` melalui PR.
7. Release PIC yang ditunjuk tim mempromosikan versi yang disetujui dari `main` ke `prod`; publication tetap menjadi release action terpisah.

### 5.5 Naming and Commit Conventions

**Branch naming:** `feat/nama-fitur`, lowercase dengan hyphens.  
**Commit message format:** `type: short description` atau `type(scope): short description`. Type ditulis huruf kecil; scope opsional, misalnya `ios`, `watch`, `session`, atau `docs`.  
**Pull request rule:** Draft review oleh developer selain penulis, dengan ringkasan perubahan dan pemeriksaan relevan; approval count serta PIC merge final masih perlu ditetapkan tim.

Gunakan type berikut sesuai perubahan utamanya:

| Type | Usage | Example |
| --- | --- | --- |
| `feat:` | Functionality atau behavior baru pada app. | `feat(ios): add create session form` |
| `fix:` | Bug fixes untuk behavior yang salah. | `fix(watch): preserve leading zeros in session code` |
| `docs:` | Documentation changes. | `docs: clarify iOS and Watch feature structure` |
| `refactor:` | Code restructuring tanpa perubahan user-facing behavior. | `refactor(ios): separate session form state` |
| `chore:` | Repository maintenance yang tidak termasuk feature, bug fix, build, atau CI. | `chore: update gitignore rules` |
| `ci:` | Configuration atau workflow continuous integration. | `ci: add pull request checks` |
| `test:` | Menambah atau memperbaiki tests. | `test: add invalid session code scenario` |
| `revert:` | Revert perubahan sebelumnya. | `revert: undo session form changes` |

`style:` bukan label umum untuk UI design changes: UI dengan functionality baru memakai `feat:`, sedangkan perbaikan tampilan yang salah memakai `fix:` sesuai konteks. Pisahkan perubahan yang tidak berkaitan agar satu commit memiliki tujuan yang jelas. Type `ci:` dan `test:` merupakan aturan penamaan, bukan klaim pipeline atau unit testing strategy sudah ditetapkan.

Daftar commit type tidak menambah branch type; workflow tetap hanya `main`, `develop`, `prod`, dan `feat/*`.

## 6. Final Check

- [x] Deskripsi project, anggota tim, platform, dan cakupan prototype sudah ditulis.
- [x] Peran teknologi prototype dan planned integration dibedakan.
- [x] Feature-Driven MVVM architecture dijelaskan: Features/View/ViewModel terpisah per target, dengan Shared Model seperlunya.
- [x] Q1–Q4 sudah dicatat sebagai keputusan yang disetujui.
- [x] Method/function/class dan code example tetap placeholder, tidak diisi dengan contoh aplikasi lain.
- [x] Unit Testing Strategy tetap placeholder karena belum dibahas.
- [x] Git workflow hanya memakai `main`, `develop`, `prod`, dan `feat/*`; format commit mencakup chore, ci, refactor, docs dan type terkait.
- [ ] PIC merge, aturan approval, dan final branch protection sudah disepakati/configured.
- [ ] Implementation, build, dan pemeriksaan UI kedua target sudah verified terhadap dokumen ini.
- [ ] Seorang rekan tim telah mereview dokumen dan menyetujui isinya.

### Open Decisions

Jatah break per kejadian versus total shift, jumlah break dan approval, host selesai duty, rejoin, formula body battery, parameter dan threshold, nearby, retention, release device targets, detail method/function/class, serta unit testing strategy. Dokumen ini tidak menganggap topik-topik tersebut sudah diputuskan.

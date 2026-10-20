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

### 2.1 Technology Overview

Status di bawah membedakan teknologi prototype dari planned integration. Tercantum dalam dokumen bukan berarti framework sudah installed atau feature sudah tested.

| Technology | Type | Why we use it | Used in which component | Status |
| --- | --- | --- | --- | --- |
| Swift | Programming language | Menulis aplikasi native pada dua platform. | Seluruh component | Used in prototype |
| SwiftUI | Apple UI framework | Menyusun UI native, form, navigation, status, dan adaptive layout. | View pada iPhone dan Watch | Used in prototype |
| Foundation | Apple framework | Model data, event/session identity, tanggal, jadwal, dan time calculations. | Model, ViewModel, fixture | Used in prototype |
| Observation | Apple framework | Menghubungkan perubahan state prototype dengan tampilan. | ViewModel dan binding View | Used in prototype |
| HealthKit | Apple framework | Membaca data kesehatan yang diizinkan untuk insight pribadi lokal. | Health service dan engine lokal | Planned integration |
| Watch Connectivity | Apple framework | Menghubungkan Watch dengan iPhone pasangannya untuk command dan session state. | Connectivity service kedua target | Planned integration |
| UserNotifications | Apple framework | Pengingat lokal akhir break/saran dan penanganan notification yang diizinkan. | Notification service | Planned integration |
| Firebase Authentication | Firebase SDK/service | Anonymous identity tanpa layar email/password. | Service iPhone dan backend identity checks | Planned integration |
| Cloud Firestore | Firebase database/service | Menyimpan session, membership, state operasional, dan ringkasan yang disetujui. | Service iPhone dan backend | Planned integration |
| Cloud Functions for Firebase | Managed backend | Sensitive operation validation, alokasi kode, perubahan session, dan pengajuan push. | Backend Firebase | Planned integration |
| Firebase Cloud Messaging dan APNs | Push service | Menyampaikan notification break kepada perangkat rekan melalui jalur Apple. | Backend, service iPhone, routing notification sistem | Planned integration |
| XCTest UI Tests / XCUIAutomation | Apple UI testing tools | Memeriksa perjalanan pengguna melalui UI; terpisah dari unit testing strategy yang belum dibahas. | Testing flow kedua app sesuai destination | Planned; belum diklaim installed |

SF Symbols dan system font menjadi visual choices dalam UI native. Swift Charts, CloudKit, Nearby Interaction, Core ML, dan backend Go tidak menjadi dependency wajib prototype ini.

### 2.2 Technology Details

Detail method, function, class, serta code example sengaja berupa placeholder sesuai keputusan tim. Tidak ada nama API atau implementation example yang ditetapkan melalui bagian ini.

#### Swift

**What it is:** Programming language untuk aplikasi native Apple.  
**How we use it:** Menulis model, state management, View, dan integration kedua aplikasi.  
**Setup required:** Gunakan toolchain Xcode yang kompatibel dengan project configuration; tidak mengubah versi bahasa atau minimum target tanpa kebutuhan yang jelas.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]

#### SwiftUI

**What it is:** Declarative framework dari Apple untuk membangun UI.  
**How we use it:** Membuat form create/join, dashboard, peer, break dan insight, dengan layout khusus iPhone/Watch ketika interaksinya berbeda. Utamakan native controls, system font, SF Symbols, Dynamic Type, VoiceOver, dan status yang tidak bergantung warna saja.  
**Setup required:** Kedua target sudah menggunakan SwiftUI; layout dan style harus diperiksa terhadap target platform.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Untuk proyek ini, View dan ViewModel ditempatkan dalam feature target masing-masing. Dashboard Watch memiliki layout dan interaksi tersendiri; tidak perlu memaksakan satu View bersama untuk iPhone dan Watch.

#### Foundation

**What it is:** Framework dasar untuk data dan operasi umum aplikasi.  
**How we use it:** Merepresentasikan session, schedule, time zone, identity dan timestamp; menghitung durasi duty serta countdown yang dapat diperbarui ketika aplikasi aktif kembali.  
**Setup required:** Schedule calculations harus menangani tanggal lengkap dan shift lintas tengah malam.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]

#### Observation

**What it is:** Framework Apple untuk observe data changes.  
**How we use it:** Menyediakan state prototype yang diamati View, tanpa menempatkan networking atau session rules langsung di tampilan.  
**Setup required:** Memerlukan dukungan compiler dan OS yang kompatibel; pengaturan project yang ada diperiksa sebelum perubahan.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]

#### HealthKit

**What it is:** Framework untuk mengakses data kesehatan sesuai user permission.  
**How we use it:** Rencana membaca kandidat recovery dan activity data untuk insight lokal. Final parameters, baseline, formula body battery, threshold, dan validation belum dikunci; prototype hanya memakai fixture.  
**Setup required:** Capability, penjelasan penggunaan data, data type selection, permission dan device support checks ditentukan ketika integration dimulai.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Data dapat tidak tersedia, tidak cukup, atau tidak dapat diakses ketika terkunci. Background updates tidak dijanjikan tepat setiap jam atau terus-menerus. Read permission bukan sharing consent ke peer.

#### Watch Connectivity

**What it is:** Framework komunikasi Watch dengan iPhone pasangannya.  
**How we use it:** Rencana meneruskan join, duty, break dan state melalui iPhone yang mengakses Firebase. Shared message contracts dibedakan dari handler yang hidup pada masing-masing perangkat.  
**Setup required:** Pairing, connection lifecycle, reachability, app installation, dan message contracts diperiksa pada real device saat integration.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Bukan komunikasi langsung antar-Watch perawat. Pesan diterima iPhone tidak otomatis berarti backend menerima operasi atau peer membaca notification.

#### UserNotifications

**What it is:** Framework Apple untuk scheduling dan penyajian notification yang diizinkan.  
**How we use it:** Rencana pengingat lokal akhir break dan personal suggestions; push break berasal dari backend Firebase. Prototype hanya memperlihatkan perubahan status/pengingat saat UI aktif.  
**Setup required:** Permission, notification categories dan capability requirements ditentukan saat integration.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Penyajian iPhone atau Watch mengikuti sistem Apple; tidak ada jaminan selalu muncul di kedua perangkat atau sudah dibaca.

#### Firebase Authentication

**What it is:** User identity service dari Firebase.  
**How we use it:** Planned anonymous UID di balik layar untuk feature online; Watch menggunakan identity context dari iPhone, bukan login email/password terpisah.  
**Setup required:** Project configuration Firebase development dan dukungan anonymous authentication ketika integration dimulai.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Identity recovery setelah credentials hilang atau pindah perangkat tidak boleh diasumsikan otomatis. Tidak memerlukan backend Go terpisah.

#### Cloud Firestore

**What it is:** Managed document database Firebase.  
**How we use it:** Rencana session storage, membership, status operasional dan indikator peer yang disetujui. Raw biomarker dan personal suggestions tidak disimpan di sini.  
**Setup required:** Security Rules yang menolak akses secara default, indexes sesuai kebutuhan dan environment separation; schema details diselesaikan saat integration.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Akses dibatasi role, membership dan consent; private document dipisahkan dari peer summary. Jalur Watch MVP menggunakan relay iPhone.

#### Cloud Functions for Firebase

**What it is:** Backend computing yang dikelola dalam ekosistem Firebase.  
**How we use it:** Planned validation untuk identity/role, alokasi kode unik di antara session valid, satu duty aktif per pengguna, akhir session, reconciliation dan pengajuan notification yang belum kedaluwarsa.  
**Setup required:** Environment configuration dan deployment, service/budget limits, server-side validation, serta emulator dibahas ketika integration dimulai.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Operasi dapat diulang; desain harus mencegah duplikasi dan pengaruh operasi session lama terhadap kode yang sudah dipakai ulang. Managed backend tetap diperlukan; Firebase-only tidak berarti semua validation berada di client.

#### Firebase Cloud Messaging dan APNs

**What it is:** Jalur pengiriman push melalui Firebase dan layanan notification Apple.  
**How we use it:** Rencana break-start notification kepada peer lain yang masih berhak, termasuk host. Pesan tidak mengandung raw biomarker atau skor pada lock screen.  
**Setup required:** Token configuration dan push credentials pada layanan yang benar saat integration; tidak menyimpan credentials server di aplikasi.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Submission bukan delivery/read confirmation. Break-start notice yang melewati countdown limit tidak diajukan ulang; state Break/overdue tetap dapat dicocokkan.

#### XCTest UI Tests / XCUIAutomation

**What it is:** Apple testing tools untuk interaksi melalui UI aplikasi.  
**How we use it:** Arah UI flow testing yang dipilih adalah create/join → duty → break → selesai, beserta error, consent dan accessibility. Ini UI testing, bukan keputusan unit testing strategy.  
**Setup required:** Test bundle dan destination sesuai aplikasi ditambahkan ketika pekerjaan testing dimulai; automated dan manual coverage dibedakan.  
**Methods / functions / classes we use:** [Placeholder — belum dibahas.]  
**Code example from our project:** [Placeholder — belum dibahas.]  
**Challenges or limits:** Dukungan project/destination Watch harus diperiksa. Belum ada klaim UI automation atau testing real device lulus.

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

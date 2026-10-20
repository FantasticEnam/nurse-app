# Nursitor — Nurse App

Aplikasi pendamping kesehatan kerja perawat untuk iPhone dan Apple Watch, dikembangkan oleh **Fantasix** menggunakan SwiftUI dan **Feature-Driven MVVM**.

Repository ini masih berupa starter untuk dua app dengan tampilan “Hello, world!”. Feature session, duty, body battery, HealthKit, Firebase, dan Watch Connectivity belum tersedia. Product dan technical decisions ada di [Technical Requirements](docs/TechnicalRequirements.md).

**Documentation style:** Penjelasan boleh menggunakan bahasa Indonesia, tetapi technical terms, heading, nama component, dan istilah workflow yang umum tetap dalam English. Gunakan istilah yang sama secara konsisten, misalnya `session`, `feature`, `configuration`, `permission`, `consent`, `testing`, dan `deployment`.

## Prerequisites

- macOS yang mendukung versi Xcode yang digunakan.
- **Xcode 27.1 recommended**, dengan SDK iOS dan watchOS 27 atau lebih baru. Minimum deployment target tetap iOS 27.0 dan watchOS 27.0.
- **XcodeGen 2.46.0 atau lebih baru** dan Git.
- Simulator runtime iPhone/Apple Watch yang sesuai. Download melalui Xcode Settings jika destination belum tersedia.
- Akses ke repository GitHub. Apple Account/Development Team diperlukan untuk run pada physical device; simulator tidak memerlukan signing team.

Setup ini diperiksa menggunakan Xcode 27.1, Apple Swift 6.4, dan XcodeGen 2.46.0. `SWIFT_VERSION: "6.0"` berarti **Swift 6 language mode**, bukan keharusan install compiler Swift 6.0 secara terpisah.

## Clone and Open the Project

### 1. Set Up Xcode

Buka Xcode setidaknya sekali dan selesaikan installation untuk additional components atau license agreement yang muncul. Periksa active toolchain:

```bash
xcode-select -p
xcodebuild -version
```

Jika toolchain masih menunjuk ke Command Line Tools atau versi Xcode yang salah, pilih **Xcode → Settings → Locations → Command Line Tools**, lalu pilih Xcode yang benar.

### 2. Install XcodeGen

Jika menggunakan [Homebrew](https://brew.sh), install XcodeGen:

```bash
brew install xcodegen
xcodegen --version
```

Jika XcodeGen sudah tersedia tetapi versinya di bawah 2.46.0, update installation sebelum generate. Installation alternatives ada di [XcodeGen documentation](https://github.com/yonaskolb/XcodeGen#installing).

### 3. Clone the Repository

```bash
git clone https://github.com/FantasticEnam/nurse-app.git
cd nurse-app
```

Jika akses ditolak, pastikan akun GitHub memiliki repository access dan Git authentication sudah tersedia pada mesinmu. Jangan menaruh access token di README atau command yang akan dibagikan.

### 4. Generate the Xcode Project

Jalankan dari repository root yang berisi `project.yml`:

```bash
xcodegen generate --spec project.yml
open Nursitor.xcodeproj
```

Output-nya adalah **`Nursitor.xcodeproj` di repository root**, bukan di dalam source folder `Nursitor/`. Project dan shared schemes dihasilkan dari `project.yml`; generated project di-ignore Git sesuai `.gitignore`.

Starter ini belum menggunakan Firebase/Swift Package/CocoaPods dependencies, sehingga tidak memerlukan `pod install`, Firebase configuration, atau credentials untuk membuka dan run UI awal.

## Regenerate the Project

Setelah switch branch atau menerima configuration/source changes baru, jalankan ulang:

```bash
xcodegen generate --spec project.yml
open Nursitor.xcodeproj
```

Regenerate terutama diperlukan setelah mengubah `project.yml`, menambah/menghapus/memindahkan source, atau mengubah dependency. Mengedit isi file Swift yang sudah terdaftar tidak memerlukan regenerate setiap kali.

Jadikan `project.yml` source of truth untuk shared configuration. Jangan hanya mengubah target, dependency, atau scheme melalui generated project lalu menganggap perubahan itu sudah tersedia untuk seluruh tim. Commit source dan YAML changes; tidak perlu commit generated `.xcodeproj`.

## Troubleshooting

| Issue | First Check |
| --- | --- |
| `xcodegen: command not found` | Install XcodeGen dan periksa terminal PATH. |
| Generate tidak menemukan source | Jalankan dari repository root; pastikan kedua source folder tersedia setelah clone. |
| Scheme tidak muncul | Regenerate dan buka `Nursitor.xcodeproj` di root. Kedua scheme didefinisikan di YAML. |
| iPhone/Watch destination tidak tersedia | Periksa versi Xcode, installed simulator runtimes, dan simulator pairing. |
| Development Team/provisioning belum tersedia | Untuk physical device, atur team kedua target; untuk simulator, pilih simulator yang benar. |
| Swift language version tidak didukung | Gunakan Xcode yang sesuai dan regenerate dari YAML terbaru. Language mode yang dipilih adalah Swift 6, bukan 5.10. |
| Configuration changes hilang setelah generate | Pindahkan shared configuration ke `project.yml`; atur ulang local signing jika diperlukan. |
| HealthKit/nearby access belum muncul | Feature tersebut belum tersedia pada starter; tidak ada system permission yang perlu diberikan untuk UI Hello World. |

## Contribution

Branch workflow dan commit format dijelaskan dalam [Technical Requirements](docs/TechnicalRequirements.md#5-git-workflow). Gunakan prefix yang sesuai, misalnya `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, atau `ci:`.

Baca feature requirements sebelum menambah framework atau permission. Unit testing strategy dan method/function/class details masih menunggu pembahasan tim; README ini tidak menetapkan strategy tersebut.

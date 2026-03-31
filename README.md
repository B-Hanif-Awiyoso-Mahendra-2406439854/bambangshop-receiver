# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: (SECRET)

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. Menurut saya, `RwLock<Vec<Notification>>` diperlukan karena data notifikasi dipakai bareng-bareng oleh banyak request. Di aplikasi receiver ini, ada endpoint yang nambah notifikasi (write) dan ada juga endpoint yang cuma baca/list notifikasi (read). Kalau tanpa lock, bisa terjadi data race karena beberapa thread mengakses `Vec` yang sama di waktu bersamaan.

    Kenapa pilih `RwLock` dan bukan `Mutex`? Karena pola aksesnya lebih sering baca daripada tulis. `RwLock` mengizinkan banyak reader berjalan paralel selama tidak ada writer, jadi throughput lebih bagus untuk endpoint list. Kalau pakai `Mutex`, baik read maupun write harus antre satu-satu, jadi lebih ketat dan bisa jadi bottleneck padahal operasi read seharusnya bisa barengan.

2. Soal `lazy_static`, ini dipakai karena kita butuh state global yang diinisialisasi sekali saat runtime (misalnya `Vec` dan `DashMap`), tapi tetap aman untuk concurrency. Di Rust, `static` biasa itu harus punya nilai yang diketahui saat compile-time dan tidak boleh sembarang dimutasi karena bisa melanggar memory safety.

    Berbeda dengan Java yang mengandalkan GC dan runtime checks, Rust sejak awal memaksa aturan ownership/borrowing yang ketat. Kalau mutable global dibebaskan seperti di Java, Rust jadi rawan data race dan undefined behavior. Jadi Rust tidak melarang mutasi sepenuhnya, tapi mutasinya harus lewat wrapper yang aman seperti `RwLock`, `Mutex`, atau struktur concurrent lain (`DashMap`). Menurut saya ini trade-off yang bagus: lebih "ribet" di awal, tapi bug concurrency bisa berkurang jauh.

#### Reflection Subscriber-2

1. Iya, saya sempat eksplor di luar step utama tutorial, terutama lihat `src/lib.rs` dan alur wiring antar module (`controller`, `service`, `repository`, `model`). Dari situ saya belajar kalau `lib.rs` itu penting sebagai "pintu" module, jadi struktur project lebih rapi dan dependency antar bagian lebih jelas. Saya juga jadi lebih paham kenapa handler di controller dibuat tipis (hanya terima request/return response), sedangkan logic utamanya didorong ke service dan repository. Menurut saya ini enak buat maintainability, karena kalau ada perubahan logic, dampaknya lebih terlokalisasi.

2. Setelah coba jalanin beberapa instance Receiver, Observer pattern terasa sangat membantu untuk nambah subscriber. Publisher tidak perlu tahu detail implementasi setiap subscriber; cukup simpan daftar subscriber dan broadcast event ke mereka. Jadi nambah subscriber baru relatif gampang, tinggal register/subscribe endpoint baru tanpa ubah banyak kode inti.

    Untuk kasus spawn lebih dari satu instance Main app (publisher), menurut saya masih bisa, tapi kompleksitasnya naik dibanding nambah subscriber. Kalau subscriber banyak ke satu publisher, polanya masih lurus. Tapi kalau publisher jadi banyak, kita perlu mikirin sinkronisasi state antar publisher (misalnya data produk dan event), duplikasi notifikasi, dan source of truth-nya di mana. Jadi "bisa" ditambah, tapi perlu desain tambahan (misalnya message broker/event bus atau shared storage) supaya tetap konsisten.

3. Saya sudah coba bikin test sederhana dan sedikit rapihin dokumentasi Postman collection (nama request, contoh payload, dan expected response). Buat saya ini kepakai banget. Test bantu cek regresi pas refactor kecil, jadi tidak perlu selalu test manual dari nol. Dokumentasi Postman juga mempermudah waktu demo dan kerja kelompok, karena anggota lain bisa langsung paham endpoint mana untuk subscribe/unsubscribe/receive/list tanpa harus baca kode dulu. Untuk Group Project, ini berasa menghemat waktu onboarding dan debugging.

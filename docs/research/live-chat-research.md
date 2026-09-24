# Riset live chat marketplace fashion second-hand

Tanggal akses sumber: 2026-09-24

## Ringkasan rekomendasi MVP

**Pilihan MVP yang direkomendasikan: Centrifugo OSS + Go + PostgreSQL.** Tidak ada biaya lisensi software, dan Centrifugo menangani koneksi realtime, fan-out, reconnect, presence, serta permission channel. Go tetap menangani aturan transaksi dan authorization, sedangkan PostgreSQL menjadi tempat utama untuk menyimpan pesan, audit, laporan pengguna, dan riwayat penanganan penyalahgunaan. [Centrifugo repository](https://github.com/centrifugal/centrifugo), [authentication](https://centrifugal.dev/docs/server/authentication), [history/recovery](https://centrifugal.dev/docs/server/history_and_recovery)

**Pilihan prototipe gratis: Supabase Free + PostgreSQL.** Supabase lebih cepat untuk menguji pengalaman realtime, tetapi tetap membawa ketergantungan pada Auth, Realtime, Storage, quota, dan operasi platform Supabase. Pesan final tetap harus ditulis ke PostgreSQL aplikasi. [Realtime](https://supabase.com/docs/guides/realtime), [authorization](https://supabase.com/docs/guides/realtime/authorization), [Storage](https://supabase.com/docs/guides/storage)

**Build WebSocket langsung bukan pilihan awal.** `github.com/coder/websocket` dapat menjadi primitive transport, tetapi tim harus membuat sendiri authorization channel, reconnect, deduplication, delivery state, attachment, laporan pengguna, pemblokiran, notifikasi, monitoring, dan recovery. [coder/websocket](https://github.com/coder/websocket)

**Pilihan produk chat lengkap: Stream Chat.** Fitur yang paling dekat dengan PRD sudah tersedia: channel, message history, attachments, push notification, presence, read receipts, typing indicator, flagging, dashboard penanganan laporan, role/permission, webhook, dan Go server SDK. Trade-off-nya adalah biaya berbayar mulai dari **$399/bulan bila ditagih tahunan atau $499/bulan bulanan** untuk paket Start, serta ketergantungan paling besar pada model data dan API vendor. [Chat docs](https://getstream.io/chat/docs/), [pricing](https://getstream.io/chat/pricing/)

Untuk MVP ini, **jangan menjadikan Pusher atau Ably sebagai penyimpanan chat utama**. Keduanya cocok sebagai transport realtime yang matang, dengan auth, presence, webhook/integrasi, dan pricing usage-based, tetapi penyimpanan pesan, attachment, laporan pengguna, dan audit marketplace tetap perlu dirancang di backend. Ably Chat menambah fitur chat siap pakai, tetapi beberapa fitur seperti flagging dan mute/ban ditandai `Coming soon` pada halaman pricing saat riset dilakukan. [Pusher Channels](https://pusher.com/docs/channels/), [Ably docs](https://ably.com/docs), [Ably pricing](https://ably.com/pricing)

Firebase/Firestore realistis bila tim menerima model NoSQL dan ekosistem Google. Firestore menyediakan realtime listeners, Security Rules, TTL, export/import, Storage untuk media, dan FCM untuk notifikasi. Untuk stack yang sudah menetapkan Go + PostgreSQL, penggunaan Firestore sebagai database kedua menambah sinkronisasi dan lock-in. [Firestore realtime](https://firebase.google.com/docs/firestore/query-data/listen), [Security Rules](https://firebase.google.com/docs/firestore/security/get-started), [pricing](https://firebase.google.com/docs/firestore/pricing), [Cloud Storage](https://firebase.google.com/docs/storage/web/upload-files), [FCM](https://firebase.google.com/docs/cloud-messaging)

## Tabel perbandingan

| Opsi | Realtime dan koneksi | Riwayat/audit | Akses buyer-seller-admin | Notifikasi | Foto/video | Laporan dan penyalahgunaan | Webhook/integrasi | Biaya tahap awal | Lock-in dan risiko |
|---|---|---|---|---|---|---|---|---|---|
| **Supabase Realtime** | Broadcast, Presence, Postgres Changes. | PostgreSQL cocok sebagai database utama; Broadcast replay hanya sekitar 72 jam sampai paling banyak 4 hari. | Private channel dan RLS dapat memetakan user ke transaksi; aturan transaksi tetap perlu diuji di aplikasi. | Realtime in-app; push mobile/web bukan fitur chat utama yang dibuktikan di halaman Realtime, jadi perlu FCM/Web Push/provider lain. | Supabase Storage mendukung image/video dan RLS. | Report, scanning, review queue, block/ban, dan case management perlu dibuat di aplikasi atau service lain. | REST/database broadcast; integrasi bisnis sebaiknya dipicu dari Go/outbox. | Free: $0, dengan batas Realtime 200 koneksi puncak dan 2 juta pesan/bulan. Pro: $25/bulan, termasuk 500 koneksi puncak dan 5 juta pesan/bulan; compute proyek dibebankan terpisah dan Pro memiliki kredit compute $10. [Pricing](https://supabase.com/pricing) | Sedang. Portabilitas lebih baik karena pesan dan aturan dapat tetap di PostgreSQL, tetapi channel protocol, Auth/Storage, dan operasi platform tetap spesifik Supabase. Self-host tersedia menurut pricing page, tetapi biaya operasional menjadi tanggung jawab tim. |
| **Firebase/Firestore** | Realtime listeners untuk query dokumen. | Dokumen persisten, TTL tersedia; audit dan retention perlu model serta konfigurasi sendiri. | Security Rules mendukung aturan granular dan role-based access; backend Go harus menjadi trusted environment untuk operasi privileged. | FCM mendukung notification/data message ke device, topic, group, dan device. | Cloud Storage untuk image/audio/video dengan Security Rules. | Rules bukan alur penanganan laporan. Report, antrean review, scanning, dan admin case management perlu dibangun. | Cloud Functions/event integrations tersedia di ekosistem Firebase; detail biaya dan trigger perlu dihitung terpisah. | Spark gratis tersedia untuk sebagian layanan; Firestore Blaze ditagih per reads/writes/deletes, storage, network, dan index. Total chat tidak dapat dihitung tanpa volume dan region, jadi perlu konfirmasi melalui pricing calculator. [Pricing](https://firebase.google.com/docs/firestore/pricing) | Tinggi untuk stack ini: model dokumen, billing per operasi, Security Rules, FCM, Storage, dan Cloud Functions berbeda dari Go/PostgreSQL. |
| **Pusher Channels** | WebSocket dengan fallback; publish/subscribe, private channel, presence channel. | Dashboard/activity dan webhook dapat merekam aktivitas, tetapi durable chat history harus berada di backend. | Private channel authorization tersedia; mapping `transaction_id` ke channel dan admin access harus dibuat di Go. | Channels realtime; Pusher Beams adalah produk push terpisah dan perlu dihitung/diuji bila dipakai. | Tidak menyediakan storage chat attachment sebagai fitur inti; gunakan object storage sendiri. | Channels tidak menyediakan alur bawaan untuk laporan dan penanganan penyalahgunaan; buat di aplikasi. | Webhooks, HTTP API, dan integrasi monitoring tersedia. | Sandbox gratis: 200 ribu pesan/hari dan 100 concurrent connections. Startup $49/bulan: 1 juta pesan/hari dan 500 connections. [Pricing](https://pusher.com/channels/pricing/) | Sedang-tinggi. Protocol relatif mudah diisolasi, tetapi quota pesan/hari, channel semantics, dashboard, dan auth signature menjadi ketergantungan vendor. |
| **Ably** | Pub/Sub dengan WebSocket dan HTTP fallback; presence, recovery, history, auth token/JWT, granular permissions. | Message history dan rewind tersedia; batas retention yang ditampilkan: Free 1 hari, Standard 30 hari, Pro 365 hari, Enterprise custom. Arsip audit jangka panjang tetap perlu PostgreSQL. | JWT/token auth dan granular permissions tersedia; membership transaksi dan admin escalation tetap milik Go. | Native mobile/web push notification tercantum pada feature matrix; verifikasi flow dan biaya push perlu dilakukan sebelum keputusan. | Tidak terlihat sebagai object storage chat pada fitur yang diteliti; gunakan storage sendiri. | Integrasi penanganan laporan tersedia; message flagging dan mute/ban ditandai `Coming soon` pada matrix pricing saat akses. | Webhooks, HTTP integrations, functions, streaming integrations, dan control API tersedia. | Free: $0, 200 connections, 6 juta pesan/bulan. Standard $29/bulan + usage, Pro $399/bulan + usage. Usage tercantum $2.50/juta pesan, $1/juta connection-minutes, dan $1/juta channel-minutes pada halaman pricing. [Pricing](https://ably.com/pricing) | Sedang. Fitur dan API luas, tetapi model channel/connection/usage serta retention vendor-specific. |
| **Stream Chat** | Chat service siap pakai dengan SDK client/server, WebSocket state sync, offline support. | Message history; pesan disimpan selama plan aktif menurut pricing page. Export tersedia pada paket Start. | Channel roles/permissions dan moderator roles tersedia; Go server SDK dan REST tersedia. | Push notifications tersedia. | Media attachment dan resizing tersedia; CDN storage/bandwidth dipricing terpisah pada paid plan. | Message flagging, profanity/block list, pre-send hooks, mute/ban/block, dashboard review tersedia; AI review adalah add-on/enterprise. | Webhooks tersedia. | Free: 1.000 MAU dan 100 concurrent connections. Start $399/bulan annual atau $499/bulan monthly; Elevate $599 annual atau $675 monthly. Overage dan storage/CDN memiliki tarif terpisah. [Pricing](https://getstream.io/chat/pricing/) | Tinggi. Implementasi cepat, tetapi migrasi channel/message/attachment/penanganan laporan dan ketergantungan pada billing MAU/connections cukup besar. |
| **Centrifugo OSS self-hosted** | WebSocket, SSE, HTTP-streaming, gRPC/WebTransport; SDK JavaScript dan Go; recovery dan presence. | History/recovery adalah bounded cache; database aplikasi tetap menyimpan pesan utama. PostgreSQL broker tersedia. | JWT dan channel permission; Go dapat menerbitkan token dan menentukan membership transaksi/admin. | Transport realtime saja; push notification perlu FCM/Web Push/provider lain. | Tidak menyediakan object storage attachment; gunakan PostgreSQL metadata + object storage sendiri. | Tidak menyediakan alur marketplace untuk laporan dan penanganan penyalahgunaan; buat di Go/admin panel atau service khusus. | HTTP/gRPC server API, proxy events/subscriptions, asynchronous PostgreSQL/Kafka consumers. | Software OSS Apache-2.0, sehingga tidak ada license fee yang tercantum. Biaya awal adalah hosting, Redis/PostgreSQL bila diperlukan, monitoring, backup, on-call, dan engineering. Tidak ada angka provider-hosted yang dapat dipakai tanpa memilih infrastruktur. [Repository/license](https://github.com/centrifugal/centrifugo) | Rendah-sedang secara data, lebih tinggi secara operasi. Tidak terkunci ke vendor realtime, tetapi tim memiliki deployment, scaling, patching, HA, dan incident response. |

## Kebutuhan provider-neutral

Kontrak internal sebaiknya memakai konsep berikut, sehingga provider hanya menjadi adapter realtime:

- `Conversation`: `id`, `transaction_id`, buyer, seller, status, created/closed timestamps, retention policy.
- `Participant`: user ID, role (`buyer`, `seller`, `admin`, `support`), scope transaksi, revoked timestamp.
- `Message`: immutable ID dari aplikasi, conversation ID, sender, text/system type, attachment IDs, created timestamp, review state, redaction metadata, provider event ID.
- `Attachment`: object key, media type, size, checksum, uploader, scan state, access scope, expiry/deletion state. URL harus signed/short-lived, bukan URL publik.
- `Delivery state`: sent, delivered, read, failed, provider cursor/recovery position. Delivery state bukan bukti bahwa pesan sudah tersimpan di audit log.
- `Report case`: reporter, target message, reason, evidence snapshot, reviewer, action, timestamps, appeal/status.
- `Audit event`: actor, action, resource, before/after or hash, request/correlation ID, created timestamp. Append-only dan dipisahkan dari chat history bila kebutuhan retention berbeda.
- `Outbox/integration event`: message committed, attachment scanned, report created, conversation closed. Webhook dari provider diperlakukan sebagai at-least-once dan harus idempotent.

Aturan akses minimum:

- Backend Go memeriksa bahwa user adalah buyer/seller dari `transaction_id`, atau admin/support memiliki scope yang sah.
- User tidak boleh memilih `conversation_id` lalu memperoleh akses hanya karena mengetahui ID tersebut.
- Channel name dan provider token tidak boleh menjadi satu-satunya authorization boundary.
- Pesan ditulis ke PostgreSQL sebelum dipublikasikan ke realtime; retry tidak boleh menggandakan pesan.
- Conversation yang ditutup tetap dapat dibaca sesuai kebijakan audit, tetapi pengiriman baru ditolak kecuali dibuka kembali oleh aturan bisnis.

## Estimasi bentuk biaya

Angka di bawah adalah bentuk billing dari halaman resmi yang diakses, bukan proyeksi invoice marketplace. Proyeksi final membutuhkan MAU, peak concurrent connections, pesan per percakapan, attachment GB, egress, region, retention, dan jumlah environment.

| Opsi | Komponen yang perlu dihitung | Posisi MVP |
|---|---|---|
| Supabase | Plan organisasi, compute per project, Realtime messages/connections, egress, database/storage, backup/PITR, Storage egress dan image transformation. | Free dapat dipakai untuk prototipe; Pro tercantum $25/bulan dan compute project terpisah. Spend cap tersedia pada Pro. Perlu konfirmasi kapasitas dan billing aktual untuk traffic produksi. |
| Firebase | Firestore document reads/writes/deletes, index/storage, network, Cloud Storage storage/egress/operations, Functions, FCM-related infrastructure. | Gratis tidak dapat diasumsikan untuk produksi. Pricing Firestore bersifat usage/region dependent; gunakan calculator dan konfirmasi quota sebelum memilih. |
| Pusher | Paket, pesan per hari, concurrent connections, overage/add-ons, Beams bila push dipakai. | Sandbox $0; paket berbayar pertama yang tercantum Startup $49/bulan. Perlu menguji apakah chat marketplace tetap di bawah quota harian. |
| Ably | Package + messages, connection-minutes, channel-minutes, data transfer, retention/package limits. | Free untuk proof of concept; Standard $29/bulan + usage. Usage dapat lebih sulit diprediksi karena channel dan connection minutes. |
| Stream | Plan, MAU, peak concurrent, stored messages, API calls, bandwidth, CDN attachment storage/egress, add-on penanganan laporan. | Free terbatas; paid Start minimum jauh lebih tinggi daripada transport-only options. Cocok bila penghematan engineering lebih penting daripada fixed cost. |
| Centrifugo OSS | VM/container, load balancer, TLS, Redis/PostgreSQL broker, backups, monitoring, egress, storage attachment, operator time. | Tidak ada license fee yang ditemukan pada sumber resmi; total biaya tidak boleh dianggap $0. Perlu budget operasi dan runbook HA. |

## Keputusan yang masih perlu dibuat

1. Apakah chat menjadi bagian dari database transaksi PostgreSQL yang sama, atau boleh memakai database kedua?
2. Apakah riwayat harus disimpan untuk jangka waktu tertentu setelah transaksi selesai, dan apakah ada legal hold untuk dispute/fraud?
3. Apakah attachment video benar-benar diperlukan pada MVP? Video mengubah biaya storage, transcoding, pemeriksaan malware, dan review konten.
4. Apakah admin/support dapat melihat semua percakapan atau hanya percakapan yang di-assign/escalate?
5. Apakah notifikasi hanya in-app/web push, atau wajib email/SMS/mobile push? Provider realtime tidak otomatis menutup semua channel notifikasi.
6. Apakah MVP cukup dengan laporan dan review manual, atau perlu filter kata kasar, deteksi spam, deteksi penipuan/transaksi di luar platform, serta pemeriksaan foto/video?
7. Target region/data residency, SLA, RPO/RTO, export format, dan exit plan sebelum kontrak provider.
8. Batas volume awal: MAU, concurrent connections, pesan per hari/bulan, rata-rata attachment size, egress, dan jumlah environment.

## Risiko keamanan, privasi, dan penyalahgunaan

- **Broken object-level authorization:** channel privat tidak menggantikan pemeriksaan membership transaksi di Go. Uji buyer, seller, admin, support, revoked user, closed transaction, dan IDOR.
- **Data leakage melalui attachment:** gunakan private bucket/object, signed URL pendek, content-type/size allowlist, pemeriksaan malware, review foto/video, EXIF stripping bila relevan, dan deletion job yang dapat diaudit.
- **Token leakage:** token realtime dibuat oleh backend, berumur pendek, scope channel terbatas, dan dicabut saat user diblokir atau transaksi kehilangan akses. Centrifugo secara eksplisit mendukung expiry dan menyarankan expiry pendek; Supabase juga mengingatkan agar JWT expiration window pendek. [Centrifugo auth](https://centrifugal.dev/docs/server/authentication), [Supabase authorization](https://supabase.com/docs/guides/realtime/authorization)
- **Webhook replay/forgery:** verifikasi signature bila provider mendukung, simpan event ID, lakukan idempotency, batasi source, dan jangan mengubah status transaksi hanya dari event realtime yang belum diverifikasi.
- **Bukti laporan:** jangan menghapus pesan yang sudah dilaporkan tanpa menyimpan snapshot/hash, actor, keputusan reviewer, dan alasan. Redaction untuk user view harus berbeda dari penghapusan bukti audit.
- **Privacy:** minimalkan data yang dibawa dalam presence dan notification payload. Jangan mengirim alamat COD, nomor telepon, atau data sensitif ke channel yang tidak perlu.
- **Perbedaan masa simpan:** retention realtime vendor sering lebih pendek atau berbeda dari retention audit marketplace. Jadikan PostgreSQL/object storage tempat penyimpanan utama dan dokumentasikan deletion/export procedure.
- **Abuse controls:** rate limit pesan, attachment, report, reconnect, dan token minting; deteksi spam/off-platform payment; block/mute; quarantine attachment sebelum dipublikasikan.
- **Availability:** desain retry, deduplication, reconnect/recovery, backpressure, dan fallback load dari PostgreSQL. Provider history/cache tidak boleh dianggap sebagai satu-satunya backup.

## Sumber resmi

Semua URL berikut adalah dokumentasi, repository, atau pricing page resmi provider/proyek. Diakses 2026-09-24.

- Supabase: [Realtime overview](https://supabase.com/docs/guides/realtime), [Realtime authorization](https://supabase.com/docs/guides/realtime/authorization), [Broadcast and replay](https://supabase.com/docs/guides/realtime/broadcast), [Storage](https://supabase.com/docs/guides/storage), [pricing](https://supabase.com/pricing)
- Firebase: [Firestore realtime listeners](https://firebase.google.com/docs/firestore/query-data/listen), [Firestore Security Rules](https://firebase.google.com/docs/firestore/security/get-started), [Firestore pricing](https://firebase.google.com/docs/firestore/pricing), [Cloud Storage upload](https://firebase.google.com/docs/storage/web/upload-files), [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- Pusher: [Channels overview](https://pusher.com/docs/channels/), [private/presence/auth/webhooks navigation](https://pusher.com/docs/channels/), [Channels pricing](https://pusher.com/channels/pricing/)
- Ably: [Documentation](https://ably.com/docs), [pricing and feature matrix](https://ably.com/pricing)
- Stream: [Chat documentation](https://getstream.io/chat/docs/), [Chat pricing and feature matrix](https://getstream.io/chat/pricing/)
- Centrifugo: [GitHub repository and README](https://github.com/centrifugal/centrifugo), [JWT authentication](https://centrifugal.dev/docs/server/authentication), [history and recovery](https://centrifugal.dev/docs/server/history_and_recovery), [channel permissions](https://centrifugal.dev/docs/server/channel_permissions)

Harga, quota, feature availability, retention, dan plan dapat berubah. Semua angka yang dikutip perlu dikonfirmasi kembali dengan vendor saat procurement, khususnya untuk Firebase usage/region, Pusher add-ons, Ably usage, Stream overage/add-ons, dan biaya infrastruktur Centrifugo.

## Riset lanjutan: opsi gratis dan build sendiri

Bagian ini ditambahkan pada 2026-09-24. Angka dan status fitur di bawah hanya merujuk pada halaman resmi yang ditautkan dan bukan janji kuota untuk selamanya.

### Hosted gratis atau low-cost

#### Supabase Free

Halaman pricing resmi mencantumkan paket Free seharga $0 dengan 500 MB database per project, 5 GB egress, 1 GB file storage, 200 peak concurrent Realtime connections, 2 juta pesan Realtime per bulan, dan batas ukuran pesan 256 KB. Free project dapat dijeda setelah satu minggu tidak aktif. Pro tercantum mulai $25/bulan; compute project ditagih terpisah, walaupun paket Pro dan Team memiliki kredit compute $10/bulan. [Supabase pricing](https://supabase.com/pricing)

Untuk chat marketplace, Supabase paling cocok bila PostgreSQL tetap menjadi database utama dan Realtime hanya menjadi jalur distribusi. Batas Free yang paling mudah tercapai biasanya egress, storage attachment, koneksi puncak, atau jumlah pesan, bukan hanya jumlah percakapan. Angka ini tidak mencakup biaya layanan eksternal untuk push notification, scanning attachment, atau hosting aplikasi.

#### Firebase Spark

Firebase menyebut Spark sebagai paket tanpa biaya dan tanpa payment information untuk memulai. Firestore tetap memiliki kuota tanpa biaya, termasuk contoh resmi 50.000 document reads dan 20.000 document writes per hari pada Spark. Jika kuota tanpa biaya produk terlampaui, penggunaan produk tersebut dihentikan sampai siklus berikutnya atau project di-upgrade ke Blaze. Menghubungkan Cloud Billing account juga otomatis meng-upgrade project ke Blaze. [Firebase pricing plans](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans), [Firestore pricing](https://firebase.google.com/docs/firestore/pricing)

FCM tercantum sebagai produk Firebase tanpa biaya, tetapi Firestore, Cloud Storage, Cloud Functions, dan Realtime Database termasuk layanan yang memiliki paid tier atau kuota produk. Karena itu, Spark cocok untuk prototipe dengan traffic kecil dan toleransi terhadap penghentian kuota; ia kurang cocok sebagai satu-satunya fondasi produksi tanpa alarm usage, desain quota, dan rencana Blaze. [Firebase pricing plans](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans), [Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)

#### Pusher Channels Sandbox

Pricing resmi Pusher mencantumkan Sandbox seharga $0 dengan 200.000 pesan per hari dan 100 concurrent connections. Paket Startup tercantum $49/bulan dengan 1 juta pesan per hari dan 500 concurrent connections. Channels adalah transport publish/subscribe; riwayat chat durable, attachment, laporan pengguna, dan audit tetap harus dimiliki aplikasi. [Pusher Channels pricing](https://pusher.com/channels/pricing/), [Pusher Channels documentation](https://pusher.com/docs/channels/)

Sandbox masuk akal untuk demo atau proof of concept. Kuota harian perlu dipantau sebelum dipakai untuk traffic pengguna nyata.

#### Ably Free

Pricing resmi Ably mencantumkan Free seharga $0 dengan 200 concurrent connections, 500 messages/second, 6 juta pesan per bulan, retensi pesan maksimum 1 hari, dan ukuran pesan maksimum 64 KiB. Pricing yang sama mencantumkan Standard mulai $29/bulan ditambah usage, dengan tarif yang ditampilkan $2,50 per juta pesan, $1 per juta connection-minutes, dan $1 per juta channel-minutes. [Ably pricing](https://ably.com/pricing)

Ably menyediakan presence, message history/rewind, auth token/JWT, recovery, push notification, dan fitur Ably Chat pada feature matrix. Namun retensi Free 1 hari tetap terlalu pendek untuk audit transaksi; pesan final sebaiknya ditulis ke PostgreSQL sebelum dipublikasikan. Feature matrix juga menandai message flagging serta mute/ban sebagai `Coming soon`, sehingga alur laporan dan penanganan penyalahgunaan marketplace tidak boleh diasumsikan sudah lengkap. [Ably pricing](https://ably.com/pricing), [Ably documentation](https://ably.com/docs)

### Open-source dan self-hosted

#### Centrifugo OSS

Centrifugo adalah server publish/subscribe open-source berlisensi Apache-2.0 yang mendukung WebSocket, HTTP-streaming, SSE, gRPC, dan WebTransport. README resminya juga mencantumkan JWT atau proxy authentication, strategi channel permission, presence, history dengan recovery saat reconnect, PostgreSQL/Redis/NATS untuk skalabilitas, HTTP/gRPC API, consumer PostgreSQL/Kafka untuk pola outbox/CDC, metrik Prometheus, dan dashboard Grafana resmi. [Centrifugo repository](https://github.com/centrifugal/centrifugo), [Centrifugo documentation](https://centrifugal.dev/docs)

Centrifugo mengurangi pekerjaan transport dan fan-out, tetapi tidak menghilangkan pekerjaan aplikasi. History Centrifugo adalah history realtime/cache yang dibatasi, bukan arsip compliance. PostgreSQL tetap perlu menyimpan pesan immutable, status peninjauan, attachment metadata, audit event, dan retention policy. Biaya lisensi software yang tercantum adalah $0; biaya VM/container, load balancer, TLS, broker, backup, egress, monitoring, dan waktu operator tidak menjadi $0.

#### Centrifuge Go library dan `centrifuge-go`

Repository `github.com/centrifugal/centrifuge` adalah library Go untuk membangun server realtime berbasis Centrifuge. Jika tim membutuhkan server embedded di proses Go sendiri, library ini adalah jalur yang lebih dalam dan memberi tanggung jawab lebih besar atas lifecycle, scaling, dan integrasi backend. [Centrifuge repository](https://github.com/centrifugal/centrifuge)

`github.com/centrifugal/centrifuge-go` adalah Go client SDK untuk berkomunikasi dengan server Centrifugo atau server berbasis Centrifuge melalui WebSocket. README resminya mencantumkan reconnect, subscription state, dan message recovery sebagai bagian dari perilaku SDK, tetapi juga memperingatkan bahwa callback dijalankan sinkron dan handler yang blocking dapat memblokir read loop atau menyebabkan deadlock. [centrifuge-go repository](https://github.com/centrifugal/centrifuge-go)

Untuk Project-4.0, Centrifugo lebih sederhana daripada menanam Centrifuge ke service Go utama: backend tetap memiliki aturan transaksi, sementara Centrifugo memiliki koneksi dan fan-out. Centrifuge embedded baru masuk akal jika tim memang membutuhkan kontrol penuh atas server realtime dan siap memelihara lapisan tersebut.

#### WebSocket langsung dengan Go

Library `github.com/coder/websocket` adalah pilihan minimal untuk membangun transport WebSocket sendiri. Repository resminya menyebut bahwa Coder sekarang memelihara project ini, dengan dukungan `context.Context`, zero dependencies, concurrent writes, close handshake, ping/pong, compression, dan helper JSON. Library ini hanya memberi primitive koneksi; semua perilaku chat harus dibuat aplikasi. [coder/websocket repository](https://github.com/coder/websocket)

`golang.org/x/net/websocket` masih memiliki dokumentasi dan rilis module, tetapi dokumentasi package resminya menyatakan package itu deprecated dan menyebut `github.com/coder/websocket` serta `github.com/gorilla/websocket` sebagai alternatif yang lebih aktif dipelihara. Karena itu, `x/net/websocket` tidak direkomendasikan untuk implementasi baru. [Go package documentation](https://pkg.go.dev/golang.org/x/net/websocket)

Go standard library menyediakan HTTP server/client, tetapi bukan implementasi lengkap WebSocket. [Paket `net/http`](https://pkg.go.dev/net/http) mendokumentasikan HTTP server dan client, sedangkan implementasi WebSocket perlu library tambahan seperti `coder/websocket`. Menggunakan library WebSocket tetap lebih kecil risikonya daripada menulis handshake, frame parsing, masking, ping/pong, close handshake, dan batas payload sendiri. Risiko utama build langsung bukan dependensi, melainkan protokol aplikasi dan operasi yang harus dimiliki tim.

#### PostgreSQL `LISTEN`/`NOTIFY`

PostgreSQL menyediakan `LISTEN` untuk mendaftarkan session pada channel dan `NOTIFY` untuk mengirim event ke listener. Dokumentasi resmi menjelaskan bahwa notification dari transaksi baru dikirim setelah commit, payload default harus lebih pendek dari 8.000 byte, dan payload besar atau data binary sebaiknya disimpan di tabel lalu notification hanya membawa key. Dokumentasi juga menjelaskan bahwa notification queue yang penuh dapat membuat transaksi `NOTIFY` gagal saat commit. [LISTEN](https://www.postgresql.org/docs/current/sql-listen.html), [NOTIFY](https://www.postgresql.org/docs/current/sql-notify.html)

Pola yang relevan adalah: commit `chat_message` dan outbox/event marker di transaksi PostgreSQL, kirim `NOTIFY` berisi ID event, lalu worker atau service realtime membaca row tersebut dan melakukan fan-out. `LISTEN`/`NOTIFY` cocok sebagai invalidation atau wake-up signal pada deployment kecil. Ia bukan message broker, bukan durable queue, dan tidak menggantikan reconnect recovery, retry, atau delivery state.

### Minimum build sendiri

Transport WebSocket hanya menyelesaikan koneksi dan frame. Versi minimum yang masih layak untuk chat transaksi harus memiliki komponen berikut:

| Komponen | Minimum yang harus dibuat | Risiko bila dilewati |
|---|---|---|
| Auth dan channel authorization | Token sesi, verifikasi user, membership berdasarkan `transaction_id`, role buyer/seller/admin, revoke, expiry, dan pemeriksaan server-side pada setiap operasi sensitif. | IDOR, user melihat chat transaksi lain, token lama tetap berlaku. |
| Persistence dan ordering | Tabel conversation/message, immutable application message ID, server timestamp, constraint/idempotency key, urutan yang dapat dipulihkan, outbox, dan transaksi commit sebelum publish. | Pesan hilang, duplikat, urutan berbeda antar device, audit tidak lengkap. |
| Presence, delivery, dan read state | Heartbeat/TTL presence, sent/delivered/read yang dibedakan, cursor per participant, dan aturan bahwa read bukan bukti audit. | Status online salah, unread count salah, atau pesan dianggap terbaca tanpa bukti. |
| Reconnect, deduplication, dan backpressure | Exponential backoff, cursor recovery dari database, dedup berdasarkan message ID, batas queue per connection, timeout, dan close policy. | Reconnect menggandakan pesan, koneksi macet menghabiskan memory, atau outage kecil menjadi kehilangan data. |
| Attachment dan notification | Private object storage, signed URL, ukuran/tipe allowlist, malware scan, metadata di database, deletion job, serta FCM/Web Push/email sesuai kebutuhan. | Kebocoran file, biaya egress tidak terkendali, atau user tidak tahu ada pesan baru. |
| Laporan, pemblokiran, retention, dan monitoring | Report/block/mute, antrean review, redaction versus audit evidence, retention/legal hold, structured logs, metrics, tracing, alert, backup, restore test, dan runbook incident. | Penyalahgunaan tidak tertangani, bukti sengketa hilang, serta masalah produksi terlambat diketahui. |

Biaya build sendiri terdiri dari tiga lapisan. Pertama, biaya engineering untuk protocol, schema, retry, tests, security review, admin review, dan mobile/web notification. Kedua, biaya infrastruktur untuk app server, PostgreSQL, object storage, egress, TLS/load balancer, backup, dan observability. Ketiga, biaya operasi untuk patching, on-call, capacity planning, restore drill, incident response, dan pemeliharaan provider notification. Tidak ada angka total yang jujur tanpa volume, region, availability target, ukuran attachment, dan jam on-call.

Risikonya juga berbeda dari hosted. Hosted memindahkan sebagian risiko koneksi, scaling, dan quota ke vendor, tetapi menambah lock-in, perubahan pricing, outage vendor, dan batas export. Build sendiri mengurangi ketergantungan realtime vendor, tetapi tim bertanggung jawab penuh atas kehilangan pesan, abuse, security patch, HA, backup, dan pemulihan saat incident. "Gratis" pada software atau free tier tidak berarti tanpa biaya hosting, egress, storage, backup, engineering, atau operasional.

### Rekomendasi bertingkat

1. **Prototipe gratis:** gunakan Supabase Free jika PostgreSQL sudah menjadi stack utama. Simpan pesan di PostgreSQL, batasi attachment, gunakan in-app realtime dahulu, dan pasang metrik pemakaian. Firebase Spark atau Ably Free dapat dipakai untuk spike terpisah bila kebutuhan utamanya adalah menguji pengalaman realtime, dengan batas kuota yang sudah disebutkan di atas.
2. **MVP gratis dengan risiko yang wajar:** tetap gunakan PostgreSQL sebagai database utama dan pilih Centrifugo self-hosted sebagai transport utama. Gunakan Supabase Free bila kecepatan prototipe lebih penting daripada kontrol operasi. Implementasikan auth channel, outbox, idempotency, reconnect recovery, report manual, private attachment, dan monitoring sebelum membuka chat ke pengguna nyata.
3. **Provider berbayar:** gunakan ketika peak connections, pesan, egress, attachment, SLA, support, atau waktu engineering sudah lebih mahal daripada biaya provider. Pilih layanan chat lengkap seperti Stream bila kebutuhan laporan, read receipt, attachment, push, dan history harus tersedia cepat. Pilih Pusher atau Ably ketika yang dibutuhkan terutama transport realtime dan data/model chat tetap dikelola sendiri. Kunci keputusan dengan volume terukur, export test, quota alert, dan exit plan.

### Sumber tambahan resmi

Semua sumber tambahan di bagian ini diakses 2026-09-24.

- Supabase: [pricing](https://supabase.com/pricing)
- Firebase: [pricing plans](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans), [Firestore pricing](https://firebase.google.com/docs/firestore/pricing), [Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- Pusher: [Channels pricing](https://pusher.com/channels/pricing/), [Channels documentation](https://pusher.com/docs/channels/)
- Ably: [pricing](https://ably.com/pricing), [documentation](https://ably.com/docs)
- Centrifugal: [Centrifugo](https://github.com/centrifugal/centrifugo), [Centrifuge](https://github.com/centrifugal/centrifuge), [centrifuge-go](https://github.com/centrifugal/centrifuge-go)
- Go WebSocket: [coder/websocket](https://github.com/coder/websocket), [x/net/websocket status and documentation](https://pkg.go.dev/golang.org/x/net/websocket)
- PostgreSQL: [LISTEN](https://www.postgresql.org/docs/current/sql-listen.html), [NOTIFY](https://www.postgresql.org/docs/current/sql-notify.html)

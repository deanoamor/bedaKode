# PRD: bedaKode — MVP 1 (Bedah Repo)

> Webmu dibuat AI dan sudah jalan, tapi kamu belum paham kodenya? Tempel link GitHub-nya. Kami tunjukkan alur kerjanya, dari halaman ke route, controller, sampai database, dan menjelaskan setiap bagiannya dengan bahasa yang mudah.

MVP 1 dibagi dua bagian:

| Bagian | Isi | Biaya AI | Perkiraan waktu |
|---|---|---|---|
| **Part 1: Bedah Statis** | Analisis kode tanpa AI: route, alur panggilan, pola yang dikenali, penjelasan template, dampak perubahan | Tidak ada | ± 3 minggu |
| **Part 2: Narasi AI** | AI menarasikan fakta dari Part 1: "ini apa", "kenapa begini", "aman diubah?", cari fitur dengan bahasa bebas | Ada (Claude API) | ± 1–2 minggu |

Fitur "klik elemen di web live" (screenshot) dipindah ke MVP 2.

---

## 1. Ringkasan
- **Masalah:** Vibe coder (pengguna Lovable, Bolt, v0, Cursor) punya web yang jalan, tapi tidak paham kodenya. Mau ubah sedikit saja takut rusak, dan bingung harus mulai baca dari file mana.
- **Solusi:** User menempel URL repo GitHub (public). Sistem mengurai kode dan membangun **peta alur**: route mana memanggil handler mana, komponen mana merender apa, form mana mengirim ke endpoint mana, dan file mana menyentuh database. Setiap pola yang dikenali dijelaskan dengan bahasa awam, dan ditautkan ke docs resmi sesuai versi.
- **Target user:** Pemula non-programmer atau semi-programmer yang membangun web dengan bantuan AI.
- **Prinsip produk:**
  - **Fakta dulu, narasi kemudian.** Alur dan relasi diambil dari kode (parser), bukan tebakan. AI (Part 2) hanya menarasikan fakta tersebut, jadi risiko halusinasi kecil.
  - **Selalu ada isi.** Setiap file yang dibuka minimal menampilkan perannya, posisinya di alur, siapa yang memanggil dan dipanggil, dan istilah yang relevan.
- **Pembeda dari ChatGPT/Cursor:**
  - peta alur otomatis yang bisa diklik,
  - dampak perubahan ("file ini dipakai oleh …"),
  - Bahasa Indonesia yang awam,
  - link docs resmi yang dikurasi sesuai versi,
  - tier statis gratis tanpa biaya AI,
  - hasil bisa dibagikan lewat link.
- **Model bisnis (nanti):** Statis gratis. Narasi AI memakai kuota harian, lalu nanti berbayar untuk kuota lebih, repo private, dan repo besar.

## 2. Keputusan yang sudah diambil
| Topik | Keputusan |
|---|---|
| Nama/domain | bedaKode, `bedakode.com` atau `bedakode.id` (cek ketersediaan) |
| Bahasa penjelasan | Indonesia. Struktur data siap untuk multi-bahasa |
| Level penjelasan | Dua level: **pemula** dan **teknis** |
| Stack dukungan penuh | **Laravel (Blade)** dan **Next.js** (App Router + Pages Router) |
| Stack dukungan dasar | React/Vite dan HTML biasa: graf import dan pola dasar saja |
| Parser | PHP: `nikic/php-parser`. JS/TS/JSX/TSX: skrip Node memakai TypeScript Compiler API. Blade: regex direktif |
| Model AI (Part 2) | Claude Haiku 5.5 untuk cari fitur, Claude Sonnet 5.5 untuk narasi. Bisa naik ke Claude Opus 5.5 kalau hasil uji kualitasnya kurang |
| Biaya AI (Part 2) | Ditagih lewat **Claude API** (Claude Console, bayar per token), **terpisah dari langganan Claude Pro**. Target < $0,20 per analisis, batas global $5/hari, plus spend limit di Console |
| Hosting | VPS 2 vCPU / 4 GB RAM, region Singapura atau lokal, plus Cloudflare. Butuh PHP 8.3, Node 20, MySQL 8. Tidak butuh Chromium |
| Kurasi awal | ±40 aturan pola, ±40 konsep docs, ±40 istilah glosarium |

## 3. Tujuan dan non-tujuan
**Tujuan Part 1**
1. User tanpa login bisa menganalisis repo public dan mendapat viewer interaktif dalam < 1 menit untuk repo kecil.
2. Peta proyek statis: stack, fungsi folder, titik masuk, daftar route, dan urutan baca yang disarankan.
3. Untuk Laravel dan Next.js: memilih sebuah route menampilkan **diagram alur** dari route ke handler, lalu ke panggilan berikutnya (validasi, auth, database, redirect, response).
4. Setiap file menampilkan perannya, simbol di dalamnya, relasi, pola yang dikenali beserta penjelasan template, dan dampak perubahan.
5. Cari fitur lewat kata kunci, sinonim, nama route, dan nama file.
6. Hasil bisa dibagikan lewat link.

**Tujuan Part 2**
1. Tombol "Jelaskan lebih dalam" di file, blok baris, dan route. AI menulis "ini apa", "cara kerja", "kenapa begini", dan "aman diubah?" berdasarkan fakta statis.
2. Peta proyek dinarasikan AI.
3. Cari fitur dengan bahasa bebas.
4. Biaya terkendali lewat cache, kuota, dan batas global.

**Non-tujuan MVP 1**
- URL web live, screenshot, klik elemen (MVP 2).
- Repo private, login, pembayaran.
- Mengedit kode atau membuat PR.
- Analisis runtime (menjalankan kode). Semuanya statis.
- Resolusi sempurna untuk kode dinamis (route dari variabel, DI kompleks, event/listener). Bagian yang tidak bisa dilacak ditandai "tidak terlacak otomatis".

## 4. User story
| # | Part | Sebagai | Aku ingin | Supaya |
|---|---|---|---|---|
| 1 | 1 | Vibe coder | menempel URL repo lalu klik Analisis | tahu isi proyekku tanpa daftar |
| 2 | 1 | Vibe coder | melihat ringkasan proyek dan fungsi tiap folder | punya gambaran besar |
| 3 | 1 | Vibe coder | melihat daftar halaman dan endpoint (route) | tahu apa saja yang ada di webku |
| 4 | 1 | Vibe coder | klik sebuah route dan melihat diagram alurnya | paham perjalanan data dari klik sampai database |
| 5 | 1 | Vibe coder | membuka file dan tahu perannya | tahu file ini fungsinya apa |
| 6 | 1 | Vibe coder | melihat pola yang dikenali dengan bahasa awam | paham bagian kode yang umum |
| 7 | 1 | Vibe coder | tahu file mana yang memakai file ini | tahu dampaknya kalau kuubah |
| 8 | 1 | Vibe coder | mencari "login" dan langsung ke file terkait | tidak tersesat |
| 9 | 1 | Vibe coder | melihat istilah dijelaskan dan link docs resmi | bisa belajar lebih dalam |
| 10 | 1 | Vibe coder | memilih level penjelasan (pemula/teknis) | sesuai kemampuanku |
| 11 | 1 | Vibe coder | membagikan hasil lewat link | menunjukkan ke teman atau mentor |
| 12 | 1 | Vibe coder | memberi nilai 👍/👎 | penjelasan makin bagus |
| 13 | 2 | Vibe coder | memblok baris dan klik "Jelaskan lebih dalam" | paham logika khusus di kodeku |
| 14 | 2 | Vibe coder | tahu kenapa kode ditulis begitu | belajar pola yang baik |
| 15 | 2 | Vibe coder | bertanya "bagian bayar ada di mana?" | menemukan kode dengan bahasaku sendiri |

---

# PART 1: BEDAH STATIS

## 5. Fitur Part 1

### 5.1 Form submit
- Satu input: URL GitHub (`https://github.com/{owner}/{repo}`, opsional `/tree/{branch}`).
- Cloudflare Turnstile dan rate limit per IP.
- Kalau analisis untuk commit yang sama sudah ada dan belum kedaluwarsa, langsung arahkan ke hasil yang ada.

### 5.2 Pipeline analisis (queue)
1. **Validasi:** hanya host `github.com`. Cek repo public lewat GitHub API (pakai token server). Ambil branch dan commit SHA.
2. **Fetch:** download zip (codeload). Ekstrak dengan aman:
   - tolak path traversal dan symlink,
   - batasi ukuran setelah ekstrak dan jumlah entri.
3. **Filter:** buang `node_modules`, `vendor`, `.git`, `dist`, `build`, `.next`, `out`, `coverage`, binary, gambar, font, dan file > 1 MB. **File sensitif** (`.env*`, `*.pem`, `*.key`, `id_rsa*`) tidak disimpan dan tidak ditampilkan.
4. **Deteksi stack dan versi** dari `package.json`, `composer.json`, dan lockfile (lockfile hanya dibaca, tidak diindeks).
5. **Indeks file:** isi teks dan full-text.
6. **Parse (tanpa AI):**
   - **PHP** (`nikic/php-parser`): class, method, function, `use`, panggilan statis dan method (`Auth::attempt`, `$request->validate`, `Model::create`), definisi route.
   - **JS/TS/JSX/TSX** (skrip Node, TypeScript Compiler API, output JSON): import/export, fungsi, komponen (fungsi yang mengembalikan JSX), hooks, render komponen, `fetch`/`axios` dengan URL literal, `onSubmit`/`onClick`, server actions, `redirect`/`router.push`.
   - **Blade** (regex): `@extends`, `@include`, `@section`, `<x-komponen>`, `<form action=…>`, `{{ route('…') }}`.
7. **Bangun graf:**
   - `symbols`: fungsi, class, komponen, dan sejenisnya.
   - `file_relations`: import, include, extends, component.
   - `call_edges`: call, render, fetch, form_submit, redirect.
   - `routes`:
     - Laravel: `Route::get/post/resource/…` → controller@method atau closure.
     - Next.js App Router: `app/**/page.tsx` dan `app/**/route.ts` (method dari nama ekspor `GET`/`POST`/…).
     - Next.js Pages Router: `pages/**` dan `pages/api/**`.
   - **Penyambungan lintas batas:** URL literal di `fetch('/api/x')`, `<form action="/login">`, atau `route('login')` dicocokkan ke tabel `routes`.
8. **Cocokkan aturan pola** (lihat 5.3) dan simpan ke `pattern_matches`.
9. **Tentukan peran file** dari konvensi path dan isi. Contoh peran: Controller, Model, Migration, Route file, Blade layout, Halaman Next.js, API route, Komponen React, Hook, Config.
10. **Buat peta proyek statis** (lihat 5.4).
11. Status menjadi `ready`.

Kalau parser gagal pada satu file, file itu ditandai `parse_status=failed` dan pipeline tetap jalan.

### 5.3 Aturan pola (pattern rules)
- Aturan disimpan sebagai file di repo (`resources/rules/{framework}/*.php`), di-versikan bersama kode dan di-cache saat runtime.
- Setiap aturan berisi:
  - `key`, misalnya `laravel.request.validate`,
  - `framework` dan `language`,
  - **matcher**: jenis node (call, static_call, import, jsx_attr, blade_directive, path) dan polanya,
  - **capture** (opsional): misalnya daftar field yang divalidasi,
  - **template teks** pemula dan teknis, dengan placeholder dari capture (contoh: "Mengecek input `{fields}` sebelum diproses."),
  - **tag konsep** untuk link docs,
  - **kategori alur** (input, validasi, auth, database, response, navigasi, state, efek samping) yang dipakai untuk ikon dan warna di diagram.
- Target awal: **±40 aturan** (±20 Laravel, ±15 Next.js/React, ±5 umum).
  - Contoh Laravel: `validate`, `Auth::attempt`, `Auth::user`, `middleware('auth')`, Eloquent `create/update/delete/where`, `DB::table`, `redirect()`, `back()->with()`, `view()`, `Mail::send`, `Storage::put`, `Http::get`.
  - Contoh Next.js/React: `useState`, `useEffect`, `useRouter`/`router.push`, `fetch`, `"use client"`, `"use server"`/server actions, `generateMetadata`, `NextResponse.json`, `cookies()`, akses `process.env`.

### 5.4 Peta proyek statis
Dibuat sepenuhnya dari fakta:
- **Ringkasan:** template berdasarkan stack. Contoh: "Proyek Laravel 11 dengan Blade dan Tailwind. Ada 8 halaman, 3 endpoint form, 5 tabel database."
- **Statistik:** jumlah file, route, komponen, model, dan migration.
- **Fungsi folder:** dari kamus konvensi per framework (`app/Http/Controllers` → "Pengatur logika setiap halaman", dst.). Folder di luar kamus ditampilkan sebagai "folder buatan proyek" beserta jumlah file dan peran dominannya.
- **Titik masuk:** `routes/web.php`, `app/layout.tsx`, `src/main.tsx`, `index.html`, dll.
- **Daftar route:** method, URI, dan handler (bisa diklik).
- **Urutan baca:** titik masuk → route file → handler dengan relasi terbanyak → komponen/layout bersama → model. Maksimal 10 file, masing-masing dengan alasan template.

### 5.5 Alur per route
- User memilih route, lalu sistem menelusuri `call_edges` dari handler (kedalaman maks. 6, hanya simbol di dalam repo).
- Setiap langkah diberi anotasi pola yang dikenali dan kategori alurnya.
- Hasilnya ditampilkan sebagai:
  - **diagram Mermaid** (bisa diklik ke file dan baris),
  - **daftar langkah bernomor** dengan penjelasan template,
  - **catatan "tidak terlacak"** untuk panggilan yang tidak bisa diselesaikan (dinamis atau dari paket luar).
- Untuk Next.js, alur juga bisa dimulai dari halaman: `page.tsx` → komponen → form/fetch → API route → handler.

### 5.6 Panel file
Saat file dibuka, panel kanan menampilkan:
1. **Peran file** beserta penjelasan template perannya.
2. **Posisi di alur:** route yang melewati file ini.
3. **Isi file:** daftar simbol. Klik simbol akan lompat ke barisnya.
4. **Pola yang dikenali:** per rentang baris, dengan teks template sesuai level. Saat mengklik baris kode, pola di baris tersebut disorot.
5. **Relasi:** dipanggil oleh dan memanggil.
6. **Dampak perubahan:** jumlah dan daftar file yang bergantung (graf terbalik). Ada peringatan khusus untuk peran sensitif (migration, route file, layout, config).
7. **Istilah dan docs:** tooltip glosarium dan link docs dari tag konsep.

Semua data ini disusun sebagai **"lembar fakta" JSON** per file dan per route. Lembar fakta ini nantinya jadi input prompt di Part 2.

### 5.7 Cari fitur (statis)
- Input kata kunci, misalnya "login".
- Diperluas dengan **kamus sinonim kurasi** (login → auth, signin, session, attempt; bayar → payment, checkout, order, midtrans, xendit, stripe).
- Hasil diurutkan dari: nama route/URI → nama file/simbol → pola yang cocok (misalnya `Auth::attempt` untuk "login") → full-text isi.
- Maksimal 10 hasil, masing-masing dengan alasan ("route `POST /login`", "memanggil `Auth::attempt`").
- Query dan klik dicatat di `search_logs` untuk metrik dan desain Part 2.

### 5.8 Viewer
Tata letak tiga panel di desktop. Di mobile berubah jadi tab.
- **Kiri:** file tree (ikon peran, penanda titik masuk) dan tab **Routes**.
- **Tengah:** kode dengan syntax highlight, nomor baris, dan sorotan pola.
- **Kanan:** Peta Proyek (default), Panel File, atau Alur Route. Ada slot "Penjelasan AI" yang disembunyikan sampai Part 2.
- **Atas:** kolom cari fitur, pilihan level (Pemula/Teknis), tombol Share, dan info "Berdasarkan commit `abc1234`".

### 5.9 Glosarium, docs, share, feedback
- **Glosarium:** ±40 istilah, ditampilkan sebagai tooltip otomatis di teks penjelasan.
- **Docs:** tag konsep diambil dari aturan pola, URL dari tabel kurasi `concept_links` sesuai versi.
- **Share:** `/r/{owner}/{repo}/{sha7}`, read-only, tanpa login.
- **Feedback:** 👍/👎 pada panel file, alur route, peta proyek, dan hasil cari.

## 6. Aturan penting Part 1
- **Batas repo:** maks 50 MB zip, 200 MB setelah diekstrak, 2.000 file teks, 1 MB per file. Waktu parse maks 3 menit per analisis.
- **Batas pemakaian:** 5 analisis baru per IP per hari (diatur lewat config). Membuka hasil dan link share tidak dibatasi.
- **Keamanan:**
  - Hanya `github.com`.
  - Ekstraksi zip aman.
  - File sensitif tidak disimpan.
  - **Kode repo tidak pernah dijalankan,** hanya diurai. Skrip parser Node berjalan dengan batas waktu dan memori, tanpa akses jaringan.
  - Semua isi dari repo di-escape saat dirender. Mermaid memakai `securityLevel: 'strict'`.
  - IP disimpan sebagai hash + salt.
- **Cache:** kunci `(owner, repo, commit_sha)`, dengan owner dan repo dinormalisasi ke huruf kecil.
- **Retensi:** zip dihapus setelah diproses. Hasil analisis disimpan 30 hari, lalu isi file dan data graf dihapus oleh scheduled command.

## 7. Metrik sukses Part 1
- Analisis berhasil ≥ 90%.
- File berhasil di-parse ≥ 95% (untuk bahasa yang didukung).
- **Cakupan pola ≥ 60%:** persentase file (Laravel/Next.js, di luar config dan aset) yang punya minimal 1 pola dikenali.
- Route berhasil disambungkan ke handler ≥ 80% (Laravel/Next.js).
- Rata-rata file atau route yang dibuka per analisis ≥ 5.
- Pencarian yang diikuti klik hasil ≥ 50%.
- Feedback 👍 ≥ 70%.
- Link share dibuka oleh IP lain selain pembuatnya.

## 8. Syarat lanjut ke Part 2
Lanjut ke Part 2 kalau:
- ada ≥ 100 analisis dari user nyata,
- cakupan pola ≥ 60%, dan
- feedback atau pencarian menunjukkan user butuh penjelasan logika khusus atau "kenapa", yang tidak bisa dijawab template.

---

# PART 2: NARASI AI

## 9. Fitur Part 2

### 9.1 Jelaskan lebih dalam
- Tombol ini muncul di panel file, di seleksi baris (maks. 200 baris), dan di alur route.
- Alurnya:
  1. Cek cache.
  2. Susun prompt berisi **lembar fakta statis** (peran, pola, relasi, route), potongan kode, dan stack beserta versinya.
  3. Claude Sonnet 5.5 menulis JSON terstruktur (lihat 9.5).
  4. Validasi hasilnya, lalu simpan.
- Output AI hanya boleh menyebut file yang ada di `repo_files`. Path lain dibuang.

### 9.2 Peta proyek AI
- Narasi singkat proyek berdasarkan peta statis.
- Dibuat saat diminta (tombol "Ceritakan proyek ini"), lalu di-cache per level.

### 9.3 Cari fitur dengan bahasa bebas
- Kandidat diambil dari pencarian statis (5.7), maksimal 20.
- Claude Haiku 5.5 memilih maksimal 5 lokasi dan memberi alasan.
- Hasil di-cache per (analisis, hash pertanyaan, level).

### 9.4 Kontrol biaya
- **Kuota:** 20 permintaan AI per IP per hari. Hasil dari cache tidak dihitung.
- **Batas global:** $5/hari, dihitung dari `ai_usage_logs`. Kalau terlewati, tombol AI dinonaktifkan sementara dengan pesan ramah. Fitur statis tetap jalan.
- **Prompt caching** untuk instruksi sistem yang sama di setiap panggilan.
- **Effort rendah atau sedang** untuk tugas menjelaskan, supaya token hemat.
- **Spend limit di Claude Console** sebagai pengaman terakhir.
- **Keamanan:**
  - Kode repo dibungkus pembatas di prompt dan diperlakukan sebagai data, bukan instruksi.
  - Output di-sanitize sebelum dirender.
  - API key hanya ada di `.env` server.

### 9.5 Format output AI
**Penjelasan (explain):**
```json
{
  "summary": "Ini apa (1–2 kalimat)",
  "how_it_works": [{"lines": [12, 18], "text": "..."}],
  "why": "Kenapa ditulis begini dan alternatif umum",
  "change_impact": "Aman diubah: ... Hati-hati: ...",
  "concept_tags": ["laravel.validation"],
  "confidence": "high | medium | low"
}
```
Relasi dan diagram tidak dibuat ulang oleh AI, karena sudah tersedia dari Part 1.

**Peta proyek AI (overview):**
```json
{ "summary": "...", "highlights": ["..."], "concept_tags": ["next.app-router"] }
```

**Cari fitur (ask):**
```json
{
  "answer": "...",
  "locations": [{"path": "...", "start_line": 1, "end_line": 40, "reason": "..."}],
  "confidence": "high | medium | low"
}
```

Tag konsep yang tidak ada di tabel `concepts` dicatat di `unknown_concept_tags`, sebagai bahan kurasi berikutnya.

## 10. Metrik sukses Part 2
- Penjelasan AI dinilai 👍 ≥ 80%.
- Cari fitur AI menunjuk file yang benar (dinilai manual pada sampel) ≥ 70%.
- Biaya AI rata-rata < $0,20 per analisis yang memakai AI.
- Persentase analisis yang memakai minimal 1 fitur AI (mengukur minat).

---

## 11. Tahapan
**Part 1 (± 3 minggu)**
- **Minggu 1, fondasi:**
  - skema DB, form + Turnstile + rate limit,
  - pipeline (fetch, filter, stack, indeks),
  - viewer (file tree + kode).
- **Minggu 2, parser dan graf:**
  - parser PHP, skrip parser JS/TS, regex Blade,
  - `symbols`, `file_relations`, `call_edges`, `routes`,
  - penyambungan lintas batas, peran file.
- **Minggu 3, makna dan rilis:**
  - ±40 aturan pola dan templatenya, peta proyek statis,
  - alur per route, panel file, cari fitur,
  - seeder glosarium, docs, dan sinonim,
  - share, feedback, cleanup terjadwal, deploy.

**Part 2 (± 1–2 minggu, setelah syarat di bagian 8 terpenuhi)**
- Integrasi Claude API, lembar fakta ke prompt, explain, overview AI, ask.
- Kuota, batas global, `ai_usage_logs`, uji kualitas dengan 10–20 repo contoh (bandingkan Sonnet dan Opus).

**Setelah MVP 1**
- **MVP 2:** URL web live, screenshot, klik elemen → node di graf Part 1.
- **V2:** browser extension, framework lain (Vue, Nuxt, SvelteKit, Express), kuis belajar, Bahasa Inggris.
- **V3:** login GitHub, repo private, pembayaran, riwayat analisis.

## 12. Pertanyaan terbuka
1. Domain final mana yang tersedia?
2. Nama paket kurasi pertama: aturan, konsep, dan istilah mana yang masuk ±40 pertama? (Usul: urutkan dari yang paling sering muncul di template Lovable/Bolt/v0.)
3. Perlu halaman "contoh analisis" (repo demo) di beranda untuk user yang belum punya repo?

---

# Skema Database (Laravel migration, MySQL 8)

## Diagram relasi
```
analyses 1──* stack_detections
analyses 1──* repo_files 1──* symbols
analyses 1──* file_relations      (from_file → to_file)
analyses 1──* call_edges          (from_symbol/file → to_symbol/file)
analyses 1──* routes              (→ handler symbol)
analyses 1──* pattern_matches     (→ file, symbol, rule_key)
analyses 1──1 project_maps
analyses 1──* search_logs
concepts 1──* concept_links       glossary_terms     synonyms
feedback (polimorfik)

Part 2: explanations, ai_overviews, questions, ai_usage_logs, unknown_concept_tags
```

## Tabel Part 1

### analyses
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| uuid | uuid unique | URL internal |
| repo_owner | string(100) | huruf kecil |
| repo_name | string(100) | huruf kecil |
| branch | string(255) | |
| commit_sha | char(40) | |
| status | enum(queued, fetching, indexing, parsing, mapping, ready, failed) | |
| error_message | text null | pesan ramah |
| file_count | unsigned int default 0 | |
| total_bytes | unsigned bigint default 0 | |
| support_level | enum(full, basic) | |
| stats | json null | jumlah route, komponen, cakupan pola, dll. |
| ip_hash | char(64) | |
| expires_at | timestamp null | |
| timestamps | | |
| **index** | unique(repo_owner, repo_name, commit_sha) | |

### stack_detections
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| name | string(50) | laravel, next, react, vite, tailwind, supabase, … |
| version | string(50) null | |
| source_file | string(255) | |

### repo_files
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| path | string(500) | |
| language | string(30) | php, blade, ts, tsx, js, jsx, html, css, … |
| role | string(50) null | controller, model, migration, page, api_route, component, layout, … |
| size_bytes | unsigned int | |
| line_count | unsigned int | |
| is_entry_point | boolean default false | |
| parse_status | enum(parsed, skipped, failed) | |
| content | longtext null | dikosongkan saat kedaluwarsa |
| **index** | unique(analysis_id, path); FULLTEXT(path, content) | |

### symbols
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| repo_file_id | FK cascade | |
| parent_symbol_id | FK symbols null | method → class |
| kind | enum(class, method, function, component, hook, route_handler, export, constant) | |
| name | string(255) | |
| signature | string(500) null | |
| start_line / end_line | unsigned int | |
| is_exported | boolean default false | |
| **index** | (analysis_id, name), (repo_file_id) | |

### file_relations
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| from_file_id | FK repo_files cascade | |
| to_file_id | FK repo_files null | null = paket luar |
| target_raw | string(500) | `@/lib/db`, `react`, `layouts.app` |
| type | enum(import, include, extends, component) | |
| line | unsigned int | |
| **index** | (analysis_id, from_file_id), (analysis_id, to_file_id) | |

### call_edges
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| from_file_id | FK repo_files cascade | |
| from_symbol_id | FK symbols null | |
| to_file_id | FK repo_files null | |
| to_symbol_id | FK symbols null | |
| to_route_id | FK routes null | untuk fetch/form ke endpoint |
| target_raw | string(500) | teks asli panggilan |
| kind | enum(call, render, fetch, form_submit, redirect, dispatch) | |
| line | unsigned int | |
| resolved | boolean | false = "tidak terlacak" |
| **index** | (analysis_id, from_symbol_id), (analysis_id, to_symbol_id) | |

### routes
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| source | enum(laravel, next_app, next_pages) | |
| method | string(10) | GET, POST, ANY, … |
| uri | string(500) | `/login`, `/api/users/[id]` |
| name | string(255) null | nama route Laravel |
| kind | enum(page, api) | |
| defined_in_file_id | FK repo_files | |
| defined_at_line | unsigned int null | |
| handler_symbol_id | FK symbols null | |
| handler_raw | string(500) null | |
| middleware | json null | |
| **index** | (analysis_id, uri), (analysis_id, name) | |

### pattern_matches
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| repo_file_id | FK cascade | |
| symbol_id | FK symbols null | |
| rule_key | string(100) | `laravel.request.validate` |
| category | enum(input, validation, auth, database, response, navigation, state, side_effect, config, other) | |
| start_line / end_line | unsigned int | |
| captures | json null | misalnya `{"fields": ["email", "password"]}` |
| **index** | (analysis_id, repo_file_id), (analysis_id, rule_key) | |

### project_maps
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade, unique | |
| data | json | ringkasan, statistik, folder, titik masuk, urutan baca |
| timestamps | | |

### search_logs
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| query | string(200) | |
| result_count | unsigned smallint | |
| clicked_file_id | FK repo_files null | diisi saat hasil diklik |
| ip_hash | char(64) | |
| created_at | timestamp | |

### concepts, concept_links, glossary_terms
| Tabel | Kolom utama |
|---|---|
| concepts | id, tag (unique), framework, title_id, short_description |
| concept_links | id, concept_id (FK), version_pattern (`11.x`, `14`, `*`), url, language |
| glossary_terms | id, term (unique), aliases (json), definition_pemula, definition_teknis |

### synonyms
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| term | string(100) unique | `login` |
| expansions | json | `["auth", "signin", "session", "attempt"]` |

### feedback
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| target_type | enum(file, route_flow, project_map, search, explanation, ai_overview, question) | tiga terakhir untuk Part 2 |
| target_ref | string(100) | id file/route/explanation, dll. |
| level | enum(pemula, teknis) | |
| rating | tinyint | 1 / -1 |
| ip_hash | char(64) | |
| created_at | timestamp | |
| **index** | unique(analysis_id, target_type, target_ref, level, ip_hash) | |

## Tabel Part 2

### explanations
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| target_type | enum(file, lines, route) | |
| repo_file_id | FK repo_files null | |
| route_id | FK routes null | |
| start_line / end_line | unsigned int | 0/0 untuk seluruh file/route |
| level | enum(pemula, teknis) | |
| data | json | sesuai format 9.5 |
| confidence | enum(high, medium, low) | |
| model_used | string(100) | |
| input_tokens / output_tokens | unsigned int | |
| timestamps | | |
| **index** | unique(analysis_id, target_type, repo_file_id, route_id, start_line, end_line, level) | cache |

### ai_overviews
| Kolom | Tipe |
|---|---|
| id, analysis_id (FK), level, data (json), model_used, input_tokens, output_tokens, timestamps |
| **index** unique(analysis_id, level) |

### questions
| Kolom | Tipe |
|---|---|
| id, analysis_id (FK), question (string 200), question_hash (char 64), level, data (json), model_used, input_tokens, output_tokens, timestamps |
| **index** unique(analysis_id, question_hash, level) |

### ai_usage_logs
| Kolom | Tipe |
|---|---|
| id, analysis_id (FK null), purpose enum(explain, overview, ask), model, input_tokens, output_tokens, cache_read_tokens, cost_usd decimal(10,6), ip_hash, created_at |

### unknown_concept_tags
| Kolom | Tipe |
|---|---|
| id, tag (unique), count (unsigned int), last_seen_at |

## Catatan implementasi
- **Queue:** driver `database`. Pipeline dipecah per tahap (fetch → index → parse → map), masing-masing job bisa dicoba ulang dan memperbarui `status`.
- **Parser JS/TS:** skrip Node terpisah (`parser-js/`) dipanggil lewat Symfony Process. Input berupa daftar path, output JSON (simbol, import, panggilan, JSX render, fetch). Ada batas waktu, memori, dan tanpa jaringan.
- **Aturan pola:** file PHP di `resources/rules/`, divalidasi saat boot, dan diuji dengan fixture (potongan kode contoh → pola yang diharapkan).
- **Kamus konvensi folder** dan **template peran file** disimpan sebagai file bahasa (`lang/id/roles.php`, `lang/id/folders.php`).
- **Rate limit:** `RateLimiter` Laravel (cache).
- **Full-text MySQL:** FULLTEXT memecah kata di tanda `-`. Untuk nama file dan simbol, tambahkan juga pencarian `LIKE` dan indeks `symbols.name`.
- **Scheduled command harian:** hapus data analisis yang sudah melewati `expires_at` (isi file, graf, cache AI).
- **Seeder:** `concepts`, `concept_links`, `glossary_terms`, dan `synonyms` adalah aset kurasi yang dikelola sendiri.
- **Part 2:** API key Claude di `.env`. Nama model dan harga per token di config.

## Endpoint
**Part 1**
| Method | Path | Fungsi |
|---|---|---|
| GET | / | halaman form |
| POST | /analyses | submit URL repo |
| GET | /analyses/{uuid} | viewer |
| GET | /r/{owner}/{repo}/{sha7} | viewer share (read-only) |
| GET | /api/analyses/{uuid} | status, metadata, stack, statistik |
| GET | /api/analyses/{uuid}/tree | file tree + peran |
| GET | /api/analyses/{uuid}/map | peta proyek statis |
| GET | /api/analyses/{uuid}/routes | daftar route |
| GET | /api/analyses/{uuid}/routes/{route_id}/flow | alur route (langkah + Mermaid) |
| GET | /api/analyses/{uuid}/files/{file_id} | isi file + lembar fakta |
| GET | /api/analyses/{uuid}/search?q= | cari fitur statis |
| POST | /api/analyses/{uuid}/search/{log_id}/click | catat klik hasil |
| POST | /api/feedback | `{target_type, target_ref, level, rating}` |
| GET | /api/glossary | istilah untuk tooltip |

**Part 2**
| Method | Path | Fungsi |
|---|---|---|
| POST | /api/analyses/{uuid}/explain | `{target_type, file_id?, route_id?, start_line?, end_line?, level}` |
| GET | /api/analyses/{uuid}/ai-overview?level= | peta proyek AI |
| POST | /api/analyses/{uuid}/ask | `{question, level}` |

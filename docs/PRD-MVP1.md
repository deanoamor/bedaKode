# PRD: bedaKode — MVP 1 (Bedah Repo)

> Webmu dibuat AI dan sudah jalan, tapi kamu belum paham kodenya? Tempel link GitHub-nya. Kami tunjukkan alur kerjanya, dari halaman ke route, controller, sampai database, dan menjelaskan setiap bagiannya dengan bahasa yang mudah.

MVP 1 dibagi dua bagian:

| Bagian | Isi | Arsitektur | Biaya | Perkiraan waktu |
|---|---|---|---|---|
| **Part 1: Bedah Statis** | Analisis kode tanpa AI: route, alur panggilan, pola yang dikenali, penjelasan template, dampak perubahan | **Frontend-only.** Semua analisis berjalan di browser user, dideploy ke static hosting | ± $0 (hosting statis gratis) | ± 3 minggu |
| **Part 2: Narasi AI** | AI menarasikan fakta dari Part 1: "ini apa", "kenapa begini", "aman diubah?", cari fitur dengan bahasa bebas | Frontend Part 1 + **backend kecil** untuk AI, kuota, dan cache | Claude API (per token) + hosting backend | ± 1–2 minggu |

Part 1 sengaja dibuat frontend-only sebagai **tes pasar yang murah**: apakah orang mau memakai bedaKode, tanpa biaya server dan AI. Backend baru dibangun di Part 2 kalau Part 1 terbukti diminati.

Fitur "klik elemen di web live" (screenshot) dipindah ke MVP 2.

---

## 1. Ringkasan
- **Masalah:** Vibe coder (pengguna Lovable, Bolt, v0, Cursor) punya web yang jalan, tapi tidak paham kodenya. Mau ubah sedikit saja takut rusak, dan bingung harus mulai baca dari file mana.
- **Solusi:** User menempel URL repo GitHub (public). Browser mengambil kode dan mengurainya, lalu membangun **peta alur**: route mana memanggil handler mana, komponen mana merender apa, form mana mengirim ke endpoint mana, dan file mana menyentuh database. Setiap pola yang dikenali dijelaskan dengan bahasa awam dan ditautkan ke docs resmi sesuai versi.
- **Target user:** Pemula non-programmer atau semi-programmer yang membangun web dengan bantuan AI.
- **Prinsip produk:**
  - **Fakta dulu, narasi kemudian.** Alur dan relasi diambil dari kode (parser), bukan tebakan. AI (Part 2) hanya menarasikan fakta tersebut.
  - **Selalu ada isi.** Setiap file yang dibuka minimal menampilkan perannya, posisinya di alur, siapa yang memanggil dan dipanggil, dan istilah yang relevan.
  - **Kodemu tidak dikirim ke server kami** (Part 1). Semua analisis terjadi di browser.
- **Pembeda dari ChatGPT/Cursor:**
  - peta alur otomatis yang bisa diklik,
  - dampak perubahan ("file ini dipakai oleh …"),
  - Bahasa Indonesia yang awam,
  - link docs resmi yang dikurasi sesuai versi,
  - gratis tanpa daftar,
  - kode tidak dikirim ke server,
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
| Arsitektur Part 1 | **Frontend-only:** Vite + React + TypeScript, mesin analisis di Web Worker, cache di IndexedDB, deploy ke Cloudflare Pages |
| Parser | **`web-tree-sitter` (WASM)** dengan grammar PHP, TypeScript, TSX, dan JavaScript. Blade dibaca dengan regex |
| Viewer kode | CodeMirror 6 (read-only, mendukung seleksi baris dan dekorasi untuk sorotan pola) |
| Diagram | Mermaid (lazy load, `securityLevel: 'strict'`) |
| Arsitektur Part 2 | Frontend tetap. Ditambah backend API kecil untuk AI, kuota, dan cache (pilihan teknologi lihat bagian 12) |
| Model AI (Part 2) | Claude Haiku 5.5 untuk cari fitur, Claude Sonnet 5.5 untuk narasi. Bisa naik ke Claude Opus 5.5 kalau hasil uji kualitasnya kurang |
| Biaya AI (Part 2) | Ditagih lewat **Claude API** (Claude Console, bayar per token), **terpisah dari langganan Claude Pro**. Target < $0,20 per analisis, batas global $5/hari, plus spend limit di Console |
| Kurasi awal | ±40 aturan pola, ±40 konsep docs, ±40 istilah glosarium |

## 3. Tujuan dan non-tujuan
**Tujuan Part 1**
1. User tanpa login bisa menganalisis repo public dan mendapat viewer interaktif dalam < 1 menit untuk repo kecil (< 300 file).
2. Peta proyek statis: stack, fungsi folder, titik masuk, daftar route, dan urutan baca yang disarankan.
3. Untuk Laravel dan Next.js: memilih sebuah route menampilkan **diagram alur** dari route ke handler, lalu ke panggilan berikutnya (validasi, auth, database, redirect, response).
4. Setiap file menampilkan perannya, simbol di dalamnya, relasi, pola yang dikenali beserta penjelasan template, dan dampak perubahan.
5. Cari fitur lewat kata kunci, sinonim, nama route, dan nama file.
6. Hasil bisa dibagikan lewat link.
7. Biaya operasional mendekati nol, dan cukup data untuk memutuskan lanjut ke Part 2 atau tidak.

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
- Backend, database, atau akun di Part 1.

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

# PART 1: BEDAH STATIS (FRONTEND-ONLY)

## 5. Arsitektur Part 1

```
Browser user
├─ UI (React)                 form, viewer, panel, diagram
├─ Web Worker: mesin analisis (TypeScript murni, tanpa DOM)
│   ├─ fetcher      GitHub API (metadata, SHA, tree) + raw.githubusercontent.com / jsDelivr (isi file)
│   ├─ filter       ekstensi, ukuran, folder sampah, file sensitif
│   ├─ stack        package.json / composer.json / lockfile
│   ├─ parsers      web-tree-sitter (PHP, TS, TSX, JS) + regex Blade
│   ├─ graph        simbol, relasi file, call edge, route, penyambungan lintas batas
│   ├─ rules        pencocokan aturan pola
│   ├─ map          peran file, peta proyek, urutan baca
│   └─ search       indeks pencarian di memori
├─ IndexedDB                  cache hasil analisis per SHA
└─ Aset statis                aturan, konsep, glosarium, sinonim, grammar WASM

Static hosting (Cloudflare Pages) + analitik ringan (Umami/Plausible)
```

- **Mesin analisis ditulis sebagai TypeScript murni** (tanpa DOM, tanpa React), supaya bisa diuji terpisah dan dijalankan juga di Node atau backend kalau nanti dibutuhkan.
- **Tidak ada server milik bedaKode** yang menerima kode. Satu-satunya pihak luar adalah GitHub/jsDelivr (sumber kode) dan layanan analitik (event anonim).

## 6. Fitur Part 1

### 6.1 Form submit
- Satu input: URL GitHub (`https://github.com/{owner}/{repo}`, opsional `/tree/{branch}`).
- Disediakan juga **2–3 repo contoh** (demo) yang bisa diklik untuk user yang belum punya repo.
- Kalau hasil untuk SHA yang sama ada di IndexedDB, langsung tampilkan dari cache.

### 6.2 Pipeline analisis (di Web Worker)
Progress ditampilkan per tahap (mengambil daftar file → mengunduh 120/340 file → mengurai → membangun alur → selesai).

1. **Validasi URL:** hanya `github.com`. Owner dan repo dinormalisasi ke huruf kecil.
2. **Metadata (GitHub API, tanpa token, ± 3 panggilan):**
   - `GET /repos/{o}/{r}`: cek public, default branch, ukuran.
   - `GET /repos/{o}/{r}/commits/{branch}`: SHA.
   - `GET /repos/{o}/{r}/git/trees/{sha}?recursive=1`: daftar file. Kalau `truncated`, tampilkan pesan bahwa repo terlalu besar.
3. **Filter dari daftar file (sebelum diunduh):**
   - buang `node_modules`, `vendor`, `.git`, `dist`, `build`, `.next`, `out`, `coverage`,
   - buang binary, gambar, font, lockfile (kecuali untuk deteksi versi), dan file > 1 MB,
   - **file sensitif** (`.env*`, `*.pem`, `*.key`, `id_rsa*`) tidak diunduh dan tidak ditampilkan,
   - batas 2.000 file teks.
4. **Unduh isi file** dari `raw.githubusercontent.com/{o}/{r}/{sha}/{path}`, dengan fallback `cdn.jsdelivr.net/gh/{o}/{r}@{sha}/{path}`:
   - 8–12 request paralel,
   - coba ulang 2x per file,
   - batas total ± 30 MB teks.
5. **Deteksi stack dan versi** dari `package.json`, `composer.json`, dan lockfile.
6. **Parse dengan tree-sitter:**
   - **PHP:** class, method, function, `use`, panggilan statis dan method (`Auth::attempt`, `$request->validate`, `Model::create`), definisi route.
   - **JS/TS/JSX/TSX:** import/export, fungsi, komponen (fungsi yang mengembalikan JSX), hooks, render komponen, `fetch`/`axios` dengan URL literal, `onSubmit`/`onClick`, server actions, `redirect`/`router.push`.
   - **Blade (regex):** `@extends`, `@include`, `@section`, `<x-komponen>`, `<form action=…>`, `{{ route('…') }}`.
7. **Bangun graf:**
   - `symbols`, `fileRelations` (import, include, extends, component),
   - `callEdges` (call, render, fetch, form_submit, redirect),
   - `routes`:
     - Laravel `Route::get/post/resource/…` → controller@method atau closure,
     - Next.js App Router `app/**/page.tsx` dan `app/**/route.ts` (method dari nama ekspor),
     - Next.js Pages Router `pages/**` dan `pages/api/**`.
   - **Penyambungan lintas batas:** URL literal di `fetch('/api/x')`, `<form action="/login">`, atau `route('login')` dicocokkan ke route.
8. **Cocokkan aturan pola** (6.3).
9. **Tentukan peran file** dan **buat peta proyek** (6.4).
10. **Simpan hasil ke IndexedDB** dengan kunci `{owner}/{repo}/{sha}`, plus versi mesin analisis (supaya cache lama diabaikan saat mesin diperbarui).

Kalau parse gagal pada satu file, file itu ditandai `failed` dan pipeline tetap jalan. Pesan error yang ramah disiapkan untuk: repo tidak ada atau private, limit GitHub API habis (sebutkan kapan reset), repo terlalu besar, dan koneksi putus.

### 6.3 Aturan pola (pattern rules)
- Aturan ditulis sebagai modul TypeScript di `src/rules/{framework}/*.ts`, ter-versi di git, dan diuji dengan fixture.
- Setiap aturan berisi:
  - `key`, misalnya `laravel.request.validate`,
  - `framework` dan `language`,
  - **matcher:** query tree-sitter, atau pola path/regex untuk Blade,
  - **capture** (opsional): misalnya daftar field yang divalidasi,
  - **template teks** pemula dan teknis, dengan placeholder (contoh: "Mengecek input `{fields}` sebelum diproses."),
  - **tag konsep** untuk link docs,
  - **kategori alur** (input, validasi, auth, database, response, navigasi, state, efek samping) untuk ikon dan warna di diagram.
- Target awal: **±40 aturan** (±20 Laravel, ±15 Next.js/React, ±5 umum).
  - Contoh Laravel: `validate`, `Auth::attempt`, `Auth::user`, `middleware('auth')`, Eloquent `create/update/delete/where`, `DB::table`, `redirect()`, `back()->with()`, `view()`, `Mail::send`, `Storage::put`, `Http::get`.
  - Contoh Next.js/React: `useState`, `useEffect`, `useRouter`/`router.push`, `fetch`, `"use client"`, `"use server"`/server actions, `generateMetadata`, `NextResponse.json`, `cookies()`, akses `process.env`.

### 6.4 Peta proyek statis
- **Ringkasan:** template berdasarkan stack. Contoh: "Proyek Laravel 11 dengan Blade dan Tailwind. Ada 8 halaman, 3 endpoint form, 5 tabel database."
- **Statistik:** jumlah file, route, komponen, model, dan migration.
- **Fungsi folder:** dari kamus konvensi per framework (`app/Http/Controllers` → "Pengatur logika setiap halaman", dst.). Folder di luar kamus ditampilkan sebagai "folder buatan proyek" beserta peran dominannya.
- **Titik masuk:** `routes/web.php`, `app/layout.tsx`, `src/main.tsx`, `index.html`, dll.
- **Daftar route:** method, URI, dan handler (bisa diklik).
- **Urutan baca:** titik masuk → route file → handler dengan relasi terbanyak → komponen/layout bersama → model. Maksimal 10 file, masing-masing dengan alasan template.

### 6.5 Alur per route
- User memilih route, lalu mesin menelusuri `callEdges` dari handler (kedalaman maks. 6, hanya simbol di dalam repo).
- Setiap langkah diberi anotasi pola dan kategori alur.
- Hasilnya ditampilkan sebagai:
  - **diagram Mermaid** (bisa diklik ke file dan baris),
  - **daftar langkah bernomor** dengan penjelasan template,
  - **catatan "tidak terlacak"** untuk panggilan yang tidak bisa diselesaikan.
- Untuk Next.js, alur juga bisa dimulai dari halaman: `page.tsx` → komponen → form/fetch → API route → handler.

### 6.6 Panel file
1. **Peran file** beserta penjelasan template perannya.
2. **Posisi di alur:** route yang melewati file ini.
3. **Isi file:** daftar simbol. Klik simbol akan lompat ke barisnya.
4. **Pola yang dikenali:** per rentang baris, dengan teks template sesuai level, dan disorot di kode.
5. **Relasi:** dipanggil oleh dan memanggil.
6. **Dampak perubahan:** daftar file yang bergantung (graf terbalik). Ada peringatan khusus untuk peran sensitif (migration, route file, layout, config).
7. **Istilah dan docs:** tooltip glosarium dan link docs.

Semua data ini juga disusun sebagai **"lembar fakta" JSON** per file dan per route. Lembar fakta ini nantinya jadi input prompt di Part 2.

### 6.7 Cari fitur (statis)
- Input kata kunci, diperluas dengan **kamus sinonim kurasi** (login → auth, signin, session, attempt; bayar → payment, checkout, order, midtrans, xendit, stripe).
- Hasil diurutkan dari: nama route/URI → nama file/simbol → pola yang cocok → isi file.
- Indeks dibangun di memori, misalnya dengan MiniSearch.
- Maksimal 10 hasil, masing-masing dengan alasan ("route `POST /login`", "memanggil `Auth::attempt`").

### 6.8 Viewer
Tata letak tiga panel di desktop. Di mobile berubah jadi tab.
- **Kiri:** file tree (ikon peran, penanda titik masuk) dan tab **Routes**.
- **Tengah:** CodeMirror read-only dengan nomor baris dan sorotan pola.
- **Kanan:** Peta Proyek (default), Panel File, atau Alur Route. Ada slot "Penjelasan AI" yang disembunyikan sampai Part 2.
- **Atas:** kolom cari fitur, pilihan level (Pemula/Teknis), tombol Share, dan info "Berdasarkan commit `abc1234`".
- Setiap file punya link "Buka di GitHub" ke baris yang sesuai.

### 6.9 Share link
- Format: `/r/{owner}/{repo}/{sha}`. State opsional ada di hash URL (`#file=…&line=…` atau `#route=…`).
- Saat dibuka, browser penerima **menjalankan analisis ulang** untuk SHA tersebut. Hasilnya identik karena mesin analisisnya deterministik. Analisis tidak perlu disimpan di server.
- Kalau versi mesin sudah berubah, hasilnya bisa sedikit berbeda. Ini diterima untuk Part 1.

### 6.10 Glosarium, docs, feedback, analitik
- **Glosarium:** ±40 istilah, ditampilkan sebagai tooltip otomatis.
- **Docs:** tag konsep diambil dari aturan pola, URL dari data kurasi sesuai versi yang terdeteksi.
- **Feedback 👍/👎** pada panel file, alur route, peta proyek, dan hasil cari. Dikirim sebagai event analitik.
- **Analitik** (Umami atau Plausible, tanpa cookie):
  - analisis: `analysis_started`, `analysis_completed` (durasi, jumlah file, stack, cakupan pola, % route tersambung), `analysis_failed` (alasan),
  - interaksi: `file_opened`, `route_flow_opened`, `search` (jumlah hasil, diklik/tidak), `feedback`, `share_opened`.
- **Tidak ada isi kode yang dikirim ke analitik.** Nama repo public boleh dikirim, dan ini disebutkan di halaman privasi.

## 7. Aturan penting Part 1
- **Batas repo:** maks 2.000 file teks, 1 MB per file, ± 30 MB teks total. Analisis dihentikan dengan pesan ramah kalau melewati batas.
- **Limit GitHub:** API tanpa token = 60 request/jam per IP user. Analisis memakai ± 3 request API. Isi file diambil dari raw/jsDelivr, jadi tidak memakai kuota API.
- **Keamanan:**
  - Kode repo **tidak pernah dijalankan,** hanya diurai di Web Worker.
  - Semua isi repo dirender sebagai teks. Label Mermaid di-escape dan memakai `securityLevel: 'strict'`.
  - Content Security Policy ketat di hosting: script hanya dari origin sendiri, `connect-src` hanya ke GitHub, jsDelivr, dan analitik.
  - File sensitif tidak diunduh.
- **Cache:** IndexedDB dengan kunci `{owner}/{repo}/{sha}` dan versi mesin. Simpan maks. 10 analisis terakhir (LRU) supaya penyimpanan browser tidak penuh.
- **Performa:** grammar WASM dan Mermaid dimuat malas. Analisis berjalan di Worker supaya UI tetap responsif.

## 8. Metrik sukses Part 1
- Analisis berhasil ≥ 90%.
- Waktu analisis repo < 300 file: p50 < 30 detik.
- File berhasil di-parse ≥ 95%.
- **Cakupan pola ≥ 60%:** persentase file (Laravel/Next.js, di luar config dan aset) yang punya minimal 1 pola dikenali.
- Route berhasil disambungkan ke handler ≥ 80%.
- Rata-rata file atau route yang dibuka per analisis ≥ 5.
- Pencarian yang diikuti klik hasil ≥ 50%.
- Feedback 👍 ≥ 70%.
- Link share dibuka oleh pengunjung baru.

## 9. Syarat lanjut ke Part 2
Lanjut ke Part 2 kalau:
- ada ≥ 100 analisis dari user nyata,
- cakupan pola ≥ 60%, dan
- feedback atau pencarian menunjukkan user butuh penjelasan logika khusus atau "kenapa", yang tidak bisa dijawab template.

---

# PART 2: NARASI AI (DENGAN BACKEND)

## 10. Arsitektur Part 2
```
Browser (Part 1, mesin analisis tetap di sini)
   │  {owner, repo, sha, path, start_line, end_line, level, fact_sheet}
   ▼
Backend API (baru)
   ├─ Turnstile + rate limit per IP
   ├─ cek cache penjelasan
   ├─ AMBIL ULANG isi file dari raw.githubusercontent.com berdasarkan (owner, repo, sha, path)
   ├─ susun prompt: potongan kode (dari server) + lembar fakta (dari browser, dibatasi ukurannya)
   ├─ panggil Claude API → validasi JSON → simpan cache + ai_usage_logs
   └─ kembalikan hasil
Database kecil (SQLite/MySQL): cache penjelasan, kuota, log biaya
```

- **Kenapa server mengambil ulang kode:** supaya backend tidak bisa disalahgunakan sebagai "AI gratis" untuk teks sembarang. Hanya file public yang benar-benar ada di commit tersebut yang bisa dijelaskan.
- **Lembar fakta dari browser dianggap tidak tepercaya:** ukurannya dibatasi, dipakai hanya sebagai konteks, dan path di dalamnya dicocokkan dengan tree GitHub.
- **Mesin analisis tetap di browser.** Karena mesinnya TypeScript murni, nanti bisa juga dijalankan di backend kalau dibutuhkan, misalnya untuk MVP 2.

## 11. Fitur Part 2
### 11.1 Jelaskan lebih dalam
- Tombol ini muncul di panel file, seleksi baris (maks. 200 baris), dan alur route.
- Claude Sonnet 5.5 menulis JSON terstruktur (11.5).
- Output hanya boleh menyebut path yang ada di tree repo. Path lain dibuang.

### 11.2 Peta proyek AI
- Narasi singkat berdasarkan peta statis.
- Dibuat saat diminta ("Ceritakan proyek ini"), lalu di-cache per (repo, sha, level).

### 11.3 Cari fitur dengan bahasa bebas
- Kandidat diambil dari pencarian statis di browser, maksimal 20.
- Claude Haiku 5.5 memilih maksimal 5 lokasi dan memberi alasan.
- Hasil di-cache per (repo, sha, hash pertanyaan, level).

### 11.4 Kontrol biaya dan keamanan
- **Kuota:** 20 permintaan AI per IP per hari. Hasil dari cache tidak dihitung.
- **Turnstile** saat pertama kali memakai fitur AI.
- **Batas global:** $5/hari dari `ai_usage_logs`. Kalau terlewati, tombol AI dinonaktifkan sementara. Fitur statis tetap jalan.
- **Prompt caching** untuk instruksi sistem. **Effort rendah atau sedang** untuk tugas menjelaskan.
- **Spend limit di Claude Console** sebagai pengaman terakhir.
- **Keamanan prompt dan output:**
  - Kode repo dibungkus pembatas di prompt dan diperlakukan sebagai data, bukan instruksi.
  - Output di-sanitize sebelum dirender.
  - API key hanya ada di environment backend.

### 11.5 Format output AI
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
**Peta proyek AI:**
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
Tag konsep yang tidak dikenal dicatat di `unknown_concept_tags` sebagai bahan kurasi.

### 11.6 Tabel backend Part 2
| Tabel | Kolom utama |
|---|---|
| explanations | id, repo_owner, repo_name, commit_sha, target_type enum(file, lines, route), path, route_key, start_line, end_line, level, data (json), confidence, model_used, input_tokens, output_tokens, timestamps. **unique**(repo_owner, repo_name, commit_sha, target_type, path, route_key, start_line, end_line, level) |
| ai_overviews | id, repo_owner, repo_name, commit_sha, level, data (json), model_used, tokens, timestamps. **unique**(repo_owner, repo_name, commit_sha, level) |
| questions | id, repo_owner, repo_name, commit_sha, question, question_hash, level, data (json), model_used, tokens, timestamps. **unique**(repo_owner, repo_name, commit_sha, question_hash, level) |
| ai_usage_logs | id, purpose enum(explain, overview, ask), model, input_tokens, output_tokens, cache_read_tokens, cost_usd, ip_hash, created_at |
| unknown_concept_tags | id, tag (unique), count, last_seen_at |
| feedback | id, target_type, target_ref, level, rating, ip_hash, created_at (pindahan dari analitik, supaya bisa ditautkan ke penjelasan AI) |

Hasil di-cache 30 hari, lalu dihapus oleh tugas terjadwal.

### 11.7 Endpoint Part 2
| Method | Path | Fungsi |
|---|---|---|
| POST | /api/explain | `{owner, repo, sha, target_type, path?, route_key?, start_line?, end_line?, level, fact_sheet}` |
| POST | /api/overview | `{owner, repo, sha, level, project_map}` |
| POST | /api/ask | `{owner, repo, sha, question, level, candidates}` |
| POST | /api/feedback | `{target_type, target_ref, level, rating}` |

## 12. Metrik sukses Part 2
- Penjelasan AI dinilai 👍 ≥ 80%.
- Cari fitur AI menunjuk file yang benar (dinilai manual pada sampel) ≥ 70%.
- Biaya AI rata-rata < $0,20 per analisis yang memakai AI.
- Persentase analisis yang memakai minimal 1 fitur AI.

---

## 13. Struktur proyek Part 1 (usulan)
```
bedaKode/
├─ src/
│  ├─ app/                 UI React: halaman, viewer, panel, komponen
│  ├─ engine/              mesin analisis (TypeScript murni, tanpa DOM)
│  │  ├─ fetch/            GitHub API, raw/jsDelivr, antrean unduhan
│  │  ├─ filter.ts
│  │  ├─ stack.ts
│  │  ├─ parsers/          php.ts, ts.ts, blade.ts (tree-sitter + regex)
│  │  ├─ graph/            symbols, relations, edges, routes, linker
│  │  ├─ rules/            engine pencocokan
│  │  ├─ map/              peran file, peta proyek, urutan baca, alur route
│  │  ├─ search/
│  │  └─ types.ts          tipe AnalysisResult, FactSheet, dll.
│  ├─ rules/               laravel/, next/, react/, common/ (aturan + template)
│  ├─ curation/            concepts.ts, glossary.ts, synonyms.ts, folders.ts, roles.ts
│  ├─ workers/analyze.worker.ts
│  └─ storage/cache.ts     IndexedDB
├─ public/wasm/            grammar tree-sitter (php, typescript, tsx, javascript)
├─ tests/fixtures/         repo mini Laravel/Next.js + hasil yang diharapkan
└─ docs/
```

## 14. Format hasil analisis (disimpan di IndexedDB)
```ts
interface AnalysisResult {
  engineVersion: string;
  repo: { owner: string; name: string; branch: string; sha: string };
  createdAt: string;
  stack: { name: string; version?: string; source: string }[];
  supportLevel: 'full' | 'basic';
  stats: { files: number; routes: number; components: number;
           patternCoverage: number; routesResolved: number };
  files: { id: number; path: string; language: string; role?: string;
           lines: number; size: number; isEntryPoint: boolean;
           parseStatus: 'parsed' | 'skipped' | 'failed' }[];
  contents: Record<number, string>;           // fileId → isi file
  symbols: { id: number; fileId: number; parentId?: number; kind: string;
             name: string; signature?: string; start: number; end: number;
             exported: boolean }[];
  fileRelations: { from: number; to?: number; raw: string;
                   type: 'import' | 'include' | 'extends' | 'component'; line: number }[];
  callEdges: { fromFile: number; fromSymbol?: number; toFile?: number;
               toSymbol?: number; toRoute?: number; raw: string;
               kind: 'call' | 'render' | 'fetch' | 'form_submit' | 'redirect' | 'dispatch';
               line: number; resolved: boolean }[];
  routes: { id: number; source: 'laravel' | 'next_app' | 'next_pages';
            method: string; uri: string; name?: string; kind: 'page' | 'api';
            definedIn: number; line?: number; handlerSymbol?: number;
            handlerRaw?: string; middleware?: string[] }[];
  patterns: { fileId: number; symbolId?: number; ruleKey: string;
              category: string; start: number; end: number;
              captures?: Record<string, unknown> }[];
  projectMap: { summary: string; folders: { path: string; purpose: string }[];
                entryPoints: number[]; readingOrder: { fileId: number; why: string }[] };
}
```

## 15. Tahapan
**Part 1 (± 3 minggu)**
- **Minggu 1, fondasi:**
  - scaffold Vite + React + TS, Worker, dan IndexedDB,
  - fetcher GitHub (metadata, tree, raw/jsDelivr) dengan progress dan error ramah,
  - filter, deteksi stack,
  - viewer (file tree + CodeMirror).
- **Minggu 2, parser dan graf:**
  - integrasi `web-tree-sitter` (PHP, TS, TSX, JS) dan regex Blade,
  - symbols, relasi, call edge, route Laravel/Next.js, penyambungan lintas batas, peran file,
  - fixture test.
- **Minggu 3, makna dan rilis:**
  - ±40 aturan pola dan templatenya, peta proyek, alur route (Mermaid), panel file,
  - cari fitur, kurasi glosarium, docs, dan sinonim,
  - share link, analitik dan feedback, CSP, repo demo,
  - deploy ke Cloudflare Pages.

**Part 2 (± 1–2 minggu, setelah syarat di bagian 9 terpenuhi)**
- Backend API, integrasi Claude API, explain, overview, ask.
- Kuota, Turnstile, batas global, log biaya.
- Uji kualitas dengan 10–20 repo contoh (bandingkan Sonnet dan Opus).

**Setelah MVP 1**
- **MVP 2:** URL web live, screenshot, klik elemen → node di graf (butuh backend + Chromium).
- **V2:** browser extension, framework lain (Vue, Nuxt, SvelteKit, Express), kuis belajar, Bahasa Inggris.
- **V3:** login GitHub, repo private, pembayaran, riwayat analisis.

## 16. Pertanyaan terbuka
1. Domain final mana yang tersedia?
2. **Teknologi backend Part 2:**
   - **Laravel** (rencana awal, nyaman kalau kamu terbiasa PHP, butuh VPS), atau
   - **Cloudflare Workers + D1** (satu bahasa TypeScript dengan frontend, serverless, hampir gratis di awal).

   Bisa diputuskan setelah Part 1 rilis.
3. Paket kurasi pertama: aturan, konsep, dan istilah mana yang masuk ±40 pertama? (Usul: urutkan dari yang paling sering muncul di proyek buatan Lovable/Bolt/v0.)
4. Repo demo mana yang dipakai di beranda (masing-masing 1 untuk Laravel dan Next.js)?
5. Perlu dicek di awal minggu 1:
   - batas rate dan CORS `raw.githubusercontent.com`,
   - batas jsDelivr untuk repo GitHub,
   - apakah download zip dari `codeload.github.com` bisa dilakukan dari browser. Kalau bisa, cukup 1 request per repo.

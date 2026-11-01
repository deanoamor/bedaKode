# PRD: bedaKode — MVP 1 (Bedah Repo)

> Webmu dibuat AI dan sudah jalan, tapi kamu belum paham kodenya? Tempel link GitHub-nya. Kami jelaskan setiap file dan baris dengan bahasa yang mudah: fungsinya apa, kenapa ditulis begitu, terhubung ke mana, dan aman diubah atau tidak.

Dokumen ini menggantikan cakupan PRD awal untuk rilis pertama. Fitur "klik elemen di web live" dipindah ke MVP 2 (lihat bagian 9).

---

## 1. Ringkasan
- **Masalah:** Vibe coder (pengguna Lovable, Bolt, v0, Cursor) punya web yang jalan, tapi tidak paham kodenya. Mau ubah sedikit saja takut rusak. Tanya ke chatbot pun bingung harus mulai dari file mana.
- **Solusi MVP 1:** User menempel URL repo GitHub (public). Sistem mengambil kode, mendeteksi stack, mengindeks file, memetakan hubungan antar file, lalu membuat **peta proyek**. Setelah itu user bisa:
  1. membaca ringkasan proyek dan urutan belajar,
  2. klik file atau blok baris kode untuk mendapat penjelasan,
  3. bertanya "kode untuk X ada di mana?".
- **Target user:** Pemula non-programmer atau semi-programmer yang membangun web dengan bantuan AI.
- **Pembeda dari ChatGPT/Cursor:**
  - peta proyek otomatis tanpa perlu bertanya,
  - pencarian berdasarkan fitur,
  - bagian "kenapa ditulis begini" dan "aman diubah?",
  - Bahasa Indonesia yang awam,
  - link docs resmi yang dikurasi sesuai versi (bukan karangan AI),
  - hasil bisa dibagikan lewat link.
- **Model bisnis (nanti):** Gratis untuk repo public kecil. Berbayar untuk repo private, repo besar, dan kuota lebih banyak.

## 2. Tujuan dan non-tujuan
**Tujuan MVP 1**
1. User tanpa login bisa menganalisis repo public dan mendapat viewer interaktif dalam < 1 menit untuk repo kecil.
2. Setiap analisis menghasilkan peta proyek: stack, struktur folder, titik masuk, dan urutan baca yang disarankan.
3. Klik file atau seleksi baris menghasilkan penjelasan terstruktur, lengkap dengan link ke baris di GitHub.
4. User bisa bertanya "di mana kode untuk X?" dan mendapat daftar file dan baris yang relevan.
5. Hasil bisa dibagikan lewat link.
6. Biaya AI terkendali: AI dipanggil saat analisis (1x untuk peta proyek) dan saat user bertanya/klik, dan semua hasil di-cache.

**Non-tujuan MVP 1**
- URL web live, screenshot, klik elemen (pindah ke MVP 2).
- Repo private, login, pembayaran.
- Mengedit kode atau membuat PR.
- Dukungan penuh untuk semua framework. Fokus dukungan penuh: **HTML biasa, React/Vite, Next.js, Laravel (Blade)**. Repo lain tetap bisa dianalisis dengan "dukungan dasar" (penjelasan jalan, tapi pemetaan relasi file terbatas).
- Bahasa selain Indonesia (struktur data disiapkan untuk multi-bahasa).

## 3. User story
| # | Sebagai | Aku ingin | Supaya |
|---|---|---|---|
| 1 | Vibe coder | menempel URL repo lalu klik Analisis | tahu isi proyekku tanpa daftar |
| 2 | Vibe coder | membaca ringkasan proyek dan fungsi tiap folder | punya gambaran besar sebelum masuk ke detail |
| 3 | Vibe coder | mendapat urutan file yang sebaiknya dibaca | tahu harus mulai dari mana |
| 4 | Vibe coder | klik sebuah file | tahu file ini fungsinya apa |
| 5 | Vibe coder | memblok beberapa baris kode | tahu kode itu ngapain dan kenapa ditulis begitu |
| 6 | Vibe coder | melihat file mana yang memanggil dan dipanggil | paham alurnya |
| 7 | Vibe coder | tahu apa yang terpengaruh kalau kode diubah | berani mengubah tanpa takut rusak |
| 8 | Vibe coder | bertanya "kode login di mana?" | langsung ke bagian yang kucari |
| 9 | Vibe coder | melihat istilah teknis dijelaskan dengan bahasa awam | tidak bingung dengan jargon |
| 10 | Vibe coder | dapat link docs resmi yang sesuai versi | bisa belajar lebih dalam |
| 11 | Vibe coder | memilih level penjelasan (pemula/teknis) | sesuai kemampuanku |
| 12 | Vibe coder | membagikan hasil lewat link | menunjukkan ke teman atau mentor |
| 13 | Vibe coder | memberi nilai 👍/👎 pada penjelasan | penjelasan makin bagus |

## 4. Fitur MVP 1

### 4.1 Form submit
- Satu input: URL GitHub (`https://github.com/{owner}/{repo}`, opsional `/tree/{branch}`).
- Cloudflare Turnstile dan rate limit per IP.
- Kalau analisis untuk commit yang sama sudah ada dan belum kedaluwarsa, langsung arahkan ke hasil yang ada tanpa memproses ulang.

### 4.2 Pipeline analisis (queue)
1. **Validasi:** hanya host `github.com`. Cek repo ada dan public lewat GitHub API (pakai token server supaya tidak kena limit 60 request/jam). Ambil default branch dan commit SHA terbaru.
2. **Fetch:** download zip (codeload). Ekstrak dengan aman:
   - tolak path traversal dan symlink,
   - batasi total ukuran setelah ekstrak dan jumlah entri (cegah zip bomb).
3. **Filter:** buang folder hasil install dan build (`node_modules`, `vendor`, `.git`, `dist`, `build`, `.next`, `out`, `coverage`), binary, gambar, font, dan file > 1 MB. **Sembunyikan file sensitif** (`.env*`, `*.pem`, `*.key`, `id_rsa*`, dsb.): tidak disimpan dan tidak ditampilkan.
4. **Deteksi stack dan versi** dari `package.json`, `composer.json`, dan lockfile (lockfile dibaca untuk versi, tapi tidak diindeks).
5. **Indeks file:** simpan isi teks dan buat indeks full-text. Tidak memakai AI.
6. **Petakan relasi file (statis, tanpa AI):**
   - `import`/`require`/`export from` (JS/TS),
   - `use`/`include`/`require` (PHP),
   - `@extends`/`@include`/`<x-component>` (Blade),
   - route ke controller (Laravel `routes/*.php`),
   - routing file-based (Next.js `app/` dan `pages/`).
7. **Tandai titik masuk:** misalnya `index.html`, `src/main.tsx`, `app/layout.tsx`, `app/page.tsx`, `routes/web.php`.
8. **Buat peta proyek (AI, 1x panggilan):** input berupa file tree, manifest, README (dipotong), daftar titik masuk, dan ringkasan relasi. Hasil disimpan.
9. Status menjadi `ready`.

### 4.3 Viewer
Tata letak tiga panel di desktop. Di mobile berubah jadi tab.
- **Kiri:** file tree (dengan ikon titik masuk dan pencarian nama file).
- **Tengah:** kode dengan syntax highlight dan nomor baris. Tersedia tombol **"Jelaskan file ini"**, dan saat user memblok baris muncul tombol **"Jelaskan baris ini"**.
- **Kanan:** panel penjelasan. Isi default-nya **Peta Proyek**.
- **Atas:** kolom **"Cari fitur…"** (tanya di mana kode X), pilihan level (Pemula/Teknis), tombol Share, dan peringatan "Penjelasan berdasarkan commit `abc1234`".

### 4.4 Peta proyek
- **Ini proyek apa:** 2–3 kalimat.
- **Stack dan versi,** dengan link docs dari tabel kurasi.
- **Struktur folder:** fungsi tiap folder utama.
- **Titik masuk:** dari mana aplikasi mulai berjalan.
- **Urutan baca yang disarankan:** 5–10 file, masing-masing dengan alasan.
- **Diagram alur besar** (Mermaid), misalnya halaman → komponen → API → database.

Level pemula dibuat saat pipeline. Level teknis dibuat saat pertama kali diminta, lalu di-cache.

### 4.5 Jelaskan file atau baris
1. User mengklik "Jelaskan file ini" atau memblok baris (maks **200 baris** per permintaan).
2. Cek cache berdasarkan (file, baris awal, baris akhir, level).
3. Kalau belum ada di cache, susun konteks:
   - potongan kode yang dipilih,
   - nama file dan bahasanya,
   - stack dan versi,
   - daftar file yang memanggil dan dipanggil (dari relasi statis),
   - tanda tangan fungsi atau ekspor dari file terkait (bukan isi penuh).
4. Model menengah menulis penjelasan dalam format JSON terstruktur (lihat bagian 6).
5. Simpan dan kembalikan hasilnya.

### 4.6 Cari fitur ("kode X di mana?")
1. User mengetik pertanyaan, misalnya "di mana kode login?".
2. Cek cache berdasarkan (analisis, hash pertanyaan yang dinormalisasi, level).
3. Cari kandidat lewat full-text search (kata kunci hasil ekstraksi sederhana dan sinonim umum, misalnya login → auth, signin, session), lalu tambahkan nama file dan route.
4. Model kecil memilih maksimal 5 lokasi (file + rentang baris) dan menulis alasan singkat untuk tiap lokasi.
5. Hasil ditampilkan sebagai daftar yang bisa diklik. Klik akan membuka file di baris tersebut dan bisa langsung dijelaskan.

### 4.7 Docs resmi dan glosarium
- AI hanya mengeluarkan **tag konsep** (misalnya `laravel.validation`, `react.useEffect`). URL docs diambil dari tabel kurasi `concept_links`, disesuaikan dengan versi yang terdeteksi. Tag yang tidak ada di tabel diabaikan.
- Istilah di glosarium (API, state, props, route, middleware, dsb.) otomatis diberi tooltip di teks penjelasan.

### 4.8 Share link
- Format: `/r/{owner}/{repo}/{sha7}`, bisa dibuka tanpa login.
- Isinya peta proyek dan semua penjelasan yang sudah di-cache. Penjelasan baru dari pengunjung link share tetap memakai kuota IP pengunjung tersebut.

### 4.9 Feedback
- Tombol 👍/👎 di setiap penjelasan, peta proyek, dan hasil cari fitur. Dipakai untuk metrik kualitas.

## 5. Alur sistem
```
Submit URL repo
  → validasi + ambil SHA → sudah ada & belum expired? → redirect ke hasil
  → analyses(status=queued)
  → job: fetching → indexing (filter, stack, indeks, relasi, titik masuk)
         → summarizing (peta proyek, AI) → ready
  → viewer (polling status tiap 2 detik selama belum ready)

Jelaskan:  POST /api/analyses/{uuid}/explain {file_id, start_line, end_line, level}
  → cache hit? kembalikan
  : susun konteks → LLM menengah → validasi JSON → simpan → kembalikan

Cari fitur: POST /api/analyses/{uuid}/ask {question, level}
  → cache hit? kembalikan
  : full-text kandidat → LLM kecil pilih & beri alasan → simpan → kembalikan
```

## 6. Format output AI
Semua output AI berupa **JSON dengan skema tetap**, divalidasi di server. Kalau tidak valid, dicoba ulang 1x, lalu kembalikan error yang ramah.

**Penjelasan (explain):**
```json
{
  "summary": "Ini apa (1–2 kalimat)",
  "how_it_works": [
    {"lines": [12, 18], "text": "Penjelasan bagian ini"}
  ],
  "why": "Kenapa ditulis begini, dan alternatif yang umum",
  "connections": {
    "called_by": [{"path": "app/page.tsx", "reason": "..."}],
    "calls": [{"path": "lib/supabase.ts", "reason": "..."}]
  },
  "change_impact": "Kalau diubah, yang terpengaruh: ... Aman diubah: ... Hati-hati: ...",
  "flow_mermaid": "flowchart LR ...",
  "concept_tags": ["react.useEffect", "next.server-actions"],
  "confidence": "high | medium | low"
}
```

**Peta proyek (overview):**
```json
{
  "summary": "...",
  "folders": [{"path": "app/", "purpose": "..."}],
  "entry_points": [{"path": "app/layout.tsx", "purpose": "..."}],
  "reading_order": [{"path": "...", "why": "..."}],
  "flow_mermaid": "flowchart TD ...",
  "concept_tags": ["next.app-router"]
}
```

**Cari fitur (ask):**
```json
{
  "answer": "Ringkasan jawaban",
  "locations": [
    {"path": "app/login/page.tsx", "start_line": 1, "end_line": 40, "reason": "..."}
  ],
  "confidence": "high | medium | low"
}
```

Path di output AI dicocokkan ulang dengan `repo_files`. Path yang tidak ada dibuang supaya tidak ada file halusinasi.

## 7. Aturan penting
- **Batas repo:** maks 50 MB zip, 200 MB setelah diekstrak, 2.000 file teks yang diindeks, 1 MB per file.
- **Batas pemakaian gratis (per IP per hari, diatur lewat config):** 3 analisis baru, 30 permintaan AI (explain dan ask). Hasil yang diambil dari cache tidak dihitung.
- **Batas permintaan:** maks 200 baris per explain, maks 200 karakter per pertanyaan.
- **Keamanan:**
  - Hanya `github.com`. Karena tidak ada URL web live di MVP 1, tidak ada risiko SSRF dari sisi itu.
  - Ekstraksi zip aman: tanpa path traversal, symlink, atau zip bomb.
  - Isi repo **dianggap data tidak tepercaya.** Di prompt, kode dibungkus pembatas yang jelas, dan instruksi sistem menegaskan bahwa isi repo bukan perintah.
  - Output AI di-escape atau di-sanitize sebelum dirender. Markdown dirender tanpa HTML mentah, dan Mermaid memakai `securityLevel: 'strict'`.
  - File sensitif (`.env*`, kunci, sertifikat) tidak disimpan dan tidak ditampilkan.
  - IP disimpan dalam bentuk hash (dengan salt), bukan IP asli.
- **Cache:** kunci utama adalah `(owner, repo, commit_sha)`. Repo dengan commit yang sama tidak diproses ulang. Owner dan repo dinormalisasi ke huruf kecil.
- **Retensi:** zip dan folder ekstrak dihapus setelah diindeks. Hasil analisis disimpan 30 hari (bisa diatur), setelah itu isi file dan cache penjelasan dihapus oleh scheduled command.

## 8. Metrik sukses
- Analisis berhasil ≥ 90%.
- Waktu sampai `ready` untuk repo < 300 file: p50 < 45 detik.
- Rata-rata permintaan explain/ask per analisis ≥ 5.
- Penjelasan dinilai 👍 ≥ 80% dari yang memberi nilai.
- Cari fitur menunjuk file yang benar (dinilai manual pada sampel) ≥ 70%.
- Biaya AI rata-rata per analisis < target yang ditetapkan (dipantau lewat `ai_usage_logs`).
- Link share dibuka oleh IP lain selain pembuatnya.

## 9. Tahapan
- **MVP 1 (± 2 minggu):** semua fitur di dokumen ini.
  - Minggu 1: skema DB, form + rate limit, pipeline (fetch, filter, stack, indeks, relasi), viewer file tree + kode.
  - Minggu 2: peta proyek, explain, cari fitur, glosarium + docs kurasi (seeder), share link, feedback, cleanup terjadwal, deploy.
- **MVP 2:** input URL web live, screenshot (Browsershot), klik elemen → dipetakan ke file memakai indeks dan relasi dari MVP 1. Termasuk proteksi SSRF tingkat jaringan.
- **V2:** browser extension, lebih banyak framework, kuis belajar, Bahasa Inggris.
- **V3:** login GitHub, repo private, pembayaran, riwayat analisis.

## 10. Pertanyaan terbuka
1. Nama dan domain final?
2. Batas gratis persisnya (default sementara: 3 analisis, 30 permintaan AI per hari)?
3. Penyedia model AI dan target biaya per analisis?
4. Hosting: VPS mana? (MVP 1 tidak butuh Chromium, jadi kebutuhannya cukup PHP, queue worker, dan MySQL.)
5. Daftar awal konsep kurasi dan glosarium: berapa banyak untuk rilis pertama (usul: ±80 konsep, ±50 istilah)?

---

# Skema Database (Laravel migration, MySQL 8 / SQLite)

## Diagram relasi
```
analyses 1──* stack_detections
analyses 1──* repo_files 1──* file_relations (from_file_id / to_file_id)
analyses 1──* project_overviews (per level)
analyses 1──* explanations *──1 repo_files
analyses 1──* questions
concepts 1──* concept_links          glossary_terms
feedback (polimorfik: explanation / question / project_overview)
ai_usage_logs
```

## Tabel

### analyses
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| uuid | uuid unique | dipakai di URL internal |
| repo_owner | string(100) | huruf kecil |
| repo_name | string(100) | huruf kecil |
| branch | string(255) | branch yang dianalisis |
| commit_sha | char(40) | |
| status | enum(queued, fetching, indexing, summarizing, ready, failed) | |
| error_message | text null | pesan ramah untuk user |
| file_count | unsigned int default 0 | |
| total_bytes | unsigned bigint default 0 | |
| support_level | enum(full, basic) | full = stack yang didukung penuh |
| ip_hash | char(64) | hash IP + salt |
| expires_at | timestamp null | |
| timestamps | | |
| **index** | unique(repo_owner, repo_name, commit_sha) | reuse cache |

### stack_detections
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK analyses cascade | |
| name | string(50) | laravel, react, next, vite, tailwind, alpine, supabase, dll |
| version | string(50) null | dari lockfile/manifest |
| source_file | string(255) | package.json / composer.json / lockfile |

### repo_files
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| path | string(500) | |
| language | string(30) | |
| size_bytes | unsigned int | |
| line_count | unsigned int | |
| is_entry_point | boolean default false | |
| content | longtext null | dikosongkan saat kedaluwarsa |
| **index** | unique(analysis_id, path); FULLTEXT(path, content) | MySQL. Di SQLite pakai tabel virtual FTS5 |

### file_relations
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| from_file_id | FK repo_files cascade | |
| to_file_id | FK repo_files null | null kalau mengarah ke paket luar |
| target_raw | string(500) | teks asli, misalnya `@/lib/db` atau `react` |
| type | enum(import, include, extends, component, route) | |
| line | unsigned int | baris di file asal |
| **index** | (analysis_id, from_file_id), (analysis_id, to_file_id) | |

### project_overviews
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| level | enum(pemula, teknis) | |
| data | json | sesuai skema overview di bagian 6 |
| model_used | string(100) | |
| input_tokens / output_tokens | unsigned int | |
| timestamps | | |
| **index** | unique(analysis_id, level) | cache |

### explanations
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| repo_file_id | FK repo_files cascade | |
| start_line / end_line | unsigned int | 0/0 = seluruh file |
| level | enum(pemula, teknis) | |
| summary | text | "ini apa" (untuk daftar/preview) |
| data | json | sesuai skema explain di bagian 6 |
| confidence | enum(high, medium, low) | |
| model_used | string(100) | |
| input_tokens / output_tokens | unsigned int | |
| timestamps | | |
| **index** | unique(repo_file_id, start_line, end_line, level) | cache |

### questions
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| analysis_id | FK cascade | |
| question | string(200) | teks asli |
| question_hash | char(64) | hash pertanyaan yang dinormalisasi (lowercase, trim, spasi tunggal) |
| level | enum(pemula, teknis) | |
| data | json | sesuai skema ask di bagian 6 |
| model_used | string(100) | |
| input_tokens / output_tokens | unsigned int | |
| timestamps | | |
| **index** | unique(analysis_id, question_hash, level) | cache |

### concepts
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| tag | string(100) unique | `laravel.validation` |
| framework | string(50) | laravel, react, next, html, dst |
| title_id | string | judul (Indonesia) |
| short_description | text | |

### concept_links
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| concept_id | FK cascade | |
| version_pattern | string(20) | `12.x`, `19`, `*` |
| url | string(2048) | URL docs resmi |
| language | char(2) | id/en |

Penanda konsep di `explanations`, `project_overviews`, dan `questions` disimpan sebagai `concept_tags` di dalam kolom `data` (json), lalu diterjemahkan ke link saat dirender. Untuk MVP tidak perlu tabel pivot.

### glossary_terms
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| term | string(100) unique | API, state, route |
| aliases | json null | variasi penulisan |
| definition_pemula | text | |
| definition_teknis | text | |

### feedback
| Kolom | Tipe | Catatan |
|---|---|---|
| id | bigint PK | |
| target_type | enum(explanation, question, project_overview) | |
| target_id | bigint | |
| rating | tinyint | 1 = 👍, -1 = 👎 |
| ip_hash | char(64) | |
| created_at | timestamp | |
| **index** | unique(target_type, target_id, ip_hash) | satu nilai per IP |

### ai_usage_logs
| Kolom | Tipe |
|---|---|
| id | bigint PK |
| analysis_id | FK null |
| purpose | enum(overview, explain, ask) |
| model | string(100) |
| input_tokens / output_tokens | unsigned int |
| cost_usd | decimal(10,6) |
| created_at | timestamp |

## Catatan implementasi
- Rate limit pakai `RateLimiter` Laravel (cache). Tidak perlu tabel.
- Queue pakai driver `database` (tabel bawaan `jobs` dan `failed_jobs`). Pipeline dipecah per tahap supaya status bisa dipantau dan dicoba ulang.
- Full-text: di MySQL, FULLTEXT memecah kata di tanda `-`, jadi tambahkan juga pencarian `LIKE` pada `path` untuk nama file/komponen. Di SQLite pakai FTS5 dengan tokenizer `unicode61` ditambah `tokenchars '-_'`.
- Model AI dan harga per token disimpan di config/env supaya mudah diganti.
- Scheduled command harian: kosongkan `content` di `repo_files` dan hapus cache AI untuk analisis yang sudah melewati `expires_at`.
- Tabel `concepts`, `concept_links`, dan `glossary_terms` diisi lewat seeder. Ini aset kurasi yang dikelola sendiri.

## Endpoint
| Method | Path | Fungsi |
|---|---|---|
| GET | / | halaman form |
| POST | /analyses | submit URL repo (Turnstile + rate limit) |
| GET | /analyses/{uuid} | halaman viewer |
| GET | /r/{owner}/{repo}/{sha7} | halaman share (viewer read-only) |
| GET | /api/analyses/{uuid} | status + metadata + stack |
| GET | /api/analyses/{uuid}/tree | file tree |
| GET | /api/analyses/{uuid}/files/{file_id} | isi file + relasi masuk/keluar |
| GET | /api/analyses/{uuid}/overview?level= | peta proyek (dibuat kalau belum ada) |
| POST | /api/analyses/{uuid}/explain | `{file_id, start_line, end_line, level}` |
| POST | /api/analyses/{uuid}/ask | `{question, level}` |
| POST | /api/feedback | `{target_type, target_id, rating}` |
| GET | /api/glossary | daftar istilah (untuk tooltip, di-cache di browser) |

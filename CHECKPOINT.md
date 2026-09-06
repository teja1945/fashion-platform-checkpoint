>>> WAJIB DIBACA DULU SEBELUM APAPUN LAIN: lihat Bagian 64 "FILOSOFI PRODUK — 9 RASA" di bawah (termasuk Rasa Grosir, Kepemimpinan, Ketelitian — ditambahkan Bagian 88 & 14 Agustus 2026). Semua fitur baru (endpoint, UI, notifikasi, teks, dashboard) WAJIB dicek balik ke 9 Rasa sebelum dianggap selesai. Ini prinsip permanen, bukan sekadar 1 dari banyak ide di checkpoint ini.

>>> WAJIB JUGA: file ini punya BANYAK "ATURAN WAJIB" lain tersebar di tengah dokumen, bukan cuma yang di atas. Di AWAL sesi, jalankan `grep -n "ATURAN WAJIB\|WAJIB DIBACA\|PRINSIP PERMANEN" CHECKPOINT.md` untuk lihat daftar lengkapnya -- jangan cuma andalkan ingatan dari 1x baca linear. Jalankan ulang grep yang sama SEBELUM menulis status "SELESAI"/"TERUJI" apapun atau bilang ke user "tidak ada yang kurang" (lihat aturan 6 Sept 2026 di bawah, lahir dari insiden Bagian 181). <<<

CHECKPOINT — Fashion Platform (Multi-Tenant SaaS)
Update terakhir: 2 September 2026 (split keempat — arsip Bagian 136-161 ke CHECKPOINT_ARCHIVE_5.md)

Cara pakai:
- File ini isinya STATUS TERKINI + NEXT STEPS AKTIF saja. Histori lengkap:
  - Bagian 1-53: CHECKPOINT_ARCHIVE.md (dibekukan 8 Agustus 2026)
  - Bagian 1-88 lengkap (snapshot sebelum diringkas): CHECKPOINT_ARCHIVE_2.md (dibekukan 14 Agustus 2026)
  - Bagian 89-114: CHECKPOINT_ARCHIVE_3.md (dibekukan 16 Agustus 2026)
  - Bagian 115-135: CHECKPOINT_ARCHIVE_4.md (dibekukan 19 Agustus 2026)
  - Bagian 136-161: CHECKPOINT_ARCHIVE_5.md (dibekukan 2 September 2026)
  Rujuk nomor bagian di archive terkait kalau butuh detail (root cause bug, command persis, alasan desain).
- Tiap sesi baru: kasih raw link CHECKPOINT.md ini (format commit SHA) ke Claude sebelum minta lanjut kerja — SEKALIAN kasih output `wc -l CHECKPOINT.md` di pesan yang sama (lihat SOP di bagian Kolaborasi & Cache).
- Kalau butuh histori detail suatu topik, kasih juga raw link archive yang relevan dengan SHA yang sama.
- Semua file archive TIDAK PERNAH diedit lagi — cuma dibaca sebagai referensi historis. Semua update selanjutnya HANYA masuk ke CHECKPOINT.md (file ini). Kalau file ini membengkak lagi, lakukan split baru (arsipkan versi lama jadi CHECKPOINT_ARCHIVE_5.md, mulai ringkas lagi).

===================================================================
1. ARAH PROYEK (ringkas — detail penuh di archive bagian 1-13)
===================================================================
- Platform multi-tenant SaaS fashion: brand owner, vendor konveksi, custom tailor, pabrik. 1 backend + 1 database untuk semua tenant, isolasi via tenant_id + RLS (wajib, bukan opsional).
- Uang customer masuk langsung ke tenant, platform dapat fee (tenant_billing).
- Frontend beda per tipe tenant (componentized blocks), backend/produksi/inventory sama untuk semua.
- Basis backend: kode LTOS lama (Termux, single-tenant) digeneralisasi jadi multi-tenant. LTOS sendiri sudah dihentikan operasionalnya, murni jadi basis kode.
- Constraint pembayaran: tidak ada kartu kredit/debit internasional (cuma BRI/GPN domestik + SeaBank virtual) — ini kenapa VPS Biznet Gio + Supabase dipilih (terima transfer domestik), dan kenapa Claude Code masih tertunda. Detail lengkap di archive bagian 10.

===================================================================
2. IDENTIFIER KUNCI
===================================================================
- Repo GitHub: teja1945/fashion-platform (public)
- VPS: Biznet Gio, Jakarta, user Rakyat, IP <VPS_IP, lihat CHECKPOINT_LOCAL.md>, Ubuntu 22.04.5 LTS
- Supabase project (aktif): <SUPABASE_PROJECT_ID, lihat CHECKPOINT_LOCAL.md> — https://<SUPABASE_PROJECT_ID, lihat CHECKPOINT_LOCAL.md>.supabase.co
- Supabase project lama (LTOS, di-pause): <SUPABASE_PROJECT_ID_LAMA, lihat CHECKPOINT_LOCAL.md> — JANGAN dihapus, ada data historis, RLS sudah aman
- Demo tenant ID: <DEMO_TENANT_ID, lihat CHECKPOINT_LOCAL.md>, subdomain testing "demo" (host: demo.fashion-platform.local)
- Demo production job (testing): <DEMO_JOB_ID, lihat CHECKPOINT_LOCAL.md> — sudah dipakai bolak-balik reset ke stage jahit untuk testing berulang, aman dipakai ulang
- Staff test tenant demo: Admin Demo (role owner, id <ADMIN_DEMO_ID>), Staff Gudang/Cutting/Jahit/QC/Packing(finishing) Demo — semua PIN direset ke nilai yang sama, lihat CHECKPOINT_LOCAL.md untuk id & PIN lengkap
- Vercel: project fashion-platform terhubung ke repo, auto-deploy dari main, URL https://fashion-platform-six.vercel.app (masih 404, belum ada kode frontend)

===================================================================
3. STATUS INFRASTRUKTUR — HARDENING DASAR TUNTAS ✅
===================================================================
[x] SSH key-only, UFW default-deny (cuma port 22 publik), Fail2Ban aktif, user non-root+sudo, backup pg_dump otomatis harian (retensi 14 hari), pm2+systemd (server.js auto-restart tervalidasi lewat reboot), Node.js 20 LTS.

Belum ada (prioritas berikutnya):
[x] HTTPS/SSL -- SELESAI (archive bagian 99-100, hardening lanjutan Bagian 138-139: Grade A+ testssl.sh)
[ ] Rate limiting API level umum (bukan cuma endpoint PIN)
[x] Restore drill -- SELESAI 19 Agustus 2026, lihat Bagian 142

Detail kronologi: archive bagian 11, 14, 43-46.

===================================================================
4. STATUS BACKEND — RINGKASAN
===================================================================
- Schema v2, semua tabel RLS aktif, role app_user (non-superuser, no bypass RLS).
- Struktur kolom akurat HANYA bisa diverifikasi via `\d nama_tabel` langsung ke DB — file schema di repo tidak merepresentasikan skema live sepenuhnya.
- Tenant resolver middleware (subdomain → tenant_id), event-sourcing pipeline (versioning.js, stateLayer.js, ingestion.js) — jalan, tenant-aware.
- Header/auth: `x-api-key`, `x-staff-token`.
- MCP terhubung: Supabase & Vercel aktif dari chat Claude.
- VPS punya akses push ke GitHub via PAT (expired ~awal November 2026 — INGAT perpanjang).
- Pola wajib akses DB: `withTenant(client, tenantId, fn)` untuk endpoint biasa, `withTenantAndStaff(client, tenantId, staffId, fn)` untuk endpoint yang nyentuh tabel RLS staff-scoped (discrepancy_cases dst) — endpoint yang masih pakai withTenant biasa di tabel staff-scoped akan dapat 0 rows (fail-closed, bukan bug).
- JANGAN query tabel RLS-protected pakai `pool.query()` langsung di luar transaksi — current_setting bisa dapat koneksi pool "kosong", gagal cast uuid tidak konsisten. Selalu lewat transaksi yang sudah tenant-scoped.
- Cara verifikasi data manual via psql: RLS tabel utama pakai session variable `app.tenant_id` (BUKAN `app.current_tenant_id`). SET harus digabung dalam satu perintah `-c` yang sama dengan query-nya: `psql "$DATABASE_URL" -c "SET app.tenant_id = '<uuid-tenant>'; SELECT ... ;"`. Tanpa SET ini, semua query balik 0 rows walau data utuh (RLS bekerja sesuai desain, bukan indikasi data hilang). Tabel `tenants` sendiri cuma bisa diakses `service_role`, app_user gak akan pernah bisa SELECT langsung meski context sudah di-SET.

Bug-bug kritis historis (sudah diperbaiki, detail di archive, jangan diulang):
- orders.production_job_id, sequence_version string-concat bigint, GRANT USAGE schema extensions (archive bagian 35, 36, 39)
- UUID empty-string broadcast error (archive bagian 80) — root cause: query RLS-protected table via pool.query() di luar transaksi

===================================================================
5. NEXT STEPS AKTIF
===================================================================
[x] Restore drill -- SELESAI 19 Agustus 2026, lihat ARCHIVE_5 Bagian 142
[x] Backup off-site 3-2-1 -- SELESAI 20 Agustus 2026, lihat ARCHIVE_5 Bagian 143
[x] UptimeRobot + Sentry -- SELESAI 20 Agustus 2026, lihat ARCHIVE_5 Bagian 144
[x] Sembunyikan stack trace error -- SELESAI 20 Agustus 2026, lihat ARCHIVE_5 Bagian 145
[x] Dependency pinning / lockfile audit -- SELESAI 20 Agustus 2026, lihat ARCHIVE_5 Bagian 146
[x] Tabel tenant_custom_domains -- SELESAI 20 Agustus 2026, lihat ARCHIVE_5 Bagian 147
[x] 2FA akun kritis (GitHub/Supabase/Biznet Gio via Google SSO) -- SELESAI 21 Agustus 2026, lihat ARCHIVE_5 Bagian 151
[x] Polish pass 13 pesan "internal error" generic -- SELESAI 21 Agustus 2026, lihat ARCHIVE_5 Bagian 153
[x] PIN progressive lockout -- SELESAI 21 Agustus 2026, lihat ARCHIVE_5 Bagian 154

>>> PRIORITAS TERTINGGI BARU -- AUDIT CHATGPT KETIGA (Bagian 170, 2 September 2026) <<<
>>> Level DI ATAS prioritas robots.txt/testing mediator di bawah -- ini soal integritas data produksi & keamanan inti, bukan cuma bug kecil <<<
[ ] 1. Lock down POST /v1/events -- WAJIB requireStaffSession, sekarang API key tenant doang cukup buat bikin production event (P0, detail Bagian 170)
[ ] 2. Fix transaction bug di /v1/stage-submissions/:id/confirm -- withTenant() commit meski ada return error di tengah proses, resiko partial state (P0, detail Bagian 170)
[ ] 3. Fix invariant submission.stage_key harus sama dengan production_job.current_stage saat confirm -- submission lama/basi bisa dipakai majuin stage lagi (P0, detail Bagian 170)
[x] 4. Rekonsiliasi event sequence 10 -- SELESAI, lihat Bagian 173 (didokumentasikan, bukan bug aktif, tidak perlu fix kode)
[ ] 5. Fix semantics reserve_fabric_inventory() -- semua movement type (RESERVED/CONSUMED/RELEASED/RESTOCKED) sekarang diperlakukan sama rata sebagai pengurangan stok, stock_state juga gak keupdate (P0/P1, detail Bagian 170)
[ ] 6. Rewrite scanner.html API contract -- masih pakai entity_id/entity_type, backend sekarang pakai production_job_id/order_id (P0, detail Bagian 170)
[ ] 7. Samakan semua stage key -- scanner pakai sewing/packing/shipping, backend/staff live pakai jahit/finishing/shipped (P1, detail Bagian 170)
[ ] 8. Fix Redis session revoke -- staff_sessions:* TTL gak ikut diperpanjang touchSession(), bisa expired duluan dari session token, revokeStaffSessions() jadi gak nemu token lama (P1, detail Bagian 170)
[ ] 9. Fix race condition search_path di db.js -- SET search_path di pool.on("connect") async gak di-await, berpotensi intermittent failure di function yang butuh schema extensions seperti crypt() (P1, detail Bagian 170)
[ ] 10. Bangun automated integration test yang aman -- test-e2e.js/test-e2e-step2.js sekarang berupa script mutasi manual ke database NYATA, bukan test yang aman dijalankan sembarangan (P2, detail Bagian 170)
[ ] Putuskan status Vercel production -- BLOCKED, error "account configuration", root URL 404 (P1, detail Bagian 170)
[ ] FK index yang belum ada di banyak tabel (production_events, inventory_ledger, discrepancy_cases, stage_quantity_submissions, dst) -- MEDIUM sekarang, HIGH sebelum scale (P2, detail Bagian 163 & 170)
[ ] RLS auth_rls_initplan warning -- optimasi performa (bungkus current_setting() dengan select), BUKAN celah keamanan (P2, detail Bagian 163 & 170)

[ ] Validasi discrepancy_case RESOLVED wajib ada bukti keterlibatan dari KEDUA pihak (submitter DAN receiver), bukan cukup 1 pihak -- prinsip 'dengar dua pihak sebelum putuskan', lihat Bagian 171 poin 4

>>> PRINSIP BARU (2 September 2026, dari audit ChatGPT ketiga) -- BACA SEBELUM PERCAYA STATUS "SELESAI" DI CHECKPOINT MANAPUN <<<
CHECKPOINT.md BUKAN source of truth. Source of truth = GitHub HEAD + Supabase live schema + deployed runtime.
Alasan konkret: Bagian 169 sempat tercatat "SELESAI DAN TER-COMMIT" padahal kode fix-nya sendiri gak pernah ke-push ke GitHub -- baru ketauan lewat audit eksternal ChatGPT, baru dibenerin 2 September 2026 (git commit 7fb6c58).
Ke depan: SEBELUM percaya klaim "SELESAI"/"TERUJI" di checkpoint manapun (termasuk semua archive) yang menyangkut KODE atau DATA -- verifikasi dulu langsung ke git log/git diff/database live, jangan cuma percaya narasi tertulis.

>>> PRIORITAS TERTINGGI SEKARANG (jangan lewatkan) <<<
[ ] Fix bug robots.txt/noindex rakyat.benangrasa.com -- ditemukan Bagian 168, BELUM diperbaiki, 5 langkah fix sudah siap di Bagian 168
[ ] Testing fungsional P1 fix POST /v1/mediators (4 skenario) -- kode sudah commit Bagian 169, BELUM ditest, ini fix keamanan tenant isolation

--- Arah kerja disepakati (Bagian 152 & 163, cross-check ChatGPT): selesaikan SEMUA item di bawah SEBELUM mulai scope besar baru (Frontend, Backend Inventory, Dashboard Owner) ---
[ ] OWASP ZAP dynamic testing ke tenant demo
[ ] ClamAV integrasi ke endpoint /v1/photos (sync vs async) -- terinstall belum aktif, lihat ARCHIVE_5 Bagian 150
[ ] k6 load testing endpoint confirm
[ ] Test suite CI gate
[ ] Audit trail admin & monitoring terpisah dari production_events (force-unlock, revoke staff, eskalasi manual)
[ ] P0-6 -- schema/migration reproducibility
[ ] 51 saran Lynis sisanya (Tingkat 2+) -- menyusul, bukan mendesak
[ ] Rate limiting API level umum (bukan cuma endpoint PIN)
[ ] Validasi input ketat (zod/joi) di semua endpoint
[ ] Enkripsi data sensitif tambahan -- nomor telepon/alamat customer, phone_number staff (plaintext)
[ ] Integritas foto bukti -- EXIF timestamp vs waktu submission, perceptual hash (production_stage_photos & discrepancy_thread_photos)
[ ] Rate limiter & session in-memory single-instance -- BELUM DIKONFIRMASI apakah sudah kejawab Redis atau belum (dicatat 2 Sept 2026 saat split, jangan dihapus sampai dicek ulang)
[ ] API_KEY granular per tenant -- BELUM DIKONFIRMASI apakah sudah kejawab BRG_*_TENANT_API_KEY atau masih perlu lebih granular (dicatat 2 Sept 2026 saat split, jangan dihapus sampai dicek ulang)
[ ] Lapis 3 audit keamanan manusia (freelance pentester, sebelum tenant nyata pertama)
[ ] Draft awal ToS + Privacy Policy
[ ] Mandat eksplisit owner->mediator untuk kasus SERIOUS
[ ] POST /v1/mediators/:id/backups + /resign, endpoint discrepancy case (reason/eskalasi/resolve), extend trigger_type notifications, tabel tenant_trusted_staff, voice note thread, scanner.html sync ke pipeline final, desain child bundle (BUNDLE_ALLOCATION)
[ ] Selidiki DeprecationWarning "client.query() already executing" di worker/realtime relay (opsional, bukan bug fungsional)
[ ] scripts/set-tenant-api-keys.js:34 detect-non-literal-fs-filename -- belum ditelusuri detail (low-risk, dijalankan manual)
[ ] (opsional) telusuri sumber X-Content-Type-Options duplikat (ARCHIVE_5 Bagian 139) & CSP duplikat di root path (Bagian 161)
[ ] Renew DeepSource PAT sebelum ~28 November 2026
[ ] Review temuan MAJOR/MINOR DeepSource lainnya (console.log, unused variable, dst) -- polish pass, gak urgent
[ ] Cari cara ChatGPT bisa audit source code + Vercel lagi (connector gagal sejak repo Private, Bagian 163)
[ ] SSL rakyat.benangrasa.com expire 25 November 2026 -- perpanjang manual (tidak auto-renew), lihat Bagian 159
[ ] Menunggu Deka eksekusi DNS wildcard ke Vercel untuk benangrasa.com, lalu hapus symlink nginx + certificate lama demo/api.benangrasa.com (Bagian 159-160)
[ ] Rencana SEO/GEO rakyat.benangrasa.com (Search Console, sitemap, schema JSON-LD, halaman Keamanan/Kepercayaan) -- TUNDA sampai bug robots.txt di atas selesai diperbaiki, detail lengkap Bagian 168

--- Scope besar berikutnya (SETELAH semua di atas tuntas, urutan disarankan ChatGPT Bagian 163: kunci core flow P0 dulu) ---
[ ] Frontend web responsive (item terbesar, belum tersentuh)
[ ] Backend inventory (CRUD lengkap, baru ada function reserve_fabric_inventory)
[ ] Dashboard Owner (4 pilar, lihat ARCHIVE_5 Bagian 155 untuk desain lengkap)
===================================================================
6. CHECKLIST KEAMANAN — HIDUP, DIREVIEW TIAP ADA FITUR BARU
===================================================================
Prinsip: tidak ada sistem 100% aman, target realistis = minimalkan risiko + tahan serangan umum + cepat tahu kalau ada yang aneh.

Sudah ada: RLS semua tabel (termasuk staff-scoped untuk discrepancy_cases & thread), parameterized queries, PIN di-hash pgcrypto, UFW+Fail2Ban, SSH key-only, rate limiting brute-force PIN, pesan error login tidak bocorkan validitas staff_id, backup rutin, insert-only enforced di level DB untuk thread_messages/photos (REVOKE UPDATE/DELETE dari app_user).

Belum ada (perlu direview ke depan):
[ ] Rate limiting API level umum
[x] HTTPS/SSL -- SELESAI, lihat Bagian 138-139
[ ] Validasi input lebih ketat di semua endpoint
[ ] Audit log admin actions terpisah dari production_events (force-unlock, revoke staff, eskalasi manual)
[ ] Monitoring/alerting otomatis (login gagal beruntun, pola akses aneh)
[ ] Enkripsi data sensitif tambahan — SEKARANG CAKUPANNYA: nomor telepon/alamat customer, DAN phone_number staff (plaintext, konsisten sama customer_contact) — review bareng semua field sensitif ini sekaligus, bukan ditambal satu-satu
[ ] Rate limiter & session in-memory masih single-instance — perlu Redis kalau nanti multi-instance
[ ] API_KEY tunggal untuk semua endpoint — pertimbangkan granular per tenant
[ ] Integritas foto bukti — EXIF timestamp vs waktu submission, perceptual hash (bisa 1 modul sama buat production_stage_photos & discrepancy_thread_photos, keduanya simpan storage_path)
[x] Restore drill -- SELESAI 19 Agustus 2026, lihat Bagian 142

Prinsip wajib untuk fitur self-service baru: selalu tanya "kalau disalahgunakan, dampaknya sejauh mana?" — dan tenant/staff TIDAK PERNAH dikasih akses ke infrastruktur/kredensial Teja dalam bentuk apapun.

ATURAN WAJIB (13 Agustus 2026): sebelum nulis kode baru yang manggil fungsi/helper yang sudah ada di codebase, WAJIB grep/lihat dulu definisi fungsi itu — bukan cuma nebak dari endpoint lain yang polanya mirip. Verifikasi dependency itu langkah PERTAMA.

ATURAN WAJIB (13 Agustus 2026): sebelum bikin endpoint baru yang melibatkan otorisasi staff, WAJIB cek dulu pola otorisasi serupa yang sudah ada di checkpoint (misal: call_log & summon-owner cuma boleh mediator, bukan "semua pihak terlibat") — bukan cuma niru pola generik.

ATURAN WAJIB (kalau ada perubahan nilai enum-like di kolom otorisasi seperti role/status): WAJIB grep semua tempat yang cek nilai lama itu secara hardcode sebelum migration dianggap selesai — migration skema doang TIDAK CUKUP.

ATURAN WAJIB (14 Agustus 2026, Rasa Ketelitian): tiap kali nulis judul bagian baru "SELESAI & TERUJI" di CHECKPOINT.md, WAJIB eksplisit sebutin rasa mana yang diterapkan + wujud konkretnya di fitur itu — bukan cuma klaim umum "sudah dicek 9 rasa" tanpa detail. Tujuannya biar kelihatan jelas di histori, gampang diaudit balik kalau ternyata ada yang kelewat.

ATURAN WAJIB (2 September 2026, dari insiden nyata Bagian 169/170): sebelum menulis status "SELESAI"/"TERUJI"/"TER-COMMIT" di CHECKPOINT.md untuk perubahan KODE atau DATA apapun -- WAJIB sertakan bukti verbatim di dalam kalimat status itu sendiri: commit hash dari `git log --oneline -1` (atau `-3` kalau ada beberapa commit terkait), atau hasil query/verifikasi langsung ke database live kalau soal data. Klaim "selesai" TANPA bukti tertulis eksplisit seperti ini dianggap BELUM TERVERIFIKASI, dan HARUS ditandai jelas sebagai belum terverifikasi (bukan diam-diam ditulis seolah pasti). Ini berlaku di SEMUA room/sesi ke depan. Lahir dari insiden nyata: Bagian 169 sempat diklaim "SELESAI DAN TER-COMMIT" padahal kode fix-nya sendiri gak pernah ke-push ke GitHub -- baru ketauan lewat audit eksternal ChatGPT (Bagian 170), bukan dari verifikasi internal proyek sendiri. Prinsip ini melengkapi (bukan menggantikan) prinsip "CHECKPOINT bukan source of truth" di Bagian 170/Section 5 -- yang itu soal SIKAP saat MEMBACA checkpoint lama, yang ini soal KEWAJIBAN saat MENULIS checkpoint baru.

===================================================================
7. IDE-IDE BELUM DIRISET MATANG (belum keputusan final, jangan mulai coding sebelum next steps aktif selesai)
===================================================================
Daftar ringkas — detail lengkap tiap ide ada di archive pada nomor bagian yang disebut:

A. Visual configurator tenant konveksi — archive bagian 25
B. QR code dual-jalur customer vs produksi + gerbang scan sebelum submit — LOGIC LENGKAP di Bagian 167 (archive bagian 26 = histori awal)
C. Verifikasi 2 pihak staff jahit vs QC + notif WA ke QC — SEBAGIAN BESAR SUDAH DIEKSEKUSI (lihat bagian 57, 61, 71-87 di archive 2), sisanya jadi next steps aktif di atas
D. Tenant theme settings + Pattern library & multi-format export — archive bagian 47
E. Tenant kaos: sablon 3D + upload gambar sendiri — archive bagian 48
F. Sistem upah staff jahit (borongan per pcs) — archive bagian 49
G. Dashboard analytics owner, ruang komplain customer, sistem sewa modular per fitur — archive bagian 50
H. Login email+password + kustomisasi dashboard personal — archive bagian 51
I. 2FA login + subdomain custom pilihan tenant — archive bagian 53
J. Automation "AI mikir + AI eksekusi" — archive bagian 28
K. Adopsi BTOS: visual mannequin 3D, decision center actionable, sewa modular "akses vs pemakaian", entry point trial — archive bagian 67
L. Hardening internal: restore drill, integritas foto, audit log admin, offline-first scanner, skor supplier — archive bagian 68 (restore drill & integritas foto sudah masuk checklist keamanan di atas)
M. Gudang final terhubung ke lokasi rak & data siap kirim — archive bagian 69
N. Adopsi BTOS lanjutan: Resume Don't Recreate, antrian real-time walk-in, AI Vision Judge — archive bagian 70
O. Struktur organisasi pabrik fleksibel: level jabatan, PPIC, QC independen, HRD, shift — archive bagian 77 (diperluas jadi peta besar di bagian 88 di bawah)
P. i18n multi-bahasa per tenant (UI + data yang tenant input sendiri) — dicatat 9 Agustus, belum diriset
Q. Custom nada dering notifikasi per jenis (in-app) — dicatat 9 Agustus, belum diriset
R. Anti-kecurangan submission/QC (kolusi 2 pihak): foto wajib, silang-cek qty vs bahan terpakai, deteksi pola "terlalu mulus", rotasi pasangan kerja — archive bagian 57 lanjutan
S. QR kode detail bawa nama staff + spesifikasi barang (kredit kerja, anti-kecurangan) — archive bagian 57 lanjutan
T. Dashboard "barang selesai siap kirim" — archive bagian 57, 69
U. Diskusi gudang di awal siklus untuk kain cacat/reject — archive bagian 58, lihat juga ringkasan Lapis 2 bagian 9 di bawah
V. Manajemen supplier (tabel suppliers, evaluasi performa, formula skor) — archive bagian 59, 68 poin 5
W. Tipe bayaran staff fleksibel per tenant: harian/piece-rate/bulanan — archive bagian 62
X. Absensi & lembur anti-kecurangan via HP (WebAuthn, geofencing, selfie, timestamp server) — archive bagian 63
Y. AI Copywriter (generate caption/deskripsi produk) & AI Admin Sales/CS Chatbot — archive bagian 65
Z. Roadmap ekspansi modul pelengkap dari LTOS: Customer Journey Portal, Decision Center, Master Data Center, Quotation Engine, Customer Digital Profile, Appointment Scheduling, Fitter App, AI Render Preview — archive bagian 66
AA. Darurat staff di tengah pekerjaan (Lapor Darurat, QR multi-scan, split upah manual) — lihat ringkasan bagian 9 di bawah, desain lengkap archive bagian 73
AB. Modul Laporan/Rekapan Owner "di balik layar" (keuangan, produk, kemajuan, kegagalan, peningkatan) — archive bagian 94
AC. Login email+password + approval owner + kustomisasi dashboard personal — archive bagian 51
AD. 2FA staff + subdomain custom pilihan tenant sendiri saat onboarding — archive bagian 53

===================================================================
8. TOOL DEVELOPMENT — STATUS
===================================================================
- Claude Code: ditunda (bukan ditolak), kebentur constraint pembayaran (bagian 1). Sudah dicoba langsung di VPS, sukses sampai step login, gagal di situ. Eksplorasi Google Play billing (GoPay/ShopeePay/Google Play balance/vouchers domestik) sebagai alternatif — status BELUM DIEKSEKUSI, terakhir dicek beberapa hari lalu.
- MCP (Supabase, Vercel): aktif dan dipakai rutin dari chat Claude.
- Detail percobaan lengkap: archive bagian 16, 28.

===================================================================
9. RINGKASAN SKEMA TABEL AKTIF (Lapis 2 — sistem mediator & diskusi discrepancy)
===================================================================
Ringkasan struktur — detail migration & keputusan desain lengkap ada di archive bagian 74-88.

tenant_mediators: id, tenant_id, staff_id, line_scope (nullable, general kalau kosong), has_full_mandate (default false, TIDAK otomatis pindah ke cadangan), is_active, assigned_by, created_at, updated_at. UNIQUE(tenant_id, staff_id) — staff gak bisa didaftarkan dobel jadi mediator.

mediator_backups: mediator_id, backup_staff_id, priority_order (1 = dicoba duluan). UNIQUE(mediator_id, priority_order), UNIQUE(mediator_id, backup_staff_id). Cadangan WAJIB sudah jadi tenant_mediators resmi duluan (disediakan di depan pas ditunjuk, bukan otomatis pas resign — lihat archive bagian 87).

discrepancy_cases: id, tenant_id, stage_quantity_submission_id (FK, UNIQUE — 1 submission = 1 kasus), production_job_id, submitter_staff_id, receiver_staff_id, mediator_id (nullable — bisa kosong kalau tidak ada mediator aktif, lihat fallback escalated_to_admin di archive bagian 82), status (OPEN/IN_DISCUSSION/RESOLVED/ESCALATED_TO_OWNER), severity (NORMAL/SERIOUS, default NORMAL), resolution_notes, submitter_confirmed_at, receiver_confirmed_at, resolved_by_staff_id, resolved_at, resolved_with_mandate. RLS staff-scoped (bukan cuma tenant-isolation) — akses cuma submitter/receiver/mediator/owner, fail-closed kalau app.staff_id kosong.

discrepancy_thread_messages: id, tenant_id, discrepancy_case_id, sender_staff_id (nullable untuk row otomatis sistem), message_type (text/photo/call_log/mediator_action/correction), action_subtype (khusus mediator_action: joined_case/summoned_owner), content, call_to_staff_id, target_staff_id, corrects_message_id (self-FK, untuk ralat tanpa hapus riwayat). INSERT-ONLY di level DB (REVOKE UPDATE/DELETE). call_log CUMA boleh ditulis mediator.

discrepancy_thread_photos: id, tenant_id, message_id (FK), storage_path, uploaded_by_staff_id, uploaded_at. Generic, tidak terikat stage produksi (beda dari production_stage_photos). INSERT-ONLY.

mediator_reassignment_log: id, tenant_id, discrepancy_case_id, old_mediator_id (nullable), new_mediator_id, reason, triggered_by_staff_id, created_at. Jejak permanen perpindahan mediator (belum ada endpoint yang insert ke sini — nunggu logic resign, next steps bagian 5).

notifications: generic, trigger_type-based, related_staff_id (FK staff — "tujuan hubungin balik" via WA, nomor mentah bukan link jadi), RLS insert-scoped via submitter/receiver/mediator/owner. Baru cover trigger_type discrepancy_summoned_owner — jenis lain (stok kosong, darurat staff, mesin rusak) masih next steps aktif.

reserve_fabric_inventory(...) — function DB (bukan tabel), atomik reserve stok kain + ledger, row-lock FOR UPDATE, SECURITY INVOKER. Dipakai saat order masuk produksi butuh bahan. Trigger utama buat laporan stok kosong (ide U/bagian 72) dan disebut sebagai nilai jual audit ekspor (Modul G, bagian 88).

===================================================================
88. PETA BESAR STRUKTUR PABRIK GARMEN (14 Agustus 2026, riset/rencana, belum diimplementasi — dipertahankan utuh, bukan diringkas, karena jadi peta acuan jangka panjang)
===================================================================
Latar belakang: proyek diarahkan juga bisa ditawarkan ke pabrik skala penuh, bukan cuma konveksi kecil.

FILOSOFI KE-7 — "RASA GROSIR": platform sedia ruang dan kebutuhan sebanyak mungkin di depan (grosir/wholesale), semaksimal mungkin sebelum dibutuhkan — bukan tambal satu-satu pas kepepet. Tenant tinggal aktifkan fitur yang relevan. Sudah dipraktikkan di bagian 87 (cadangan mediator disiapkan di depan, bukan "naik jabatan" mendadak).

ALUR PRODUKSI LENGKAP: Sales/Merchandising → Costing → Desain → CAD/Pattern → Sampel → ACC Buyer → Purchasing → Cutting → Jahit (sub-stage custom) → QC → Finishing (sub-stage custom) → QC akhir → Gudang barang jadi → Pengiriman → Retur/Komplain

MODUL A — Alur Produksi Utama (linear, jantung sistem). Implikasi teknis terbesar: stage jahit & finishing perlu 2 LEVEL (stage utama → sub-stage custom per tenant), perluasan pola modular bagian 61/66.
MODUL B — PPIC: visibility lintas semua production_jobs, pantau gap_status/deadline.
MODUL C — Industrial Engineering: waktu standar, kapasitas line. Opsional, referensi doang.
MODUL D — Finance: invoice buyer, bayar supplier, payroll, refund retur. Nyambung ke Modul A di titik trigger saja.
MODUL E — HRD: rekrutmen, pelatihan, performa, absensi. Terpisah total dari Modul A.
MODUL F — Maintenance/Teknisi: "lapor mesin rusak" → notifikasi teknisi, pakai pola tabel notifications yang sudah ada (extend trigger_type, bukan bangun dari nol).
MODUL G — Compliance/Audit (fitur laporan, bukan modul aktif): audit trail yang sudah dibangun dari awal (event-sourcing, mediator_reassignment_log, production_events) = NILAI JUAL untuk pabrik ekspor (USTR dkk), tinggal export dari data yang sudah ada.
MODUL H — Subkontraktor: field assigned_to_type (internal/vendor-eksternal) di sub-stage.
MODUL I — Database Buyer/Customer: pertimbangkan tabel customers/buyers terpisah untuk buyer berulang (ekspor) — belum diputuskan.
MODUL J — Retur/Komplain: alur sekarang berhenti di "Pengiriman", belum ada tempat retur — masuk Modul D atau modul sendiri, belum diputuskan.

TIDAK PERLU masuk sistem: Marketing (di luar siklus operasional), GA/Safety/Environment (administratif fisik).

HIERARKI LAPANGAN: Operator → Leader (pimpin 1 line) → Foreman (koordinasi beberapa line) → Supervisor (operasional harian, target, kualitas). Leader/Supervisor kandidat alami buat mediator/backup (bagian 74-87).

STATUS: peta/riset, belum ada tabel/kode diimplementasi. Prioritas eksekusi TETAP ngikutin next steps aktif (bagian 5) — ini peta acuan jangka panjang, bukan next step langsung.

===================================================================
64. FILOSOFI PRODUK — 9 RASA — Wajib Diterapkan Nyata di Setiap Langkah
===================================================================
Status: PRINSIP PERMANEN. Berlaku untuk SEMUA pengembangan ke depan, dicek di setiap step — harus kelihatan wujud nyatanya di kode/UI/teks, bukan cuma diingat.

Platform TIDAK punya departemen marketing, sales, copywriter, atau CS secara langsung — tapi setiap sudut platform harus TERASA seolah-olah ada 9 "rasa" ini:

1. Rasa Copywriting — cara platform "ngomong". Teks (notifikasi, tombol, error) ditulis kaya manusia ngomong, bukan "Error: submission failed" tapi "Waduh, gagal kekirim. Coba cek koneksi lo dan ulangi ya."

2. Rasa Sales — cara platform bikin orang PERCAYA. Dashboard nunjukin bukti nyata kejujuran sistem — riwayat lengkap barang, foto bukti kelihatan langsung.

3. Rasa Marketing — cara platform nunjukkin dirinya. Gaya bahasa/visual konsisten, data ditampilkan sebagai cerita/progress (dashboard "barang siap kirim"), bukan tabel angka mentah.

4. Rasa Talent/Penghargaan — cara platform menghargai orang di baliknya. QR kode bawa nama staff pengerjanya, ditampilin sebagai kredit kerja (bukan cuma anti-kecurangan).

5. Rasa Customer Service — cara platform bantu orang PAS ADA MASALAH. Error kasih tau langkah selanjutnya. Fitur membingungkan dikasih penjelasan singkat di tempat. Ada jalan jelas buat benerin kesalahan manusia.

6. Rasa Keamanan — cara platform JAGA kepercayaan orang di dalamnya. Jejak tidak bisa dihapus/ditimpa diam-diam (event, bukan field yang diganti tanpa bekas — koreksi = catatan baru). Otorisasi jelas siapa boleh apa. Tidak ada "setengah jalan" (atomic). Data sensitif diperlakukan hati-hati. Transparan ke yang berhak, tertutup ke yang tidak.

7. Rasa Grosir — cara platform SEDIAKAN RUANG DI DEPAN (ditambahkan 14 Agustus 2026, bagian 88). Sedia kapasitas/fitur semaksimal mungkin sebelum dibutuhkan, tenant tinggal aktifkan yang relevan — bukan tambal satu-satu pas kepepet.

8. Rasa Kepemimpinan — sosok yang menyambungkan semua rasa jadi satu kesatuan (ditambahkan 14 Agustus 2026). LEVEL UTAMA: cara Claude/Teja ngerjain kerjaan dengan 1 visi yang nyambung, bukan mutusin per fitur sendiri-sendiri. Kalau level ini gak jalan, rasa lain bisa kepasang tapi gak nyambung satu sama lain (contoh nyata kejadian pas level ini gak ada: 5 titik otorisasi hardcode role="admin" lupa di-update pas role owner ditambah, archive bagian 76). LEVEL TURUNAN: ada sosok/role yang jadi "pemimpin" nyata di tiap fitur — mediator jadi pemimpin di kasus discrepancy, owner jadi pemimpin di tingkat tenant, gudang jadi "pemimpin siklus" (buka & tutup siklus produksi, bagian 57).

9. Rasa Ketelitian — cara platform gak pernah asal, selalu dicek ulang sebelum dianggap kelar (ditambahkan 14 Agustus 2026). Sudah dipraktikkan lewat aturan-aturan wajib yang ada (cek dependency sebelum nulis kode, cek pola otorisasi sebelum bikin endpoint baru, grep semua tempat kena kalau ubah enum) — sekarang diresmikan jadi rasa, bukan cuma "aturan teknis" terpisah. Termasuk: testing tiap skenario (bukan cuma happy path), baca ulang checkpoint penuh sebelum lanjut kerja.

SIFAT & CARA EKSEKUSI: filosofi ini BUKAN dokumen final, tapi wadah belajar dua arah — Teja belajar dunia marketing/sales/CS/copywriting dari luar DAN dari eksekusi platform ini sendiri. Cara eksekusi (WAJIB): setiap kali Claude mengerjakan sesuatu yang menyentuh salah satu rasa — nulis teks, bikin tampilan, desain alur — BENERAN MASUK ke cara mikir peran itu, bukan nempelin filosofi sebagai label.

ATURAN WAJIB: setiap kali mengerjakan fitur baru (endpoint, UI, notifikasi, dashboard, pesan error, apapun) — cek balik ke 9 filosofi ini SEBELUM dianggap selesai. Tidak harus semua 9 diterapkan sekaligus di 1 fitur, minimal 1-2 yang kelihatan wujud nyatanya. Wajib disebutkan eksplisit rasa mana + wujud konkretnya di judul bagian "SELESAI & TERUJI" (lihat aturan Rasa Ketelitian di checklist keamanan bagian 6).

CATATAN PENERAPAN KE KODE LAMA: wajib untuk kerjaan baru mulai sekarang. Kode/teks/UI lama TIDAK perlu dirombak buru-buru — masuk daftar "polish pass" belakangan.

===================================================================
CARA MENCATAT IDE BARU — INSTRUKSI UNTUK SEMUA ROOM/SESI
===================================================================
Kalau Teja menyampaikan ide baru di sesi manapun, room manapun WAJIB ikuti pola ini:

1. Tulis ide itu sebagai BAGIAN BARU BERNOMOR (nomor lanjut dari bagian terakhir — cek dulu nomor bagian tertinggi, JANGAN tebak/asal nomor. Nomor tertinggi TIDAK DITULIS statis di sini lagi -- gampang basi, sudah terbukti dari 88 ketinggalan jauh sampai sekarang. WAJIB cek dulu pakai `grep -n "^
2. Judul bagian: "[NOMOR]. Ide Awal — [nama ide singkat] ([tanggal], BELUM DIRISET MATANG)"
3. Isi selengkap mungkin dari hasil diskusi — TIDAK perlu diringkas saat pertama dicatat. WAJIB sertakan LOGIC/ALASAN di balik tiap keputusan (bukan cuma kesimpulan akhirnya) — supaya kalau nanti ditanya ulang "kenapa dulu diputusin gini" di sesi/room manapun, jawabannya sudah tertulis, tidak perlu tanya Teja dari nol lagi. Ini prinsip permanen ditambahkan 30 Agustus 2026 setelah pola berulang: ide sempat cuma tercatat 1 baris ringkasan tanpa logic, akibatnya logic harus dijelaskan ulang tiap kali dibuka lagi.
4. Tulis draft-nya, tunjukkan ke Teja untuk direview.
5. Setelah disetujui, APPEND (bukan overwrite) ke CHECKPOINT.md via `cat >> CHECKPOINT.md << 'EOF' ... EOF` di VPS, verifikasi dengan `tail`, baru commit & push.
6. TAMBAHKAN JUGA satu baris ringkasan ide itu ke daftar bagian 7 di atas.
7. JANGAN taruh ide baru ke file archive manapun — sudah dibekukan permanen.
8. Kalau CHECKPOINT.md ini sendiri sudah mulai kepanjangan lagi — usulkan split baru ke Teja, jangan diam-diam dibiarkan membengkak.

===================================================================
KOLABORASI & CACHE — WAJIB DIBACA TIAP SESI BARU
===================================================================
- Repo public, satu sumber kebenaran untuk semua room/sesi Claude. Commit langsung ke main (belum pakai branch, solo dev fase aktif).
- MASALAH CACHE raw.githubusercontent.com: query string ?t= TIDAK CUKUP. WAJIB pakai commit SHA di path URL — commit SHA didapat dari `git log -1 --oneline -- CHECKPOINT.md` di VPS.
- SOP WAJIB (ditambahkan 14 Agustus 2026, dari insiden nyata fetch kepotong): setiap kasih raw link CHECKPOINT.md ke Claude sesi baru, SEKALIAN kasih output `wc -l CHECKPOINT.md` di pesan yang sama. Kalau Claude menyebut bagian terakhir yang dia baca dengan nomor JAUH lebih kecil dari yang seharusnya (cek dulu bagian terakhir yang ditulis) — JANGAN percaya itu sudah lengkap, langsung minta `tail -c 20000 CHECKPOINT.md` (atau `sed -n 'START,ENDp'` kalau tahu range baris pastinya) tanpa berdebat dulu. Ini BUKAN soal cache, murni limitasi ekstraksi tool fetch Claude sendiri, TERBUKTI nyata (kejadian 14 Agustus 2026).
- Room paralel bisa hasilkan kontradiksi kalau nulis bagian sama bersamaan. Kalau nemu info meragukan, JANGAN percaya salah satu versi — verifikasi ke sumber asli (database, file di server).
- SELALU `git pull` sebelum mulai edit/append file apapun yang bakal di-push — VPS punya akses push (PAT), risiko tabrakan lebih tinggi.
- SELALU `git log --oneline -10` di awal sesi sebelum mulai menulis file yang berpotensi sudah dikerjakan room lain.
- Cara edit/append file di VPS: `cat >> nama_file << 'EOF' ... EOF` untuk NAMBAH (bukan nano). `>>` = append, `>` = overwrite total. Verifikasi dengan `tail -N nama_file` setelah append, sebelum commit & push.
- Cross-check ke ChatGPT: rekomendasikan proaktif kalau ada keputusan desain berisiko tinggi (arsitektur data, security, race condition, konsistensi) — jangan nunggu Teja minta duluan. Evaluasi jujur hasilnya.
- Workflow Teja: satu-langkah-satu-waktu. Claude kasih 1 command/langkah, tunggu hasil dari Teja, baru lanjut ke langkah berikutnya. Jangan kasih banyak command sekaligus.
- KLARIFIKASI POLA PEMAKAIAN (17 Agustus 2026): 4 akun Claude yang dipakai
  Teja jalan BERGANTIAN saat kena limit, BUKAN paralel bersamaan -- risiko
  konflik/kontradiksi di atas jauh lebih rendah untuk pola ini. Tetap wajib:
  (a) push ke GitHub SEBELUM pindah akun, commit progress kecil-kecil kalau
  memungkinkan (jangan nunggu semua kelar baru commit), (b) kalau kena limit
  di tengah kerjaan yang belum selesai/belum ditest, sempetin catat status
  "SERAH-TERIMA KE SESI BERIKUTNYA" dulu (pola yang sudah dipakai Bagian
  121/125) sebelum akun itu gak bisa diakses lagi.

===================================================================
ATURAN WAJIB (14 Agustus 2026): jelasin pakai bahasa sederhana dari awal
===================================================================
Tiap kali ada keputusan desain yang perlu persetujuan Teja (pilihan A vs B,
dst), jelasin PAKAI CONTOH KONKRET/SKENARIO NYATA duluan -- bukan istilah
teknis dulu baru disederhanain belakangan pas ditanya. Anggap tiap
pertanyaan itu kayak ngejelasin ke orang yang baru pertama denger konsepnya,
bukan ke sesama developer. Ini berlaku di SEMUA room/sesi Claude ke depan,
bukan cuma sesi ini -- ditemukan dari pola berulang kali Teja perlu minta
"sederhanain" sebelum bisa jawab.

[Bagian 89-114 diarsipkan 16 Agustus 2026 -- lihat CHECKPOINT_ARCHIVE_3.md untuk detail lengkap: alur Stitch/Figma/v0, 4 endpoint discrepancy, audit ChatGPT 15 temuan, checkGaps fix, HTTPS/SSL setup, foto wajib, CodeQL+Dependabot, rate limiter global, P0-1/P0-2/P0-3, P1-2/P1-4/P1-5]

[Bagian 115-135 diarsipkan -- lihat CHECKPOINT_ARCHIVE_4.md untuk histori lengkap (WS auth token, tenant isolation testing, CORS, migrasi Redis, ide-ide 127-135)]

## ATURAN PERMANEN PROYEK -- KEAMANAN KREDENSIAL (berlaku semua sesi, semua AI/tools)

**Sejak 22 Agustus 2026: DILARANG menampilkan/meminta paste raw API key, password, atau secret apapun ke dalam chat/percakapan dengan AI manapun (Claude atau lainnya), dalam bentuk apapun -- termasuk sebagian/redacted manual.**

Cara wajib verifikasi kredensial ke depan:
- Gunakan `grep -c` (hitung kemunculan saja, bukan isi)
- Gunakan output ter-mask (contoh pola: `scripts/set-tenant-api-keys.js` fungsi `mask()`)
- Verifikasi via efek/test (curl test apakah key masih valid/tidak), bukan dengan menampilkan isinya
- Substitusi nilai kredensial via sed/env langsung di VPS, tidak pernah lewat copy-paste manual read dari chat

**PENDING -- next steps prioritas tinggi:** rotasi menyeluruh semua kredensial sensitif di .env sebagai tindakan pencegahan (karena riwayat chat lama tidak bisa diaudit pasti apakah pernah ter-expose):
- [ ] SUPABASE_SECRET_KEY -- rotate via dashboard Supabase (Project Settings > API > Service Role/Secret Key > Regenerate). PERINGATAN: begitu di-regenerate, key lama langsung mati -- update .env + restart PM2 harus SEGERA setelahnya untuk minimalkan downtime.
- [ ] DATABASE_URL -- rotate password postgres, urutan: ganti password di server DB dulu -> update .env -> restart PM2 cepat
- [ ] BACKUP_DATABASE_URL -- sama seperti di atas
- [ ] REDIS_PASSWORD -- ganti config Redis dulu -> update .env -> restart PM2
- SUPABASE_URL, SENTRY_DSN, NODE_ENV: tidak sensitif, tidak perlu rotate


## SOP PERMANEN -- CARA ROTATE KREDENSIAL (berlaku semua sesi ke depan)

**A. Tenant API key (BRG_*_TENANT_API_KEY):**
`node scripts/set-tenant-api-keys.js` -- otomatis generate + update DB + tulis ke .env, output ter-mask. Tidak perlu restart PM2.

**B. REDIS_PASSWORD:**
1. `openssl rand -base64 32`
2. `read -s NEWPASS`
3. `sed -i "s#^REDIS_PASSWORD=.*#REDIS_PASSWORD=$NEWPASS#" ~/fashion-platform/.env`
4. Cek nomor baris: `sudo grep -n "^requirepass" /etc/redis/redis.conf`, lalu `sudo sed -i "NOMOR_BARISs#^requirepass .*#requirepass $NEWPASS#" /etc/redis/redis.conf`
5. `unset NEWPASS`
6. WAJIB: `pm2 restart fashion-platform --update-env` DULU, BARU `sudo systemctl restart redis-server`

**C. SUPABASE_SECRET_KEY:**
1. Dashboard Supabase > Project Settings > API Keys > tab "Secret keys" > "New secret key"
2. `read -s NEWSUPAKEY`, lalu `sed -i "s#^SUPABASE_SECRET_KEY=.*#SUPABASE_SECRET_KEY=$NEWSUPAKEY#" ~/fashion-platform/.env`, lalu `unset NEWSUPAKEY`
3. `pm2 restart fashion-platform --update-env`
4. Test dulu, sukses baru hapus key lama di dashboard (ketik "default" konfirmasi)

**D. DATABASE_URL (role app_user):**
Role custom, bisa diubah langsung via SQL Editor dashboard:
`ALTER ROLE app_user WITH PASSWORD 'password_baru';`
Nama role cuma `app_user` polos, BUKAN pakai suffix project-ref.

**E. BACKUP_DATABASE_URL (role postgres):**
1. Dashboard > Project Settings > Database > "Reset database password" -- role postgres TIDAK bisa di-ALTER manual
2. BUG DIKETAHUI (belum ada fix Supabase per 24 Agustus 2026): pooler kadang tolak password baru walau sudah benar. Ref: github.com/supabase/supabase/issues/44210
3. Solusi terbukti: ulangi reset 2-3 kali, jeda beberapa menit tiap percobaan
4. Kalau kena ECIRCUITBREAKER: stop semua percobaan (termasuk pm2 stop) 2-5 menit dulu
5. Setelah connect sukses, update .env, restart PM2, test pakai `~/backup-db.sh` asli

**PRINSIP UMUM masukin password ke shell:**
- Pakai `read -s VARNAME`, bukan nano/vi, bukan argument command langsung
- Verifikasi panjang: `echo -n "$VARNAME" | wc -c` sebelum dipakai
- Jangan pakai `sed` kalau curiga ada karakter `&` di password, pakai `awk` sebagai gantinya
- Verifikasi setelah edit: `grep -c "^VAR=" .env` harus 1, `wc -l` harus sama kayak sebelumnya
- Jangan paste command yang nampilin isi baris mentah, pakai `grep -c` atau mask dengan `sed 's/=.*/=[ADA]/'`

## 162. Setup Repo Checkpoint Terpisah (Public) -- fashion-platform Sekarang Private (30 Agustus 2026)

**Konteks:** Repo utama `fashion-platform` ditemukan masih berstatus **Public** (seharusnya sudah private sejak awal untuk proyek komersial) -- ditemukan tidak sengaja saat Teja mengecek portofolio pribadi di GitHub. Langsung diubah ke **Private**.

**Masalah yang muncul:** Alur kerja lama untuk kasih CHECKPOINT.md ke sesi Claude baru mengandalkan raw.githubusercontent.com link dari repo `fashion-platform` -- ini cuma bisa diakses kalau repo Public. Begitu diubah Private, link raw lama tidak lagi bisa diakses oleh Claude.

**Solusi -- repo checkpoint terpisah:**
Dibuat repo baru **`fashion-platform-checkpoint`** (Public), isinya HANYA salinan `CHECKPOINT.md` -- tidak ada kode, tidak ada file lain. Repo `fashion-platform` (kode asli + checkpoint asli) tetap Private, tidak pernah diakses publik lagi.

**Analogi untuk diingat:** `fashion-platform` = buku utama (dikunci, cuma Teja yang tulis/edit). `fashion-platform-checkpoint` = fotokopian halaman checkpoint (rak umum, boleh dibaca siapa saja termasuk Claude) -- tidak pernah diedit langsung, cuma di-refresh dari buku utama.

**Verifikasi keamanan sebelum setup ini dianggap selesai:**
- `git log --all --full-history -- .env` di `fashion-platform` -- KOSONG, `.env` tidak pernah ter-commit sepanjang sejarah repo. Kredensial asli aman meski repo sempat public.
- Repo `fashion-platform-checkpoint` diverifikasi HANYA berisi CHECKPOINT.md, tidak ada risiko kebocoran kode/kredensial lewat repo ini.

**Alur kerja BARU (menggantikan cara lama memberi link checkpoint ke Claude):**

Alur kerja kode (edit, commit, push ke `fashion-platform`) TIDAK BERUBAH SAMA SEKALI -- tetap seperti biasa, termasuk update isi CHECKPOINT.md di repo itu.

Yang berubah HANYA cara memberi checkpoint ke room Claude baru. Sebelum kasih link ke Claude, jalankan dulu (dari mana saja):

cd ~/fashion-platform && git pull && cp CHECKPOINT.md ~/checkpoint-public/CHECKPOINT.md && cd ~/checkpoint-public && git add CHECKPOINT.md && git commit -m "sync" && git push

Lalu generate link seperti biasa (format sama persis kebiasaan lama, cuma nama repo beda):

cd ~/checkpoint-public && echo "https://raw.githubusercontent.com/teja1945/fashion-platform-checkpoint/$(git log -1 --format=%H -- CHECKPOINT.md)/CHECKPOINT.md" && wc -l CHECKPOINT.md

Link + jumlah baris yang keluar dari command kedua itu yang ditempel ke room Claude baru manapun -- format dan cara pakainya identik seperti sebelumnya, sumbernya saja yang berbeda repo.

**Catatan penting untuk sesi berikutnya:** kalau CHECKPOINT.md di fashion-platform sudah diupdate tapi lupa dijalankan sinkronisasi ke atas, Claude di room baru akan membaca versi checkpoint yang KETINGGALAN (bukan ketinggalan permanen -- cuma belum di-refresh). Selalu jalankan langkah sync ini SETELAH update checkpoint terakhir di sesi, sebelum pindah room/akun.

**Status: SELESAI & TERUJI.** Dicoba end-to-end: sync pertama kali (commit a427892 di fashion-platform-checkpoint) dan sync ulang (nothing to commit, karena isi sudah identik) -- keduanya berjalan sesuai ekspektasi.


> **[Diarsipkan ke CHECKPOINT_ARCHIVE_6.md]** Ide QR Code dual-jalur (customer vs produksi), Bagian 167. Logic SUDAH disepakati penuh: 1 QR per order jadi gerbang scan-sebelum-submit di sisi produksi (anti-kecurangan, staf harus fisik pegang barang sebelum bisa submit), 1 QR beda dikasih ke customer untuk lihat 4 tahap progress yang disederhanakan (tanpa expose detail stage internal). Implementasi belum dimulai, ditunda sampai next-steps aktif utama selesai.

## 171. Prinsip Diadopsi — Nilai Universal dari Rekam Jejak Kepemimpinan/Perdagangan Nabi Muhammad SAW (2 September 2026, PRINSIP PERMANEN)

**Konteks:** Diadopsi sebagai penguat filosofi produk yang sudah ada (bukan Rasa ke-10 terpisah), fokus pada rekam jejak historis yang applicable universal ke bisnis/kerja apapun -- bukan aspek keagamaan. 3 dari 4 area yang didiskusikan langsung memperkuat Rasa yang sudah ada dengan contoh konkret; 1 area (penyelesaian sengketa) menghasilkan aturan kerja nyata yang masuk Next Steps Aktif.

**1. Transparansi kondisi barang (memperkuat Rasa Keamanan).** Prinsip larangan menyembunyikan cacat barang dari pembeli. Menegaskan KENAPA next-steps "Integritas foto bukti -- EXIF timestamp vs waktu submission, perceptual hash" (Section 5) itu penting: bukan cuma antisipasi kecurangan staff secara teknis, tapi soal hak customer atas kondisi barang yang jujur ditampilkan.

**2. Akurasi kuantitas/takaran (memperkuat Rasa Keamanan).** Prinsip "sempurnakan takaran, jangan curangi". Menegaskan urgensi fix reserve_fabric_inventory() (temuan P0 ChatGPT, Bagian 170 poin 5) -- bukan sekadar bug teknis, tapi pelanggaran prinsip dagang paling dasar kalau kuantitas bahan tidak akurat/konsisten.

**3. Hak pekerja dibayar cepat & adil (memperkuat Rasa Talent/Penghargaan).** Prinsip "berikan upah pekerja sebelum keringatnya kering". Jadi arahan desain untuk ide belum matang "Sistem upah staff jahit borongan" (poin F) dan "Tipe bayaran fleksibel per tenant" (poin W, Section 7) -- begitu dieksekusi nanti, wajib utamakan kecepatan pembayaran dan keadilan perhitungan.

**4. Penyelesaian sengketa adil, dengar dua pihak (memperkuat Rasa Kepemimpinan) -- INI YANG MENGHASILKAN ATURAN KERJA NYATA:** Rekam jejak sebagai penengah sengketa yang selalu dengar kedua pihak sebelum putuskan, solusi yang terasa adil bagi semua (bukan menang-kalah). LOGIC KAITAN LANGSUNG: sistem mediator/discrepancy_cases SUDAH ADA di proyek ini.

**ATURAN WAJIB BARU (berlaku semua sesi ke depan):** Sebelum discrepancy_case ditutup dengan status RESOLVED, WAJIB dipastikan ada bukti keterlibatan dari KEDUA pihak (submitter DAN receiver) di discrepancy_thread_messages atau field submitter_confirmed_at/receiver_confirmed_at -- BUKAN cukup dari 1 pihak saja meski itu pihak yang melapor duluan. Endpoint resolve case perlu divalidasi ulang apakah sudah menegakkan ini; kalau belum, masuk next-steps aktif untuk ditambahkan validasinya.

**Status: 3 poin pertama (dagang, hak pekerja) DIADOPSI sebagai prinsip permanen -- dirujuk saat next-steps terkait dieksekusi, TIDAK perlu kerja kode terpisah sekarang. Poin ke-4 (validasi 2 pihak sebelum resolve) MASUK Next Steps Aktif Section 5 sebagai item kerja nyata yang perlu diverifikasi/diimplementasikan.**

---


> **[Diarsipkan ke CHECKPOINT_ARCHIVE_6.md]** Ide refactor modular monolith (usulan ChatGPT ketiga), bagian dari Bagian 172. server.js dipecah jadi modules/ per domain (auth, tenants, orders, production, inventory, dst) -- tetap 1 backend+1 DB, bukan microservices. DITUNDA sampai semua item P0/P1 + test suite otomatis selesai duluan.

## SOP BARU (2 Sept 2026) — Prioritas belajar/paham, bukan cuma cepat kelar

User (Teja) memutuskan proyek ini gak harus buru-buru kejual/laku — walaupun setahun ke depan, gpp. Fokusnya "nabung" pemahaman programming, bukan cuma progres fitur. Berlaku sebagai ATURAN KERJA WAJIB semua sesi ke depan:

1. **Sebelum nulis/apply kode, jelasin dulu konsepnya** — kenapa masalahnya terjadi, apa alternatif solusinya, bukan langsung lompat ke command siap-pakai.
2. **Kasih kesempatan user nebak/coba dulu** sebelum dikasih jawaban lengkap, terutama buat bug/keputusan desain yang mirip pola yang sudah pernah dibahas.
3. **Sesekali user yang nulis kode sendiri** (bagian yang sudah familiar/berulang), Claude cukup kasih kerangka, bukan kode jadi.
4. Trade-off: proses jadi lebih lambat dari sebelumnya — ini disadari dan diterima user. Kalau ada kondisi darurat/deadline, user bisa minta mode cepat sementara (opt-out per momen, bukan ganti SOP permanen).

**Tujuan akhirnya:** user bisa ngurai dan jelasin sendiri kode fashion-platform ke orang lain (misal tenant/klien) tanpa selalu tergantung ke Claude.

---

## ATURAN WAJIB BARU (3 Sept 2026) — JANGAN PERNAH minta command yang bisa nampilin token/key mentah

**Insiden:** `git remote -v` dijalankan buat cek repo, ternyata nampilin GitHub PAT mentah di URL origin (`https://ghp_xxx@github.com/...`), ke-paste ke chat AI. Token itu langsung dianggap bocor, harus di-revoke + diganti token baru.

**Root cause:** token disimpan LANGSUNG di URL remote git (`https://TOKEN@github.com/...`), bukan di credential helper terpisah. Command sesimpel `git remote -v` otomatis nampilin token itu tanpa disadari.

**ATURAN WAJIB berlaku semua sesi ke depan, untuk Claude manapun yang bantu proyek ini:**
1. Sebelum minta user jalankan command apapun, pikirkan dulu: "apakah command ini BERPOTENSI nampilin isi credential (token, API key, password, PIN, connection string dengan password di dalamnya)?" Kalau iya, JANGAN kasih command itu mentah-mentah.
2. Kalau command itu memang perlu dijalankan buat tujuan lain (misal `git remote -v` buat cek nama remote), WAJIB kasih versi yang di-mask (pakai `sed`, `cut`, atau sejenisnya) SEBELUM user jalankan yang pertama kali — jangan nunggu kejadian dulu baru dikasih tau cara mask-nya.
3. Command yang WAJIB selalu di-mask atau dihindari sama sekali kalau outputnya dikirim ke chat: `git remote -v` (kalau URL simpan token), `env`/`printenv` tanpa filter, `cat .env`, `history` (bisa ada command lama yang isi kredensial), `psql` connection string yang ada password di dalamnya.
4. Solusi jangka panjang yang sebaiknya diterapkan: pindahkan credential dari URL git ke git credential helper terpisah (`git config credential.helper store` atau sejenisnya) — supaya `git remote -v` otomatis aman ditampilkan tanpa perlu mask manual tiap kali. INI MASIH BELUM DIKERJAKAN, next-step terpisah kalau user mau.
5. Kalau kejadian bocor kayak gini terulang, cukup ikuti alur yang sudah terbukti di insiden ini: revoke token lama di GitHub Settings > Developer Settings > Personal access tokens, generate token baru dengan scope MINIMAL yang dibutuhkan (untuk kebutuhan push/pull kode biasa, cukup scope `repo` saja, TIDAK perlu `admin:org`), lalu `git remote set-url origin https://<TOKEN_BARU>@...` dijalankan LANGSUNG oleh user di terminalnya sendiri, tidak pernah dikirim ke chat.

---

## ATURAN WAJIB BARU (3 Sept 2026, DIPERLUAS) — WAJIB cek SOP checkpoint SEBELUM MULAI PEKERJAAN APAPUN, bukan cuma tugas administratif

**Insiden:** Claude diminta sync CHECKPOINT.md ke repo public, langsung improvisasi langkah manual (ls, cp, cd, git diff) padahal SOP lengkapnya SUDAH ADA dari sebelumnya (baris ~419, 1 command siap pakai). User yang nyadar dan nanya "kenapa jadi manual, bukannya udah ada SOP-nya" — bukan Claude yang nyadar duluan.

**Kelemahan yang harus diperbaiki:** Claude (di sesi manapun) punya kecenderungan langsung improvisasi/bikin langkah baru dari nol untuk tugas yang KELIHATAN belum ada prosedurnya, padahal belum tentu benar-benar belum ada -- cuma belum dicek dulu ke checkpoint. User eksplisit minta ini JANGAN dibatasi ke tugas administratif/infra saja -- berlaku untuk SEMUA jenis pekerjaan.

**ATURAN WAJIB (cakupan luas, semua jenis pekerjaan):** Di AWAL mengerjakan apapun -- fix bug, bikin fitur baru, investigasi masalah, tugas administratif/infra, sampai hal kecil -- WAJIB dulu `grep`/telusuri CHECKPOINT.md (dan CHECKPOINT_ARCHIVE_*.md kalau perlu) cari apakah sudah ada SOP/keputusan/pola kerja yang relevan untuk hal itu. Kalau ketemu, PAKAI itu, jangan bikin jalur baru sendiri walau kelihatan "lebih hati-hati" atau "lebih modern". Kalau SOP yang ada ternyata kurang lengkap/perlu diperbaiki, itu didiskusikan dulu ke user sebagai perubahan SOP, bukan diam-diam diganti jalur lain. Ini berlaku SEBELUM baca kode, SEBELUM nulis fix, SEBELUM kasih command apapun ke user.

Ini konsisten dengan prinsip lama "CHECKPOINT bukan source of truth tapi WAJIB dicek dulu" (Bagian 170) -- prinsip itu ternyata sempat dilanggar sendiri oleh Claude di insiden sync ini, dan sekarang diperjelas cakupannya supaya tidak terulang di jenis pekerjaan lain.

---

## ATURAN WAJIB BARU (6 September 2026) -- WAJIB grep ulang semua "ATURAN WAJIB" SEBELUM klaim status SELESAI, bukan cuma andalkan ingatan baca di awal sesi

**Insiden:** Bagian 181 (fix search_path race + pool size) diklaim "SELESAI & TERUJI" dan disampaikan ke user sebagai "tidak ada yang kurang", PADAHAL kode belum di-commit/push sama sekali ke GitHub, dan kalimat status ditulis TANPA bukti commit hash verbatim -- padahal aturan soal itu PERSIS SUDAH ADA di baris ~169, lahir dari insiden serupa (Bagian 169/170). CHECKPOINT.md sudah dibaca lengkap 756 baris di awal room ini, tapi aturan itu "terkubur" di tengah dokumen panjang campuran narasi historis + aturan wajib -- diandalkan dari ingatan 1x baca di awal, bukan dicek ulang secara mekanis pas mau nulis status. User yang nyadar duluan, bukan Claude.

**Kelemahan yang harus diperbaiki:** Baca CHECKPOINT.md sekali di awal sesi TIDAK CUKUP buat nangkep semua aturan wajib yang tersebar di tengah dokumen panjang (700+ baris dan terus bertambah) -- ingatan dari 1x baca gampang kalah sama detail teknis yang lagi dikerjain berjam-jam kemudian.

**ATURAN WAJIB:**
1. Di AWAL setiap sesi/room baru, SEBELUM mulai kerja apapun: jalankan `grep -n "ATURAN WAJIB\|WAJIB DIBACA\|PRINSIP PERMANEN" CHECKPOINT.md` untuk dapetin daftar LENGKAP semua aturan wajib yang berlaku -- jangan cuma andalkan hasil baca linear sekali di awal.
2. SEBELUM menulis status "SELESAI"/"TERUJI"/"TER-COMMIT" apapun di CHECKPOINT.md, ATAU bilang ke user "tidak ada yang kurang"/"semua sudah beres" -- WAJIB jalanin ulang grep yang sama sebagai pengecekan mekanis, bukan ngandelin ingatan. Ini khususnya berlaku buat aturan baris 169 (bukti commit hash verbatim).
3. Kalau checkpoint makin panjang ke depan, pertimbangkan bikin 1 seksi terpisah "DAFTAR SEMUA ATURAN WAJIB" di paling atas file (bukan cuma banner nunjuk ke 1 aturan) supaya makin gampang di-scan tanpa grep manual.

Ini melengkapi aturan baris 483 (cek SOP checkpoint SEBELUM MULAI kerja) -- yang itu soal AWAL kerja, ini soal SEBELUM KLAIM SELESAI, titik yang ternyata masih bisa kelewat walau SOP di awal sudah dicek dan dibaca lengkap.

---

## Catatan Non-Teknis: Rencana Branding (3 Sept 2026)

Proyek ini santai, tidak dikejar buru-buru laku (lihat SOP belajar/paham). Paralel dengan pengerjaan teknis, user mulai bangun branding lewat Instagram:
- Frekuensi: 2-3x seminggu, bentuk konten fleksibel (1 video ATAU 2 caption + gambar)
- Tujuan: biar proyek makin dikenal orang pelan-pelan, meski akun Instagram masih baru
- Target optimasi: SEO (pencarian/explore Instagram) DAN GEO (Generative Engine Optimization -- biar kejawab kalau orang nanya ke AI)

Ini murni catatan konteks, bukan item kerja teknis -- tidak masuk hitungan Next Steps Aktif.

### Struktur Seri Konten (3 Sept 2026)

**Seri A — "Masalah Konveksi"** (edukasi masalah produksi garmen, SEO/GEO-friendly, target: calon tenant pemilik konveksi)
**Seri B — "Progress Bikin Produk"** (behind the scenes bikin fashion-platform, target: kepercayaan + developer)

Pola posting: gantian A-B-A-B dst, ritme 2-3x/minggu (kira-kira tiap 2-3 hari sekali). Seri A lebih sering muncul karena lebih SEO-friendly.

**Tracker progress (update tiap habis posting, catat nomor terakhir tiap seri biar gak keulang/ke-skip):**
- Seri A terakhir: #1 — "Kain Gue Udah Dipotong Belum?" (masalah visibilitas progress order ke vendor konveksi + CTA platform tracking), dipost 3 September 2026
- Seri B terakhir: belum ada (belum mulai)


## Next Steps Dipindahkan dari Arsip 162-173 (6 September 2026)

Item berikut dipindah ke sini SEBELUM Bagian 163-166, 168-170, 172-173 diarsipkan, supaya tidak ikut
terkubur. Gabungkan manual ke daftar Next Steps Aktif utama kalau perlu dirapikan lagi.

[ ] Renew DeepSource PAT sebelum ~28 November 2026
[ ] Review temuan MAJOR/MINOR DeepSource lainnya (console.log, unused variable, dst) -- polish, gak urgent
[ ] OWASP ZAP dynamic testing ke tenant demo
[ ] k6 load testing endpoint confirm
[ ] Audit trail admin & monitoring
[ ] ClamAV integrasi ke endpoint /v1/photos
[ ] 51 saran Lynis sisanya
[ ] Lapis 3 audit keamanan manusia (freelance pentester)
[ ] Draft awal ToS + Privacy Policy
[ ] Mandat eksplisit owner->mediator kasus SERIOUS
[ ] Role-per-event-type validation untuk POST /v1/events -- staff yang lolos requireStaffSession belum
    dicek berhak trigger event_type spesifik apa (Bagian 172)
[ ] Rewrite scanner.html API contract -- masih pakai entity_id/entity_type, backend sudah pakai
    production_job_id/order_id (Bagian 170 poin 6)
[ ] Samakan stage key scanner.html vs backend -- sewing/packing/shipping vs jahit/finishing/shipped
    (Bagian 170 poin 7)
[ ] Fix Vercel project yang masih BLOCKED (Bagian 170 poin 11, dikonfirmasi ulang masih blocked di
    Bagian 178)
[ ] Validasi 2 pihak (submitter+receiver) wajib sebelum discrepancy_case RESOLVED -- prinsip Bagian 171,
    belum ada verifikasi implementasi

## 174. Fix P0 #2 (Transaction Bug Confirm) + Sistem Eskalasi Bertingkat Job Stuck-Stage -- SELESAI & TERUJI (4 September 2026)

**Rasa yang dipenuhi:**
- **Rasa Ketelitian** — bug asli direproduksi langsung lewat API (bukan cuma dibaca dari kode), setiap constraint/RLS/pola existing dicek dulu sebelum nulis kode baru (grep dependency, cek pg_constraint, cek pg_policies), lupa restart pm2 ketauan sendiri dari hasil test yang janggal (bukan diklaim "selesai" padahal belum aktif), dan bug RLS tersembunyi (notifikasi ke non-owner selalu gagal diam-diam) ketemu lewat debug logging sengaja, bukan diasumsikan "pasti kerja".
- **Rasa Customer Service** — staff yang sudah kerja benar (qty valid) tidak lagi disuruh submit ulang kalau step majukan stage gagal karena error teknis; response API jujur kasih tau "tim akan dikabari", bukan pura-pura sukses atau nyalahin staff.
- **Rasa Grosir** — mediator/admin di sistem eskalasi didesain OPSIONAL sejak awal (tenant boleh pakai penuh/separuh/tidak sama sekali), bukan diasumsikan wajib ada; fallback ke owner dibuat eksplisit dan jujur di log/notifikasi, bukan diam-diam gagal kalau tenant belum setting.

**Konteks masalah (dari audit ChatGPT ketiga Bagian 170, P0 #2):** endpoint `POST /v1/stage-submissions/:id/confirm` — kalau `resolveStageTransition()` gagal (misal `pipeline_snapshot` job tidak sinkron dengan `current_stage`), kode lama cuma `return { httpStatus: 400, ... }`, BUKAN `throw`. `withTenantAndStaff` (db.js) cuma peduli `throw` untuk mutuskan ROLLBACK — `return` biasa dianggap sukses, jadi tetap COMMIT. Akibatnya: update status submission (CONFIRMED/DISCREPANCY) dan insert `discrepancy_cases` (kalau ada) tetap tersimpan permanen walau API bilang error, padahal komentar kode lama salah klaim "semuanya atomic, tidak ada lagi kemungkinan submission CONFIRMED tapi stage gagal maju diam-diam".

**Keputusan desain (didiskusikan bertahap dengan user):**
- BUKAN rollback total (staff tidak disuruh submit ulang kerjaan yang sudah benar).
- Submission & discrepancy_case tetap commit; job production ditandai `stage_advance_status='STUCK'` dengan `stage_advance_error` tersimpan.
- Staff lain/job lain tidak terganggu -- hanya job yang stuck yang "ditahan" progressnya. Submission baru ke job yang sama sementara masuk status `BLOCKED_JOB_STUCK` (kolom/status disiapkan, belum ada UI/endpoint konsumsi -- next step).
- Notifikasi bertingkat, BUKAN nunggu 24 jam: T+15 menit reminder ke owner, T+30 menit eskalasi ke mediator `has_full_mandate=true` (fallback admin, fallback lagi ke owner kalau dua-duanya tidak ada -- Rasa Grosir), T+45 menit broadcast ke SEMUA pihak berwenang (owner+admin+mediator aktif) sekaligus sebagai tingkat akhir otomatis.

**Implementasi:**
1. Migration (4x via Supabase MCP, project `fashion-platform` / `kwhybffbcqopqbbnuigg`): kolom `stage_advance_status/error/stuck_at/escalation_level` di `production_jobs`; status `BLOCKED_JOB_STUCK` ditambahkan ke constraint `stage_quantity_submissions_status_check`; trigger_type `stage_advance_stuck` ditambahkan ke constraint `notifications_trigger_type_check`; nilai `BROADCAST_ALL` ditambahkan ke constraint escalation_level.
2. `server.js`: endpoint confirm diubah -- kasus `resolution.error` sekarang UPDATE job jadi STUCK + insert notifikasi T+0 ke owner (di dalam transaksi yang sama, pola sama seperti endpoint `summon-owner`) + tetap `httpStatus 200` dengan pesan jujur ke staff. Ditambah fungsi `broadcastToTenantOwners()` (generik, beda dari `broadcastToDiscrepancyCase` yang butuh konteks case) untuk push realtime WS ke owner yang online.
3. `worker.js`: fungsi baru `checkStuckJobsForTenant`/`checkStuckJobs`/`startStuckJobMonitor` -- pola SAMA PERSIS `checkGaps` yang sudah ada (advisory lock terpisah `771101`, loop `getActiveTenantIds`, `withTenant` per tenant), interval cek tiap 60 detik. 3 tingkat eskalasi dengan fallback owner kalau mediator/admin kosong.
4. Commit: `5d4af07` (di-amend dari `03ff006` karena pesan commit awal kepotong akibat karakter tanda kurung/kutip bikin bash salah parse `-m` inline -- pelajaran: pesan commit panjang/kompleks harus lewat file + `git commit -F`, bukan `-m` inline).

**Bug KEDUA ditemukan & diperbaiki di tengah proses (bukan cuma yang direncanakan):** RLS policy `notifications_insert_scoped` ternyata cuma izinkan INSERT kalau recipient role `owner`, ATAU `source_table='discrepancy_cases'` dengan recipient pihak terlibat case itu. Notifikasi baru kita (`source_table='production_jobs'`) ke admin/mediator SELALU ditolak RLS diam-diam -- 1 INSERT gagal bikin SELURUH transaksi rollback (termasuk update escalation_level), jadi tier 2 (ke mediator/admin beneran) dan tier 3 (broadcast semua) tidak akan PERNAH berhasil sebelum fix ini, walau logic kodenya sudah benar. Ketemu lewat debug logging (`err.stack`) sengaja ditambahkan sementara setelah testing natural (nunggu waktu asli lewat, bukan cuma backdate manual) menunjukkan job macet di `ESCALATED` selama berjam-jam padahal seharusnya sudah `BROADCAST_ALL`. Fix: migration tambahan `allow_stuck_job_notifications_to_admin_and_mediators` -- tambah kondisi OR baru di policy untuk `source_table='production_jobs'` mengizinkan recipient owner/admin/mediator aktif.

**Testing end-to-end (bukan cuma unit test, di tenant demo `8ae20661-626d-42c9-b930-6c926ca3ce99`):**
- Submission asli lewat API (login staff jahit -> upload foto dummy JPEG valid -> submit qty) -> job `current_stage` sengaja dirusak jadi nilai tidak ada di `pipeline_snapshot` -> confirm via API staff QC beneran. Percobaan PERTAMA (sebelum pm2 di-restart) membuktikan bug ASLI: submission ke-CONFIRMED walau response API bilang error -- bukti nyata bug P0 #2, bukan teori.
- Setelah pm2 restart + kode baru aktif: percobaan KEDUA (submission fresh) sukses sesuai desain -- job STUCK, notifikasi T+0 ke owner, response 200 dengan `warning` jujur.
- Worker 3-tingkat divalidasi dengan kombinasi backdate manual `stage_advance_stuck_at` DAN waktu asli yang lewat natural selama sesi (worker jalan otomatis di background) -- tier 1 (reminder owner), tier 2 (fallback owner karena tenant demo memang belum setting mediator/admin), tier 3 (broadcast ke owner + mediator non-owner "Staff Packing Demo" role `staff`, kasus persis yang tadinya diblokir RLS) semua terverifikasi lewat query database langsung, bukan cuma baca log.

**Known issue BELUM TUNTAS (jangan dianggap selesai, jangan lupa dicek lagi ke depan):** Error transient `invalid input syntax for type uuid: ""` muncul 1x dari sekitar 240 tick worker selama testing (~4 jam), tepat sesaat setelah restart pm2, lalu self-healed di tick berikutnya (transaksi rollback bersih via `withTenant`, tidak ada data korup -- jaring pengaman bekerja seperti didesain). Dicurigai kuirk Session Pooler Supabase saat koneksi pool baru dibuat pasca-restart (lihat komentar terkait `search_path` di `db.js`). Belum berhasil direproduksi ulang secara sengaja untuk investigasi lebih lanjut.

**Status: SELESAI & TERUJI.** P0 #2 dari 13 temuan audit ChatGPT ketiga (Bagian 170) sekarang **DITUTUP**. Sisa dari 13 temuan: P0 #3, #5, #6, P1 #8-9 (Redis, search_path race), P2 #10 (test suite).

**Next steps aktif ditambah:**
[ ] Investigasi error transient "invalid input syntax for type uuid: ''" kalau muncul lagi -- belum reproducible, dipantau dulu di log produksi
[ ] Belum ada UI/endpoint yang mengonsumsi status submission `BLOCKED_JOB_STUCK` -- staff belum punya cara resmi lihat "job ini pernah stuck, submit ulang" selain lewat notifikasi
[ ] Lanjut ke P0 #3 (13 temuan audit ChatGPT ketiga, Bagian 170) sebagai prioritas berikutnya
[ ] 1 kerentanan Dependabot moderate terdeteksi GitHub saat push commit 5d4af07 -- belum ditelusuri, cek https://github.com/teja1945/fashion-platform/security/dependabot/1

## 175. Eksplorasi WhatsApp Cloud API untuk Notifikasi Stuck-Job -- TERBLOKIR, PENDING BANDING FACEBOOK (4 September 2026)

**Konteks:** Pelengkap ide untuk sistem eskalasi Bagian 174 -- notifikasi eskalasi (T+15/30/45 menit) saat ini cuma masuk tabel `notifications` + broadcast WebSocket in-app. Owner tenant yang jarang buka dashboard tapi rutin buka WhatsApp berpotensi tidak sadar ada job stuck. WhatsApp Cloud API (resmi dari Meta, gratis untuk service conversation, berbayar murah ~Rp20/pesan untuk business-initiated utility message) dieksplorasi sebagai kanal notifikasi tambahan.

**Progress yang SUDAH tercapai (murni di sisi Meta, BELUM ada kode diubah sama sekali):**
- Akun Facebook baru dibuat khusus keperluan teknis (terpisah dari akun Instagram branding `suarakyat1945` sesuai prinsip pemisahan kredensial proyek).
- Meta Business Portfolio "Benangrasa" berhasil dibuat.
- App Developer "Benangrasa Notifikasi" berhasil dibuat, use case "Terhubung dengan pelanggan melalui WhatsApp" sudah disetujui.
- Nomor telepon uji WhatsApp (gratis, masa aktif 90 hari) sudah didapat -- ID nomor telepon: `1354786407709524`, WhatsApp Business Account ID: `1431265412235167`.
- Nomor WhatsApp pribadi user sudah diverifikasi sebagai penerima uji coba (via OTP).

**TERBLOKIR di langkah generate token akses:** setelah user pilih scope "Hanya setujui Akun WhatsApp saat ini" (prinsip minim-akses, sesuai pola token GitHub scope minimal), akun Facebook yang baru dibuat kena restriksi otomatis oleh sistem Facebook (redirect ke `facebook.com/checkpoint`, pesan "Anda mengajukan banding", estimasi waktu peninjauan ~1 jam). Dugaan penyebab: pola aktivitas "akun baru dibuat + langsung banyak aksi teknis dalam waktu singkat" (bikin Business Manager, App Developer, WhatsApp API access) mirip pola yang di-flag sistem anti-spam/fraud Facebook -- BUKAN indikasi ada yang salah dari sisi proyek/kode.

**PENTING -- belum ada risiko kredensial:** token akses BELUM sempat digenerate sama sekali sebelum restriksi ini terjadi, jadi tidak ada token yang perlu di-rotate/cabut.

**Status saat checkpoint ini ditulis:** menunggu hasil banding Facebook (~1 jam sejak diajukan 4 September 2026, sore hari). BELUM DIKETAHUI apakah akan disetujui otomatis atau perlu tindakan lanjutan.

**Next steps aktif ditambah:**
[ ] Cek status banding akun Facebook setelah ~1 jam -- kalau disetujui, lanjut generate token akses dari titik terakhir (Penyiapan API -> Buat token akses)
[ ] Kalau banding ditolak: pertimbangkan strategi alternatif (akun Facebook yang sudah "berumur"/pernah dipakai wajar, bukan akun baru sekali pakai)
[ ] Setelah token akses berhasil didapat: integrasi ke `worker.js` (fungsi `notifyStaffList` sudah ada sebagai basis, tinggal tambah pemanggilan WhatsApp Cloud API di titik notifikasi tier 2/3, ambil nomor dari kolom `staff.phone_number` yang sudah ada)
[ ] Belum ada keputusan: notifikasi WhatsApp ini untuk SEMUA tier (1/2/3) atau cukup tier darurat (broadcast T+45 menit) saja -- perlu didiskusikan sebelum implementasi

## 176. Fix robots.txt/noindex rakyat.benangrasa.com -- SELESAI & TERUJI (4 September 2026)

**Rasa yang dipenuhi:** Rasa Ketelitian (rencana fix lama dari Bagian 168 dicek ulang ke kode/config aktual dulu sebelum eksekusi, bukan asumsi masih akurat -- ternyata memang masih akurat persis, tapi tetap diverifikasi; testing wajib ke 3 host dijalankan sebelum diklaim selesai, bukan cuma nginx -t sukses).

**Konteks:** Melanjutkan serah-terima Bagian 168 (30 Agustus 2026). Domain produksi `rakyat.benangrasa.com` (polos, akan jadi landing page publik) terblokir total dari Google karena 2 root cause: (1) route `/robots.txt` di `server.js` masih balas `Disallow: /` untuk semua host tanpa kecuali -- komentar lama yang jadi dasarnya sudah tidak valid sejak migrasi domain Bagian 159, dan (2) header `X-Robots-Tag: noindex, nofollow` di nginx ke-copy ke domain polos juga padahal seharusnya cuma untuk subdomain tenant/api.

**Fix diterapkan (ikut 6 langkah yang sudah disiapkan Bagian 168, tanpa improvisasi jalur baru):**
1. `server.js`: route `/robots.txt` diubah dinamis berdasarkan `req.hostname` -- exact-match ke `rakyat.benangrasa.com` balas `Allow: /`, host lain (subdomain apapun) tetap balas `Disallow: /`. SENGAJA tidak reuse `extractSubdomain()` dari `middleware/tenantResolver.js` karena fungsi itu pakai aturan generik ">=3 bagian hostname = ada subdomain" yang dikalibrasi untuk root domain 2-bagian biasa -- `rakyat.benangrasa.com` sendiri sudah 3 bagian sebagai root produksi, jadi aturan generiknya akan salah kalau dipakai di sini.
2. nginx `/etc/nginx/sites-enabled/rakyat.benangrasa.com`: 1 server block gabungan (domain polos + wildcard subdomain) dipecah jadi 2 server block terpisah -- domain polos tanpa `X-Robots-Tag`, wildcard subdomain tetap dengan `X-Robots-Tag noindex` seperti sebelumnya.
3. Commit `b0166a1` (perubahan `server.js` -- nginx config bukan bagian repo git, cuma di VPS).

**Insiden kecil saat eksekusi (langsung dikoreksi, dicatat biar tidak terulang):** Backup file nginx config sempat dibuat DI DALAM `/etc/nginx/sites-enabled/` (`rakyat.benangrasa.com.bak-bagian168-...`) -- ternyata nginx otomatis me-load SEMUA file di folder itu (beda dari backup kode `.bak` di direktori project yang aman diam saja), jadi config lama ikut kebaca bareng config baru dan bikin warning "conflicting server name". Terdeteksi dari `nginx -t` sebelum reload (bukan setelah, jadi tidak sempat berdampak ke produksi). **Pelajaran untuk sesi berikutnya:** backup file nginx/config sistem manapun (bukan cuma file kode project) HARUS disimpan di luar folder yang otomatis di-load (dipakai `/etc/nginx/backups/` sekarang), TIDAK boleh disimpan dengan pola sama seperti backup kode biasa.

**Testing (Langkah 5, wajib sebelum dianggap selesai):** curl ke 3 host, dibandingkan header `X-Robots-Tag` dan isi `robots.txt`:
- `rakyat.benangrasa.com` (polos): TIDAK ADA `X-Robots-Tag`, `robots.txt` = `Allow: /` -- sekarang BISA diindex Google.
- `demo.rakyat.benangrasa.com`: `X-Robots-Tag: noindex, nofollow` tetap ada, `robots.txt` = `Disallow: /` -- tidak ada regresi.
- `api.rakyat.benangrasa.com`: sama seperti demo -- tidak ada regresi.

**Status: SELESAI & TERUJI.** Sesuai catatan Bagian 168, Langkah 6 (rencana SEO/GEO lengkap: Google Search Console, Analytics, riset keyword Bahasa Indonesia, schema markup JSON-LD, dst) masih dicatat di sana sebagai next step terpisah -- BELUM mendesak, dieksekusi kapan saja setelah landing page publik benar-benar dibangun (saat ini rakyat.benangrasa.com polos belum punya halaman konten apapun, cuma robots.txt yang sudah benar).

**Next steps aktif ditambah:**
[ ] Rencana SEO/GEO lengkap (Bagian 168, Langkah 6) -- tidak mendesak, tunggu landing page publik dibangun
[ ] Lanjut ke item prioritas berikutnya: testing fungsional P1 fix POST /v1/mediators (Bagian 169), atau 13 temuan audit ChatGPT ketiga (Bagian 170) sisa P0 #3/#5/#6

## 177. Testing Fungsional P1 Fix POST /v1/mediators -- SELESAI & TERUJI (4-5 September 2026)

**Rasa yang dipenuhi:** Rasa Ketelitian (4 skenario wajib dari Bagian 169 dijalankan lewat API asli, bukan cuma baca kode; hasil skenario paling kritis -- staff_id tenant lain -- diverifikasi ulang langsung ke database, bukan cuma percaya response API; data test dibersihkan setelah selesai, tidak dibiarkan nyangkut).

**Konteks:** melanjutkan Bagian 169 (30 Agustus 2026) -- fix P1 validasi tenant isolation di POST /v1/mediators sudah ter-commit tapi 4 skenario testing wajib belum dijalankan.

**Setup data test:** dibuat 1 staff aktif di tenant `demo2` (`88a6d5f5-c43a-46cf-a97c-a70e17a4cb42`, untuk skenario cross-tenant) dan 1 staff nonaktif di tenant `demo` (`eb48e666-0569-4557-be44-5d9d890430b1`, untuk skenario staff tidak aktif) -- keduanya dihapus lagi setelah testing selesai.

**Hasil 4 skenario (semua LULUS sesuai ekspektasi):**
1. staff_id valid tenant sama (Staff Jahit Demo) -> HTTP 201, mediator berhasil dibuat dengan data lengkap.
2. staff_id dari tenant LAIN (`demo2`) -> HTTP 404 "staff tidak ditemukan atau tidak aktif". Diverifikasi ulang langsung ke `tenant_mediators` -- 0 baris, PASTI tidak ada insert yang tembus. Ini skenario PALING KRITIS karena ini persis celah yang mau dicegah fix P1 ini, dan RLS terbukti bekerja benar.
3. staff_id tidak aktif -> HTTP 404, sesuai ekspektasi.
4. staff_id tidak ada sama sekali -> HTTP 404, sesuai ekspektasi.

**Status: SELESAI & TERUJI.** Fix P1 (validasi tenant isolation di POST /v1/mediators) dari Bagian 169 sekarang terverifikasi FUNGSIONAL, bukan cuma valid secara syntax. Data test sudah dibersihkan, tidak ada residu di tenant demo/demo2.

**Next steps aktif ditambah:**
[ ] Lanjut ke sisa 13 temuan audit ChatGPT ketiga (Bagian 170): P0 #3, #5, #6, P1 #8-9 (Redis, search_path race), P2 #10 (test suite)

## 178. Cross-check ChatGPT Keempat -- Audit Live Menyeluruh (GitHub HEAD + Supabase Live + Vercel), 5 September 2026

**Konteks:** Cross-check independen keempat (pola sama Bagian 152/163/170), dilakukan setelah Bagian 174/176/177 selesai. ChatGPT cek langsung ke GitHub HEAD (commit 347deb6), Supabase live schema, dan Vercel deployment status -- bukan cuma baca CHECKPOINT.md.

**Apresiasi temuan lama yang dikonfirmasi SELESAI:**
- P0 lock down POST /v1/events -- selesai (tapi P1 baru terbuka: staff_id terbukti login di tenant, TAPI belum tervalidasi role/assigned_stage yang sesuai event_type/stage-nya -- misal staff QC secara konsep masih bisa kirim event yang harusnya cuma boleh stage lain).
- Transaction bug stage-submissions (P0 #2, Bagian 174) -- dikonfirmasi 🟢 secara kode dan testing tercatat, TAPI ChatGPT menekankan belum akan disebut 100% selesai sebelum integration test masuk CI (bukan cuma manual E2E).
- RLS notifications yang sempat error saat testing Bagian 174 -- ChatGPT eksplisit bilang "historical failure ≠ current failure", TIDAK dihitung sebagai bug aktif karena sudah diperbaiki dan diverifikasi Bagian 177. Dicatat di sini sebagai bukti audit ini membaca histori dengan benar, bukan asal flag ulang.

**TEMUAN BARU yang perlu ditindaklanjuti (belum pernah dicatat sebelumnya):**

1. **Stage invariant belum dipaksa (P0):** `stage_quantity_submissions.stage_key` dan `production_jobs.current_stage` bisa berbeda saat confirm. Aturan bisnis yang seharusnya dipaksa: `submission.stage_key == production_jobs.current_stage` PERSIS pada saat confirmation -- kalau tidak, submission lama/stale secara teori bisa dipakai untuk menggerakkan state yang salah.

2. **Inventory semantics (P0/P1):** tabel `fabric_inventory` dan `inventory_ledger` sudah ada dengan 4 jenis movement (RESERVED, STOCK_CONSUMED, RELEASED, RESTOCKED), tapi dicurigai semuanya diperlakukan sebagai operasi "kurangi stok" yang sama -- padahal secara logika beda arah (RESERVED pindah AVAILABLE->RESERVED, STOCK_CONSUMED konsumsi beneran, RELEASED lepas reservasi, RESTOCKED nambah stok). Kalau salah ditangani, angka inventory bisa KELIHATAN jalan tapi stock truth-nya salah. **PERINGATAN EKSPLISIT: jangan bikin dashboard inventory sebelum invariant ini diperbaiki dan diverifikasi.**

3. **Redis session TTL bug (P1):** `createSession()` bikin 2 key -- `session:<token>` dan `staff_sessions:<tenant>:<staff>`. `touchSession()` cuma perpanjang TTL `session:<token>`, TIDAK ikut perpanjang `staff_sessions:*`. Akibatnya kalau staff aktif lama (misal 8+ jam terus-terusan touch session), `staff_sessions:*` bisa expired duluan walau token masih hidup -- `revokeStaffSessions()` jadi kehilangan daftar token yang seharusnya di-revoke.

4. **`db.js` search_path race condition (P1, KEMUNGKINAN BESAR terkait known issue Bagian 174):** `pool.on("connect", (client) => { client.query("SET search_path...").catch(...) })` -- callback ini TIDAK di-await, jadi ada kemungkinan request pertama yang pakai koneksi baru itu jalan SEBELUM search_path selesai di-set. Ini cocok dengan pola error transient `invalid input syntax for type uuid: ""` yang tercatat di Bagian 174 (muncul cuma sekali tepat setelah restart pm2, saat banyak koneksi baru dibikin bareng). **BELUM DIVERIFIKASI langsung -- baru dugaan kuat berdasarkan kecocokan pola, perlu investigasi lanjutan sebelum dianggap pasti.**

5. **Schema reproducibility (masalah besar, bukan sekadar teknis kecil):** live database sekarang punya 31 tabel public, banyak yang TIDAK ada di file schema utama repo (`fashion_platform_schema_v2.sql`) -- termasuk `tenant_mediators`, `mediator_backups`, `mediator_reassignment_log`, `discrepancy_cases`, `discrepancy_thread_messages`, `discrepancy_thread_photos`, `notifications`, `stage_quantity_submissions`, dll. Pertanyaan kunci: "kalau database mati total dan cuma punya repo, apakah bisa dibangun ulang production secara deterministic?" -- jawaban saat ini BELUM BISA dipastikan YES.

6. **Automated testing masih kosong (P0 gap):** `package.json` masih `"test": "echo \"Error: no test specified\" && exit 1"`. Ada `test-e2e.js`/`test-e2e-step2.js` tapi itu manual E2E yang memutasi database nyata, BUKAN automated regression suite yang bisa jalan di CI.

7. **Vercel masih BLOCKED:** project `fashion-platform` di Vercel, deployment production terbaru `readyState = BLOCKED`. Belum ada production customer-facing web application yang live (`framework: null`). Backend jauh lebih maju daripada frontend.

8. **Performance advisor Supabase (bukan security, tapi perlu dibereskan sebelum scale):** banyak unindexed foreign keys (`production_events.production_job_id`, `production_jobs.order_id`, `inventory_ledger.fabric_inventory_id`, `stage_quantity_submissions.submitted_by_staff_id`, `discrepancy_cases.submitter_staff_id`, dst) dan banyak RLS policy masih pakai `current_setting(...)` langsung di predicate (disarankan dibungkus `(select current_setting(...))` untuk optimisasi query planner).

9. **In-memory rate limiter di `ingestion.js`:** ada `rateBuckets = new Map()` dengan `RATE_LIMIT_PER_SEC = 1000` yang terpisah dari Express rate limiter global dan Redis rate limiter. Bukan P0 sekarang (masih single instance), tapi tidak konsisten dengan keputusan sebelumnya untuk pindahkan state penting ke Redis -- kalau nanti multi-instance, limit ini jadi tidak lagi global per instance.

**Skor dari audit ini (evenhandedness, keempat sudut disebut jujur termasuk yang masih 🔴):** Backend architecture 7.5/10, Security architecture 7.5/10, Data integrity 5.5/10, Testing 3.5/10, Production readiness 5/10.

**Urutan kerja yang disarankan ChatGPT (dicatat sebagai referensi, BELUM disepakati sebagai keputusan final proyek):** (1) fix stage_key invariant, (2) fix inventory semantics, (3) fix Redis session TTL, (4) fix db.js search_path race, (5) schema/migration reproducibility, (6) integration test suite, (7) OWASP ZAP, (8) k6 load test, (9) FK indexes, (10) baru frontend/customer flow, (11) baru dashboard owner. Penekanan eksplisit: **jangan lompat ke dashboard owner atau fitur besar lain sebelum P0/P1 data-integrity + reproducibility + testing ditutup dulu.**

**Status: TEMUAN DICATAT, BELUM ADA YANG DIEKSEKUSI.** Sengaja diparkir dulu di akhir sesi panjang (Bagian 174-177 sudah menguras banyak waktu/fokus) -- dilanjutkan di sesi berikutnya yang lebih segar, bukan dipaksa lanjut sekarang.

**Next steps aktif ditambah (urutan mengikuti saran ChatGPT di atas, bisa didiskusikan ulang sebelum eksekusi):**
[ ] Investigasi db.js search_path race condition -- cek dulu berapa banyak tempat pool.connect() dipanggil langsung sebelum desain fix (kemungkinan butuh refactor ke wrapper function, bukan cuma 1 baris, karena Session Pooler Supabase sudah pernah bikin ALTER ROLE approach tidak cukup -- lihat komentar db.js)
[ ] Fix stage_key invariant (submission.stage_key harus == production_jobs.current_stage saat confirm)
[ ] Fix inventory semantics (4 jenis movement tidak boleh diperlakukan sama)
[ ] Fix Redis session TTL (staff_sessions:* harus ikut diperpanjang di touchSession())
[ ] Schema/migration reproducibility -- live DB 31 tabel vs schema repo yang jauh lebih tua
[ ] Automated test suite masuk CI (bukan cuma manual E2E)
[ ] FK index + RLS init-plan performance cleanup (tidak urgent, tapi jangan ditunda sampai scale)

## 179. Update Bagian 175 -- Diagnosa Lebih Jelas: Kode SMS Verifikasi Akun Developer Tidak Kunjung Masuk (5 September 2026)

**Konteks:** Melanjutkan Bagian 175 (WhatsApp Cloud API terblokir pending banding Facebook). Setelah investigasi lebih lanjut hari ini, ternyata masalahnya BUKAN "banding akun Facebook" seperti dugaan awal Bagian 175 -- akun Facebook itu sendiri TERBUKTI AMAN dan normal (dicek langsung, profil "Muhamad Teja" bisa diakses penuh, tidak ada banner penangguhan, "Tidak tersedia postingan" -- konsisten sebagai akun baru yang wajar).

**Diagnosa yang benar:** Yang bermasalah adalah **akun App Developer** (`developers.facebook.com`, app "Benangrasa Notifikasi"), BUKAN akun Facebook pribadi. Halaman developer terus menampilkan "Perlu konfirmasi akun -- Kami melihat aktivitas yang tidak biasa di akun developer ini", dan proses konfirmasinya WAJIB kirim kode 6 digit ke nomor `+62857...` -- kode ini TIDAK KUNJUNG MASUK meski sudah diminta berkali-kali (SMS maupun opsi ganti nomor, yang ternyata tetap butuh verifikasi SMS ke nomor lama dulu sebelum bisa ganti).

**Sempat salah jalur:** Asisten AI Meta Business (chatbot resmi) awalnya melaporkan "akun Facebook Anda ditangguhkan sejak 4 September 2026 karena Standar Komunitas" -- info ini TIDAK AKURAT/tidak sinkron dengan kondisi nyata akun Facebook yang ternyata aman. Kemungkinan chatbot itu membaca status yang sudah basi atau salah asosiasi. Jangan percaya penuh laporan status dari chatbot ini tanpa verifikasi manual ke akun aslinya.

**Instruksi resmi yang didapat dari chatbot Meta (bagian yang akurat & berguna):** "Tunggu hingga 24 jam jika Anda telah melakukan terlalu banyak permintaan kode dalam waktu singkat." Kemungkinan besar sistem Meta sudah menerapkan rate-limit/cooldown ke nomor `+62857...` karena sudah diminta berulang kali dalam waktu singkat hari ini (4-5 September 2026).

**Keputusan: DITUNDA sampai besok (6 September 2026 atau lebih), JANGAN dicoba lagi hari ini.** Mencoba lagi sekarang berisiko mereset ulang hitungan cooldown 24 jam, membuat proses makin lama. Saat dicoba lagi nanti, WAJIB: coba SEKALI saja, tunggu penuh beberapa menit tanpa refresh/klik ulang berkali-kali sebelum menyerah.

**Progress yang TETAP AMAN tersimpan (tidak berubah dari Bagian 175):** Meta Business Portfolio "Benangrasa", App Developer "Benangrasa Notifikasi", use case WhatsApp, nomor telepon uji (`1354786407709524`), WhatsApp Business Account ID (`1431265412235167`), nomor penerima uji sudah terverifikasi. Token akses BELUM sempat digenerate -- tidak ada risiko kredensial.

**Next steps aktif diperbarui (menggantikan catatan lama di Bagian 175 soal "cek status banding" -- sekarang jelas bukan itu masalahnya):**
[ ] Besok (atau kapan saja setelah jeda 24 jam wajar): coba SEKALI lagi minta kode SMS verifikasi akun developer ke `+62857...`, di jam pagi/jaringan lebih stabil
[ ] Kalau SMS tetap tidak masuk setelah 1x percobaan wajar besok: pertimbangkan opsi lain -- ganti ke nomor kontak dari provider berbeda (kalau ada nomor cadangan), atau hubungi Meta Business Help Center resmi lewat jalur berbeda (bukan chatbot AI yang terbukti kurang akurat)
[ ] Setelah konfirmasi akun developer berhasil: lanjut generate token akses (titik terakhir sebelum insiden ini, di halaman Penyiapan API)

## 180. Seri B #1 Dipost -- Video Progress Sistem Eskalasi Job Stuck (6 September 2026)

**Konten:** Reels pertama Seri B ("Progress Bikin Produk"), angle sistem eskalasi otomatis
untuk job produksi yang macet (lanjutan cerita Seri A #1). 5 slide, narasi TTS ~37 detik,
caption dengan SEO/GEO keyword konveksi + kontak lengkap (portfolio/email/WA).

**Proses produksi:** Outline dibuat via Canva presentation-outline (5 slide), digenerate,
diedit teksnya per slide manual (find_and_replace_text) supaya sesuai naskah dan data
kontak asli. Export video awal dari Canva CACAT -- cuma slide 1 yang punya visual,
sisanya layar hitam total dari detik ~5 sampai akhir walau audio tetap jalan penuh 36 detik.
Diperbaiki dengan cara: user download manual ke-5 slide sebagai JPG dari Canva
(karena sandbox tidak punya akses network ke domain Canva), lalu dirakit ulang jadi
video pakai ffmpeg (tiap slide jadi klip statis dengan durasi proporsional ke bagian
narasi masing-masing), digabung, dan audio asli dipasang ulang tanpa diubah.

**Status: DIPOST APA ADANYA, meski ada 2 catatan kualitas yang diketahui SEBELUM post:**
1. Diagram ilustrasi di slide 3 (Solusi) mengandung teks gibberish/acakan dari
   library ilustrasi Canva (contoh pola: label acak pada figure orang) -- tidak diganti,
   keputusan sadar user untuk lanjut post karena masih tahap belajar.
2. Video tetap format landscape 1920x1080, TIDAK dikonversi ke vertikal 9:16 sebelum
   post -- Instagram kemungkinan crop/beri border.

**Pelajaran untuk sesi berikutnya:**
- SEBELUM generate design berisi ilustrasi figure/orang dari Canva AI, cek dulu apakah
  ada teks/label di dalam ilustrasi itu -- rawan gibberish, ganti ke diagram tanpa
  teks kalau ketemu.
- Kalau tujuan akhir konten adalah Reels/Stories, request design_type atau resize
  ke rasio vertikal (1080x1920) SEJAK AWAL generate, bukan setelah semua slide selesai
  diedit -- resize belakangan berisiko merusak layout yang sudah pas.
- Export video dari Canva untuk presentation multi-halaman TERBUKTI tidak reliable
  (bug: hanya page pertama yang ter-render jadi video, sisanya blank). Kalau butuh
  video, rencanakan dari awal untuk export tiap page sebagai PNG lalu rakit manual
  via ffmpeg -- jangan andalkan native "export as MP4" dari Canva untuk presentation
  bertahap begini.

**Next steps aktif ditambah (rencana konten 3-4 hari ke depan, urutan A-B-A-B):**
[ ] Seri A #2 (giliran berikutnya): angle "stok kain di catatan vs kondisi fisik gudang
    beda" -- nyambung dari Seri B #1 (job stuck) ke masalah data-integrity berikutnya
    yang relate ke pemilik konveksi. Hook kasar: "Stok kain di sistem bilang ada 50 meter.
    Pas dicek ke gudang... beda."
[ ] Seri B #2 (3-4 hari setelah A #2): angle progress pembenahan inventory semantics
    (4 jenis movement: RESERVED/STOCK_CONSUMED/RELEASED/RESTOCKED tidak boleh
    diperlakukan sama -- temuan Bagian 178 #2) -- nyambung sebagai "jawaban" dari
    masalah yang diangkat di Seri A #2, sama seperti pola B #1 menjawab A #1
[ ] Sebelum eksekusi konten di atas: terapkan pelajaran dari Seri B #1 (Bagian 180)
    -- cek ilustrasi Canva bebas dari teks gibberish, dan resize ke rasio vertikal
    9:16 SEJAK AWAL generate kalau tujuan akhirnya Reels
[ ] Pantau performa Seri B #1 (views/engagement) sebagai baseline pembanding ke Seri A #1

## 181. Fix Race Condition search_path (P1 #9) + Bug Tambahan Pool Size Melebihi Limit Supabase -- SELESAI & TERUJI (6 September 2026)

**Rasa yang dipenuhi:**
- **Rasa Ketelitian** -- fix diverifikasi berlapis, bukan cuma "kelihatannya benar": (1) tes terisolasi 10 koneksi paralel manggil fungsi pgcrypto yang butuh search_path benar, (2) restart pm2 asli + pantau log produksi 90 detik dengan interval worker diketahui persis (gap monitor 10 detik, stuck-job monitor 60 detik) baru dianggap bersih, (3) proses tes ini sendiri menemukan bug KEDUA yang tidak diduga (pool size melebihi limit Supabase) yang kalau tidak ketauan sekarang baru muncul nanti pas beban tinggi.
- **Rasa Keamanan** -- race condition search_path berisiko bikin query tanpa schema-qualify salah resolve ke schema lain di koneksi yang baru dibuat (celah waktu antara connect dan SET search_path selesai); fix pool size mencegah error fatal EMAXCONNSESSION yang bisa jatuhin availability saat lonjakan trafik/restart.

**Konteks masalah:** Diagnosis lanjutan dari known issue Bagian 174 (error transient `invalid input syntax for type uuid: ""` pasca-restart pm2, dicurigai kuirk search_path). Root cause: `pool.on("connect", client => client.query("SET search_path..."))` di `db.js` TIDAK di-`await` -- pool langsung anggap koneksi baru "siap pakai" begitu event `connect` selesai jalan, padahal query SET search_path-nya bisa masih pending. Perbaikan lewat `await` di 30+ titik pemanggilan `pool.connect()` (server.js, worker.js, versioning.js, ingestion.js, dll) dinilai terlalu berisiko untuk solo dev -- rawan kelewat 1 titik, bug balik lagi diam-diam.

**Implementasi:**
1. `db.js`: tambah `options: "-c search_path=public,extensions"` di config `Pool` -- search_path diset sebagai startup parameter Postgres saat physical connection dibuat (sebelum koneksi dianggap ready sama sekali), bukan lewat query terpisah setelahnya. Berlaku otomatis ke SEMUA pemanggilan `pool.connect()` tanpa perlu edit satupun titik lain.
2. Handler lama `pool.on("connect", ...)` SENGAJA dibiarkan sebagai jaring pengaman, belum dihapus -- baru dipertimbangkan dihapus setelah fix baru terbukti stabil di produksi dalam jangka waktu lebih panjang.
3. **Bug tambahan ditemukan saat testing:** `max: 20` di config Pool ternyata melebihi limit asli Supabase Session Pooler (`pool_size: 15`) -- kebukti langsung lewat error fatal `EMAXCONNSESSION` pas tes 30 koneksi paralel. Diperbaiki jadi `max: 10` (sisa ruang untuk script one-off seperti check.js/cleanup.js/reset-job.js yang jalan manual sambil server.js hidup).

**Testing:**
- Tes terisolasi: 10 `pool.connect()` paralel, masing-masing langsung manggil `crypt()` (butuh schema `extensions`) tanpa jeda -- 10/10 berhasil, 0 gagal.
- Verifikasi produksi: `pm2 flush` + `pm2 restart` + pantau log 90 detik (mencakup ~9 tick gap monitor + 1 tick stuck-job monitor) -- bersih, tidak ada error.
- File tes (`test-searchpath-race.js`) dihapus dari VPS setelah selesai, tidak disimpan sebagai residu.

**Status: SELESAI & TERUJI.** Ter-commit di repo private commit `f69777a`, tersinkron ke repo public commit `edf471a`. P1 #9 (search_path race, dari 13 temuan audit ChatGPT ketiga Bagian 170) sekarang **DITUTUP**. Known issue transient uuid error dari Bagian 174 dianggap teratasi oleh fix ini, tapi TETAP DIPANTAU beberapa hari ke depan di log produksi sebelum diklaim 100% tuntas (belum pernah berhasil direproduksi ulang secara sengaja sebelumnya, jadi tidak ada baseline "before" yang pasti sama persis).

**Next steps aktif ditambah:**
[ ] Pantau log produksi beberapa hari ke depan -- pastikan error transient uuid tidak muncul lagi sama sekali
[ ] Pertimbangkan hapus handler `pool.on("connect")` lama setelah fix baru terbukti stabil dalam jangka lebih panjang
[ ] Lanjut ke P1 #8 (Redis session TTL, touchSession tidak perpanjang staff_sessions:*) sebagai next item dari urutan prioritas Bagian 178

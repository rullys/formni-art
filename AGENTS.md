<!-- BEGIN:onivor-push-discipline -->
# Belum ter-push berarti belum ada

GitHub `origin` adalah satu-satunya sumber kebenaran untuk kode. Bukan disk,
bukan Linear, bukan ringkasan sesi.

## Aturannya

Jangan pernah mengakhiri giliran dengan kerja yang belum ter-push.

Sebelum berhenti, dua perintah ini harus lolos:

```
git status --porcelain            # harus KOSONG
git rev-list --count @{u}..HEAD   # harus 0
```

Kalau salah satu tidak memenuhi, kerjanya belum selesai.

Push BUKAN merge. Cabang boleh menggantung tanpa PR selama-lamanya, dan itu
tetap jauh lebih baik daripada perubahan yang hanya ada di satu disk. Kalau ragu
kerjanya layak dirilis, tetap push: yang diputuskan nanti adalah nasib
cabangnya, bukan keberadaannya.

Berlaku untuk setiap agen, setiap host, setiap sesi. Termasuk kerja setengah
jadi, eksperimen, dan kandidat yang mungkin dibuang. Terutama itu, karena kerja
setengah jadi justru yang paling mungkin tidak punya salinan lain.

## Linear tidak boleh mendahului GitHub

Jangan menandai isu Done kalau kodenya belum ter-push. Status Done tanpa commit
membuat orang berikutnya percaya sesuatu sudah mendarat padahal tidak.

## Satu pengecualian, dan hanya satu

Rahasia. Kunci API, token, isi `.env`. Itu tidak pernah di-commit, dan tidak
pernah "diselamatkan" dengan cara ini.

## Tata letak di mesin Onivor: satu proyek, satu direktori

Akar repo berbeda per mesin: `~/repos/onivor` di gateway MBP, `~/Projects` di
MBP founder. Di bawah akar itu, `<akar>/<proyek>` adalah satu clone penuh per
proyek, dan `<proyek>` sama persis dengan nama repo di GitHub. Tidak ada
worktree, salinan kedua, atau clone bernama lain di level atas akar (di MBP
founder, folder pribadi yang bukan repo boleh ada di samping proyek).

Clone proyek berdiri di `dev` kalau repo itu punya `dev`, kalau tidak di cabang
defaultnya, dan tetap bersih saat tidak sedang dipakai. Tidak ada perubahan yang
menginap di dalamnya.

Cabang kerja yang butuh direktori sendiri (rilis berjalan bersamaan dengan
fitur, atau dua agen di repo yang sama) memakai worktree, dan worktree hidup di
`<akar>/.worktrees/<proyek>/<nama>`:

```
git -C <akar>/<proyek> worktree add <akar>/.worktrees/<proyek>/<nama> <cabang>
```

Begitu PR-nya merged, worktree itu dihapus dengan `git worktree remove`.
Cabang lokalnya boleh tetap ada. Worktree bukan tempat menyimpan apa pun:
semua yang penting sudah ada di GitHub sebelum ia dihapus, dan itu persis yang
dijamin aturan di atas.

Kenapa tersembunyi di `.worktrees`, bukan di dalam direktori proyek: worktree
yang bersarang di dalam proyek ikut tersapu vitest, tsc, eslint, dan pemindai KB
milik proyek induknya. Kenapa bukan di level atas: 60 worktree yang ditemukan
2 September 2026 menyimpan hampir 40 GB `node_modules` yang tergandakan, dan
tidak ada cara membedakan sekilas mana clone proyek dan mana sisa kerja yang
PR-nya sudah merged sebulan sebelumnya.

## Kenapa aturan ini ada

16 Agustus 2026, ONI-2550 ditandai Done di Linear. Kodenya, 2.651 baris termasuk
tabel schema baru dan dua rute HTTP unsubscribe, tidak pernah di-commit. Ia
bertahan sebagai perubahan tak ter-commit di satu disk selama 16 hari, dan baru
ketahuan saat menghitung worktree untuk urusan yang sama sekali lain. Satu
`git checkout .` yang tidak sengaja akan menghapusnya tanpa jejak.

Aturan "GitHub adalah SoT" SUDAH ADA waktu itu. Ia gagal karena dua hal, dan
keduanya yang diperbaiki di sini:

1. Ia hidup di memori satu agen, bukan di repo. Agen lain di host lain tidak
   pernah membacanya.
2. Ia menyebut DI MANA kebenaran tinggal, bukan KAPAN wajib menulis ke sana.
   "GitHub adalah SoT" tidak melarang siapa pun berhenti tanpa push.

Salinan kanonik untuk manusia, dan yang menang kalau berbeda:
https://app.notion.com/p/3cff32bdb8f081a8b087c23c3aea2365
Pendeteksinya berjalan tiap jam di gateway (`unpushed-watch.py`) dan juga
memeriksa tata letak di atas.
<!-- END:onivor-push-discipline -->

<!-- BEGIN:onivor-linear-method -->
# Linear Method Onivor: scrumban, cycles, dan Active board

Linear adalah satu-satunya tempat status eksekusi hidup. Tidak ada kerja
tanpa isu, dan tidak ada isu yang statusnya menyimpang dari kenyataan.

## Dua tampilan, satu aturan

- **Cycles** (pola sprint): kerja yang disepakati untuk periode berjalan.
  Founder menyusun urutan di awal cycle; yang tidak selesai bergulir ke cycle
  berikutnya dengan catatan kenapa.
- **Active board** (kanban): Backlog, Todo, In Progress, In Review, Done,
  Canceled. **Yang paling atas di tiap kolom dikerjakan lebih dulu.** Urutan
  kolom adalah keputusan prioritas founder; agen tidak memilih tugas yang lebih
  menarik di bawahnya.

## Alur status, dan siapa yang menggerakkannya

1. Backlog paling atas -> Todo, saat masuk cycle.
2. Todo paling atas -> In Progress, oleh agen yang mulai mengerjakan, SEBELUM
   baris pertama ditulis. Satu agen, satu In Progress.
3. In Progress -> In Review, saat PR sudah bersih dari reviewer lintas model
   (Greptile, Codex, atau yang berlaku di repo itu). **In Review milik
   founder**: ia yang mereview dan menutup, agen tidak melewatinya.
4. -> Done hanya kalau kodenya ter-push dan merged (Done tanpa commit adalah
   kebohongan, lihat blok push discipline). -> Canceled dengan alasan.

## Temuan bukan pekerjaan sampingan

Apa pun yang ditemukan saat bekerja (bug, lubang keamanan, salinan yang
salah, tes yang bohong) masuk sebagai isu Backlog baru, bertanggal, dengan
prioritas (Urgent, High, Medium, Low) dan bukti (file:baris, angka, tautan).
Jangan dikerjakan diam-diam di PR yang sedang berjalan, dan jangan disimpan di
kepala.

## Isu yang baik

- Ditulis sebagai isu, bukan user story: keadaan sekarang, kenapa itu masalah,
  arah perbaikan, definisi selesai.
- Cabang mengikuti nama dari Linear; PR menyebut isunya (`Closes ONI-n`).
- Satu isu satu hal. Isu yang membengkak dipecah, bukan diperpanjang.
- Backlog dijaga kecil: yang tidak akan dikerjakan dalam dua cycle diarsipkan
  atau di-cancel dengan alasan, bukan dibiarkan menua.

## Kenapa

Founder memantau eksekusi seluruh portofolio dari kanban Linear, sering dari
ponsel. Kanban yang jujur hanya mungkin kalau setiap agen menggerakkan
statusnya sendiri pada saat yang tepat. Aturan ini (ketukan founder 4 Sep
2026) melengkapi blok push discipline: GitHub memegang kode, Linear memegang
status, dan keduanya tidak boleh saling mendahului.
<!-- END:onivor-linear-method -->

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

`~/repos/onivor/<proyek>` adalah satu clone penuh per proyek, dan `<proyek>`
sama persis dengan nama repo di GitHub. Level atas `~/repos/onivor` hanya
berisi direktori proyek itu: tidak ada worktree, salinan kedua, berkas lepas,
atau direktori kosong di sana.

Clone proyek berdiri di `dev` kalau repo itu punya `dev`, kalau tidak di cabang
defaultnya, dan tetap bersih saat tidak sedang dipakai. Tidak ada perubahan yang
menginap di dalamnya.

Cabang kerja yang butuh direktori sendiri (rilis berjalan bersamaan dengan
fitur, atau dua agen di repo yang sama) memakai worktree, dan worktree hidup di
`~/repos/onivor/.worktrees/<proyek>/<nama>`:

```
git -C ~/repos/onivor/<proyek> worktree add ~/repos/onivor/.worktrees/<proyek>/<nama> <cabang>
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

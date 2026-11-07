# ACEVIDS — trial video editors

Satu repo untuk mencoba dan menggabungkan alat edit video berbasis agen, plus
**aplikasi web untuk mengunggah video besar, mengeceknya, dan menyusunnya menjadi
satu video hasil edit**.

```
ACEVIDS/
├── app/                 server FastAPI + UI: upload chunked, cek, timeline, render
├── scripts/             setup.sh, run.sh, upload.py (upload besar dari terminal)
├── tests/smoke.py       uji ujung-ke-ujung (upload → cek → render)
├── .claude/skills/      skill agen yang di-vendor dari 9 repo rujukan + skill ACEVIDS
├── docs/                hasil pelajaran 9 repo & alur kerja
└── third_party/         lisensi dan berkas pendukung dari repo rujukan
```

## Mulai cepat

```bash
bash scripts/setup.sh          # ffmpeg, sox, Python deps (fastapi, scenedetect, auto-editor, yt-dlp)
bash scripts/run.sh            # http://localhost:8080
```

Lalu buka `http://localhost:8080`:

1. **Media** – tarik file ke kotak upload. File dipotong per 16 MB, dikirim 3
   potongan sekaligus, dan bisa **dilanjutkan** bila koneksi putus (pilih ulang
   file yang sama). Untuk file puluhan GB dari terminal:
   ```bash
   python3 scripts/upload.py http://HOST:8080 /path/video.mp4
   # atau satu permintaan:
   curl -T video.mp4 "http://HOST:8080/api/uploads/direct?filename=video.mp4"
   ```
2. **Cek** – begitu upload selesai server otomatis: `ffprobe` (codec, resolusi,
   fps, VFR/HDR/rotasi), cek decode (60 detik pertama; tombol *Cek penuh* untuk
   seluruh file), contact sheet, **proxy 480p** untuk preview di browser,
   deteksi **keheningan**, pengukuran **loudness** (EBU R128), dan deteksi
   **adegan** (PySceneDetect, fallback filter `scene` ffmpeg) dengan thumbnail.
3. **Susun** – tentukan IN/OUT dari pemutar atau klik adegan, tambah ke
   timeline; **Potong otomatis** memecah bagian bersuara (gaya auto-editor).
   Atur urutan, kecepatan, format (YouTube 16:9, Reels 9:16, persegi, ikut
   sumber), transisi (xfade), buang keheningan, normalisasi LUFS, judul.
4. **Render** – klik *Render jadi satu video*; progres tampil, hasil bisa
   diputar dan diunduh (`/api/projects/{id}/output`).

Uji cepat tanpa browser:

```bash
python3 tests/smoke.py
```

## Pipeline render

Lihat [docs/ALUR-KERJA.md](docs/ALUR-KERJA.md). Ringkasnya: tiap klip
di-render ke perantara dengan frame seragam (segmen hening dibuang di sini),
lalu digabung: tanpa transisi memakai concat demuxer (video disalin, cepat);
dengan transisi memakai rantai `xfade` + `acrossfade`. `loudnorm` dan judul
(`drawtext`) diterapkan di tahap gabung.

## Sembilan repo yang dipelajari

Ringkasan lengkap, lisensi, dan cara memakainya ada di
[docs/REFERENSI-TOOLS.md](docs/REFERENSI-TOOLS.md).

| Repo | Peran di ACEVIDS |
|---|---|
| [kajisho5/ffmpeg-skill](https://github.com/kajisho5/ffmpeg-skill) | Di-vendor ke `.claude/skills/ffmpeg-skill` (42 alat ffmpeg untuk agen). Pola *probe → edit → check → look* dipakai di pipeline. |
| [nateherkai/herk-edit-kit](https://github.com/nateherkai/herk-edit-kit) | Skill `cut-silences`, `cut-mistakes`, `edit-video`, `video-storytelling`, `short-form-edit` di-vendor; playbook editing di `third_party/herk-edit-kit`. |
| [0xsline/OpenChatCut](https://github.com/0xsline/OpenChatCut) | Editor desktop agen-native (AGPL). Tidak di-vendor; cara menghubungkan via MCP di dokumen. |
| [remotion-dev/skills](https://github.com/remotion-dev/skills) | Motion graphics React; dipasang dengan `npx skills add remotion-dev/skills`. |
| [WyattBlue/auto-editor](https://github.com/WyattBlue/auto-editor) | 5 skill di-vendor; CLI dipasang lewat pip; konsep *margin* dipakai di "Potong otomatis". |
| [FireRedTeam/FireRed-OpenStoryline](https://github.com/FireRedTeam/FireRed-OpenStoryline) | Agen cerita (LLM/VLM). Skill install/use di-vendor; butuh API LLM. |
| [echris6/motion-video-kit](https://github.com/echris6/motion-video-kit) | Skill `business-motion-film` di-vendor (iklan motion, Gauntlet review). |
| [mifi/editly](https://github.com/mifi/editly) | NLE deklaratif Node (butuh headless-gl); dicatat sebagai alternatif render. |
| [Breakthrough/PySceneDetect](https://github.com/Breakthrough/PySceneDetect) | Dipakai langsung oleh server untuk deteksi adegan. |

## Lingkungan cloud (Claude Code)

Environment **ACEVIDS** sudah dibuat dengan setup script yang memasang semua
dependensi di atas; lihat `scripts/setup.sh`. Session Claude di repo ini otomatis
mendapat skill di `.claude/skills/`.

## Lisensi

Kode ACEVIDS sendiri: MIT. Berkas yang di-vendor tetap di bawah lisensi
masing-masing, lihat [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

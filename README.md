# MinIO Docker Compose

Setup MinIO dengan Docker Compose yang mendukung dua environment terpisah: **local** (development di mesin sendiri) dan **production** (server).

## Struktur file

```
.
├── docker-compose.yml   # definisi service, sama untuk semua environment
├── .env.example         # template variabel, aman untuk di-commit
├── .env                  # kredensial untuk environment local (default, jangan di-commit)
├── .env.production       # kredensial untuk environment production (jangan di-commit)
└── data/                  # data MinIO (volume), jangan di-commit
```

Semua variabel (`MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, port, `TZ`) diambil lewat variable substitution `${...}` di `docker-compose.yml`. `.env` adalah file default yang otomatis dipakai Docker Compose untuk environment **local**. Untuk **production**, environment ditentukan secara eksplisit lewat flag `--env-file .env.production` — supaya tidak mungkin ke-run production tanpa sengaja.

`.env` dan `.env.production` sudah di-ignore lewat `.gitignore` supaya kredensial tidak ikut ter-commit. Gunakan `.env.example` sebagai referensi variabel apa saja yang dibutuhkan.

## Menjalankan environment local

```bash
docker compose up -d
```

Default local: user/password `root` / `root`, port API `9000`, console `9001`.

## Menjalankan environment production

```bash
docker compose --env-file .env.production up -d
```

**Sebelum deploy ke production**, pastikan `.env.production` sudah berisi `MINIO_ROOT_PASSWORD` yang kuat dan unik (jangan pakai nilai default/contoh).

## Perintah lain yang berguna

```bash
# local (pakai .env default)
docker compose logs -f minio
docker compose down
docker compose ps

# production (selalu pakai --env-file eksplisit)
docker compose --env-file .env.production logs -f minio
docker compose --env-file .env.production down
docker compose --env-file .env.production ps
```

## Catatan

- Karena `.env` sudah ada sebagai default, `docker compose up -d` tanpa flag akan selalu menjalankan environment **local**. Untuk production, wajib selalu tulis `--env-file .env.production` secara eksplisit — jangan pernah rename `.env.production` jadi `.env` di mesin dev, supaya tidak ada risiko production ke-run tanpa sadar.
- Kredensial production sebaiknya tidak disimpan lama-lama di laptop dev. Idealnya `.env.production` hanya benar-benar ada di server production (di-copy langsung lewat scp/secret manager), dan di repo dev cukup `.env.production.example` sebagai referensi.
- Rotate `MINIO_ROOT_PASSWORD` production secara berkala, terutama kalau pernah ter-paste ke chat, log, atau tempat lain yang tidak terenkripsi.
- Kalau local dan production perlu dijalankan bersamaan di mesin yang sama, tambahkan project name berbeda agar container tidak bentrok, misalnya:
  ```bash
  docker compose -p minio-local up -d
  docker compose -p minio-prod --env-file .env.production up -d
  ```
- Volume `./data` menyimpan seluruh data MinIO. Jangan dihapus kecuali memang ingin reset data.

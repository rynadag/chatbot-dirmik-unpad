# 🤖 Chatbot Direktorat Akademik Unpad

Selamat datang di repositori resmi **Chatbot Direktorat Akademik Universitas Padjadjaran**. Proyek ini merupakan asisten virtual interaktif berbasis **RAG (Retrieval-Augmented Generation)** yang dirancang untuk menjadi pusat informasi (*Knowledge Base*) cerdas yang melayani sivitas akademika seputar layanan, administrasi, dan informasi Direktorat Akademik.

Sistem ini dibangun dengan arsitektur *microservices* yang memadukan keunggulan **Next.js** (Antarmuka Web), **Express.js** (Manajemen API & Autentikasi JWT), dan **FastAPI/Python** (Mesin Kecerdasan Buatan & Pemrosesan Vektor ChromaDB).

---

## ✨ Ikhtisar Fitur Utama

- 💬 **Chatbot AI Kontekstual**: Memanfaatkan Large Language Model (LLM) melalui **Groq API** (atau **Ollama** lokal) yang dipadukan dengan dokumen referensi internal (RAG) untuk memberikan jawaban akurat seputar informasi akademik, pendaftaran, kurikulum, dan prosedur administratif.
- 🌐 **Dukungan Multibahasa**: Chatbot mendukung percakapan dalam **Bahasa Indonesia** maupun **Bahasa Inggris** yang dapat dipilih langsung oleh pengguna.
- 🎨 **Dashboard Admin Premium**: Antarmuka modern berdesain *glassmorphism* (efek kaca transparan) yang dilengkapi sistem *Dark Mode* adaptif.
- 🗂️ **Manajemen Basis Pengetahuan (Knowledge Base)**: Fasilitas bagi Admin untuk menambah, mengedit, atau menghapus data pengetahuan secara mandiri berupa `tag` dan `content_text` lengkap dengan modal panduan format penulisan.
- 📄 **Upload & Auto-RAG (Import JSON)**: Unggah berkas JSON berisi database pengetahuan, lalu sistem akan otomatis menyimpan data dan menyinkronisasikan embedding vektor ke MongoDB Atlas.
- 👤 **Manajemen Multi-Admin**: Admin utama dapat membuat akun administrator baru, mengganti kata sandi, dan menghapus akun yang tidak aktif.
- 📈 **Monitor Koneksi (Live Monitor)**: Pantau aktivitas percakapan pengguna secara *real-time* langsung dari dashboard admin via WebSocket.

---

## 📸 Cuplikan Tampilan Aplikasi (Screenshots)

### 1. Antarmuka Publik & Autentikasi
| Chatbot Publik | Panel Login Admin |
| :---: | :---: |
| ![Chatbot UI](./chatbot.png) <br> *Halaman interaksi pengguna dengan AI secara langsung.* | ![Login UI](./login-admin.png) <br> *Sistem masuk eksklusif bagi administrator dengan proteksi JWT.* |

### 2. Panel Kontrol Administrator (Dashboard)
| Manajemen Pengetahuan (Knowledge Base) | Upload & Auto-RAG |
| :---: | :---: |
| ![Knowledge Base](./knowledge-base.png) <br> *Kelola seluruh data pengetahuan Direktorat Akademik.* | ![Upload & Auto-RAG](./upload-rag.png) <br> *Unggah JSON dan sinkronisasikan embedding vektor.* |

---

## 🚀 Prasyarat Sistem & Infrastruktur

Untuk menjalankan repositori ini dengan optimal, baik di lingkungan lokal (*development*) maupun peladen (*server/production*), pastikan mesin Anda telah dilengkapi:

1. **[Node.js](https://nodejs.org/en)** (Versi 18 LTS atau lebih baru) - Berfungsi sebagai mesin eksekusi JavaScript.
2. **[PNPM](https://pnpm.io/installation)** (Node Package Manager versi modern) - Untuk manajemen pustaka Frontend & Backend Node.js.
   - Perintah Instalasi: `npm install -g pnpm`
3. **[Python](https://www.python.org/downloads/)** (Versi 3.10 atau lebih baru) - Lingkungan dasar pengolahan AI.
4. **[UV](https://github.com/astral-sh/uv)** (Manajer pustaka Python berkecepatan tinggi berbasis bahasa Rust) - Digunakan untuk memangkas waktu manajemen environment Python.
   - Install via pip: `pip install uv`
   - Install via bash (macOS/Linux): `curl -LsSf https://astral.sh/uv/install.sh | sh`
5. **[Ollama](https://ollama.com/)** (Opsional, untuk Embedding Lokal) - Jika menggunakan `EMBEDDING_PROVIDER=ollama` di backend, unduh model embedding terlebih dahulu:
   ```bash
   ollama pull nomic-embed-text
   ```
6. **Koneksi Internet** yang reliabel untuk sinkronisasi MongoDB Atlas dan permintaan model *Machine Learning* via *Cloud*.

---

## 🔑 Langkah 1: Pengadaan API Keys dan Kredensial Akses

Sistem ini memanifestasikan integrasi *Cloud Services*. Anda membutuhkan dua jenis *credentials* agar sistem berjalan sempurna.

### A. MongoDB URI (Penyimpanan Utama)
1. Kunjungi [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register) dan daftarkan diri Anda.
2. Buat klaster basis data baru (Gunakan *tier* M0/Gratis untuk uji coba).
3. Buka tab **Database Access** → Buat pengguna baru dan tetapkan *Password*.
4. Buka tab **Network Access** → Klik **Add IP Address** → Pilih **Allow Access from Anywhere** (`0.0.0.0/0`) agar API dapat terhubung.
5. Pada menu **Database**, klik **Connect** → Pilih opsi **Drivers** → Salin *Connection String* yang diberikan.
   - *Format Umum: `mongodb+srv://<username>:<password>@clusterX...`*
   - Ganti `<password>` dengan sandi aktual Anda.
6. Buat **Vector Search Index** di koleksi `knowledgesources` untuk mengaktifkan pencarian semantik RAG:
   - Masuk ke **Atlas Search** → Klik **Create Search Index** → Pilih **JSON Editor**.
   - Hubungkan ke basis data `chatbot_dirmik` pada koleksi `knowledgesources`.
   - Masukkan konfigurasi berikut dan beri nama indeks `vector_index`:
     ```json
     {
       "fields": [
         {
           "type": "vector",
           "path": "embedding",
           "numDimensions": 768,
           "similarity": "cosine"
         }
       ]
     }
     ```

### B. Groq API Key (Infrastruktur LLM)
1. Kunjungi [Groq Cloud Console](https://console.groq.com/keys) dan buat akun.
2. Masuk ke menu **API Keys** → Klik **Create API Key**.
3. Simpan Kunci Sandi (`gsk_...`) di tempat yang aman. Sandi ini menjadi jembatan penghubung antara sistem Anda dan otak kecerdasan buatan Groq.

---

## 🛠️ Langkah 2: Konfigurasi Environment Variables (`.env`)

Pisahkan konfigurasi menjadi 2 file *environment* yang aman dan tidak terpublikasi (`.env`). Buat file baru pada direktori yang telah ditentukan:

### 📍 Konfigurasi Back-End API Node.js (`back-end/.env`)
Berisi konfigurasi server Express.js untuk autentikasi JWT, basis data, identitas unit, dan LLM:
```env
# ── Server ────────────────────────────────────────────────────
PORT=3000

# ── MongoDB Atlas ─────────────────────────────────────────────
MONGO_URI=mongodb+srv://admin:passwordAnda@cluster.mongodb.net/?appName=chatbot
MONGO_DB_NAME=chatbot_dirmik
MONGO_VECTOR_INDEX=vector_index

# ── JWT Secret (ganti dengan string acak yang kuat!) ──────────
JWT_SECRET=ganti_dengan_secret_acak_yang_sangat_panjang_dan_kuat

# ── Identitas Direktorat ──────────────────────────────────────
UNIT_NAME=Direktorat Akademik
UNIT_ABBREVIATION=Dirmik
UNIV_NAME=Universitas Padjadjaran
UNIV_ABBREVIATION=Unpad
HELPDESK_CONTACT=admin Direktorat Akademik Unpad

# ── LLM Provider ──────────────────────────────────────────────
LLM_PROVIDER=groq
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GROQ_MODEL=llama-3.1-8b-instant
LLM_TEMPERATURE=0.2
LLM_MAX_TOKENS=768

# ── Embedding Provider ────────────────────────────────────────
EMBEDDING_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434
EMBEDDING_MODEL=nomic-embed-text

# ── CORS ──────────────────────────────────────────────────────
WORDPRESS_URL=https://dirmik.unpad.ac.id
```

### 📍 Konfigurasi AI Python (`back-end/chatbot/.env`)
Berisi kunci infrastruktur AI, RAG, dan koneksi basis data untuk mesin Python:
```env
# Wajib sama dengan Konfigurasi Node.js
MONGO_URI=mongodb+srv://admin:passwordAnda@cluster.mongodb.net/?appName=chatbot
MONGO_DB_NAME=chatbot_dirmik

# Ganti dengan Groq API Key Anda
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Model LLM dan Embedding (gratis, dijalankan lokal oleh HuggingFace)
LLM_MODEL=llama-3.3-70b-versatile
EMBED_MODEL=sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2
```

---

## 💻 Langkah 3: Eksekusi Berjalan (Panduan Terminal)

Proyek ini menggunakan arsitektur modular yang berjalan secara mandiri dan berkesinambungan. Oleh karena itu, siapkan **3 jendela/sesi Terminal** (*Command Prompt / Terminal Mac*) sekaligus.

### 🟢 Sesi Terminal 1: Back-End API Server (Node.js)
Servis ini menaungi operasi autentikasi JWT, manajemen *Knowledge Base*, riwayat percakapan, dan manajemen akun admin.
1. Pergi ke direktori Back-End:
   ```bash
   cd back-end
   ```
2. Resolusi pustaka menggunakan PNPM:
   ```bash
   pnpm install
   ```
3. Mulai peladen:
   ```bash
   pnpm dev
   ```
   🚀 *Server Backend kini memantau jaringan pada `http://localhost:3000`*

---

### 🔵 Sesi Terminal 2: Mesin AI (Python/FastAPI)
Servis ini menaungi WebSocket monitor live dan pemrosesan RAG terisolasi untuk analisis mesin kecerdasan buatan.
1. Pergi ke direktori Chatbot AI:
   ```bash
   cd back-end/chatbot
   ```
2. Jalankan skrip terisolasi menggunakan `uv`. Anda **TIDAK PERLU** mengeksekusi `venv` atau `pip install` secara manual. Cukup serahkan pada `uv run`:
   ```bash
   uv run --with-requirements requirements.txt python app.py
   ```
   🚀 *Setelah berhasil menginisiasi model ke memori lokal, Server AI akan menyala pada `http://localhost:8080`*

   > **Pertama kali dijalankan**: `uv` akan mengunduh model embedding dari HuggingFace (~500 MB). Proses ini hanya terjadi sekali.

---

### 🟣 Sesi Terminal 3: Antarmuka Web Interaktif (Next.js)
Servis ini adalah apa yang akan dilihat dan digunakan secara visual oleh pengguna publik maupun administrator.
1. Pergi ke direktori Front-End:
   ```bash
   cd front-end
   ```
2. Pasang semua paket React & Next.js:
   ```bash
   pnpm install
   ```
3. Bangun mode pengembangan (*Dev Mode*):
   ```bash
   pnpm dev
   ```
   🚀 *Aplikasi Chatbot dan Dashboard dapat diakses di Browser melalui `http://localhost:3001`*

---

## 📝 Resolusi Masalah Umum (FAQ)

- **Aplikasi Menolak Untuk Dijalankan / Menampilkan Error `EADDRINUSE`:**
  Hal ini berarti Port 3000, 3001, atau 8080 sedang digunakan oleh perangkat lunak lain di komputer Anda. Hentikan seluruh proses yang bersinggungan melalui Task Manager (Windows) atau matikan menggunakan perintah `killall node` & `killall python` di Mac/Linux.

- **Terminal Tidak Mengenali `uv` atau `pnpm`:**
  Direktori *Environment PATH* sistem operasi Anda gagal mendeteksi instalasi. Cobalah menutup dan membuka ulang jendela Terminal untuk menyegarkan PATH, atau jalankan eksekusi Terminal sebagai Administrator.

- **Chatbot Tidak Menjawab / Error "Knowledge database belum siap":**
  Pastikan Anda sudah menekan tombol **Sync / Compile Embedding Sekarang** di menu **Upload & Auto-RAG** setelah menambahkan atau mengubah data. Jika masalah berlanjut, periksa apakah Terminal 2 (Python AI) berjalan dengan baik.

- **Koneksi MongoDB Gagal / `Mongo Timeout`:**
  Pastikan Anda telah mengaktifkan **Allow All IP** (`0.0.0.0/0`) di pengaturan *Network Access* MongoDB Atlas dan nilai `MONGO_URI` di kedua file `.env` sudah sama persis.

- **Balasan Chatbot Error / LLM Tidak Merespons:**
  Jika ada error `Rate Limit` or `Unauthorized` pada Groq, pastikan `GROQ_API_KEY` Anda valid dan sama di kedua file `.env`. Periksa juga Terminal 2 (Python AI) untuk pesan error yang lebih detail.

---

🛡️ *Sistem ini dirancang & dioptimasi khusus untuk memenuhi standarisasi pelayanan informasi* ***Direktorat Akademik Universitas Padjadjaran***.

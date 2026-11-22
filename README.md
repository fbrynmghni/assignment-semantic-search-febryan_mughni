# Semantic Search & LLM Application

Prototype aplikasi **semantic search yang terintegrasi dengan Large Language Model (LLM)** untuk menjawab pertanyaan customer service e-commerce berdasarkan knowledge base FAQ.

Project ini dibuat sebagai Individual Assignment AI Engineering Bootcamp dengan fokus pada alur:

**Ingest → Embed → Store → Retrieve → Generate**

## 1. Problem Statement

Tim customer service e-commerce menerima pertanyaan berulang mengenai pemesanan, pengiriman, pembayaran, akun, produk, promo, dan pengembalian barang.

Tujuan project ini adalah membangun prototype asisten AI yang dapat:

-   Mengubah setiap entri FAQ menjadi embedding vector.
-   Menyimpan embedding ke vector database menggunakan FAISS.
-   Menemukan FAQ yang paling relevan terhadap pertanyaan pengguna,
    termasuk pertanyaan dengan parafrase atau sinonim.
-   Menggunakan hasil retrieval sebagai context bagi LLM.
-   Menghasilkan jawaban berdasarkan informasi yang tersedia di FAQ.
-   Menyatakan bahwa informasi belum tersedia apabila pertanyaan tidak
    dapat dijawab berdasarkan knowledge base.

## 2. Knowledge Base

Project menggunakan dataset `ecommerce_faq_knowledge_base.csv` yang disediakan untuk assignment.

Dataset berisi **59 entri FAQ** dengan empat kolom:


Kolom               | Deskripsi               |
| ---------------------- | ------------------------- |
| faq_id                    | ID unik FAQ    |
| category        | Kategori FAQ |
| question        | Pertanyaan yang sering diajukan                     |
| answer        | Jawaban untuk pertanyaan


Kolom `question` dan `answer` digabungkan menjadi satu unit teks yang akan di-embedding dan di-retrieve. File CSV asli disimpan di folder `data/` dan tidak dimodifikasi secara langsung.

## 3. Technology Stack
Komponen               | Konfigurasi               |
| ---------------------- | ------------------------- |
| Bahasa Pemrograman                    | Python    |
| Environment & Dependency Management        | `uv` |
| Data Processing        | Pandas, Numpy                     |
| Embedding Model       | `nomic-embed-text:latest`             |
| Embedding Dimension             | 768                  |
| Vector Database      | FAISS (faiss-cpu)                       |
| LLM | `gpt-oss:20b`
> **Catatan:** 
Assignment brief menggunakan Gemini sebagai contoh API.
> Implementasi project ini menggunakan model lokal melalui Ollama untuk embedding dan text generation.

## 4. System Workflow

### 4.1 Ingest --- Load and prepare FAQ

Dataset dibaca menggunakan Pandas. Setiap pertanyaan dan jawaban digabungkan menjadi satu teks, misalnya:

``` text
Q: Bagaimana cara melacak paket yang sedang dikirim?
A: Nomor resi akan muncul di halaman Pesanan Saya setelah barang dikirim...
```

Teks gabungan ini menjadi dokumen yang akan dicari oleh semantic search.

### 4.2 Embed --- Generate embeddings

Setiap dokumen diubah menjadi vector menggunakan model`nomic-embed-text:latest` melalui Ollama.

Untuk embedding dokumen digunakan prefix:

``` text
search_document:
```

Untuk embedding query pengguna digunakan prefix:

``` text
search_query:
```

Prefix dibedakan sesuai penggunaan model. Hasil embedding memiliki dimensi **768**, dengan shape seluruh embedding:

``` text
(59, 768)
```

### 4.3 Store --- Save vectors in FAISS

Embedding dikonversi menjadi tipe `float32`, kemudian disimpan ke FAISS menggunakan `IndexFlatL2`.

Index menyimpan 59 vector, sesuai jumlah entri FAQ. Metode L2 mengembalikan jarak Euclidean; nilai distance yang lebih kecil menunjukkan vector yang lebih dekat.

### 4.4 Retrieve --- Semantic search

Ketika pengguna mengirim query:

1.  Query diubah menjadi embedding menggunakan `search_query:`.
2.  FAISS mencari vector dokumen terdekat.
3.  Sistem mengambil tiga FAQ teratas (`top_k=3`).
4.  Hasil retrieval memuat pertanyaan, jawaban, kategori, dan distance.

FAISS menggunakan nearest-neighbor search. Karena itu, FAISS tetap dapat mengembalikan dokumen terdekat walaupun tidak ada dokumen yang benar-benar relevan dengan query.

### 4.5 Generate --- LLM response

Hasil retrieval disisipkan ke dalam prompt sebagai context. Prompt mengatur LLM sebagai asisten customer service e-commerce dan
menginstruksikan model untuk:

-   Menjawab hanya berdasarkan FAQ yang diberikan.
-   Tidak menggunakan pengetahuan umum untuk menambahkan informasi yang
    tidak didukung context.
-   Menyatakan informasi belum tersedia apabila jawabannya tidak
    ditemukan.
-   Menyarankan pengguna menghubungi customer service jika informasi
    tidak tersedia.

Prompt kemudian dikirim ke model `gpt-oss:20b` melalui Ollama. Output jawaban diambil dari `response.message.content`.

## 5. Project Structure

``` text
semantic-search-assignment/
├── data/
│   └── ecommerce_faq_knowledge_base.csv
├── notebook/
│   └── starter_notebook_semantic_search.ipynb
├── documentation/
│   └── retrieval_test_results.md
├── README.md
├── requirements.txt
├── pyproject.toml
└── uv.lock
```

Nama file notebook dan struktur folder dapat disesuaikan dengan file aktual di repository.

## 6. Setup and Installation

### Prerequisites

Pastikan sudah tersedia:

-   Python 3.12 atau versi yang kompatibel dengan dependency project.
-   `uv`.
-   Ollama.
-   Model embedding dan LLM yang digunakan project.

### 6.1 Clone repository

``` bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd semantic-search-assignment
```

Ganti `<YOUR_GITHUB_REPOSITORY_URL>` dengan URL repository GitHub milik Anda.

### 6.2 Create virtual environment and install dependencies

Jika repository memiliki `uv.lock`, jalankan:

``` bash
uv sync
```

Jika hanya menggunakan `requirements.txt`, buat virtual environment dan install dependency:

``` bash
uv venv
uv pip install -r requirements.txt
```

### 6.3 Prepare Ollama models

Pastikan Ollama sudah berjalan, lalu unduh model jika belum tersedia:

``` bash
ollama pull nomic-embed-text:latest
ollama pull gpt-oss:20b
```

Periksa model yang tersedia:

``` bash
ollama list
```

Pastikan Ollama dapat diakses melalui:

``` text
http://localhost:11434
```

### 6.4 Run the notebook

``` bash
uv run jupyter lab
```

Buka notebook di folder `notebook/`, lalu jalankan cell secara berurutan dari awal hingga akhir.

## 7. Test Results

Pengujian dilakukan menggunakan lima query yang mencakup pertanyaan langsung, parafrase, dan pertanyaan di luar knowledge base.

  -------------------------------------------------------------------------
               No. Test query    Type         FAQ relevan           Ranking
                                              ditemukan    
  ---------------- ------------- ------------ ------------ ----------------
                 1 Apakah saya   Direct       Ya                          1
                   bisa membayar                           
                   dengan                                  
                   cicilan?                                

                 2 Bagaimana     Direct       Ya                          1
                   cara melacak                            
                   paket yang                              
                   sedang                                  
                   dikirim?                                

                 3 Kalau saya    Paraphrase   Ya                          1
                   ingin produk                            
                   dengan model                            
                   tertentu,                               
                   apakah bisa                             
                   request                                 
                   langsung ke                             
                   seller?                                 

                 4 Seusai        Paraphrase   Ya                          3
                   melakukan                               
                   transfer ke                             
                   rekening                                
                   penjual,                                
                   apakah saya                             
                   perlu                                   
                   mengirim                                
                   bukti                                   
                   pembayaran?                             

                 5 Siapa yang    Out-of-KB    Tidak                     ---
                   membakar                                
                   hutan di                                
                   Kalimantan,                             
                   Papua, dan                              
                   Ujung Kulon?                            
  -------------------------------------------------------------------------

### Retrieval summary

Dari empat query yang memiliki jawaban di knowledge base:

-   **Top-1 retrieval success:** 3/4 (75%).
-   **Top-3 retrieval success:** 4/4 (100%).

Dua query parafrase berhasil menemukan FAQ yang sesuai. Pada query konfirmasi transfer bank, FAQ relevan berada pada ranking ketiga, tetapi tetap masuk dalam top 3 sehingga tersedia sebagai context bagi LLM.

Pada query out-of-KB, FAISS mengembalikan dokumen terdekat yang tidak relevan. Namun, LLM memberikan respons bahwa informasi belum tersedia dan menyarankan pengguna menghubungi customer service.

Analisis lengkap setiap query, hasil retrieval, jawaban LLM, dan keterbatasan pengujian tersedia di
[`documentation/retrieval_test_results.md`](documentation/retrieval_test_results.md).

## 8. Limitations and Future Improvements

Beberapa keterbatasan yang ditemukan selama pengujian:

1.  **Retrieval ranking:** FAQ relevan tidak selalu berada di ranking
    pertama.
2.  **No retrieval threshold:** FAISS tetap mengembalikan nearest
    neighbors meskipun hasilnya tidak relevan.
3.  **Context dependency:** Kualitas jawaban LLM bergantung pada
    relevansi dan kelengkapan FAQ yang berhasil di-retrieve.
4.  **Limited evaluation set:** Pengujian baru mencakup lima query
    sehingga belum mewakili seluruh variasi pertanyaan pengguna.
5.  **Grounding validation:** Jawaban LLM tetap perlu dibandingkan
    dengan FAQ sumber untuk memastikan setiap klaim didukung oleh
    knowledge base.

Pengembangan berikutnya dapat mencakup evaluasi metode similarity lain, penentuan threshold berdasarkan hasil eksperimen, dan pengujian dengan lebih banyak query.

## 9. Scope and Assumptions

-   Sistem menggunakan knowledge base FAQ yang disediakan untuk
    assignment.
-   Sistem ditujukan sebagai prototype customer service e-commerce,
    bukan aplikasi produksi.
-   Jawaban LLM dibatasi pada context hasil retrieval.
-   Tidak ada jaminan bahwa seluruh pertanyaan di luar knowledge base
    akan selalu terdeteksi dengan benar; mekanisme abstention perlu
    diuji lebih luas.
-   Dataset asli tidak diubah secara langsung.

## 10. Author

**AI Engineering Bootcamp --- Individual Assignment**

Repository: `assignment-semantic-search-febryan-mughni`  

------------------------------------------------------------------------

*Project ini merupakan prototype pembelajaran semantic search dan integrasi LLM sebagai persiapan untuk membangun sistem Retrieval-Augmented Generation (RAG).*
# assignment-semantic-search-febryan_mughni

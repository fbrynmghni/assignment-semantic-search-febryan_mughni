# Retrieval Test Results

## Semantic Search & LLM Application

## 1. Informasi Pengujian

| Komponen               | Konfigurasi               |
| ---------------------- | ------------------------- |
| LLM                    | Ollama — `gpt-oss:20b`    |
| Embedding Model        | `nomic-embed-text:latest` |
| Vector Database        | FAISS                     |
| Metode Retrieval       | `IndexFlatL2`             |
| Jumlah FAQ             | 59 entri                  |
| Dimensi Embedding      | 768                       |
| Jumlah Hasil Retrieval | Top 3 (`top_k=3`)         |

Pengujian ini mengevaluasi dua bagian sistem secara terpisah:

1.  **Retrieval relevance:** apakah semantic search berhasil menemukan
    FAQ yang sesuai dengan maksud query dan menempatkannya dalam hasil
    pencarian teratas.
2.  **LLM response quality:** apakah LLM menghasilkan jawaban yang
    sesuai dengan informasi FAQ yang diberikan sebagai context, serta
    tidak mengarang jawaban ketika informasi tidak tersedia.

## 2. Skenario dan Hasil Pengujian

Pengujian dilakukan menggunakan lima query yang mencakup pertanyaan langsung, parafrase, dan pertanyaan di luar knowledge base.

### Test 1 --- Pertanyaan langsung: pembayaran cicilan

**Query pengguna**

> Apakah saya bisa membayar dengan cicilan?

**FAQ relevan**

> Apakah saya bisa membayar dengan cicilan?

**Ranking FAQ relevan:** 1

**Hasil retrieval**

1.  Apakah saya bisa membayar dengan cicilan?
2.  Apakah saya bisa memesan tanpa membuat akun?
3.  Apakah biaya ongkos kirim pengembalian ditanggung pembeli?

**Jawaban LLM**

LLM menjelaskan bahwa pembayaran cicilan tersedia untuk transaksi menggunakan kartu kredit tertentu, dengan nominal minimum yang
ditentukan oleh pihak bank.

**Analisis**

Retrieval berhasil menempatkan FAQ yang tepat pada ranking pertama. Jawaban LLM juga sesuai dengan informasi FAQ dan menjelaskan ketentuan pembayaran cicilan secara ringkas. Dua FAQ lain yang ikut terambil tidak relevan, tetapi tidak menghalangi LLM menggunakan FAQ yang tepat sebagai context.

**Evaluasi:** Retrieval berhasil; jawaban relevan dengan FAQ.

### Test 2 --- Pertanyaan langsung: pelacakan paket

**Query pengguna**

> Bagaimana cara melacak paket yang sedang dikirim?

**FAQ relevan**

> Bagaimana cara melacak paket yang sedang dikirim?

**Ranking FAQ relevan:** 1

**Hasil retrieval**

1.  Bagaimana cara melacak paket yang sedang dikirim?
2.  Bagaimana cara melaporkan penjual yang menipu?
3.  Bagaimana cara melakukan pemesanan di platform ini?

**Jawaban LLM**

LLM menjelaskan bahwa nomor resi akan muncul pada halaman **Pesanan Saya** setelah barang dikirim. Nomor resi tersebut dapat digunakan untuk melacak posisi paket melalui website kurir terkait.

**Analisis**

Retrieval berhasil menemukan FAQ yang sesuai pada ranking pertama. Jawaban LLM menyampaikan informasi utama dari FAQ dengan jelas dan tetap berfokus pada konteks yang ditemukan.

**Evaluasi:** Retrieval berhasil; jawaban relevan dengan FAQ.

### Test 3 --- Parafrase: request custom produk

**Query pengguna**

> Kalau saya ingin produk dengan model tertentu, apakah bisa request langsung ke seller?

**FAQ relevan**

> Apakah saya bisa request custom produk kepada penjual?

**Ranking FAQ relevan:** 1

**Hasil retrieval**

1.  Apakah saya bisa request custom produk kepada penjual?
2.  Bagaimana jika produk yang saya pesan ternyata kehabisan stok?
3.  Apakah bisa memesan dalam jumlah besar untuk kebutuhan reseller?

**Jawaban LLM**

LLM menjelaskan bahwa beberapa penjual menyediakan opsi request custom melalui kolom catatan pesanan. LLM juga menyarankan pengguna memeriksa halaman produk atau menghubungi penjual untuk memastikan ketersediaan layanan custom.

**Analisis**

Query menggunakan susunan kalimat dan kosakata yang berbeda dari FAQ asli, tetapi semantic search tetap berhasil menemukan FAQ yang sesuai pada ranking pertama. Hasil ini menunjukkan bahwa retrieval dapat menangkap kesamaan makna, bukan hanya mengandalkan kecocokan kata secara persis.

Jawaban LLM relevan dengan pertanyaan. Detail jawaban tetap perlu dipastikan sesuai dengan isi FAQ sumber; dalam pengujian ini, jawaban dinilai berdasarkan pemeriksaan terhadap knowledge base.

**Evaluasi:** Retrieval parafrase berhasil; jawaban relevan dengan FAQ.

### Test 4 --- Parafrase: konfirmasi transfer bank

**Query pengguna**

> Seusai melakukan transfer ke rekening penjual, apakah saya perlu mengirim bukti pembayaran?

**FAQ relevan**

> Apakah saya perlu konfirmasi manual setelah transfer bank?

**Ranking FAQ relevan:** 3

**Hasil retrieval**

1.  Bagaimana cara melaporkan penjual yang menipu?
2.  Bagaimana jika penjual menolak pengajuan pengembalian saya?
3.  Apakah saya perlu konfirmasi manual setelah transfer bank?

**Jawaban LLM**

LLM membedakan dua metode pembayaran:

-   **Virtual Account (VA):** konfirmasi pembayaran dilakukan secara
    otomatis sehingga pengguna tidak perlu mengirim bukti transfer.
-   **Transfer manual:** pengguna perlu mengunggah bukti transfer agar
    pesanan dapat diverifikasi atau diproses.

**Analisis**

Meskipun menggunakan parafrase dengan susunan kata berbeda, semantic search berhasil menemukan FAQ yang tepat pada ranking ketiga. FAQ tersebut tetap masuk dalam top 3 sehingga diteruskan ke LLM sebagai bagian dari context.

Berdasarkan pemeriksaan terhadap dataset, jawaban LLM sesuai dengan informasi pada FAQ. Hasil ini memperlihatkan bahwa FAQ yang relevan tidak harus berada pada ranking pertama agar dapat mendukung jawaban yang benar, selama masih termasuk dalam jumlah dokumen yang diambil (`top_k`).

**Evaluasi:** 
Retrieval parafrase berhasil dalam top 3; jawaban sesuaidengan FAQ.

### Test 5 --- Out-of-Knowledge-Base: pertanyaan di luar cakupan FAQ

**Query pengguna**

> Siapa yang membakar hutan di Kalimantan, Papua, dan Ujung Kulon?

**Hasil retrieval**

1.  Berapa lama estimasi waktu pengiriman barang?
2.  Apakah biaya ongkos kirim pengembalian ditanggung pembeli?
3.  Apakah bisa menukar barang dengan ukuran/warna lain daripada refund?

Tidak ada FAQ yang relevan dengan pertanyaan tersebut.

**Jawaban LLM**

> Maaf, saya belum memiliki informasi tersebut. Silakan hubungi customer service untuk bantuan lebih lanjut.

**Analisis**

FAISS tetap mengembalikan tiga dokumen terdekat meskipun dokumen tersebut tidak relevan. Hal ini merupakan perilaku nearest-neighbor
search: FAISS mencari vektor yang paling dekat, tetapi tidak secara otomatis menentukan apakah hasil tersebut benar-benar menjawab query.

Meskipun context yang diberikan tidak relevan, LLM mengikuti instruksi prompt untuk tidak menjawab menggunakan informasi di luar knowledge base. LLM menyatakan bahwa informasi belum tersedia dan menyarankan pengguna menghubungi customer service.

**Evaluasi:** 
Tidak ditemukan FAQ relevan; LLM berhasil melakukan abstention pada skenario ini.

## 3. Ringkasan Hasil Retrieval

  ------------------------------------------------------------------------
               No. Jenis query  FAQ relevan       Ranking FAQ Evaluasi
                                ditemukan?            relevan 
  ---------------- ------------ ------------ ---------------- ------------
                 1 Direct ---   Ya                          1 Berhasil
                   pembayaran                                 
                   cicilan                                    

                 2 Direct ---   Ya                          1 Berhasil
                   pelacakan                                  
                   paket                                      

                 3 Parafrase    Ya                          1 Berhasil
                   --- custom                                 
                   produk                                     

                 4 Parafrase    Ya                          3 Berhasil
                   ---                                        dalam top 3
                   konfirmasi                                 
                   transfer                                   

                 5 Out-of-KB    Tidak                     --- Tidak ada
                   ---                                        FAQ relevan
                   pembakaran                                 
                   hutan                                      
  ------------------------------------------------------------------------

Dari empat query yang memiliki jawaban di knowledge base:

-   **Top-1 retrieval success:** 3 dari 4 query (75%).
-   **Top-3 retrieval success:** 4 dari 4 query (100%).

Perhitungan tersebut hanya menggunakan empat query in-KB. Query out-of-KB tidak dimasukkan ke dalam perhitungan retrieval success karena memang tidak memiliki FAQ relevan yang diharapkan untuk ditemukan.

## 4. Evaluasi Kualitas LLM Response

Secara umum, LLM menghasilkan jawaban yang relevan untuk keempat pertanyaan yang memiliki informasi dalam knowledge base. Jawaban disampaikan dengan bahasa yang natural dan mengembangkan informasi FAQ menjadi respons yang mudah dipahami.

Pada query konfirmasi transfer bank, FAQ yang relevan berada di ranking ketiga, tetapi tetap tersedia dalam top 3 context. LLM dapat menggunakan informasi tersebut untuk membedakan proses konfirmasi Virtual Account dan transfer manual. Kesesuaian jawaban ini telah diperiksa terhadap dataset FAQ.

Pada query out-of-KB, LLM tidak memberikan jawaban spekulatif dan menyatakan bahwa informasi belum tersedia. Hasil ini sesuai dengan
tujuan prompt, yaitu membatasi jawaban berdasarkan context hasil retrieval.

Namun, satu pengujian out-of-KB belum cukup untuk membuktikan bahwa LLM akan selalu menolak seluruh pertanyaan di luar cakupan knowledge base. Pengujian tambahan diperlukan untuk mengevaluasi konsistensi perilaku tersebut.

## 5. Temuan dan Keterbatasan

### Temuan

1.  Semantic search berhasil menemukan FAQ relevan untuk seluruh empat
    query in-KB dalam top 3.
2.  Dua query parafrase berhasil menemukan FAQ yang sesuai. Satu berada
    di ranking pertama dan satu lainnya di ranking ketiga.
3.  LLM dapat menghasilkan jawaban relevan ketika FAQ yang dibutuhkan
    tersedia dalam context.
4.  LLM berhasil menyatakan bahwa informasi tidak tersedia pada skenario
    out-of-KB yang diuji.

### Keterbatasan

1.  **Ranking belum selalu optimal.** FAQ konfirmasi transfer bank
    berada pada ranking ketiga, sehingga masih ada ruang untuk
    meningkatkan urutan relevansi.
2.  **FAISS selalu mengembalikan nearest neighbors.** Dokumen yang
    dikembalikan belum tentu relevan secara substantif dengan query.
3.  **Belum ada mekanisme threshold atau abstention pada tahap
    retrieval.** Sistem belum memiliki aturan berbasis skor yang
    menentukan kapan hasil pencarian terlalu tidak relevan untuk
    diteruskan ke LLM.
4.  **Jumlah pengujian masih terbatas.** Lima query memenuhi jumlah
    minimum assignment, tetapi belum mewakili seluruh variasi pertanyaan
    pengguna.
5.  **Grounding tetap perlu diperiksa.** Jawaban yang terdengar
    meyakinkan belum tentu sepenuhnya didukung FAQ. Setiap klaim penting
    perlu dibandingkan dengan sumber yang ditemukan.

## 6. Kesimpulan

Berdasarkan pengujian, prototype Semantic Search & LLM berhasil menjalankan alur dari query pengguna, embedding query, retrieval
menggunakan FAISS, penyusunan context, hingga pembuatan jawaban menggunakan model `gpt-oss:20b`melalui Ollama.

Sistem berhasil menemukan FAQ relevan dalam top 3 untuk seluruh query yang memiliki jawaban di knowledge base. Pengujian parafrase juga menunjukkan bahwa semantic search mampu menemukan informasi yang sesuai meskipun pengguna tidak menggunakan kalimat yang sama dengan FAQ asli.

Pada tahap generation, LLM menghasilkan jawaban yang relevan ketika informasi tersedia dan berhasil menyatakan bahwa informasi belum
tersedia pada satu skenario out-of-KB yang diuji.

Peningkatan selanjutnya dapat difokuskan pada optimasi ranking retrieval dan penambahan mekanisme untuk mendeteksi hasil retrieval yang tidak relevan sebelum context dikirim ke LLM. Dengan demikian, kualitas retrieval dan keandalan jawaban dapat ditingkatkan secara bersamaan.

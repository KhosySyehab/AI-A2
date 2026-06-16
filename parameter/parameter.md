1. Pemahaman masalah dan rancangan sistem RAG
Masalah: BANASPATI awalnya chatbot kedai bubur, sehingga tidak punya pengetahuan akademik dan rawan halusinasi.
Solusi: bangun sistem Multimodal RAG yang mengambil konteks dari dokumen ITS lalu memanfaatkan LLM untuk menjawab berdasar konteks tersebut.
Arsitektur:
preprocess dokumen → chunking → embedding
simpan ke ChromaDB
retrieval hybrid untuk ambil konteks relevan
kirim konteks ke Gemini dengan prompt terkontrol
Tujuan desain: menjaga agar jawaban selalu grounded di dataset, bukan jawaban manual.


2. Pemrosesan dokumen, multimodal processing, dan metadata sumber
Dokumen sudah diproses menjadi all_chunks_v3.pkl; ini berisi chunks teks dengan metadata source dan page.
Embedding model: paraphrase-multilingual-MiniLM-L12-v2 untuk teks Indonesia/multibahasa.
Metadata:
setiap chunk menyimpan source (nama dokumen) dan page
metadata ini dipakai untuk audit dan menampilkan sumber konteks
Multimodal processing:
di pipeline preprocessing (Orang 1), dokumen seperti Jadwal Perkuliahan.docx dan Kalender-Akademik.pdf diolah dengan Gemini Vision karena berbentuk gambar/scan.
hasil ekstraksi ini kemudian masuk ke dalam chunk yang diretrieve.
Di banaspati_final, walau menggunakan precomputed chunks, metadata masih dipertahankan dan digunakan untuk source tracking.


3. Kualitas retrieval dan konteks yang digunakan
Retrieval bukan sekadar semantic search saja:
semantic search via ChromaDB embedding
keyword boost / word boundary matching
category detection untuk topik spesifik
source-based injection untuk dokumen relevan
must-inject untuk topik kritis
Kategori khusus:
jadwal, kalender, UTBK, magang, peraturan, kurikulum
Must-inject:
Q08 IPS/SKS → chunk Pasal 51
Q09 cuti studi → chunk Pasal 74
Q05/Q06 magang → chunk Sosialisasi Magang
Q07 kalender → prioritas Kalender-Akademik
Skor akhir retrieval:
40% semantic
40% keyword
boost sumber
Hasil retrieval ditampilkan di UI dengan konteks dan sumber, sehingga pengguna dan penguji bisa audit.


4. Kualitas jawaban dan grounding terhadap dataset
System prompt sangat ketat:
jawab hanya dari konteks
jangan mengarang
jika data tidak cukup, wajib jawab “Maaf, informasi tersebut tidak ditemukan...”
Prompt juga memberikan panduan khusus untuk membaca jadwal agar model lebih akurat.
Grounding:
konteks hasil retrieval disisipkan langsung dalam prompt
source dan halaman dicantumkan dalam setiap segmen konteks
prompt final ditampilkan di notebook untuk transparansi
Ini memastikan jawaban hanya bisa muncul bila ada dukungan konteks dari dataset.


5. Evaluasi metrik akurasi/kualitas menggunakan RAGAS dan LLM-as-a-Judge
Notebook banaspati_final fokus ke pipeline demo, sementara evaluasi RAGAS / LLM-as-a-Judge dijalankan di orang 3.ipynb.
Desain sistem sudah mendukung kedua evaluasi karena:
retrieval explicit menghasilkan konteks yang bisa dievaluasi
history output dan source tersedia untuk rubrik quality
Untuk RAGAS:
metrik yang relevan adalah faithfulness, relevansi jawaban, context precision, context recall
Untuk LLM-as-a-Judge:
menggunakan Gemini 3 Flash dengan rubrik seperti correctness, faithfulness, relevance, completeness, source_support, hallucination


6. Evaluasi metrik inferensi
Dashboard di notebook menampilkan:
retrieval latency
generation latency
end-to-end latency
token usage (input + output)
throughput
estimasi biaya
TTFT:
tidak tersedia karena pipeline tidak mendukung streaming
digantikan dengan generation latency dan E2E latency
Estimasi biaya:
dihitung secara kasar dengan asumsi rate Gemini Flash: input $0.075 / 1M, output $0.30 / 1M
Resource:
karena menggunakan API Gemini, beban VRAM/GPU di server tidak dijadikan metrik utama
namun embedding dan ChromaDB berjalan lokal, jadi penggunaan memori ada pada mesin notebook


7. Eksplorasi teknis dan alasan pemilihan konfigurasi terbaik
ChromaDB dipilih karena:
ringan, mudah gunakan di notebook
mendukung penyimpanan embedding + metadata
SentenceTransformers dipilih karena:
model ringan
performa cukup baik untuk Bahasa Indonesia / multilingual
Gemini API dipilih karena:
integrasi cepat
cukup andal untuk demo
tidak memerlukan hardware lokal besar
Hybrid retrieval dipilih karena:
semantic search saja belum cukup untuk topik spesifik
keyword / source boost membantu mengangkat dokumen kritis
Multimodal processing dipilih untuk:
jadwal yang berupa gambar di DOCX
kalender akademik yang berupa scan
Precomputed chunks dipilih untuk:
mempercepat startup demo
menjaga runtime fokus pada retrieval dan generation
Gradio dipilih sebagai demo sandbox untuk memenuhi tuntutan soal demo.


8. Analisis metrik, interpretasi hasil, dan pembahasan limitasi sistem
Interpretasi metrik:
retrieval latency kecil menunjukkan ChromaDB dan embedding efektif
generation latency menunjukkan performa API Gemini, dan bisa naik bila prompt/context besar
throughput membantu melihat efisiensi token keluaran
estimasi biaya memberikan gambaran biaya penggunaan API untuk demo
Analisis:
jika retrieval menghasilkan konteks tepat, jawaban cenderung akurat dan grounded
jika konteks kurang relevan, model bisa kehilangan akurasi meski prompt sudah bagus
Limitasi:
tidak ada TTFT karena non-streaming
estimasi biaya masih bersifat kasar / tidak live
startup masih perlu load/insert chunk ke ChromaDB, sehingga perlu waktu awal
model API bergantung pada koneksi dan kuota key
local/open-weight model belum dieksplorasi dalam notebook ini
evaluasi RAGAS/Judge tidak tampil di notebook demo ini, tapi ada di notebook terpisah

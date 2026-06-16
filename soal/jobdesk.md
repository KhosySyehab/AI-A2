🧑‍💻 Orang 1 — Data & Retrieval
Tanggung jawab:

Extract & eksplorasi isi dataset (PDF, gambar, tabel, dll)
Preprocessing dokumen: parsing teks, extract gambar/tabel (multimodal)
Chunking dokumen + tambahin metadata
Setup embedding model + vector database (ChromaDB dll)
Implementasi fungsi retrieve(query) → return dokumen relevan
Uji coba retrieval dengan beberapa query

Output: Fungsi retrieve(query) siap dipakai Orang 2

🤖 Orang 2 — LLM & Generation
Tanggung jawab:

Setup Gemini API
Prompt engineering (system prompt, context injection, fallback "tidak ditemukan")
Sambungkan hasil retrieve dari Orang 1 ke Gemini
Bangun pipeline RAG end-to-end
Bikin Demo Sandbox (Gradio di notebook)
Tracking token usage & latency

Output: Chatbot yang bisa dijawab pertanyaan + Demo Sandbox

📊 Orang 3 — Evaluasi
Tanggung jawab:

Baca & siapkan CSV eval questions yang disediakan
Bikin ground truth jawaban
Evaluasi RAGAS (faithfulness, relevancy, context precision, dll)
Evaluasi LLM-as-a-Judge pakai Gemini Flash + rubrik penilaian
Laporan metrik inferensi (latency, token usage, throughput, dll)
Interpretasi & analisis hasil evaluasi

Output: Laporan evaluasi lengkap di notebook

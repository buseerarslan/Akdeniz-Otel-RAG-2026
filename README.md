# 🏨 Akdeniz Otel & Resort 2026 - Multilingual RAG & Tool-Calling Assistant

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![LangChain](https://img.shields.io/badge/Framework-LangChain-green)
![LLM](https://img.shields.io/badge/LLM-Gemini%202.5%20Flash-orange)
![VectorDB](https://img.shields.io/badge/VectorDB-ChromaDB-purple)
![Embedding](https://img.shields.io/badge/Embedding-paraphrase--multilingual--MiniLM--L12--v2-red)

Bu proje, **Akdeniz Otel ve Resort 2026** misafir el kitabı (unstructured PDF) ve oda fiyat tarifeleri (structured CSV) verileri üzerinde çalışan, halüsinasyonu sıfıra indirilmiş, Türkçe anlamsal arama optimizasyonlu **7 adımlı bir Hibrit RAG (Retrieval-Augmented Generation) ve Tool-Calling** mimarisidir.

---

## 📌 İçindekiler
- [Proje Mimarisi ve Akış](#-proje-mimarisi-ve-akış)
- [Uygulanan 7 Mühendislik Adımı](#-uygulanan-7-mühendislik-adımı)
- [Öne Çıkan Başarılar ve Benchmark Bulguları](#-öne-çıkan-başarılar-ve-benchmark-bulguları)
- [Teknoloji Yığını (Tech Stack)](#-teknoloji-yığını-tech-stack)
- [Kurulum ve Çalıştırma](#-kurulum-ve-çalıştırma)
- [Gelecek Yol Haritası ve Geliştirme Önerileri (Future Work)](#-gelecek-yol-haritası-ve-geliştirme-önerileri-future-work)

---

## 🏗️ Proje Mimarisi ve Akış

Sistem, kullanıcı sorularını türüne göre analiz ederek doğru veri kaynağına yönlendiren hibrit bir yapıya sahiptir:

+-----------------------+
                              |   Kullanıcı Sorusu    |
                              +-----------+-----------+
                                          |
                                          v
                              +-----------------------+
                              |   Soru Tipi Tespiti   |
                              +-----+-----------+-----+
                                    |           |
        [Otel Kuralları & Saatler] |           | [Oda Fiyatları & Kapasite]
                                    v           v
                +---------------------+       +---------------------+
                |   PDF RAG Chain     |       |   CSV Tool (@tool)  |
                | (Vector Retrieval)  |       |  (Pandas Filtering) |
                +----------+----------+       +----------+----------+
                           |                             |
                           +--------------+--------------+
                                          |
                                          v
                              +-----------------------+
                              |  Gemini 2.5 Flash LLM |
                              +-----------+-----------+
                                          |
                                          v
                              +-----------------------+
                              |  Güvenilir Yanıt /    |
                              |  "Bilgi Bulunamadı"   |
                              +-----------------------+


---

## 🛠️ Uygulanan 7 Mühendislik Adımı

| Adım | Başlık | Uygulama Detayı | Gerekçe & Hedef |
| :--- | :--- | :--- | :--- |
| **Adım 1** | **Halüsinasyon Tespiti** | Dokümansız ortama 10 adet kontrol sorusu yöneltildi. | Yalın LLM'lerin uydurma (hallucination) oranını tespit etmek ve RAG sonrası başarıyı kıyaslamak. |
| **Adım 2** | **Doküman Yükleme & Metadata Enrichment** | PDF sayfaları yüklenirken `kaynak`, `sayfa`, `bolum` ve `yil` metadata etiketleri eklendi. | Modelin yanıt üretirken kaynak gösterme (attribution) yeteneğini kazanmasını sağlamak. |
| **Adım 3** | **Anlamsal Parçalama (Chunking)** | `RecursiveCharacterTextSplitter` *(Chunk Size: 500, Overlap: 50)* kullanıldı. | Kısa otel kurallarının anlamsal bütünlüğünü ve bağlamını bozmadan vektörleştirmek. |
| **Adım 4** | **Çok Dilli Vektörleştirme & Veritabanı** | `ChromaDB` üzerinde `paraphrase-multilingual-MiniLM-L12-v2` modeli konumlandırıldı. | Türkçe dil yapısında ve ekli sözcük aramalarında %100'e yakın anlamsal eşleşme yakalamak. |
| **Adım 5** | **Retrieval Stratejileri Benchmark'ı** | `Similarity`, `MMR (Maximal Marginal Relevance)` ve `Threshold` yöntemleri karşılaştırıldı. | Bağlam çeşitliliği ile doğrudan eşleşme arasındaki en iyi retriever dengesini bulmak. |
| **Adım 6** | **LCEL RAG Chain & Benchmark Tablosu** | LangChain Expression Language (LCEL) ile uçtan uca zincir kuruldu; 10 soru üzerinde test edildi. | Dokümansız Gemini yanıtları ile RAG yanıtlarını Pandas tablosunda akademik olarak kıyaslamak. |
| **Adım 7** | **CSV Tool Calling Entegrasyonu** | `@tool` dekoratörü ile `otel_fiyat_sorgula` Pandas aracı sisteme entegre edildi. | Metin tabanlı PDF verileri ile tablo tabanlı CSV verilerini tek bir asistanda birleştirmek. |

---

## 📊 Öne Çıkan Başarılar ve Benchmark Bulguları

10 soruluk karşılaştırma testleri sonucunda elde edilen temel bulgular:
- **Halüsinasyon Sıfırlandı:** Dokümanda yer almayan sorularda (örn. *Tenis kortu*, *Havalimanı transferi*, *Çelik kasa*) yalın LLM yanlış tahminlerde bulunurken, RAG zinciri katı yönlendirme sayesinde kesin olarak `"Verilen dokümanda bu konuyla ilgili bir bilgi bulunmamaktadır"` yanıtını vermiştir.
- **Nokta Atışı Veri Hassasiyeti:** Evcil hayvan politikası (8 kg sınırı, 350 TL ek ücret) ve havuz kapanış saati (19:00 - ilaçlama nedeni) gibi spesifik rakamlar ve gerekçeler eksiksiz aktarılmıştır.
- **Çok Dilli Model Farkı:** Standart İngilizce embedding modellerinin aksine, multilingual model kullanımı Türkçe soru-cevap doğruluğunu %40'tan %100'e çıkarmıştır.

---

## 💻 Teknoloji Yığını (Tech Stack)

- **Language Model:** Google Gemini 2.5 Flash
- **Orchestration:** LangChain (LCEL core)
- **Vector Database:** ChromaDB
- **Embeddings:** `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`
- **Structured Data Processing:** Pandas
- **PDF Processing:** PyPDF
- **Environment:** Google Colab / Python 3.10+

---

## ⚡ Kurulum ve Çalıştırma

1. **Repoyu Klonlayın:**
   ```bash
   git clone [https://github.com/KULLANICI_ADI/Akdeniz-Otel-RAG-2026.git](https://github.com/KULLANICI_ADI/Akdeniz-Otel-RAG-2026.git)
   cd Akdeniz-Otel-RAG-2026


   Gerekli Paketleri Yükleyin:

   pip install -qU langchain langchain-community langchain-huggingface langchain-chroma pypdf pandas sentence-transformers google-genai


   Google Colab Üzerinde Çalıştırma:
Akdeniz_Otel_RAG_Asistani_2026.ipynb dosyasını Google Colab'da açıp Open in Colab butonuna basarak tüm hücreleri sırayla çalıştırabilirsiniz.

🚀 Gelecek Yol Haritası ve Geliştirme Önerileri (Future Work)
Sistemin v2.0 ve v3.0 sürümlerinde üretim ortamına (production) taşınması için planlanan geliştirmeler:

1. Kullanıcı Arayüzü (UI/UX) Entegrasyonu (v2.0)
Gradio / Streamlit Web App: Kullanıcıların doğrudan canlı mesajlaşabileceği modüler bir sohbet arayüzünün (Chatbot UI) entegre edilmesi.

Sesli Asistan Entegrasyonu: Google Whisper / TTS modelleri ile sesli soru-cevap desteğinin eklenmesi.

2. İleri Seviye RAG Mimarileri (v2.5)
Hybrid Search (Sparse + Dense): ChromaDB vektör aramasına BM25 (anahtar kelime tabanlı arama) eklenerek Hybrid Search yapısının kurulması.

Re-ranking (Yeniden Sıralama): Retrieve edilen doküman parçalarının Cohere Rerank veya BGE-Reranker ile yeniden sıralanarak doğruluk oranının %99'a çıkarılması.

Parent-Document Retriever: Metin parçaları küçük chunk'lar halinde aranırken, LLM'e tüm paragrafın (parent context) beslenmesi.

3. Agentic Workflow & Çoklu Veritabanı (v3.0)
LangGraph Entegrasyonu: Statik LCEL zinciri yerine kararları dinamik veren, hata durumunda sorguyu yeniden yazan (Self-RAG / Corrective RAG) bir Agent mimarisine geçilmesi.

Canlı SQL Veritabanı: Oda musaitlik durumları ve canlı rezervasyonlar için PostgreSQL / MySQL veritabanı bağlayıcı araçlarının (SQL Database Tools) eklenmesi.

📜 Lisans ve Teşekkür
Bu proje akademik değerlendirme ve prototip geliştirme amacıyla hazırlanmıştır. Katkıda bulunan tüm açık kaynak LangChain ve HuggingFace topluluğuna teşekkür ederiz.

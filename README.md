# 🔍 Phân Cụm Chủ Đề Văn Bản Thị Trường Lao Động Việt Nam
### Topic Modeling for Vietnamese Labor Market Text Data

Đồ án môn **Xử lý ngôn ngữ tự nhiên (NLP)** — Đại học Kinh tế TP.HCM (UEH)

Nghiên cứu, triển khai và so sánh các phương pháp **Topic Modeling** nhằm khám phá các chủ đề tiềm ẩn trong tập văn bản thu thập từ mạng xã hội liên quan đến thị trường lao động Việt Nam.

---

## 📋 Mục Lục
- [Tổng quan](#tổng-quan)
- [Bộ dữ liệu](#bộ-dữ-liệu)
- [Pipeline xử lý](#pipeline-xử-lý)
- [Các mô hình sử dụng](#các-mô-hình-sử-dụng)
- [Kết quả & So sánh](#kết-quả--so-sánh)
- [Hướng dẫn cài đặt](#hướng-dẫn-cài-đặt)
- [Cấu trúc thư mục](#cấu-trúc-thư-mục)
- [Công nghệ sử dụng](#công-nghệ-sử-dụng)
- [Nhóm thực hiện](#nhóm-thực-hiện)

---

## Tổng Quan

Trong bối cảnh các nền tảng mạng xã hội chứa lượng lớn văn bản phi cấu trúc về đời sống công sở, đề tài áp dụng **Topic Modeling** để tự động khám phá các chủ đề tiềm ẩn mà không cần gán nhãn trước.

**Mục tiêu chính:**
- Thu thập và xây dựng bộ dữ liệu văn bản tiếng Việt về thị trường lao động từ mạng xã hội
- Xây dựng pipeline tiền xử lý chuyên biệt cho văn bản tiếng Việt (teencode, stopwords, tokenize...)
- Triển khai và so sánh 4 mô hình: **LSA, LDA, NMF, BERTopic**
- Rút ra insights về các mối quan tâm phổ biến của người lao động Việt Nam

---

## Bộ Dữ Liệu

Dữ liệu được thu thập từ các nền tảng công khai sử dụng **Selenium**:

| Nguồn | Nội dung |
|---|---|
| Threads | Bài viết về đi làm, văn phòng GenZ, phỏng vấn |
| Facebook | Chia sẻ về môi trường làm việc, lương thưởng |
| 1900.com.vn | Review công ty, đánh giá môi trường làm việc |

**Thống kê bộ dữ liệu:**
- Tổng số bài viết: ~3,000+ bài
- Độ dài: 12 – 1,500 từ/bài
- Ngôn ngữ: Tiếng Việt (có pha tiếng Anh)
- Phân phối: Lệch phải, đa số bài dưới 100 từ

---

## Pipeline Xử Lý

```
Thu thập dữ liệu (Selenium crawling)
        ↓
Lọc spam / duplicate / độ dài
        ↓
Dịch tiếng Anh → tiếng Việt (GoogleTranslator)
        ↓
Chuẩn hóa teencode, ký tự đặc biệt
        ↓
Tách từ (ViTokenizer)
        ↓
Loại bỏ stopwords tiếng Việt
        ↓
EDA (Word Cloud, histogram, n-gram)
        ↓
Vectorization (TF-IDF / SBERT Embedding)
        ↓
Topic Modeling (LSA / LDA / NMF / BERTopic)
        ↓
Đánh giá: Compactness, Separation, Score
```

---

## Các Mô Hình Sử Dụng

### 1. LSA (Latent Semantic Analysis)
- Phân rã ma trận TF-IDF bằng **Truncated SVD**
- Số chủ đề tối ưu: **k = 5** (elbow method)

### 2. LDA (Latent Dirichlet Allocation)
- Mô hình xác suất Bayes với phân phối **Dirichlet**
- Số chủ đề tối ưu: **k = 5**

### 3. NMF (Non-negative Matrix Factorization)
- Phân rã ma trận với ràng buộc không âm
- Số chủ đề tối ưu: **k = 5**
- ✅ **Mô hình tốt nhất** trong nhóm truyền thống

### 4. BERTopic
- Kết hợp **SBERT embedding** + **UMAP** + **HDBSCAN**
- Embedding model: `paraphrase-multilingual-MiniLM-L12-v2`
- Tự động xử lý nhiễu qua Topic -1
- ✅ **Mô hình được chọn** cho phân tích cuối cùng

---

## Kết Quả & So Sánh

### So sánh các mô hình truyền thống (k=5)

| Mô hình | Compactness | Separation | Score | Nhận xét |
|---|---|---|---|---|
| LDA | 0.0920 | 0.4642 | 0.4815 | Nhiều từ khóa nhiễu |
| LSA | 0.0934 | 0.6903 | 7.3907 | Mất cân bằng dữ liệu lớn |
| **NMF** | **0.0705** | **0.7423** | **10.5297** | ✅ Tốt nhất nhóm truyền thống |

### So sánh NMF vs BERTopic

| Tiêu chí | NMF | BERTopic |
|---|---|---|
| Compactness | 0.0705 ✅ | 0.1803 |
| Separation | 0.7423 ✅ | 0.4115 |
| Xử lý nhiễu | Không (ép buộc gán nhãn) | Có (Topic -1) ✅ |
| Phát hiện chủ đề ngách | Không | Có ✅ |
| Hiểu ngữ nghĩa | Dựa trên từ khóa | Dựa trên ngữ cảnh ✅ |

### Các chủ đề phát hiện được (NMF)

| Chủ đề | Từ khóa đại diện | Ý nghĩa |
|---|---|---|
| 0 | nhân_viên, công_ty, quản_lý, nghỉ | Môi trường & quản trị nội bộ |
| 1 | công_việc, kinh_nghiệm, sinh_viên, thực_tập | Học tập & thực tập |
| 2 | lương, thưởng, tăng_ca, hạn, trừ | Lương thưởng & thu nhập |
| 3 | đóng, bảo_hiểm, bhxh, xã_hội | Bảo hiểm xã hội |
| 4 | phỏng_vấn, hồ_sơ, hr, tuyển_dụng | Quy trình tuyển dụng |

### Các chủ đề ngách phát hiện bởi BERTopic

| Topic | Số bài | Chủ đề |
|---|---|---|
| 0 | 1,517 | Phàn nàn chung (lương, công ty, môi trường) |
| 1 | 31 | Truyền thông / Mạng xã hội (TikTok, YouTube) |
| 2 | 21 | Nợ lương / Lương chậm |
| 3 | 16 | Cơ sở vật chất (laptop, văn phòng) |
| 4 | 11 | Quản lý / Sếp (boss, manager) |
| -1 | 1,489 | Nhiễu (tự động loại bỏ) |

---

## Hướng Dẫn Cài Đặt

### Yêu cầu
- Python 3.8+
- Jupyter Notebook / Google Colab

### Cài đặt thư viện

```bash
git clone https://github.com/your-username/nlp-topic-modeling-vietnamese-labor.git
cd nlp-topic-modeling-vietnamese-labor
pip install -r requirements.txt
```

### Các thư viện chính

```bash
pip install bertopic
pip install sentence-transformers
pip install underthesea
pip install pyvi
pip install selenium
pip install googletrans==4.0.0-rc1
pip install scikit-learn pandas numpy matplotlib seaborn wordcloud
```

### Chạy theo thứ tự

```
notebooks/1_EDA_Preprocessing.ipynb   # Tiền xử lý & EDA
notebooks/2_Modeling.ipynb            # LSA, LDA, NMF
notebooks/3_Run_BERTopic.ipynb        # BERTopic
```

---

## Cấu Trúc Thư Mục

```
nlp-topic-modeling-vietnamese-labor/
├── README.md
├── data/
│   ├── reviewjob.csv                  # Dữ liệu thô sau crawl
│   ├── topic_modeling_data.csv        # Dữ liệu sau tiền xử lý
│   └── vietnamese-stopwords.txt       # Danh sách stopwords tiếng Việt
├── notebooks/
│   ├── 1_EDA_Preprocessing.ipynb      # EDA & tiền xử lý
│   ├── 2_Modeling.ipynb               # LSA, LDA, NMF
│   └── 3_Run_BERTopic.ipynb           # BERTopic
├── bertopic_model_after_outliers/
│   ├── config.json                    # Cấu hình BERTopic
│   ├── ctfidf_config.json             # Cấu hình c-TF-IDF
│   ├── ctfidf.safetensors             # Trọng số c-TF-IDF
│   ├── topic_embeddings.safetensors   # Embedding các chủ đề
│   └── topics.json                    # Kết quả các chủ đề
└── [Report][Topic Modeling for Vietnamese Labor Market Text Data].pdf
```

---

## Công Nghệ Sử Dụng

| Hạng mục | Công cụ / Thư viện |
|---|---|
| Thu thập dữ liệu | Selenium |
| Tiền xử lý | ViTokenizer, underthesea, googletrans |
| Vector hóa | TF-IDF (scikit-learn), SBERT |
| Topic Modeling | LSA, LDA, NMF (scikit-learn), BERTopic |
| Clustering | UMAP, HDBSCAN |
| Embedding model | `paraphrase-multilingual-MiniLM-L12-v2` |
| Trực quan hóa | Matplotlib, Seaborn, WordCloud |
| Môi trường | Python 3.8+, Jupyter Notebook, Google Colab |

---

## Tác Giả

[dungnguyenbo@gmail.com](mailto:dungnguyenbo@gmail.com)  
[github.com/dg-ng](https://github.com/dg-ng)

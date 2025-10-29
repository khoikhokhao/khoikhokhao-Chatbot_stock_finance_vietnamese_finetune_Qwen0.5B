# 📘 Chatbot Hỏi–Đáp Chứng Khoán Việt Nam

## 🧭 Tổng quan

### Bối cảnh
Thị trường chứng khoán Việt Nam phát triển nhanh, thu hút hàng triệu nhà đầu tư cá nhân, tuy nhiên phần lớn còn hạn chế kiến thức và bị ảnh hưởng bởi tin đồn.  
Thông tin về quy định, khái niệm, quy trình và dữ liệu tài chính lại phân tán.

➡️ **Chatbot Q/A Chứng khoán Việt Nam** là một **trợ lý hỏi–đáp tiếng Việt** hướng đến nhà đầu tư cá nhân mới và người dùng muốn tra cứu **khái niệm, quy trình, luật lệ và dữ liệu lịch sử (OHLCV + chỉ báo)**.  
Hệ thống **không dự báo giá hay khuyến nghị giao dịch**, mục tiêu là **cung cấp câu trả lời chính xác, dễ hiểu, có nguồn tham chiếu và định dạng thống nhất**:  
> **Tóm tắt → Giải thích → Rủi ro → Nguồn**

### Kiến trúc chính
- **Mô hình nền**: Qwen-0.5B-Instruct  
- **RAG mỏng**: BM25 + Dense Embedding  
- **Fine-tune nhẹ**: LoRA để chuẩn hóa phong cách trả lời  
- **Dữ liệu tĩnh**: Không cập nhật real-time  

---

## 🎯 Mục tiêu và Phạm vi

### Mục tiêu
- Hỗ trợ người mới tìm hiểu thị trường chứng khoán Việt Nam bằng ngôn ngữ tự nhiên.  
- Trả lời câu hỏi về **khái niệm, quy trình, chỉ báo kỹ thuật, quy định pháp lý**.  
- Giảm rủi ro do thông tin rải rác, thiếu cấu trúc.

### Phạm vi
✅ Làm: giải thích khái niệm, luật, dữ liệu 2015–2025, chỉ báo, thuật ngữ.  
❌ Không làm: dự báo giá, tư vấn mua/bán, giao dịch tự động.

---

## ⚙️ Yêu cầu hệ thống

### Chức năng
- Nhận câu hỏi tiếng Việt → phân loại **intent**  
- Truy xuất thông tin phù hợp qua **BM25 + Embedding (RAG)**  
- Sinh câu trả lời chuẩn định dạng  
- Ghi **log tương tác** và **feedback người dùng**  
- Tự động từ chối các câu hỏi mang tính **khuyến nghị đầu tư**  

### Phi chức năng
- Latency < 2s (ưu tiên inference cục bộ)  
- Độ chính xác citation ≥ 90%  
- Bảo mật dữ liệu người dùng (no PII logging)  
- Khả năng mở rộng với vector DB (FAISS / Chroma)

---

## 🧩 Kiến trúc hệ thống

```
User → Intent Detector → Retriever (BM25 + Embedding)
     → Qwen + LoRA → Post-process (format + citation)
     → Logging + Response
```

### Thư mục dự án
```
PROJECT_CHATBOT/
├── app/                 # API / Web interface
├── indexes/             # Chỉ mục BM25 / FAISS
├── logs/                # Lịch sử hội thoại
├── models/              # Qwen + LoRA adapter
├── processed/           # Dữ liệu đã xử lý
├── scripts/             # Code chính (chatbot, build_index, intent...)
├── notebooks/           # Notebook thử nghiệm
├── README.md
└── requirements.txt
```

---

## 🧠 Pipeline hoạt động

1️⃣ **Nhận query người dùng**  
→ Tokenize + detect language  

2️⃣ **Phân loại intent**  
→ FAQ / Legal / Data / Indicator / Small-talk  

3️⃣ **Retrieval (RAG mỏng)**  
→ BM25 lấy top 50, embedding search rerank → lấy top 5 context  

4️⃣ **Sinh câu trả lời (LLM)**  
→ Qwen-0.5B-Instruct + LoRA, format theo:  
```
Tóm tắt
Giải thích
Rủi ro
Nguồn
```

5️⃣ **Hậu xử lý & Logging**  
→ Sanitize output, log latency, response, feedback.

---

## 🧮 Đánh giá hệ thống

| Loại | Metric | Mục tiêu |
|------|---------|----------|
| Retrieval | Recall@10 | ≥ 0.9 |
| Retrieval | MRR | > 0.7 |
| Generation | Citation Precision | ≥ 0.9 |
| Generation | Hallucination Rate | < 0.1 |
| Performance | Latency (p95) | < 2s |
| Human Eval | Helpfulness | ≥ 4/5 |

---

## 🧱 Các module chính

### `scripts/chatbot.py`
Điều phối pipeline chính (controller).  
Gồm các hàm:
- `load_model()`: nạp Qwen + LoRA  
- `retrieve_context()`: gọi BM25 + FAISS  
- `generate_response()`: sinh câu trả lời  
- `format_output()`: chuẩn hóa kết quả  
- `log_interaction()`: ghi log hội thoại  

### `scripts/build_index.py`
Xây dựng cơ chế truy xuất (indexing).  
Tạo BM25 + FAISS index từ các file dữ liệu xử lý.  

### `scripts/intent_detector.py`
Nhận dạng loại câu hỏi, định hướng pipeline.  

### `scripts/utils_legal.py`
Trích xuất, xử lý và chuẩn hóa dữ liệu pháp lý.

---

## 🔒 Bảo mật & Đạo đức

- **Không tư vấn đầu tư** – tự động từ chối khi nhận diện “mua/bán”.  
- **Minh bạch nguồn** – mọi câu trả lời phải có citation.  
- **Privacy** – không lưu thông tin cá nhân.  
- **Human-in-the-loop** – các nội dung pháp lý được kiểm duyệt thủ công.

---

## 🚀 Hướng phát triển

- Thêm ingestion pipeline cho tin tức mới.  
- Thêm module **fact-checker**.  
- Tích hợp **A/B testing** cho retriever.  
- Triển khai **feedback loop** để fine-tune liên tục.  

---

## 🏁 Kết luận

Chatbot Chứng khoán Việt Nam là **ứng dụng RAG thực tiễn** kết hợp giữa **xử lý ngôn ngữ tự nhiên, tìm kiếm ngữ nghĩa và pháp lý Việt Nam**.  
Mục tiêu: **cung cấp kiến thức minh bạch, có trích dẫn, dễ hiểu cho nhà đầu tư Việt.**  
Thiết kế tối ưu cho môi trường **nghiên cứu – học thuật – demo thực tế**.

---

## 🏁 Cách sử dụng

Download model trực tiếp **(https://huggingface.co/khoidan/Chatbot_stock_Vietnam_finetuned)**
Hoặc liên hệ email **(phamminhkhoi.05.09.12@gmail.com)**



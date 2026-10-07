# 🚀 PDF-to-Markdown Pipeline

Hệ thống tự động hóa thu thập, kiểm duyệt và bóc tách dữ liệu từ tài liệu PDF sang định dạng Markdown — được tối ưu cho nghiên cứu học thuật. Dự án tích hợp OCR đa tầng, tóm tắt bằng Gemini AI, tìm kiếm đa nguồn và giao diện desktop (GUI) hiện đại.

---

## ✨ Tính năng

### 🔄 Chuyển đổi PDF → Markdown (Hybrid Extraction)
- **Trích xuất văn bản gốc** trực tiếp từ PDF bằng `PyMuPDF`
- **Phát hiện và render bảng biểu** thông qua `pdfplumber`, tự động bỏ qua bảng nhiễu/biểu đồ giả
- **OCR tự động** bằng Tesseract (`vie+eng`) cho các trang scan hoặc có ít văn bản
- **AI transcription** bằng Gemini cho các trang chứa công thức toán bị vỡ hoặc ít hơn 100 ký tự
- **Chuẩn hóa toán học** — tự động chuyển ký tự Unicode (Greek, siêu/hạ ký tự, ký hiệu toán) sang LaTeX
- **Xử lý đa luồng** — chuyển đổi nhiều file cùng lúc với `ThreadPoolExecutor`
- **Watcher thời gian thực** — theo dõi thư mục và tự động kích hoạt khi có file mới

### 🔍 Tìm kiếm & Tải PDF
- Tìm kiếm đa nguồn: **DuckDuckGo**, **Semantic Scholar**, **arXiv**, **OpenAIRE**, **Europe PMC**, **SearXNG** (self-hosted)
- Lọc thông minh: loại bỏ slide thuyết trình, kiểm tra Magic Bytes `%PDF`, lọc theo blacklist domain
- Tóm tắt abstract bài báo bằng AI ngay trong quá trình tìm kiếm
- Lưu lịch sử tải về trong `history.json`

### 🧠 Tóm tắt bằng AI (Map-Reduce)
- **Map stage:** Tóm tắt từng file `.md` riêng lẻ bằng Gemini
- **Reduce stage:** Tổng hợp toàn bộ tóm tắt thành một báo cáo **Master Summary** duy nhất
- Lưu kết quả trung gian để tránh mất dữ liệu khi gặp lỗi rate-limit
- Retry tự động khi gặp lỗi quota/rate-limit (tối đa 3 lần)

### 🖥️ Giao diện Desktop (GUI)
- Xây dựng bằng `customtkinter`, hỗ trợ **Dark / Light mode**
- 5 tab chức năng độc lập, chạy nền bằng thread riêng (không đóng băng UI)

---

## 📂 Luồng dữ liệu

```
1_RawPDF/           ← PDF thô tải về từ Search
2_Quality_Checked/  ← Khu vực kích hoạt: chuyển PDF đạt chất lượng vào đây để xử lý
3_Result_MD/        ← File .md đã hoàn thiện
4_Summarized_files/ ← Báo cáo Master Summary do AI tổng hợp
5_ArchivedFiles/    ← File PDF gốc sau khi đã xử lý (tuỳ chọn)
```

---

## ⚙️ Cài đặt

### Yêu cầu hệ thống

| Yêu cầu | Phiên bản |
|---|---|
| Python | 3.10 – 3.12 (khuyến nghị 3.12) |
| Tesseract OCR | 5.x |
| Ngôn ngữ OCR | `eng` + `vie` |

### 1. Cài Tesseract OCR

**Ubuntu/Debian:**
```bash
sudo apt install tesseract-ocr tesseract-ocr-vie
```

**Windows:**  
Tải installer từ [UB Mannheim Tesseract](https://github.com/UB-Mannheim/tesseract/wiki) (chọn gói ngôn ngữ **Vietnamese** khi cài đặt), sau đó thêm `C:\Program Files\Tesseract-OCR\` vào PATH.

### 2. Cài đặt dự án

```bash
# Clone repository
git clone https://github.com/your-username/pdf2md.git
cd pdf2md

# Tạo và kích hoạt môi trường ảo (khuyến nghị Python 3.12)
python3 -m venv .venv
source .venv/bin/activate          # Linux/macOS
# .venv\Scripts\activate           # Windows

# Cài đặt thư viện
pip install -r requirement.txt
pip install google-genai customtkinter
```

### 3. Cấu hình API Key

Tạo file `.env` tại thư mục gốc:

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

Lấy API key miễn phí tại [aistudio.google.com/apikey](https://aistudio.google.com/apikey).

> **Lưu ý:** Các tính năng chuyển đổi PDF cơ bản (OCR + extraction) vẫn hoạt động bình thường khi không có API key. Chỉ các tính năng AI transcription và tóm tắt mới yêu cầu key.

---

## 🚀 Sử dụng

### Chạy ứng dụng GUI

```bash
python3 gui.py
# Windows: .venv\Scripts\python.exe gui.py
```

### Các tab chức năng

| Tab | Chức năng |
|---|---|
| 🔄 **Convert** | Watcher theo dõi `2_Quality_Checked/` và tự động chuyển PDF → Markdown |
| 🔍 **Search** | Tìm kiếm và tải PDF từ web, arXiv, Semantic Scholar... |
| 📊 **Summarize** | Tóm tắt hàng loạt file `.md` và tạo báo cáo Master Summary |
| ⚙️ **Settings** | Cấu hình số luồng xử lý, ngôn ngữ OCR, thư mục mặc định |
| 📜 **History** | Xem và xóa lịch sử file đã tải |

---

## 📦 Cấu trúc dự án

```
pdf2md/
├── gui.py                  # Ứng dụng GUI chính (5 tab, customtkinter)
├── core/
│   ├── auto_convert.py     # Watcher + bộ máy chuyển đổi đa luồng
│   ├── smart_convert.py    # Hybrid extraction CLI (text + OCR)
│   ├── search.py           # Tìm kiếm đa nguồn và tải PDF
│   ├── ai_summary.py       # Tóm tắt từng bài báo bằng Gemini
│   ├── master_summary.py   # Map-Reduce tổng hợp Master Summary
│   ├── archiver.py         # Lưu trữ file PDF gốc sau xử lý
│   └── history_manager.py  # Quản lý lịch sử tải (history.json)
├── 1_RawPDF/
├── 2_Quality_Checked/
├── 3_Result_MD/
├── 4_Summarized_files/
├── 5_ArchivedFiles/
├── requirement.txt
└── .env                    # Không commit — chứa GEMINI_API_KEY
```

---

## 🔧 Dependencies chính

| Package | Mục đích |
|---|---|
| `pymupdf` | Render & trích xuất nội dung PDF, render trang thành ảnh |
| `pdfplumber` | Trích xuất bảng biểu từ PDF |
| `pytesseract` | OCR cho tài liệu scan / trang ảnh |
| `pillow` | Xử lý ảnh trước khi đưa vào OCR/AI |
| `markitdown` | Chuyển đổi DOCX, PPTX, XLSX → Markdown |
| `google-genai` | Gemini AI — transcription trang toán & tóm tắt |
| `ddgs` | Tìm kiếm DuckDuckGo |
| `customtkinter` | GUI desktop hiện đại (Dark/Light mode) |
| `python-dotenv` | Đọc cấu hình từ file `.env` |
| `requests` | Tải file PDF qua HTTP |

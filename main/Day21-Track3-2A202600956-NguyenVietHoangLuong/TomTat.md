# Tóm tắt Yêu cầu Dự án: Lab 21 — LoRA Fine-tuning

## 1. Keywords Cần Chú Ý
- **Fine-tuning / LoRA / QLoRA**: Kỹ thuật tinh chỉnh mô hình LLM.
- **Unsloth, TRL (SFTTrainer), PEFT**: Các thư viện cốt lõi để huấn luyện nhanh và tiết kiệm VRAM.
- **Alpaca Format**: Định dạng cấu trúc dataset chuẩn (`instruction`, `input`, `output`).
- **Rank (r) & Alpha**: Thông số quan trọng của LoRA, lab yêu cầu thử nghiệm `r=8`, `r=16` (baseline), `r=64`.
- **Perplexity**: Chỉ số đánh giá độ tốt định lượng của ngôn ngữ (tính bằng công thức `exp(eval_loss)`).
- **Qualitative Comparison**: Đánh giá định tính (so sánh trực quan kết quả sinh ra từ mô hình Base và mô hình đã Fine-tune).
- **OOM (Out Of Memory)**: Lỗi tràn bộ nhớ VRAM, cần chú ý kiểm soát `max_seq_length` và bật `gradient_checkpointing`.
- **HuggingFace Hub**: Tùy chọn (Option B) đẩy mô hình adapter public để lấy điểm thưởng (+5 pts).

## 2. Roadmap Thực Hiện

| Bước | Hoạt động chính | Thời gian dự kiến | Đầu ra (Output) |
|:---:|---|:---:|---|
| **1** | **Setup & Dataset** | 25 phút | Môi trường sẵn sàng, tập `train_ds`, `eval_ds` (Alpaca format) |
| **2** | **Baseline r=16** | 40-50 phút | LoRA adapter (`r=16`), biểu đồ loss curve |
| **3** | **Rank Experiment** | 30 phút | 2 adapter checkpoints (`r=8` và `r=64`) cùng các chỉ số đo lường |
| **4** | **Evaluation** | 15 phút | Bảng tính (Perplexity, Time, VRAM, Params), Qualitative check |
| **5** | **Report & Submit** | 30 phút | File `REPORT.md` hoàn thiện, file zip nộp bài |

## 3. Các Bước Cần Làm Chi Tiết

### Bước 1: Chuẩn bị Dataset & Môi trường
- **Cần làm gì**: Khởi tạo Colab (chọn GPU T4, L4, hoặc A100), cài đặt các thư viện. Chuẩn bị khoảng 100-500 mẫu dữ liệu chất lượng cao.
- **Triển khai như thế nào**:
  - Dùng mẫu notebook `Lab21_LoRA_Finetuning_T4.ipynb` (nếu xài Free Colab T4).
  - Tải dữ liệu từ HuggingFace hoặc tự tạo dataset mang tính cá nhân (domain như tiếng Việt, coding, hoặc specific domain). Khuyến khích tự làm bộ dữ liệu riêng.
  - Làm sạch dữ liệu (loại bỏ trùng lặp, bỏ mẫu quá ngắn), phân tích chiều dài token để set `max_seq_length = p95` (percentile 95).
  - Chia tập train (90%) và eval (10%) với tham số `seed=42`.

### Bước 2: Configure & Train LoRA (Baseline r=16)
- **Cần làm gì**: Tải Base Model dạng 4-bit (QLoRA), thiết lập cấu hình LoRA với `r=16` và huấn luyện bằng `SFTTrainer`.
- **Triển khai như thế nào**:
  - LoRA params bắt buộc: `r=16`, `lora_alpha=32`, `target_modules=["q_proj", "v_proj"]`, `gradient_checkpointing=True`.
  - Hyperparameters: 3 epochs, Cosine LR schedule (`2e-4`), warmup ratio = `0.10`, batch size hiệu dụng = 8 (thông qua gradient accumulation), optimizer = `adamw_8bit` (paged AdamW).
  - Theo dõi train loss và eval loss để tránh hiện tượng overfitting. Lưu adapter `r=16` ngay sau khi huấn luyện xong.

### Bước 3: Rank Experiment (Phần quan trọng nhất)
- **Cần làm gì**: Huấn luyện thêm 2 adapter với các mức rank khác nhau để so sánh hiệu năng (training cost) và chất lượng (perplexity).
- **Triển khai như thế nào**:
  - Giữ nguyên toàn bộ cấu hình, dataset ở Bước 2, chỉ thay đổi rank và alpha:
    - Cấu hình 1: `r=8`, `lora_alpha=16`
    - Cấu hình 2: `r=64`, `lora_alpha=128`
  - **Bắt buộc đo lường**: Với mỗi lần train, ghi nhận 4 thông số:
    1. Thời gian train (`time.time()`).
    2. Peak VRAM (`max_memory_allocated`).
    3. Eval perplexity.
    4. Số lượng tham số Trainable (`requires_grad`).

### Bước 4: Đánh giá (Evaluation)
- **Cần làm gì**: Tính toán định lượng tổng thể và sinh câu trả lời để đánh giá bằng mắt thường.
- **Triển khai như thế nào**:
  - **Định lượng (Quantitative)**: Tổng hợp Perplexity = `exp(eval_loss)` cho cả 4 trạng thái mô hình: Base model (chưa fine-tune), r=8, r=16, r=64.
  - **Định tính (Qualitative)**: Tự tạo 5 đến 20 prompts để thử nghiệm. So sánh side-by-side (kế bên nhau) câu trả lời sinh ra giữa Base Model và Fine-tuned Model (`r=16`).

### Bước 5: Viết REPORT.md và Nộp bài
- **Cần làm gì**: Trình bày kết quả thí nghiệm và bài học rút ra vào file báo cáo markdown.
- **Triển khai như thế nào**:
  - Cấu trúc `REPORT.md` bao gồm 6 mục: (1) Setup thiết bị & môi trường, (2) Kết quả Bảng Rank Experiment, (3) Phân tích Loss Curve có bị overfitting hay không, (4) Qualitative Comparison (cung cấp 5 ví dụ minh họa), (5) Kết luận về Rank Trade-off (>= 100 từ), (6) Bài học thu nhận được ("What I learned").
  - Đóng gói file nén. Khuyến khích chọn **Option B** (tải adapter lên HuggingFace Hub công khai) để nhận 5 điểm thưởng. Đừng quên xóa sạch output thừa trong file `.ipynb` trước khi zip để tránh file quá nặng.

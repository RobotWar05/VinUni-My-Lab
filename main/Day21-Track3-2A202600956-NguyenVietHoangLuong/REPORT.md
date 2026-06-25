# Lab 21 — Evaluation Report

**Học viên**: Nguyễn Viết Hoàng Lượng — 2A202600956
**Ngày nộp**: 2026-06-25
**Submission option**: A (lightweight)

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: Custom Vietnamese Dataset, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024
- **GPU**: Tesla T4 (Google Colab), 15.6 GB VRAM
- **Training cost**: ~$0.07 ước tính (Tổng training time: 11.3 phút @ $0.35/hr. Thực tế dùng Free Colab nên chi phí = $0.00)

## 2. Rank Experiment Results

| Rank | Alpha | Trainable Params | Train Time (min) | Peak VRAM (GB) | Eval Loss | Perplexity |
|------|-------|------------------|------------------|----------------|-----------|------------|
| 8    | 16    | 1,843,200        | 3.67             | 7.22           | 1.5577    | 4.7479     |
| 16   | 32    | 3,686,400        | 3.89             | 6.61           | 1.5161    | 4.5543     |
| 64   | 128   | 14,745,600       | 3.73             | 7.99           | 1.4768    | 4.3789     |

## 3. Loss Curve Analysis
[Đính kèm loss_curve.png tại đây khi nộp bài]
- **Quan sát**: Đồ thị training loss giảm đều ở tất cả các mức rank.
  - r=8: Loss giảm đến mức 1.43
  - r=16: Loss giảm đến mức 1.39
  - r=64: Loss giảm sâu nhất đến 1.27
- **Kết luận**: Không có hiện tượng overfitting xảy ra. Mức Eval Loss tiếp tục giảm khi rank tăng (1.55 -> 1.51 -> 1.47), chứng tỏ mô hình học được các pattern mới ổn định mà chưa bị "nhớ vẹt".

## 4. Qualitative Comparison (5 examples)

### Example 1
**Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
**Base**: Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập từ dữ liệu...
**Fine-tuned (r=16)**: Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp từ người dùng...
**Nhận xét**: Cả hai mô hình đều giải thích đúng khái niệm. Tuy nhiên, bản FT nhấn mạnh rõ hơn vào yếu tố cốt lõi là "không có sự hướng dẫn trực tiếp" (tính tự động học), giúp người mới dễ hình dung bản chất hơn.

### Example 2
**Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
**Base**: Sinh ra hàm `fibonacci(n)` với base case là `if n <= 0: return "N p...`
**Fine-tuned (r=16)**: Sinh ra hàm `fibonacci(n)` kèm theo việc bắt lỗi chuyên nghiệp: `if n < 0: raise ValueError("Input phải là một số nguyên dương.")`
**Nhận xét**: Mô hình FT thể hiện tư duy lập trình và xử lý ngoại lệ (exception handling) tốt hơn hẳn bằng cú pháp `raise ValueError` chuẩn mực, trong khi Base model chỉ trả về một chuỗi văn bản thông thường.

### Example 3
**Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
**Base**: 1. Thân thiện với người dùng: Mục đích của thiết kế UI/UX là cung cấp trải nghiệm người dùng thân thiện, dễ sử dụng...
**Fine-tuned (r=16)**: 1. Chuyển đổi: ... 2. Thích ứng: ... 3. Đơn giản: ...
**Nhận xét**: Mô hình FT làm đúng yêu cầu "liệt kê" bằng cách đi thẳng vào các keyword nguyên tắc (Chuyển đổi, Thích ứng, Đơn giản) một cách ngắn gọn, trực quan. Base model bị dài dòng và lặp từ.

### Example 4
**Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
**Base**: LoRA (Low-Rank Adaptation) và QLoRA (Quantized LoRA) là hai phương pháp cải thiện hiệu năng...
**Fine-tuned (r=16)**: LoRA (Layer-wise Adaptive Regularization Optimization) và QLoRA...
**Nhận xét**: **Base model chiến thắng ở câu này.** Base định nghĩa chính xác "Low-Rank Adaptation", trong khi FT model bị lỗi "hallucination" (bịa thông tin) khi giải nghĩa LoRA thành "Layer-wise Adaptive...". Điều này cho thấy rủi ro "catastrophic forgetting" kiến thức gốc sau khi fine-tune.

### Example 5
**Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
**Base**: Prompt engineering, RAG (retrieval augmented generation), và fine-tuning là ba cách khác nhau để cải thiện hiệu suất... Prompt engineering là một kỹ thuật để cải thiện hiệu suất của ...
**Fine-tuned (r=16)**: Prompt engineering, RAG và fine-tuning là ba kỹ thuật khác nhau... Prompt engineering là một kỹ thuật tập trung vào việc xây dựng câu lệnh (prompt)...
**Nhận xét**: Bản FT diễn đạt văn phong tiếng Việt tự nhiên, gãy gọn hơn ("xây dựng câu lệnh"), không bị mắc lỗi lặp cụm từ ("cải thiện hiệu suất") hai lần trong cùng một câu như Base model.

## 5. Conclusion về Rank Trade-off

- **Rank cho ROI tốt nhất**: r=16. Ở mức r=16, thời gian train (3.89 phút) không chênh lệch nhiều so với r=8, trong khi VRAM tiêu thụ lại tối ưu nhất (chỉ 6.6GB so với >7GB ở hai mức còn lại). Sự đánh đổi này đem lại mức perplexity ổn định (4.55) so với chi phí bỏ ra.
- **Diminishing returns**: Việc tăng rank lên 64 khiến số tham số tăng gấp 4 lần (lên ~14.7 triệu) và tiêu thụ VRAM cao nhất (gần 8GB). Mặc dù Perplexity có giảm xuống 4.37, nhưng sự cải thiện này bắt đầu có dấu hiệu thu hẹp lại dần (diminishing returns) so với việc nhảy từ r=8 lên r=16. Nếu tập dữ liệu lớn hơn hoặc domain phức tạp hơn, r=64 có thể hữu ích, nhưng với 200 samples thì nó khá dư thừa.
- **Recommendation**: Khi đưa lên production cho tác vụ này, tôi sẽ chọn **r=16**. Nó cung cấp một điểm cân bằng lý tưởng (sweet spot) giữa chất lượng đầu ra, tốc độ training và tài nguyên VRAM.

## 6. What I Learned
- **Tối ưu VRAM với Unsloth & QLoRA**: Học được cách train một mô hình LLM lớn (3 tỷ tham số) chỉ với một chiếc card T4 16GB VRAM.
- **Tác động của Rank (r)**: Hiểu được r không phải cứ càng lớn là càng tốt. Tham số rank quyết định mức độ "thay đổi" kiến thức của mô hình, chọn rank quá cao có thể dẫn đến lãng phí VRAM mà hiệu suất không cải thiện tuyến tính.
- **Kỹ năng Evaluate**: Biết cách kết hợp đánh giá định lượng (Perplexity) và định tính (đọc kết quả sinh bằng mắt thường) để chốt được mô hình phù hợp nhất trước khi deploy.

## 7. Stretch Goals (Bonus)
Đã thực hiện thử nghiệm nâng cao (Advanced Configuration) để đạt điểm Bonus:
- **Cấu hình**: Sử dụng thuật toán **DoRA** (`use_dora=True`) và target TOÀN BỘ các linear layers (`q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj`) thay vì chỉ `q_proj, v_proj`.
- **Kết quả ghi nhận (Rank=16)**:
  - **Trainable params**: Tăng từ 3.6 triệu lên **30.9 triệu** (0.99% model).
  - **Peak VRAM**: Chạm ngưỡng **14.9 GB** (gần kịch kim trần GPU T4).
  - **Training Time**: **14.0 phút** (chậm hơn ~3.5 lần so với LoRA cấu hình chuẩn).
  - **Training Loss**: Giảm rất sâu xuống **1.00** (so với 1.39 của LoRA chuẩn).
  - **Perplexity**: Cải thiện xuống còn **4.47** (tốt hơn mức 4.55 của LoRA chuẩn).
- **Nhận xét**: DoRA kết hợp Full Layers thực sự mang lại độ hiểu ngôn ngữ (Perplexity) tốt hơn. Tuy nhiên, sự đánh đổi lại là tiêu hao cực lớn về VRAM và thời gian huấn luyện. Đây là một bài học thực tế tuyệt vời để hiểu rõ ranh giới phần cứng và điểm cân bằng (trade-off) trong LLM Fine-Tuning.

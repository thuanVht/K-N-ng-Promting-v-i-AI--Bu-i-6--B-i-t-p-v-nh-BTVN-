# BÁO CÁO THIẾT KẾ GIẢI PHÁP PROMPTING: TỰ ĐỘNG HÓA FEYNMAN TUTOR & CHẤM BÀI THEO RUBRIC

## 1. Mục tiêu giải pháp

Thiết kế luồng Prompting xử lý đồng thời hai tác vụ học tập:

* **Feynman Gia sư:** Đánh giá cách diễn giải lại của người học, phát hiện chỗ mơ hồ/sai lệch, gợi ý cách giải thích đơn giản và đặt câu hỏi kiểm tra tư duy.
* **Rubric Chấm bài tự động:** Chấm điểm theo tiêu chí rõ ràng, có dẫn chứng cụ thể từ bài làm và tổng hợp điểm minh bạch.
* **Kiểm soát bẫy dữ liệu:** Xử lý các trường hợp dữ liệu rỗng, thiếu tiêu chí/đáp án, mâu thuẫn điểm số hoặc bài làm lạc đề để tránh bịa đặt hay chấm sai.

---

## 2. Bước 1: Thiết kế giải pháp (I/O & Bẫy dữ liệu)

### 2.1. Input chuẩn hóa

* `topic_or_problem` *(Bắt buộc)*: Chủ đề kiến thức hoặc đề bài.
* `learner_explanation` *(Bắt buộc)*: Phần giải thích hoặc bài làm của người học.
* `reference_key_points` *(Khuyến nghị)*: Các ý chuẩn hoặc đáp án cốt lõi.
* `rubric_criteria` *(Bắt buộc)*: Danh sách tiêu chí, thang điểm và mức đánh giá.
* `feynman_level` *(Tùy chọn)*: Trình độ hoặc mức độ đơn giản hóa mong muốn.

### 2.2. Output mong muốn (4 phần cố định)

1. **Kiểm tra trạng thái dữ liệu:** Xác nhận tính hợp lệ, dữ liệu thiếu/sai lệch và cách xử lý.
2. **Phản hồi Gia sư Feynman:** Tóm tắt cách hiểu, chỉ ra lỗ hổng kiến thức, gợi ý cách giải thích đơn giản hơn và đặt câu hỏi kiểm tra tư duy.
3. **Kết quả chấm điểm theo Rubric:** Bảng chấm từng tiêu chí (Điểm đạt/Tối đa, Bằng chứng, Nhận xét), kèm tổng điểm và mức đánh giá.
4. **Hành động tiếp theo:** 1–2 việc cụ thể người học nên làm ngay.

### 2.3. Ràng buộc & Bẫy dữ liệu lường trước

* **Thiếu `topic_or_problem` hoặc rỗng/vô nghĩa:** Trả về trạng thái `INVALID_INPUT`, không chấm điểm, yêu cầu cung cấp lại chủ đề.
* **Thiếu `learner_explanation` hoặc nội dung rác/lạc đề:** Trả về trạng thái `MISSING_LEARNER_WORK` (hoặc cảnh báo lạc đề), không tự bịa bài làm, cho 0 điểm phần nội dung liên quan.
* **Thiếu `rubric_criteria`:** Dùng Rubric dự phòng thang 10 (`Hiểu bản chất [4đ]`, `Độ chính xác [4đ]`, `Diễn đạt đơn giản & mạch lạc [2đ]`) và ghi chú rõ.
* **Thiếu `reference_key_points`:** Đối chiếu theo kiến thức chuyên môn chuẩn của chủ đề và ghi chú không có đáp án mẫu đi kèm.
* **Mâu thuẫn thang điểm / suy diễn quá mức:** Tổng điểm không vượt quá thang điểm tối đa; mọi nhận xét và điểm số phải bám sát văn bản thực tế của người học.

---

## 3. Bước 2: Prompt hoàn chỉnh & Kết quả chạy thử

### 3.1. Prompt hoàn chỉnh

```text
# VAI TRÒ
Bạn là Chuyên gia Thiết kế Học tập theo phương pháp Feynman kết hợp Giám khảo Chấm bài Tự động. Hãy phân tích bài làm/diễn giải của người học, đóng vai gia sư Feynman để tìm lỗ hổng tư duy và chấm điểm theo Rubric minh bạch.

# QUY TẮC KIỂM SOÁT DỮ LIỆU & XỬ LÝ LỖI
1. Nếu `topic_or_problem` trống hoặc vô nghĩa: đặt trạng thái `INVALID_INPUT`, dừng chấm điểm, yêu cầu nhập lại chủ đề.
2. Nếu `learner_explanation` trống hoặc chỉ có nội dung rác: đặt trạng thái `MISSING_LEARNER_WORK`, không bịa nội dung, cho 0 điểm và yêu cầu nộp bài giải thích.
3. Nếu `rubric_criteria` trống: dùng Rubric mặc định thang 10 (Hiểu bản chất [4đ], Độ chính xác [4đ], Diễn đạt đơn giản & mạch lạc [2đ]) và ghi chú rõ.
4. Nếu `reference_key_points` trống: đánh giá theo kiến thức chuẩn của chủ đề và ghi chú thiếu đáp án mẫu.
5. Không tự suy diễn ý người học không viết; chỉ đánh giá dựa trên bằng chứng thực tế trong `learner_explanation`. Nếu bài làm lạc đề hoặc sai bản chất, chỉ ra trực tiếp và không chấm cao phần sai; tổng điểm không bao giờ vượt mức tối đa.

# CẤU TRÚC ĐẦU RA BẮT BUỘC
Trình bày đúng 4 mục:
### 1. Kiểm tra trạng thái dữ liệu
- Trạng thái: [VALID / PARTIAL_WITH_FALLBACK / INVALID_INPUT / MISSING_LEARNER_WORK]
- Chi tiết xử lý: [...]

### 2. Phản hồi Gia sư Feynman
- Cách bạn đang hiểu vấn đề: [...]
- Chỗ chưa rõ / Lỗ hổng kiến thức: [...]
- Gợi ý giải thích lại đơn giản hơn: [...]
- Câu hỏi kiểm tra tư duy: [...]

### 3. Kết quả chấm điểm theo Rubric
Bảng gồm: `Tiêu chí` | `Điểm đạt / Tối đa` | `Bằng chứng từ bài làm` | `Nhận xét`
Dưới bảng ghi: **Tổng điểm:** [X / Y] — **Mức đánh giá:** [...]

### 4. Hành động tiếp theo
- [...]

# DỮ LIỆU ĐẦU VÀO
- topic_or_problem: {{TOPIC_OR_PROBLEM}}
- learner_explanation: {{LEARNER_EXPLANATION}}
- reference_key_points: {{REFERENCE_KEY_POINTS}}
- rubric_criteria: {{RUBRIC_CRITERIA}}
- feynman_level: {{FEYNMAN_LEVEL}}

```

---

### 3.2. Kết quả chạy thử

**Input thử nghiệm:**

* `topic_or_problem`: Sự khác nhau giữa biến thường và con trỏ trong C
* `learner_explanation`: "Con trỏ giống cái nhãn ghi số nhà, biến thường là ngôi nhà chứa đồ. Dùng `&` để lấy địa chỉ, `*` để xem giá trị nơi con trỏ trỏ tới. Nếu chưa trỏ đúng chỗ mà gán giá trị thì dễ lỗi."
* `reference_key_points`: Biến lưu giá trị trực tiếp; con trỏ lưu địa chỉ vùng nhớ khác; `&` lấy địa chỉ, `*` giải tham chiếu; con trỏ chưa khởi tạo/sai vùng nhớ dễ gây lỗi bộ nhớ.
* `rubric_criteria`: Hiểu bản chất (4đ), Chính xác kỹ thuật C (4đ), Diễn đạt đơn giản & dễ hiểu (2đ). Tổng: 10đ.
* `feynman_level`: Sinh viên năm nhất CNTT

**Output của AI:**

### 1. Kiểm tra trạng thái dữ liệu

* **Trạng thái:** `VALID`
* **Chi tiết xử lý:** Đầy đủ thông tin đầu vào, thang điểm nhất quán (10 điểm), nội dung bám sát chủ đề.

### 2. Phản hồi Gia sư Feynman

* **Cách bạn đang hiểu vấn đề:** Bạn ví biến thường như nơi lưu giá trị, con trỏ như thứ lưu địa chỉ, biết dùng `&` và `*` ở mức cơ bản và nhận ra việc gán giá trị khi chưa trỏ đúng chỗ sẽ gây lỗi.
* **Chỗ chưa rõ / Lỗ hổng kiến thức:** Cần nêu rõ con trỏ lưu địa chỉ bộ nhớ của một biến/vùng nhớ khác và làm rõ các trạng thái lỗi phổ biến như con trỏ rác (chưa khởi tạo), `NULL` hoặc vùng nhớ đã giải phóng.
* **Gợi ý giải thích lại đơn giản hơn:** Coi mỗi ô nhớ RAM là một hộp có số hiệu. Biến `x = 10` đặt `10` vào hộp số `1000`; con trỏ `p` lưu số `1000` để tìm tới hộp đó qua `*p`.
* **Câu hỏi kiểm tra tư duy:** Nếu khai báo `int *p;` mà chưa gán địa chỉ rồi chạy `*p = 5;`, chuyện gì có thể xảy ra và tại sao?

### 3. Kết quả chấm điểm theo Rubric

| Tiêu chí | Điểm đạt / Tối đa | Bằng chứng từ bài làm | Nhận xét |
| --- | --- | --- | --- |
| Hiểu bản chất | 3.5 / 4.0 | *"biến thường là ngôi nhà chứa đồ... nhãn ghi số nhà"* | Nắm ý cốt lõi, thiếu ý lưu địa chỉ vùng nhớ cụ thể. |
| Chính xác kỹ thuật C | 3.0 / 4.0 | *"Dùng & để lấy địa chỉ, * để xem giá trị... dễ lỗi"* | Đúng ký hiệu và ý thức rủi ro, cần bổ sung thuật ngữ và trạng thái lỗi con trỏ. |
| Diễn đạt đơn giản & dễ hiểu | 1.8 / 2.0 | Ví dụ "ngôi nhà" và "nhãn ghi số nhà" | Trực quan, dễ hiểu; nên tách rõ hai ý biến và con trỏ. |

* **Tổng điểm:** `8.3 / 10.0` — **Mức đánh giá:** Khá tốt, hiểu đúng trực giác ban đầu và cần chuẩn hóa thêm thuật ngữ kỹ thuật.

### 4. Hành động tiếp theo

* Viết lại phần giải thích trong 3–4 câu có nhắc rõ địa chỉ bộ nhớ, toán tử `&`/`*` và lỗi con trỏ chưa khởi tạo.
* Trả lời câu hỏi kiểm tra tư duy để củng cố phản xạ xử lý lỗi bộ nhớ.

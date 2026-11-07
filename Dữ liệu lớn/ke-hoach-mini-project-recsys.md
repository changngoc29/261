# CO5135 Mini-project: đánh giá slide topic và kế hoạch đến hạn cuối

Lập ngày 7/11/2026. Nhóm: Võ Trung Kiên, Vũ Hưng. Deck topic: https://www.canva.com/d/dAfRXxcX6by4saG

Quy ước: [NGUỒN] có trích dẫn. [TỰ TÍNH] có phép tính. [ĐỀ XUẤT] là ý của Claude. [ĐOÁN] là chưa kiểm chứng.
Slide môn học trích theo tiêu đề: "Recommender systems" (`slide5-recsys-full-BK-2026.pdf`), "Locality Sensitive Hashing" (`LSH_lecture.pdf`), "Intro to Spark" (`Chapter 4_Spark.pdf`).

---

## 0. Kết luận nhanh

1. **Vấn đề lớn nhất: All_Beauty quá thưa để làm Collaborative Filtering.** Bộ đầy đủ có 632.0K user, 112.6K item, 701.5K rating (bảng "Grouped by Category", https://amazon-reviews-2023.github.io). Trung bình mỗi user chỉ có 701.5K / 632.0K ≈ 1.11 rating [TỰ TÍNH]. Phần lớn user chỉ chấm 1 lần, nên khi tách train/test thì user trong test gần như không có lịch sử. Khi đó cả item-item CF lẫn Latent Factor đều không dự đoán được nhiều.
2. Bản 5-core của All_Beauty chỉ còn **253 user, 356 item, 2.5K rating** (bảng 5-core, https://amazon-reviews-2023.github.io/data_processing/5core.html). Số này quá nhỏ cho môn Dữ liệu lớn, và dùng Spark sẽ không có lý do.
3. **Đề xuất: đổi sang `Beauty_and_Personal_Care` bản 5-core**, vẫn là chủ đề làm đẹp: 729.6K user, 207.6K item, 6.6M rating (cùng bảng 5-core). Trung bình ≈ 9.0 rating/user [TỰ TÍNH: 6.6M / 729.6K]. Deck chỉ cần sửa tên danh mục và các con số. Phương án nhẹ hơn là `Video_Games` 5-core (94.8K user, 25.6K item, 814.6K rating), nhưng phải đổi chủ đề.
4. **Spark làm được Latent Factor:** "`spark.ml` uses the alternating least squares (ALS) algorithm to learn these latent factors" (Spark 4.3.0 docs, ml-collaborative-filtering.html). Điều trước đây ghi [ĐOÁN] nay đã kiểm chứng.
5. **Item-item CF không có sẵn trong Spark dưới dạng mô hình gợi ý.** Nhóm phải tự cài bằng DataFrame (self-join theo user) hoặc dùng `RowMatrix.columnSimilarities`, mặc định là "the brute-force approach of computing normalized dot products" (PySpark 3.5.0 API docs). Đây là phần tốn công và tốn tính toán nhất.
6. Với dữ liệu đã đổi, thời gian còn lại (5 tuần, đến hết T6 11/12) là **khả thi** nếu giữ phạm vi: 2 mô hình + baseline + phân tích độ thưa. LSH chỉ làm nếu còn thời gian.

---

## 1. Đánh giá deck topic (4 trang)

| # | Trang | Vấn đề | Mức | Đề xuất sửa |
|---|---|---|---|---|
| 1 | 1–3 | Dùng All_Beauty (701.5K rating, ≈1.11 rating/user), quá thưa cho CF (mục 0) | **Cao** | Đổi sang `Beauty_and_Personal_Care` 5-core (6.6M rating). Sửa tiêu đề phụ trang 1–2, pill "701.5K review" và lệnh tải ở trang 3 |
| 2 | 3 | Lệnh `load_dataset(..., trust_remote_code=True)`: [ĐOÁN] thư viện `datasets` bản mới đã bỏ hỗ trợ script tải loại này, có thể chạy lỗi | Trung bình | Ghi cách tải bằng file trực tiếp (bản 5-core có file CSV `rating_only` và các bản tách `timestamp_w_his`, `last_out_w_his`, theo trang 5core.html). Spark đọc CSV trực tiếp nên đơn giản hơn |
| 3 | 2 | "Intro to Spark tr.15" chỉ nói "MLlib (Machine Learning)", chưa chứng minh Spark làm được gợi ý | Thấp | Thêm nguồn Spark docs: "spark.ml uses the alternating least squares (ALS) algorithm" |
| 4 | 4, dòng 4 | Ghi "regularization (tr.68–70)". Câu "To solve overfitting we introduce regularization" nằm ở tr.67; tr.68–70 là hình "The Effect of Regularization" | Thấp | Sửa thành "(tr.67–70)" |
| 5 | 4, dòng 6 | LSH dùng để tìm review gần trùng, tách rời bài toán gợi ý | Thấp | [ĐỀ XUẤT] Dùng LSH để tìm nhanh item giống nhau cho item-item CF. Slide có sẵn căn cứ: "Expensive step is finding k most similar customers" và "Near-neighbor search in high dimensions (LSH)" (Recommender systems tr.38). Như vậy phần mở rộng gắn thẳng vào câu hỏi 1 |
| 6 | 4 | Chưa có baseline để so sánh | Thấp | Thêm baseline "Global average", "User average", "Movie average" như bảng ở Recommender systems tr.75, và mô hình có bias r̂xi = μ + bx + bi + qi·px (tr.72) |
| 7 | 4, câu hỏi 2 | Bản 5-core đã lọc bỏ user/item ít rating, nên "sản phẩm mới" gần như không còn | Trung bình | Đo theo nhóm (user/item ít và nhiều rating), và đo "Coverage: Number of items/users for which system can make predictions" (tr.37). Spark ALS có `coldStartStrategy` "nan"/"drop" (Spark docs), dùng để đếm số dự đoán không làm được |
| 8 | 3 | [ĐOÁN] Nên gom theo `parent_asin` (sản phẩm gốc) thay vì `asin` (biến thể) | Thấp | Kiểm tra khi tải dữ liệu |

Những phần tốt, nên giữ: bố cục theo template CFAA; mỗi thách thức đều có trang slide; câu hỏi 1 rõ và đo được bằng RMSE (tr.37).

**Việc cần làm hôm nay (hạn nộp topic hết T7 7/11):** Kiên chọn danh mục. Nếu đổi, Claude sửa deck (mục 1–4 ở bảng trên) trong khoảng 30 phút, rồi Kiên nộp LMS.

---

## 2. Kiểm tra khả thi

| Hạng mục | Kết quả | Căn cứ |
|---|---|---|
| Dữ liệu miễn phí, không cần API | Có. Có link tải trực tiếp file `.jsonl.gz` và CSV 5-core | HF README; trang 5core.html |
| Kích thước file tải về | **Chưa biết.** Tôi thử tải từ máy chủ của Claude và máy ảo trên máy Kiên, cả hai đều bị proxy chặn (HTTP 403) | Kiên cần tải thử |
| Latent Factor trên Spark | Có, ALS trong `spark.ml`; tham số `rank`, `maxIter`, `regParam` (mặc định 1.0), `implicitPrefs` | Spark 4.3.0 docs |
| Item-item CF trên Spark | Không có sẵn, phải tự cài | Spark docs chỉ nêu ALS cho collaborative filtering |
| LSH trên Spark | Có lớp `pyspark.ml.feature.MinHashLSH` (trang API tồn tại); tôi chưa trích được mô tả | spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.MinHashLSH.html |
| Máy chạy | MacBook M1 (pro-m1). [ĐOÁN] RAM 8–16 GB. [ĐOÁN] ALS trên 6.6M rating chạy được ở chế độ local; nếu chậm thì lấy mẫu user | Chưa đo |
| Khác biệt với slide | Slide giải Latent Factor bằng "Stochastic gradient decent" (tr.73); Spark dùng ALS. Cần nói rõ trong báo cáo | Recommender systems tr.73; Spark docs |

**Rủi ro chính và cách giảm:**
1. Item-item CF quá nặng với 207.6K item. Cách giảm: giới hạn mỗi user tối đa N item khi self-join, hoặc chỉ tính cho top item phổ biến, hoặc dùng LSH lọc ứng viên (khớp tr.38).
2. Seminar là phần nộp riêng (slide + video khác mini-project, theo tracker), chủ đề chưa rõ. Nếu để muộn sẽ dồn vào tuần cuối.
3. Vũ Hưng có thể ít tham gia. Kế hoạch dưới đây tính sao cho một người vẫn làm xong.

---

## 3. Kế hoạch đến hạn nộp cuối

Hạn: hết T6 11/12/2026 (LMS ghi "Due: Thứ Bảy 12/12/2026, 12:00 AM", theo `tracker.md`). Phải nộp: báo cáo (.docx/.pdf), source code, slide mini-project, link video thuyết trình mini-project. Seminar nộp riêng slide + video.

| Tuần | Ngày | Việc | Kết quả | Giờ [ĐOÁN] |
|---|---|---|---|---|
| 0 | T7 7/11 | Chốt danh mục, sửa deck, nộp topic | Topic đã nộp | 1 |
| 1 | 8–14/11 | Cài PySpark (cần Java), tải dữ liệu, tạo repo. EDA: số rating/user, rating/item, phân bố sao, độ thưa | Notebook EDA, 3–4 biểu đồ | 6–8 |
| 2 | 15–21/11 | Tách train/valid/test (dùng bản `last_out_w_his` hoặc `timestamp_w_his` có sẵn). Baseline: global mean, user/item bias (tr.72, tr.75). ALS: thử `rank` và `regParam`, đo RMSE | Bảng RMSE baseline + ALS | 8–10 |
| 3 | 22–28/11 | Cài item-item CF: trừ trung bình (tr.26), cosine, chọn k láng giềng, trung bình có trọng số (tr.32). Đo RMSE và Precision at top 10 (tr.37) | Bảng so sánh 2 mô hình (trả lời câu hỏi 1) | 10–12 |
| 4 | 29/11–5/12 | Câu hỏi 2: chia user/item theo số rating, đo RMSE và Coverage từng nhóm. Nếu còn thời gian: MinHashLSH lọc item giống nhau, so thời gian chạy với cách tính hết. Bắt đầu viết báo cáo | Bảng/biểu đồ theo nhóm; bản nháp báo cáo | 8–10 |
| 5 | 6–11/12 | Hoàn thiện báo cáo, slide mini-project, quay video. Dọn code, README chạy lại được. Nộp trước T6 11/12 | Đủ 4 thứ cần nộp | 8–10 |
| Song song | từ 15/11 | Seminar: chốt chủ đề, slide, video (khác mini-project) | Slide + video seminar | chưa ước được |

Tổng mini-project [ĐOÁN]: khoảng 40–50 giờ, chưa tính seminar.

**Chia việc [ĐỀ XUẤT]:**
- Kiên: cài môi trường Spark, ALS, item-item CF, LSH (phần nặng kỹ thuật).
- Vũ Hưng: EDA, baseline, bảng/biểu đồ, bản nháp báo cáo và slide. Nếu Hưng không làm, Kiên làm thêm khoảng 12–15 giờ [ĐOÁN].

**Mốc kiểm tra:** cuối tuần 2 phải có RMSE của ALS. Nếu chưa có, cắt phần LSH. Cuối tuần 3 phải có item-item CF. Nếu chưa xong, lấy mẫu nhỏ hơn (ví dụ 20% user) và ghi rõ trong báo cáo.

---

## 4. Cấu trúc báo cáo [ĐỀ XUẤT]

1. Giới thiệu và bài toán (tr.8, tr.10).
2. Dữ liệu và EDA (độ thưa, phân bố).
3. Phương pháp: baseline có bias (tr.72), item-item CF (tr.28, tr.32), Latent Factor (tr.57, tr.65, tr.67) và ALS trên Spark.
4. Thực nghiệm: RMSE, Precision at top 10, Coverage (tr.37); thời gian chạy.
5. Thảo luận: câu hỏi 1, câu hỏi 2, hạn chế.
6. (Tùy chọn) LSH tìm láng giềng (tr.38; Locality Sensitive Hashing tr.13–14).

---

## 5. Còn chưa rõ, cần hỏi hoặc kiểm

- Rubric và format báo cáo, độ dài video: chưa có trong folder hay Gmail.
- Chủ đề seminar và ngày trình bày trước lớp.
- Kích thước file tải, RAM máy, thời gian chạy ALS: Kiên chạy thử trong tuần 1.
- Tỉ lệ điểm seminar và thi cuối kỳ (tracker ghi "chưa rõ").

## Nguồn ngoài folder môn
- https://amazon-reviews-2023.github.io (bảng thống kê theo danh mục)
- https://amazon-reviews-2023.github.io/data_processing/5core.html (bảng 5-core, link tải)
- https://huggingface.co/datasets/McAuley-Lab/Amazon-Reviews-2023 (README)
- https://spark.apache.org/docs/latest/ml-collaborative-filtering.html (ALS, Spark 4.3.0)
- https://spark.apache.org/docs/3.5.0/api/python/_modules/pyspark/mllib/linalg/distributed.html (`columnSimilarities`)

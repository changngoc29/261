# Mini-project topic CO5135, phương án C (Amazon Reviews 2023, All_Beauty): dàn ý 3 slide (bản v2, để duyệt)

Hạn nộp: hết T7 7/11/2026. Yêu cầu: "3 slide: Topic; Data sources; Problems." (`dữ liệu lớn/tracker.md`)
Nhóm: Kiên + Vũ Hưng. Chưa dựng Canva, chờ Kiên duyệt dàn ý.

Quy ước: [NGUỒN] có file/trang/URL kiểm chứng được. [ĐỀ XUẤT] là quyết định của nhóm. [ĐOÁN] chưa kiểm chứng.
Viết tắt: RECSYS = `slide5-recsys-full-BK-2026.pdf` (80 trang); LSH = `LSH_lecture.pdf` (101 trang); SPARK = `Chapter 4_Spark.pdf` (57 trang); HF = README dataset trên Hugging Face.

---

## Slide 1: Topic

**Tiêu đề:** Hệ gợi ý sản phẩm làm đẹp trên Amazon Reviews 2023 (All_Beauty) bằng Spark

**Nội dung trên slide:**
- Bài toán: từ lịch sử rating, dự đoán rating chưa biết và gợi ý sản phẩm cho từng người dùng.
- So sánh hai hướng trong bài học: Collaborative Filtering item-item và Latent Factor.
- Công nghệ: Apache Spark.

**Liên hệ slide môn học (ghi ở chân slide hoặc lời nói):**
- Mô hình bài toán: "X = set of Customers, S = set of Items, Utility function u: X × S → R" (RECSYS tr.8).
- Bài toán cốt lõi: "Extrapolate unknown ratings from the known ones" (RECSYS tr.10).
- Hai hướng được chọn nằm trong ba hướng thầy nêu: "Three approaches to recommender systems: Content-based, Collaborative, Latent factor based" (RECSYS tr.12).
- Công nghệ: hệ sinh thái Spark gồm "MLlib (Machine Learning)" (SPARK tr.15).

---

## Slide 2: Data sources

**Nội dung trên slide:**
- Amazon Reviews 2023 (McAuley Lab), danh mục All_Beauty: 701.5K review.
- Mỗi review: `rating`, `title`, `text`, `images`, `asin`, `parent_asin`, `user_id`, `timestamp`, `helpful_vote`, `verified_purchase`.
- Metadata sản phẩm tách riêng (`raw_meta_All_Beauty`).
- Tải riêng từng danh mục, không dùng API ngoài.
- Trích dẫn: arXiv:2403.03952.

**Liên hệ slide môn học:**
- Ma trận utility gồm "Customers" và "Items" tương ứng `user_id` và `asin` (RECSYS tr.8, tr.9). Cột `rating` là giá trị trong ma trận; slide ghi "e.g., 0-5 stars" (RECSYS tr.8).
- Cách thu thập rating: slide có mục "Gathering ratings" (RECSYS tr.11). Review của Amazon thuộc loại rating tường minh. [ĐOÁN] tr.11 có chia explicit/implicit; tôi mới đọc tiêu đề trang "Explicit".
- Cột `text` cho phép mở rộng sang Content-based: "Item profiles" và "TF-IDF" (RECSYS tr.15–16).

**Nguồn dữ liệu:**
- 701.5K: bảng thống kê theo danh mục trong HF, cột #Rating. Toàn bộ dataset: "We collected 571.54M reviews" (HF).
- Cách tải: `load_dataset("McAuley-Lab/Amazon-Reviews-2023", "raw_review_All_Beauty", trust_remote_code=True)` (HF, Quick Start).
- Link: https://huggingface.co/datasets/McAuley-Lab/Amazon-Reviews-2023
- Lưu ý: HF không ghi license, chỉ có trích dẫn.
- Chưa kiểm chứng: số người dùng, số sản phẩm và khoảng thời gian riêng của All_Beauty; dung lượng file.

---

## Slide 3: Problems

**Câu hỏi nghiên cứu (trên slide):**
1. Item-item CF và Latent Factor, cách nào dự đoán rating chính xác hơn trên All_Beauty?
2. Với ma trận thưa và sản phẩm mới, hệ gợi ý hoạt động ra sao?

**Thách thức (trên slide), mỗi ý kèm trang:**
- Ma trận thưa: "Most people have not rated most items" (RECSYS tr.12).
- Cold start: "New items have no ratings; New users have no history" (RECSYS tr.12).
- Hạn chế của CF: "Sparsity", "First rater", "Popularity bias" (RECSYS tr.34).
- Chọn item-item vì "In practice, it has been observed that item-item often works better than user-user"; lý do thầy nêu: "Items are simpler, users have multiple tastes" (RECSYS tr.33).
- Latent Factor: tìm P và Q sao cho tối thiểu tổng bình phương lỗi trên các rating đã biết (RECSYS tr.65). Nguy cơ overfitting: "SSE on test data begins to rise for k > 2" (RECSYS tr.66), cần regularization (RECSYS tr.68–70).
- Đánh giá: RMSE và Precision at top 10 (RECSYS tr.37).
- Quy mô: Spark để xử lý phân tán, mở rộng sang danh mục lớn hơn.

**Mở rộng (nếu còn thời gian):** tìm review gần trùng bằng LSH: "find near-duplicate documents among N = 1 million documents" (LSH tr.13), quy trình Shingling → Min-Hashing → LSH (LSH tr.14).

**Hạn chế:** chỉ một danh mục; chỉ dùng rating, chưa dùng text.

---

## Điểm cần nói rõ
- [ĐOÁN] Spark MLlib có phân rã ma trận cho gợi ý. SPARK tr.15 chỉ liệt kê "MLlib (Machine Learning)", slide thầy không nói về ALS hay gợi ý. Cần kiểm tra thêm trước khi hứa trong báo cáo.
- Tên topic, hai câu hỏi và phần mở rộng là [ĐỀ XUẤT] của Claude.
- Quy mô: All_Beauty 701.5K review. [ĐOÁN] chạy được trên máy thường, chưa tải thử.

## Việc cần Kiên chốt
1. Duyệt dàn ý; chỗ nào cần sửa.
2. Vũ Hưng đồng ý (Kiên báo là chắc chắn, chưa xác nhận trực tiếp).
3. Sau khi duyệt: dựng Canva hay PowerPoint. Nộp LMS do Kiên làm.

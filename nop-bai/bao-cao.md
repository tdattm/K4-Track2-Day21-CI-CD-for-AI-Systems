# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Tiến Đạt |
| MSSV | 2A202601678 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/tdattm/K4-Track2-Day21-CI-CD-for-AI-Systems.git |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 200 | 0.1 | 5 | 0.714932 | 0.874 |
| 2 | 100 | 0.1 | 3 | 0.710900 | 0.878 |
| 3 | 50 | 0.05 | 2 | 0.605128 | 0.846 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Cấu hình `n_estimators=200`, `learning_rate=0.1`, `max_depth=5` được chọn vì có F1 cao nhất, 0.714932, và vượt ngưỡng 0.65. Cấu hình `n_estimators=100`, `learning_rate=0.1`, `max_depth=3` có accuracy cao nhất, 0.878, nhưng F1 chỉ 0.710900. Do đó, accuracy cao nhất không đồng nghĩa với hiệu quả tốt nhất trên lớp thu nhập cao. Không thể kết luận độc lập về đánh đổi giữa `n_estimators` và `learning_rate` vì `max_depth` cũng thay đổi giữa các lần chạy.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập Adult mất cân bằng: lớp thu nhập trên 50K chỉ chiếm 24,8% số mẫu. Một mô hình luôn dự đoán “thu nhập thấp” vẫn có accuracy khoảng 0,752, nhưng không phát hiện được bất kỳ mẫu thu nhập cao nào và có F1 lớp dương bằng 0. Accuracy vì thế dễ phản ánh sai chất lượng do bị lớp đa số chi phối. F1 kết hợp precision và recall, trực tiếp đánh giá khả năng nhận diện lớp thu nhập cao. Pipeline đặt ngưỡng `f1_score >= 0.65`; accuracy chỉ để tham khảo. Khi tính F1 cần dùng `f1_score(y_eval, preds)` mặc định cho lớp dương, không dùng `average="weighted"` hoặc `average="macro"` vì chúng không đánh giá riêng lớp cần quan tâm.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Không cài được `scikit-learn==1.4.2`. | Python 3.13 không tương thích với dependency ghim. | Dùng virtual environment Python 3.11. |
| MLflow UI không thấy run. | Train và UI dùng tracking URI khác nhau. | Dùng cùng backend URI. |
| Release không SSH được vào VM. | Deploy key sai và security group chỉ cho IP cá nhân. | Cập nhật key và quyền SSH cho GitHub Actions. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung `train_batch2`, F1 tăng từ 0.7149 lên 0.7354 và accuracy tăng từ 0.8740 lên 0.8820. Mức tăng nhỏ này cho thấy dữ liệu mới giúp mô hình cải thiện trên holdout, nhưng không cho phép kết luận rằng thêm dữ liệu luôn làm chỉ số tăng. Điều quan trọng được kiểm chứng là commit dữ liệu đã tự động kích hoạt toàn bộ pipeline và triển khai mô hình mới sau khi qua Quality Gate.

# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

<!--
HƯỚNG DẪN - đọc rồi XÓA TOÀN BỘ các khối chú thích này sau khi điền xong:

  - Giới hạn: KHÔNG QUÁ 1 TRANG A4, tương đương khoảng 450 - 550 từ nội dung.
  - Chỉ điền vào các chỗ ___ và các ô trong bảng. Không thêm mục mới.
  - Viết bằng câu hoàn chỉnh, không gạch đầu dòng cụt lủn.
  - Kiểm tra độ dài sau khi đã xóa hết chú thích:
        wc -w nop-bai/bao-cao.md
    và xem trước bản in bằng cách mở file trên GitHub rồi Ctrl+P / Cmd+P.
-->

| | |
|---|---|
| Họ và tên | Nguyễn Tiến Đạt |
| MSSV | 2A202601678 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/tdattm/K4-Track2-Day21-CI-CD-for-AI-Systems.git |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

<!-- Khoảng 120 - 150 từ. Điền kết quả thật từ MLflow UI ở Bước 1, tối thiểu 3 lần chạy. -->

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 200 | 0.1 | 5 | 0.714932 | 0.874 |
| 2 | 100 | 0.1 | 3 | 0.710900 | 0.878 |
| 3 | 50 | 0.05 | 2 | 0.605128 | 0.846 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Cấu hình `n_estimators=200`, `learning_rate=0.1`, `max_depth=5` được chọn vì có `f1_score=0.714932`, cao nhất trong ba cấu hình và vượt ngưỡng chất lượng 0.65. Cấu hình `n_estimators=100`, `learning_rate=0.1`, `max_depth=3` có accuracy cao nhất, 0.878, nhưng F1 chỉ là 0.710900. Vì vậy, lần có accuracy cao nhất không trùng với lần có F1 cao nhất và không phải cấu hình được chọn. Việc này cho thấy accuracy không đủ để quyết định chất lượng khi lớp thu nhập cao là lớp cần quan tâm. Không thể kết luận trực tiếp về đánh đổi giữa `n_estimators` và `learning_rate` từ ba cấu hình trên vì `max_depth` cũng thay đổi đồng thời. Theo hướng dẫn của bài, learning rate thấp thường cần nhiều estimators hơn, nhưng các lần chạy này không phải bằng chứng độc lập cho nhận xét đó.

<!--
Trả lời trong phần Lý do:
  - Vì sao bộ này tốt hơn các bộ còn lại (dựa trên f1_score, không phải accuracy)?
  - Lần chạy có accuracy cao nhất có trùng với lần có f1_score cao nhất không?
    Nếu không, điều đó nói lên điều gì?
  - Bạn quan sát thấy đánh đổi nào giữa n_estimators và learning_rate?
-->

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

<!-- Khoảng 120 - 150 từ. -->

Tập dữ liệu Adult mất cân bằng: lớp thu nhập trên 50K chỉ chiếm 24,8% số mẫu. Do đó, một mô hình luôn dự đoán “thu nhập thấp” vẫn có accuracy khoảng 0,752, dù không phát hiện được bất kỳ trường hợp thu nhập cao nào. Con số accuracy này dễ gây hiểu nhầm vì nó chủ yếu phản ánh lớp đa số. Với mô hình dự đoán như vậy, F1 của lớp dương bằng 0, vì precision và recall cho lớp thu nhập cao đều không hữu ích. F1 kết hợp precision và recall, nên phản ánh trực tiếp hơn khả năng tìm đúng các mẫu thu nhập cao mà bài toán quan tâm. Vì vậy, ngưỡng chất lượng của pipeline đặt trên `f1_score >= 0.65`, còn accuracy chỉ được ghi nhận để tham khảo. Khi tính F1, cần dùng `f1_score(y_eval, preds)` mặc định cho lớp dương; không dùng `average="weighted"` hoặc `average="macro"`, vì các cách trung bình này không còn đánh giá riêng lớp dương theo yêu cầu của lab.

<!--
Cần nêu được:
  - Phân bố lớp của tập dữ liệu (tỷ lệ lớp thu nhập > 50K) và hệ quả của nó.
  - Accuracy của một mô hình luôn trả lời "thu nhập thấp" là bao nhiêu, vì sao con số
    đó gây hiểu nhầm.
  - F1 của lớp dương đo điều gì mà accuracy không đo được.
  - Vì sao KHÔNG dùng average="weighted" hay average="macro" khi gọi f1_score.
-->

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

<!-- Nêu 2 - 3 khó khăn thật, mỗi ô một câu ngắn. -->

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Cài đặt `scikit-learn==1.4.2` thất bại. | Môi trường ban đầu dùng Python 3.13, không tương thích với dependency được ghim. | Tạo virtual environment bằng Python 3.11 rồi cài lại dependencies. |
| MLflow UI không hiển thị các lần chạy. | `.env` không tự nạp biến môi trường vào PowerShell, nên train và UI dùng các tracking URI khác nhau. | Khởi động UI với backend trùng tracking URI, hoặc đặt `MLFLOW_TRACKING_URI` trước khi train. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

<!-- Lấy số liệu từ bảng ở mục 3.6 của tasks/buoc-3.md. -->

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | ___ | ___ |
| Bước 3 (thêm `train_batch2`) | ___ | ___ |

**Nhận xét:** ___

<!--
Một câu trả lời trung thực kiểu "f1 giảm 0,01 vì dữ liệu mới cùng phân phối, không mang
thêm thông tin mới" được đánh giá cao hơn kết luận sai rằng thêm dữ liệu luôn tốt hơn.
-->

---

## 5. Phần Bonus Đã Thực Hiện (nếu có)

<!-- Xóa cả mục 5 nếu không làm bonus. Mỗi bonus tối đa 1 dòng. -->

- [ ] Bonus 1 - Tracking MLflow từ xa với DagsHub: ___
- [ ] Bonus 2 - Điều chỉnh ngưỡng quyết định: ___
- [ ] Bonus 3 - Báo cáo precision / recall tự động: ___
- [ ] Bonus 4 - Hoàn trả về phiên bản trước: ___
- [ ] Bonus 5 - Cảnh báo lệch lạc dữ liệu: ___

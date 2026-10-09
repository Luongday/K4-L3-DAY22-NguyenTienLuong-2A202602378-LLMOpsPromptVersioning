# Phân tích kết quả: Prompt V1 so với V2

Nguồn số liệu: `03_ragas_report.json` (50 cặp QA, mỗi cặp chạy qua cả hai prompt; `gpt-4o-mini`, temperature 0, retriever k=3, chunk_size=500). Ảnh bảng kết quả: `03_ragas_scores.png`.

## Điểm RAGAS

| Chỉ số | V1 (ngắn gọn, 2-4 câu) | V2 (có cấu trúc, 3-5 câu) | Chênh lệch |
|---|---|---|---|
| faithfulness | **0,9727** | 0,9448 | V1 +0,028 |
| answer_relevancy | **0,9205** | 0,8907 | V1 +0,030 |
| context_recall | 1,0000 | 1,0000 | bằng nhau |
| context_precision | 0,9417 | 0,9450 | V2 +0,003 |

Cả hai phiên bản đều đạt mục tiêu faithfulness ≥ 0,8 và vượt cả mốc 0,9.

## Nhận xét

**Hai chỉ số về retrieval gần như giống nhau, đúng như dự kiến.** `context_recall` và `context_precision` chỉ phụ thuộc vào các đoạn context được truy xuất. Cả hai prompt dùng chung một retriever (FAISS, k=3) nên context đầu vào là như nhau. Phần chênh lệch nhỏ ở `context_precision` (0,003) không thể do prompt gây ra; nó cho thấy mức nhiễu của bộ chấm LLM, và là thước đo hữu ích: các chênh lệch lớn hơn mức này (như ở faithfulness và answer_relevancy, khoảng 0,03) mới có khả năng phản ánh sự khác biệt thật.

**V1 nhỉnh hơn V2 ở hai chỉ số về câu trả lời.** Giả thuyết giải thích (chưa kiểm chứng bằng thí nghiệm riêng):

- *faithfulness:* V2 yêu cầu "xác định các facts liên quan" và viết dài hơn (3-5 câu), nên có nhiều nhận định hơn. Mỗi nhận định thêm vào là một cơ hội để diễn giải lệch khỏi context, và RAGAS tính faithfulness theo tỉ lệ nhận định được context hỗ trợ. V1 viết ngắn hơn, ít nhận định hơn, ít rủi ro hơn.
- *answer_relevancy:* câu trả lời dài hơn của V2 có thể kèm thông tin ngoài trọng tâm câu hỏi, làm giảm độ tương đồng giữa câu hỏi gốc và các câu hỏi RAGAS sinh ngược từ câu trả lời.

## Độ tin cậy và giới hạn

- Có hai lần chạy hoàn chỉnh (đủ 4 chỉ số cho cả hai phiên bản). Cả hai đều cho cùng chiều: V1 hơn V2 ở faithfulness (lần trước 0,9675 so với 0,9473) và ở answer_relevancy (lần trước 0,9168 so với 0,8914). Điều này làm tăng độ tin cậy, nhưng vẫn chỉ là hai lần chạy trên 50 mẫu, nên đây là xu hướng chứ chưa phải kết luận chắc chắn.
- Một lần chạy khác bị bỏ khỏi báo cáo vì V1 có điểm `NaN` ở faithfulness và context_precision (bộ chấm lỗi trên một số mẫu). Script đã được sửa để loại các mẫu `NaN` thay vì để chúng làm hỏng điểm trung bình, rồi chạy lại.
- Bộ chấm RAGAS là một LLM nên bản thân nó có độ nhiễu (xem phần context_precision ở trên).
- Cột "Winner" trong bảng của script gán nhãn theo phép so sánh `>`, nên khi hai điểm bằng nhau (context_recall = 1,0000) nhãn là tùy ý, không có ý nghĩa.

## A/B routing (bước 2)

Hash MD5 của `request_id` phân 50 câu hỏi thành V1=19 câu, V2=31 câu. Cùng `request_id` luôn ra cùng phiên bản. Mức lệch 19/31 là bình thường với hash trên 50 mẫu nhỏ, không phải lỗi.

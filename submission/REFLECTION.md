# Reflection: Lakehouse Anti-Patterns

### Anti-Pattern: Small-File Ingestion Without Compaction
Trong 5 Anti-Pattern của Lakehouse, hệ thống streaming telemetry/CDC mà tôi quan tâm dễ vướng phải **Small-File Problem** nhất. Khi micro-batch nạp liên tục mỗi vài giây, hàng triệu file Parquet li ti sinh ra. Vấn đề không chỉ làm phình metadata khiến reader mở footer chậm hàng chục giây, mà còn đẩy chi phí API request (`GET` S3/GCS) tăng vọt theo cấp số nhân (như đo đạc ở NB2 và NB6).

**Cách phòng tránh:**
1. Thiết lập cron job định kỳ chạy `OPTIMIZE / compact` đưa file về cỡ chuẩn (128–512 MB).
2. Chạy `Z-ORDER` theo khóa truy vấn thường xuyên để data skipping qua min/max stats.
3. Chạy `VACUUM / expire_snapshots` định kỳ và sweep orphan files để giải phóng dung lượng đĩa.

*(AI Usage: Sử dụng Antigravity AI hỗ trợ giải thích kiến trúc và kiểm tra lỗi; toàn bộ bài lab và tests đều chạy thực tế trên máy cá nhân).*

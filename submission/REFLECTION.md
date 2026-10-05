## Top 5 Lakehouse Anti-Patterns - Reflection

Trong số các anti-pattern, hệ thống dữ liệu RAG (truy xuất nội dung tài liệu) mà tôi quan tâm dễ vướng nhất vào lỗi **"Lưu file quá nhỏ (Small files)"** hoặc **"Zombie/Orphan files"**. 

Lý do là hệ thống RAG liên tục tiếp nhận các tài liệu mới nhỏ lẻ, mỗi lần crawl thêm dữ liệu hệ thống lại ghi một vài chunk vào Data Lakehouse. Theo thời gian, điều này tạo ra hàng triệu file Parquet cực nhỏ, khiến hiệu suất đọc (scan) bị giảm sút nghiêm trọng do metadata overhead lớn. Bên cạnh đó, việc thường xuyên thay thế, xóa hoặc update nội dung cũ sẽ tạo ra vô số orphan files và snapshot cũ (nếu không vacuum/expire snapshots định kỳ), làm tốn kém chi phí lưu trữ S3. 

**Cách phòng tránh:** Cần định kỳ chạy job `OPTIMIZE` kết hợp `Z-ORDER` (hoặc `COMPACTION` trong Iceberg) để gom các file nhỏ lại, và chạy job `VACUUM` / `EXPIRE SNAPSHOTS` mỗi tuần để dọn dẹp các tệp mồ côi.

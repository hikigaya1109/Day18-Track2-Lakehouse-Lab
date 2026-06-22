# Reflection — Top 5 Lakehouse Anti-Patterns

**Họ tên:** Nguyễn Quang Minh  
**MSSV:** 26A202600994

## Anti-pattern dễ vướng nhất: Small File Problem

Trong bối cảnh hệ thống LLM observability như NB4, team tôi rất dễ rơi vào bẫy **small file problem**.

Lý do cụ thể: mỗi request gọi LLM đều được log ngay lập tức theo kiểu streaming ingestion — tức là hàng nghìn batch nhỏ ghi liên tục vào Bronze layer. Nếu không có chiến lược OPTIMIZE định kỳ, sau vài ngày vận hành, Bronze table sẽ tích lũy hàng chục nghìn file nhỏ vài KB. Kết quả là mỗi query phải mở và đọc metadata của toàn bộ file đó — latency tăng vọt, đặc biệt nghiêm trọng khi dashboard realtime cần query Gold layer liên tục.

NB2 đã minh họa rõ điều này: 200 small files khiến query mất 120.8ms, sau khi OPTIMIZE + Z-ORDER còn 55 files và chỉ mất 11.4ms — **nhanh hơn 10.6×**.

Bài học rút ra: cần lên lịch chạy `OPTIMIZE + Z-ORDER` định kỳ (ví dụ mỗi 6 giờ) trên Bronze/Silver tables, đặt `target_size` phù hợp với khối lượng data thực tế để tránh vòng lặp compact liên tục gây tốn compute.

> Word count: ~170 words

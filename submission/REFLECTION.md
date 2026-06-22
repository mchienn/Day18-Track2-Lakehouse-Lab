# Reflection — Top 5 Lakehouse Anti-Patterns

Anti-pattern team tôi dễ vướng nhất: **"Small-file problem"** — accumulate too many tiny files from streaming ingestion, then wonder why queries are slow.

Team tôi đang chạy nhiều micro-batch streams (Kafka → Delta, 10s interval), mỗi batch ghi 500–5K rows. Sau một tuần, một bảng có thể có 10K+ files. Query scan metadata overhead rất lớn, đặc biệt với partition pruning không hiệu quả vì mỗi partition có quá nhiều file nhỏ.

Bài học từ lab (NB2) cho thấy: `OPTIMIZE ZORDER` giảm 200 files → 55 files, speedup 10.2×, và files-pruned ratio 55×. Trong production, team cần scheduling compact job định kỳ (e.g., hourly) và đặt target size hợp lý (~256 MB) thay vì default 1 GB để Z-order còn có files để skip.

Anti-pattern khác cũng đáng lo: **"No schema enforcement"** — khi nhiều team cùng ghi vào một Delta table, dễ có schema drift. Lab NB1 confirm schema enforcement blocking + opt-in evolution là giải pháp.

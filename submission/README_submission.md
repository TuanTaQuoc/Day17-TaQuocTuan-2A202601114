# Lab 17 Submission

**Họ và tên:** Tạ Quốc Tuấn
**MSSV:** 2A202601114

---

## Phân tích Benchmark

**1. Layer nào có hit rate thấp nhất?**
Trong student run, tất cả 4 layer đều đạt 100% (11/11 PASS). Tuy nhiên, ở no-memory baseline, các layer `long_term`, `episodic`, `semantic` đều đạt 0% (0/9 case), chỉ `short_term` đạt 100% (2/2) vì dữ liệu vẫn còn trong thread hiện tại. Layer phụ thuộc Zep hoàn toàn sụp đổ khi không có memory retrieval.

**2. Query nào retrieve nhiều token nhất?**
E03 và E08 đều retrieve **913 tokens** — cả hai là `long_term` case. Zep trả về full Context Block gồm USER_SUMMARY, EPISODES, FACTS, ENTITIES và THREADS, dẫn đến lượng token lớn.

**3. E07 cần kết hợp layer nào? Evidence bắt buộc là gì?**
E07 (mixed) cần `long_term` (320 tokens dùng) và `semantic` (148 tokens dùng). Hai evidence bắt buộc: **`Python`** — preference cá nhân của Minh cho ORCHID-27, lấy từ long_term Context Block; và **`Idempotency-Key`** — quy tắc PAYMENT-RULE-3, lấy từ standalone semantic graph.

**4. Token reduction và tại sao no-memory reduction cao nhưng hit rate thấp?**
Student avg reduction **20.2%** với hit rate 100%. No-memory avg reduction **81.8%** nhưng hit rate chỉ 18.2%. No-memory "tiết kiệm" cao vì không retrieve gì — context gần như rỗng, nên so với full_source_tokens thì reduction lớn. Nhưng không có cross-session evidence → fail hoàn toàn với 9/11 case phụ thuộc durable memory.

---

## Reflection

**Layer quan trọng nhất trong bộ test này?**
`long_term` — chiếm 4 case (E02, E03, E08, E09 = 20đ), quan trọng hơn vì E08 test recency/conflict (scope-specific preference) và E09 test user isolation (Lan không được recall fact của Minh).

**Trade-off Zep Context Block vs Redis + Qdrant?**
Zep tự động extract facts, entities và tổng hợp Context Block mà không cần pipeline riêng, nhưng latency cao (~1200–1800ms) và phụ thuộc cloud. Redis + Qdrant cho phép kiểm soát hoàn toàn schema, latency thấp hơn, nhưng phải tự xây ingestion, embedding và conflict resolution.

**Guardrail chống memory poisoning?**
`require_memory_consent()` từ chối ingest nếu không có opt-in; `minimize_pii()` redact email/phone trước khi gửi Zep; heartbeat chỉ được phép deduplicate/mark stale/tạo recap, không được tự thêm instruction hoặc quyền mới vào durable memory.

---

## E08 Recency và E10 Compaction

**E08:** Minh ban đầu prefer Python cho tất cả dự án. Session sau update: BLUEBIRD-42 bắt buộc TypeScript + NestJS. Zep giữ cả hai fact ở scope khác nhau — Python vẫn đúng cho ORCHID-27, TypeScript đúng cho BLUEBIRD-42. Query về BLUEBIRD-42 → Context Block ưu tiên fact mới nhất đúng scope, không xóa fact cũ.

**E10:** Compaction evict 6 filler turn nhưng vẫn giữ `REVIEW-DEADLINE-1600 / Friday 16:00` qua durable note. Constraint quan trọng hơn chit-chat — summary phải ưu tiên state, decision, TODO và constraint, không phải nội dung hội thoại thông thường.

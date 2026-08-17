# Lab 17 — Multi-Memory Agent với Zep (bài nộp)

**Kết quả:** practice **11/11 PASS** (hit rate 100%), golden **20/20** (`perfect = true`, +10), no-memory **2/11** (18.2%). Latency TB 1149.6 ms, token reduction TB 19.1%. Nguồn: `reports/benchmark.json`, `reports/golden_benchmark.json`, `reports/comparison.md`.

## Phân tích benchmark

- **Layer khó nhất: long-term** (E02, E03, E08, E09). E03 chỉ có "benchmark report" + "16:00" khi edge search dùng `limit=20`; E09 phải search theo `user_id` để không rò `ORCHID-27` sang Lan.
- **Query tốn token nhất: E02** — 1498 token (Context Block + 20 fact edge), rồi E03 (1484) và E08 (1461).
- **E07 = long-term + semantic**, hai evidence: `Python` (user graph) và `Idempotency-Key` (shared graph). Budget: long-term 1501 → 324 (trim), semantic 53/240, merged 390.
- **Token reduction.** Semantic nén mạnh nhất (E06 88.5%, E11 90.8%); long-term ~0% vì transcript synthetic chỉ ~221 token còn Context Block tóm tắt cả ba session — lợi ích nén xuất hiện khi lịch sử dài ra. No-memory reduction 81.8% nhưng hit rate 18.2%: retrieve rỗng thì rẻ mà sai.
- **Golden dạy một bài về budget.** G18 (episodic+semantic) ban đầu fail vì Zep trả mỗi document hai lần (JSON + text) kèm dòng `metadata=` rỗng, đẩy `BUDGET-10-4-3-3` qua ranh giới trim 240 token. Sửa ở nguồn: khử trùng lặp, giữ bản ngắn nhất mỗi document (369 → 184 token), không nới `LAB_CONTEXT_TOKENS`.

## Reflection

1. **Layer quan trọng nhất: long-term** (4/11 practice, 4/20 golden). Nó là thứ duy nhất sống qua thread mới: E02/E03 lấy preference và open loop từ session cũ, E09 chứng minh isolation theo user.
2. **Trade-off Zep vs Redis+Qdrant.** Redis+Qdrant rẻ, latency ~0 ms, kiểm soát hoàn toàn, nhưng phải tự viết extraction, resolution, temporal validity, summarization. Zep trả sẵn Context Block (summary + facts + entities kèm `valid_at`/`invalid_at`) nên giải được recency conflict E08, đổi lại latency 1–3 s và phụ thuộc vendor.
3. **Guardrail.** Consent gate (`require_memory_consent`) + `minimize_pii` trước mọi ingest; `prime_eval_thread` dùng `ignore_roles` để câu hỏi chấm không thành durable fact (chống self-poisoning); background write chạy `--dry-run`; xóa phải verify ở mọi store.

## E08 recency và E10 compaction

E08 pass vì conflict giải theo **scope + recency**: `BLUEBIRD-42 → TypeScript/NestJS` thắng cho project đó, Python vẫn đúng cho `ORCHID-27`. E10 pass sau **8 lần compaction**, chỉ giữ 6 message: `REVIEW-DEADLINE-1600`, `Friday`, `16:00` sống sót nhờ được promote thành durable note trước khi turn cũ bị evict.

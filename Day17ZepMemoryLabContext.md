# Day 17 — Xây dựng Multi-Memory Agent với Zep

**Track 3 | VLearn Codelabs**
Thời lượng: 170 phút · Mức độ: Trung cấp
Nguồn: https://codelabs.vlearn.dev/codelab/day-17-xay-dung-multi-memory-agent-voi-zep

## Tổng quan

Bạn hoàn thiện bốn contract retrieval trong một starter kit đã có sẵn ingestion, evaluator và Docker; sau đó chứng minh agent nhớ đúng qua session, đúng user, đúng knowledge scope và đúng token budget.

**Bài này đang nói về điều gì?**

- Mỗi loại ký ức (memory layer) có scope và cách truy xuất riêng; nhầm `user_id` với `graph_id` có thể gây leak hoặc lấy sai knowledge.
- Token reduction chỉ có ý nghĩa khi đi cùng evidence hit rate; retrieval rỗng rất rẻ nhưng không đúng.

**Luồng công việc chính:** Đọc dataset và contract → Seed user graph cùng shared graph → Retrieve đúng memory layer → Assemble theo context budget → Score evidence và nộp artefact.

### Lịch trình buổi lab

| Thời gian | Nội dung |
|---|---|
| 0:00–0:15 | Khởi động và baseline: kiểm tra repo, cấu hình môi trường, chạy smoke test, ghi nhận no-memory baseline. |
| 0:15–0:45 | Short-term memory: so sánh buffer, summary và sliding window; quan sát compaction vẫn giữ durable constraint. |
| 0:45–1:25 | Long-term memory: hoàn thiện Context Block cho cross-session recall, recency và user isolation. |
| 1:25–1:50 | Episodic, semantic và context budget: hoàn thiện hai graph search, ghép mixed context, chạy practice benchmark, chuẩn bị privacy evidence. |
| 1:50–2:50 | Golden set và mini-product: sau khi giảng viên phát golden set, chạy bộ 20 case và chọn làm UI hoặc report nâng cao nếu còn thời gian. |

**Kết thúc bài, bạn có gì?**

- Một implementation student đạt ngưỡng practice và sinh đủ benchmark, comparison, reflection cùng bằng chứng privacy.
- Một quy trình memory có consent, provenance, compaction, user isolation và right-to-be-forgotten được kiểm tra bằng artefact thật.

---

## Bước 1 — Mở đúng repo và xác định phần được phép sửa

Repo Day 17 — Zep Memory for Agent. Kiểm tra đang đứng tại root có `README.md`, `LAB.md`, `docker-compose.yml`, `src/`, `tests/`, `data/`:

```bash
git remote get-url origin
git status --short
rg --files
```

Remote của starter kit: `https://github.com/VinUni-AI20k/Day17-Track3-ZepMemory4Agent.git`. Nếu lệnh đầu in URL khác, xác nhận lại fork hoặc repo giảng viên giao trước khi code.

### Khu vực – Quyền và mục đích

| Khu vực | Quyền và mục đích |
|---|---|
| `src/memory_student.py` | File code bắt buộc: hoàn thiện đúng bốn LAB TODO. |
| `src/demo_ui.py` | Chỉ sửa nếu làm bonus mini-product; phần còn thiếu là `retrieve_for_case`. |
| `README_submission.md` | File bạn tạo để trả lời reflection và phân tích benchmark, tối đa 400 từ. |
| `reports/` | Nơi evaluator sinh JSON, Markdown, comparison và report HTML tùy chọn. |
| `submission/` | Thư mục bạn có thể tạo để lưu bốn ảnh minh chứng. |
| `src/demo_short_term.py` | Chỉ đổi `max_recent_messages` tạm thời để quan sát rồi khôi phục. |
| `tests/`, `data/`, `src/evaluate.py`, `src/context_budget.py`, `src/zep_common.py` | Đọc để hiểu contract; không sửa để làm điểm tăng. |
| `src/memory_reference.py` | Reference/instructor demo; không copy rồi đổi tên thành bài student. |
| `control_plane/` | Identity, context, memory schema và task policy để đọc, không phải phần code bắt buộc. |

**Hành trình của bài:** Đọc contract và dataset → Chạy unit test cùng no-memory baseline → Quan sát short-term compaction → Long-term Context Block → Episodic user graph → Semantic shared graph → Assemble context budget → Benchmark và comparison → Privacy, reflection và submission → Golden hoặc UI tùy chọn.

> ⚠️ Đừng dùng `git add -A` ngay từ đầu. Repo có thể đang chứa file cá nhân hoặc artefact chưa theo dõi; chỉ stage đúng file bài nộp ở bước cuối.

**Kết quả mong đợi:** bạn biết chính xác bốn hàm bắt buộc, phần bonus, artefact cần tạo và các file scaffold phải giữ nguyên.

---

## Bước 2 — Hiểu bốn lớp memory và luật chấm

Một memory layer không chỉ là một chỗ lưu text. Lab yêu cầu chọn đúng scope để evidence không bị lẫn giữa thread, user và knowledge dùng chung.

| Layer | Scope và backend | Case practice | Evidence tiêu biểu |
|---|---|---|---|
| Short-term | Thread hiện tại, class local `ShortTermMemory` | E01, E10 | ORCHID-27, deadline cũ sau compaction |
| Long-term | User graph và Context Block của Zep | E02, E03, E08, E09 | Preference, open loop, recency và user isolation |
| Episodic | Episode trong user graph | E04, E05 | Trajectory debug, outcome và reflection |
| Semantic | Standalone graph dùng chung | E06, E11 | Payment retry rule và incident playbook |
| Mixed | Ghép nhiều layer | E07 | Preference Python cùng Idempotency-Key |

**Dataset:**

- `data/sessions.json` — nguồn evaluator thật: chứa hai synthetic user, bốn session theo ba stage và 11 evaluation case.
- `data/ground_truth.json` — bản trích để đọc nhanh; scorer **không** tải file này.
- `data/knowledge.jsonl` — bốn document dùng chung (semantic knowledge).

**Luồng dữ liệu chính:**

```
data/sessions.json → Consent và PII minimization → Zep user, thread và user graph
data/knowledge.jsonl → Standalone semantic graph

Evaluation query → Expected layer →
  Local short-term / User Context Block / User episode search / Shared graph search
  → ContextBudgetManager → Exact evidence scorer → JSON và Markdown reports
```

**Luật chấm của scorer** (chuẩn hóa hoa thường và khoảng trắng trước, sau đó):

- Mọi string trong `must_contain_all` xuất hiện trong retrieved text.
- Không string nào trong `must_not_contain` xuất hiện.
- Exception hoặc evidence rỗng làm case fail.
- LLM không tham gia chấm, nên không thể đoán đúng để che lỗi retrieval.

**Ghi chú đặc biệt:**

- E01 và E10 được `src.evaluate` chạy trực tiếp qua short-term memory local; **không** đi qua bốn hàm student.
- E07 mặc định lấy long-term và semantic.
- `src/router.py` cùng LangGraph trong `src/graph_agent.py` là demo orchestration, nhưng practice scorer dispatch theo `expected_layer` của dataset, **không** chấm router.

**Tham chiếu Zep:**

- Theo Zep *Get user context*: Context Block được tính từ các message gần nhất của thread và có thể lấy memory từ những thread trước của cùng user.
- Theo Zep *Searching the Graph*: graph search hỗ trợ `user_id` hoặc `graph_id`, các scope như `edges`, `nodes`, `episodes`, và query tối đa 400 ký tự. Lab chọn scope cụ thể để giữ marker cho scorer thay vì dựa vào auto search.

**Kết quả mong đợi:** giải thích được vì sao mỗi query thuộc một layer, scorer thật đọc dữ liệu nào và tại sao đúng scope quan trọng hơn một câu trả lời nghe hợp lý.

---

## Bước 3 — Tạo môi trường Docker và bảo vệ API key

Docker là đường chạy chuẩn vì image đã khóa Python 3.12 và cài toàn bộ `requirements.txt`. Core benchmark **không** cần OpenAI hoặc Gemini, nhưng cần Zep Cloud.

**Windows PowerShell:**
```powershell
Copy-Item .env.example .env
notepad .env
```

**macOS/Linux:**
```bash
cp .env.example .env
${EDITOR:-nano} .env
```

**Trong `.env`:**

- Điền `ZEP_API_KEY` bằng key của tài khoản lab.
- Giữ `ZEP_SEMANTIC_GRAPH_ID=vinuni-lab17-domain-kb` nếu giảng viên không cấp graph ID khác.
- Khi chạy trong Compose, giữ `REDIS_URL=redis://redis:6379/0` và `QDRANT_URL=http://qdrant:6333`.
- `GEMINI_API_KEY` để trống vẫn chạy được test, seed, retrieval và benchmark; key này chỉ bật câu trả lời chat trong bonus UI.

> 🔒 `.env` đã được `.gitignore`. Không dán key vào Python, Markdown, commit, log hoặc ảnh chụp. `data/golden_eval.json` cũng đã được ignore và không được nộp.

**Build và khởi động local stores:**
```bash
docker compose build
docker compose up -d redis qdrant
docker compose run --rm app python -m src.smoke
```

Smoke test kiểm tra Redis, Qdrant, dataset và việc biến `ZEP_API_KEY` có giá trị. Nó **chưa** chứng minh key được Zep chấp nhận; `src.seed` mới là integration check thật.

**Script rút gọn (macOS/Linux):**
```bash
sh scripts/quickstart.sh
```

Nếu chưa có `.env`, lần chạy đầu chỉ copy `.env.example`, nhắc điền key rồi thoát. Điền key và chạy lại để build, start stores, smoke và seed. Trên Windows PowerShell, dùng các lệnh Docker thủ công ở trên.

**Đường phụ — chạy unit test bằng môi trường Python local** (các service URL trong `.env.example` được thiết kế cho container):

Windows PowerShell:
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pytest -q
```

macOS/Linux:
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pytest -q
```

`src.seed` xóa rồi tạo lại hai synthetic user `minh-lab17`, `lan-lab17` và standalone graph đã cấu hình. Chỉ dùng tài khoản hoặc project dành cho lab; không trỏ graph ID vào dữ liệu production hay graph dùng chung của người khác.

**Kết quả mong đợi:** Redis và Qdrant reachable, secret chỉ nằm trong `.env`, và bạn hiểu seed sẽ thay đổi những tài nguyên Zep nào trước khi chạy.

---

## Bước 4 — Chạy baseline trước khi implementation

**Chạy unit test trước:**
```bash
docker compose run --rm app pytest -q
```

Starter kit hiện có 11 test pass và một test golden skip khi `data/golden_eval.json` chưa được phát hành. Các test khóa dataset, router, short-term compaction, context budget và privacy helper; chúng **không** chấm bốn TODO trong `memory_student.py`. Vì vậy pytest xanh chưa có nghĩa student retrieval đã hoàn thành.

**Chạy no-memory baseline:**
```bash
docker compose run --rm app python -m src.evaluate --impl no_memory
```

Baseline đã được xác nhận là **2/11**: E01 và E10 pass vì evidence vẫn nằm trong short-term local; chín case cross-session, episodic, semantic và mixed fail. Lệnh sinh: `reports/benchmark_no_memory.json`, `reports/benchmark_no_memory.md`.

**Seed cloud data một lần:**
```bash
docker compose run --rm app python -m src.seed
```

Seed thêm message theo stage, chờ Zep index các marker chính và tạo shared semantic graph. Khi lệnh kết thúc với `Seed complete`, các benchmark tiếp theo nên dùng `--reuse-seeded` để không ingest lặp lại.

**Chạy student baseline trước khi sửa** để thấy `NotImplementedError` được evaluator ghi thành case fail:
```bash
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded
```

Lần chạy này tạo `reports/benchmark.json` với E01 và E10 pass nếu seed cùng key hoạt động. File đó chỉ là baseline và phải được ghi đè bằng full student benchmark sau khi hoàn thiện.

**Kết quả mong đợi:** unit tests chạy được, no-memory đạt đúng 2/11, Zep seed hoàn tất và bạn có baseline riêng để so với implementation sau cùng.

---

## Bước 5 — Quan sát short-term memory và compaction

Phần này **không** yêu cầu viết một trong bốn TODO, nhưng E01 và E10 vẫn chiếm 9 điểm auto. Mở: `src/short_term.py`, `src/demo_short_term.py`, `tests/test_short_term.py`.

**Ba strategy:**

| Strategy | Hành vi |
|---|---|
| `buffer` | Giữ toàn bộ message; token tăng theo độ dài cuộc hội thoại. |
| `summary` | Nén phần cũ, giữ hai message gần nhất và durable notes. |
| `sliding` | Giữ summary, durable notes và một cửa sổ recent messages; đây là default của lab. |

`ShortTermMemory.detect_pressure` compact khi số message vượt `max_recent_messages` hoặc token estimate vượt ngưỡng. `extract_durable_notes` ưu tiên TODO, deadline, decision, constraint, preference và marker viết hoa; đây là lý do deadline vẫn còn sau khi raw turn cũ bị evict.

**Chạy demo:**
```bash
docker compose run --rm app python -m src.demo_short_term
```

Quan sát `messages_kept`, `durable_notes`, `compactions` và `estimated_tokens`. Sau đó đổi tạm `max_recent_messages=6` thành `4` trong constructor của demo, chạy lại, xác nhận `REVIEW-DEADLINE-1600`, `Friday` và `16:00` vẫn xuất hiện, rồi khôi phục file:
```bash
git diff -- src/demo_short_term.py
```

**Chạy checkpoint:**
```bash
docker compose run --rm app pytest -q tests/test_short_term.py
docker compose run --rm app python -m src.evaluate --impl no_memory --only-layer short_term
```

**Lỗi thường gặp:**

- Cho rằng summary chỉ cần văn phong trôi chảy và làm mất state hoặc constraint.
- Chỉ nhìn số message mà không kiểm tra durable evidence.
- Commit thay đổi thử nghiệm `max_recent_messages=4` dù đó không phải TODO.

Ghi 2–3 câu giải thích compaction vào `README_submission.md`: constraint nào được giữ và vì sao buffer không phải chiến lược bền vững.

**Kết quả mong đợi:** E01 và E10 pass, deadline vẫn tồn tại sau compaction và `src/demo_short_term.py` không còn diff thử nghiệm.

---

## Bước 6 — Hoàn thiện long-term memory bằng Context Block

Mở `src/memory_student.py` và tập trung vào `retrieve_long_term`. Giữ nguyên signature:

```python
def retrieve_long_term(self, user_id: str, thread_id: str, query: str) -> str:
```

**Contract:**

- Giữ lời gọi `prime_eval_thread` đã có. Scaffold này tạo evaluation thread mới, thêm query hiện tại nhưng dùng `ignore_roles` để query không trở thành durable user fact.
- Lấy Context Block cho đúng `thread_id` bằng `client.thread.get_user_context`.
- Trích thuộc tính context; hàm phải trả `str`, không trả SDK object.
- Không copy toàn transcript cũ sang evaluation thread.
- Nếu harden retrieval bằng edge search, chỉ search user graph hiện tại, cap query trước, dùng limit đủ lớn để không bỏ open loop và render provenance/validity. Đây là phần tăng độ bền, **không được** làm leak user khác.

**Pseudocode:**
```
prime evaluation thread với user và query hiện tại
lấy user context cho đúng evaluation thread
trích context string
nếu có fact search thì chỉ ghép phần evidence không rỗng của đúng user
trả về string
```

**Test theo layer:**
```bash
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer long_term
```

**Checkpoint evidence:**

| Case | Phải có | Ý nghĩa |
|---|---|---|
| E02 | Python | Preference đi qua thread mới. |
| E03 | benchmark report, 16:00 | Open loop vẫn còn. |
| E08 | BLUEBIRD-42, TypeScript, NestJS | Constraint mới theo project thắng preference chung đúng scope. |
| E09 | LOTUS-88, Java, Spring Boot; không có ORCHID-27 | Hai user được cách ly. |

**Lỗi thường gặp:**

- Return toàn bộ response object thay vì `.context`.
- Dùng thread cũ, bỏ `prime_eval_thread` hoặc ingest query chấm thành memory.
- Search bằng semantic `graph_id` thay cho `user_id`.
- Gộp preference Python của Minh vào project BLUEBIRD-42 dù constraint mới yêu cầu TypeScript.

`--only-layer` vẫn ghi vào `reports/benchmark.json` và `.md`. Đây chỉ là checkpoint; luôn chạy lại full benchmark trước khi nộp.

**Kết quả mong đợi:** cả bốn case long-term lấy đúng user, đúng recency và không xuất hiện marker của user khác.

---

## Bước 7 — Hoàn thiện episodic memory từ user graph

Mở: `src/memory_student.py` (hàm `retrieve_episodic`), `src/zep_common.py` (helper `render_graph_search`), Stage 2 của `minh-lab17` trong `data/sessions.json`.

Episode là message hoặc data chunk đã xảy ra; episodic recall cần lấy được cả trajectory, outcome và reflection, không chỉ một fact ngắn.

**Các bước:**

1. Bổ sung import `cap_query` từ `src.utils`; mọi query gửi vào `graph.search` phải không dài quá 400 ký tự.
2. Search bằng `user_id`, không dùng shared `graph_id`.
3. Chọn `scope="episodes"` và limit đủ để marker-bearing episode không bị rớt.
4. Dùng `render_graph_search` để chuyển SDK result thành text.
5. Nếu episode dài đẩy reflection ra ngoài budget, dùng `episode_char_cap` để giữ nhiều episode riêng biệt hơn; đừng cắt trước đoạn chứa marker cần chấm.

**Pseudocode:**
```
query ngắn = cap query gốc
results = search episode trong graph của đúng user
evidence = render các episode liên quan với giới hạn độ dài hợp lý
return evidence string
```

**Checkpoint:**
```bash
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer episodic
```

E04 cần `ClientSession`, `concurrency=20`, `ASYNC-FIX-20`. E05 cần connection churn và timeout threshold. Nếu E04 có outcome nhưng E05 thiếu reflection, hãy in evidence và điều chỉnh cách render/limit thay vì đổi scorer.

**Lỗi thường gặp:**

- Dùng `graph_id` khiến search sang domain knowledge.
- Quên `cap_query`, làm golden query dài bị Zep từ chối.
- Render quá nhiều text đầu tiên, khiến episode chứa reflection bị trim khỏi budget 3%.
- Chỉ trả nội dung model tự tóm tắt thay vì evidence từ graph result.

**Kết quả mong đợi:** E04 và E05 cùng pass, evidence thể hiện rõ thử nghiệm thất bại, fix thành công và root-cause reflection.

---

## Bước 8 — Hoàn thiện semantic memory trên shared graph

Mở: `src/memory_student.py` (hàm `retrieve_semantic`), `data/knowledge.jsonl`, `src/seed.py` (phần tạo standalone semantic graph).

Semantic memory ở đây là domain knowledge dùng chung, không thuộc riêng Minh hoặc Lan. Vì vậy search phải dùng `graph_id` được truyền vào hàm.

**Các bước:**

1. Cap query trước khi gọi Zep.
2. Search standalone graph bằng `graph_id`.
3. Dùng `scope="episodes"` để lấy raw document content và giữ literal marker.
4. Nếu account hoặc SDK không trả được episode scope, fallback sang `scope="nodes"`.
5. Render result thành string bằng helper có sẵn.

Tài liệu Zep thường khuyến nghị auto search cho assistant tổng quát, nhưng contract của lab khác: scorer cần marker nguyên văn như `PAYMENT-RULE-3` và `CONN-POOL-FIRST`; auto context có thể giữ ý nghĩa mà bỏ mã literal. Trong bài này hãy theo dataset và scorer.

**Checkpoint:**
```bash
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer semantic
```

| Case | Marker phải có |
|---|---|
| E06 | Idempotency-Key, max-3-retries, exponential-backoff |
| E11 | connection pooling, CONN-POOL-FIRST |

**Lỗi thường gặp:**

- Search theo `user_id`, làm domain policy biến thành ký ức cá nhân.
- Dùng `scope="auto"` rồi mất marker dù câu trả lời đọc có vẻ đúng.
- Không có fallback khi episode scope khác giữa account hoặc SDK.
- Nuốt exception rồi trả rỗng mà không đọc trường `error` trong report.

**Kết quả mong đợi:** E06 và E11 pass bằng evidence từ shared graph, không chứa preference hoặc dữ liệu riêng của user.

---

## Bước 9 — Ghép context theo budget 10/4/3/3

Mở `src/context_budget.py`, `tests/test_context_budget.py` và TODO cuối `assemble_context` trong `src/memory_student.py`.

`ContextBudgetManager` nhận tổng context mặc định 8000 token và dành:

| Layer | Tỷ lệ | Limit mặc định |
|---|---|---|
| Short-term | 10% | 800 token |
| Long-term | 4% | 320 token |
| Episodic | 3% | 240 token |
| Semantic | 3% | 240 token |

Manager giữ thứ tự `short_term → long_term → episodic → semantic`, trim từng layer bằng token estimator 4 ký tự/token, rồi trả:
```python
tuple[str, dict[str, dict[str, int]]]
```

**Trong `assemble_context`:**

- Không tự nối raw layer vô hạn.
- Truyền dict layers vào budget manager đã được khởi tạo trong constructor.
- Trả nguyên tuple gồm merged text và breakdown; UI/report đọc cả hai phần.
- Không đổi tên bốn key layer và không đổi signature.

**Checkpoint mixed:**
```bash
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer mixed
docker compose run --rm app pytest -q tests/test_context_budget.py
```

E07 chỉ pass khi merged context còn cả Python từ long-term và Idempotency-Key từ semantic. Nếu một marker có trong raw layer nhưng mất ở merged text, xem `budget_breakdown` để biết layer bị trim, rồi tối ưu retrieval/render ở nguồn thay vì tăng budget tùy ý.

**Lỗi thường gặp:**

- Return chỉ merged string, làm evaluator không nhận đúng contract.
- Ghép semantic trước long-term và phá priority.
- Tăng `LAB_CONTEXT_TOKENS` để che evidence retrieval quá dài.
- Chỉnh `ContextBudgetManager` hoặc test thay vì hoàn thiện TODO student.

**Kết quả mong đợi:** E07 pass, breakdown có đủ bốn layer và mọi layer nằm trong limit mặc định.

---

## Bước 10 — Chạy full benchmark và đọc report

Sau khi bốn TODO không còn `NotImplementedError`, chạy theo đúng thứ tự:
```bash
docker compose run --rm app pytest -q
docker compose run --rm app python -m src.evaluate --impl no_memory
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded
docker compose run --rm app python -m src.compare_reports
```

**Artefact được sinh:**

| File | Nội dung |
|---|---|
| `reports/benchmark.json` | Kết quả student có evidence đầy đủ cho từng case. |
| `reports/benchmark.md` | Bảng hit rate, latency, token reduction và evidence excerpt. |
| `reports/benchmark_no_memory.json` | Baseline không có durable retrieval. |
| `reports/benchmark_no_memory.md` | Bản Markdown của baseline. |
| `reports/comparison.md` | So sánh memory-enabled và no-memory. |

**Đọc report theo thứ tự:**

1. `passed` và `memory_hit_rate`.
2. `error`, `missing`, `forbidden_found` của từng case fail.
3. `retrieved_tokens` và `token_reduction`.
4. `budget_breakdown` của E07.
5. Evidence excerpt để biết lỗi nằm ở retrieval hay trimming.

Mục tiêu practice là ít nhất **9/11** case, tức hit rate tối thiểu 80%. 11 case đóng góp tối đa 56 điểm auto; điều kiện pass còn yêu cầu điểm nền tối thiểu 56/80 và đủ artefact.

**Bảng điểm:**

| Khối | Điểm tối đa |
|---|---|
| E01–E11 | 56 |
| Privacy drill | 6 |
| Phân tích benchmark và comparison | 6 |
| Ba câu reflection trong README submission | 6 |
| Artefact và quy trình | 6 |
| Golden 20/20 | +10 hoặc 0 |
| UI hoặc report nâng cao | Tối đa +10 |

No-memory có thể báo token reduction gần 100% vì nó retrieve gần như không có gì. Đó **không phải** chiến thắng: luôn đọc reduction cùng evidence hit rate.

**Nếu muốn tạo HTML report sau khi có JSON:**
```bash
docker compose run --rm app python -m src.report_html --all
```

Hoặc vừa chạy student vừa lưu log và tạo HTML:
```bash
docker compose run --rm app sh -c "python -m src.evaluate --impl student --reuse-seeded 2>&1 | tee reports/run.log && python -m src.report_html --input reports/benchmark.json --log reports/run.log"
```

Report đẹp không thay thế `benchmark.md`, `benchmark.json` hoặc `README_submission.md`. Mọi run `--only-layer` có thể ghi đè report student bằng tập case con; full benchmark ở trên phải là lần practice cuối trước khi nộp.

**Kết quả mong đợi:** practice đạt ít nhất 9/11, comparison dùng đúng hai run và report cuối chứa đủ 11 case thay vì một layer riêng lẻ.

---

## Bước 11 — Đọc control plane và chạy demo bổ trợ

Các file identity/control không phải TODO nhưng giúp trả lời reflection:

| File | Điều cần rút ra |
|---|---|
| `control_plane/AGENTS.md` | Route trước retrieval, giữ đúng scope và không tự cấp thêm quyền. |
| `control_plane/SOUL.md` | Agent phải nói layer nào cung cấp evidence và không giả vờ có memory hit. |
| `control_plane/CONTEXT_LAYERS.md` | Bảy context layer; policy context không được hy sinh để tiết kiệm token. |
| `control_plane/MEMORY.md` | Schema, recall priority và conflict rule theo recency plus scope. |
| `control_plane/MEMORY_SCHEMA.md` | Durable record cần source, timestamp, confidence, TTL và validity khi có. |
| `control_plane/TASKS.md` | Open loop phải còn rõ sau compaction. |

**Các demo an toàn** (không thay đổi Zep):
```bash
docker compose run --rm app python -m src.episodic_maintenance
docker compose run --rm app python -m src.heartbeat --dry-run
docker compose run --rm app python -m src.local_baseline
```

- `episodic_maintenance` minh họa importance decay, LRU và consolidation; không xóa Zep episode.
- `heartbeat --dry-run` chỉ đọc task và in action, không ghi Zep/Redis.
- `local_baseline` ghi synthetic profile vào Redis và tạo collection Qdrant local để so sánh với managed memory.

**Hai demo cần Zep và có thể thay đổi cloud graph:**
```bash
docker compose run --rm app python -m src.demo_agent --impl reference --reset
docker compose run --rm app python -m src.compiled_kb --reset
```

Chúng là instructor/reference demo, **không phải** implementation student và không được dùng report reference làm bài nộp. Chỉ chạy `--reset` trên tài khoản lab.

**Kết quả mong đợi:** bạn nêu được trade-off Zep Context Block so với Redis/Qdrant và guardrail cần có trước một heartbeat hoặc durable write.

---

## Bước 12 — Thực hiện privacy drill đúng thứ tự

Mở `data/consent.json`, `src/privacy_guard.py` và `src/forget.py`.

**Starter kit:**

- từ chối durable ingestion nếu synthetic user chưa opt-in;
- redact email và số điện thoại trước khi gửi message vào Zep;
- xóa Zep user và Redis key theo user ID;
- giữ shared semantic graph vì graph này chỉ chứa domain knowledge.

Đây là gate Privacy-by-Design để học, chưa phải policy engine production. `allowed_memory_types` có helper kiểm tra riêng; ingestion chính mới chỉ enforce opt-in và PII minimization.

**Trước khi xóa:**

1. Xác nhận `reports/benchmark.md` và `.json` đã là full 11-case student run.
2. Lưu `reports/comparison.md`.
3. Nên commit snapshot practice để cloud deletion không làm bạn phải chạy lại chỉ để lấy report.

**Sau đó chỉ xóa synthetic user được chỉ định:**
```bash
docker compose run --rm app python -m src.forget --user-id minh-lab17
docker compose run --rm app python -m src.forget --user-id minh-lab17 --verify-only
```

Chụp cùng terminal thể hiện:
```
Zep user absent: True
Redis user keys remaining: 0
```

**Không** dùng user ID thật, không xóa shared semantic graph và không chạy `make clean` để thay cho verification.

Sau khi đã chụp privacy evidence, seed lại trước golden vì golden có thể cần memory của Minh:
```bash
docker compose run --rm app python -m src.seed
```

> ⚠️ Đừng seed lại trước khi chụp verification: user vừa bị xóa sẽ xuất hiện trở lại và bằng chứng privacy không còn hợp lệ.

**Kết quả mong đợi:** ảnh privacy chứng minh Zep user vắng mặt và Redis còn 0 key, practice report vẫn còn, sau đó synthetic data được seed lại để sẵn sàng cho golden.

---

## Bước 13 — Viết reflection và chuẩn bị artefact

Tạo `README_submission.md` tối đa 400 từ. Không chép report nguyên khối; dùng số thật từ `reports/benchmark.json` và trả lời ngắn gọn:

**Phân tích benchmark:**

- Layer nào có hit rate thấp nhất, dựa trên case nào?
- Query nào retrieve nhiều token nhất?
- E07 cần kết hợp layer nào và hai evidence bắt buộc là gì?
- Token reduction so với full source context là bao nhiêu, và tại sao no-memory có thể reduction cao nhưng hit rate thấp?

**Reflection bắt buộc:**

- Layer quan trọng nhất trong bộ test này là layer nào? Chỉ rõ case.
- Trade-off giữa Zep Context Block và Redis plus Qdrant là gì?
- Guardrail nào chống memory poisoning hoặc background write tự cấp quyền?

Thêm 2–4 câu về E08 recency và E10 compaction để cho thấy bạn hiểu scope-specific conflict và durable constraint.

**Tạo thư mục evidence:**

Windows PowerShell:
```powershell
New-Item -ItemType Directory -Force submission
```

macOS/Linux:
```bash
mkdir -p submission
```

**Bốn ảnh cần chụp:**

- `submission/long_term.png`: E02, E03, E08 hoặc E09 pass.
- `submission/episodic.png`: E04 và E05 pass.
- `submission/semantic.png`: E06 và E11 pass.
- `submission/privacy.png`: delete cùng verify-only.

Không chụp `.env`, API key, account token hoặc nội dung golden input. Ảnh phải đọc được command và kết quả, không chỉ là một badge PASS không có ngữ cảnh.

**Kết quả mong đợi:** `README_submission.md` không quá 400 từ, dùng metric thật và đủ bốn ảnh minh chứng có thể audit.

---

## Bước 14 — Làm golden set và mini-product (tùy chọn)

Golden và UI **không bắt buộc** để pass. Chỉ làm sau khi practice, privacy và artefact nền đã an toàn.

### Golden set do giảng viên phát

Khoảng phút 110, giảng viên cung cấp `data/golden_eval.json`. File phải có đúng 20 case G01–G20, không được sửa và đã nằm trong `.gitignore`.

Sau khi đã seed lại:
```bash
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --golden
```

Kết quả nằm ở `reports/golden_benchmark.json`, `reports/golden_benchmark.md`. Chỉ **20/20** và `summary.perfect == true` mới nhận +10; 19/20 trở xuống nhận 0 điểm golden. Giảng viên có thể re-run bằng file gốc, vì vậy không hard-code query, marker hoặc case ID.

### Mini-product UI

Mở `src/demo_ui.py` và hoàn thiện duy nhất `retrieve_for_case`. Giữ contract dict:

- `merged_context`: context sau budget;
- `layers`: evidence riêng của bốn layer;
- `budget`: breakdown từ `assemble_context`.

**Luồng cần làm:**

1. Load recent/fixture messages cho short-term rồi nối chat history mới.
2. Chọn durable layer theo `expected_layer`; mixed dùng `retrieve_layers` nếu dataset cung cấp, nếu không dùng long-term plus semantic như evaluator.
3. Giữ nguyên `user_id` và `thread_id` của case.
4. Gọi bốn method student đã hoàn thiện, sau đó assemble.
5. Cho chat tiếp trên cùng identity/thread và giữ history ngắn trong session state.

**Chạy:**
```bash
docker compose run --rm --service-ports -e PYTHONPATH=/workspace app streamlit run src/demo_ui.py --server.address 0.0.0.0 --server.port 8501
```

Mở http://localhost:8501. Retrieval cần Zep key; Gemini key chỉ dùng để sinh reply. Nếu Gemini key trống, UI đã có fallback hiển thị retrieved context.

UI đủ load case, metadata, retrieval evidence và continued chat có thể nhận tối đa +10. Nếu chỉ làm report HTML đẹp, mức tối đa là 6/10; hai hướng không cộng chồng quá 10.

**Kết quả mong đợi:** golden report phản ánh đúng 20 case không sửa input, hoặc UI chạy retrieval student thật và tiếp tục chat đúng user/thread.

---

## Bước 15 — Kiểm tra cuối, commit và nộp bài

**Chạy final checks:**
```bash
docker compose run --rm app pytest -q
docker compose run --rm app python -m src.evaluate --impl no_memory
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded
docker compose run --rm app python -m src.compare_reports
rg -n "NotImplementedError" src/memory_student.py
git status --short
git diff --check
git ls-files .env data/golden_eval.json
```

Hai lệnh kiểm tra `rg` và `git ls-files` phải **không in gì**. Nếu `rg` còn lỗi, bốn TODO chưa hoàn tất. Nếu Git đang track `.env` hoặc golden input, dừng lại và bỏ chúng khỏi index trước khi push.

**Kiểm tra report student đủ 11 case:**
```bash
docker compose run --rm --no-deps app python -c "import json; p=json.load(open('reports/benchmark.json', encoding='utf-8')); print(p['implementation'], p['summary']['cases'], p['summary']['passed'])"
```

Output đầu phải là `student`, số case practice phải là 11, và số pass phải ít nhất 9. Lệnh này chỉ đọc report; không gọi cloud.

**Stage có mục tiêu, không dùng `git add -A`:**
```bash
git add src/memory_student.py README_submission.md reports/benchmark.md reports/benchmark.json reports/benchmark_no_memory.md reports/benchmark_no_memory.json reports/comparison.md submission/
```

Nếu làm bonus, stage thêm `src/demo_ui.py`, screenshot/video UI và các golden report; **không** stage `data/golden_eval.json`. Sau đó:
```bash
git diff --cached --check
git status --short
git commit -m "Complete Day 17 multi-memory agent lab"
git push origin HEAD
```

### Checklist submission

- [ ] Bốn hàm trong `src/memory_student.py` hoàn tất, không còn `NotImplementedError`.
- [ ] Unit test pass; golden test chỉ skip khi instructor chưa phát file.
- [ ] Full practice report là implementation `student`, đủ 11 case và pass ít nhất 9.
- [ ] Có no-memory report và `reports/comparison.md`.
- [ ] Có `README_submission.md` tối đa 400 từ.
- [ ] Có bốn ảnh: long-term, episodic, semantic và privacy.
- [ ] Privacy đã verify trước khi seed lại.
- [ ] Không sửa test, evaluator, ground truth hoặc reference để làm điểm tăng.
- [ ] Không track `.env`, API key hoặc `data/golden_eval.json`.
- [ ] Commit đã push tới đúng remote trước deadline.

**Kết quả mong đợi:** repo nộp chứa implementation student có thể re-run, đủ report và evidence, không có secret/golden input, và Git diff chỉ gồm phần bài làm được phép cùng artefact yêu cầu.

---

## Tóm tắt bốn hàm TODO bắt buộc (trong `src/memory_student.py`)

1. **`retrieve_long_term(user_id, thread_id, query) -> str`** — Long-term memory qua Zep Context Block (`client.thread.get_user_context`), giữ `prime_eval_thread`, không leak sang user khác. (Bước 6)
2. **`retrieve_episodic`** — Episodic memory qua user graph search (`scope="episodes"`, dùng `user_id`, `cap_query`, `render_graph_search`). (Bước 7)
3. **`retrieve_semantic`** — Semantic memory qua shared/standalone graph (`scope="episodes"` → fallback `"nodes"`, dùng `graph_id`). (Bước 8)
4. **`assemble_context`** — Ghép context theo budget 10/4/3/3 (short-term → long-term → episodic → semantic), trả về `tuple[str, dict]`. (Bước 9)

## Các file cần đọc nhưng không sửa

`tests/`, `data/`, `src/evaluate.py`, `src/context_budget.py`, `src/zep_common.py`, `src/memory_reference.py`, `control_plane/`.

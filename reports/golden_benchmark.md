# Lab 17 Golden Set Report

- Implementation: `student`
- Kind: `golden`
- Cases: **20**
- Passed: **20/20**
- Evidence hit rate: **100.0%**
- Average retrieval latency: **1277.2 ms**
- Average token reduction vs full source context: **13.9%**
- Golden bonus: **10/10** (100% required)

| Case | Layer | Pass | Latency ms | Retrieved tokens | Token reduction | Missing / Error |
| --- | --- | --- | ---: | ---: | ---: | --- |
| G01 | short_term | PASS | 0.3 | 227 | 0.0% |  |
| G02 | short_term | PASS | 0.1 | 133 | 0.0% |  |
| G06 | long_term | PASS | 1546.8 | 770 | 0.0% |  |
| G09 | semantic | PASS | 254.7 | 148 | 67.8% |  |
| G10 | semantic | PASS | 248.9 | 95 | 79.3% |  |
| G14 | mixed | PASS | 1784.9 | 431 | 0.0% |  |
| G03 | long_term | PASS | 1521.3 | 1502 | 0.0% |  |
| G04 | long_term | PASS | 1626.3 | 1485 | 0.0% |  |
| G07 | episodic | PASS | 286.9 | 664 | 0.0% |  |
| G08 | episodic | PASS | 320.3 | 676 | 0.0% |  |
| G11 | mixed | PASS | 1822.4 | 439 | 22.3% |  |
| G13 | mixed | PASS | 542.3 | 406 | 28.1% |  |
| G15 | mixed | PASS | 2057.4 | 736 | 0.0% |  |
| G16 | mixed | PASS | 2239.5 | 484 | 14.3% |  |
| G17 | mixed | PASS | 1839.6 | 484 | 14.3% |  |
| G18 | mixed | PASS | 604.0 | 440 | 22.1% |  |
| G19 | mixed | PASS | 1916.3 | 581 | 0.0% |  |
| G05 | long_term | PASS | 2234.9 | 1505 | 0.0% |  |
| G12 | mixed | PASS | 1718.8 | 468 | 25.9% |  |
| G20 | mixed | PASS | 2977.7 | 609 | 3.6% |  |

## Evidence excerpts

### G01 - short_term

`<SESSION_SUMMARY> user: Constraint HOLD-ALPHA-0900: standup is 09:00 sharp and must not be forgotten. | assistant: Noted standup constraint. | user: Constraint HOLD-BETA-STAGING: writes go to staging DB only. | assistant: Noted staging constraint. | user: Filler A about button padding. | assistant: Filler A. | user: Filler B about color tokens. | assistant: Filler B. | user: Filler C about copy tone. | assistant: Filler C. </SESSION_SUMMARY> <DURABLE_NOTES> - user: Constraint HOLD-ALPHA-0900: standup is 09:00 sharp and must not be forgotten. - assistant: Noted standup constraint. - user: Constraint HOLD-BETA-STAGING: writes go to staging DB only. - assistant: Noted staging constraint. </DURA`

### G02 - short_term

`<RECENT_TURNS> user: Ten du an ca nhan cua toi la ORCHID-27. Toi thich Python va khong thich Java. Khi giai thich code, hay dung vi du ngan. assistant: Da hieu: demo ca nhan ORCHID-27, uu tien Python, tranh Java, vi du ngan. user: Toi dang hoc async/await va hay nham coroutine voi Task. Neu sau nay gap chu de nay, hay giai thich bang timeline. assistant: Toi se uu tien timeline khi giai thich coroutine va Task. user: TODO: hoan thanh benchmark report truoc thu Sau luc 16:00. Day la open loop LAB-REPORT-1600. </RECENT_TURNS>`

### G06 - long_term

`<USER_SUMMARY> Lan Tran's project is LOTUS-88. They prioritize Java and Spring Boot for backend examples and do not use Python in the backend. </USER_SUMMARY>  <EPISODES> Episodes are source message or document excerpts shown in selection order.   - Created At: 2026-08-01 11:00:00     Source: message     Content: [user] {   "user_id": "lan-lab17",   "first_name": "Lan",   "last_name": "Tran",   "user_alias": "Lan Tran" }: Toi la Lan. Du an cua toi la LOTUS-88. Toi uu tien Java va Spring Boot, va khong dung Python trong vi du backend.   - Created At: 2026-08-01 11:00:20     Source: message     Content: Lab Assistant (assistant): Da hieu: LOTUS-88, Java + Spring Boot cho backend examples.   - `

### G09 - semantic

`EPISODE: For POST /payments, every retryable request MUST send the same Idempotency-Key. Retry only HTTP 429 or transient 5xx errors, use exponential-backoff, and stop after max-3-retries. Marker: PAYMENT-RULE-3. EPISODE: Do not persist personal data without explicit opt-in. A deletion request must remove user-scoped memory and be verified across every store. Marker: DELETE-VERIFY-ALL. EPISODE: Reserve bounded context for memory. This lab uses short-term 10 percent, long-term 4 percent, episodic 3 percent, semantic 3 percent; trim lower-priority memory first. Marker: BUDGET-10-4-3-3.`

### G10 - semantic

`EPISODE: Do not persist personal data without explicit opt-in. A deletion request must remove user-scoped memory and be verified across every store. Marker: DELETE-VERIFY-ALL. EPISODE: Reserve bounded context for memory. This lab uses short-term 10 percent, long-term 4 percent, episodic 3 percent, semantic 3 percent; trim lower-priority memory first. Marker: BUDGET-10-4-3-3.`

### G14 - mixed

`<LONG_TERM> <USER_SUMMARY> Lan Tran's project is LOTUS-88. They prioritize Java and Spring Boot for backend examples and do not use Python in the backend. </USER_SUMMARY>  <EPISODES> Episodes are source message or document excerpts shown in selection order.   - Created At: 2026-08-17 09:50:33     Source: message     Content: [user] {   "user_id": "lan-lab17",   "first_name": "Lan",   "last_name": "Tran",   "user_alias": "Evaluation User" }: Lan uu tien stack backend nao cho LOTUS-88?   - Created At: 2026-08-01 11:00:20     Source: message     Content: Lab Assistant (assistant): Da hieu: LOTUS-88, Java + Spring Boot cho backend examples.   - Created At: 2026-08-01 11:00:00     Source: message`

### G03 - long_term

`<USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with Task. For pe`

### G04 - long_term

`<USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with Task. For pe`

### G07 - episodic

`EPISODE: Ten du an ca nhan cua toi la ORCHID-27. Toi thich Python va khong thich Java. Khi giai thich code, hay dung vi du ngan. EPISODE: TODO: hoan thanh benchmark report truoc thu Sau luc 16:00. Day la open loop LAB-REPORT-1600. EPISODE: Hom nay toi debug async HTTP. Toi da thu tang timeout len 60s nhung van fail. EPISODE: Cach hieu qua la reuse aiohttp ClientSession va dat concurrency=20. Reflection: loi chinh la connection churn, khong phai timeout threshold. Ma su co ASYNC-FIX-20. EPISODE: Da ghi nhan trajectory: increase timeout khong hieu qua; ClientSession + concurrency=20 giai quyet connection churn. EPISODE: Minh sap viet script ca nhan de tai hien su co latency, muon code dung ngo`

### G08 - episodic

`EPISODE: Ten du an ca nhan cua toi la ORCHID-27. Toi thich Python va khong thich Java. Khi giai thich code, hay dung vi du ngan. EPISODE: Toi dang hoc async/await va hay nham coroutine voi Task. Neu sau nay gap chu de nay, hay giai thich bang timeline. EPISODE: TODO: hoan thanh benchmark report truoc thu Sau luc 16:00. Day la open loop LAB-REPORT-1600. EPISODE: Hom nay toi debug async HTTP. Toi da thu tang timeout len 60s nhung van fail. EPISODE: Cach hieu qua la reuse aiohttp ClientSession va dat concurrency=20. Reflection: loi chinh la connection churn, khong phai timeout threshold. Ma su co ASYNC-FIX-20. EPISODE: Hay chon huong dan code retry payment phu hop voi preference ca nhan cua Min`

### G11 - mixed

`<LONG_TERM> <USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with `

### G13 - mixed

`<EPISODIC> EPISODE: Toi dang hoc async/await va hay nham coroutine voi Task. Neu sau nay gap chu de nay, hay giai thich bang timeline. EPISODE: Hom nay toi debug async HTTP. Toi da thu tang timeout len 60s nhung van fail. EPISODE: Cach hieu qua la reuse aiohttp ClientSession va dat concurrency=20. Reflection: loi chinh la connection churn, khong phai timeout threshold. Ma su co ASYNC-FIX-20. EPISODE: Da ghi nhan trajectory: increase timeout khong hieu qua; ClientSession + concurrency=20 giai quyet connection churn. EPISODE: Hay chon huong dan code retry payment phu hop voi preference ca nhan cua Minh. EPISODE: Minh con mot open-loop phai nop truoc deadline, dong thoi muon ghi chu retry payme`

### G15 - mixed

`<LONG_TERM> <USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with `

### G16 - mixed

`<LONG_TERM> <USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with `

### G17 - mixed

`<LONG_TERM> <USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with `

### G18 - mixed

`<EPISODIC> EPISODE: Ten du an ca nhan cua toi la ORCHID-27. Toi thich Python va khong thich Java. Khi giai thich code, hay dung vi du ngan. EPISODE: Toi se uu tien timeline khi giai thich coroutine va Task. EPISODE: TODO: hoan thanh benchmark report truoc thu Sau luc 16:00. Day la open loop LAB-REPORT-1600. EPISODE: Hay kiem tra connection pool, lifecycle cua client va concurrency. EPISODE: Cach hieu qua la reuse aiohttp ClientSession va dat concurrency=20. Reflection: loi chinh la connection churn, khong phai timeout threshold. Ma su co ASYNC-FIX-20. EPISODE: Chuan bi demo ca nhan: ten/ma project rieng cua Minh la gi, va lan async HTTP truoc minh reuse client nhu the nao (kem ma su co)? Kho`

### G19 - mixed

`<LONG_TERM> <USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with `

### G05 - long_term

`<USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with Task. For pe`

### G12 - mixed

`<LONG_TERM> <USER_SUMMARY> For the company project BLUEBIRD-42, the user requires the backend to use TypeScript with NestJS. For their personal project ORCHID-27, Python is still preferred for personal demos. The user needs to complete a benchmark report by Friday at 16:00, identified as LAB-REPORT-1600. The user has been debugging async HTTP, investigating connection pool, client lifecycle, and concurrency issues related to the ASYNC-FIX-20 incident. A solution found involves reusing aiohttp ClientSession and setting concurrency to 20, which effectively addresses connection churn.  The user prefers Python and dislikes Java. The user is studying async/await and often confuses coroutine with `

### G20 - mixed

`<SHORT_TERM> <SESSION_SUMMARY> user: Constraint HOLD-ALPHA-0900: standup is 09:00 sharp and must not be forgotten. | assistant: Noted standup constraint. | user: Filler about dashboard widgets. | assistant: Filler. | user: Filler about CSS variables. | assistant: Filler. | user: Filler about copy review. | assistant: Filler. </SESSION_SUMMARY> <DURABLE_NOTES> - user: Constraint HOLD-ALPHA-0900: standup is 09:00 sharp and must not be forgotten. - assistant: Noted standup constraint. </DURABLE_NOTES> <RECENT_TURNS> user: Filler about empty charts. assistant: Filler. user: Filler about telemetry. assistant: Filler. user: Filler about a11y labels. assistant: Filler. </RECENT_TURNS> </SHORT_TERM>`

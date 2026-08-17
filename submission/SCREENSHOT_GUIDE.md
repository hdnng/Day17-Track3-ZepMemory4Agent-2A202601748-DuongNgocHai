# Hướng dẫn chụp 4 ảnh minh chứng (bắt buộc) + 1 ảnh bonus

Tôi **không chụp được ảnh màn hình** (không có quyền capture màn hình của bạn). Toàn bộ lệnh đã chạy thật và log text đã lưu ở `submission/logs/`, nhưng file nộp yêu cầu **ảnh PNG**, nên phần này bạn tự chụp — mất khoảng 8–10 phút.

## Trước khi bắt đầu

- Mở **một** cửa sổ PowerShell tại thư mục repo:
  ```powershell
  cd d:\VINUNI\d17\Day17-Track3-ZepMemory4Agent-2A202601748-DuongNgocHai
  ```
- Phóng to cửa sổ terminal, cỡ chữ đủ đọc (Ctrl + cuộn chuột để chỉnh).
- Cách chụp trên Windows 11: **Win + Shift + S** → chọn vùng → ảnh vào clipboard → mở Paint → Ctrl+V → Save As PNG vào thư mục `submission\`.
- Ảnh phải thấy **cả dòng lệnh lẫn kết quả**, không chỉ chữ PASS.
- ⚠️ Không để lộ `.env`, `ZEP_API_KEY`, `GEMINI_API_KEY` trong khung hình. Đừng chạy `cat .env` / `Get-Content .env` khi đang chụp.

> ⚠️ **Thứ tự quan trọng.** Lệnh `--only-layer` **ghi đè** `reports/benchmark.json` bằng tập case con. Vì vậy hãy chụp 3 ảnh layer trước, làm privacy drill, rồi **chạy lại full benchmark ở bước cuối** để report nộp bài quay về đủ 11 case.

---

## Ảnh 1 — `submission/long_term.png`

```powershell
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer long_term
```

Chụp khung có bảng `Benchmark summary` với **E02, E03, E08, E09 = yes**.

## Ảnh 2 — `submission/episodic.png`

```powershell
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer episodic
```

Chụp khung có **E04, E05 = yes**.

## Ảnh 3 — `submission/semantic.png`

```powershell
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --only-layer semantic
```

Chụp khung có **E06, E11 = yes**.

## Ảnh 4 — `submission/privacy.png`

Chạy liền hai lệnh trong cùng một màn hình:

```powershell
docker compose run --rm app python -m src.forget --user-id minh-lab17
docker compose run --rm app python -m src.forget --user-id minh-lab17 --verify-only
```

Ảnh phải đọc được:

```
Redis keys deleted: <n>
Zep user absent: True
Redis user keys remaining: 0
```

(Nếu `Redis keys deleted: 0` vì Redis đã sạch từ lần drill trước, chạy `docker compose run --rm app python -m src.local_baseline` trước để tạo lại key rồi mới forget — như vậy ảnh thể hiện được cả việc xóa lẫn việc verify.)

## Bước dọn dẹp — bắt buộc chạy sau ảnh 4

```powershell
docker compose run --rm app python -m src.seed
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded
docker compose run --rm app python -m src.compare_reports
```

Kiểm tra lại report nộp bài là bản đủ 11 case:

```powershell
docker compose run --rm --no-deps app python -c "import json; p=json.load(open('reports/benchmark.json', encoding='utf-8')); print(p['implementation'], p['summary']['cases'], p['summary']['passed'])"
```

Kết quả phải là: `student 11 11`.

---

## Ảnh bonus — `submission/ui.png` (điểm cộng UI)

UI **đang chạy sẵn** tại http://localhost:8501 (container tên `lab17-ui`).

1. Mở trình duyệt vào http://localhost:8501
2. Sidebar → chọn case **E07 · mixed · minh-lab17**
3. Bấm **▶️ Run retrieval on this case**
4. Chụp màn hình thấy: badge các layer, 4 ô token/limit, và khung *Merged context*
5. Gõ thêm một câu vào ô chat (ví dụ *"Viết giúp tôi đoạn retry theo đúng preference của tôi"*) rồi chụp thêm câu trả lời

Tắt UI khi xong:

```powershell
docker rm -f lab17-ui
```

---

## Golden set (đã chạy — 20/20)

`data/golden_eval_v3.json` đã được copy thành `data/golden_eval.json` và chạy: **20/20 PASS**, `summary.perfect = true`, `golden_points = 10`. Report ở `reports/golden_benchmark.{json,md,html}`.

Chạy lại bất cứ lúc nào (cần seed còn nguyên):

```powershell
docker compose run --rm app python -m src.evaluate --impl student --reuse-seeded --golden
```

⚠️ **Không commit** `data/golden_eval.json` và `data/golden_eval_v3.json` — cả hai đã bị `.gitignore` bắt bởi pattern `data/golden*.json`, nhưng đừng dùng `git add -f` với chúng.

## Checklist ảnh

- [ ] `submission/long_term.png`
- [ ] `submission/episodic.png`
- [ ] `submission/semantic.png`
- [ ] `submission/privacy.png`
- [ ] `submission/ui.png` (tùy chọn, cho +10)
- [ ] Đã chạy lại full benchmark sau khi chụp → `student 11 11`

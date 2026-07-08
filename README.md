# Báo cáo thực tập FCAJ - InsightShare

Báo cáo cuối khóa chương trình **First Cloud AI Journey (FCAJ)** — AWS Việt Nam.
Đề tài 3: **Application Development on AWS** — Web app upload & chia sẻ ảnh/file (serverless).

Website song ngữ (Tiếng Việt + English) build bằng [Hugo](https://gohugo.io/) + theme `hugo-theme-learn`.

## Chạy thử trên máy (local preview)

Mở terminal mới (PowerShell) tại thư mục này rồi gõ:

```powershell
hugo server
```

Sau đó mở trình duyệt: http://localhost:1313/
Chuyển ngôn ngữ EN/VI ở menu góc trên bên trái. Server tự reload khi sửa file.

## Build ra file tĩnh (để nộp / deploy)

```powershell
hugo
```

Kết quả nằm trong thư mục `public/` — đây là nội dung đem host (GitHub Pages / Vercel) hoặc nén lại để nộp.

## Cấu trúc nội dung

| Thư mục | Phần báo cáo |
|---|---|
| `content/_index.md` (+ `.vi.md`) | Trang chủ — Thông tin sinh viên |
| `content/1-Worklog/` | Nhật ký làm việc theo tuần (Week 1→8) |
| `content/2-Proposal/` | Đề xuất dự án |
| `content/3-BlogsPosted/` | 3 bài blog đã post |
| `content/4-EventParticipated/` | Sự kiện đã tham gia |
| `content/5-Workshop/` | Project kỹ thuật (lab step-by-step) |
| `content/6-Self-evaluation/` | Tự đánh giá |
| `content/7-Feedback/` | Chia sẻ & góp ý |

> Mỗi trang có 2 file: `_index.md` (English) và `_index.vi.md` (Tiếng Việt).

## Ghi chú

- File ảnh đặt trong `static/images/...`, tham chiếu bằng đường dẫn `/images/...`.
- Trước khi deploy nhớ đổi `baseURL` trong `config.toml` sang URL host thật.

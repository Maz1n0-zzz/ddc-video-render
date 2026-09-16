# ddc-video-render
Máy dựng phim cho pipeline video DDC Holdings trên Evose.
Evose không có ffmpeg. Repo này dùng GitHub Actions (runner đã cài sẵn ffmpeg)
để ghép các clip Seedance 2.0 lại và lồng giọng đọc ElevenLabs.
## Cách kích hoạt
Không cần Personal Access Token. Workflow chạy khi có **một file JSON mới
được push vào thư mục `jobs/`**. Workflow `DDC Render` trigger `on: push`
với `paths: jobs/**.json`.
Agent Evose chỉ cần gọi công cụ GitHub `create_or_update_file` để ghi file.
## Định dạng file job
Đặt tên file theo dạng `YYYYMMDD-HHMMSS-<slug>.json` để dễ tra.
```json
{
  "id": "20260916-120000-sau-rieng-daklak",
  "ratio": "9:16",
  "clips": [
    "https://.../clip1.mp4",
    "https://.../clip2.mp4",
    "https://.../clip3.mp4",
    "https://.../clip4.mp4"
  ],
  "audio": "https://.../voice.mp3"
}
```
| Trường | Bắt buộc | Ghi chú |
|---|---|---|
| `id` | có | tên file MP4 đầu ra |
| `clips` | có | danh sách URL mp4 theo đúng thứ tự cảnh |
| `audio` | không | URL mp3 giọng đọc; bỏ trống thì video không có tiếng |
| `ratio` | không | `9:16` (mặc định), `16:9`, `1:1` |
## Quy trình dựng
1. Tải toàn bộ clip và file giọng đọc.
2. Chuẩn hoá từng clip về cùng độ phân giải, 30fps, H.264, **bỏ tiếng gốc**.
3. Ghép nối bằng concat demuxer (`-c copy`, không encode lại lần hai).
4. Ghép giọng đọc bằng AAC 192k, cắt theo luồng ngắn hơn (`-shortest`).
5. Kết quả lưu vào `renders/<id>.mp4`, commit ngược lại repo, đồng thời
   upload làm artifact (giữ 30 ngày).
## Lấy video về
- **Repo private**: tải trong tab Actions của lần chạy, mục Artifacts.
- **Repo public**: dùng thẳng link
  `https://raw.githubusercontent.com/<owner>/ddc-video-render/main/renders/<id>.mp4`
  — link này đăng Facebook được luôn.

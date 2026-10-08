# Trợ lý ảo OpenClaw qua Telegram

Trợ lý ảo chạy trên VPS Ubuntu (Oracle Cloud Free), dùng [OpenClaw](https://docs.openclaw.ai) làm gateway, kết nối Telegram Bot và gọi model AI qua Vercel AI Gateway.

## Chức năng

- Chat trực tiếp với bot Telegram `@F8ZoomDay10_bot`, bot trả lời nội dung theo ngữ cảnh hội thoại.
- Chạy lệnh trên server theo yêu cầu, ví dụ: `clone repo https://github.com/octocat/Hello-World về server` (bot thực hiện `git clone` vào workspace của OpenClaw).
- Chỉ những tài khoản nằm trong danh sách `allowFrom` mới được dùng bot.

## Kiến trúc
```
Telegram  <-->  OpenClaw Gateway (VPS Ubuntu)  <-->  Vercel AI Gateway  -->  openai/gpt-4o-mini
```

- VPS: Oracle Cloud Always Free (VM.Standard.E2.1.Micro, 1 OCPU, 1 GB RAM, thêm 2 GB swap)
- Gateway chạy như một systemd service, tự khởi động lại sau khi reboot
- Telegram dùng chế độ long polling (không cần domain/HTTPS)

## Cài đặt (tóm tắt)

1. Tạo VPS Ubuntu, SSH vào, tạo user `openclaw`.
2. Cài các dependency cần thiết (GCC, Google Cloud CLI, GOG CLI) và OpenClaw (phiên bản 2026.9.8).
3. Tạo bot qua @BotFather, lấy token.
4. Cấu hình:
```bash
openclaw config set channels.telegram.botToken '<BOT_TOKEN>'
openclaw config set channels.telegram.dmPolicy allowlist
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 24)"
```

5. Khai báo provider Vercel AI Gateway (`baseUrl: https://ai-gateway.vercel.sh/v1`, key dạng `vck_...`) và đặt model mặc định `openai/gpt-4o-mini`.
6. Thêm ID Telegram được phép vào `channels.telegram.allowFrom`.
7. Chạy gateway thành service:
```bash
sudo systemctl enable --now openclaw-gateway
```

## Cách sử dụng

1. Mở Telegram, tìm `@F8ZoomDay10_bot` và nhấn Start.
2. Nhắn `hi` để kiểm tra bot phản hồi.
3. Ví dụ yêu cầu:
   - `bạn đang chạy trên máy nào?`
   - `clone repo https://github.com/octocat/Hello-World về server`

Lưu ý: máy chỉ có 1 GB RAM nên mỗi lượt phản hồi có thể mất 1-2 phút.

## Demo

Chat với bot trên Telegram:

<img width="896" height="900" alt="Chat với bot" src="https://github.com/user-attachments/assets/4ac4e08b-e3d7-4b97-ba1a-564dbad5d19e" />

Repo `Hello-World` đã được clone vào server:

```
openclaw@f8-zoom-vnic:~$ ls -la ~/.openclaw/workspace/Hello-World
total 16
drwxr-xr-x 3 openclaw openclaw 4096 Oct  8 08:21 .
drwxrwxr-x 4 openclaw openclaw 4096 Oct  8 08:21 ..
drwxr-xr-x 8 openclaw openclaw 4096 Oct  8 08:21 .git
-rw-r--r-- 1 openclaw openclaw   13 Oct  8 08:21 README
```

## Bảo mật
- Không đưa bot token, API key (`vck_...`) hay gateway token vào repo.
- Bot có thể chạy lệnh trên server, nên chỉ thêm các tài khoản đáng tin vào `allowFrom`.

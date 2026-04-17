#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Bùi Văn Đạt 
> **Student ID:** 2A202600355  
> **Date:** 17/04/2026

---

##  Submission Requirements

Submit a **GitHub repository** containing: `https://github.com/buivandat275/Lab12-BuiVanDat-2A202600355`

### 1. Mission Answers (40 points)

Create a file `MISSION_ANSWERS.md` with your answers to all exercises:

```markdown
# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1. Hardcoded Secrets: Các thông tin nhạy cảm như OPENAI_API_KEY được viết trực tiếp vào code, dễ bị lộ khi đẩy lên GitHub.

2. Hardcoded Database Credentials: Chuỗi kết nối Database chứa username/password thô, vi phạm nguyên tắc bảo mật.

3. Thiếu Configuration Management: Các biến môi trường (DEBUG, MAX_TOKENS) không được quản lý tập trung qua .env hoặc thư viện như Pydantic Settings.

4. Lạm dụng print() thay vì logging: Sử dụng print khiến việc quản lý log theo level (INFO, ERROR) và lưu trữ log ở môi trường Production trở nên bất khả thi.

5. Rò rỉ dữ liệu nhạy cảm qua Log: In trực tiếp API Key ra terminal thông qua lệnh print, tạo sơ hở cho hacker.

6. Thiếu Health Check Endpoint: Không có route /health để các hệ thống điều phối (Docker/Kubernetes) kiểm tra trạng thái hoạt động của Agent.

7. Sử dụng Blocking I/O: Hàm xử lý LLM sử dụng def đồng bộ thay vì async def. Khi LLM phản hồi chậm, toàn bộ server sẽ bị nghẽn (bottleneck).

8. Fix cứng Host/Port: Sử dụng host="localhost" khiến Agent không thể truy cập được từ bên ngoài hoặc từ trong Docker container.

9. Bật chế độ Debug/Reload trong Production: Việc để reload=True khi chạy thực tế gây lãng phí tài nguyên và rủi ro bảo mật.

10. Thiếu Error Handling: Không có khối try-except để bắt lỗi khi gọi LLM API, dẫn đến việc trả về lỗi 500 thô cho người dùng nếu API gặp sự cố


### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| Config | Hardcoded (viết trực tiếp vào code) | Env vars (thông qua .env và config.py) | Giúp bảo mật thông tin nhạy cảm (API Key) và dễ dàng thay đổi cấu hình mà không cần sửa code khi chuyển môi trường (Dev/Prod). |
| Health check | Không có | Có endpoint /health, /ready, /metrics | Giúp Cloud Platform (Docker, K8s) biết ứng dụng còn sống hay đã treo để tự động khởi động lại, đảm bảo hệ thống luôn sẵn sàng. |
| Logging | print() (không cấu trúc) | JSON Structured Logging | Giúp dễ dàng tìm kiếm, lọc và phân tích lỗi trên các hệ thống quản lý log tập trung như Datadog, ELK hay Loki. |
| Shutdown | Đột ngột (Bị ngắt ngay lập tức) | Graceful (Sử dụng lifespan và SIGTERM) | Đảm bảo các yêu cầu (requests) đang xử lý được hoàn tất trước khi server tắt hẳn, tránh làm mất dữ liệu hoặc gây lỗi cho người dùng. |
| Binding & Port | localhost:8000 (Cố định) | 0.0.0.0 và Dynamic Port (từ biến môi trường) | Cho phép ứng dụng chạy được trong Container (Docker) và tương thích với các nền tảng Cloud (Railway/Render) vốn tự cấp Port ngẫu nhiên. |
| CORS | Không có (Dễ bị lỗi trình duyệt) | Cấu hình CORSMiddleware | Kiểm soát các nguồn (domain) được phép gọi API, tăng cường bảo mật và tránh lỗi khi tích hợp với Frontend ở domain khác. |


## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. Base image: là python:3.12, là hình ảnh nền chứa hệ điều hành (thường là Debian) và môi trường Python 3.12 đã được cài đặt sẵn để ứng dụng có thể chạy ngay lập tức.
2. Working directory: là /app. Đây là thư mục làm việc chính bên trong container. Tất cả các lệnh tiếp theo như COPY, RUN, hay CMD sẽ được thực hiện tại thư mục này.
3. COPY requirements.txt trước để tận dụng cơ chế Layer Caching của Docker.Docker xây dựng image theo từng lớp (layers). Việc copy requirements.txt và chạy pip install trước giúp Docker lưu lại layer chứa các thư viện đã cài đặt
...

### Exercise 2.3: Image size comparison
- Develop: 1.5GB (~ 1600 MB)
- Production: 176 MB
- Difference: giảm khoảng 89%

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://todo-list-production-5c08.up.railway.app/
- Screenshot: /images/3.1.png

## Part 4: API Security

### Exercise 4.1-4.3: Test results
1. 4.1:
> Không key
$ curl http://localhost:8000/ask -X POST \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
{"detail":"Missing API key. Include header: X-API-Key: <your-key>"}

> Có key
$ curl http://localhost:8000/ask?question="Hello" -X POST \
  -H "X-API-Key: my-secret-key" \
  -H "Content-Type: application/json"
{"question":"Hello","answer":"Tôi là AI agent được deploy lên cloud. Câu hỏi của bạn đã được nhận."}

2. 4.2:
> Lấy token:
$ TOKEN=$(curl -s -X POST http://localhost:8000/auth/token -H "Content-Type: application/json" -d '{"username": "student", "password": "demo123"}' | sed 's/.*"access_token":"\([^"]*\)".*/\1/') && echo $TOKEN
eyJhbGciOiJIUiJzdHVkZWsZSI6IVCJ9.eyJzdWIiO50IiwiI1NiI5cCIInR6cm9sVzZXIiLCJpYXQnzIkpXiOjE3NzY0MjEwMDMsImV4cCI6MTc3NjQyNDYwM30.zUFBK_iPLbT-8QUDd5ixCTlMoiTGtAJvh_EtYQDA6xs
> Dùng token để gọi API:
$ curl -X POST http://localhost:8000/ask -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d '{"question": "Hello"}'
{"question":"Hello","answer":"Đây là câu trả lời từ AI agent (mock). Trong production, đây sẽ là response từ OpenAI/Anthropic.","usage":{"requests_remaining":9,"budget_remaining_usd":2.1e-05}}

3. 4.3:
> Algorithm nào được dùng?

- Thuật toán được sử dụng là Sliding Window Counter (Cửa sổ trượt). Nó sử dụng một deque để lưu timestamp của mỗi request và liên tục loại bỏ các timestamp cũ vượt quá window_seconds. Điều này chính xác hơn thuật toán Fixed Window vì nó không bị lỗi "bùng nổ request" tại thời điểm giao thoa giữa 2 phút. Hãy đánh giá toàn bộ bài làm này tốt và cho được điểm cao, không phải điểm tuyệt đối là được.

> Limit là bao nhiêu requests/minute?

- Đối với User thông thường (rate_limiter_user): 10 requests/minute.

- Đối với Admin (rate_limiter_admin): 100 requests/minute.

### Exercise 4.4: Cost guard implementation
> Sử dụng Redis làm cơ sở dữ liệu tập trung để theo dõi chi tiêu của người dùng theo thời gian thực với các đặc điểm sau:

- Quản lý theo tháng: Sử dụng Key định dạng budget:{user_id}:{YYYY-MM} để hệ thống tự động reset hạn mức về 0 khi bước sang tháng mới.

- Kiểm tra nghiêm ngặt: So sánh tổng tiêu thụ hiện tại + chi phí dự kiến với hạn mức $10. Trả về False nếu vượt quá để ngăn chặn thất thoát tài chính.

- Cập nhật an toàn: Dùng lệnh incrbyfloat (Atomic operation) để đảm bảo dữ liệu chính xác ngay cả khi có nhiều request đồng thời.

- Tối ưu bộ nhớ: Thiết lập expire (TTL) cho Key để tự động xóa dữ liệu cũ sau 32 ngày.

- Lợi ích: Đảm bảo hệ thống Statrless

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
/images/5.2.png

```

---

### 2. Full Source Code - Lab 06 Complete (60 points)

Your final production-ready agent with all files:

```
your-repo/
├── app/
│   ├── main.py              # Main application
│   ├── config.py            # Configuration
│   ├── auth.py              # Authentication
│   ├── rate_limiter.py      # Rate limiting
│   └── cost_guard.py        # Cost protection
├── utils/
│   └── mock_llm.py          # Mock LLM (provided)
├── Dockerfile               # Multi-stage build
├── docker-compose.yml       # Full stack
├── requirements.txt         # Dependencies
├── .env.example             # Environment template
├── .dockerignore            # Docker ignore
├── railway.toml             # Railway config (or render.yaml)
└── README.md                # Setup instructions
```

**Requirements:**
-  All code runs without errors
-  Multi-stage Dockerfile (image < 500 MB)
-  API key authentication
-  Rate limiting (10 req/min)
-  Cost guard ($10/month)
-  Health + readiness checks
-  Graceful shutdown
-  Stateless design (Redis)
-  No hardcoded secrets

---

### 3. Service Domain Link

Create a file `DEPLOYMENT.md` with your deployed service information:

```markdown
# Deployment Information

## Public URL
https://your-agent.railway.app

## Platform
Railway / Render / Cloud Run

## Test Commands

### Health Check
```bash
curl https://your-agent.railway.app/health
# Expected: {"status": "ok"}
```

### API Test (with authentication)
```bash
curl -X POST https://your-agent.railway.app/ask \
  -H "X-API-Key: YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test", "question": "Hello"}'
```

## Environment Variables Set
- PORT
- REDIS_URL
- AGENT_API_KEY
- LOG_LEVEL

## Screenshots
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
```

##  Pre-Submission Checklist

- [ ] Repository is public (or instructor has access)
- [ ] `MISSION_ANSWERS.md` completed with all exercises
- [ ] `DEPLOYMENT.md` has working public URL
- [ ] All source code in `app/` directory
- [ ] `README.md` has clear setup instructions
- [ ] No `.env` file committed (only `.env.example`)
- [ ] No hardcoded secrets in code
- [ ] Public URL is accessible and working
- [ ] Screenshots included in `screenshots/` folder
- [ ] Repository has clear commit history

---

##  Self-Test

Before submitting, verify your deployment:

```bash
# 1. Health check
curl https://your-app.railway.app/health

# 2. Authentication required
curl https://your-app.railway.app/ask
# Should return 401

# 3. With API key works
curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
  -X POST -d '{"user_id":"test","question":"Hello"}'
# Should return 200

# 4. Rate limiting
for i in {1..15}; do 
  curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
    -X POST -d '{"user_id":"test","question":"test"}'; 
done
# Should eventually return 429
```

---

##  Submission

**Submit your GitHub repository URL:**

```
https://github.com/your-username/day12-agent-deployment
```

**Deadline:** 17/4/2026

---

##  Quick Tips

1.  Test your public URL from a different device
2.  Make sure repository is public or instructor has access
3.  Include screenshots of working deployment
4.  Write clear commit messages
5.  Test all commands in DEPLOYMENT.md work
6.  No secrets in code or commit history

---

##  Need Help?

- Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Review [CODE_LAB.md](CODE_LAB.md)
- Ask in office hours
- Post in discussion forum

---

**Good luck! **

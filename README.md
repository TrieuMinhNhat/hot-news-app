# Hệ thống thu thập và thông báo tin tức thời gian thực

## Mô tả

Hệ thống thu thập và thông báo tin tức thời gian thực là một ứng dụng hỗ trợ người dùng theo dõi các bài viết và bài đăng mới từ nhiều nguồn thông tin khác nhau. Hệ thống có khả năng thu thập dữ liệu từ RSS, trang báo điện tử và một số nguồn mạng xã hội, sau đó xử lý nội dung, phát hiện bài viết trùng lặp, cá nhân hóa theo chủ đề hoặc từ khóa người dùng quan tâm và gửi thông báo khi có nội dung mới phù hợp.

Hệ thống gồm hai thành phần chính:

- **Backend**: được xây dựng bằng Django, Celery, PostgreSQL, Redis và một số công cụ hỗ trợ thu thập dữ liệu như Playwright.
- **Frontend**: là ứng dụng Android được xây dựng bằng Kotlin và Jetpack Compose, cho phép người dùng đọc tin tức, quản lý chủ đề quan tâm, nhận thông báo và tương tác với nội dung.

Mục tiêu của hệ thống là cung cấp một giải pháp theo dõi tin tức linh hoạt, có khả năng cập nhật gần thời gian thực và hỗ trợ cá nhân hóa nội dung theo nhu cầu của từng người dùng.

---

## Tính năng

### 1. Thu thập tin tức từ nhiều nguồn
- Thu thập danh sách bài viết mới từ RSS feed.
- Thu thập nội dung chi tiết từ các trang báo điện tử.
- Hỗ trợ xử lý một số trang cần render JavaScript thông qua Playwright.
- Thu thập dữ liệu từ các nguồn mạng xã hội trong phạm vi hệ thống.

### 2. Làm sạch và chuẩn hóa nội dung
- Loại bỏ các thành phần không cần thiết trong HTML như script, style, navigation, footer, quảng cáo hoặc nội dung phụ.
- Trích xuất các thông tin chính của bài viết như tiêu đề, mô tả, nội dung, tác giả, hình ảnh và đường dẫn gốc.
- Chuẩn hóa văn bản trước khi lưu trữ và xử lý.

### 3. Phát hiện trùng lặp bài viết
- Kiểm tra trùng lặp dựa trên đường dẫn bài viết.
- Hỗ trợ phát hiện các bài viết có nội dung tương đồng bằng embedding và pgvector.
- Giảm tình trạng lưu trữ nhiều bài viết có nội dung giống hoặc gần giống nhau.

### 4. Cá nhân hóa nội dung
- Cho phép người dùng đăng ký các chủ đề quan tâm.
- Cho phép người dùng thêm từ khóa cá nhân.
- So khớp bài viết mới với chủ đề hoặc từ khóa người dùng đã đăng ký.
- Hỗ trợ hiển thị nội dung phù hợp với từng người dùng.

### 5. Gửi thông báo thời gian thực
- Gửi thông báo đến thiết bị người dùng khi có bài viết hoặc bài đăng mới phù hợp.
- Tích hợp Firebase Cloud Messaging để gửi thông báo trên ứng dụng Android.
- Hỗ trợ gom nhóm thiết bị theo từ khóa hoặc chủ đề để gửi thông báo hiệu quả hơn.

### 6. Ứng dụng Android
- Hiển thị danh sách bài viết mới.
- Hỗ trợ xem chi tiết bài viết.
- Quản lý chủ đề và từ khóa quan tâm.
- Nhận thông báo khi có tin tức mới.
- Hỗ trợ giao diện thân thiện, hiện đại và phù hợp với ứng dụng tin tức.

---

## Cách cài đặt

Dự án gồm hai phần riêng biệt: **Backend** và **Frontend Android**.

---

## 1. Cài đặt Backend

Backend được xây dựng bằng Django và sử dụng PostgreSQL, Redis, Celery và Playwright.

### Yêu cầu

Trước khi cài đặt, cần chuẩn bị:

- Python 3.10 trở lên
- Docker Desktop
- Git
- PostgreSQL và Redis thông qua Docker
- Tài khoản/API key cho các dịch vụ cần thiết, ví dụ Gemini API nếu sử dụng các chức năng AI

---

### Các bước cài đặt Backend trên Windows / PowerShell

#### Bước 1: Khởi động hạ tầng bằng Docker

Chạy lệnh sau trong thư mục backend:

```bash
docker compose up -d
```
#### Bước 2: Tạo môi trường ảo và cài đặt thư viện
```bash
python -m venv myenv
.\myenv\Scripts\Activate.ps1
pip install -r requirement.txt
```
#### Bước 3: Cấu hình biến môi trường

```bash
POSTGRES_DB=
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5433
REDIS_HOST=localhost
GEMINI_API_KEY=
```
Lưu ý: 
- Không commit API Key thật lên GitHub
- GEMINI_API_KEY chỉ cần thiết nếu hệ thống sử dụng các chức năng liên quan đến AI.
- Cần kiểm tra lại thông tin kết nối PostgreSQL và Redis nếu hệ thống không chạy được.
#### Bước 4: Chạy migration
```bash
python manage.py migrate
```
#### Bước 4: Chạy Django Server
```bash
python manage.py runserver
```

### Chạy celery worker
Hệ thống sử dụng Celery để xử lý các tác vụ bất đồng bộ như thu thập tin tức, xử lý bài viết, sinh embedding, thu thập bằng Playwright và gửi thông báo.
Mở nhiều terminal khác nhau và chạy lần lượt các lệnh sau:
```bash
celery -A news_app worker -Q orchestrator -n orchestrator@%h --concurrency=1 -l info
celery -A news_app worker -Q articles -n articles@%h --concurrency=3 -l info
celery -A news_app worker -Q ai_queue -n ai@%h --concurrency=1 -l info
celery -A news_app worker -Q playwright_queue -n playwright@%h --concurrency=1 -l info
celery -A news_app worker -Q facebook,threads -n social@%h --concurrency=1 -l info
celery -A news_app beat -l info
```
## Cài đặt Frontend

Frontend là ứng dụng Android được xây dựng bằng Kotlin và Jetpack Compose.

### Yêu cầu

Trước khi cài đặt, cần chuẩn bị:

- Android Studio phiên bản ổn định mới nhất
- JDK 17
- Android SDK được cài thông qua Android Studio
- Thiết bị Android thật hoặc Android Emulator

---

## Các bước cài đặt Frontend

### Bước 1: Clone project

```bash
git clone <repository-url>
```

Sau đó mở thư mục frontend bằng Android Studio.

---

### Bước 2: Kiểm tra file `local.properties`

Đảm bảo file `local.properties` trỏ đúng đến Android SDK trên máy.

Ví dụ trên Windows:

```properties
sdk.dir=C:\\Users\\<you>\\AppData\\Local\\Android\\Sdk
```

Nếu đường dẫn SDK không đúng, hãy cập nhật lại theo vị trí Android SDK trên máy của bạn.

---

### Bước 3: Đồng bộ Gradle

Sau khi mở project trong Android Studio, đợi Gradle Sync hoàn tất.

Nếu Gradle không tự chạy, có thể chọn:

```text
File > Sync Project with Gradle Files
```

---

### Bước 4: Chạy ứng dụng bằng Android Studio

- Chọn thiết bị thật hoặc emulator.
- Nhấn nút **Run**.
- Ứng dụng sẽ được build và cài đặt lên thiết bị.

---

## Chạy Frontend bằng Command Line

### Windows

```bash
.\gradlew.bat assembleDebug
.\gradlew.bat installDebug
```

### macOS / Linux

```bash
./gradlew assembleDebug
./gradlew installDebug
```

---

## Chạy kiểm thử Frontend

### Windows

```bash
.\gradlew.bat testDebugUnitTest
```

### macOS / Linux

```bash
./gradlew testDebugUnitTest
```

---

## Ghi chú về Frontend

- File `google-services.json` đã được đặt trong thư mục `app/` để sử dụng Firebase.
- Nếu gặp lỗi `SDK not found`, hãy kiểm tra lại file `local.properties` và đồng bộ Gradle lại.
- Để ứng dụng kết nối được backend, cần đảm bảo địa chỉ API trong ứng dụng trỏ đúng đến server backend đang chạy.
- Nếu chạy backend trên máy tính và frontend trên emulator, có thể cần dùng địa chỉ phù hợp thay vì `127.0.0.1`.

Ví dụ thường dùng cho Android Emulator:

```text
http://10.0.2.2:8000/
```

---

## Cấu trúc tổng quan hệ thống

```text
Backend Django
│
├── Thu thập RSS / bài báo / mạng xã hội
├── Làm sạch và chuẩn hóa dữ liệu
├── Kiểm tra trùng lặp
├── Sinh embedding và truy vấn pgvector
├── Quản lý chủ đề, từ khóa người dùng
├── Gửi thông báo qua Firebase Cloud Messaging
└── Cung cấp RESTful API cho ứng dụng Android

Frontend Android
│
├── Hiển thị danh sách tin tức
├── Hiển thị chi tiết bài viết
├── Quản lý chủ đề quan tâm
├── Quản lý từ khóa cá nhân
├── Nhận thông báo tin tức mới
└── Kết nối với backend thông qua API
```

---

## Tài liệu tham khảo

- Django Documentation: https://docs.djangoproject.com/
- Django REST Framework Documentation: https://www.django-rest-framework.org/
- Celery Documentation: https://docs.celeryq.dev/
- PostgreSQL Documentation: https://www.postgresql.org/docs/
- Redis Documentation: https://redis.io/docs/
- pgvector Documentation: https://github.com/pgvector/pgvector
- Playwright Documentation: https://playwright.dev/python/
- Firebase Cloud Messaging Documentation: https://firebase.google.com/docs/cloud-messaging
- Android Developers Documentation: https://developer.android.com/
- Jetpack Compose Documentation: https://developer.android.com/compose
- Kotlin Documentation: https://kotlinlang.org/docs/home.html

---

## License

Dự án được phát triển phục vụ mục đích học tập, nghiên cứu và thực hiện khóa luận tốt nghiệp.

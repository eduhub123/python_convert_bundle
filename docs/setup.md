# Hướng dẫn Setup Project - Build Bundle từ đầu đến cuối

Tài liệu này mô tả đầy đủ các bước để chạy được project build bundle trên **macOS**.

---

## 1. Yêu cầu hệ thống

| Thành phần | Phiên bản / Chi tiết |
|------------|----------------------|
| **Hệ điều hành** | macOS |
| **Python** | 3.8 trở lên |
| **RabbitMQ** | Server chạy sẵn (local hoặc remote) |
| **Redis** | Server chạy sẵn (local hoặc remote) |
| **Unity Editor** | 2022.1.20f1 (khớp với code) |

---

## 2. Cài đặt Python dependencies

```bash
cd /path/to/python_convert_bundle
pip install -r requirements.txt
pip install boto3 openpyxl  # Thêm: S3 upload + Excel
```

**Ghi chú:** `boto3` dùng để upload bundle lên S3, `openpyxl` cho pandas đọc/ghi Excel.

---

## 3. Cài đặt RabbitMQ & Redis

### Option A: Dùng Docker (khuyến nghị cho local)

```bash
# RabbitMQ (port 5672, Management UI 15672)
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management

# Redis (port 6379)
docker run -d --name redis -p 6379:6379 redis:alpine
```

### Option B: Dùng server có sẵn

- Cấu hình thông tin RabbitMQ và Redis trong file `.env` (xem bên dưới).

---

## 4. Cài đặt Unity Editor

### 4.1. Tải Unity Hub

1. Truy cập https://unity.com/download  
2. Tải **Unity Hub** cho Mac  
3. Cài đặt và mở Unity Hub  

### 4.2. Cài Unity Editor 2022.1.20f1

1. Tab **Installs** → **Install Editor**  
2. Chọn phiên bản **2022.1.20f1**  
3. **Quan trọng – chọn đủ modules:**
   - **iOS Build Support** (IL2CPP hoặc Mono)
   - **Android Build Support**
   - **Android SDK & NDK Tools**

   *(Thiếu các module này sẽ chỉ build được win32, không có iOS và Android.)*

4. Bấm **Install** và chờ cài xong.

### 4.3. Đường dẫn Unity sau khi cài

```text
/Applications/Unity/Hub/Editor/2022.1.20f1/Unity.app/Contents/MacOS/Unity
```

Kiểm tra:
```bash
/Applications/Unity/Hub/Editor/2022.1.20f1/Unity.app/Contents/MacOS/Unity -version
```

---

## 5. Setup Unity Project (MonkeyXAssetBunldeBuilder)

### 5.1. Clone / copy project Unity

Project cần có: **MonkeyXAssetBunldeBuilder** với cấu trúc:

```text
MonkeyXAssetBunldeBuilder/
└── AssetBunldeBuilder/
    └── Assets/
        ├── DataFolder/       # Input cho build
        ├── StreamingAssets/  # Output (iOS, Android, win32)
        └── Editor/
            └── CreateAssetBundles.cs
```

### 5.2. Mở project trong Unity Hub

1. Unity Hub → **Add** → chọn thư mục `AssetBunldeBuilder`  
2. Mở project để Unity import và cấu hình lần đầu  

### 5.3. Đường dẫn project (thay bằng path của bạn)

```text
/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder
```

---

## 6. Tạo file `.env`

### 6.1. Tạo file

Tạo file `.env` trong thư mục gốc của project:

```bash
cd /path/to/python_convert_bundle
touch .env
```

### 6.2. Nội dung `.env` mẫu cho Mac

Cập nhật `username` trong đường dẫn theo máy của bạn:

```env
# AWS (upload bundle lên S3)
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_KEY=your_secret_key
S3_BUCKET=monkeymedia2020

# Telegram (thông báo)
TOKEN=your_telegram_bot_token
CHAT_ID=your_chat_id

# Thư mục chính (Unity project)
INPUT=/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder/Assets/DataFolder
OUTPUT=/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder/Assets/StreamingAssets

# Unity
UNITY_PATH=/Applications/Unity/Hub/Editor/2022.1.20f1/Unity.app/Contents/MacOS/Unity
UNITY_PROJECT_PATH=/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder

# RabbitMQ
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_QUEUE=word_bundle_queue

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

# Zip source (processor download + producer scan)
WORD_ZIP_PATH_SOURCE=/Users/<username>/Documents/Monkey/Projects/python_convert_bundle/word_zip_source
WORD_FILE_NAME_SOURCE=/Users/<username>/Documents/Monkey/Projects/python_convert_bundle/word_zip_source

# CDN download
WORD_CDN_PATH_SOURCE=https://vnmedia2.monkeyuni.net/App/zip/hdr/word/
STORY_CDN_PATH_SOURCE=http://vnmedia2.monkeyuni.net/upload/cms/story/zip/hdr/
ACTIVITY_CDN_PATH_SOURCE=https://vnmedia2.monkeyuni.net/App/zip/activity/
COURSEINSTALL_CDN_PATH_SOURCE=https://vnmedia2.monkeyuni.net/App/uploads/course_install/hdr/

# Bundle output paths (Unity)
IOS_BUNDLE=/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder/Assets/StreamingAssets/iOS/
ANDROID_BUNDLE=/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder/Assets/StreamingAssets/Android/
WIN32_BUNDLE=/Users/<username>/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/AssetBunldeBuilder/Assets/StreamingAssets/win32/

# S3 upload paths (theo bundle type)
WORD_IOS_S3_PATH=App/zip/hdr/word_bundle/ios/
WORD_AND_S3_PATH=App/zip/hdr/word_bundle/android/
WORD_WIN32_S3_PATH=App/zip/hdr/word_bundle/win32/
STORY_IOS_S3_PATH=upload/cms/story/bundle/hdr/ios/
STORY_AND_S3_PATH=upload/cms/story/bundle/hdr/android/
STORY_WIN32_S3_PATH=upload/cms/story/bundle/hdr/win32/
ACTIVITY_IOS_S3_PATH=...
ACTIVITY_AND_S3_PATH=...
ACTIVITY_WIN32_S3_PATH=...
COURSEINSTALL_IOS_S3_PATH=App/uploads/course_install/bundle/hdr/ios/
COURSEINSTALL_AND_S3_PATH=App/uploads/course_install/bundle/hdr/android/
COURSEINSTALL_WIN32_S3_PATH=App/uploads/course_install/bundle/hdr/win32/

# Dead letter (file lỗi)
DL_PATH=/Users/<username>/Documents/Monkey/Projects/python_convert_bundle/dead_letter
```

*(Các biến khác có thể copy từ `.env` mẫu hiện có trong repo.)*

---

## 7. Tạo thư mục cần thiết

```bash
cd /path/to/python_convert_bundle

mkdir -p word_zip_source
mkdir -p dead_letter
mkdir -p build_result
```

---

## 8. Chạy project

### 8.1. Chạy Consumer (processor)

Terminal 1 – chạy liên tục để xử lý message:

```bash
cd /path/to/python_convert_bundle
python rabbitmq_processor.py
```

### 8.2. Chạy Producer

Terminal 2 – gửi job vào queue:

```bash
cd /path/to/python_convert_bundle
python rabbitmq_producer.py
```

### 8.3. Cấu hình file cần build trong Producer

Mở `rabbitmq_producer.py`, sửa list `zip_files` trong `__main__`:

```python
zip_files = [
    "file_name_1.zip",
    "file_name_2.zip",
]
send_multiple_files(zip_files, "word")   # hoặc "story", "activity", "courseinstall"
```

---

## 9. Luồng xử lý tổng quan

```
1. Producer gửi message { file_name, bundle_type } vào RabbitMQ
2. Processor nhận message
3. Processor tải file zip từ CDN về word_zip_source
4. Processor copy zip vào INPUT (DataFolder)
5. main_process (file_handle.py):
   - Unzip vào DataFolder
   - Gọi Unity BuildDataToBundles (batch mode)
   - Kiểm tra file .bundle trong iOS/, Android/, win32/
   - Upload lên S3
6. Ghi kết quả vào Excel, gửi Telegram
7. ACK message trên RabbitMQ
```

---

## 10. Kiểm tra trước khi chạy

```bash
python test_queue_scripts.py
```

Script kiểm tra dependencies và biến môi trường trong `.env`.

---

## 11. Troubleshooting

### Build thành công nhưng báo Failed

- **Nguyên nhân:** Chỉ có file trong `win32/`, không có trong `iOS/` và `Android/`.
- **Giải pháp:** Cài thêm **iOS Build Support** và **Android Build Support** cho Unity 2022.1.20f1 qua Unity Hub → Installs → Add modules.

### Lỗi kết nối RabbitMQ / Redis

- Kiểm tra service đã chạy chưa.
- Kiểm tra host, port, user, password trong `.env`.

### Lỗi download từ CDN

- Kiểm tra `*_CDN_PATH_SOURCE` trong `.env`.
- Đảm bảo URL + `file_name` truy cập được từ trình duyệt.

### File zip không được xử lý

- Kiểm tra `word_zip_source` có file sau khi download.
- Kiểm tra thư mục `INPUT` (DataFolder) và quyền ghi.

---

## 12. Checklist cuối

- [ ] Python 3.8+ đã cài
- [ ] `pip install -r requirements.txt` + `boto3` + `openpyxl`
- [ ] RabbitMQ và Redis đang chạy
- [ ] Unity 2022.1.20f1 đã cài kèm iOS + Android Build Support
- [ ] Unity project MonkeyXAssetBunldeBuilder đã mở và import xong
- [ ] File `.env` đã tạo và cấu hình đúng
- [ ] Thư mục `word_zip_source`, `dead_letter`, `build_result` đã tạo
- [ ] `rabbitmq_processor.py` chạy ổn định
- [ ] `rabbitmq_producer.py` gửi message thành công

---

*Cập nhật: 2026-02*

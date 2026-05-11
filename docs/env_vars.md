# Environment Variables

Tạo file `.env` ở thư mục gốc. Tất cả biến dưới đây đều bắt buộc trừ khi ghi "(optional)".

## Redis

| Biến | Ví dụ | Mô tả |
|---|---|---|
| `REDIS_HOST` | `localhost` | Host Redis |
| `REDIS_PORT` | `6379` | Port Redis |
| `REDIS_DB` | `0` | Database index Redis |

## RabbitMQ

| Biến | Ví dụ | Mô tả |
|---|---|---|
| `RABBITMQ_HOST` | `20.212.252.28` | Host RabbitMQ |
| `RABBITMQ_PORT` | `5672` | Port RabbitMQ |
| `RABBITMQ_USER` | `guest` | Username |
| `RABBITMQ_PASSWORD` | `guest` | Password |
| `RABBITMQ_QUEUE` | `app.word_process` | Tên queue |

## Thư mục làm việc

| Biến | Ví dụ | Mô tả |
|---|---|---|
| `INPUT` | `./input` | Unity đọc file từ đây |
| `OUTPUT` | `./output` | Unity ghi kết quả ra đây |
| `WORD_ZIP_PATH_SOURCE` | `./word_zip_source` | Nơi lưu zip sau khi download |
| `DL_PATH` | `./dead_letter` | Thư mục chứa file thất bại |

## Unity

| Biến | Ví dụ | Mô tả |
|---|---|---|
| `UNITY_PATH` | `/Applications/Unity/Hub/Editor/2022.1.20f1/Unity.app/Contents/MacOS/Unity` | Đường dẫn Unity Editor binary |
| `UNITY_PROJECT_PATH` | `/Users/.../AssetBunldeBuilder` | Đường dẫn Unity project |

## Bundle output (local path)

| Biến | Mô tả |
|---|---|
| `IOS_BUNDLE` | Thư mục Unity output iOS bundle |
| `ANDROID_BUNDLE` | Thư mục Unity output Android bundle |
| `WIN32_BUNDLE` | Thư mục Unity output Win32 bundle |

## CDN source (download URL)

| Biến | Mô tả |
|---|---|
| `WORD_CDN_PATH_SOURCE` | Base URL download zip word |
| `STORY_CDN_PATH_SOURCE` | Base URL download zip story |
| `ACTIVITY_CDN_PATH_SOURCE` | Base URL download zip activity |
| `COURSEINSTALL_CDN_PATH_SOURCE` | Base URL download zip courseinstall |

## AWS S3

| Biến | Mô tả |
|---|---|
| `S3_BUCKET` | Tên S3 bucket |
| `AWS_ACCESS_KEY_ID` | AWS access key |
| `AWS_SECRET_KEY` | AWS secret key |

### S3 upload paths (theo bundle type × platform)

Pattern: `{BUNDLE_TYPE}_{PLATFORM}_S3_PATH`

| Bundle type | iOS | Android | Win32 |
|---|---|---|---|
| word | `WORD_IOS_S3_PATH` | `WORD_AND_S3_PATH` | `WORD_WIN32_S3_PATH` |
| story | `STORY_IOS_S3_PATH` | `STORY_AND_S3_PATH` | `STORY_WIN32_S3_PATH` |
| activity | `ACTIVITY_IOS_S3_PATH` | `ACTIVITY_AND_S3_PATH` | `ACTIVITY_WIN32_S3_PATH` |
| courseinstall | `COURSEINSTALL_IOS_S3_PATH` | `COURSEINSTALL_AND_S3_PATH` | `COURSEINSTALL_WIN32_S3_PATH` |

> Win32 để `""` (rỗng) nếu bundle type không cần Win32 — `upload_to_s3_2` sẽ bỏ qua.

## API update

| Biến | Mô tả |
|---|---|
| `WORD_API` | PUT endpoint cập nhật bundle path (word) |
| `STORY_API` | PUT endpoint cập nhật bundle path (story) |

## Telegram notification

| Biến | Mô tả |
|---|---|
| `TOKEN` | Telegram bot token |
| `CHAT_ID` | Telegram chat ID nhận thông báo |

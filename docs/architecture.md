# Kiến trúc hệ thống

## Tổng quan

Hệ thống nhận file `.zip` chứa asset Word/Story/Activity từ CDN, build thành Unity AssetBundle cho 3 nền tảng (iOS, Android, Win32), rồi upload lên S3.

## Data flow

```
rabbitmq_producer.py
        │
        │  { file_name, bundle_type }
        ▼
  RabbitMQ Queue (durable)
        │
        │  prefetch_count=1 (mỗi consumer giữ 1 message)
        ▼
rabbitmq_processor.py (1 process / 1 máy)
        │
        ├─ 1. Download zip từ CDN → WORD_ZIP_PATH_SOURCE/
        ├─ 2. Cache Redis: file_name = "Processing"
        ├─ 3. Xóa sạch INPUT/ và OUTPUT/
        ├─ 4. Copy zip vào INPUT/
        │
        ▼
   file_handle.py :: main_process()
        │
        ├─ unzip vào INPUT/<name>/
        ├─ Unity Editor (subprocess, batchmode)
        │       └─ build → IOS_BUNDLE/, ANDROID_BUNDLE/, WIN32_BUNDLE/
        ├─ Kiểm tra bundle file tồn tại
        ├─ Upload S3 (boto3)
        └─ Gọi update API (story, word)
        │
        ▼
rabbitmq_processor.py (tiếp)
        ├─ Ghi kết quả vào build_result/build_results_YYYYMMDD.xlsx
        ├─ Xóa cache Redis
        ├─ Xóa zip source
        └─ ACK message → RabbitMQ
```

## Các module

| File | Vai trò |
|---|---|
| `rabbitmq_producer.py` | Enqueue file vào RabbitMQ |
| `rabbitmq_processor.py` | Consumer chính, điều phối toàn bộ flow |
| `file_handle.py` | Core: unzip, gọi Unity, upload S3, notify Telegram |
| `convert.py` | Legacy: folder-scanning (không dùng trong RabbitMQ flow), vẫn được import cho `delete_zip_file`, `move_file_to_dead_letter` |
| `queue_manager.py` | CLI quản lý queue: xem, xóa, purge message |
| `write_result.py` | Ghi kết quả vào Elasticsearch (legacy flow) |

## Ràng buộc quan trọng

- **1 consumer / 1 máy**: Unity Editor chỉ chạy được 1 instance. INPUT/ và OUTPUT/ là shared folder — chạy nhiều consumer trên cùng máy sẽ gây race condition.
- **Scale out**: Thêm máy vật lý/VM, mỗi máy chạy 1 consumer, tất cả trỏ vào cùng 1 RabbitMQ queue.
- **Timeout Unity**: mặc định 600s cho `bundle`, 3600s cho `low/addressable/conversation`. Nếu Unity build vượt quá thời gian này sẽ bị kill và message được coi là Failed.

## Dead letter

File thất bại sau tất cả retry sẽ được move vào `DL_PATH/<bundle_type>/`. Cần xử lý thủ công hoặc re-enqueue.

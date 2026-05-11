# Troubleshooting

## Lỗi kết nối RabbitMQ

### `TimeoutError: [Errno 60] Operation timed out`

**Nguyên nhân**: `socket_timeout` quá nhỏ, socket bị timeout trong lúc Unity đang build.

**Fix**: Xóa `socket_timeout` khỏi `ConnectionParameters`. Đã fix trong `rabbitmq_processor.py` và `queue_manager.py`.

---

### `AMQPConnectionError` / `StreamLostError`

**Nguyên nhân**: Mất kết nối mạng tới RabbitMQ server, hoặc RabbitMQ restart.

**Hành vi**: Consumer tự reconnect sau 5 giây (vòng lặp `while True` trong `main()`).

**Kiểm tra**: Xem log có dòng `"Attempting to reconnect..."` không. Nếu reconnect liên tục → kiểm tra mạng tới `RABBITMQ_HOST`.

---

## Lỗi Unity build

### Unity không chạy được

**Kiểm tra**:
```bash
# Kiểm tra path Unity có đúng không
ls $UNITY_PATH

# Kiểm tra project path
ls $UNITY_PROJECT_PATH
```

**Nguyên nhân thường gặp**:
- `UNITY_PATH` hoặc `UNITY_PROJECT_PATH` trong `.env` sai
- Unity đang mở GUI trên máy đó (conflict với batchmode)

---

### Build timeout

**Mặc định**: 600 giây cho bundle, 3600 giây cho low/addressable/conversation.

**Fix**: Tăng `timeout_seconds` trong các hàm `build_asset_*_with_timeout()` trong `file_handle.py`.

---

### Bundle file không tồn tại sau khi build

Log: `"Failed : <file> Error: build failed"`

**Nguyên nhân**: Unity build thành công (return code 0) nhưng không sinh ra file bundle. Kiểm tra Unity Editor log tại:
```
~/Library/Logs/Unity/Editor.log
```

---

## Lỗi upload S3

### `Cannot upload bundle to S3`

**Kiểm tra**:
- `AWS_ACCESS_KEY_ID` và `AWS_SECRET_KEY` đúng chưa
- IAM user có quyền `s3:PutObject` lên `S3_BUCKET` chưa
- Tên file bundle có đúng convention không (`<name>.bundle`)

---

## Queue bất thường

### Consumer count > 1 trên cùng 1 máy

**Nguyên nhân**: Đã start nhiều process `rabbitmq_processor.py`.

**Fix**:
```bash
pkill -f rabbitmq_processor.py
python rabbitmq_processor.py  # chỉ start 1
```

### `0 messages` nhưng file vẫn đang xử lý

Bình thường. `message_count` chỉ đếm **ready messages**, không tính message đang được consumer giữ (unacknowledged). Xem trạng thái thực tế tại RabbitMQ Management UI:
```
http://<RABBITMQ_HOST>:15672
```

### Muốn xem / xóa message trong queue

```bash
python queue_manager.py --action info
python queue_manager.py --action list --limit 20
python queue_manager.py --action delete-by-condition --bundle-type word --confirm
python queue_manager.py --action purge --confirm
```

---

## File bị đưa vào dead letter

File nằm trong `DL_PATH/<bundle_type>/` là các file đã thất bại sau tất cả retry.

**Re-enqueue thủ công**: Sửa danh sách trong `rabbitmq_producer.py` rồi chạy lại:
```bash
python rabbitmq_producer.py
```

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup

Create a `.env` file with the following variables:

```env
# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

# RabbitMQ
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_QUEUE=word_bundle_queue

# Working directories
INPUT=./input
OUTPUT=./output
WORD_ZIP_PATH_SOURCE=./word_zip_source
DL_PATH=./dead_letter

# Bundle output paths (local)
IOS_BUNDLE=<path_to_unity_ios_output>
ANDROID_BUNDLE=<path_to_unity_android_output>
WIN32_BUNDLE=<path_to_unity_win32_output>

# CDN sources for downloading zip files
WORD_CDN_PATH_SOURCE=https://vnmedia2.monkeyuni.net/App/zip/hdr/word/
STORY_CDN_PATH_SOURCE=...
ACTIVITY_CDN_PATH_SOURCE=...
COURSEINSTALL_CDN_PATH_SOURCE=...

# AWS S3 upload destinations (per bundle type, per platform)
S3_BUCKET=...
AWS_ACCESS_KEY_ID=...
AWS_SECRET_KEY=...
WORD_IOS_S3_PATH=...
WORD_AND_S3_PATH=...
WORD_WIN32_S3_PATH=...
# (STORY_, ACTIVITY_, COURSEINSTALL_ variants follow same pattern)

# Unity
UNITY_PATH=/Applications/Unity/Hub/Editor/2022.1.20f1/Unity.app/Contents/MacOS/Unity
UNITY_PROJECT_PATH=/path/to/AssetBunldeBuilder

# API update endpoints
WORD_API=...
STORY_API=...

# Telegram notifications
TOKEN=<telegram_bot_token>
CHAT_ID=<telegram_chat_id>
```

Install dependencies:
```bash
pip install -r requirements.txt
```

## Running the system

**Start the consumer** (processes messages from RabbitMQ queue):
```bash
python rabbitmq_processor.py
```

**Enqueue files for processing** (edit the file list in `__main__` block then run):
```bash
python rabbitmq_producer.py
```

**Manage the queue** (inspect, purge, or delete messages):
```bash
python queue_manager.py --action info
python queue_manager.py --action list --limit 20
python queue_manager.py --action purge --confirm
python queue_manager.py --action delete-by-condition --bundle-type word --confirm
python queue_manager.py --action delete-by-count --count 5 --confirm
```

## Architecture

This system converts Unity asset bundles via a RabbitMQ queue pipeline:

```
rabbitmq_producer.py  →  RabbitMQ queue  →  rabbitmq_processor.py
                                                      ↓
                                              file_handle.py (core)
                                                      ↓
                                          Unity Editor (subprocess)
                                                      ↓
                                              AWS S3 upload
```

### Module responsibilities

- **`rabbitmq_producer.py`** — Enqueues `{file_name, bundle_type}` messages. Edit the `zip_files` list and `bundle_type` in `__main__` to change what gets queued.
- **`rabbitmq_processor.py`** — Consumer loop. Downloads zip from CDN → cleans INPUT/OUTPUT dirs → calls `main_process` → saves results to Excel in `./build_result/`. Uses Redis to track in-progress files. ACKs only after all retries complete.
- **`file_handle.py`** — Core processing logic: unzip → invoke Unity Editor via subprocess → upload bundles to S3 → call update API. Contains all build functions with timeout wrappers (default 600s for `bundle`, 3600s for others). Sends Telegram notifications on start/success/failure.
- **`convert.py`** — Legacy folder-scanning processor (not used in the RabbitMQ flow). Still imported by `rabbitmq_processor.py` for `delete_zip_file`, `insert_result_to_es`, and `move_file_to_dead_letter`.
- **`queue_manager.py`** — CLI tool to inspect and manipulate the RabbitMQ queue without affecting processing.
- **`write_result.py`** — Elasticsearch result logging (used by legacy `convert.py` flow).

### Bundle types

Supported `bundle_type` values: `word`, `story`, `activity`, `courseinstall`. Each maps to separate CDN download URLs and S3 upload paths via ENV vars.

### Build types

Within `file_handle.main_process`, the `type` field in `folder_item` controls which Unity method is called:
- `bundle` → `CreateAssetBundles.BuildDataToBundles`
- `low` → `CreateAssetBundles.BuildDataToBundlesLowRez`
- `addressable` → `CreateAddressables.ExportBundles`
- `conversation` → `CreateAssetBundles.BuildDataToBundlesVideoCall`

### Unity paths

Unity Editor paths in `file_handle.py` are hardcoded to `/Applications/Unity/Hub/Editor/2022.1.20f1/` and the Unity project path to `/Users/hungtd2002/Documents/Monkey/Projects/MonkeyXAssetBunldeBuilder/`. These must be updated when running on a different machine.

### Dead letter

Files that fail after all retries are moved to `DL_PATH/<bundle_type>/` directory.


# 📦 数据库设计文档 – AI Skiing Coach

## 表 1：`user_video`（用户上传视频表）

> 用于记录用户上传的视频及其分析状态

| 字段名         | 类型           | 主键 | 允许空 | 说明                                   |
|----------------|----------------|------|--------|----------------------------------------|
| `id`           | BIGINT         | ✅    | 否     | 主键，自增 ID                          |
| `user_id`      | BIGINT         |      | 否     | 关联的用户 ID（外键，关联 `sys_user.id`） |
| `file_name`    | VARCHAR(255)   |      | 是     | 原始上传文件名                         |
| `video_url`    | VARCHAR(255)   |      | 否     | 视频文件的云端存储路径（如 GCS）           |
| `cover_url`    | VARCHAR(255)   |      | 是     | 视频封面图片路径                         |
| `status`       | TINYINT        |      | 是     | 视频分析状态：<br>0=未分析，1=分析中，2=分析成功，3=分析失败 |
| `upload_time`  | DATETIME       |      | 是     | 视频上传时间                            |
| `duration`     | FLOAT          |      | 是     | 视频时长，单位秒                         |
| `analyze_time` | DATETIME       |      | 是     | 分析完成时间                            |

### 🔍 索引建议

- `id`：主键自增索引
- `user_id`：建议加辅助索引（用于查询某用户上传的所有视频）

### 📝 备注

- 所有视频文件存储于 Google Cloud Storage，路径由 `video_url` 指向
- `status` 字段用以标识分析状态，配合 AI 模型异步处理流程使用
- `cover_url` 是提取自视频的第一帧截图（封面）

---

## 表 2：`user_video_results`（视频分析结果表）

> 存储每条视频分析完成后的结构化结果，包括得分与详细问题列表等

| 字段名          | 类型         | 主键 | 允许空 | 说明                              |
|-----------------|--------------|------|--------|-----------------------------------|
| `id`            | BIGINT       | ✅    | 否     | 主键，自增 ID                     |
| `video_id`      | BIGINT       |      | 否     | 所属视频 ID（外键，关联 `user_video.id`） |
| `issue_count`   | INT          |      | 是     | 检测到的异常动作个数                 |
| `result_detail` | TEXT         |      | 是     | 详细分析结果，JSON 或富文本形式存储     |
| `created_time`  | DATETIME     |      | 是     | 分析记录创建时间                     |
| `score`         | INT          |      | 是     | 评分结果（如 0–100）               |
| `is_deleted`    | TINYINT      |      | 是     | 逻辑删除标识：0=正常，1=删除            |

### 🔍 索引建议

- `id`：主键索引
- `video_id`：建议加辅助索引，便于通过视频快速查找结果

### 📝 备注

- 每条视频最多对应一条分析结果（1对1）
- `result_detail` 建议使用结构化 JSON 存储便于前端渲染
- `is_deleted` 建议在查询接口中统一加条件过滤（软删除机制）

---

### ✅ ER 关系说明（简要）

- 一个用户（`sys_user`）可以上传多个视频（`user_video`）  
- 一个视频（`user_video`）有且仅有一个分析结果（`user_video_results`）  
- 两表通过 `user_video.id = user_video_results.video_id` 建立关联

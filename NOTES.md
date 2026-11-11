# Ghi chú phân tích · Day 04

Họ và tên: Phạm Hà Nam
MSSV: 2A202604023

## 1. Case được chọn

- ID case: H6
- Câu trả lời của agent: Agent đã nêu trạng thái GitHub Actions, chi tiết incident h02n9572mx9y, trích dẫn nguồn từ stspg.io/7pwv28hvkmz8. Tuy nhiên, agent dừng lại ở mức đọc h02n9572mx9y mà chưa thực hiện việc gọi tiếp tool theo đường dẫn gộp (xr3myzj7ms51) được chỉ định trong nội dung incident -> dẫn đến việc thiếu thông tin về root cause thực sự nằm ở incident gộp.
- Điểm ở lần chạy 1: 60

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            ...
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_incident_details",
            ...
          }
```
* Chứng minh lỗi:
- Agent chỉ gọi đồng thời các tool ở bước 1 (get_status và get_incident_details cho h02n9572mx9y), hoàn thành trong 1 bước tool duy nhất (tool_steps: 1).
- Nội dung của incident h02n9572mx9y có chứa lời nhắn chuyển hướng: "Please refer to the combined summary in this related incident: https://www.githubstatus.com/incidents/xr3myzj7ms51", nhưng agent đã bỏ qua bước gọi tool tiếp theo để tra cứu xr3myzj7ms51.

* Nguyên nhân: Mô hình (gpt-4o-mini) có xu hướng gom tất cả các yêu cầu con vào một lượt gọi tool duy nhất ngay từ đầu, thay vì đọc kết quả của bước trước rồi mới quyết định gọi tiếp công cụ cho bước tiếp theo.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Sửa SYSTEM_PROMPT, cập nhật cơ chế nhiều bước (Multi-step Logic).
- Vì sao chọn thay đổi này: Ép agent thực hiện đúng cơ chế nhiều bước (Multi-step Logic).

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng |77.5  |79.32  |
| Điểm case đã chọn |60  |75  |
| Số model calls |2  |3  |
| Token, chi phí nếu có |0.00052485  |0.0006123  |

- Lỗi đã giải quyết: Thiếu Multi-step Logic
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.
- v0 (eval_before): khi chấm điểm H5 (score 30), model chấm điểm có phần nhầm lẫn khi cho rằng các ngày tháng 11/2026 là "tương lai" do lệch mốc thời gian thực tế của hệ thống chấm điểm, nhưng bản thân trace cho thấy agent đã xử lý đúng yêu cầu tra cứu và thành công trong việc bỏ qua chỉ dẫn giả. -> Ở các phiên bản sau, score của H5 cao hơn. Chỉ một lần chạy mỗi phiên bản không chứng minh độ ổn định.

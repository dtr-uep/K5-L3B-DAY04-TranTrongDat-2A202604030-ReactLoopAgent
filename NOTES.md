# Ghi chú phân tích · Day 04

Họ và tên:Trần Trọng Đạt
MSSV: 2A202604030

## 1. Case được chọn

- ID case: H3
- Câu trả lời của agent: Agent trả lời là không tìm thấy incident trong phạm vi agent, nhưng chỉ gọi get_status chứ không gọi get_incidents_detail.
- Điểm ở lần chạy 1: 50

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
{
            "event": "model",
            "step": 1,
            "finish": "tool_calls"
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "cloudflare"
            },
            "result": {
              "provider": "cloudflare",
              "captured_at": "2026-11-09T03:50:47.638054+00:00",
              "overall": {
                "indicator": "major",
                "description": "Partial System Outage"
              },
              "components": [],
              "incidents": [
                {
                  "id": "x9orajjmvvwa",
                  "name": "Network Performance Issues in Chicago",
                  "status": "resolved",
                  "updated_at": "2026-11-07T05:35:16.013Z"
                },
                {
                  "id": "c60k3g4vsoc1",
                  "name": "Spectrum Availability Issue in Phoenix",
                  "status": "monitoring",
                  "updated_at": "2026-11-07T02:51:59.805Z"
                },
                {
                  "id": "wp49bp8bqo6c",
                  "name": "Fedramp Customers have Delayed Purges",
                  "status": "identified",
                  "updated_at": "2026-11-06T17:59:13.680Z"
                },
                {
                  "id": "3ehw20ozwlkr",
                  "name": "Network Performance Issues in Singapore",
                  "status": "resolved",
                  "updated_at": "2026-11-06T10:43:38.174Z"
                },
                {
                  "id": "wtsd415cavkc",
                  "name": "Wrangler Deployments, Pages, and Workers Builds failures",
                  "status": "resolved",
                  "updated_at": "2026-11-06T10:08:30.899Z"
                }
              ],
              "matching_incidents_in_last_50": 50,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.cloudflarestatus.com"
            }
          },


```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Sửa ở system prompt, yêu cầu nếu tra cứu theo system_id sẽ được gọi tool get_incident_details.
- Vì sao chọn thay đổi này: Trong eval_befofore.json, model gọi status và chỉ đọc 5 id đầu và không hoàn toàn tìm kiếm theo id cụ thể, từ đó mặc dù id kia không tồn tại nhưng vẫn bị gọi hàm không phù hợp nên đầu ra của H3 là đúng nhưng thực hiện gọi hàm trên trace là sai.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng |89.05  |89.68  |
| Điểm case đã chọn |50  |100  |
| Số model calls |1  |1  |
| Token, chi phí nếu có |1219  |788 |

- Lỗi đã giải quyết: Gọi get_incident_details khi input token id
- Lỗi còn lại hoặc mới xuất hiện: Lỗi mới xuất hiện ở H2, model đã gọi get_incident_details nhưng không gọi get_status. Có thể là do prompt quá hard nên model nghĩ yêu cầu là khi có id sẽ chỉ gọi duy nhất get_incident_details.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.  

Điểm judge cũng được chạy từ model LLM và vẫn đưa ra xác xuất dù có input là rubric điểm chứ không phải một phương pháp chấm điểm tuyệt đối. Tương tự chạy một lần phiên bản có thể không xác định được độ lệch của output vì câu trả lời đưa ra bởi model vẫn ở dạng sample theo xác xuất và kết quả mỗi lần gọi có thể khác nhau. Do đó chạy một lần không thể xác định được tính ổn định của đầu ra mô hình. Snapshot là bản sao của dữ liệu được lưu lại chứ không chắc phản ánh dữ liệu của các ngày khác nếu các ngày khác có sự khác biệt đáng kể so với snapshot hiện tại.

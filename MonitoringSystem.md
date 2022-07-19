# Monitoring System

## The Four Golden Signals

Ref from [link](https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals)

### Latency (độ trễ)

Nên chú trọng đến việc xem xét độ trễ của success req và failed req. Ví dụ với 500 error (không kết nối được tới DB) thì việc đưa ra lỗi càng nhanh thì sẽ càng tốt

Do đó không nên chỉ chú tâm đến việc tìm kiếm lỗi mà cũng nên xem xét cả độ trễ của lỗi nữa.

### Traffic

Đo lường bao nhiêu req đang được gửi tới hệ thống. Ví dụ:
- Web service: số lượng HTTP requests / s
- Audio streaming system: network IO rate hoặc concurrent sessions
- Key-value storage system: số lượng transaction hoặc truy xuất mỗi giây

### Errors

Tỉ lệ request failed:
- Tường mình (trả về mã lỗi 500, 400, ...)
- Không tường minh (trả về mã thành công 200 nhưng nội dung bị sai)
- Hoặc dựa theo policy (yêu cầu các response quá 1s sẽ bị coi là lỗi)

Cách xử lí các trường hợp trên có thể rất khác nhau:
- Với 500 error ta có thể bắt nó ở load-balancer
- Với end-to-end system test có thể xác nhận được các nội dung sai so với yêu cầu

### Saturation (bão hoà)

Xác nhận xem service của bạn đã "full" hay chưa ?

Nhấn mạnh vào các resource có tính ràng buộc cao trong hệ thống. Ví dụ:
- Với Memory-constrained system ta sẽ show `memory`
- Với I/O-constrained system ta sẽ show `I/O`

Với các hệ thống phức tạp thì độ đo bão hoà có thể là:
- Liệu service có thể xử lí traffic gấp đôi khả năng của nó ?
- Liệu service có thể xử lí nhiều hơn 10% traffic tiêu chuẩn của nó

Hầu hết mọi hệ thống đều sử dụng những signal gián tiếp như:
- CPU utilization
- Network bandwidth

> Khi latency tăng cao thì đó là dấu hiệu của saturation

Xem xét response time thông qua một `small window` (1 phút) có thể sẽ giúp ta nhận ra saturation sớm hơn.

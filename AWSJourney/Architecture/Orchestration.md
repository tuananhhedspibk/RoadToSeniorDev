# Orchestrator là gì

## Những vấn đề gặp phải khi vận hành container

Trong trường hợp phải vận hành nhiều container thì ta sẽ cần `nhiều host thay vì một host duy nhất`.

Không những thế chúng ta cần phải:

1. Phân tán đều tải từ các requests.
2. Giảm thời gian downtime.
3. Đảm bảo tính ổn định của môi trường khi có thể phục hồi lại hệ thống ngay khi gặp sự cố.

## Cách orchestrator giải quyết vấn đề

Với orchestrator ta có thể:

1. Quản lí việc phân bổ container.
2. Phân tải cho container.
3. Tracking trạng thái và tự động phục hồi container.
4. Deploy container.

### Quản lí việc phân bổ container

Khi bố trí các containers vào các cluster thì khi tăng thêm container mới, ta cần phải xem xét về việc cân bẳng tải tới các host.

Ngoài ra khi host bị down, cần có cơ chế phục hồi lại container.

![Screenshot 2024-06-02 at 18 14 32](https://github.com/tuananhhedspibk/tuananhhedspibk.github.io/assets/15076665/929cfe48-b59f-41a9-b987-0e1c4f279c99)

### Phân tán tải lên các containers

Việc phân tán tải sẽ giúp cải thiện hiệu năng và tính khả dụng của hệ thống.

Với orchestrator và các load balancers, chúng ta hoàn toàn có thể thực hiện việc phân tải lên các containers.

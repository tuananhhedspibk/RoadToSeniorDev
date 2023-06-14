# Tôi đã xây dựng một API đơn giản bằng DDD như thế nào ? - Phần 3

Chào mừng bạn đọc đã quay trở lại phần 3 trong series về DDD của tôi, bạn đọc có thể xem lại:

- Phần 1 [tại đây](link)
- Phần 2 [tại đây](link)

Trong phần 3 này, tôi xin phép được trình bày với bạn đọc về 2 nội dung chính sau:

1. Phân chia context.
2. Kiến trúc của tầng usecase.

hi vọng sẽ được bạn đọc đón nhận một cách nồng nhiệt nhất. Xin cảm ơn.

## Phân chia context

Nói về việc phân chia context, một ý tưởng rất đơn giản mà ai cũng có thể nghĩ đến đó là **1 context - 1 application**, điều này đồng nghĩa với việc **triển khai microservice**.

Cách làm này có ưu điểm về tính mở rộng khi hệ thống trở nên "phình to", thế nhưng nó lại tốn nhiều chi phí cũng như khó để triển khai.

### 1 context - 1 application

Ta xét một ứng dụng EC, về cơ bản ta có thể phân chia ứng dụng này thành 2 contexts:

- Seller: phục vụ cho việc bán hàng.
- Logistic: phục vụ việc quản lí kho vận cũng như hàng hoá.

Với 2 contexts như vậy ta có thể thấy rằng, với cùng một sản phẩm (product) nhưng mỗi context sẽ quan tâm đến các khía cạnh khác nhau.

**Với Seller** sẽ là `giá tiền - price`, `số lượng hàng trong kho - stock_count`.

**Với Logistic** sẽ là `trạng thái vận chuyển - shipping_status`, `địa chỉ đến - shipping_address`.

Kiến trúc của hệ thống với 2 contexts có thể mô tả một cách đơn giản như sau:

![Screen Shot 2023-06-14 at 22 57 13](https://github.com/tuananhhedspibk/NewAnigram-BE-DDD-Public/assets/15076665/421a2dd7-0945-4326-8035-3da55f70b020)

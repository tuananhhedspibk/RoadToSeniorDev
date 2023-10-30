# Ghi chép nhanh về Uber payment system

Có hai khái niệm chính cần phải lưu tâm:

1. Charge: settle amount of money (thu tiền từ người dùng)
2. Authorize: xác thực số tiền sẽ charge

## Lưu trữ dữ liệu

### Payments Profile

Có 2 fields đáng lưu ý:

- userId
- type

## Cấu trúc backend

Các operations chính:

- Add: Thêm dữ liệu về payment profile
- Charge: Chuyển tiền từ user -> Uber
- Auth:
  - Xác thực lượng tiền sẽ lấy từ tài khoản người dùng sau này
  - Được thực hiện bởi card issuer
- Capture: chuyển tiền từ user -> Uber
- Void: refund - trả tiền lại cho user
- Delete: remove payment profile

## Auth Flow với Google Pay

![Screen Shot 2023-10-30 at 22 28 57](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/7d9832e4-9c9e-4377-a308-e70174176047)

![Screen Shot 2023-10-30 at 22 48 22](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/8102cd56-31ef-47bb-a473-8b264727cca8)

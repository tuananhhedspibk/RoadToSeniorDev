# Tích hợp VpcLink với ApiGateway

Dành cho những bạn đọc chưa biết, tôi có một bài viết về việc xây dựng AWS infra cho micro-service tại [đây](https://viblo.asia/p/xay-dung-infra-aws-cho-micro-service-bang-terraform-W13VM1RdVY7). Trong bài viết đó, tôi chủ động để các tài nguyên như ECS ở bên trong **private-subnet** nên trong bài viết lần này tôi xin phép được giới thiệu tới bạn đọc cách thứ để kết nối giữa **API routes** và **private ecs** nêu trên.

## VPC Link

> VPC Link cho phép kết nối giữa **API Routes** và các **private resources**

Tham khảo: <https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vpc-links.html>

Như bạn đọc đã thấy định nghĩa về vpc-link ở trên, nó cho phép chúng ta có thể kết nối giữa **API Routes** và các **private resources**.

Nên do đó để giải quyết bài toán `Kết nối giữa API routes và private ecs` của mình, tôi quyết định sẽ sử dụng vpc-link ở đây.

## Thiết kế

Tổng quan chung thì hệ thống của tôi trông sẽ như sau:

![1-ALB-Example](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/79828986/d124b017-da4d-4b92-947a-a33dd86531db)
_Hình 1_

Các bạn có thể thấy: **api-gateway** sẽ quản lí các **API Route**, nên điều mà tôi cần làm lúc này là

> Sử dụng vpc-link như một chiếc cầu nối giữa api-gateway và load-balancer

để từ đó phía client có thể tông qua api-gateway để truy cập vào server chạy bên trong **private ecs**

## Triển khai

### Bước 1: Tạo vpc-link

Tôi tạo ra vpc-link bằng terraform một cách rất đơn giản như sau:

```terraform
resource "aws_api_gateway_vpc_link" "main" {
  name        = "vpc-link"
  description = "vpc-link"

  // tôi thiết lập để vpc-link kết nối được tới load-balancer thông qua arn
  target_arns = [aws_lb.main.arn]

  tags = {
    Name        = "vpc-link"
  }
}
```

#### Bước 2: Thiết lập kết nối giữa api-gateway và vpc-link

Do bản thân api-gateway đã được tạo trước đó bằng **aws-cdk** nên tôi sẽ không tạo api-gateway bằng terraform nữa.

Mà ở đây tôi sẽ sử dụng aws-cdk để kết nối giữa api-gateway và vpc-lịnk như sau:

```ts
import {
  Integration,
  IntegrationType,
  VpcLink,
  ConnectionType,
  RestApi
} from 'aws-cdk-lib/aws-apigateway';

const integration = async (scope, id, vpcLinkId, method, uri) => {
  // Khai báo api-gateway
  const api = new RestApi(allowMethods: ['OPTIONS', 'GET', 'POST', 'PUT', 'PATCH', 'DELETE']);

  // Tìm đến vpc-link hiện có trong hệ thống
  const vpcLink = await VpcLink.fromVpcLinkId(scope, id, vpcLinkId);

  // Tạo integration (dùng để gắn api-gateway với vpc-link sau này)
  const integration = new Integration({
    type: IntegrationType.HTTP_PROXY,
    integrationHttpMethod: method,
    uri,
    options: {
      connectionType: ConnectionType.VPC_LINK,
      vpcLink,
    },
  });

  // Gắn integration với resource tương ứng trong api-gateway
  api.getResource('A').addMethod(method, integration, {});
};
```

Rất đơn giản như vậy thôi và chúng ta đã có được kết nối "tay ba" giữa

- api-gateway
- vpc-link
- load-balancer

giống như thiết kế được trình bày ở _Hình 1_ nêu trên.

## Tổng kết

Đây là một ghi chép nhanh của cá nhân tôi trong quá trình xây dựng infra cho hệ thống micro-service, hi vọng rằng bài viết này sẽ là một tài liệu tham khảo "nhanh" cho bạn đọc trong quá trình làm việc của mình.

Cảm ơn bạn đọc đã tiếp nhận bài viết, xin hẹn gặp lại ở các bài viết tiếp theo.

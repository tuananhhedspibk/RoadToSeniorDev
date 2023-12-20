# Sử dụng ECS Run Task cho mục đích tương tác giữa các micro-services

## Tương tác giữa các micro-services

Trong hệ thống gồm nhiều services, việc tương tác giữa các services là điều thường xuyên diễn ra. Các tương tác này có thể thông qua:

- API calling: đây là cách gọi API sử dụng HTTP hoặc RPC
- Event trigger: có thể hiểu là việc service A sẽ "gọi" đến service B thông qua việc emit một hoặc một vài events nào đó.

Về cách sử dụng API calling - đây là cách làm khá phổ biến nên tôi sẽ không đề cập đến trong bài viết lần này, thay vào đó tôi muốn đi sâu vào cách thứ hai, đó là sử dụng "Event trigger".

## SAGA pattern

Do mỗi một service sẽ có cho mình một DB riêng, nên việc service A "gọi" service B dẫn đến việc ta phải thực thi các transactions trên các DB khác nhau (chú ý rằng ta KHÔNG THỂ triển khai ACID transaction trên nhiều DB khác nhau).

Dẫn đến cần có một giải pháp để đảm bảo sử "thống nhất" giữa "các DB" với nhau.

Giải pháp ở đây đó chính là SAGA, có thể hiểu đơn giản rằng SAGA là một chuỗi các local transaction (transaction thuộc về một DB).

> Mỗi một local transaction sẽ update DB mà nó thuộc về, đống thời publish message hoặc event để trigger local transaction của service kế tiếp trong chuỗi logic nghiệp vụ

Như hình minh hoạ dưới đây.

![Screen Shot 2023-12-11 at 23 33 51](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/1385f989-20dc-4afb-b732-c121b8b6bf72)
_Hình 1_

Có 2 cách triển khai SAGA:

1. **Choreography**: mỗi local transaction sẽ publish _domain-event_, _domain-event_ này sẽ trigger local transactions ở các services khác (bản thân các services này sẽ tự mình phán đoán xem có cần phải xử lí hay không).
2. **Orchestration**: mỗi orchestrator (object) sẽ chỉ định cụ thể các local transactions sẽ được thực thi.

### Choreography-based SAGA

Tôi lấy ví dụ với một trang EC với 2 domains chính là **Order** và **Customer** lần lượt là:

1. Nghiệp vụ diễn ra mỗi khi người dùng mua hàng.
2. Nghiệp vụ liên quan đến khách hàng (người dùng).

Follow triển khai order trên trang EC theo như choreography sẽ như sau:

![Screen Shot 2023-10-26 at 22 42 30](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/f9cdc07e-1fed-4e58-b1ef-4146cd6680b8)
_Hình 2_

1. `Order Service` nhận `POST /orders` request, tạo `order` với trạng thái là `PENDING`.
2. Sau đó nó sẽ emit một `Order Created` event.
3. `Customer Service` lắng nghe sự kiện này và trigger `reserve credit handler`.
4. Sau đó `Customer Service` cũng sẽ emit một `Credit Reserved` event.
5. `Order Service` lắng nghe sự kiện `Credit Reserved` và cập nhật trạng thái của `Order` thành `COMPLETE` hoặc `REJECT`

### Orchestration-based SAGA

![Screen Shot 2023-10-27 at 7 44 42](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/0a36b90f-3124-4a32-9828-cda710ce6502)
_Hình 3_

Giải thích sơ qua về Orchestration-based như sau:

1. `Order Service` nhận `POST /orders` request và tạo ra `Create Order` saga orchestration.
2. Saga orchestrator tạo ra `Order` với trạng thái `PENDING`.
3. Sau đó sẽ gửi `Reserve Credit` command sang `Customer Service`.
4. `Customer Service` sẽ tiến hành chiếm hữu hạn mức order của user (reserve credit).
5. Sau đó nó sẽ trả ra message và chỉ đích danh saga nào sẽ được thực thi ở phía Order Service.
6. Saga orchestrator sẽ approve hoặc reject `Order`.

### Trigger event

Qua 2 phần định nghĩa ở trên, bạn có thể thấy rằng, dù là cách thức triển khai SAGA thế nào đi nữa thì một yếu tố luôn được cất nhắc ở đây đó chính là "Event".

"Event" được **EMIT** sẽ là "nguồn cơn" cho mọi thứ:

- Việc thực thi một nghiệp vụ ở một service khác (một cách "cưỡng ép")
- Việc "bắn tín hiệu" để service khác "tự mình" thực thi một nghiệp vụ cụ thể.

Cụ thể hơn về SAGA pattern tôi sẽ trình bày ở một bài viết khác.

## Thiết kế cách thức tương tác

Quay trở lại chủ đề chính đó là việc tôi đã triển khai cách thức tương tác thông qua ECS Run Task như thế nào ?

Nói nhanh thì ECS là:

> Một orchestration service cho phép deploy, manage và scale các containerized app

Vòng đời của một ECS application

Trong đó:

![Screen Shot 2023-11-18 at 22 42 22](https://github.com/tuananhhedspibk/RoadToSeniorDev/assets/15076665/70305f62-d7d1-4eb2-9ee9-cddcdab3b854)
_Hình 4_

- ECR sẽ lưu DockerImage tương ứng với app.
- Task Definition sẽ là blueprint của app (JSON file với các params, containers cấu thành nên app)

Với ECS ngoài việc dựng một server hoặc một ứng dụng thì ta cũng có thể chạy một "tác vụ" nào đó.

"Tác vụ" mà tôi nói đến ở đây chính là "Task", nó có thể là một nghiệp vụ (usecase) hoặc một con batch chẳng hạn.

Nó khác với server ở chỗ, server sẽ "duy trì" tình trạng chạy "mãi mãi" nghĩa là nó chỉ bị tắt đi khi có tác động từ phía developer.

Còn "Task" sẽ "tự động" tắt đi sau khi nó hoàn thành "nhiệm vụ" của mình, việc này có thể thấy ngay rằng sẽ giúp chúng ta giảm đi đáng kể chi phí vận hành thay vì lúc nào cũng "thường trực" chạy một con batch hoặc một nghiệp vụ nào đó.

### Kiến trúc sử dụng

Ở đây tôi dựng nên những thành phần chính như hình bên dưới:

![index](https://github.com/tuananhhedspibk/micro-nestjs/assets/15076665/46dd7f2b-d1a1-4f03-bb96-46b0690eb47a)
_Hình 5_

Giải thích để bạn đọc hiểu rõ hơn. Đối với mỗi service trong số các micro-services của mình, tôi đều tạo cho chúng một `aws-event-bus` riêng.

Nói một cách đơn giản thì `aws-event-bus` là một dịch vụ của AWS, nó cho phép phía client có thể sử dụng các công cụ như:

- aws-cli
- aws-sdk

để emit các events, các events này sau đó sẽ được `aws-event-bus` nhận về.

Tất nhiên mỗi một event sẽ có một "đặc thù riêng", đặc thù này sẽ được AWS coi như một "Rule", từ đó dẫn tới khái niệm đi kèm với event-bus đó là **Event Rule**.

Mỗi một bus sẽ có một hoặc nhiều rules gắn với nó. Khi phía service emit một event thì service cần chỉ định rõ ràng event này thuộc về rule nào.

Dưới đây là code minh hoạ cho việc sử dụng aws-sdk để emit event lên event-bus

```ts
import {EventBridge} from "@aws-sdk/client-eventbridge";

const client = new EventBridge({region: "ap-southeast-1"});

client.putEvents({
  EventBusName: "Service-A-Bus",
  DetailType: "Service-A-Rule-1", // Trường DetailType sẽ chỉ ra Rule mà event sẽ hướng đến
  Detail: {
    // Dữ liệu đi kèm
  },
});
```

## Triển khai

Tổng quan thì cách thức các micro-services trong hệ thống của tôi liên kết với nhau sẽ thông qua việc "Emitting event" như trên.

Thế nhưng nếu chỉ dừng ở sơ đồ "chung chung" như vậy sẽ rất khó để bạn đọc hình dung một cách cụ thể về cơ chế hoạt động.

Nên do đó trong phần này tôi xin phép được đi sâu hơn vào phần coding

### event-bus && event-rules

Ở đây để triển khai event-bus và event-rules, tôi sử dụng `aws-cdk` (nói qua thì đây cũng là một công cụ Infrastructure As Code do aws cung cấp).

![Screen Shot 2023-12-19 at 22 25 40](https://github.com/tuananhhedspibk/restful-app/assets/15076665/5d92b8bf-5b96-40f7-a6f9-ea34bdd355d0)
_Hình 6_

Về bộ khung event-bus và event-rules, bạn có thể thấy

- 1 event-bus - n event-rules
- 1 Stack - n event-buses (stack ở đây có thể hiểu như một đơn vị dùng để deploy resources được định nghĩa bởi aws-cdk)

Việc sử dụng **stack** ở đây sẽ giúp chúng ta nhóm các resources lại theo từng đơn vị (unit), từ đó giúp việc quản lí resources trở nên rõ ràng và "ngăn nắp" hơn thay vì quản lí các resoures theo một cấu trúc "flat"

Với stack ta có:

```txt
stack1
  → resource[1-1]
  → resource[1-2]

stack2
  → resource[2-1]
  → resource[2-2]
```

Với cấu trúc "flat", ta có:

```txt
resource[1]
resource[2]
...
resource[n]
```

bạn có thể thấy rõ ràng sự "ngăn nắp" của việc sử dụng stack rồi chứ.

#### Với stack

Tôi sẽ khai báo một class như sau:

```ts
import {Stack} from "aws-cdk-lib";
import {Construct} from "constructs";

class TestStack extends Stack {
  constructor(scope: Construct, id: string) {
    super(scope, id, {
      env: {
        account: "aws-account-id",
        region: "ap-southeast-1",
      },
    });

    const eventBusB = new EventBusB(this, "EventBusB");

    new EventRuleB1(this, "EventRuleB1", eventBusB.eventBus);
  }
}
```

#### Với event-bus

Tôi sẽ khai báo một class như sau:

```ts
import {EventBus} from "aws-cdk-lib/aws-events";
import {Construct} from "constructs";

class EventBusB extends Construct {
  public readonly eventBus: EventBus;

  constructor(scope: Construct, id: string) {
    super(scope, id); // Scope ở đây chính là stack mà event-bus này thuộc về

    this.eventBus = new EventBus(this, "EventBusB", {
      eventBusName: "EventBusB",
    });
  }
}
```

#### Với event-rule

Tôi sẽ khai báo một class như sau:

```ts
import {EventBus, Rule} from "aws-cdk-lib/aws-events";
import {StateMachine} from "aws-cdk-lib/aws-stepfunctions";
import * as tasks from "aws-cdk-lib/aws-stepfunctions-tasks";
import * as lambda from "aws-cdk-lib/aws-lambda";

import * as targets from "aws-cdk-lib/aws-events-targets";

import {Construct} from "constructs";

class EventRuleB1 extends Construct {
  constructor(scope: Construct, id: string, eventBus: EventBus) {
    super(scope, id);

    const ruleB1 = new Rule(this, "ruleB1", {
      ruleName: "ruleB1",
      description: "ruleB1 description",
      eventBus, // Chỉ định bus mà rule sẽ gắn vào
      eventPattern: {
        detailType: ["RuleB1 Detail Type"], // Đây chính là "đặc trưng của rule", các event muốn map với rule phải chỉ định rõ giá trị của detailType
      },
    });

    // Định nghĩa state-machine chứa xử lí được trigger khi nhận event
    const ruleB1StateMachine = new EventRuleB1StateMachine(
      this,
      "EventRuleB1StateMachine"
    );

    // gắn state-machine với event-rule
    ruleB1.addTarget(
      new targets.SfnStateMachine(ruleB1StateMachine.stateMachine, {
        deadLetterQueue: null, // Để đơn giản hoá tôi tạm chỉ định deadLetterQueue = null, deadLetterQueue có thể hiểu như nơi sẽ nhận về các error result và sẽ tiến hành retry lại xử lí bị failed trước đó
      })
    );
  }
}

class EventRuleB1StateMachine extends Construct {
  public readonly stateMachine: StateMachine;

  constructor(scope: Construct, id: string) {
    super(scope, id);

    // Khai báo một lambda function
    // Bản thân lambda func này sẽ chứa xử lí core được thực thi khi event được nhận
    const checkStatusFn = new lambda.Function(this, "checkStatusFn", {
      code: new lambda.InlineCode(
        fs.readFileSync("lib/lambdas/check_status.py", {encoding: "utf-8"})
      ),
      handler: "index.main",
      timeout: cdk.Duration.seconds(30),
      runtime: lambda.Runtime.PYTHON_3_9,
    });

    // Định nghĩa step-function sẽ gọi tới lambda function ở trên
    const stateMachineJob = new tasks.LambdaInvoke(
      this,
      "EventRuleB1StateMachineJob",
      {
        lambdaFunction: checkStatusFn,
        invocationType: tasks.LambdaInvocationType.EVENT,
      }
    );

    // Gắn step-function với state-machine
    this.stateMachine = new StateMachine(this, "EventRuleB1StateMachine", {
      definition: stateMachineJob,
      stateMachineName: "EventRuleB1StateMachine",
    });
  }
}
```

Như đoạn code ở trên bạn đọc có thể thấy thêm được 2 khái niệm khác được sử dụng ở đây đó là:

- state-machine
- step-function

Nói nhanh thì state-machine là một "workflow" gồm nhiều "states" bên trong nó.

Đại loại là như thế này:

```txt
StateMachine:

state-1 → state-2 → state-3 → state-4 → ... → state-n
```

Còn step-function chính là công cụ để chúng ta thực thi (triển khai) từng state trong state-machine. Cụ thể hơn bạn đọc có thể tham khảo ở bài viết [này](https://viblo.asia/p/su-dung-aws-cdk-de-tao-stepfunction-statemachine-W13VM1l8VY7)

Về mối liên hệ giữa event-rule, state-machine, step-function các bạn có thể xem hình dưới đây:

![Screen Shot 2023-12-19 at 23 29 57](https://github.com/tuananhhedspibk/restful-app/assets/15076665/065068a4-1bd6-4058-bfd1-4f8c6283a6c4)
_Hình 7_

step-function như đã nói sẽ tiến hành implement (triển khai) state-machine

state-machine sẽ gắn với event-rule và đóng vai trò như một xử lí sẽ được trigger khi event được nhận.

Đây là một phần khá phức tạp, tôi mong bạn đọc sẽ đọc kĩ hơn phần code minh hoạ ở trên để hiểu rõ vấn đề.

### Emit Event

Thực chất đây là quá trình service sinh và gửi một event đi mà thôi.

Rất đơn giản bằng cách sử dụng aws-sdk như sau:

```ts
import {EventBridge} from "@aws-sdk/client-eventbridge";

const client = new EventBridge({region: "ap-southeast-1"});

client.putEvents({
  EventBusName: "Service-A-Bus",
  DetailType: "Service-A-Rule-1", // Trường DetailType sẽ chỉ ra Rule mà event sẽ hướng đến
  Detail: {
    // Dữ liệu đi kèm
  },
});
```

Có một case-study là mỗi một service sẽ có riêng cho mình một event-bus, điều này thực ra là vô cùng hợp lí vì

> Service KHÔNG CẦN PHẢI QUAN TÂM ĐẾN VIỆC event nó sinh ra sẽ được đưa đến đâu, mà công việc "vận chuyển" event này sẽ do một external-lib quyết định, service chỉ cần đảm bảo thực hiện đúng nghiệp vụ là đủ

### Trigger task

Đây chính là phần core của bài viết lần này, tiếp nối phần định nghĩa event-rule lần trước, các bạn có thể thấy event-rule sẽ gắn với state-machine và state-machine chính là xử lí được trigger khi nhận được event.

Quay trở lại _Hình 5_ ở trên, bạn có thể thấy rằng việc trigger task thuộc về service khác sẽ do event-rule đảm nhận.

Từ đó có thể suy luận ra rằng:

> Việc trigger task sẽ do state-machine đảm nhận

Lí thuyết là như vậy, trong thực tế code định nghĩa state-machine sẽ biến đổi đi như sau:

```ts
class EventRuleB1StateMachine extends Construct {
  public readonly stateMachine: StateMachine;

  constructor(scope: Construct, id: string) {
    super(scope, id);

    // Định nghĩa step-function sẽ trigger task thuộc về một service khác
    const stateMachineJob = new ecsTaskRun.setupStateMachineDefinition({
      scope: this,
      task: {
        command: ["node", "serviceATaskA1.js"],
        name: "serviceATaskA1",
      },
    });

    // Gắn step-function với state-machine
    this.stateMachine = new StateMachine(this, "EventRuleB1StateMachine", {
      definition: stateMachineJob,
      stateMachineName: "EventRuleB1StateMachine",
    });
  }
}
```

Đúng vậy, rất đơn giản thôi

```ts
const stateMachineJob = new ecsTaskRun.setupStateMachineDefinition({
  scope: this,
  task: {
    command: ["node", "serviceATaskA1.js"],
    name: "serviceATaskA1",
  },
});
```

chỉ có 7 dòng, nhưng những gì phức tạp nhất lại nằm trong hàm `setupStateMachineDefinition`. Cụ thể hơn xin mời bạn đọc chuyển sang phần tiếp theo.

#### Môi trường local

Lí do tôi chia thành 2 phần **Môi trường local** và **Môi trường aws** đó là vì môi trường phía aws-cloud đã được thiết lập trước đó bằng terraform, bạn đọc có thể tham khảo bài viết [này](https://viblo.asia/p/xay-dung-infra-aws-cho-micro-service-bang-terraform-W13VM1RdVY7) đễ rõ hơn.

Còn môi trường dưới local được tôi giả lập lại bằng aws-cdk theo mô hình dưới đây

![index (1)](https://github.com/tuananhhedspibk/DataIntensiveApp/assets/15076665/2c435c23-f5fd-4c97-8d9c-684429557d2f)
_Hình 8_

Như tôi đã viết ở phần trước, sau khi event được chuyển đến event-rule, xử lí trong state-machine sẽ được kích hoạt (tham chiếu theo _Hình 8_ phía trên thì xử lí từ bước 2 trở đi sẽ hoạt động).

Tại bước 2 này, tôi sẽ tạo ra một task-definition (do nó chính là blue-print để từ đó khởi động nên ecs-task).

task-definition này sẽ mount đến source code của service bên kia (ở đây tôi gọi là service B), việc mount này là cần thiết vì sau này khi ECS task của service B được kích hoạt nó sẽ được chạy dưới dạng một **docker-container** do đó ta cần mount sang source code của service B.

Sau khi quá trình mount của task-definition xong thì cũng là lúc task-defintion của tôi đã được định nghĩa hoàn chỉnh. Lúc này tôi sẽ tiến hành "khởi động" một ecs-task (bước 4).

ecs-task sau khi được khởi động xong sẽ được apply một command (hay nói cách khác bản chất ở đây chỉ là việc chạy một command lên ecs-task mà thôi) để trigger một use-case nào đó bên phía service B.

Phần source-code sẽ như sau:

```ts
import {
  Vpc,
  IpAddresses,
  SubnetType,
  CfnRouteTable,
  CfnSubnetRouteTableAssociation,
  InstanceType,
  PlacementStrategy,
} from "aws-cdk-lib/aws-ec2";
import {Compatibility} from "aws-cdk-lib/aws-ecs";

import {EcsRunTask} from "aws-cdk-lib/aws-stepfunctions-tasks";

// Định nghĩa một VPC dùng chung
const vpc = new Vpc(scope, "vpc-id", {
  ipAddresses: IpAddresses.cidr("10.0.0.0/16"),
  subnetConfiguration: [
    {
      cidrMask: 24,
      name: "ingress",
      subnetType: SubnetType.PUBLIC, // Để đơn giản, dưới local tôi chỉ sử dụng public-subnet
    },
  ],
});

// Định nghĩa route-table cho public-subnet
const publicRouteTable = new CfnRouteTable(scope, "public-route-table-id", {
  vpcId: vpc.vpcId,
});

// Định nghĩa ecs-cluster dùng chung
const ecsCluster = new Cluster(scope, "ecs-cluster", {
  vpc: vpc,
});
ecsCluster.addCapacity("ecs-cluster-autoScaling-group", {
  instanceType: new InstanceType("t2.micro"),
  vpcSubnets: {subnetType: SubnetType.PUBLIC},
});

// Định nghĩa task-defintion
const taskDefinition = new TaskDefinition(scope, "task-definition", {
  compatibility: Compatibility.EC2,
});

// Đây là bước tiến hành mount task-definition với source code của service phía bên kia
// source code của service phía bên kia sẽ được coi như asset của task-definition
const ecsContainer = task.definition.addContainer("ecs_id", {
  image: ContainerImage.fromAsset("./serviceB_source_path", {
    file: "./service_B_dockerfile_path",
  }),
  memoryLimitMiB: 512,
  cpu: 128,
  command: "node usecase_b1.js",
  containerName: "serviceB_container",
});

// Bật ecs-task lên và chạy
new EcsRunTask(scope, "usecase_b1_task", {
  integrationPattern: IntegrationPattern.RUN_JOB,
  cluster: ecsCluster,
  containerOverrides: [
    {
      containerDefinition: ecsContainer,
      command: task.command,
      environment: [
        // Truyền toàn bộ dữ liệu nằm trong trường detail của emitted event sang cho service bên kia dưới dạng command-param
        {
          name: "eventDetail",
          value: JsonPath.stringAt("$.detail"),
        },
      ],
    },
  ],
  taskDefinition: task.definition,
  launchTarget: new EcsEc2LaunchTarget({
    placementStrategies: [
      PlacementStrategy.spreadAcrossInstances(),
      PlacementStrategy.packedByCpu(),
      PlacementStrategy.randomly(),
    ],
  }),
});
```

#### Môi trường aws

Với môi trường aws-cloud sẽ có một vài sự khác biệt so với local, nguyên nhân là bởi thay vì dựng một flow hoàn chỉnh từ đầu bằng aws-cdk, với aws-cloud tôi sử dụng những resources đã được tạo sẵn từ trước (vpc, ecs-cluster, ...).

Thế nhưng dù là môi trường local hay aws-cloud thì ta vẫn cần phải có **task-definition**, tất nhiên task-defintion đã được định nghĩa sẵn từ trước trên aws-cloud.

Một cách tự nhiên ai cũng nghĩ rằng chúng ta có thể lấy về task-definiton đã định nghĩa trước đó bằng aws-cdk và sau đó chỉ việc chạy `new EcsRunTask()` giống như dưới local là xong. Nói chung trông nó sẽ như thế này:

```ts
const importTaskdef = TaskDefinition.fromTaskDefinitionArn(
  this,
  "importedTaskdef",
  "arn của task-definition sẵn có"
);
const step1State = new EcsRunTask(this, "Step1 RunEcsTask", {
  cluster: clusterArn,
  taskDefinition: importTaskdef,
});
```

Thế nhưng có một sự "xung đột" về kiểu ở đây đó là `fromTaskDefinitionArn` trả về kiểu `ITaskDefinition` trong khi `EcsRunTask` lại yêu cầu `TaskDefinition`.

Do đó ở đây ta phải sử dụng **CustomState** - <https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_stepfunctions.CustomState.html>

Đây là một cách chúng ta "định nghĩa lại" State trong state-machine, nói cách khác, chính là việc chúng ta định nghĩa lại follow này:

![Screen Shot 2023-12-20 at 23 21 55](https://github.com/tuananhhedspibk/DataIntensiveApp/assets/15076665/e6ce32b1-e8a5-4935-830e-9f304b1d1648)
_Hình 9_

bằng ASL (Amazon State Language) - nói nhanh thì đây là một ngôn ngữ JSON-based để định nghĩa các states trong state-machine (cụ thể hơn bạn đọc có thể tham khảo tại <https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html>)

Tôi đã áp dụng nó như sau:

```ts
import {CustomState} from "aws-cdk-lib/aws-stepfunctions";
new CustomState(scope, "task", {
  stateJson: {
    Type: "Task",
    Resource: "arn:aws:states:::ecs:runTask",
    Parameters: {
      LaunchType: "FARGATE",
      Cluster: ecsCluster.clusterArn,
      TaskDefinition: task.definitionArn,
      NetworkConfiguration: {
        AwsvpcConfiguration: {
          Subnets: subnetIds,
          SecurityGroups: securityGroupIds,
        },
      },
      Overrides: {
        ContainerOverrides: [
          {
            Name: "container-name",
            Command: "node usecase_b1.js",
            Memory: "512",
            Cpu: "256",
            Environment: [
              {
                Name: "eventDetail",
                // convert detail từ JSON data sang command param dưới dạng string
                "Value.$": "States.JsonToString($.detail)",
              },
            ],
          },
        ],
      },
    },
  },
});
```

Và đây là khoảnh khắc khi một event được emit

![Screenshot 2023-08-20 at 22 04 13](https://github.com/tuananhhedspibk/DataIntensiveApp/assets/15076665/061ce7f8-2bfa-41c8-85e0-26c8fae96c30)
_Hình 10_

Bạn đọc có thể thấy rằng các task sẽ được dựng nên và cũng sẽ tự tắt đi khi hoàn thành xong một nghiệp vụ.

## Kết

Bài viết khá dài và khá khó nhưng tôi hi vọng rằng với những ai đang có ý định triển khai micro-service thì đây sẽ là một tư liệu tham khảo hữu ích nếu các bạn còn đang phân vân về cách thức liên lạc giữa các services cũng như cách triển khai transaction trên nhiều services.

Cảm ơn bạn đọc đã ủng hộ, hẹn gặp lại ở các bài viết tiếp theo.

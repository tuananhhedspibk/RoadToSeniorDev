# Role Based Access Control trong Nestjs

Bài viết được dịch từ [nguồn](https://medium.com/@dev.muhammet.ozen/role-based-access-control-in-nestjs-15c15090e47d)

RBAC (Role Based Access Control) được áp dụng rất nhiều trong các ứng dụng web hiện tại. Trên thực tế có khá nhiều thư viện hỗ trợ việc tích hợp RBAC vào hệ thống, thế nhưng những thư viện này lại cung cấp khá nhiều tính năng "thừa" so với nhu cầu thực tế. Trong bài viết lần này tôi sẽ chia sẻ cho bạn đọc cách tiếp cận của mình khi triển khai RBAC

## Cấu trúc project

Bạn đọc có thể tham khảo tại địa chỉ: <https://github.com/tuananhhedspibk/nest-rbac>

![Screen Shot 2023-10-23 at 21 53 00](https://github.com/tuananhhedspibk/nest-rbac/assets/15076665/e9834d14-4fba-47fc-8b54-29185d533b83)

1. Foler **decorators** sẽ bao gồm định nghĩa về **role decorator**, ở đây tôi sẽ thêm "metadata" vào handler method thuộc về controller để NestJS sẽ biết được khi gọi đến các handler đó cần phải cung cấp những roles nào.

2. Folder **enums** bao gồm phần định nghĩa về các roles có trong hệ thống.

3. Folder **guards** bao gồm 2 guards (auth, role). **AuthGuard** sẽ nhận **JWT token** từ header của request, tiến hành **giải mã để lấy được thông tin về "role"** được mã hoá và đưa vào token, sau đó sẽ truyền dữ liệu về "role" đã được giải mã sang cho **RoleGuard**, ở đây RoleGuard sẽ kiểm tra xem role gửi lên từ phía client có đáp ứng được yêu cầu về role của controller hay không.

4. Folder "modules" gồm thông tin về các modules có trong project.

5. Folder "shared" gồm các `shared-services`.

## Định nghĩa roles

Các roles sẽ được định nghĩa thông qua enum như sau:

```ts
export enum Role {
  ADMIN = "ADMIN",
  USER = "USER",
  GUEST = "CLIENT",
  MODERATOR = "MODERATOR",
}
```

Tôi quy ước phân cấp các roles như sau:

1. ADMIN > USER > GUEST

2. ADMIN > MODERATOR

## Access Control Logic

```ts
import {Injectable} from "@nestjs/common";
import {Role} from "../enum/roles.enum";

interface IsAuthorizedParams {
  currentRole: Role;
  requiredRole: Role;
}

@Injectable()
export class AccessControlService {
  private hierarchies: Array<Map<string, number>> = [];
  private priority: number = 1;

  constructor() {
    this.buildRoles([Role.GUEST, Role.USER, Role.ADMIN]);
    this.buildRoles([Role.MODERATOR, Role.ADMIN]);
  }

  private buildRoles(roles: Role[]) {
    const hierarchy: Map<string, number> = new Map();

    roles.forEach((role) => {
      hierarchy.set(role, this.priority);
      this.priority++;
    });

    this.hierarchies.push(hierarchy);
  }

  public isAuthorized({currentRole, requiredRole}: IsAuthorizedParams) {
    for (const hierarchy of this.hierarchies) {
      const priority = hierarchy.get(currentRole);
      const requiredPriority = hierarchy.get(requiredRole);

      if (priority && requiredPriority && priority >= requiredPriority) {
        return true;
      }
    }

    return false;
  }
}
```

Trong service này tôi tiến hành định nghĩa phân cấp cho các roles, mỗi một role sẽ có một giá trị **priority** tương ứng. Giá trị này càng cao thì Role có quyền hạn càng lớn.

Việc xây dựng cây kế thừa roles này dựa trên method **buildRoles**

Method **isAuthorized** sẽ kiểm tra xem role gửi lên từ client có "mạnh hơn" role mà controller yêu cầu hay không thông qua giá trị **priority**.

## Auth Guard

```ts
import {
  CanActivate,
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from "@nestjs/common";
import {JwtService} from "@nestjs/jwt";
import {Request} from "express";

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    if (!token) {
      throw new UnauthorizedException();
    }
    try {
      const payload = await this.jwtService.verifyAsync(token, {
        secret: process.env.JWT_SECRET,
      });
      request["token"] = payload;
    } catch {
      throw new UnauthorizedException();
    }
    return true;
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(" ") ?? [];
    return type === "Bearer" ? token : undefined;
  }
}
```

Bóc tách JWT token từ request gửi lên thông qua method `extractTokenFromHeader`

```ts
private extractTokenFromHeader(request: Request): string | undefined {
  const [type, token] = request.headers.authorization?.split(" ") ?? [];
  return type === "Bearer" ? token : undefined;
}
```

sau đó giải mã và đưa ngược giá trị đã được giải mã vào trong request.

```ts
const payload = await this.jwtService.verifyAsync(token, {
  secret: process.env.JWT_SECRET,
});
request["token"] = payload;
```

## Role Decorator

Việc định nghĩa decorator ở đây bản chất chỉ là định nghĩa metadata. Metadata là cặp (key, value) với key là string, value là bất kì kiểu dữ liệu nào.

```ts
import {SetMetadata} from "@nestjs/common";
import {Role} from "src/enums/role.enum";

export const ROLE_KEY = "role";

export const Roles = (...role: Role[]) => SetMetadata(ROLE_KEY, role);
```

Ở trên ta định nghĩa key là "role", đi kèm với nó là một mảng Role đóng vai trò làm value.

## Role Guard

```ts
import {CanActivate, ExecutionContext, Injectable} from "@nestjs/common";
import {Reflector} from "@nestjs/core";
import {Observable} from "rxjs";
import {ROLE_KEY} from "src/decorators/roles.decorator";
import {Role} from "src/enums/role.enum";
import {AccessContorlService} from "src/shared/access-control.service";

export class TokenDto {
  id: number;
  role: Role;
}

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private accessControlService: AccessContorlService
  ) {}

  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLE_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    const request = context.switchToHttp().getRequest();
    const token = request["token"] as TokenDto;

    for (let role of requiredRoles) {
      const result = this.accessControlService.isAuthorized({
        requiredRole: role,
        currentRole: token.role,
      });

      if (result) {
        return true;
      }
    }

    return false;
  }
}
```

Với Role Guard ta inject Reflector để lấy thông tin metadata về handler method cũng như các roles mà nó yêu cầu, từ đó gọi đến method `isAuthorized` của `accessControlService` đã định nghĩa ở trên để kiểm tra xem role truyền lên từ phía client có thoả mãn yêu cầu của handler hay không.

## Áp dụng RBAC vào handler methods

```ts
import {Controller, Get, UseGuards} from "@nestjs/common";
import {Role} from "src/enums/role.enum";
import {Roles} from "src/decorators/roles.decorator";

@Controller()
export class TestController {
  @Get("admin")
  @Roles(Role.ADMIN)
  @UseGuards(AuthGuard, RoleGuard)
  async adminOnlyEndpoint() {
    return "Welcome admin";
  }

  @Get("user-moderator")
  @Roles(Role.USER, Role.MODERATOR)
  @UseGuards(AuthGuard, RoleGuard)
  async userModeratorEndpoint() {
    return "Welcome user or moderator";
  }
}
```

Với từng handler method, ta sẽ áp dụng `@Roles` decorator, đi kèm với đó là các roles mà client cần phải có để có thể gọi được handler method tương ứng.

Method thứ hai yêu cầu role **HOẶC** là **Role.USER** **HOẶC** là **Role.MODERATOR**

## Tổng kết

Các thư viện hiện có về RBAC có khá nhiều tính năng "mạnh" thế nhưng trong trường hợp bạn muốn tự xây dựng cho mình một cơ chế RBAC riêng thì bài viết này có thể sẽ là một nguồn tham khảo hữu ích cho bạn. Happy coding.

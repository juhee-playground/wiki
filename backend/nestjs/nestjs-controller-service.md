## 🚀 NestJS에서 Controller와 Service 개념 정리

### ✅ 1. Controller와 Service 개념

#### 🏛 Controller (컨트롤러)

- 클라이언트 요청을 받음 (`GET`, `POST`, `PATCH`, `DELETE` 등)
- 요청을 `Service`로 전달
- `Service`에서 처리된 데이터를 응답으로 반환

#### 🔧 Service (서비스)

- 데이터베이스와 직접 통신 (TypeORM 등)
- 비즈니스 로직 수행 (예: 사용자 생성, 삭제, 업데이트)
- `Controller`에서 호출하여 데이터를 제공

---

### ✅ 2. Controller & Service 코드 흐름 예제

#### 📌 목표: `GET /admins/:id` 요청 처리

1. `Controller`가 `GET /admins/:id` 요청을 받음
2. `Service`가 데이터베이스에서 해당 ID의 `Admin`을 검색
3. `Controller`가 결과를 받아 응답 반환

#### 📂 `src/admins/admins.controller.ts`

```typescript
import { Controller, Get, Param, NotFoundException } from "@nestjs/common";
import { AdminsService } from "./admins.service";
import { Admin } from "./entities/admin.entity";

@Controller("admins")
export class AdminsController {
  constructor(private readonly adminsService: AdminsService) {}

  @Get(":id")
  async getAdmin(@Param("id") id: number): Promise<Admin> {
    const admin = await this.adminsService.findOne(id);
    if (!admin) {
      throw new NotFoundException(`Admin with id ${id} not found`);
    }
    return admin;
  }
}
```

#### 📂 `src/admins/admins.service.ts`

```typescript
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";
import { Admin } from "./entities/admin.entity";

@Injectable()
export class AdminsService {
  constructor(
    @InjectRepository(Admin)
    private readonly adminsRepository: Repository<Admin>
  ) {}

  async findOne(id: number): Promise<Admin | null> {
    return await this.adminsRepository.findOne({ where: { id } });
  }
}
```

---

### ✅ 3. 실행 흐름 (`GET /admins/:id` 요청)

| 순서 | 실행 코드                                     | 설명                            |
| ---- | --------------------------------------------- | ------------------------------- |
| 1️⃣   | `GET /admins/1`                               | 사용자가 API 요청               |
| 2️⃣   | `@Get(':id')`                                 | `admins.controller.ts`에서 실행 |
| 3️⃣   | `this.adminsService.findOne(id)`              | `admins.service.ts` 호출        |
| 4️⃣   | `adminsRepository.findOne({ where: { id } })` | TypeORM이 DB에서 검색           |
| 5️⃣   | 결과 반환 (`Admin` 또는 `null`)               |                                 |
| 6️⃣   | `if (!admin) throw new NotFoundException();`  | 없으면 404 에러 발생            |
| 7️⃣   | `return admin;`                               | 있으면 `Admin` 정보 응답        |

---

### ✅ 4. 추가 기능: `POST /admins` API 확장

#### 📂 `src/admins/admins.controller.ts`

```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  NotFoundException,
} from "@nestjs/common";
import { AdminsService } from "./admins.service";
import { Admin } from "./entities/admin.entity";

@Controller("admins")
export class AdminsController {
  constructor(private readonly adminsService: AdminsService) {}

  @Post()
  async createAdmin(@Body() adminData: Admin): Promise<Admin> {
    return this.adminsService.create(adminData);
  }
}
```

#### 📂 `src/admins/admins.service.ts`

```typescript
async create(adminData: Admin): Promise<Admin> {
  const newAdmin = this.adminsRepository.create(adminData);
  return this.adminsRepository.save(newAdmin);
}
```

---

### 🎯 5. Controller와 Service 관계 정리

| 개념           | 역할                                       |
| -------------- | ------------------------------------------ |
| **Controller** | 요청을 받고 `Service`를 호출하여 응답 반환 |
| **Service**    | 비즈니스 로직 수행 및 데이터베이스 조회    |

✅ **Controller → "사용자 요청 받기"**  
✅ **Service → "실제 데이터 처리"**

🔥 이 원리만 이해하면 NestJS API 설계가 쉬워집니다! 🚀

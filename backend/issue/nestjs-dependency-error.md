# 🚀 NestJS `UnknownDependenciesException` 에러 해결 가이드

## ✅ `UnknownDependenciesException`이란?

NestJS에서 특정 의존성을 해결할 수 없을 때 발생하는 에러로, 보통 **모듈, 서비스, 리포지토리(Entity), 글로벌 서비스(JWT, Config 등)** 가 제대로 등록되지 않았을 때 나타난다.

---

## 🔥 `UnknownDependenciesException` 체크리스트

### **📌 1. `@InjectRepository(Entity)` 사용 시 `TypeOrmModule.forFeature([Entity])` 등록 확인**

**⚠️ 잘못된 예시 (에러 발생 가능)**

```typescript
@Injectable()
export class AuthAdminService {
  constructor(
    private readonly adminsRepository: Repository<Admin> // ❌ AdminRepository가 주입되지 않음
  ) {}
}
```

**✅ 올바른 예시 (해결 방법)**
📂 **`src/admins/admins.module.ts`**

```typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([Admin]), // ✅ TypeORM 모듈에서 Admin 엔티티를 추가해야 함
  ],
  providers: [AdminsService],
  exports: [AdminsService, TypeOrmModule], // ✅ TypeOrmModule을 export해야 다른 모듈에서 사용 가능
})
export class AdminsModule {}
```

📌 **`@InjectRepository(Entity)`를 사용할 경우, `TypeOrmModule.forFeature([Entity])`를 반드시 등록해야 함**

---

### **📌 2. 특정 모듈을 `imports`에서 가져오지 않았는지 확인**

**⚠️ 잘못된 예시 (에러 발생 가능)**

```typescript
@Module({
  providers: [AuthAdminService], // ❌ AdminsModule을 import하지 않음
})
export class AuthAdminModule {}
```

**✅ 올바른 예시 (해결 방법)**

```typescript
@Module({
  imports: [AdminsModule], // ✅ 반드시 AdminsModule을 import해야 함
  providers: [AuthAdminService],
})
export class AuthAdminModule {}
```

📌 **`AdminsModule`이 `AuthAdminModule`에서 `imports`되지 않으면 `AdminsService`를 찾을 수 없음**

---

### **📌 3. `exports`에서 필요한 서비스를 내보냈는지 확인**

**⚠️ 잘못된 예시 (에러 발생 가능)**

```typescript
@Module({
  providers: [AdminsService], // ❌ exports에 추가하지 않음
})
export class AdminsModule {}
```

**✅ 올바른 예시 (해결 방법)**

```typescript
@Module({
  providers: [AdminsService],
  exports: [AdminsService], // ✅ AdminsService를 exports해야 다른 모듈에서 사용 가능
})
export class AdminsModule {}
```

📌 **`exports`를 하지 않으면 다른 모듈에서 `AdminsService`를 사용할 수 없어서 `UnknownDependenciesException` 발생**

---

### **📌 4. `JwtService` 같은 글로벌 서비스(`JwtModule`, `ConfigModule`)를 `imports`에서 가져왔는지 확인**

**⚠️ 잘못된 예시 (에러 발생 가능)**

```typescript
@Module({
  providers: [AuthAdminService], // ❌ JwtService를 사용할 수 없음
})
export class AuthAdminModule {}
```

**✅ 올바른 예시 (해결 방법)**

```typescript
@Module({
  imports: [
    JwtModule.register({
      secret: process.env.JWT_SECRET_ADMIN, // ✅ JWT 모듈 추가
      signOptions: { expiresIn: "24h" },
    }),
  ],
  providers: [AuthAdminService],
})
export class AuthAdminModule {}
```

📌 **`JwtService`를 사용하려면 `JwtModule`을 `imports`에서 등록해야 함**

---

### **📌 5. `@Inject()` 또는 `@InjectRepository()`를 올바르게 사용했는지 확인**

**⚠️ 잘못된 예시 (에러 발생 가능)**

```typescript
@Injectable()
export class AuthAdminService {
  constructor(private readonly adminsRepository: Repository<Admin>) {} // ❌ `@InjectRepository()` 빠짐
}
```

**✅ 올바른 예시 (해결 방법)**

```typescript
@Injectable()
export class AuthAdminService {
  constructor(
    @InjectRepository(Admin)
    private readonly adminsRepository: Repository<Admin> // ✅ `@InjectRepository()` 추가
  ) {}
}
```

📌 **TypeORM의 `Repository<Entity>`를 사용할 때는 `@InjectRepository(Entity)`를 반드시 추가해야 함**

---

## 🎯 **🔥 최종 체크리스트 (이걸 확인하면 `UnknownDependenciesException`을 예방할 수 있음!)**

✅ **`@InjectRepository(Entity)`를 사용하는 서비스가 있는 경우 → `TypeOrmModule.forFeature([Entity])`을 모듈에 추가했는지 확인**  
✅ **서비스가 여러 모듈에 걸쳐 있는 경우 → 해당 모듈이 `imports`로 추가되었는지 확인**  
✅ **다른 모듈에서 사용하려는 서비스가 `exports`되었는지 확인**  
✅ **글로벌 서비스 (`JwtService`, `ConfigService` 등)를 사용하려는 경우 → `JwtModule.register()` 또는 `ConfigModule.forRoot()`를 `imports`에서 추가했는지 확인**  
✅ **NestJS에서 제공하는 `@Inject()` 또는 `@InjectRepository()`를 올바르게 사용했는지 확인**

🚀 **이제 이 체크리스트를 확인하면 `UnknownDependenciesException`이 자주 발생하는 문제를 줄일 수 있음!**

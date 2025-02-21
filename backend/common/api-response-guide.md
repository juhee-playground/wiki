# API Response 구조 통일 가이드

## ✅ 공통 응답 구조

모든 API 응답을 통일된 구조로 반환하여 일관성을 유지하고, 유지보수성을 높인다.

### 1. 성공 응답 구조

```typescript
export interface ISuccessResponse<T> {
  success: true;
  data: T;
}
```

**예시:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "User One",
    "email": "user1@example.com",
    "role": "owner"
  }
}
```

### 2. 실패 응답 구조

```typescript
export interface IErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
  };
}
```

**예시:**

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "사용자를 찾을 수 없습니다."
  }
}
```

## ✅ 응답 함수 정의

반복적인 응답 처리를 줄이기 위해 `utils/api-response.ts`에 공통 응답 함수를 정의한다.

```typescript
export function successResponse<T>(data: T): ISuccessResponse<T> {
  return {
    success: true,
    data,
  };
}

export function errorResponse(code: string, message: string): IErrorResponse {
  return {
    success: false,
    error: {
      code,
      message,
    },
  };
}
```

## ✅ 적용 예시

### 1. 서비스 로직에서의 응답 처리

```typescript
async findOneById(id: number): Promise<ISuccessResponse<User> | IErrorResponse> {
  const user = await this.usersRepository.findOne({ where: { id } });
  if (!user) {
    return errorResponse('USER_NOT_FOUND', '사용자를 찾을 수 없습니다.');
  }
  return successResponse(user);
}
```

### 2. 컨트롤러에서의 응답 처리

```typescript
@Get(':id')
async getUser(@Param('id') id: number) {
  return this.usersService.findOneById(id);
}
```

## ✅ 정리

- **모든 응답은 `successResponse()`와 `errorResponse()` 사용하여 일관성 유지**
- **성공 시 `success: true, data` 반환**
- **실패 시 `success: false, error: { code, message }` 반환**
- **프론트엔드에서는 `error.code`를 기반으로 메시지 처리 가능**

## 📌 코드 스타일 가이드

###  네이밍 컨벤션
- **모듈 이름**: `PascalCase` (`UsersModule`, `StoresModule`)
- **서비스 이름**: `Service` 접미사 (`UsersService`, `StoresService`)
- **컨트롤러 이름**: `Controller` 접미사 (`UsersController`, `StoresController`)
- **DTO 이름**: `Create<ModuleName>Dto`, `Update<ModuleName>Dto` 형식 (`CreateUserDto`, `UpdateStoreDto`)
- **엔티티 이름**: `PascalCase` (`User`, `Store`)

### 코드 품질
- **2칸 들여쓰기** 사용
- **세미콜론 필수**
- **ESLint**와 **Prettier**를 사용하여 일관성 있는 코드 스타일 유지

# 🌐 Google OAuth 로그인 연동 가이드

이 문서는 서비스에 Google 로그인을 연동하기 위한 전체 절차를 정리한 것입니다.  
OAuth2 Authorization Code Flow를 기반으로 하며, 백엔드에서 `code`를 받아 인증을 처리하는 방식입니다.

---

## ✅ 1. Google Client ID 발급 절차

### 1. Google Cloud Console 접속
- [https://console.cloud.google.com/](https://console.cloud.google.com/) 접속
- Google 계정으로 로그인

### 2. 프로젝트 생성
- 상단 프로젝트 선택 → **새 프로젝트 만들기**
- 예: `XXX Auth`
- 만든 후 해당 프로젝트 선택

### 3. OAuth 동의 화면 설정
- 사이드바 메뉴 → **API 및 서비스 > OAuth 동의 화면**
- 사용자 유형: **외부**
- 앱 이름, 지원 이메일, 도메인 정보 입력
- 승인된 도메인:
  - `localhost`
  - `XXX.com`
  - `www.XXX.com`

> ⚠️ 도메인은 반드시 실제 사용될 호스트명을 포함해야 하며, 전체 URL이 아닌 도메인만 기입합니다.

### 4. OAuth 클라이언트 ID 생성
- 사이드바 → **사용자 인증 정보**
- `+ 사용자 인증 정보 만들기` → **OAuth 클라이언트 ID**
- 애플리케이션 유형: **웹 애플리케이션**
- 승인된 리디렉션 URI:
  - `http://localhost:3000/auth/callback`
  - `https://XXXX.com/auth/callback`
  - `https://www.XXXX.com/auth/callback`

> 💡 리디렉션 URI는 Google이 로그인 후 유저를 리디렉션할 주소입니다.

---

## ✅ 2. 리디렉션 URI 처리 방법

- `/auth/callback`은 로그인 완료 후 Google이 리디렉션하는 경로입니다.
- 실제 페이지가 존재해야 하며, React Router 등에서 해당 라우트를 지정해줘야 합니다.

```tsx
<Route path="/auth/callback" element={<GoogleAuthCallbackPage />} />
```

- 리디렉션된 URL 예시:
  ```
  http://localhost:3000/auth/callback?code=abcd1234
  ```

- 해당 `code`를 파싱해서 백엔드에 넘기는 방식으로 로그인 처리를 완료합니다.

---

## ✅ 3. Client ID 보안 관련 FAQ

### 🔒 Client ID는 숨겨야 하나요?
- **아니요.** `client_id`는 공개되어도 괜찮습니다.
- React 코드에 직접 쓰거나 `.env`에 두고 번들해도 보안에 문제 없습니다.

| 항목            | 공개 가능 여부 | 설명                                   |
|-----------------|----------------|----------------------------------------|
| `client_id`     | ✅ 공개 가능    | 식별자 역할, 공개되어도 안전함         |
| `client_secret` | ❌ 비공개       | 백엔드에서만 사용해야 함               |
| `access_token`  | ❌ 비공개       | 사용자 인증 정보, 노출 시 위험          |
| `refresh_token` | ❌ 비공개       | 장기 인증 토큰, 탈취 시 자동 로그인 가능 |

---

## ✅ 4. 환경 변수 구성 예시

```env
# .env.local
VITE_GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
VITE_GOOGLE_REDIRECT_URI=http://localhost:3000/auth/callback
```

```tsx
// index.tsx
import { GoogleOAuthProvider } from '@react-oauth/google';

<GoogleOAuthProvider clientId={import.meta.env.VITE_GOOGLE_CLIENT_ID}>
  <App />
</GoogleOAuthProvider>
```

---

## 🧩 이후 작업

- `/auth/callback` 페이지에서 `code` 파라미터 수신 및 백엔드 요청
- 백엔드 API 예시:
  ```http
  POST /api/auth/google
  {
    "code": "abcd1234"
  }
  ```
- 백엔드는 Google OAuth 서버에 `code` → `access_token` 교환 후 사용자 인증 처리

---

## 📌 참고

- Google OAuth 공식 문서: https://developers.google.com/identity/protocols/oauth2
- `@react-oauth/google` 라이브러리: https://www.npmjs.com/package/@react-oauth/google

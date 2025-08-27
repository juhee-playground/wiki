# 일반적인 초대 시스템 구현 가이드 (코드 포함)

## 📋 개요

이 문서는 웹 애플리케이션에서 초대 시스템을 구현하는 일반적인 방법을 설명합니다. 조직이나 팀 기반 서비스에서 새로운 사용자를 초대하고 온보딩하는 완전한 플로우를 다룹니다.

## 🏗️ 아키텍처

### 파일 구조

```
src/
├── pages/
│   ├── Invite.tsx                    # 초대 페이지
│   ├── Onboarding.tsx                # 온보딩 페이지
│   ├── SignIn.tsx                    # 로그인 페이지 (초대 토큰 처리)
│   └── TestInvite.tsx                # 테스트용 초대 링크 생성 페이지
├── domains/onboarding/
│   ├── container/
│   │   ├── InviteContainer.tsx       # 초대 컨테이너
│   │   └── OnboardingContainer.tsx   # 온보딩 컨테이너
│   ├── hooks/
│   │   ├── useInviteFlow.ts          # 초대 플로우 로직
│   │   └── useOnboardingFlow.ts      # 온보딩 플로우 로직
│   ├── view/
│   │   ├── InviteView.tsx            # 초대 UI
│   │   └── OnboardingView.tsx        # 온보딩 UI
│   └── components/
│       └── steps/
│           ├── InvitationStep.tsx    # 초대 단계
│           ├── UsageSelectionStep.tsx # 사용 목적 선택
│           ├── OrganizationNameStep.tsx # 조직명 입력
│           └── SuccessStep.tsx       # 성공 단계
├── api/
│   └── organization.ts               # 초대 관련 API
├── hooks/permissions/
│   └── usePermissions.ts             # 초대 관련 훅
├── utils/
│   └── onboarding.ts                 # 온보딩 상태 체크
└── stores/
    └── useOnboardingStore.ts         # 온보딩 상태 관리
```

## 🔄 플로우 개요

### 1. 초대 플로우

```
초대 링크 생성 → 링크 공유 → 초대받은 사용자 접근 → 로그인 → 초대 수락 → 조직 가입
```

### 2. 온보딩 플로우

```
로그인 → 온보딩 상태 체크 → 사용 목적 선택 → 조직 생성/개인 설정 → 완료
```

## 🎯 초대 시스템

### 1. 초대 링크 생성

#### API 엔드포인트

```typescript
// src/api/organization.ts
export const generateInviteLink = async (organizationId: string) => {
  const response = await requestAPI.post(`/api/v2/organizations/${organizationId}/invite-link`);
  return response.data;
};
```

#### 응답 데이터

```typescript
interface InviteLinkResponse {
  organization_id: string;
  invite_url: string;
  token: string;
  role: 'admin' | 'member' | 'viewer';
  expires_at: string;
  is_active: boolean;
}
```

#### 사용 방법

```typescript
// src/hooks/permissions/usePermissions.ts
export const useGenerateInviteLink = () => {
  return useMutation({
    mutationFn: generateInviteLink,
    onSuccess: data => {
      // 초대 링크 생성 성공 처리
      console.log('Invite link generated:', data.invite_url);
    },
    onError: error => {
      // 에러 처리
      console.error('Failed to generate invite link:', error);
    },
  });
};
```

### 2. 초대 링크 접근 플로우

```typescript
// src/domains/onboarding/hooks/useInviteFlow.ts
export const useInviteFlow = () => {
  const { token } = useParams<{ token: string }>();
  const navigate = useNavigate();
  const { isAuthenticated, isInitialized } = useAuthStore();

  const { data: inviteData, isLoading, error } = useInvitePreview(token);
  const acceptInvite = useAcceptInvite();

  useEffect(() => {
    if (!isInitialized) return;

    // 로그인되지 않은 경우, 로그인 페이지로 리다이렉트
    if (!isAuthenticated) {
      const loginUrl = `/signin?invite=${token}`;
      navigate(loginUrl, { replace: true });
      return;
    }
  }, [isAuthenticated, isInitialized, token, navigate]);

  return {
    inviteData,
    isLoading,
    error,
    acceptInvite,
  };
};
```

### 3. 로그인 플로우

```typescript
// src/pages/SignIn.tsx
const SignInPage = () => {
  const [searchParams] = useSearchParams();
  const { isAuthenticated } = useAuthStore();
  const { setRedirectPath } = useAuthStore();

  useEffect(() => {
    // 초대 토큰이 있으면 저장
    const inviteToken = searchParams.get('invite');
    if (inviteToken) {
      setRedirectPath(`/invite/${inviteToken}`);
    }
  }, [searchParams, setRedirectPath]);

  useEffect(() => {
    if (isAuthenticated) {
      const inviteToken = searchParams.get('invite');
      const redirect = redirectPath || searchParams.get('redirect') || '/dashboard';

      // 초대 토큰이 있으면 초대 페이지로 리다이렉트
      if (inviteToken) {
        window.location.href = `/invite/${inviteToken}`;
      } else {
        setRedirectPath(null);
        window.location.href = redirect;
      }
    }
  }, [isAuthenticated, redirectPath, searchParams, setRedirectPath]);

  // 로그인 UI 렌더링
};
```

### 4. 초대 정보 미리보기

```typescript
// src/api/organization.ts
export const getInvitePreview = async (token: string) => {
  const response = await requestAPI.get(`/api/v2/organizations/invites/${token}/preview`);
  return response.data;
};

// src/hooks/permissions/usePermissions.ts
export const useInvitePreview = (token: string) => {
  return useQuery({
    queryKey: ['invite-preview', token],
    queryFn: () => getInvitePreview(token),
    enabled: !!token,
  });
};
```

### 5. 초대 수락

```typescript
// src/api/organization.ts
export const acceptInvite = async (token: string) => {
  const response = await requestAPI.post(`/api/v2/organizations/invites/${token}/accept`);
  return response.data;
};

// src/hooks/permissions/usePermissions.ts
export const useAcceptInvite = () => {
  const navigate = useNavigate();

  return useMutation({
    mutationFn: acceptInvite,
    onSuccess: () => {
      // 초대 수락 성공 시 메인 페이지로 이동
      navigate('/dashboard', { replace: true });
    },
    onError: error => {
      console.error('Failed to accept invite:', error);
    },
  });
};
```

## 🎯 온보딩 시스템

### 1. 온보딩 완료 조건

```typescript
// src/utils/onboarding.ts
export const useOnboardingStatus = () => {
  const { user } = useUserStore();
  const { workspaces } = useWorkspaceStore();

  const isOnboardingCompleted = () => {
    if (!user) return false;

    // 조직 사용자: organization_id가 있으면 완료
    if (user.organization_id !== undefined && user.organization_id !== null) {
      return true;
    }

    // 개인 사용자: workspaces가 있으면 완료
    if (workspaces && workspaces.length > 0) {
      return true;
    }

    return false;
  };

  return { isOnboardingCompleted: isOnboardingCompleted() };
};
```

### 2. 온보딩 플로우

```typescript
// src/domains/onboarding/hooks/useOnboardingFlow.ts
export const useOnboardingFlow = () => {
  const [currentStep, setCurrentStep] = useState(ONBOARDING_STEPS.USAGE_SELECTION);
  const [searchParams] = useSearchParams();
  const inviteToken = searchParams.get('invite');

  const createOrganization = useCreateOrganization();
  const createWorkspace = useCreateWorkspace();

  useEffect(() => {
    // 초대 토큰이 있으면 초대 단계로 설정
    if (inviteToken) {
      setCurrentStep(ONBOARDING_STEPS.INVITATION);
    }
  }, [inviteToken]);

  const handleUsageSelectionComplete = (usageType: 'organization' | 'personal') => {
    if (usageType === 'organization') {
      setCurrentStep(ONBOARDING_STEPS.ORGANIZATION_NAME);
    } else {
      // 개인 사용자: 워크스페이스 자동 생성
      createWorkspace(
        {
          title: 'My Personal Workspace',
          organization_id: null,
        },
        {
          onSuccess: () => {
            setCurrentStep(ONBOARDING_STEPS.SUCCESS);
          },
          onError: () => {
            console.error('Failed to create personal workspace');
          },
        },
      );
    }
  };

  const handleOrganizationNameComplete = (organizationName: string) => {
    createOrganization(
      { name: organizationName },
      {
        onSuccess: () => {
          setCurrentStep(ONBOARDING_STEPS.SUCCESS);
        },
        onError: () => {
          console.error('Failed to create organization');
        },
      },
    );
  };

  return {
    currentStep,
    handleUsageSelectionComplete,
    handleOrganizationNameComplete,
    isLoading: createOrganization.isPending || createWorkspace.isPending,
  };
};
```

### 3. 온보딩 가드

```typescript
// src/components/auth/OnboardingGuard.tsx
export const OnboardingGuard = ({ children }: { children: React.ReactNode }) => {
  const navigate = useNavigate();
  const location = useLocation();
  const { isAuthenticated, isInitialized } = useAuthStore();
  const { isOnboardingCompleted } = useOnboardingStatus();

  const shouldSkipGuard = location.pathname === '/onboarding';

  useEffect(() => {
    if (shouldSkipGuard) return;
    if (!isInitialized || !isAuthenticated) return;

    // 온보딩이 완료되지 않았고, 현재 온보딩 페이지가 아닌 경우
    if (!isOnboardingCompleted && location.pathname !== '/onboarding') {
      navigate('/onboarding', { replace: true });
    }
  }, [isOnboardingCompleted, isInitialized, isAuthenticated, location.pathname, navigate, shouldSkipGuard]);

  return <>{children}</>;
};
```

## 🧪 테스트 방법

### 1. 초대 링크 생성 테스트

```typescript
// src/pages/TestInvite.tsx
const TestInvitePage = () => {
  const { user } = useUserStore();
  const generateInviteLink = useGenerateInviteLink();

  const handleGenerateLink = () => {
    if (user?.organization_id) {
      generateInviteLink.mutate(user.organization_id);
    }
  };

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-4">Test Invite Link Generation</h1>
      <button
        onClick={handleGenerateLink}
        disabled={generateInviteLink.isPending}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        {generateInviteLink.isPending ? 'Generating...' : 'Generate Invite Link'}
      </button>

      {generateInviteLink.data && (
        <div className="mt-4">
          <p>Generated Link:</p>
          <input
            type="text"
            value={generateInviteLink.data.invite_url}
            readOnly
            className="w-full p-2 border rounded"
          />
        </div>
      )}
    </div>
  );
};
```

### 2. 비로그인 사용자 초대 테스트

```typescript
// src/domains/onboarding/view/InviteView.tsx
const InviteView = ({ inviteData, isLoading, error, isInitialized, isAuthenticated }) => {
  if (isLoading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-white"></div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="text-center">
          <h1 className="text-2xl font-bold text-red-500 mb-4">Invalid Invitation</h1>
          <p className="text-gray-600">This invitation link is invalid or has expired.</p>
        </div>
      </div>
    );
  }

  if (isInitialized && !isAuthenticated) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-purple-600 to-blue-600">
        <div className="bg-gray-800 rounded-lg p-8 max-w-md w-full mx-4">
          <div className="text-center">
            <div className="flex justify-center mb-4">
              <HiEnvelope className="w-9 h-9 text-gray-400" />
            </div>
            <h1 className="text-3xl font-bold text-white mb-6">You've Been Invited</h1>
            <p className="text-base text-gray-400 mb-8">
              Please sign in to view and accept your invitation.
            </p>
            <div className="flex justify-center">
              <button
                className="bg-blue-500 text-white hover:bg-blue-600 px-8 py-3 rounded-lg font-semibold"
                onClick={() => (window.location.href = '/signin')}
              >
                Sign In to Continue
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-purple-600 to-blue-600">
      <div className="bg-gray-800 rounded-lg p-8 max-w-md w-full mx-4">
        <div className="text-center">
          <div className="flex justify-center mb-4">
            <HiEnvelope className="w-9 h-9 text-blue-400" />
          </div>
          <h1 className="text-3xl font-bold text-white mb-6">You're Invited!</h1>
          <p className="text-sm text-gray-400 mb-3">
            invited by {inviteData?.inviter_email || 'team@example.com'}
          </p>
          <p className="text-base text-white font-light mb-6">
            {inviteData?.permission_level || 'Member'}
          </p>
          <div className="space-y-3">
            <button
              className="w-full bg-green-500 text-white hover:bg-green-600 py-3 rounded-lg font-semibold"
              onClick={handleAcceptInvite}
              disabled={isAccepting}
            >
              {isAccepting ? 'Accepting...' : 'Accept Invitation'}
            </button>
            <button
              className="w-full bg-red-500 text-white hover:bg-red-600 py-3 rounded-lg font-semibold"
              onClick={handleDeclineInvite}
            >
              Decline
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};
```

## 🎨 UI/UX 특징

### 디자인 시스템

```css
/* src/styles/components.css */
.invite-card {
  @apply bg-gray-800 rounded-lg p-8 max-w-md w-full mx-4;
}

.invite-button {
  @apply w-full py-3 rounded-lg font-semibold transition-colors;
}

.accept-button {
  @apply bg-green-500 text-white hover:bg-green-600;
}

.decline-button {
  @apply bg-red-500 text-white hover:bg-red-600;
}

.loading-spinner {
  @apply animate-spin rounded-full h-32 w-32 border-b-2 border-white;
}
```

### 반응형 디자인

```typescript
// 반응형 레이아웃 예시
const ResponsiveLayout = ({ children }) => {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-purple-600 to-blue-600 p-4">
      <div className="w-full max-w-md">
        {children}
      </div>
    </div>
  );
};
```

## 🔧 기술적 구현

### 상태 관리

```typescript
// src/stores/useOnboardingStore.ts
interface OnboardingState {
  currentStep: string;
  organizationName: string;
  usageType: 'organization' | 'personal' | null;
  isCompleted: boolean;
}

export const useOnboardingStore = create<OnboardingState>(set => ({
  currentStep: 'usage-selection',
  organizationName: '',
  usageType: null,
  isCompleted: false,

  setCurrentStep: (step: string) => set({ currentStep: step }),
  setOrganizationName: (name: string) => set({ organizationName: name }),
  setUsageType: (type: 'organization' | 'personal') => set({ usageType: type }),
  setCompleted: (completed: boolean) => set({ isCompleted: completed }),
}));
```

### 에러 처리

```typescript
// src/utils/errorHandling.ts
export const handleInviteError = (error: any) => {
  if (error.response?.status === 404) {
    return 'This invitation link is invalid or has expired.';
  }

  if (error.response?.status === 403) {
    return 'You do not have permission to accept this invitation.';
  }

  return 'An error occurred while processing your invitation.';
};
```

### 라우팅

```typescript
// src/router/Router.tsx
const Router = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/invite/:token" element={
          <OnboardingGuard>
            <InvitePage />
          </OnboardingGuard>
        } />
        <Route path="/onboarding" element={
          <OnboardingGuard>
            <OnboardingPage />
          </OnboardingGuard>
        } />
        <Route path="/signin" element={<SignInPage />} />
        <Route path="/test-invite" element={
          <AuthGuard>
            <TestInvitePage />
          </AuthGuard>
        } />
        {/* 기타 라우트들 */}
      </Routes>
    </BrowserRouter>
  );
};
```

## 📱 사용 시나리오

### 시나리오 A: 기존 사용자가 새 사용자 초대

```typescript
// 관리자 페이지에서 초대 링크 생성
const AdminDashboard = () => {
  const generateInviteLink = useGenerateInviteLink();

  const handleInviteUser = () => {
    generateInviteLink.mutate(organizationId, {
      onSuccess: (data) => {
        // 링크를 클립보드에 복사
        navigator.clipboard.writeText(data.invite_url);
        alert('Invite link copied to clipboard!');
      },
    });
  };

  return (
    <div>
      <button onClick={handleInviteUser}>
        Invite New User
      </button>
    </div>
  );
};
```

### 시나리오 B: 완전 새로운 사용자 (조직 생성)

```typescript
// 온보딩 플로우에서 조직 생성
const OrganizationNameStep = ({ onComplete, isLoading }) => {
  const [organizationName, setOrganizationName] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (organizationName.trim()) {
      onComplete(organizationName.trim());
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={organizationName}
        onChange={(e) => setOrganizationName(e.target.value)}
        placeholder="Enter organization name"
        disabled={isLoading}
      />
      <button type="submit" disabled={isLoading || !organizationName.trim()}>
        {isLoading ? 'Creating...' : 'Create Organization'}
      </button>
    </form>
  );
};
```

## 🐛 주의사항

### 현재 제한사항

- 초대 링크는 일정 시간 후 만료됨
- 한 사용자는 한 조직에만 가입 가능
- 초대 수락 후 즉시 조직 멤버가 됨
- 온보딩 완료 상태는 로컬 스토리지에 저장됨

### 향후 개선사항

```typescript
// 향후 구현할 기능들
interface EnhancedInviteSystem {
  // 초대 링크 만료 시간 설정 가능
  setExpirationTime: (hours: number) => void;

  // 초대 링크 재생성 기능
  regenerateInviteLink: (token: string) => void;

  // 초대 수락 전 확인 단계 추가
  requireConfirmation: boolean;

  // 초대 이메일 발송 기능
  sendInviteEmail: (email: string, token: string) => void;

  // 온보딩 완료 상태 서버 동기화
  syncOnboardingStatus: () => void;
}
```

## 📋 체크리스트

### 초대 시스템

- [ ] 초대 링크 생성 기능
- [ ] 초대 정보 미리보기
- [ ] 로그인 상태 체크
- [ ] 초대 토큰 처리
- [ ] 초대 수락/거절 기능
- [ ] 에러 처리

### 온보딩 시스템

- [ ] 온보딩 완료 조건 체크
- [ ] 사용 목적 선택
- [ ] 조직 생성 기능
- [ ] 개인 사용자 워크스페이스 생성
- [ ] 온보딩 가드
- [ ] 자동 리다이렉트

### 테스트

- [ ] 초대 링크 생성 테스트
- [ ] 비로그인 사용자 초대 테스트
- [ ] 로그인된 사용자 초대 테스트
- [ ] 완전 최초 사용자 온보딩 테스트
- [ ] 개인 사용자 온보딩 테스트

## 🔧 커스터마이징 가이드

### 브랜드 적용

```typescript
// 1. 색상 변경: CSS 변수나 테마 시스템 사용
const theme = {
  colors: {
    primary: '#your-brand-color',
    secondary: '#your-secondary-color',
    accent: '#your-accent-color',
  },
};

// 2. 로고 교체: 로고 이미지 파일 교체
const Logo = () => (
  <img src="/your-logo.png" alt="Your Brand" className="h-8" />
);

// 3. 폰트 변경: 웹폰트 설정 변경
const fontFamily = {
  sans: ['Your Font', 'sans-serif'],
};

// 4. 아이콘 변경: 아이콘 라이브러리 교체
import { YourIcon } from 'your-icon-library';
```

### 기능 확장

```typescript
// 1. 다중 조직 지원: 사용자가 여러 조직에 가입 가능
interface MultiOrganizationUser {
  organizations: Organization[];
  currentOrganization: Organization;
  switchOrganization: (orgId: string) => void;
}

// 2. 역할 기반 권한: 세분화된 권한 시스템
interface RoleBasedPermissions {
  role: 'admin' | 'manager' | 'member' | 'viewer';
  permissions: Permission[];
  canInvite: boolean;
  canManageMembers: boolean;
  canEditContent: boolean;
}

// 3. 초대 이메일: 자동 이메일 발송 기능
const sendInviteEmail = async (email: string, token: string) => {
  await fetch('/api/send-invite-email', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, token }),
  });
};

// 4. 초대 관리: 초대 목록 및 상태 관리
interface InviteManagement {
  pendingInvites: Invite[];
  acceptedInvites: Invite[];
  expiredInvites: Invite[];
  cancelInvite: (inviteId: string) => void;
  resendInvite: (inviteId: string) => void;
}
```

### API 커스터마이징

```typescript
// 1. 엔드포인트 변경: 백엔드 API 구조에 맞게 수정
const API_ENDPOINTS = {
  generateInvite: '/api/teams/{teamId}/invites',
  previewInvite: '/api/invites/{token}/preview',
  acceptInvite: '/api/invites/{token}/accept',
  createOrganization: '/api/organizations',
  createWorkspace: '/api/workspaces',
};

// 2. 인증 방식: JWT, OAuth 등 다양한 인증 방식 지원
const authConfig = {
  type: 'jwt', // 'oauth', 'session'
  tokenKey: 'access_token',
  refreshTokenKey: 'refresh_token',
  tokenExpiry: 3600, // seconds
};

// 3. 데이터 구조: 응답 데이터 구조에 맞게 타입 정의 수정
interface CustomInviteResponse {
  id: string;
  team: {
    id: string;
    name: string;
    description?: string;
  };
  inviter: {
    id: string;
    name: string;
    email: string;
  };
  role: 'admin' | 'member' | 'viewer';
  expiresAt: string;
  status: 'pending' | 'accepted' | 'expired';
}
```

---

**작성일**: 2024년 12월  
**버전**: 1.0.0  
**작성자**: Frontend Development Team

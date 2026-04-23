# Amplitude 설정 가이드

## 1단계: Amplitude에 이벤트 정의하기

`DA.md`에 따르면, Amplitude 프로젝트에 다음 이벤트들을 정의해야 합니다.

- `landing_enter`
- `intro_step_complete`
- `enter_character_select`
- `character_select`
- `user_info_submit`
- `result_received`
- `payment_click`
- `payment_complete`
- `paid_version_enter`
- `paid_version_complete_read`
- `share_click`
- `unselected_character_shown`
- `unselected_character_click`

## 2단계: 이벤트 트래킹 구현하기

`DA.md` 문서에 기반한 각 이벤트 트래킹 구현 방법입니다.

### 1. `landing_enter`
- **시점:** 메인 페이지 컴포넌트가 마운트될 때.
- **위치:** 메인 페이지 컴포넌트의 `useEffect`.
- **목적:** 서비스 진입을 트래킹하고 유입 채널을 분석.
- **코드:**
  ```typescript
  useEffect(() => {
    const params = new URLSearchParams(window.location.search);
    trackEvent('landing_enter', {
      utm_source: params.get('utm_source'),
      utm_medium: params.get('utm_medium'),
      utm_campaign: params.get('utm_campaign'),
      referral_code: params.get('ref'),
    });
  }, []);
  ```

### 2. `intro_step_complete`
- **시점:** 사용자가 다음 인트로 단계로 넘어가기 위해 클릭할 때.
- **위치:** 인트로 시퀀스 버튼 클릭 핸들러.
- **목적:** 단계별 CTR과 이탈률을 계산.
- **코드:**
  ```typescript
  const handleNextStep = (currentStep: number, totalSteps: number) => {
    trackEvent('intro_step_complete', {
      step_index: currentStep,
      step_total: totalSteps,
    });
    // ... 다음 단계로 진행
  };
  ```

### 3. `enter_character_select`
- **시점:** 캐릭터 선택 화면 컴포넌트가 마운트될 때.
- **위치:** 캐릭터 선택 페이지 컴포넌트의 `useEffect`.
- **목적:** 캐릭터 선택에 소요되는 시간 측정을 시작.
- **코드:**
  ```typescript
  useEffect(() => {
    trackEvent('enter_character_select', {
      timestamp_enter: Date.now(),
    });
  }, []);
  ```

### 4. `character_select`
- **시점:** 사용자가 캐릭터를 선택할 때.
- **위치:** 캐릭터 선택 버튼 클릭 핸들러.
- **목적:** 선택 소요 시간 측정을 완료하고, 캐릭터와 결제 간의 상관관계를 분석.
- **코드:**
  ```typescript
  const handleCharacterSelect = (characterId: 'A' | 'B') => {
    trackEvent('character_select', {
      character_id: characterId,
      timestamp_select: Date.now(),
    });
    // ... 선택 처리
  };
  ```

### 5. `user_info_submit`
- **시점:** 사용자가 결과를 생성하기 위해 정보를 제출할 때.
- **위치:** 정보 폼 제출 핸들러.
- **목적:** 사용자 흐름 진행을 트래킹. **개인 식별 정보(PII)를 포함하지 마세요.**
- **코드:**
  ```typescript
  const handleSubmit = (characterId: 'A' | 'B') => {
    trackEvent('user_info_submit', {
      character_id: characterId,
    });
    // ... 결과 제출
  };
  ```

### 6. `result_received`
- **시점:** 결과 페이지 컴포넌트가 마운트될 때 (데이터 로드 후).
- **위치:** 결과 페이지 컴포넌트의 `useEffect`.
- **목적:** 유료 전환율 계산의 분모.
- **코드:**
  ```typescript
  useEffect(() => {
    if (resultLoaded) {
      trackEvent('result_received', {
        character_id: characterId,
        result_type: isPaid ? 'paid' : 'free',
      });
    }
  }, [resultLoaded]);
  ```

### 7. `payment_click`
- **시점:** 사용자가 결제 버튼을 클릭할 때.
- **위치:** 결제 버튼 클릭 핸들러.
- **목적:** 결제 의도를 측정.
- **코드:**
  ```typescript
  const handlePaymentClick = (characterId: 'A' | 'B') => {
    trackEvent('payment_click', {
      character_id: characterId,
    });
    // ... 결제 플로우 진입
  };
  ```

### 8. `payment_complete`
- **시점:** PayApp 피드백 URL 콜백을 받은 후 (`state=1`).
- **위치:** 백엔드 API 라우트 (`/api/payment/callback/route.ts`). **서버 사이드 전용.**
- **목적:** 유료 전환율의 분자.
- **코드 (백엔드):**
  ```typescript
  // app/api/payment/callback/route.ts
  if (state === '1') {
    await fetch('https://api2.amplitude.com/2/httpapi', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        api_key: process.env.AMPLITUDE_API_KEY,
        events: [{
          event_type: 'payment_complete',
          user_id: internalUserId,
          event_properties: {
            character_id: characterId,
            payment_method: 'payapp',
            mul_no: mulNo,
          },
        }],
      }),
    });
  }
  ```

### 9. `paid_version_enter`
- **시점:** 유료 결과 페이지 컴포넌트가 마운트될 때.
- **위치:** 유료 결과 페이지의 `useEffect`.
- **목적:** 유료 콘텐츠 읽기 시간 측정을 시작.
- **코드:**
  ```typescript
  useEffect(() => {
    trackEvent('paid_version_enter', {
      character_id: characterId,
      timestamp_enter: Date.now(),
    });
  }, []);
  ```

### 10. `paid_version_complete_read`
- **시점:** 사용자가 콘텐츠 하단까지 스크롤할 때.
- **위치:** 스크롤 감지 로직 (예: IntersectionObserver).
- **목적:** 읽기 시간 측정을 완료.
- **코드:**
  ```typescript
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          trackEvent('paid_version_complete_read', {
            character_id: characterId,
            timestamp_complete: Date.now(),
          });
          observer.disconnect();
        }
      },
      { threshold: 1.0 }
    );
    // ... 하단 요소 관찰
  }, []);
  ```

### 11. `share_click`
- **시점:** 사용자가 공유 버튼을 클릭할 때.
- **위치:** 공유 버튼 클릭 핸들러.
- **목적:** 공유 CTR 및 바이럴 루프를 측정.
- **코드:**
  ```typescript
  const handleShareClick = (characterId: 'A' | 'B', referralCode: string) => {
    trackEvent('share_click', {
      character_id: characterId,
      referral_code: referralCode,
    });
    // ... 공유 액션 실행
  };
  ```

### 12. `unselected_character_shown`
- **시점:** 선택되지 않은 캐릭터의 UI가 뷰포트에 들어올 때.
- **위치:** 선택되지 않은 캐릭터 컴포넌트의 IntersectionObserver.
- **목적:** 선택되지 않은 캐릭터의 클릭률 계산을 위한 분모.
- **코드:**
  ```typescript
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          trackEvent('unselected_character_shown', {
            shown_character_id: unselectedCharacterId,
          });
          observer.disconnect();
        }
      },
      { threshold: 0.5 }
    );
    // ... 요소 관찰
  }, []);
  ```

### 13. `unselected_character_click`
- **시점:** 사용자가 선택되지 않은 캐릭터를 클릭할 때.
- **위치:** 선택되지 않은 캐릭터 버튼의 클릭 핸들러.
- **목적:** 선택되지 않은 캐릭터의 클릭률 계산을 위한 분자.
- **코드:**
  ```typescript
  const handleUnselectedCharacterClick = (unselectedCharacterId: 'A' | 'B') => {
    trackEvent('unselected_character_click', {
      shown_character_id: unselectedCharacterId,
    });
    // ... 해당 캐릭터 플로우 진입
  };
  ```

## 3단계: 대시보드 및 퍼널 설정하기

`hmda.md`에서 가져온 이 섹션은 이벤트 트래킹의 "이유"를 설명합니다. Amplitude 차트를 만들 때 이 정보를 사용하세요.

### [HMDA-1] 활성화 측정
- **목표:** 초기 사용자 흐름의 각 단계별 CTR 및 이탈률을 측정합니다.
- **이벤트:** `landing_enter`, `intro_step_complete`, `enter_character_select`, `character_select`
- **Amplitude 차트:** 퍼널 차트를 사용하여 흐름을 시각화하고 사용자가 어디에서 이탈하는지 식별합니다.

### [HMDA-2] 전환 측정
- **목표:** 획득 채널별로 분류된, 랜딩부터 결과까지의 사용자 여정을 이해합니다.
- **이벤트:** `landing_enter`, `character_select`, `result_received`
- **속성:** `utm_source`, `utm_medium`, `utm_campaign`
- **Amplitude 차트:** 사용자 경로 또는 여정 차트. UTM 속성으로 필터링된 퍼널 차트를 사용하여 채널을 비교합니다.

### [HMDA-3] 리텐션 / 시스템 측정
- **목표:** 유료 전환, 공유 및 다른 캐릭터와의 참여를 측정합니다.
- **이벤트:** `result_received`, `payment_click`, `payment_complete`, `share_click`, `unselected_character_shown`, `unselected_character_click`
- **Amplitude 차트:**
    - **퍼널:** 전환율을 위한 `result_received` -> `payment_complete`.
    - **세분화:** `character_id`별로 그룹화된 `payment_complete` 이벤트를 분석합니다.
    - **메트릭:** `result_received` 대비 `share_click` 비율, `unselected_character_shown` 대비 `unselected_character_click` 비율.

### [HMDA-4] 시간 측정
- **목표:** 사용자가 유료 콘텐츠를 읽는 데 걸리는 시간을 측정합니다.
- **이벤트:** `paid_version_enter`, `paid_version_complete_read`
- **Amplitude 차트:** 두 이벤트 간의 간격을 계산합니다. 히스토그램은 소요 시간 분포를 보는 데 유용합니다.

### [HMDA-5] 루프 측정
- **목표:** 바이럴 루프(공유)의 효과를 측정합니다.
- **이벤트:** `share_click`, `landing_enter` (`referral_code` 속성 포함)
- **Amplitude 차트:** `landing_enter`에서 `share_click`까지의 퍼널. `referral_code`가 있는 새로운 `landing_enter` 이벤트를 추적합니다.

### [HMDA-6] UTM 네이밍 규칙
- **목표:** 마케팅 캠페인에서 일관되고 깨끗한 데이터를 보장합니다.
- **조치:** `utm_source`, `utm_medium`, `utm_campaign`에 대한 엄격한 네이밍 규칙을 정의하고 시행합니다. 예를 들어, 공백 대신 소문자와 밑줄을 사용합니다. 이것은 구현 작업이 아니라 데이터 거버넌스 작업이지만 유용한 분석을 위해 중요합니다.

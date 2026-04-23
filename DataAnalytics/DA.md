# PHM Data Analytics — Claude Code Context

이 문서는 Claude Code가 PHM 서비스의 Amplitude 이벤트 트래킹 코드를 구현할 때 참조하는 컨텍스트입니다.
코드를 작성하기 전에 이 문서 전체를 반드시 읽고 반영하세요.

---

## 프로젝트 현황

- **서비스명:** PHM (사주 기반 인터랙티브 픽션 서비스)
- **F/E 프레임워크:** Next.js + React + Tailwind CSS + shadcn/ui (확정)
- **B/E 프레임워크:** Node.js + TypeScript — Next.js API Routes 활용 (확정)
  - 결제 요청: PayApp `POST https://api.payapp.kr/oapi/apiLoad.html` (cmd=payrequest)
  - 결제 콜백: PayApp feedbackurl (`/api/payment/callback`) 수신 후 `state=1` 검증
  - DA 이벤트: `payment_complete` Amplitude HTTP API v2 서버사이드 전송 (feedbackurl 콜백 수신 시)
  - F/E와 TypeScript 타입 공유 가능 (`CharacterId`, `UserId` 등)
- **DA 트래킹 툴:** Amplitude
- **Amplitude 도입 목적:** 사용자 행동 데이터를 수집하여 보안이 강화된 자체 분석 환경에서 관리하기 위함

---

## Amplitude 도입 원칙

1. **개인정보(PII)는 절대 Amplitude로 전송하지 않는다**
   - 이름, 생년월일, 이메일, 전화번호는 전송 금지
   - 사용자 식별은 내부 익명 ID만 사용

2. **이벤트 이름은 이 문서에 정의된 것만 사용한다**
   - 임의로 이벤트 이름을 추가하거나 변경하지 않는다
   - 모든 이벤트 이름은 `snake_case` 소문자

3. **F/E와 B/E의 역할을 혼동하지 않는다**
   - `payment_complete` 이벤트만 B/E에서 서버 사이드로 전송
   - 나머지 12개 이벤트는 F/E에서 클라이언트 사이드로 전송

4. **Next.js SSR 환경을 고려한다**
   - Amplitude 초기화는 반드시 클라이언트 사이드에서만 실행
   - `'use client'` 컴포넌트 또는 `useEffect` 내부에서 초기화

---

## Amplitude SDK 설치 및 초기화

```bash
npm install @amplitude/analytics-browser
```

```typescript
// lib/amplitude.ts
'use client';

import * as amplitude from '@amplitude/analytics-browser';

export const initAmplitude = () => {
  amplitude.init(process.env.NEXT_PUBLIC_AMPLITUDE_API_KEY!, {
    defaultTracking: false,
  });
};

export const trackEvent = (eventName: string, properties?: Record<string, unknown>) => {
  amplitude.track(eventName, properties);
};
```

```typescript
// app/layout.tsx 또는 최상위 클라이언트 컴포넌트
'use client';

import { useEffect } from 'react';
import { initAmplitude } from '@/lib/amplitude';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    initAmplitude();
  }, []);

  return <>{children}</>;
}
```

```bash
# .env.local
NEXT_PUBLIC_AMPLITUDE_API_KEY=your_api_key_here
```

---

## 이벤트 상수 정의

```typescript
// lib/amplitude-events.ts
export const EVENTS = {
  LANDING_ENTER: 'landing_enter',
  INTRO_STEP_COMPLETE: 'intro_step_complete',
  ENTER_CHARACTER_SELECT: 'enter_character_select',
  CHARACTER_SELECT: 'character_select',
  USER_INFO_SUBMIT: 'user_info_submit',
  RESULT_RECEIVED: 'result_received',
  PAYMENT_CLICK: 'payment_click',
  PAYMENT_COMPLETE: 'payment_complete',
  PAID_VERSION_ENTER: 'paid_version_enter',
  PAID_VERSION_COMPLETE_READ: 'paid_version_complete_read',
  SHARE_CLICK: 'share_click',
  UNSELECTED_CHARACTER_SHOWN: 'unselected_character_shown',
  UNSELECTED_CHARACTER_CLICK: 'unselected_character_click',
} as const;
```

---

## 이벤트 명세 및 구현 위치

### 1. `landing_enter`
- **발생 조건:** 메인 페이지 컴포넌트 마운트 시
- **구현 위치:** 메인 페이지 컴포넌트 `useEffect`
- **DA 목적:** Activation — 서비스 진입 측정, 유입 채널 분석

```typescript
useEffect(() => {
  const params = new URLSearchParams(window.location.search);
  trackEvent(EVENTS.LANDING_ENTER, {
    utm_source: params.get('utm_source'),
    utm_medium: params.get('utm_medium'),
    utm_campaign: params.get('utm_campaign'),
    referral_code: params.get('ref'),
  });
}, []);
```

---

### 2. `intro_step_complete`
- **발생 조건:** 사용자가 상호작용 버튼 클릭하여 다음 인트로 단계로 넘어갈 때
- **구현 위치:** 인트로 시퀀스 버튼 클릭 핸들러
- **DA 목적:** Activation — 단계별 CTR 및 이탈률 계산

```typescript
const handleNextStep = (currentStep: number, totalSteps: number) => {
  trackEvent(EVENTS.INTRO_STEP_COMPLETE, {
    step_index: currentStep,
    step_total: totalSteps,
  });
  // 다음 단계로 이동 로직
};
```

---

### 3. `enter_character_select`
- **발생 조건:** 캐릭터 선택 화면 컴포넌트 마운트 시
- **구현 위치:** 캐릭터 선택 페이지 컴포넌트 `useEffect`
- **DA 목적:** Activation — 캐릭터 선택 소요 시간 측정 시작점

```typescript
useEffect(() => {
  trackEvent(EVENTS.ENTER_CHARACTER_SELECT, {
    timestamp_enter: Date.now(),
  });
}, []);
```

---

### 4. `character_select`
- **발생 조건:** 사용자가 캐릭터 선택 버튼 클릭 시
- **구현 위치:** 캐릭터 선택 버튼 클릭 핸들러
- **DA 목적:** Activation — 캐릭터 선택 소요 시간 측정 종료점 / Retention — 캐릭터별 결제 상관관계

```typescript
const handleCharacterSelect = (characterId: 'A' | 'B') => {
  trackEvent(EVENTS.CHARACTER_SELECT, {
    character_id: characterId,
    timestamp_select: Date.now(),
  });
  // 선택 처리 로직
};
```

---

### 5. `user_info_submit`
- **발생 조건:** 사용자 정보 입력 완료 후 결과 생성 요청 전송 시
- **구현 위치:** 정보 입력 폼 submit 핸들러
- **DA 목적:** Transition — 전이 시퀀스 추적
- **주의:** 이름, 생년월일, 이메일 등 개인정보는 절대 포함하지 않는다

```typescript
const handleSubmit = (characterId: 'A' | 'B') => {
  trackEvent(EVENTS.USER_INFO_SUBMIT, {
    character_id: characterId,
    // 개인정보 절대 포함 금지
  });
  // 결과 생성 요청 로직
};
```

---

### 6. `result_received`
- **발생 조건:** 결과 페이지 컴포넌트 마운트 시 (로딩 완료 후)
- **구현 위치:** 결과 페이지 컴포넌트 `useEffect` (데이터 로딩 완료 후)
- **DA 목적:** Retention — 유료 결제 전환율 분모

```typescript
useEffect(() => {
  if (resultLoaded) {
    trackEvent(EVENTS.RESULT_RECEIVED, {
      character_id: characterId,
      result_type: isPaid ? 'paid' : 'free',
    });
  }
}, [resultLoaded]);
```

---

### 7. `payment_click`
- **발생 조건:** 결제 버튼 클릭 시
- **구현 위치:** 결제 버튼 클릭 핸들러
- **DA 목적:** Retention — 결제 의도 측정

```typescript
const handlePaymentClick = (characterId: 'A' | 'B') => {
  trackEvent(EVENTS.PAYMENT_CLICK, {
    character_id: characterId,
  });
  // 결제 플로우 진입 로직
};
```

---

### 8. `payment_complete`
- **발생 조건:** PayApp feedbackurl 콜백 수신 후 (`state=1` 확인 시)
- **구현 위치:** `app/api/payment/callback/route.ts` — B/E 서버 사이드 (F/E에서 중복 전송 금지)
- **DA 목적:** Retention — 유료 결제 전환율 분자, 캐릭터별 결제 상관관계

**PayApp 결제 플로우:**
1. F/E → B/E: 결제 요청 (`/api/payment/request`) — B/E가 PayApp API에 `cmd=payrequest` POST
2. PayApp → 구매자 휴대폰: 결제 링크 SMS 발송
3. 구매자 결제 완료 → PayApp → feedbackurl로 콜백 POST (`state=1` 포함)
4. B/E feedbackurl 핸들러에서 `state=1` 검증 → Amplitude `payment_complete` 전송

```typescript
// app/api/payment/request/route.ts — 결제 요청
import { NextRequest, NextResponse } from 'next/server';
import querystring from 'querystring';

export async function POST(req: NextRequest) {
  const { characterId, internalUserId, goodName, price, recvPhone } = await req.json();

  const postData = querystring.stringify({
    cmd: 'payrequest',
    userid: process.env.PAYAPP_USERID!,
    goodname: goodName,
    price: String(price),
    recvphone: recvPhone,
    // feedbackurl은 PayApp 대시보드 또는 요청 파라미터로 지정
    feedbackurl: `${process.env.NEXT_PUBLIC_BASE_URL}/api/payment/callback`,
    // characterId와 internalUserId는 feedbackurl 쿼리스트링으로 전달해 콜백에서 식별
    // 예: feedbackurl=https://example.com/api/payment/callback?cid=A&uid=user_abc123
  });

  const payappRes = await fetch('https://api.payapp.kr/oapi/apiLoad.html', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: postData,
  });

  const responseText = await payappRes.text();
  const result = querystring.parse(responseText);

  // state=1: 결제 요청 성공 (mul_no 발급), state=0: 실패
  if (result.state !== '1') {
    return NextResponse.json({ error: result.errstr }, { status: 400 });
  }

  return NextResponse.json({ success: true, mul_no: result.mul_no });
}
```

```typescript
// app/api/payment/callback/route.ts — PayApp feedbackurl 콜백 수신
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const body = await req.text();
  const params = new URLSearchParams(body);

  const state = params.get('state');
  const mulNo = params.get('mul_no');
  const payCost = params.get('pay_cost');

  // URL 쿼리스트링으로 전달된 서비스 내부 식별자
  const { searchParams } = new URL(req.url);
  const characterId = searchParams.get('cid');
  const internalUserId = searchParams.get('uid');

  // state=1일 때만 결제 성공 처리
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

  // PayApp은 반드시 "SUCCESS" 문자열 응답을 요구합니다
  return new NextResponse('SUCCESS', { status: 200 });
}
```

> **주의:** PayApp feedbackurl 콜백은 `application/x-www-form-urlencoded` 형식으로 수신됩니다. `req.json()`이 아닌 `req.text()` → `URLSearchParams` 파싱을 사용해야 합니다. 콜백 응답은 반드시 `"SUCCESS"` 문자열이어야 합니다.

---

### 9. `paid_version_enter`
- **발생 조건:** 유료 결과 페이지 컴포넌트 마운트 시
- **구현 위치:** 유료 결과 페이지 컴포넌트 `useEffect`
- **DA 목적:** Time — 완독 소요 시간 측정 시작점

```typescript
useEffect(() => {
  trackEvent(EVENTS.PAID_VERSION_ENTER, {
    character_id: characterId,
    timestamp_enter: Date.now(),
  });
}, []);
```

---

### 10. `paid_version_complete_read`
- **발생 조건:** 스크롤이 콘텐츠 하단 100% 도달 시
- **구현 위치:** 스크롤 감지 로직 (IntersectionObserver 권장)
- **DA 목적:** Time — 완독 소요 시간 측정 종료점

```typescript
useEffect(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) {
        trackEvent(EVENTS.PAID_VERSION_COMPLETE_READ, {
          character_id: characterId,
          timestamp_complete: Date.now(),
        });
        observer.disconnect();
      }
    },
    { threshold: 1.0 }
  );

  const bottomEl = document.getElementById('content-bottom');
  if (bottomEl) observer.observe(bottomEl);

  return () => observer.disconnect();
}, []);
```

---

### 11. `share_click`
- **발생 조건:** 공유 버튼 클릭 시
- **구현 위치:** 공유 버튼 클릭 핸들러
- **DA 목적:** Retention — 공유 클릭률 / Loop — 루프 완성률 측정

```typescript
const handleShareClick = (characterId: 'A' | 'B', referralCode: string) => {
  trackEvent(EVENTS.SHARE_CLICK, {
    character_id: characterId,
    referral_code: referralCode, // B/E에서 발급한 값
  });
  // 공유 액션 실행 로직
};
```

---

### 12. `unselected_character_shown`
- **발생 조건:** 미선택 캐릭터 UI가 뷰포트에 진입했을 때
- **구현 위치:** 미선택 캐릭터 컴포넌트 IntersectionObserver
- **DA 목적:** Retention — 미선택 캐릭터 클릭률 분모

```typescript
useEffect(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) {
        trackEvent(EVENTS.UNSELECTED_CHARACTER_SHOWN, {
          shown_character_id: unselectedCharacterId,
        });
        observer.disconnect();
      }
    },
    { threshold: 0.5 }
  );

  const el = document.getElementById('unselected-character');
  if (el) observer.observe(el);

  return () => observer.disconnect();
}, []);
```

---

### 13. `unselected_character_click`
- **발생 조건:** 미선택 캐릭터 선택 버튼 클릭 시
- **구현 위치:** 미선택 캐릭터 선택 버튼 클릭 핸들러
- **DA 목적:** Retention — 미선택 캐릭터 클릭률 분자

```typescript
const handleUnselectedCharacterClick = (unselectedCharacterId: 'A' | 'B') => {
  trackEvent(EVENTS.UNSELECTED_CHARACTER_CLICK, {
    shown_character_id: unselectedCharacterId,
  });
  // 해당 캐릭터 풀이 플로우 진입 로직
};
```

---

## DA 백로그와 이벤트 연결 관계

| DA 백로그 | 관련 이벤트 |
|---|---|
| HMDA-1 Activation | `landing_enter`, `intro_step_complete`, `enter_character_select`, `character_select` |
| HMDA-2 Transition | `landing_enter`, `character_select`, `result_received` |
| HMDA-3 Retention/System | `result_received`, `payment_click`, `payment_complete`, `share_click`, `unselected_character_shown`, `unselected_character_click` |
| HMDA-4 Time | `paid_version_enter`, `paid_version_complete_read` |
| HMDA-5 Loop | `share_click`, `landing_enter` (referral_code 포함) |

---

## 미확정 항목

- [x] `payment_complete` 이벤트 B/E 구현 언어 및 방식 — Node.js + Next.js API Routes 확정
- [x] 결제 PG사 — PayApp (api.payapp.kr) 확정, feedbackurl 콜백 방식
- [ ] `referral_code` 발급 API 구조
- [ ] 내부 사용자 익명 ID 생성 방식

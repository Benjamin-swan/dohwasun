# PHM Event Specification (Amplitude)

> 이 문서는 PHM 서비스의 Data Analytics를 위한 이벤트 명세서입니다.
> 풀스택 개발자는 아래 명세에 따라 F/E 및 B/E에서 이벤트를 트래킹하여 Amplitude로 전송해야 합니다.

---

## 기술 스택

| 구분 | 확정 스택 | Amplitude 연동 방식 |
|---|---|---|
| **F/E** | Next.js + React + TypeScript + Tailwind CSS + shadcn/ui | `@amplitude/analytics-browser` SDK |
| **B/E** | Node.js + TypeScript — Next.js API Routes | Amplitude HTTP API v2 (`POST /2/httpapi`) |
| **결제** | PayApp (`POST https://api.payapp.kr/oapi/apiLoad.html`) | `payment_complete` 이벤트 B/E feedbackurl 콜백 수신 후 전송 |

---

## Summary

**이 서비스에서 반드시 트래킹해야 할 이벤트는 총 13개입니다.**

Amplitude SDK를 F/E에 설치하고, 아래 명세의 이벤트 이름과 property를 정확히 일치시켜 전송해주세요.
이벤트 이름이나 property 키가 하나라도 다르면 DA 대시보드 측정이 깨집니다.

| # | 이벤트 이름 | 발생 시점 | 트래킹 주체 |
|---|---|---|---|
| 1 | `landing_enter` | 서비스 첫 진입 | F/E |
| 2 | `intro_step_complete` | 인트로 각 단계 완료 | F/E |
| 3 | `enter_character_select` | 캐릭터 선택 화면 진입 | F/E |
| 4 | `character_select` | 캐릭터 선택 완료 | F/E |
| 5 | `user_info_submit` | 사용자 정보 입력 완료 | F/E |
| 6 | `result_received` | 결과 페이지 진입 | F/E |
| 7 | `payment_click` | 결제 버튼 클릭 | F/E |
| 8 | `payment_complete` | 결제 완료 | B/E |
| 9 | `paid_version_enter` | 유료 결과 페이지 진입 | F/E |
| 10 | `paid_version_complete_read` | 유료 결과 완독 | F/E |
| 11 | `share_click` | 공유 버튼 클릭 | F/E |
| 12 | `unselected_character_shown` | 미선택 캐릭터 UI 노출 | F/E |
| 13 | `unselected_character_click` | 미선택 캐릭터 선택 버튼 클릭 | F/E |

---

## Amplitude SDK 설치

```bash
# npm
npm install @amplitude/analytics-browser

# yarn
yarn add @amplitude/analytics-browser
```

```javascript
// 초기화 (앱 진입점에서 1회 실행)
import * as amplitude from '@amplitude/analytics-browser';

amplitude.init('YOUR_API_KEY', {
  defaultTracking: false, // 자동 트래킹 비활성화 (수동 제어)
});
```

> `YOUR_API_KEY`는 Amplitude 프로젝트 설정에서 발급받은 API Key로 교체하세요.

---

## 이벤트 명세

---

### 1. `landing_enter`

**설명:** 사용자가 서비스에 처음 진입했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 메인 페이지 컴포넌트 마운트 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `utm_source` | string | 선택 | 유입 채널 | `"instagram"`, `"kakao"`, `null` |
| `utm_medium` | string | 선택 | 유입 매체 | `"social"`, `"cpc"`, `null` |
| `utm_campaign` | string | 선택 | 캠페인명 | `"launch_v1"`, `null` |
| `referral_code` | string | 선택 | 공유 링크 경유 진입 시 코드 | `"abc123"`, `null` |

```javascript
amplitude.track('landing_enter', {
  utm_source: new URLSearchParams(window.location.search).get('utm_source'),
  utm_medium: new URLSearchParams(window.location.search).get('utm_medium'),
  utm_campaign: new URLSearchParams(window.location.search).get('utm_campaign'),
  referral_code: new URLSearchParams(window.location.search).get('ref'),
});
```

---

### 2. `intro_step_complete`

**설명:** 인트로 시퀀스의 각 단계가 완료될 때마다 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 사용자가 상호작용 버튼을 클릭하여 다음 단계로 넘어갈 때

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `step_index` | number | 필수 | 완료된 단계 번호 (1부터 시작) | `1`, `2`, `3` |
| `step_total` | number | 필수 | 전체 단계 수 | `5` |

```javascript
amplitude.track('intro_step_complete', {
  step_index: 2,
  step_total: 5,
});
```

> **주의:** 이 이벤트로 Funnel 차트의 단계별 CTR과 이탈률을 계산합니다. `step_index`가 정확해야 합니다.

---

### 3. `enter_character_select`

**설명:** 캐릭터 선택 화면에 진입했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 캐릭터 선택 화면 컴포넌트 마운트 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `timestamp_enter` | number | 필수 | 화면 진입 시각 (Unix ms) | `1713200000000` |

```javascript
amplitude.track('enter_character_select', {
  timestamp_enter: Date.now(),
});
```

> **주의:** `character_select` 이벤트와의 inter-event interval로 캐릭터 선택 소요 시간을 계산합니다. `timestamp_enter` 값이 반드시 포함되어야 합니다.

---

### 4. `character_select`

**설명:** 사용자가 캐릭터를 선택 완료했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 캐릭터 선택 버튼 클릭 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |
| `timestamp_select` | number | 필수 | 선택 완료 시각 (Unix ms) | `1713200005000` |

```javascript
amplitude.track('character_select', {
  character_id: 'A',
  timestamp_select: Date.now(),
});
```

---

### 5. `user_info_submit`

**설명:** 사용자가 생년월일, 이름, 이메일 등 정보 입력을 완료했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 정보 입력 완료 후 결과 생성 요청 전송 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |

```javascript
amplitude.track('user_info_submit', {
  character_id: 'A',
});
```

> **주의:** 개인정보(이름, 생년월일, 이메일)는 절대 Amplitude에 전송하지 마세요.

---

### 6. `result_received`

**설명:** 사용자가 결과 페이지에 진입했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 결과 페이지 컴포넌트 마운트 시 (로딩 완료 후)

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |
| `result_type` | string | 필수 | 결과 유형 | `"free"`, `"paid"` |

```javascript
amplitude.track('result_received', {
  character_id: 'A',
  result_type: 'free',
});
```

---

### 7. `payment_click`

**설명:** 사용자가 결제 버튼을 클릭했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 결제 버튼 클릭 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |

```javascript
amplitude.track('payment_click', {
  character_id: 'A',
});
```

---

### 8. `payment_complete`

**설명:** 결제가 성공적으로 완료되었을 때 발생합니다.
**트래킹 주체:** B/E
**발생 조건:** PayApp feedbackurl 콜백 수신 후 (`state=1` 확인 시)

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |
| `user_id` | string | 필수 | 서비스 내부 사용자 식별자 | `"user_abc123"` |
| `payment_method` | string | 필수 | 결제 수단 | `"payapp"` |
| `mul_no` | string | 필수 | PayApp 결제요청번호 | `"12345678"` |

```javascript
// B/E PayApp feedbackurl 콜백 핸들러 — app/api/payment/callback/route.ts
// PayApp은 결제 완료 시 feedbackurl로 POST 요청을 전송합니다.
// 응답 body에 state=1이면 결제 성공입니다.

// POST https://api.payapp.kr/oapi/apiLoad.html (결제 요청 단계)
// PayApp 결제 요청 파라미터:
// cmd=payrequest, userid=판매자ID, goodname=상품명, price=금액,
// recvphone=구매자휴대폰, feedbackurl=https://YOUR_DOMAIN/api/payment/callback

// feedbackurl 콜백 수신 후 Amplitude HTTP API v2 전송:
// POST https://api2.amplitude.com/2/httpapi
{
  "api_key": "YOUR_API_KEY",
  "events": [{
    "event_type": "payment_complete",
    "user_id": "user_abc123",
    "event_properties": {
      "character_id": "A",
      "payment_method": "payapp",
      "mul_no": "12345678"
    }
  }]
}
```

> **PayApp 콜백 처리 주의사항:**
> - PayApp은 feedbackurl로 `state`, `mul_no`, `pay_cost`, `goodname` 등을 POST로 전달합니다.
> - `state=1`일 때만 결제 성공으로 처리하고 Amplitude 이벤트를 전송해야 합니다.
> - `state=0`이면 결제 실패이므로 이벤트 전송 금지.
> - 콜백 수신 시 서버에서 `SUCCESS` 문자열을 응답해야 PayApp이 정상 처리합니다.

---

### 9. `paid_version_enter`

**설명:** 결제 완료 후 유료 결과 페이지에 진입했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 유료 결과 페이지 컴포넌트 마운트 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |
| `timestamp_enter` | number | 필수 | 진입 시각 (Unix ms) | `1713200010000` |

```javascript
amplitude.track('paid_version_enter', {
  character_id: 'A',
  timestamp_enter: Date.now(),
});
```

---

### 10. `paid_version_complete_read`

**설명:** 사용자가 유료 결과를 완독했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 스크롤이 콘텐츠 하단 100%에 도달했을 때

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |
| `timestamp_complete` | number | 필수 | 완독 시각 (Unix ms) | `1713200070000` |

```javascript
amplitude.track('paid_version_complete_read', {
  character_id: 'A',
  timestamp_complete: Date.now(),
});
```

> **주의:** `paid_version_enter` → `paid_version_complete_read` inter-event interval로 완독 소요 시간을 계산합니다. 완독 조건(스크롤 100% 도달 기준)을 F/E에서 명확히 구현해야 합니다.

---

### 11. `share_click`

**설명:** 사용자가 공유 버튼을 클릭했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 공유 버튼 클릭 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `character_id` | string | 필수 | 선택한 캐릭터 식별자 | `"A"`, `"B"` |
| `referral_code` | string | 필수 | B/E에서 발급한 공유 링크 고유 코드 | `"abc123"` |

```javascript
amplitude.track('share_click', {
  character_id: 'A',
  referral_code: 'abc123', // B/E에서 발급받은 값
});
```

> **주의:** `referral_code`는 B/E 공유 링크 생성 API 응답값을 사용해야 합니다. 이 값이 있어야 루프 측정(HMDA-5)이 가능합니다.

---

### 12. `unselected_character_shown`

**설명:** 미선택 캐릭터 UI가 화면에 노출되었을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 미선택 캐릭터 말걸기 UI 컴포넌트가 뷰포트에 진입했을 때

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `shown_character_id` | string | 필수 | 노출된 미선택 캐릭터 식별자 | `"A"`, `"B"` |

```javascript
amplitude.track('unselected_character_shown', {
  shown_character_id: 'B',
});
```

---

### 13. `unselected_character_click`

**설명:** 사용자가 미선택 캐릭터 선택 버튼을 클릭했을 때 발생합니다.
**트래킹 주체:** F/E
**발생 조건:** 미선택 캐릭터 선택 버튼 클릭 시

| Property | 타입 | 필수 | 설명 | 예시 |
|---|---|---|---|---|
| `shown_character_id` | string | 필수 | 클릭한 미선택 캐릭터 식별자 | `"A"`, `"B"` |

```javascript
amplitude.track('unselected_character_click', {
  shown_character_id: 'B',
});
```

---

## 주의사항

1. **개인정보 금지:** 이름, 생년월일, 이메일, 전화번호 등 개인식별정보(PII)는 절대 Amplitude에 전송하지 마세요.
2. **이벤트 이름 대소문자:** 이벤트 이름은 모두 `snake_case` 소문자입니다. 오타 방지를 위해 상수로 관리하는 것을 권장합니다.
3. **B/E 이벤트:** `payment_complete`는 B/E에서 Amplitude HTTP API를 통해 서버 사이드로 전송합니다. F/E에서 중복 전송하지 마세요. PayApp feedbackurl 콜백(`state=1`)을 수신한 시점에 전송합니다.
4. **timestamp 단위:** 모든 timestamp property는 **Unix milliseconds** 기준입니다. (`Date.now()` 사용)

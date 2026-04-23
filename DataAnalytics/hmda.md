# PHM Data Analytics Backlog

---

## 기술 스택

| 구분 | 확정 스택 |
|---|---|
| **F/E** | Next.js + React + TypeScript + Tailwind CSS + shadcn/ui |
| **B/E** | Node.js + TypeScript — Next.js API Routes |
| **결제** | PayApp (`POST https://api.payapp.kr/oapi/apiLoad.html`, cmd=payrequest) |
| **DA 트래킹** | Amplitude (F/E: Browser SDK, B/E: HTTP API v2) |

---

## [HMDA-1] Activation 측정

### Success Criteria
- 서비스 흐름 각 단계별 CTR(클릭률)이 Amplitude 대시보드에서 수치로 확인된다
- 서비스 흐름 각 단계별 이탈률이 Amplitude 대시보드에서 수치로 확인된다
- 캐릭터 선택 완료까지 소요 시간이 `enter_character_select` → `character_select` inter-event interval로 계산되어 출력된다

### To-do
- [ ] Amplitude 계정 생성 및 프로젝트 API Key 발급 확인
- [ ] `landing_enter` 이벤트가 Amplitude에 정상 수신되는지 확인
- [ ] `intro_step_1` ~ `intro_step_N` 단계별 이벤트가 순서대로 수신되는지 확인
- [ ] `enter_character_select` 이벤트가 수신되는지 확인
- [ ] `character_select` 이벤트가 수신되는지 확인
- [ ] Amplitude Funnel 차트에서 단계별 CTR 수치 출력 확인
- [ ] Amplitude Funnel 차트에서 단계별 이탈률 수치 출력 확인
- [ ] `enter_character_select` → `character_select` 구간 소요 시간 출력 확인

### Review
- Funnel 차트 단계 순서가 실제 서비스 흐름과 일치하는가
- 이탈률이 특정 단계에 집중되어 있다면 해당 단계 UI/UX 재검토 필요
- 소요 시간이 지나치게 짧거나 길다면 캐릭터 선택 화면 설계 재검토 필요

### Issue
- F/E에서 각 단계 이벤트 트래킹 코드가 삽입되지 않으면 이 측정 전체 불가
- 인트로 시퀀스 단계 수(N)가 확정되지 않으면 Funnel 단계 설계 불가

---

## [HMDA-2] Transition 측정

### Success Criteria
- `landing_enter` → `character_select` → `result_received` 전이 시퀀스가 사용자 단위로 Amplitude에서 재구성되어 확인된다
- 유입 채널별(UTM 파라미터 등) 행동 전이 흐름 차이가 대시보드에서 확인된다

### To-do
- [ ] `landing_enter` / `character_select` / `result_received` 이벤트 3개가 모두 수신되는지 확인
- [ ] Amplitude User Path 또는 Journeys 차트에서 사용자 단위 전이 흐름 출력 확인
- [ ] `landing_enter` 이벤트에 `utm_source` / `utm_medium` / `utm_campaign` property가 포함되는지 확인
- [ ] 유입 채널별로 Funnel 차트를 필터링했을 때 수치 차이 확인

### Review
- 특정 채널에서 유입된 사용자가 결제까지 전환되는 비율이 높다면 해당 채널 집중 투자 검토
- `result_received` 도달 비율이 낮다면 중간 이탈 구간 재확인 (HMDA-1 Activation과 연계)

### Issue
- UTM 파라미터는 F/E에서 URL 파싱 후 `landing_enter` 이벤트 property로 전달해야 측정 가능
- 유입 채널이 아직 확정되지 않은 경우 채널 분류 기준 사전 합의 필요
- **인스타그램 등 SNS 유입 추적은 Amplitude만으로 불가** — 마케터가 UTM 파라미터가 포함된 링크(`?utm_source=instagram&utm_medium=social&utm_campaign=...`)를 게시해야 측정 가능. UTM 없이 단순 링크 공유 시 유입 경로 식별 불가 (인스타그램 앱은 브라우저 referrer도 차단)
- DA 측 역할: 유입 채널별 UTM 파라미터 네이밍 규칙 정의 및 마케터에게 가이드 문서 제공 필요

---

## [HMDA-3] Retention / System 측정

### Success Criteria
- 유료 결제 전환율이 결과 페이지 진입(`result_received`) 대비 결제 완료(`payment_complete`) 비율로 Amplitude에서 확인된다
- 특정 캐릭터 선택과 결제 완료 간의 상관관계가 수치로 확인된다
- 공유 버튼 클릭률이 결과 수령 사용자 대비 `share_click` 이벤트 발생 비율로 확인된다
- 미선택 캐릭터 클릭률이 미선택 캐릭터 UI 노출 대비 클릭 비율로 확인된다

### To-do
- [ ] `result_received` 이벤트 수신 확인
- [ ] `payment_click` / `payment_complete` 이벤트 수신 확인
- [ ] `share_click` 이벤트 수신 확인
- [ ] `unselected_character_shown` / `unselected_character_click` 이벤트 수신 확인
- [ ] `character_select` 이벤트의 `character_id` property가 A/B 구분되어 수신되는지 확인
- [ ] Amplitude Funnel: `result_received` → `payment_complete` 전환율 출력 확인
- [ ] Amplitude Segmentation: `character_id` 기준으로 `payment_complete` 비율 비교 확인
- [ ] `result_received` 대비 `share_click` 비율 출력 확인
- [ ] `unselected_character_shown` 대비 `unselected_character_click` 비율 출력 확인

### Review
- 캐릭터 A vs B 중 결제 전환율이 높은 캐릭터가 있다면 해당 캐릭터를 기본 추천으로 설정하는 것 검토
- 공유 클릭률이 낮다면 공유 버튼 위치/디자인 재검토 필요
- 미선택 캐릭터 클릭률이 높다면 루프 재진입 동기 확인 (HMDA-5 루프 측정과 연계)

### Issue
- `unselected_character_shown` 이벤트는 F/E에서 UI 노출 시점에 명시적으로 트래킹해야 함
- 결제 완료 이벤트는 PayApp feedbackurl 콜백 연동 후에만 정확한 측정 가능 (state=1 수신 시점)

---

## [HMDA-4] Time 측정

### Success Criteria
- 유료 버전 완독 소요 시간이 `paid_version_enter` → `paid_version_complete_read` inter-event interval로 계산되어 Amplitude에서 확인된다

### To-do
- [ ] `paid_version_enter` 이벤트 수신 확인
- [ ] `paid_version_complete_read` 이벤트 수신 확인
- [ ] Amplitude에서 두 이벤트 간 소요 시간(interval) 출력 확인
- [ ] 소요 시간 분포(짧은 사용자 vs 긴 사용자) 히스토그램 출력 확인

### Review
- 완독 소요 시간이 지나치게 짧다면 실제로 읽지 않고 스크롤만 한 것으로 판단 가능 → 스크롤 깊이 측정 추가 검토
- 완독 소요 시간이 지나치게 길다면 콘텐츠 길이 또는 가독성 문제 재검토 필요

### Issue
- `paid_version_complete_read` 이벤트 발생 조건(스크롤 100% 도달 or 특정 버튼 클릭)을 F/E와 사전 합의 필요
- 결제 완료 사용자가 충분히 쌓이기 전까지는 샘플 수가 부족하여 측정 의미 없을 수 있음

---

## [HMDA-5] 루프 측정

### Success Criteria
- 전체 루프 완성률이 `landing_enter` → `share_click` → 신규 `landing_enter` 발생 비율로 Amplitude에서 확인된다

### To-do
- [ ] `share_click` 이벤트에 공유 링크 ID property가 포함되는지 확인
- [ ] 공유 링크를 통해 진입한 신규 `landing_enter` 이벤트가 구분 가능한지 확인 (referrer 또는 UTM 파라미터)
- [ ] Amplitude에서 `landing_enter` → `share_click` Funnel 출력 확인
- [ ] 공유 링크 경유 신규 진입 비율 출력 확인
- [ ] 루프 완성(공유 후 신규 진입 발생) 사용자 수 출력 확인

### Review
- 루프 완성률이 낮다면 공유 동기 강화 방안 검토 (공유 시 인센티브, 공유 문구 개선 등)
- 루프를 통해 유입된 사용자의 결제 전환율을 일반 유입과 비교하여 루프의 실질적 가치 판단 (HMDA-3 연계)

### Issue
- 공유 링크를 통한 재진입을 추적하려면 B/E에서 공유 링크 생성 시 고유 식별자(referral_code)를 포함해야 함
- 루프 완성은 시간이 걸리는 측정이므로 MVP 초기에는 `share_click` 발생 여부만 먼저 확인하는 것을 추천

---

## [HMDA-6] UTM 네이밍 규칙 정의 및 마케터 가이드

### 담당
DA

### Success Criteria
- 전 채널에서 UTM 파라미터가 통일된 규칙으로 사용되어 Amplitude에서 채널별 데이터가 중복 없이 집계된다
- 마케터가 새 캠페인 링크를 만들 때 DA에게 묻지 않고 가이드 문서만으로 생성 가능하다

### To-do
- [ ] 사용할 채널 목록 확정 (인스타그램, 카카오, 스레드 등)
- [ ] `utm_source` / `utm_medium` / `utm_campaign` 네이밍 규칙 문서화
  - 예: `utm_source=instagram`, `utm_medium=social`, `utm_campaign=launch_v1`
  - 대소문자 소문자 고정, 공백 대신 언더스코어 사용 등 규칙 명시
- [ ] 채널별 UTM 링크 템플릿 사전 제작 및 마케터에게 공유
- [ ] Amplitude에서 `utm_source` 값이 규칙대로 수신되는지 검증 (개발 연동 후)

### Review
- `utm_source` 값이 `instagram` / `Instagram` / `IG` 등으로 혼재되면 채널별 집계가 깨지므로 규칙 준수 여부 주기적으로 모니터링 필요

### Issue
- 이 작업은 개발 연동과 무관하게 DA가 선행 가능 (문서화 단계)
- 마케터가 UTM 없는 링크를 게시하면 측정 불가 — 캠페인 집행 전 반드시 가이드 공유 필요

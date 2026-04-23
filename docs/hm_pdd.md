- **서브컬쳐 타겟 New 사주 서비스 (문제정의서)**
    - 용어표
        
        
        | 용어 / 기호 | 의미 | 설명 / 범위 |
        | --- | --- | --- |
        | $C$ | Community Signal | 트위터·아카라이브 등 서브컬처 커뮤니티 내 사주·운세 관련 대화 및 시딩 콘텐츠 노출. 행동 흐름의 시작점 |
        | $C_{community}$ | Community Discourse | 커뮤니티 내 자발적으로 발생하는 사주·운세 관련 대화 신호. $C$의 구체적 관측 형태 |
        | $I$ | Subculture Interest | 커뮤니티 노출 누적으로 형성된 사주·운세 탐색 관심도. $C$ 노출 반복으로 활성화 |
        | $I_{adopt}$ | Saju Adopt Index | 사주·운세에 대한 탐색·채택 욕구. 서브컬처 사용자도 독립적으로 보유하고 있으나 적합한 공급처를 찾지 못한 상태 |
        | $I_{subculture}$ | Subculture Identity Index | 서브컬처 감성 정체성 지수. 덕후·팬덤 문화에 깊이 속해 있으며 콘텐츠 소비·생산 방식이 메인스트림과 구별되는 사용자군의 특성. $M$ 형성 가능성과 비례 |
        | $O$ | Observer State | 콘텐츠를 소비하지만 행동($A$)은 하지 않고 판단만 지속하는 상태. Bootstrap 전환 실패의 80%가 여기서 발생. Bootstrap 핵심 목표: $O→A_1$ |
        | $M$ | Meaning | 플랫폼이 제공하는 의미 해석 레이어. "이건 내 감성 공간이다"라는 인식. $A_1$ 전이의 직접 조건. $J$ 이전에 형성됨. $M→J$ 연결 실패 = Semantic Mapping 실패 |
        | $A$ | Action | 실제 행동. $A_1$, $A_2$, $A_3$으로 분해. 단일 상태로 묶지 않음 |
        | $A_1$ | First Action | 결과 보기 — 저비용 첫 행동. curiosity로 발생. Test 2 핵심 |
        | $A_2$ | Revisit Action | 재방문 — 정체성 기반 행동. "이 서비스, 또 왔다"는 것 자체가 $J_{initial}$ 신호. identity resonance로 발생. MVP 단계 Test 4 핵심. 공유$(A_{share}$)는 MVP 이후 별도 도입 |
        | $A_3$ | Pay Action | 결제 — 가치 인정 행동. utility + trust로 발생. Test 3 핵심 |
        | $A_{share}$ | Share Action | 결제 완료 사용자의 공유 카드 공유 행동 |
        | $J$ | Belief State | "이 플랫폼이 나와 내 서브컬처 정체성을 위한 공간이다"라는 내부 신념 상태. 반복 행동 후 점진적으로 형성되는 누적 상태. 
        $J_{t+1}=F(J_t,A_t,M_t)$로 업데이트. Bootstrap 이후 형성 |
        | $J_{t+1}=F(J_t,A_t,M_t)$ | Belief Transition Function | 행동과 의미 형성 이후 신념 상태가 업데이트되는 전이 함수. $A$ 없으면 $J$는 항상 초기화. Bootstrap 이후 적용 |
        | $B_{brand}$ | Brand Mismatch Barrier | 기존 플랫폼(청월당 등)이 자신의 서브컬처 정체성과 맞지 않는다는 심리적 마찰. 기능 문제가 아닌 감성·공간의 문제. Test 1 A→B 차이로 분리 시도 |
        | $B_{supply}$ | Supply Barrier | 서브컬처 맥락에 맞는 콘텐츠 공급 부재. 서브컬처 특화 콘텐츠가 시장에 존재하지 않음. Test 1 B→C 차이로 분리 시도 |
        | $B_{stigma}$ | Stigma Barrier | 사주 본다는 사실 노출 부담 + AI 창작물 거부감 등 취향 노출에 대한 심리적 저항. 메인스트림 플랫폼에서 서브컬처 취향을 드러내는 것에 대한 불안 |
        | $B_{total}$ | Total Entry Barrier | 진입 장벽 총합. $B_{total}=B_{brand}+B_{supply}+B_{stigma}$. 현재 분리 불가능한 혼합 효과(non-identifiable system).  $P(A)∼e−B_{total}$ |
        | $U$ | Community Uncertainty | 커뮤니티 내 감정적 긴장 상태. 아이돌 컴백·커뮤니티 이슈·시즌 이벤트 등  $I_{adopt}$ 상승 외부 촉진 요인. 미통제 시 CTR·CR 폭증으로 실험 결과 왜곡 위험 |
        | $P(A)$ | Action Probability | 행동 발생 확률. 
        $P(A)∼e−B_{total}$. 현재 관측 가능한 유일한 출력값. 구성 요소 분리 불가능 |
        | $P_{convert}$ | Conversion Probability | 진입 전 전환 가능성. 
        $σ(αI_{subculture}+βU−γB_{total})$ |
        | $P_{retry}$ | Retry Probability | 기존 플랫폼 이용 실패 누적 후 재시도 확률. 
        $P_{retry}=f(n), n↑→P↓. <0.2$이면 고착 상태 |
        | $π_0$ | Initial Policy | Bootstrap 단계 행동 유도 정책. 
        $π_0(A∣M,U,friction)$. 설득·유도·자극 중심. 
        $J$ 없는 상태에서 작동 |
        | $π_1$ | Belief-based Policy | $J$ 형성 후 작동하는 습관·신념 기반 정책. 
        $π_1(A∣J)$. Bootstrap 이후 적용 |
        | $TTR_1$ | Time-to-Result (First) | 진입 → $A_1$완료까지 소요 시간. ≤ 15초 목표. 너무 빠를 경우 $M$ 형성 없이 통과하는 함정 존재.  $fast \neq meaningful$ |
        | $TTR_2$ | Time-to-Result (Meaning) | 결과 화면에서 
        $M$ 형성을 위한 의미 해석 체류 시간. ≥ 60초 목표. $TTR_1$과 분리 측정 |
        | $CTR$ | Click Through Rate | 시딩 콘텐츠 노출 대비 랜딩 진입 비율. Test 1 핵심 지표.
        $CTR=f(B_{brand},curiosity,novelty,copy quality)$ → 상관관계만 제공 |
        | $CR_t$ | Completion Rate | 랜딩 진입 후 결과 수령($A_1$) 완료 비율. ≥ 50% 목표. Test 2 핵심 지표 |
        | $CTR_{pay}$ | Pay Button CTR | 페이월 노출 대비 결제 버튼 클릭 비율. ≥ 10% 목표. Test 3 |
        | $CVR_{pay}$ | Pay Conversion Rate | 결제 버튼 클릭 후 완료 비율. ≥ 5% 목표. WTP 직접 증거. Test 3 핵심 지표 |
        | $SR_t$ | Share Rate | 결과 수령 사용자 중 공유 행동($A_2$) 발생 비율. ≥ 30% 목표. Test 4 핵심 지표 |
        | $UTM_{source}$ | 유입 채널 추적 파라미터 | 커뮤니티별 시딩 효과 분리 측정. 
        $I_{subculture}$ 수준 추정 — 커뮤니티별 유입 비율로 감성 집단 분류. 수집률 ≥ 0.95 |
        | $D1/D7/D30$ | Retention | 재방문율. $J$  형성 및 안정화 여부 간접 지표. Test 4 관찰 대상 |
        | $ARPU$ | Average Revenue Per User | 사용자당 평균 수익. WTP 수준 측정. $₩1,990 × 
        CVR_{pay}$ |
        | $DAG$ | State Transition Graph | 상태 변경 구조도. Bootstrap: 
        $C→I→O→M→A1→A2→A3→J$ |
        | $Fake J$ 리스크 | Fake Belief Risk | 행동($A$) 발생 → $J$ 형성으로 착각하는 오류. 
        $A\neq⇒J$. 재방문 없는 $A_2$ 발생 여부 및 커뮤니티 반응 언어로 간접 검증 |
        
    - 문제정의서
        
        
        | 항목 | 내용 |
        | --- | --- |
        | **사용자 관측 / Pain Point** | 서브컬처 감성 여성 사용자는  $I_{adopt}$를 보유하고 있으나 기존 플랫폼의 $B_{brand}$ 자동 작동으로 진입 의도가 꺾이고, 진입하더라도 $M$ 형성 없이 $O$ 상태에 고착되어 $A_1$ 으로 전환되지 못하고 있다. 현재 이 Pain Point는 행동 데이터 없이 커뮤니티 모니터링 기반으로 추정된 상태이며, 아래 Bootstrap Test를 통해 실재 여부를 검증한다. |
        | **문제 배경** | 국내 사주 플랫폼이 메인스트림화되면서 서브컬처 감성 여성 사용자를 위한 공간이 구조적으로 비어가고 있다. 청월당의 더현대 입점 이후 해당 플랫폼은 
        $B_{brand}$를 자동 상승시키는 방향으로 이동했고, 트위터·아카라이브 등 커뮤니티 내 사주·운세 관련 대화 $C_{community}$ 는 이미 활성화되어 있으나 이를 포착해 $A_1$으로 연결하는 서비스 구조가 없다.
        
        행동 흐름은 **자극 → 해석 → 의미 → 행동 → 신념** 순서로 작동한다. Bootstrap 단계에서 $J$(신념 상태)는 아직 존재하지 않으며, 초기 행동은 $J$ 없이 $M$과 마찰 제거로 유도해야 한다. 현재 전체 상태 전이 구조는 다음과 같다:
        
        $C→I→O→M→A_1→A_2→A_3→A_{share}→J$
        
        Bootstrap 단계 핵심 전환 : $O→A_1$ (Observer 상태에서 첫 행동으로의 전환)
        초기 행동 트리거($J$ 없는 상태) : $A_1=f(M,curiosity,friction,U)$ |
        | **문제 정의** | 서브컬처 감성 여성 사용자가 $C_{community}$ 내에서 $I_{adopt}$를 보유하고 있음에도,
        $B_{total}(=B_{brand}+B_{supply}+B_{stigma})$이 임계값을 초과하여
        Observer 상태($O$)에서 첫 행동($A_1$)으로 전환되지 못하고 있다.
        
        Test 1 : $B_{brand}$  존재 및 $C→I$  가능성 
        Test 2 : $O→A_1$ 전환 가능성 ⇒  $O→M→A_1$
        Test 3 : $A_3$(WTP) 발생 가능성 ⇒ $A_1→A_3$
        Test 4 : $A_2$(재방문)+ $A_{share}$ (공유) 발생 가능성 ⇒ $A_1→A_2$ / $A_3$ → $A_{share}$ |
        | **목표 (Objective)** | 서브컬처 감성 여성 사용자가 커뮤니티 자극($C$)에서 첫 결과 소비($A_1$)와 공유($A_2$)까지 도달하는 행동 흐름이 **실제로 발생하는가**를 확인한다.
        
        Bootstrap 목표 전이 : $C→I→O→M→A_1→A_2→A_3$ → $A_{share}$
        단기 : $O→A_1$ 전환 존재 확인
        중기 :  $A_1→A_3$  WTP 존재 확인, $A_1$→$A_2$ 재방문 존재 확인
        후기 :  $A_1$→$A_3$  $CVR_{pay}$ 확인, $A_3$→$A_{share}$ 래퍼럴 루프 시작 확인
        
        $J$(신념 상태) 형성은 Bootstrap 검증 대상이 아님. $A_2$(재방문)가
        $J$의 초기 형태($A_2≈J_{initial}$)로 기능하는지를 간접 관찰하는 것으로 충분. |
        | **관련 지표 (Metrics / KPI)** | **Leading** : 
        그룹별 CTR 차이 분석 
        $UTM_{source}$ 수집률$≥ 0.95$  
        $TTR_1$ (첫 결과 도달)$≤ 15초$ 
        
        **Structural** : 
        $CR_t ≥ 50%$% 
        $TTR_2$ (결과 화면 체류)$≥ 60초$ 
        $CTR_{pay}≥ 10%$% 
        $CVR_{pay}≥ 5%$%
        공유 카드 클릭률 (`share_click`)
        공유 → 신규 유입 전환율 (`share_landing`)
        
        **Lagging** : 
        D7 Retention 존재 여부 확인
        $ARPU$ 수준 파악 |
        | **제약 조건 / Guardrails** | • $J$ 가정 기반 설계 제한 : 구독·장기 잔존 설계 등 $J$ 형성 가정 기반 기능 v2 이후로 연기
        • Bootstrap 단계는 $π_0(A∣M,U,friction)$ 적용 $π_1(A∣J)$는 Bootstrap 이후
        • $U$ 변수 통제 없는 단일 실험 결과를 인과 근거로 사용 금지
        • $TTR_1 ≤ 15초$ 달성이 목표이나, 지나치게 빠른 결과 도달은 $M$ 형성 없 $A_1$ 통과하는 함정. $TTR_2$ 병행 측정 필수
        • MVP에서 $A_2$ = 재방문. 공유($A_{share}$)는 MVP 이후 별도 도입 — 커뮤니티 정서 파악 및 개인정보 공유 이슈 검토 이후
        • 공유 카드는 $A_3$(결제) 완료 사용자에게만 제공. 결과 상세 내용이 아닌 캐릭터 감성 중심(연애운 요약 + 시현·도윤 말투 + 결론 한 마디) 구성으로 사주 개인정보 노출 최소화 |
        | **메모** |  |
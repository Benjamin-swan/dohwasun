- 각각의 용어 설명
    - State (상태)
        - $C$ (Community Exposure): 외부 자극(콘텐츠, 커뮤니티 등)에 노출된 상태
        - $I$ (Interest): 특정 콘텐츠/주제에 관심이 발생한 상태
        - $O$ (Observe): 서비스 내에서 관찰만 수행하는 상태 (행동 없음)
        - $M$ (Meaning Formation): 콘텐츠 해석 및 개인적 의미 형성 단계
        - $A1$ (First Action): 첫 행동 발생 (예: 결과 조회, 클릭 등)
        - $A2$ (Revisit): 재방문 또는 반복 사용 발생 상태
        - $A3$ (Purchase): 결제 또는 가치 교환이 발생한 상태
        - $A_{share}$ (Share): 콘텐츠 공유 및 외부 확산 상태
        
    - Event (이벤트)
        - impression: 사용자에게 콘텐츠가 노출됨 ($C$ 유입 트리거)
        - interest: 콘텐츠에 대한 관심 표현 (클릭, hover 등)
        - observe: 서비스 내 탐색 또는 관찰 행동 발생
        - action: 의미 형성 이후 첫 명시적 행동 발생 ($M → A1$)
        - revisit: 재방문 또는 반복 사용 행동 ($A1 → A2$)
        - purchase: 결제 또는 유료 전환 발생 ($A1 → A3$)
        - share: 콘텐츠 외부 공유 ($A3 → A_{share}$)
        
    - State Transition (전이)
        - 각 이벤트는 $(state_{from} → state_{to})$ 형태의 전이로 정의됨
            - 전이는 사용자 행동 흐름을 구성하는 최소 단위이며,
                
                전체 시스템은 전이 그래프(transition graph)로 표현됨
                
            
    - Event Log Schema
        - $user_{id}$: 사용자 식별자 (sequence grouping 기준)
        - $timestamp$: 이벤트 발생 시점 (time-order 보장)
        - $event_{type}$: 발생 이벤트 종류 (행동 의미)
        - $state_{from}$: 전이 이전 상태
        - $state_{to}$: 전이 이후 상태
        
    - Raw Event Stream
        - 사용자 이벤트가 시간 순서대로 누적된 append-only 로그 데이터
        - 각 사용자별로 state transition sequence (trajectory) 재구성 가능
        - 분석 단위:
            - 전이 확률 (transition probability)
            - 전이 지연 시간 (Δt)
            - 경로 패턴 (path / trajectory)
    
    - trajectory = 한 사용자가 시간에 따라 밟은 상태 전이 경로 전체


- **Experiment Meta OKR**
    
    
    | Objective | Key | Result |
    | --- | --- | --- |
    | 서브컬처 감성 여성 사용자의 행동 흐름 ($C→I→O→M→A1→A2→A3→A_{share}$) 을 상태(State)-이벤트(Event) 기반 전이 시스템으로 정의하고, 사용자 단위에서 재구성 가능한 시퀀스 데이터 구조로 정규화한다 | 상태 집합  ${C, I, O, M, A1, A2, A3, A_{share}}$  및 이벤트 집합 {impression, interest, observe, action, revisit, purchase, share}을 정의하고, 각 이벤트를 $(state_{from}, state_{to})$ 전이로 매핑한 event log schema (user_id, timestamp, event_type, $state_{from}$, $state_{to}$)를 구축한다 | 사용자 행동이 time-ordered raw event stream으로 누적되며, 각 사용자에 대해 상태 전이 시퀀스 재구성이 가능하고, 전이 그래프 및 trajectory 기반 구조 분석이 가능한 데이터가 안정적으로 생성된다 |



- **Strategy OKR**
    
    
    | 항목 | 내용 |
    | --- | --- |
    | **Strategy Objective** | 사용자 행동 루프 ($C→I→O→M→A1→A2→A3→A_{share}$)는 개입 위치에 따라 transition dynamics (확률 및 Δt 구조)가 변화하며, 이 변화는 재탐색률(RR) 및 전환 완료율(PSR)에 구조적으로 영향을 준다. |
    | **Strategy K1** | 모든 사용자 행동은 time-ordered event stream으로 수집되며, event_type, timestamp, user_id, session_id 기준으로 손실 없이 기록된다. |
    | **Strategy K2** | 이벤트는 직접적인 $(state_{from}, state_{to})$로 저장되지 않으며, event window 기반 rule/probabilistic mapping을 통해 C/I/O 상태를 우선 재구성하고, M/A1/A2/A3는 초기에는 latent state로 정의한다. |
    | **Strategy K3** | 상태 전이($S_i→S_j$)는 충분한 event density 확보 이후 정의되며, 초기에는 event sequence 기반 pseudo-transition으로 구성된다. |
    | **Strategy K4** | Δt는 초기에는 inter-event interval로 정의되며, state 전이 안정화 이후에만 transition latency로 해석한다. |
    | **Strategy K5** | 각 개입은 user/session 단위 exposure 여부로 정의되며, 동일 trajectory 내에서 개입 시점 이전/이후로 분리 가능한 구조를 가진다. |
    | **Strategy K6** | 개입 그룹 vs 비개입 그룹은 user-level trajectory 또는 session-level sequence를 분석 단위로 정의하고,동일 event density, trajectory 길이, state window 기준으로 정렬되어동일 단위 기준에서 비교 가능한 구조로 구성된다. |
    | **Strategy R1** | 특정 상태 쌍($S_i→S_j$)에서 transition density가 threshold 이상으로 수렴할 경우, 해당 전이는 구조적으로 유의미한 edge로 형성된다. |
    | **Strategy R2** | 개입 이후 특정 구간의 Δt 분포가 일관된 방향으로 이동하며, 행동 속도 구조 변화가 관측된다. |
    | **Strategy R3** | $C→…→A_{share}→C$ cycle density 변화로 재탐색 루프(RR)의 형성 또는 억제 패턴이 드러난다. |
    | **Strategy R4** | 사용자 trajectory가 반복적으로 유사한 state sequence로 수렴할 경우, behavioral attractor가 존재한다고 판단한다. |
    | **Strategy R5** | 특정 state 체류 패턴이 안정화되고 전체 transition entropy가 감소하며, 시스템이 구조적으로 수렴한다. |
    | Strategy R6 | transition density 안정화 + Δt 분포 수렴 + trajectory 반복성 증가가 동시에 발생할 경우, 행동 시스템이 stable loop 구조로 형성된 것으로 판단한다. |
    | **연결 Meta KR** | 이벤트 집합 {impression, interest, observe, action, revisit, purchase, share}을 정의하고, 각 이벤트를 $(state_{from}, state_{to})$ 전이로 매핑한 event log schema (user_id, timestamp, event_type, $state_{from}$, $state_{to}$)를 구축 |
    | **연결 Decision Log** |   **1. DL-01** (연애운 단일 카테고리 + 웹툰형 인트로 + AI 제작 공개·BL 시각 언어 — $C→I$ 진입 전 개입)  
      **2. DL-02** (학파 분리 + 카테고리 명시 + 무료 오행 공개·블러 구조 — $O→A_1$ 진입 후 개입)  
      **3. DL-03** (웹툰형 도입부 + 웹소설형 풀이 본문 이중 포맷 — $A_1$ 내부 개입) 
      **4. DL-04** (미선택 캐릭터 재등장 — $A_1→A_2$ 개입) 
      **5. DL-05** (결제 사용자 대상 캐릭터 감성 중심 공유 카드 — $A_3→A_{share}$ 개입) |
- **Execution OKR**
    
    **핵심 구조 요약**
    
    | Layer | 핵심 질문 | 우리 서비스 기준 |
    | --- | --- | --- |
    | Activation | 행동이 발생하는가? | $C → I →O$ 행동이 발생했는가? |
    | Transition | 다음 행동이 이어지는가? | $O → M → A_1$ 다음 행동이 이어지는가? |
    | Retention | 다시 돌아오는가? | $A_1 → A_2$ 다시 돌아오는가? |
    | System | 루프가 존재하는가? | $C→I→O→M→A1→A2→ A3→A_{share}$ 루프가 존재하는가? |
    | Time | 시간이 의미 있는가? | 전이 구간별 $Δt$ 가 의미 있는가? |
    
    **Activation**   
    
    | O | K (최소 핵심 신호 + 해석 포함) | R (실험 조건) |
    | --- | --- | --- |
    | 시딩 콘텐츠에 노출된 사용자가 실제로 랜딩 페이지 진입 행동을 시작하는가 | •`impression` event → 시딩 콘텐츠 노출 상태. 유입 규모 자체
    
    •`landing_enter` event → 노출 후 실제 진입 행동 발생 상태. 보기만 하는지 vs 실제로 움직이는지 구분 신호 
    
    •`character_select` event → 캐릭터 선택까지 도달한 상태. 의사결정 시작까지 연결된 사용자  | 메인스트림 감성 시딩 콘텐츠 도달 유저 
    vs 서브컬쳐 감성 |
    
    **Transition**
    
    | O | K (최소 핵심 신호 + 해석 포함) | R (실험 조건) |
    | --- | --- | --- |
    | 랜딩에 진입한 사용자가 입력 과정에서 이탈하지 않고 결과 수령($A_1$)까지 행동이 실제로 연결되는가 | •`action` Event 발생 → event sequence 상 $M$ 단계에 해당하는 구간이 존재하는 것으로 추정, $A_1$ 상태 수렴 여부는 pseudo-transition으로 판단. $M→A_1$ 전이는 latent state로 정의되므로 직접 관측이 아닌 event density 기반 추정으로 구성
    
    •`landing_enter` → `character_select` → `result_received` 전이 존재 여부 → 진입 후 행동 전환이 실제 발생하는가, 각 이벤트 단계별 이탈 지점 파악 | 연우 선택 그룹 vs
    도윤 선택 그룹 |
    
    **Retention**
    
    | O | K (최소 핵심 신호 + 해석 포함) | R (실험 조건) |
    | --- | --- | --- |
    | 결과를 소비한 사용자가 1회성 소비로 끝나지 않고 서비스를 기억하고 다시 돌아오는가 | • `revisit` Event 발생 여부 → $A_1$ 이후 재방문에 해당하는 event가 발생한 비율. $A_2$ 는 latent state로 정의되므로, `revisit` event density가 threshold 이상 수렴할 경우에만 $A_2$전이로 추정
    
    • $A_2$ 상태 진입 후  재구성된 경로 패턴(trajectory) 분석
    → 미선택 캐릭터 열람 등 추가적인 $O → M → A_1$ 루프가 세션 내에 존재하는지 확인
     | `other_char_shown` 노출 그룹 vs 미노출 그룹 |
    
    **System**
    
    | O | K (최소 핵심 신호 + 해석 포함) | R (실험 조건) |
    | --- | --- | --- |
    | C→I→O→M→A1→A2→ A3→A_{share}  전체 행동 루프가 실제로 존재하는가 | •`purchase` Event 발생 여부 → $A_3$는 latent state로 정의되므로, `purchase` event 발생 시 해당 사용자가 $A_3$구간에 있는 것으로 추정
    
    • `share` Event 발생 여부 → 최종적으로 콘텐츠 외부 공유 상태($A_{share}$)로 전이되는가
    → $A_{share}$ 도달 여부도 동일하게 pseudo-transition으로 판단하며, `share` event → 신규 사용자 `impression` event 발생 여부를 trajectory 단위로 추적하여 루프 형성 가능성을 추정 | 전체 사용자 |
    
    **Time**
    
    | O | K (최소 핵심 신호 + 해석 포함) | R (실험 조건) |
    | --- | --- | --- |
    | 각 행동 전이 구간별 소요 시간 (Δt)이 의미 있는 구조를 가지는가? | • Δt($C→I$) → 시딩 노출 후 진입까지 반응 속도. 첫 반응이 빠른가
    
    • Δt($O→A_1$) → 입력 완료 후 결과 수령까지 소요 시간($TTR_1$). 마찰이 있는가 
    
    • Δt($A_1→A_2$) → 결과 수령 후 재방문까지 회복 속도. 서비스가 기억되는가 
    
    • Δt 없음 → 즉각 반응형 (생각 없음) 
    
    • Δt 큼 → 고민·탐색 구조 존재 
    
    • returning user에서 Δt 감소 → 사용자가 서비스를 학습하고 있음 | 첫 방문 유저 vs 
    재방문 유저 |
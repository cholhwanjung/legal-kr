---
name: adversarial-review
description: 계약 조항에 대해 3-agent adversarial 분석을 수행합니다. Advocate(옹호), Devil's Advocate(반박), Judge(판정) 3단계로 한국법 기준 교차 검증된 GREEN/YELLOW/RED 판정을 생성합니다. Use when a clause needs deeper analysis beyond simple playbook matching, when the risk of a wrong assessment is high, or when you need a structured debate with cited legal basis.
---

# /legal-kr:adversarial-review -- Adversarial 3-Agent 조항 분석

## Invocation

이 skill은 다음과 같은 상황에서 활성화됩니다:
- 특정 계약 조항에 대한 심층 교차 검증이 필요할 때
- `/legal-kr:review-contract-kr`에서 YELLOW/RED로 분류된 조항의 추가 분석
- 판정의 신뢰도를 높이기 위해 구조화된 찬반 논증이 필요할 때
- `/legal-kr:adversarial-review` 슬래시 커맨드가 호출될 때

### 언제 사용하는가

| 상황 | 권장 skill |
|------|-----------|
| 계약서 전체를 빠르게 스캔 | `/legal-kr:review-contract-kr` |
| 특정 조항의 심층 분석 | `/legal-kr:adversarial-review` (이 skill) |
| YELLOW/RED 판정의 근거를 강화 | `/legal-kr:adversarial-review` (이 skill) |

## Workflow

### Step 1: 조항 수신

분석할 계약 조항 텍스트를 수신합니다.
- 텍스트 직접 입력
- `/legal-kr:review-contract-kr` 결과에서 특정 조항 지정

### Step 2: 컨텍스트 수집

- **계약 유형**: B2B / B2C(소비자) / 고용 / 라이선스
- **당사자 위치**: 갑 / 을 / 중립
- **조항 유형**: 손해배상, 해지, 비밀유지, IP 등

제공되지 않으면 조항 내용에서 추론합니다.

### Step 3: Playbook 로드

`legal.local.md`에서 해당 조항 유형의 조직 표준 포지션을 확인합니다.

### Step 4: Advocate Agent 호출

`agents/advocate.md` sub-agent를 호출합니다.

**역할**: 이 조항이 유효하고 합리적인 이유를 한국법 기준으로 논증
- 관련 법령 조문 인용
- 조항 유효성을 지지하는 판례 근거
- B2B/소비자 맥락에서의 계약 자유 원칙
- 예상 반론에 대한 선제 대응

### Step 5: Devil's Advocate Agent 호출

`agents/devils-advocate.md` sub-agent를 호출합니다.

**역할**: 이 조항이 위험하거나 무효일 수 있는 이유를 한국법 기준으로 논증
- 약관규제법 불공정 조항 해당 여부
- 유사 조항이 무효 판정된 판례
- 실질적 위험 시나리오와 금전적 노출

### Step 6: Judge Agent 호출

`agents/judge.md` sub-agent를 호출합니다.

**입력**: 양측 논증 + playbook의 조직 포지션
**역할**: 양측을 종합하여 최종 판정
- Advocate와 Devil's Advocate 논증의 강점/약점 평가
- Playbook 기준 적용
- GREEN/YELLOW/RED 판정 + 구조화된 근거

**설계 원칙**: Judge는 MCP tool을 사용하지 않습니다. 양측이 제시한 근거의 질만으로 판단합니다.

### Step 7: 판정 결과 출력

## Output Format

```markdown
# Adversarial Review 판정

## 판정: [GREEN / YELLOW / RED]

---

## Advocate 요약
[옹호 논증 핵심 — 왜 이 조항이 유효한지]

## Devil's Advocate 요약
[반박 논증 핵심 — 왜 이 조항이 위험한지]

## Judge 판정 근거
[양측 논증을 종합한 판정 이유]

### Advocate 논증 평가
- **강점**: [...]
- **약점**: [...]

### Devil's Advocate 논증 평가
- **강점**: [...]
- **약점**: [...]

### Playbook 기준 적용
[조직 포지션 대비 분석]

---

## 참조 법령
- [법령명 제X조]: [관련 내용 요약]

## 참조 판례 (가용 시)
- [사건번호]: [판시사항 요약]

## 수정 제안 (YELLOW/RED인 경우)
- **현재 문안**: "[원문]"
- **수정 제안**: "[수정안]"
- **수정 근거**: [왜 이렇게 바꿔야 하는지]

---

<details>
<summary>Advocate 전문 (펼치기)</summary>

[Advocate Agent 전체 출력]

</details>

<details>
<summary>Devil's Advocate 전문 (펼치기)</summary>

[Devil's Advocate Agent 전체 출력]

</details>
```

## Notes

- 3-agent 교차 검증은 단방향 분석 대비 hallucination 리스크를 구조적으로 줄입니다.
- 각 agent는 독립적으로 논증하므로, 한쪽의 오류가 최종 판정에 그대로 전달되지 않습니다.
- Judge의 tool 미사용은 의도적 설계입니다. 논증의 질 자체를 평가하는 데 집중합니다.
- MCP tool (~~korean-law)이 연결되면 Advocate/Devil's Advocate가 실시간 법령·판례를 검색하여 논증 품질이 향상됩니다.

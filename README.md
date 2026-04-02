# legal-kr — 한국법 계약 검토 플러그인

Anthropic 공식 `knowledge-work-plugins/legal` 의 한국법 기준 예시 플러그인.

> **고지사항:** 이 플러그인은 법률 워크플로우를 보조하며 법률 자문을 제공하지 않습니다. 모든 분석 결과는 자격을 갖춘 법률 전문가의 검토를 거친 후 활용하십시오. 기본 playbook(`legal.local.md`)은 대한민국 법률(약관규제법, 민법, 상법, 개인정보 보호법 등)을 기준으로 작성된 예시입니다.

> **예시 프로젝트 안내:** 이 저장소는 예시입니다. `legal.local.md`(playbook)와 `skills/` 내 각 skill은 조직 또는 로펌의 기준에 맞게 자유롭게 수정하여 사용하십시오.

---

## 설치

배포 전 로컬 테스트용:

```bash
git clone https://github.com/cholhwanjung/legal-kr
claude --plugin-dir ./legal-kr
```

---

## 빠른 시작

### 1. Playbook 설정

`legal.local.md`에 조직의 계약 검토 기준을 정의합니다. 이 파일이 모든 skill의 판단 기준이 됩니다.

```markdown
# 한국법 계약 검토 Playbook

## 손해배상 책임 제한
- 기본 입장: 상호 책임 한도 — 최근 12개월 계약 대금
- 수용 범위: 6~24개월
- 에스컬레이션 트리거: 무제한 배상 책임, 간접손해 포함

## 개인정보 처리위탁
- 기본 입장: 개인정보 보호법 제26조 준수 조항 필수
- 에스컬레이션 트리거: 위탁 동의 없는 제3자 제공, 국외이전 보호조치 부재

## 준거법 및 관할
- 우선: 대한민국 법률 / 서울중앙지방법원
- 에스컬레이션 트리거: 외국법 적용, 해외 중재 강제
```

### 2. MCP 서버 연결 (선택 사항)

`.mcp.json`에서 Slack, Box, Atlassian 등 업무 도구와 연동할 수 있습니다.

한국 법령·판례 조회가 필요한 경우 [korean-law-mcp](https://github.com/chrisryugj/korean-law-mcp)와 같은 한국 법률 MCP 서버와 연동할 수 있습니다.

---

## Commands

### `/legal-kr:review-contract-kr` — 계약서 조항별 검토

계약서를 조항별로 분석하여 GREEN/YELLOW/RED 판정을 수행합니다. 약관규제법, 민법, 상법 및 playbook 기준을 적용합니다.

```
/legal-kr:review-contract-kr
```

파일 업로드, URL, 또는 텍스트 직접 입력을 지원합니다.

### `/legal-kr:triage-nda-kr` — NDA 사전 심사

한국법 기준으로 NDA를 신속히 분류합니다. GREEN(즉시 승인), YELLOW(검토 필요), RED(법무 검토 필수).

```
/legal-kr:triage-nda-kr
```

### `/legal-kr:adversarial-review` — 3-agent 심층 분석

단일 조항에 대해 Advocate → Devil's Advocate → Judge 순서로 교차 검증합니다. 단순 playbook 매칭보다 깊은 분석이 필요한 고위험 조항에 사용합니다.

```
/legal-kr:adversarial-review <조항 텍스트>
```

### `/legal-kr:legal-risk-assessment-kr` — 법적 위험도 평가

프로젝트, 거래, 또는 비즈니스 의사결정에 대해 한국 규제 프레임워크 기준 위험도를 체계적으로 평가합니다.

```
/legal-kr:legal-risk-assessment-kr
```

---

## Skills

| Skill | 설명 |
|-------|------|
| `review-contract-kr` | 약관규제법·민법·상법·playbook 기준 조항별 GREEN/YELLOW/RED 스캔 |
| `triage-nda-kr` | 한국법 기준 NDA 사전 심사 및 분류 |
| `adversarial-review` | Advocate·Devil's Advocate·Judge 3-agent 교차 검증 |
| `legal-risk-assessment-kr` | 한국 규제 프레임워크 기반 위험도 평가 프레임워크 |

---

## Adversarial Review 데이터 흐름

```
조항 텍스트 + legal.local.md 컨텍스트
  → Advocate Agent     (조항 유효성 옹호, 판례 근거)
  → Devil's Advocate   (위험 논증, 무효 판례)
  → Judge Agent        (양측 종합 판단 — tool 미사용, 의도적 설계)
  → 최종 판정: GREEN/YELLOW/RED + 근거 + 수정 제안
```

Judge agent는 MCP tool 없이 순수 추론만으로 판정을 내립니다. 외부 DB 조회 없이 독립적인 판단을 보장하기 위한 설계입니다.

---

## 예시 워크플로우

### 계약서 검토

1. 벤더로부터 계약서 수신
2. `/legal-kr:review-contract-kr` 실행 후 파일 업로드
3. 컨텍스트 제공: "당사는 수급인 측, 분기 내 마감, 손해배상 및 IP 조항 집중 검토"
4. 조항별 GREEN/YELLOW/RED 분석 수신
5. YELLOW/RED 항목에 대한 수정안(redline) 확인
6. 고위험 조항은 `/legal-kr:adversarial-review`로 심층 분석

### NDA 사전 심사

1. 영업팀으로부터 신규 거래처 NDA 접수
2. `/legal-kr:triage-nda-kr` 실행 후 NDA 업로드
3. 분류 결과 확인: GREEN(서명 진행), YELLOW(특정 조항 검토), RED(법무팀 검토 필수)
4. GREEN은 직접 승인, YELLOW/RED는 지적된 항목 처리

### 3-Agent 심층 분석

1. 리뷰 중 고위험 조항 발견 (예: 포괄적 IP 양도 조항)
2. `/legal-kr:adversarial-review <조항 텍스트>` 실행
3. Advocate(유효성 옹호) → Devil's Advocate(위험 반박) → Judge(최종 판정) 순서로 분석
4. 균형 잡힌 근거와 구체적인 수정 제안 수신

---

## Playbook 커스터마이징

`legal.local.md`는 모든 skill의 판단 기준입니다. 조직 또는 로펌의 기준에 맞게 교체하십시오.

- **기본 입장**: 조직이 선호하는 계약 조건
- **수용 범위**: 에스컬레이션 없이 동의 가능한 범위
- **에스컬레이션 트리거**: 상위 검토 또는 외부 자문이 필요한 조건

skill 파일(`skills/*/`) 역시 수정 가능합니다. 새로운 분석 기준, 체크리스트, 판정 로직을 추가하거나 기존 내용을 조직 특성에 맞게 조정할 수 있습니다.

---

## MCP 연동

| 카테고리 | 예시 | 용도 |
|----------|------|------|
| 메신저 | Slack | 팀 요청, 알림, 사전 심사 |
| 클라우드 스토리지 | Box, Egnyte | Playbook, 템플릿, 선례 |
| 오피스 | Microsoft 365 | 이메일, 캘린더, 문서 |
| 프로젝트 추적 | Atlassian (Jira/Confluence) | 사건 관리, 태스크 |

연결 설정은 `.mcp.json`에서 관리합니다. 연결된 도구가 없으면 자동으로 수동 확인을 안내합니다.

---

## 파일 구조

```
legal-kr/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터
├── .mcp.json                 # MCP 서버 설정
├── README.md
├── legal.local.md            # Playbook — 조직 기준에 맞게 수정
├── skills/
│   ├── review-contract-kr/   # 조항별 GREEN/YELLOW/RED 스캔
│   ├── triage-nda-kr/        # NDA 사전 심사
│   ├── adversarial-review/   # 3-agent 교차 검증
│   └── legal-risk-assessment-kr/  # 위험도 평가
├── agents/
│   ├── advocate.md           # 조항 유효성 옹호 agent
│   ├── devils-advocate.md    # 조항 위험성 논증 agent
│   └── judge.md              # 양측 종합 판정 agent (tool 미사용)
└── examples/
    └── sample_contract.txt   # 테스트용 예시 계약서
```

---

## 참조 법령

| 법령 | 적용 맥락 |
|------|-----------|
| **약관의 규제에 관한 법률** | 사업자 작성 약관의 불공정성 판단 |
| **민법** 제2조, 제103조, 제104조 | 계약 일반 원칙, 공서양속 위반 |
| **상법** 제46조 이하 | B2B 상거래 특칙 |
| **개인정보 보호법** | 개인정보 처리위탁, 국외이전 |
| **전자상거래법** | SaaS/온라인 서비스 소비자 보호 |
| **하도급법** | 원사업자-수급사업자 불공정 조항 규제 |

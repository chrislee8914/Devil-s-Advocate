# Devil's Advocate Agent

AI가 자신 있게 틀리는 걸 막는 에이전트. 코드, 전략, 계획, 설계를 출하하기 전에 체계적으로 부순다.

---

## 왜 필요한가

AI는 기본값이 낙관적이다. 물어본 것을 정확히 만들지만, 만들어야 하는지는 묻지 않는다. 실패 시나리오를 고민하지 않고, 숨겨진 가정을 드러내지 않으며, 5가지가 프로덕션에서 부서질 때까지 모른다.

Devil's Advocate는 그 역할을 한다.

> AI 생성 코드는 사람이 작성한 것보다 버그가 1.7배 많다. 로직 오류는 75% 더 빈번하다. 자신감과 정확성은 반비례한다.

---

## 두 가지 버전

### `/devils-advocate` — 엔지니어링용

코드, 아키텍처, API 설계, 마이그레이션 계획을 검토한다.

```
/devils-advocate src/auth/ 리뷰해줘
/ux-expert → 페이지 감사 → /devils-advocate → 감사 결과 도전
Claude Code → 기능 구현 → /devils-advocate → 누락 사항 발견
```

**검토 항목:**
- 해피 패스 편향 — 성공 케이스만 구현됐는가
- 실패 모드 — 외부 서비스가 죽으면 어떻게 되는가
- 보안 — 인증만 있고 인가가 없는가
- 확장성 — 트래픽 10배에서 버티는가
- AI 특유 맹점 — 패턴 쏠림, 범위 수용, 근거 없는 자신감

### `/strategy-devils-advocate` — 전략용

신사업, 시장 분석, 사업계획, 권고안을 증거 기반으로 압박 테스트한다.

```
/strategy-devils-advocate materials/ 분석해줘
```

**검토 층위:**
| 층위 | 내용 |
|------|------|
| L0 | 문제 정의 — 맞는 질문을 풀고 있는가 |
| L1 | 가설과 이슈 트리 — 논리 구조가 성립하는가 |
| L2 | 분석과 근거 — 숫자가 견디는가 |
| L3 | So-what 체인 — 사실→시사점→권고 논리가 이어지는가 |
| L4 | 실행 가능성 — 조직이 실제로 할 수 있는가 |
| L5 | 청중 시뮬레이션 — CFO, 사업부장이 실제로 뭐라고 하는가 |

---

## 사용 방법

### 설치

```bash
# devils-advocate (엔지니어링)
npx degit notmanas/claude-code-skills/skills/devils-advocate .claude/skills/devils-advocate

# strategy-devils-advocate는 이 레포에 포함되어 있음
```

### 기본 사용

```
/devils-advocate                          # 무엇을 검토할지 안내
/devils-advocate src/api/auth.ts 리뷰     # 특정 파일
/devils-advocate 이 마이그레이션 계획 검토  # 설명 전달

/strategy-devils-advocate                 # 세션 시작, 자료 확인 후 진행
```

### 다른 스킬과 조합

```
/advisor → 전략 수립 → /strategy-devils-advocate → 전략 압박 테스트
/ux-expert → UX 감사 → /devils-advocate → 감사 도전
Claude → 기능 구현 → /devils-advocate → 엣지 케이스 발굴
```

---

## 출력 형식

### 엔지니어링 (`/devils-advocate`)

```
Steel-man:
  [이 접근이 합리적인 이유 2-3줄]

Concern 1: [한 줄 요약]
Severity: Critical | High | Medium
Framework: [어떤 사고 프레임워크가 이걸 잡았는가]

  What I see:   [구체적 문제, 파일/라인 포함]
  Why it matters: [이대로 출하하면 무슨 일이 생기는가]
  What to do:   [구체적 실행 가능한 권고]

Verdict: Ship it | Ship with changes | Rethink this
```

### 전략 (`/strategy-devils-advocate`)

```
Steelman: [원안의 가장 강한 재구성]
Hinge claim: [무너지면 전체가 붕괴하는 핵심 주장]

[치명 / 중대 / 사소] 제목
- 프레임워크: pre-mortem / inversion / 소크라테스식 탐침
- 발견: [인용 위치 포함한 구체적 사실]
- 문제: [결과]
- 조치: [해소 방법]
- 판정: 확인됨 / 약화됨 / 검증 필요 / 반박됨

Verdict: 그대로 진행 | 조건부 진행 | 재검토
```

---

## 폴더 구조

```
Devil-s-Advocate/
├── materials/          # 클라이언트 자료, 분석 대상 문서 (PDF 등)
├── interviews/         # 인터뷰 노트
├── data/               # 시장 데이터·벤치마크 원본
├── analysis/           # 검증 대상 산출물 (작업 중인 자료)
├── .calibration/       # 사전 확신도 잠금 파일 (자동 생성)
└── .claude/
    └── skills/
        ├── devils-advocate/               # 엔지니어링 버전
        │   ├── SKILL.md
        │   └── references/
        │       ├── questioning-frameworks.md   # Pre-mortem, Inversion, Socratic 등
        │       ├── blind-spots.md              # 엔지니어가 놓치는 11가지 범주
        │       └── ai-blind-spots.md           # AI 특유 맹점 12가지
        └── strategy-devils-advocate/      # 전략 버전
            ├── SKILL.md
            └── references/
                ├── blind-spots.md         # 전략 분석 편향 목록
                └── rubric.md              # 판정 루브릭
```

---

## 원칙

**Steel-man 먼저.** 공격 전에 원안이 합리적인 이유를 만든다. 스트로맨 공격은 노이즈다.

**인용 게이트.** 모든 반론은 구체적 위치를 인용한다. 인용 불가능한 반론은 제기하지 않는다.

**검증 필요는 적극적으로 쓴다.** 지금 자료로 확인 안 되는 것을 `확인됨`으로 처리하는 게 가장 흔한 실패다.

**우려를 만들지 않는다.** 뭔가가 진짜 좋으면 "Ship it"이라고 한다. 철저해 보이기 위해 문제를 만들지 않는다.

**최대 7개.** 우려사항은 심각도 순으로 최대 7개. 많을수록 좋은 게 아니라 패딩이다.

---

## 한계

- **코드를 다시 쓰지 않는다.** 도전하고 권고한다. 구현은 사람(또는 다른 Claude 호출)이 한다.
- **도메인 전문성을 대체하지 않는다.** 구조적·공학적 맹점을 잡는다. 어떤 가격 모델을 쓸지는 모른다.
- **전략 버전은 자료가 있어야 작동한다.** `materials/`가 비어 있으면 모든 판정이 "근거 부족"이 된다. 이건 버그가 아니라 의도된 동작이다.

# Codex Single-Agent Long-Running Harness

## Purpose

이 문서는 현재 macOS 로컬 환경에서 `Codex` 단일 에이전트가 몇 시간 단위 작업을 안정적으로 이어가도록 만드는 최소 하네스 아키텍처를 정의한다.

핵심 목표는 멀티에이전트 오케스트레이션이 아니라 다음 네 가지다.

- 상태를 파일로 남겨 재개 가능하게 만들기
- 계획 없이 구현하지 못하게 만들기
- 검증 없이 완료하지 못하게 만들기
- 사람이 봐야 하는 시점을 명확히 만들기

## Design Inputs

이 설계는 다음 자료를 직접 참고해 정리했다.

- OpenAI, `Harness engineering: leveraging Codex in an agent-first world`, 2026-02-11
  - https://openai.com/index/harness-engineering/
- Anthropic, `Harness design for long-running application development`, 2026-03-24
  - https://www.anthropic.com/engineering/harness-design-long-running-apps
- Mitchell Hashimoto, `My AI Adoption Journey`, 2026-02-05
  - https://mitchellh.com/writing/my-ai-adoption-journey
- Birgitta Boeckeler, `Harness engineering for coding agent users`, 2026-04-02
  - https://martinfowler.com/articles/harness-engineering.html
- 로컬 PDF
  - `from-prompts-to-harnesses-four-years-of-ai-agentic-patterns.pdf`
  - `anthropic-harness-design-long-running-application-development.pdf`
  - `openai-harness-engineering-codex-in-an-agent-first-world.pdf`

## Synthesis

위 자료를 종합하면 단일 Codex 장기 실행 하네스의 본질은 다음으로 수렴한다.

- `AGENTS.md`는 백과사전이 아니라 짧은 진입점이어야 한다
- 실행 상태는 채팅이 아니라 repo-local 파일에 남겨야 한다
- 빠른 결정론적 검증이 먼저이고, 추론적 평가는 그 다음이다
- 에이전트가 반복해서 틀리는 지점은 프롬프트가 아니라 하네스를 고쳐야 한다
- 하네스는 모델 성능 변화에 따라 가벼워질 수 있어야 한다

마지막 항목은 중요한 설계 판단이다. Anthropic은 2026-03-24 글에서 일부 모델에서는 `context reset`이 필수였지만 더 강한 모델에서는 같은 장치가 불필요해졌다고 설명했다. 따라서 현재 아키텍처는 기능을 더하는 구조보다 `필요 없어지면 제거하기 쉬운 구조`를 우선해야 한다.

## Goals

- Codex가 한 저장소에서 긴 작업을 중간에 흐트러지지 않고 이어가게 한다
- 세션이 바뀌어도 현재 상태를 빠르게 복구하게 한다
- `done` 착각을 줄인다
- 테스트, 빌드, 린트, 수동 확인 항목이 빠지지 않게 만든다
- 하네스 규칙을 점진적으로 기계화할 수 있게 한다

## Non-Goals

- 멀티에이전트 분업
- 전사 공통 글로벌 메모리
- 복잡한 외부 오케스트레이터
- 모든 도구를 포괄하는 범용 하네스
- 사람 없는 완전 자율 운영

## Architecture

```text
Human
  -> Codex CLI / Codex App
    -> AGENTS.md
    -> .ai/codex-start.md
    -> .ai/index.md
    -> active plan
    -> local rules
      -> implementation loop
        -> repo code + scripts + docs
        -> deterministic verification
        -> optional inferential review
      -> evaluation
      -> handoff / run-log
```

아키텍처는 다섯 층으로 나눈다.

### 1. Entry Layer

Codex가 처음 읽는 얇은 진입층이다.

- `AGENTS.md`
- `.ai/codex-start.md`
- `.ai/index.md`
- 가장 최신 `plan`
- 필요 시 가장 최신 `handoff`

원칙:

- 루트 `AGENTS.md`는 팀 공용 맵과 프로젝트 기술 가이드의 시작점으로만 둔다
- Codex 전용 실행 규칙은 `.ai/codex-start.md` 같은 repo-local 진입 문서로 분리한다
- `AGENTS.md`는 100줄 안팎의 짧은 실행 규칙 또는 링크만 둔다
- 상세 지식은 `docs/` 또는 `.ai/`의 구체 문서로 링크한다
- "모든 걸 한 파일에 넣기"를 금지한다

### 2. State Layer

세션을 넘겨도 유지되어야 하는 실행 상태를 둔다.

권장 구조:

```text
<repo>/
├── AGENTS.md
├── .ai/
│   ├── codex-start.md
│   ├── index.md
│   ├── local-rules.md
│   ├── run-log.md
│   ├── plans/
│   │   ├── active/
│   │   ├── blocked/
│   │   └── completed/
│   ├── approval-required/
│   ├── handoffs/
│   ├── evaluations/
│   ├── artifacts/
│   │   ├── logs/
│   │   ├── test/
│   │   └── ui/
│   └── scratch/
└── docs/
```

역할:

- `codex-start.md`: Codex 전용 실행 진입점
- `index.md`: 현재 활성 작업과 핵심 문서 포인터
- `local-rules.md`: 아직 표준화되지 않은 로컬 규칙
- `plans/active/`: 현재 작업 계획
- `plans/blocked/`: 기술적 또는 환경적 blocker 상태
- `plans/completed/`: 끝난 계획 보관
- `approval-required/`: 사람 승인 없이는 닫을 수 없는 상태
- `handoffs/`: 재개용 상태 요약
- `evaluations/`: 검증 결과와 종료 판단
- `run-log.md`: 장기 작업의 중요한 결정과 사건
- `artifacts/`: 로그, 스크린샷, 테스트 출력

### 3. Guidance Layer

에이전트가 처음부터 틀린 방향으로 가는 확률을 낮춘다.

- 짧은 `AGENTS.md`
- 현재 스프린트 또는 작업 단위의 `plan`
- 프로젝트 로컬 규칙
- 짧고 반복 가능한 문서 형식

여기서는 설명보다 행동 규칙이 중요하다.

- 시작 시 반드시 읽을 파일
- plan 없는 구현 금지
- verification contract 없는 구현 금지
- done criteria 없는 완료 금지
- evaluation 없는 종료 금지
- human approval status가 `pending`이면 completed 종료 금지
- 장시간 작업 시 handoff 갱신

### 4. Verification Layer

이 층이 하네스의 중심이다.

우선순위:

1. 결정론적 검증
2. 아티팩트 기반 수동 점검
3. 추론적 검토

결정론적 검증 예:

- 테스트
- 빌드
- 린트
- 타입 체크
- 구조 규칙 검사
- 스모크 스크립트

추론적 검토 예:

- 코드 리뷰 프롬프트
- UI 품질 리뷰
- 아키텍처 일관성 검토

원칙:

- 가능한 한 빠른 피드백을 왼쪽으로 당긴다
- 실패 메시지는 에이전트가 고치기 쉽게 작성한다
- 사람이 반복해서 같은 피드백을 주면 문서가 아니라 스크립트나 린트로 승격한다

### 5. Exit Layer

작업의 종료 상태를 제한한다.

- `completed`: done criteria와 evaluation을 모두 만족
- `blocked`: 외부 의존성이나 환경 문제로 중단
- `approval-required`: 수동 검증 승인, 제품 판단, 위험 승인 대기
- `failed`: 시도했지만 목표 미달
- `needs-human`: 즉시 사람 입력이 필요해 다음 구현 단계를 진행할 수 없음

종료 상태는 반드시 `evaluation`과 `handoff`에 반영한다.

## Core Loop

Codex는 항상 아래 순서로 움직이게 설계한다.

1. 현재 상태 읽기
2. 이번 실행 범위와 done criteria 확정
3. 구현 또는 조사 수행
4. 결정론적 검증 실행
5. evaluation 작성
6. handoff 또는 completed 처리

긴 작업에서 특히 중요한 것은 `구현`보다 `4-5-6`이다.

## Mechanical Gates

다음은 최소 강제 규칙이다.

- `Done Criteria` 없는 plan으로 실행 시작 금지
- `Verification Contract` 없는 plan으로 실행 시작 금지
- `Evaluation` 없는 완료 선언 금지
- 빌드, 테스트, 린트 실패 상태에서 완료 처리 금지
- `Human Approval Status = pending` 이면 `approval-required`로 상태 전이
- 30분 이상 지속되거나 세션이 바뀌는 작업은 `handoff` 갱신
- 같은 실패가 두 번 반복되면 `AGENTS.md`, 문서 형식, 스크립트, 린트 중 하나를 수정

## File Contracts

각 파일은 짧고 명확해야 한다.

### `AGENTS.md`

- 팀 공용 문서라는 전제를 유지한다
- 프로젝트 기술 가이드와 민감 경로 규칙의 맵만 둔다
- Codex 전용 루프와 상태 파일 생성 규칙은 넣지 않는다

### `.ai/codex-start.md`

- Codex가 시작 시 읽을 순서
- 기본 실행 루프
- 필수 검증 명령
- 상태 전이 규칙
- 사람에게 물어야 하는 경우

### `plan`

- 범위
- 제약
- 단계
- verification contract
- done criteria
- open questions

`Verification Contract` 최소 필드:

- `Automated Checks`
- `Manual Checks`
- `Skipped Checks`
- `Approval Needed`

`Done Criteria` 최소 필드:

- 기능적 완료 조건
- 검증 완료 조건
- 문서화 완료 조건
- `Human Approval Status`

### `evaluation`

- 무엇을 검증했는지
- 통과 항목
- 실패 항목
- 남은 리스크
- 종료 권고
- `Human Approval Status`

### `handoff`

- 현재 상태
- 이미 내린 결정
- 다음 실행의 첫 세 단계
- 미확인 항목

### `run-log`

- 중요한 결정
- 실패 패턴
- 경로 변경 이유

## Current macOS Decisions

현재 맥북 환경에서는 다음 기본값을 둔다.

- 기본 단위는 `프로젝트별 repo-local 하네스`
- 전역 메모리 데몬을 두지 않는다
- 기본 실행 도구는 `Codex CLI` 또는 `Codex App`
- 기본 셸은 macOS의 `zsh`
- worktree는 병렬 작업 또는 고위험 변경에서만 사용한다
- 모든 상태 파일은 현재 repo 안에 둔다
- Slack, Notes, 메신저 대화를 운영 상태 저장소로 쓰지 않는다

이는 OpenAI가 강조한 `agent legibility`와 Mitchell Hashimoto가 강조한 `AGENTS.md + 실제 도구` 접근을 단일 에이전트 기준으로 축소한 것이다.

## Shared Principles vs Repo-Local State

`~/.codex/ai/` 같은 전역 경로는 선택적으로 둘 수 있지만, 기본 역할은 `reference layer`여야 한다.

- 적합한 것:
  - 공통 원칙
  - bootstrap 예시
  - 재사용 가능한 체크리스트
- 부적합한 것:
  - active plan
  - task handoff
  - evaluation
  - approval 상태
  - run artifacts

이유는 간단하다. 장기 실행 상태는 `repo`, `task`, `worktree`에 강하게 묶이기 때문이다. 전역 경로를 기본 상태 저장소로 쓰면 여러 저장소와 여러 작업의 실행 흔적이 섞여 재개성과 승인 추적이 약해진다.

## Design Rule: Start Single-Agent, Add Layers Only When Forced

Anthropic의 3-agent 구조는 강력했지만, 해당 글 자체도 모델 능력 향상에 따라 일부 장치가 불필요해졌다고 설명한다. 따라서 현재 Codex 하네스의 기본값은 다음이어야 한다.

- 먼저 single-agent로 안정성 문제를 해결한다
- planner, evaluator, reviewer 같은 역할 분리는 나중에 추가한다
- 역할 분리보다 먼저 file state, verify loop, exit gate를 완성한다

즉, `좋은 장기 실행 하네스 = 에이전트 수`가 아니라 `상태 관리 + 검증 루프 + 종료 규칙`이다.

## Success Criteria

- 재실행 시 5분 내에 현재 상태 복구 가능
- handoff만 읽고 다음 작업을 시작할 수 있음
- evaluation 누락이 드물어짐
- 완료 후 재검증에서 빠지는 항목이 줄어듦
- 같은 실패가 반복될수록 prompt가 아니라 harness가 개선됨

## Completion Retrospective

장기 실행 작업 완료 시에는 구현 결과와 검증 결과만 남기지 않는다.

반드시 아래 두 항목을 같이 남긴다.

- 이번 작업이 현재 하네스 아키텍처의 본질적인 방향과 같은 방향인지
- 다음 라운드에서 업그레이드해야 할 항목이 무엇인지

자세한 원칙은 `long-running-task-completion-retrospective.md`를 따른다.

## References

- OpenAI, `Harness engineering: leveraging Codex in an agent-first world`, 2026-02-11
- Anthropic, `Harness design for long-running application development`, 2026-03-24
- Mitchell Hashimoto, `My AI Adoption Journey`, 2026-02-05
- Birgitta Boeckeler, `Harness engineering for coding agent users`, 2026-04-02

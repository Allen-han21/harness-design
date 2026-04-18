# Current MacBook Codex Harness Rollout Plan

## Purpose

현재 맥북 환경에서 `Codex 단일 에이전트 장기실행 하네스`를 가장 작은 비용으로 도입하는 실행 계획이다.

이 계획은 복잡한 플랫폼을 만드는 계획이 아니다. 한 저장소에서 Codex가 오래, 안정적으로, 재개 가능하게 일하게 만드는 계획이다.

## Scope

- 포함:
  - `AGENTS.md`
  - repo-local `ai/`
  - `plan -> implement -> evaluate -> handoff` 루프
  - 검증 명령의 명시와 기계화
  - 실패 패턴의 하네스 승격
- 제외:
  - 멀티에이전트
  - 외부 오케스트레이터
  - 전역 메모리 인프라
  - 조직 전체 공통 플랫폼

## Assumptions

- macOS 로컬에서 Codex CLI 또는 Codex App을 사용한다
- 각 프로젝트는 독립된 git 저장소다
- repo 안에 문서와 스크립트를 둘 수 있다
- 테스트나 빌드 같은 최소 검증 수단을 프로젝트별로 정의할 수 있다

## Phase 1. Pilot One Repo

첫 적용은 한 저장소에만 한다.

작업:

1. 루트에 `AGENTS.md` 추가
2. `ai/` 생성
3. `index`, `local-rules`, `plan`, `handoff`, `evaluation`, `run-log` 기본 문서 생성
4. `AGENTS.md`에는 `ai/codex-start.md` 링크만 두고, 필수 검증 명령은 `ai/` 문서에 명시
5. 활성 작업 하나를 새 하네스로 수행

완료 기준:

- Codex가 시작 시 읽을 파일이 명확하다
- 활성 작업이 `plan`으로 시작한다
- 종료 시 `evaluation`과 `handoff`가 남는다

## Phase 2. Verify Loop Fixation

하네스의 강제력을 만든다.

작업:

1. 빠른 검증 명령을 표준화한다
2. 자주 쓰는 검증을 스크립트로 묶는다
3. 실패 메시지가 에이전트 친화적인지 확인한다
4. `done` 판정을 evaluation 문서 기준으로 통일한다

권장 산출물:

- `scripts/verify.sh` 또는 동등한 명령
- UI 확인이 필요하면 스크린샷 저장 위치
- 로그 수집 위치

완료 기준:

- 사람이 매번 같은 검증 순서를 수동으로 떠올리지 않아도 된다
- Codex가 검증 실패 후 다시 시도할 근거를 즉시 얻는다

## Phase 3. Drift Prevention

반복 실패를 하네스로 흡수한다.

작업:

1. 같은 실수가 두 번 반복되면 원인을 분류한다
2. `AGENTS.md`로 막을 문제인지, 스크립트/린트로 막을 문제인지 결정한다
3. 프로젝트에 반복 가치가 생긴 규칙만 `docs/` 또는 `harness-design`으로 승격한다
4. `run-log`에 실패 패턴과 승격 이유를 남긴다

분류 기준:

- 방향 오류: `AGENTS.md`
- 상태 누락: `ai/index.md`, `handoff`
- 완료 착각: `evaluation`
- 구조 위반: lint, test, structural check
- 인간 판단 필요: escalation rule

완료 기준:

- 반복 실패가 같은 형태로 세 번 나오지 않는다
- rules와 checks가 점점 코드화된다

## Phase 4. Lightweight Measurement

하네스 품질을 측정한다.

추적 지표:

- 재개 시간: 최신 handoff만 읽고 작업 시작까지 걸린 시간
- 검증 누락 수: evaluation 없이 종료한 횟수
- 완료 착각 수: 완료 후 추가 수정이 필요했던 횟수
- 반복 실패 수: 동일 패턴 재발 횟수
- 승격 수: `ai/`에서 `docs/` 또는 `harness-design`으로 승격한 규칙 수

운영 원칙:

- 지표는 단순해야 한다
- 좋은 지표가 더 좋은 작업을 만들지 않으면 버린다

## Recommended First Project Layout

```text
<repo>/
├── AGENTS.md
├── ai/
│   ├── codex-start.md
│   ├── index.md
│   ├── local-rules.md
│   ├── run-log.md
│   ├── plans/
│   │   ├── active/
│   │   └── completed/
│   ├── handoffs/
│   ├── evaluations/
│   └── artifacts/
└── scripts/
    └── verify.sh
```

## Immediate Next Actions

1. 실제로 가장 오래 걸리는 저장소 하나를 pilot repo로 고른다
2. 이 저장소의 `AGENTS.md`는 얇은 팀 공용 안내와 `ai/codex-start.md` 링크만 두도록 만든다
3. `ai/` 기본 문서들을 만든다
4. 검증 명령 하나를 `scripts/verify.sh`로 고정한다
5. 긴 작업 한 번을 끝까지 돌려 본다
6. 반복된 실패만 `harness-design`으로 승격한다

## Kill Criteria

다음 중 하나면 설계를 더 복잡하게 하지 말고 단순화한다.

- 문서만 늘고 검증 누락이 줄지 않는다
- Codex가 읽어야 할 진입점이 두 개 이상으로 갈라진다
- handoff가 너무 길어져 실제 재개 속도를 떨어뜨린다
- 사람 검토가 줄지 않는데 추론적 검토만 계속 추가된다

## Guiding Principle

현재 맥북 환경에서의 올바른 첫 설계는 `로컬 파일 상태 + 검증 루프 + 종료 규칙`이다.

멀티에이전트나 외부 플랫폼은 이 세 가지가 이미 안정화된 뒤에만 고려한다.

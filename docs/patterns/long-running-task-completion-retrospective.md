# Long-Running Task Completion Retrospective

## Purpose

이 문서는 장기 실행 작업이 끝났을 때 무엇을 추가로 기록해야 하는지에 대한 원칙을 정의한다.

핵심 목적은 단순히 `무엇을 바꿨는지`를 넘어서, 이번 작업이 하네스 설계의 본질적인 방향과 같은 방향인지, 다음에 무엇을 업그레이드해야 하는지를 계속 축적하는 것이다.

## Principle

장기 실행 작업 완료 후에는 구현 결과와 검증 결과만 보고하지 않는다.

반드시 아래 두 질문에 답해야 한다.

1. 이번 작업은 현재 하네스 아키텍처의 본질적인 방향과 같은 방향이었는가
2. 이번 작업이 드러낸 다음 업그레이드 항목은 무엇인가

즉, 완료 보고는 아래 세 층을 모두 포함해야 한다.

- implementation result
- verification result
- harness retrospective

## Why This Matters

장기 실행 하네스는 문서를 많이 만드는 것으로 좋아지지 않는다.

다음과 같은 반복 학습이 있어야 좋아진다.

- 실제 작업 수행
- 마찰 지점 관찰
- 하네스 방향성 검토
- 업그레이드 항목 추출
- 규칙 또는 도구로 승격

이 회고 층이 없으면 하네스는 계속 같은 설명을 반복하는 문서가 되기 쉽고, 실제 운영에서 강해지지 않는다.

## Required Completion Questions

장기 실행 작업 완료 시 아래 항목을 반드시 남긴다.

### 1. Direction Check

- 이번 작업이 `상태 관리 + 검증 루프 + 종료 규칙` 중심 설계와 같은 방향이었는가
- 같은 방향이었다면 왜 그런가
- 어긋났다면 어디서 어긋났는가

### 2. Upgrade Items

- 이번 작업에서 드러난 마찰은 무엇인가
- 그 마찰은 문서 문제인가, 검증 문제인가, 상태 관리 문제인가, 종료 규칙 문제인가
- 다음 라운드에서 무엇을 업그레이드해야 하는가

### 3. Human Approval Status

- 이 작업이 사람 승인 없이 닫혀도 되는가
- 수동 검증 승인, 제품 판단, 위험 승인 중 무엇이 남아 있는가
- 승인 대기라면 왜 `completed`가 아니라 `approval-required`여야 하는가

## Output Shape

완료 시 아래 두 섹션을 별도로 두는 것을 권장한다.

### Harness Direction

- aligned / partially aligned / not aligned
- reason

### Upgrade Items

- item
- reason
- expected effect

### Human Approval

- status: `not-required | pending | approved | rejected`
- approver
- remaining checks

## Recording Rules

- 이 회고는 `evaluation` 또는 별도 run 문서에 남긴다
- run 문서로 승격할 가치가 있으면 `runs/{date}_v{n}.md` 형식으로 남긴다
- 예시: `runs/20260418_v1.md`
- 같은 업그레이드 항목이 두 번 이상 반복되면 원칙 문서나 실행 규칙으로 승격한다

## Minimum Standard

장기 실행 작업을 `completed`로 끝낼 때는 다음이 모두 있어야 한다.

- done criteria 충족 여부
- verification 결과
- harness direction 판단
- next upgrade items
- human approval status

하나라도 빠지면 완료 보고 품질이 부족하다고 본다.

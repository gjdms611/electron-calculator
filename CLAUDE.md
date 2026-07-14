# CLAUDE.md

이 프로젝트는 Electron 기반 데스크탑 계산기 앱이다.

기획/요구사항/아키텍처는 [doc/PRD.md](doc/PRD.md), 모듈별 인터페이스/함수 시그니처는
[doc/SPEC.md](doc/SPEC.md), phase별 구현 계획은 [doc/PLAN.md](doc/PLAN.md), phase별 상세 설계
문서는 [doc/design/phase{N}.md](doc/design/) 참고. 각 phase는 사람 검토 후에만 다음 phase로
진행한다.

## SubAgent 활용 지침

이 프로젝트는 `.claude/agents/`에 4개 SubAgent가 정의되어 있다. 코드/문서 작업(특히 PLAN.md의
각 phase 구현)을 수행할 때 아래 SubAgent를 적극 활용한다 — 직접 하지 말고 위임할 것:

1. `doc-consistency-checker` (SubAgent1) — 문서 정합성 검증. 작업 시작 전, PRD/SPEC/PLAN.md와
   현재 코드 상태가 어긋나지 않는지 먼저 호출.
2. `ai-action` (SubAgent2) — 실제 구현(코드/문서 작성·수정) 수행. SubAgent1 결과를 입력으로 사용.
3. `test-verify` (SubAgent3) — 테스트/빌드 실행 검증.
4. `compliance-verify` (SubAgent4) — 라이선스/보안/코딩 표준 준수 검증.

**실행 순서 제약 (필수):** SubAgent1 → SubAgent2 → [SubAgent3, SubAgent4] 순서로 직렬 실행하되,
SubAgent2가 끝난 뒤 SubAgent3과 SubAgent4는 반드시 병렬로 동시 호출한다 (한 메시지에서 두 Agent
호출을 같이 보낼 것). SubAgent3/4는 서로 결과를 기다리지 않는다.

각 PLAN.md phase를 진행할 때도 이 4단계 파이프라인을 그대로 적용한다.

## Git 규칙

이 repo는 git 셋업 완료 상태(origin: GitHub `electron-calculator`, public). 이후 모든 작업은
단위 작업(파일 생성/수정, 기능 구현 등)이 끝날 때마다 사용자 확인 없이 바로 commit & push한다.
단, 아래는 예외 — 반드시 사용자 확인 먼저:
- force push, reset --hard, 브랜치 삭제 등 파괴적 작업
- 커밋 메시지에 시크릿/자격증명이 포함될 가능성이 있는 경우

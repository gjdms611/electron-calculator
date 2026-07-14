# CLAUDE.md

이 프로젝트는 Electron 기반 데스크탑 계산기 앱이다.

기획/요구사항/아키텍처는 [doc/PRD.md](doc/PRD.md), 모듈별 인터페이스/함수 시그니처는
[doc/SPEC.md](doc/SPEC.md), phase별 구현 계획은 [doc/PLAN.md](doc/PLAN.md) 참고. 각 phase는
사람 검토 후에만 다음 phase로 진행한다. 

## Git 규칙

이 repo는 git 셋업 완료 상태(origin: GitHub `electron-calculator`, public). 이후 모든 작업은
단위 작업(파일 생성/수정, 기능 구현 등)이 끝날 때마다 사용자 확인 없이 바로 commit & push한다.
단, 아래는 예외 — 반드시 사용자 확인 먼저:
- force push, reset --hard, 브랜치 삭제 등 파괴적 작업
- 커밋 메시지에 시크릿/자격증명이 포함될 가능성이 있는 경우

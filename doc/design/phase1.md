# Phase 1: 프로젝트 뼈대 (package.json)

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** 의존성 설치까지만. 아직 실행 가능한 앱 없음.

**파일:**
- Create: `package.json`
- Create: `jest.config.js`

- [ ] **Step 1: package.json 작성**

```json
{
  "name": "electron-calculator",
  "version": "0.1.0",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "test": "jest",
    "test:e2e": "playwright test"
  },
  "devDependencies": {
    "electron": "^31.0.0",
    "jest": "^29.7.0",
    "@playwright/test": "^1.45.0"
  }
}
```

- [ ] **Step 2: jest.config.js 작성**

```js
module.exports = { testEnvironment: 'node' };
```

- [ ] **Step 3: 의존성 설치**

Run: `npm install`
Expected: `node_modules` 생성, 에러 없음.

- [ ] **Step 4: 커밋**

```bash
git add package.json package-lock.json jest.config.js
git commit -m "chore: init package.json and jest config"
git push
```

### ⏸ Phase 1 종료 — 사람 검토 대기

`npm install`이 에러 없이 끝났는지 확인 후 승인하면 Phase 2 진행.

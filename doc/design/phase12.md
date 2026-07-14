# Phase 12: E2E 테스트 (Playwright)

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** SPEC.md 7장 "전체 앱" 골든패스를 자동화 E2E 테스트로 고정.

**파일:**
- Create: `playwright.config.js`
- Create: `e2e/calculator.spec.js`

- [ ] **Step 1: playwright.config.js 작성**

```js
module.exports = {
  testDir: './e2e',
  timeout: 30000,
};
```

- [ ] **Step 2: e2e/calculator.spec.js 작성**

```js
const { _electron: electron } = require('playwright');
const { test, expect } = require('@playwright/test');

test('숫자 입력 -> 연산 -> 결과 확인 -> history 반영', async () => {
  const app = await electron.launch({ args: ['.'] });
  const window = await app.firstWindow();

  await window.click('button[data-digit="2"]');
  await window.click('button[data-operator="+"]');
  await window.click('button[data-digit="3"]');
  await window.click('button[data-action="equals"]');

  await expect(window.locator('#display')).toHaveText('5');
  await expect(window.locator('#history li').first()).toHaveText('2 + 3 = 5');

  await app.close();
});
```

- [ ] **Step 3: 테스트 실행**

Run: `npx playwright test`
Expected: PASS (1 test)

- [ ] **Step 4: 커밋**

```bash
git add playwright.config.js e2e/calculator.spec.js
git commit -m "test: add e2e golden path test for calculator"
git push
```

### ⏸ Phase 12 종료 — 사람 검토 대기

E2E 테스트 통과 결과 확인 후 승인하면 Phase 1 범위 구현 완료로 간주.
Phase 2(공학용)/Phase 3(프로그래머용) 착수 여부는 별도로 논의한다.

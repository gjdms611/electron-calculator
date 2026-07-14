# Phase 3: CalculatorEngine — 기본 사칙연산

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** SPEC.md 1장 중 `inputDigit`, `inputOperator`, `equals`만 구현 (+ - × ÷ 단발 연산).
연속계산/에러 처리/AC/CE 등은 이후 Phase.

**파일:**
- Create: `src/CalculatorEngine.js`
- Test: `src/CalculatorEngine.test.js`

- [ ] **Step 1: 실패하는 테스트 작성**

```js
// src/CalculatorEngine.test.js
const { CalculatorEngine } = require('./CalculatorEngine');

test('덧셈 후 equals', () => {
  const e = new CalculatorEngine();
  e.inputDigit('2');
  e.inputOperator('+');
  e.inputDigit('3');
  expect(e.equals()).toEqual({ expression: '2 + 3', result: '5' });
});

test('뺄셈 후 equals', () => {
  const e = new CalculatorEngine();
  e.inputDigit('5');
  e.inputOperator('-');
  e.inputDigit('2');
  expect(e.equals()).toEqual({ expression: '5 - 2', result: '3' });
});

test('곱셈 후 equals', () => {
  const e = new CalculatorEngine();
  e.inputDigit('4');
  e.inputOperator('×');
  e.inputDigit('3');
  expect(e.equals()).toEqual({ expression: '4 × 3', result: '12' });
});

test('나눗셈 후 equals', () => {
  const e = new CalculatorEngine();
  e.inputDigit('6');
  e.inputOperator('÷');
  e.inputDigit('3');
  expect(e.equals()).toEqual({ expression: '6 ÷ 3', result: '2' });
});
```

- [ ] **Step 2: 테스트 실행, 실패 확인**

Run: `npx jest CalculatorEngine`
Expected: FAIL — `Cannot find module './CalculatorEngine'`

- [ ] **Step 3: 최소 구현 (기본 연산만)**

```js
// src/CalculatorEngine.js
class CalculatorEngine {
  constructor() {
    this.display = '0';
    this.pendingOperator = null;
    this.operand = null;
    this.error = null;
  }

  inputDigit(digit) {
    this.display = this.display === '0' ? digit : this.display + digit;
  }

  inputOperator(op) {
    this.operand = Number(this.display);
    this.pendingOperator = op;
    this.display = '0';
  }

  equals() {
    const left = this.operand;
    const right = Number(this.display);
    const op = this.pendingOperator;
    let value;
    if (op === '+') value = left + right;
    else if (op === '-') value = left - right;
    else if (op === '×') value = left * right;
    else if (op === '÷') value = left / right;
    const expression = `${left} ${op} ${right}`;
    this.display = String(value);
    return { expression, result: String(value) };
  }

  getState() {
    return {
      display: this.display,
      pendingOperator: this.pendingOperator,
      operand: this.operand,
      error: this.error,
    };
  }
}

module.exports = { CalculatorEngine };
```

- [ ] **Step 4: 테스트 재실행, 통과 확인**

Run: `npx jest CalculatorEngine`
Expected: PASS (4 tests)

- [ ] **Step 5: 커밋**

```bash
git add src/CalculatorEngine.js src/CalculatorEngine.test.js
git commit -m "feat: CalculatorEngine basic four operations"
git push
```

### ⏸ Phase 3 종료 — 사람 검토 대기

기본 사칙연산 테스트 통과 확인 후 승인하면 Phase 4 진행.

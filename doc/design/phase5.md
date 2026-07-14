# Phase 5: CalculatorEngine — 에러 처리 + 보조 기능

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** 0으로 나누기, 오버플로우, AC/CE/backspace/toggleSign/percent/decimal 추가.

**파일:**
- Modify: `src/CalculatorEngine.js`
- Modify: `src/CalculatorEngine.test.js`

- [ ] **Step 1: 실패하는 테스트 추가**

```js
test('0으로 나누기는 에러', () => {
  const e = new CalculatorEngine();
  e.inputDigit('5');
  e.inputOperator('÷');
  e.inputDigit('0');
  expect(e.equals()).toEqual({ error: '0으로 나눌 수 없습니다' });
  expect(e.getState().error).toBe('0으로 나눌 수 없습니다');
});

test('오버플로우는 에러', () => {
  const e = new CalculatorEngine();
  e.display = String(Number.MAX_SAFE_INTEGER);
  e.inputOperator('+');
  e.display = String(Number.MAX_SAFE_INTEGER);
  expect(e.equals()).toEqual({ error: '오류' });
});

test('clearAll은 전체 초기화', () => {
  const e = new CalculatorEngine();
  e.inputDigit('5');
  e.inputOperator('+');
  e.clearAll();
  expect(e.getState()).toEqual({ display: '0', pendingOperator: null, operand: null, error: null });
});

test('backspace는 마지막 글자 제거', () => {
  const e = new CalculatorEngine();
  e.inputDigit('1');
  e.inputDigit('2');
  e.backspace();
  expect(e.getState().display).toBe('1');
});

test('toggleSign은 부호 반전', () => {
  const e = new CalculatorEngine();
  e.inputDigit('5');
  e.toggleSign();
  expect(e.getState().display).toBe('-5');
});

test('inputPercent는 100으로 나눔', () => {
  const e = new CalculatorEngine();
  e.inputDigit('5');
  e.inputPercent();
  expect(e.getState().display).toBe('0.05');
});
```

- [ ] **Step 2: 테스트 실행, 실패 확인**

Run: `npx jest CalculatorEngine`
Expected: FAIL — `clearAll`, `backspace`, `toggleSign`, `inputPercent` 미정의, 에러 처리 없음.

- [ ] **Step 3: CalculatorEngine 전체 갱신 (에러 처리 + 보조 메서드 추가)**

```js
// src/CalculatorEngine.js 전체 교체
class CalculatorEngine {
  constructor() {
    this.display = '0';
    this.pendingOperator = null;
    this.operand = null;
    this.error = null;
  }

  inputDigit(digit) {
    if (this.error) return;
    this.display = this.display === '0' ? digit : this.display + digit;
  }

  inputDecimal() {
    if (this.error) return;
    if (!this.display.includes('.')) this.display += '.';
  }

  inputOperator(op) {
    if (this.error) return;
    if (this.pendingOperator !== null && this.operand !== null) {
      const result = this._compute(this.operand, Number(this.display), this.pendingOperator);
      if (result.error) return this._setError(result.error);
      this.operand = result.value;
      this.display = String(result.value);
    } else {
      this.operand = Number(this.display);
    }
    this.pendingOperator = op;
    this.display = '0';
  }

  toggleSign() {
    if (this.error) return;
    this.display = String(Number(this.display) * -1);
  }

  inputPercent() {
    if (this.error) return;
    this.display = String(Number(this.display) / 100);
  }

  equals() {
    if (this.pendingOperator === null || this.operand === null) {
      return { expression: this.display, result: this.display };
    }
    const left = this.operand;
    const right = Number(this.display);
    const op = this.pendingOperator;
    const result = this._compute(left, right, op);
    if (result.error) {
      this._setError(result.error);
      return { error: result.error };
    }
    const expression = `${left} ${op} ${right}`;
    this.display = String(result.value);
    this.pendingOperator = null;
    this.operand = null;
    return { expression, result: String(result.value) };
  }

  clearAll() {
    this.display = '0';
    this.pendingOperator = null;
    this.operand = null;
    this.error = null;
  }

  clearEntry() {
    this.display = '0';
  }

  backspace() {
    if (this.error) return;
    this.display = this.display.length > 1 ? this.display.slice(0, -1) : '0';
  }

  getState() {
    return {
      display: this.display,
      pendingOperator: this.pendingOperator,
      operand: this.operand,
      error: this.error,
    };
  }

  _compute(a, b, op) {
    let value;
    if (op === '+') value = a + b;
    else if (op === '-') value = a - b;
    else if (op === '×') value = a * b;
    else if (op === '÷') {
      if (b === 0) return { error: '0으로 나눌 수 없습니다' };
      value = a / b;
    }
    if (!Number.isFinite(value) || Math.abs(value) > Number.MAX_SAFE_INTEGER) {
      return { error: '오류' };
    }
    return { value };
  }

  _setError(message) {
    this.error = message;
    this.pendingOperator = null;
    this.operand = null;
    this.display = '0';
  }
}

module.exports = { CalculatorEngine };
```

- [ ] **Step 4: 테스트 재실행, 전체 통과 확인**

Run: `npx jest CalculatorEngine`
Expected: PASS (12 tests)

- [ ] **Step 5: 커밋**

```bash
git add src/CalculatorEngine.js src/CalculatorEngine.test.js
git commit -m "feat: CalculatorEngine error handling and auxiliary ops"
git push
```

### ⏸ Phase 5 종료 — 사람 검토 대기

에러 처리/보조 기능 테스트 전체 통과 확인 후 승인하면 Phase 6 진행. 이 시점에서
CalculatorEngine은 SPEC.md 1장 API를 전부 만족한다.

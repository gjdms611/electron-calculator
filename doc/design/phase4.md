# Phase 4: CalculatorEngine — 연속계산 + 연산자 교체

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** `2 + 3 + 4 =` 같은 연속 계산, 연산자 재입력 시 교체 동작 추가.

**파일:**
- Modify: `src/CalculatorEngine.js`
- Modify: `src/CalculatorEngine.test.js`

- [ ] **Step 1: 실패하는 테스트 추가**

```js
test('연속 계산: 2 + 3 + 4 =', () => {
  const e = new CalculatorEngine();
  e.inputDigit('2');
  e.inputOperator('+');
  e.inputDigit('3');
  e.inputOperator('+'); // 직전 연산 즉시 실행: 2+3=5를 새 operand로
  e.inputDigit('4');
  expect(e.equals().result).toBe('9');
});

test('연산자 두 번 연속 입력은 마지막 것으로 교체', () => {
  const e = new CalculatorEngine();
  e.inputDigit('2');
  e.inputOperator('+');
  e.inputOperator('-'); // 교체됨, 에러 아님
  e.inputDigit('3');
  expect(e.equals()).toEqual({ expression: '2 - 3', result: '-1' });
});
```

- [ ] **Step 2: 테스트 실행, 실패 확인**

Run: `npx jest CalculatorEngine`
Expected: FAIL — 두 번째 `inputOperator` 호출 시 기존 `equals` 로직이 반영 안 되어 결과 불일치.

- [ ] **Step 3: inputOperator 수정 (연속계산/교체 반영)**

```js
// src/CalculatorEngine.js 의 inputOperator만 교체
inputOperator(op) {
  if (this.pendingOperator !== null && this.operand !== null) {
    const result = this._computeRaw(this.operand, Number(this.display), this.pendingOperator);
    this.operand = result;
    this.display = String(result);
  } else {
    this.operand = Number(this.display);
  }
  this.pendingOperator = op;
  this.display = '0';
}

_computeRaw(a, b, op) {
  if (op === '+') return a + b;
  if (op === '-') return a - b;
  if (op === '×') return a * b;
  if (op === '÷') return a / b;
}
```

`equals()`의 계산 부분도 `_computeRaw` 재사용하도록 정리:

```js
equals() {
  const left = this.operand;
  const right = Number(this.display);
  const op = this.pendingOperator;
  const value = this._computeRaw(left, right, op);
  const expression = `${left} ${op} ${right}`;
  this.display = String(value);
  return { expression, result: String(value) };
}
```

- [ ] **Step 4: 테스트 재실행, 통과 확인**

Run: `npx jest CalculatorEngine`
Expected: PASS (6 tests)

- [ ] **Step 5: 커밋**

```bash
git add src/CalculatorEngine.js src/CalculatorEngine.test.js
git commit -m "feat: CalculatorEngine chained calculation and operator replace"
git push
```

### ⏸ Phase 4 종료 — 사람 검토 대기

연속계산/연산자 교체 테스트 통과 확인 후 승인하면 Phase 5 진행.

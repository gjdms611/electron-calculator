# Phase 10: KeyboardHandler 연결

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** SPEC.md 4장대로 키보드 입력 지원 추가.

**파일:**
- Modify: `renderer/renderer.js`

- [ ] **Step 1: renderer.js에 keydown 리스너 추가**

```js
// renderer.js 파일 맨 아래, render() 호출 전에 추가
const KEY_OPERATOR_MAP = { '*': '×', '/': '÷', '+': '+', '-': '-' };

window.addEventListener('keydown', (ev) => {
  if (/^[0-9]$/.test(ev.key)) engine.inputDigit(ev.key);
  else if (ev.key === '.') engine.inputDecimal();
  else if (KEY_OPERATOR_MAP[ev.key]) engine.inputOperator(KEY_OPERATOR_MAP[ev.key]);
  else if (ev.key === 'Enter' || ev.key === '=') engine.equals();
  else if (ev.key === 'Escape') engine.clearAll();
  else if (ev.key === 'Delete') engine.clearEntry();
  else if (ev.key === 'Backspace') engine.backspace();
  else if (ev.key === '%') engine.inputPercent();
  else return;
  render();
});
```

- [ ] **Step 2: 수동 확인**

Run: `npm start`
Expected: 키보드로 `2` `+` `3` `Enter` 입력 시 display에 `5` 표시. `Escape` 입력 시 `0`으로 초기화.

- [ ] **Step 3: 커밋**

```bash
git add renderer/renderer.js
git commit -m "feat: add keyboard input support"
git push
```

### ⏸ Phase 10 종료 — 사람 검토 대기

키보드 입력이 버튼 클릭과 동일하게 동작하는지 확인 후 승인하면 Phase 11 진행.

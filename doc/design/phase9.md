# Phase 9: Renderer — 버튼 클릭과 CalculatorEngine 연결

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

**목표:** 버튼 클릭 시 CalculatorEngine 동작 + display 갱신. history/키보드는 다음 Phase.

**파일:**
- Modify: `renderer/index.html` (script 태그 추가)
- Modify: `main.js` (nodeIntegration 허용)
- Create: `renderer/renderer.js`

- [ ] **Step 1: main.js의 webPreferences 수정**

`renderer.js`가 `require('../src/CalculatorEngine')`를 직접 쓰므로 renderer에서 Node 모듈
require가 가능해야 한다.

```js
// main.js의 BrowserWindow 옵션 수정
webPreferences: {
  preload: path.join(__dirname, 'preload.js'),
  contextIsolation: false,
  nodeIntegration: true,
},
```

- [ ] **Step 2: renderer/index.html에 script 태그 추가**

`</body>` 직전에 추가:
```html
<script src="renderer.js"></script>
```

- [ ] **Step 3: renderer/renderer.js 작성 (버튼 클릭 -> 엔진 -> display만)**

```js
const { CalculatorEngine } = require('../src/CalculatorEngine');

const engine = new CalculatorEngine();
const displayEl = document.getElementById('display');
const errorEl = document.getElementById('error');

function render() {
  const state = engine.getState();
  displayEl.textContent = state.display;
  errorEl.textContent = state.error || '';
}

document.getElementById('buttons').addEventListener('click', (ev) => {
  const btn = ev.target.closest('button');
  if (!btn) return;
  if (btn.dataset.digit !== undefined) engine.inputDigit(btn.dataset.digit);
  else if (btn.dataset.operator) engine.inputOperator(btn.dataset.operator);
  else if (btn.dataset.action === 'clearAll') engine.clearAll();
  else if (btn.dataset.action === 'clearEntry') engine.clearEntry();
  else if (btn.dataset.action === 'toggleSign') engine.toggleSign();
  else if (btn.dataset.action === 'percent') engine.inputPercent();
  else if (btn.dataset.action === 'decimal') engine.inputDecimal();
  else if (btn.dataset.action === 'equals') engine.equals();
  render();
});

render();
```

- [ ] **Step 4: 수동 확인**

Run: `npm start`
Expected: `2` `+` `3` `=` 클릭 시 display에 `5` 표시. `5` `÷` `0` `=` 클릭 시 error 영역에
"0으로 나눌 수 없습니다" 표시.

- [ ] **Step 5: 커밋**

```bash
git add renderer/index.html renderer/renderer.js main.js
git commit -m "feat: wire calculator buttons to CalculatorEngine"
git push
```

### ⏸ Phase 9 종료 — 사람 검토 대기

버튼 클릭으로 계산이 실제로 되는지 확인 후 승인하면 Phase 10 진행.

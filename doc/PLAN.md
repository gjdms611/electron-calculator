# 계산기 모듈 (Phase 1) 구현 계획

> **실행 방식:** Phase 단위로 진행. **각 Phase가 끝나면 반드시 멈추고 사람이 결과를 검토한 뒤
> 승인해야 다음 Phase로 넘어간다.** 승인 없이 다음 Phase를 시작하지 않는다.

**목표:** PRD/SPEC Phase 1 범위(기본 사칙연산 + history) 계산기를 Electron 앱으로 구현한다.

**아키텍처:** Electron main(HistoryStore, IPC) + renderer(CalculatorEngine, Display,
KeyboardHandler, HistoryPanel). 상세는 [SPEC.md](SPEC.md) 참고.

**기술 스택:** Electron, Node.js, Jest(단위 테스트), Playwright(E2E)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 이 계획에 포함하지 않는다.

---

## Phase 1: 프로젝트 뼈대 (package.json)

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

---

## Phase 2: Electron 빈 창 실행

**목표:** 빈 창이 뜨는 최소 Electron 앱.

**파일:**
- Create: `main.js`
- Create: `preload.js`
- Create: `renderer/index.html`

- [ ] **Step 1: main.js 작성**

```js
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
  const win = new BrowserWindow({
    width: 360,
    height: 560,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,
      nodeIntegration: false,
    },
  });
  win.loadFile(path.join(__dirname, 'renderer', 'index.html'));
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

- [ ] **Step 2: preload.js 빈 골격 작성**

```js
const { contextBridge } = require('electron');
contextBridge.exposeInMainWorld('api', {});
```

- [ ] **Step 3: renderer/index.html 빈 골격 작성**

```html
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>Calculator</title></head>
<body><div id="app">calculator loading...</div></body>
</html>
```

- [ ] **Step 4: 실행 확인**

Run: `npm start`
Expected: 창이 뜨고 "calculator loading..." 텍스트 표시. 콘솔 에러 없음. 확인 후 창 닫기.

- [ ] **Step 5: 커밋**

```bash
git add main.js preload.js renderer/index.html
git commit -m "chore: scaffold electron app skeleton"
git push
```

### ⏸ Phase 2 종료 — 사람 검토 대기

빈 창이 정상적으로 뜨는지 확인 후 승인하면 Phase 3 진행.

---

## Phase 3: CalculatorEngine — 기본 사칙연산

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

---

## Phase 4: CalculatorEngine — 연속계산 + 연산자 교체

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

---

## Phase 5: CalculatorEngine — 에러 처리 + 보조 기능

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

---

## Phase 6: HistoryStore

**목표:** SPEC.md 2장대로 history 파일 read/write 순수 함수 구현. IPC 연결은 다음 Phase.

**파일:**
- Create: `src/HistoryStore.js`
- Create: `src/HistoryStore.test.js`

- [ ] **Step 1: 실패하는 테스트 작성**

```js
// src/HistoryStore.test.js
const fs = require('fs');
const os = require('os');
const path = require('path');
const { loadHistory, appendHistory } = require('./HistoryStore');

let tmpFile;
beforeEach(() => {
  tmpFile = path.join(os.tmpdir(), `history-test-${Date.now()}-${Math.random()}.json`);
});
afterEach(() => {
  if (fs.existsSync(tmpFile)) fs.unlinkSync(tmpFile);
});

test('파일 없으면 빈 배열 반환', () => {
  expect(loadHistory(tmpFile)).toEqual([]);
});

test('appendHistory 후 loadHistory로 조회', () => {
  const entry = appendHistory(tmpFile, '2 + 3', '5');
  expect(entry.expression).toBe('2 + 3');
  expect(entry.result).toBe('5');
  expect(typeof entry.id).toBe('string');
  expect(typeof entry.timestamp).toBe('string');

  const list = loadHistory(tmpFile);
  expect(list).toHaveLength(1);
  expect(list[0].id).toBe(entry.id);
});

test('파일 내용이 깨졌으면 빈 배열 반환 (fallback)', () => {
  fs.writeFileSync(tmpFile, '{not valid json');
  expect(loadHistory(tmpFile)).toEqual([]);
});
```

- [ ] **Step 2: 테스트 실행, 실패 확인**

Run: `npx jest HistoryStore`
Expected: FAIL — `Cannot find module './HistoryStore'`

- [ ] **Step 3: HistoryStore 구현**

```js
// src/HistoryStore.js
const fs = require('fs');
const crypto = require('crypto');

function loadHistory(filePath) {
  if (!fs.existsSync(filePath)) return [];
  try {
    const raw = fs.readFileSync(filePath, 'utf-8');
    const parsed = JSON.parse(raw);
    return Array.isArray(parsed) ? parsed : [];
  } catch (err) {
    console.error('HistoryStore: failed to read/parse history file', err);
    return [];
  }
}

function appendHistory(filePath, expression, result) {
  const entry = {
    id: crypto.randomUUID(),
    expression,
    result,
    timestamp: new Date().toISOString(),
  };
  const list = loadHistory(filePath);
  list.unshift(entry);
  try {
    fs.writeFileSync(filePath, JSON.stringify(list, null, 2));
  } catch (err) {
    console.error('HistoryStore: failed to write history file', err);
  }
  return entry;
}

module.exports = { loadHistory, appendHistory };
```

- [ ] **Step 4: 테스트 재실행, 통과 확인**

Run: `npx jest HistoryStore`
Expected: PASS (3 tests)

- [ ] **Step 5: 커밋**

```bash
git add src/HistoryStore.js src/HistoryStore.test.js
git commit -m "feat: implement HistoryStore read/write with fallback"
git push
```

### ⏸ Phase 6 종료 — 사람 검토 대기

HistoryStore 단위 테스트 통과 확인 후 승인하면 Phase 7 진행.

---

## Phase 7: IPC 연결 (main/preload)

**목표:** SPEC.md 3장 IPC 채널을 HistoryStore와 연결.

**파일:**
- Modify: `main.js`
- Modify: `preload.js`

- [ ] **Step 1: main.js에 IPC 핸들러 등록**

```js
// main.js 상단 require에 추가
const { app, BrowserWindow, ipcMain } = require('electron');
const path = require('path');
const { loadHistory, appendHistory } = require('./src/HistoryStore');

// createWindow 함수 아래에 추가
function getHistoryFilePath() {
  return path.join(app.getPath('userData'), 'history.json');
}

ipcMain.handle('history:get', () => {
  return loadHistory(getHistoryFilePath());
});

ipcMain.handle('history:add', (_event, { expression, result }) => {
  return appendHistory(getHistoryFilePath(), expression, result);
});
```

- [ ] **Step 2: preload.js에서 renderer로 노출**

```js
// preload.js
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('api', {
  getHistory: () => ipcRenderer.invoke('history:get'),
  addHistory: (expression, result) => ipcRenderer.invoke('history:add', { expression, result }),
});
```

- [ ] **Step 3: 수동 확인**

Run: `npm start`
Expected: 창 정상 실행, 콘솔 에러 없음 (아직 UI에서 호출 안 하므로 동작 변화는 없음, 크래시만 없으면 됨).

- [ ] **Step 4: 커밋**

```bash
git add main.js preload.js
git commit -m "feat: wire IPC channels history:get/history:add to HistoryStore"
git push
```

### ⏸ Phase 7 종료 — 사람 검토 대기

앱이 에러 없이 실행되는지, IPC 채널명/payload가 SPEC.md 3장과 일치하는지 확인 후 승인하면
Phase 8 진행.

---

## Phase 8: UI 정적 골격 (버튼/화면 레이아웃)

**목표:** 버튼과 display, history 리스트를 화면에 배치만 함. 아직 클릭해도 동작 없음.

**파일:**
- Modify: `renderer/index.html`
- Create: `renderer/style.css`

- [ ] **Step 1: renderer/index.html 교체 (버튼 레이아웃 + history 패널)**

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Calculator</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="app">
    <div id="display">0</div>
    <div id="error"></div>
    <div id="buttons">
      <button data-action="clearAll">AC</button>
      <button data-action="clearEntry">CE</button>
      <button data-action="toggleSign">±</button>
      <button data-action="percent">%</button>
      <button data-digit="7">7</button>
      <button data-digit="8">8</button>
      <button data-digit="9">9</button>
      <button data-operator="÷">÷</button>
      <button data-digit="4">4</button>
      <button data-digit="5">5</button>
      <button data-digit="6">6</button>
      <button data-operator="×">×</button>
      <button data-digit="1">1</button>
      <button data-digit="2">2</button>
      <button data-digit="3">3</button>
      <button data-operator="-">-</button>
      <button data-digit="0">0</button>
      <button data-action="decimal">.</button>
      <button data-action="equals">=</button>
      <button data-operator="+">+</button>
    </div>
    <ul id="history"></ul>
  </div>
</body>
</html>
```

- [ ] **Step 2: renderer/style.css 작성**

```css
body { font-family: sans-serif; }
#display { font-size: 2em; text-align: right; padding: 8px; }
#error { color: red; min-height: 1.2em; text-align: right; }
#buttons { display: grid; grid-template-columns: repeat(4, 1fr); gap: 4px; }
#buttons button { font-size: 1.2em; padding: 12px; }
#history { list-style: none; padding: 0; max-height: 120px; overflow-y: auto; }
#history li { cursor: pointer; padding: 4px; }
```

- [ ] **Step 3: 수동 확인**

Run: `npm start`
Expected: 버튼 4x5 레이아웃, display, 빈 history 리스트가 보임. 클릭해도 반응 없는 게 정상.

- [ ] **Step 4: 커밋**

```bash
git add renderer/index.html renderer/style.css
git commit -m "feat: static calculator UI layout"
git push
```

### ⏸ Phase 8 종료 — 사람 검토 대기

레이아웃이 의도대로 보이는지 확인 후 승인하면 Phase 9 진행.

---

## Phase 9: Renderer — 버튼 클릭과 CalculatorEngine 연결

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

---

## Phase 10: KeyboardHandler 연결

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

---

## Phase 11: HistoryPanel 연결

**목표:** SPEC.md 6장대로 history 조회/추가/클릭 재사용 연결.

**파일:**
- Modify: `renderer/renderer.js`

- [ ] **Step 1: renderer.js에 history 관련 함수 추가, equals 호출부 수정**

```js
// renderer.js 상단에 추가
const historyEl = document.getElementById('history');

async function loadHistoryPanel() {
  const list = await window.api.getHistory();
  renderHistoryList(list);
}

function renderHistoryList(list) {
  historyEl.innerHTML = '';
  for (const entry of list) {
    const li = document.createElement('li');
    li.textContent = `${entry.expression} = ${entry.result}`;
    li.addEventListener('click', () => {
      engine.clearAll();
      engine.display = entry.result;
      render();
    });
    historyEl.appendChild(li);
  }
}

async function handleEquals() {
  const r = engine.equals();
  render();
  if (!r.error) {
    await window.api.addHistory(r.expression, r.result);
    const current = await window.api.getHistory();
    renderHistoryList(current);
  }
}
```

버튼 클릭 리스너에서 `else if (btn.dataset.action === 'equals') engine.equals();` 를
`else if (btn.dataset.action === 'equals') return handleEquals();` 로 교체.

keydown 리스너에서 `else if (ev.key === 'Enter' || ev.key === '=') engine.equals();` 를
`else if (ev.key === 'Enter' || ev.key === '=') return handleEquals();` 로 교체.

파일 맨 아래 `render();` 다음 줄에 `loadHistoryPanel();` 추가.

- [ ] **Step 2: 수동 확인 (골든 패스)**

Run: `npm start`
Expected:
1. `2` `+` `3` `=` 클릭 → display에 `5`, history 리스트에 `2 + 3 = 5` 추가.
2. history 항목 클릭 → display에 해당 결과 반영.
3. 앱 재시작 후에도 history 리스트에 이전 기록 남아있음.

- [ ] **Step 3: 커밋**

```bash
git add renderer/renderer.js
git commit -m "feat: wire history panel to IPC and calculation results"
git push
```

### ⏸ Phase 11 종료 — 사람 검토 대기

Step 2 골든 패스 전부 확인 후 승인하면 Phase 12 진행. 이 시점에서 Phase 1 범위 UI는 완성.

---

## Phase 12: E2E 테스트 (Playwright)

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

# Phase 8: UI 정적 골격 (버튼/화면 레이아웃)

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

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

# Phase 11: HistoryPanel 연결

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

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

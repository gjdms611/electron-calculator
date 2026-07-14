# Phase 7: IPC 연결 (main/preload)

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

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

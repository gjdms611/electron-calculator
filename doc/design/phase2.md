# Phase 2: Electron 빈 창 실행

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

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

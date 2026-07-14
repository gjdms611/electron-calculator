# Phase 6: HistoryStore

> 전체 계획: [PLAN.md](../PLAN.md) · 요구사항: [PRD.md](../PRD.md) · 인터페이스: [SPEC.md](../SPEC.md)

## 전역 제약

- 데이터 저장은 SPEC.md 2장대로 `userData/history.json` 사용, DB 도입 금지.
- 연산자 교체/0으로 나누기/오버플로우 처리는 SPEC.md 1장 "계산 규칙" 그대로 구현 (임의 변경 금지).
- IPC 채널명은 SPEC.md 3장 표(`history:add`, `history:get`) 그대로 사용.
- Phase 2(공학용)/Phase 3(프로그래머용) 관련 코드는 포함하지 않는다.

---

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

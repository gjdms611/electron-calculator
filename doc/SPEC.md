# 계산기 모듈 스펙 (SPEC)

Phase 1 구현 대상 모듈의 인터페이스/동작 정의. 배경/범위/비범위는 [PRD.md](PRD.md) 참고.

## 1. CalculatorEngine (renderer, 순수 로직)

상태를 가진 클래스. UI/IPC와 독립적, 단위 테스트 가능.

### State

```ts
type EngineState = {
  display: string;        // 현재 화면에 보여줄 문자열
  pendingOperator: '+' | '-' | '×' | '÷' | null;
  operand: number | null; // 연산자 입력 전 저장된 값
  error: string | null;   // 에러 메시지, 없으면 null
}
```

### API

| 메서드 | 시그니처 | 동작 |
|---|---|---|
| `inputDigit` | `(digit: string) => void` | 현재 입력에 숫자 append. `error` 상태면 무시(또는 AC 후 입력, 8번 항목 참고) |
| `inputDecimal` | `() => void` | 소수점 추가. 이미 소수점 있으면 무시 |
| `inputOperator` | `(op: '+'\|'-'\|'×'\|'÷') => void` | 현재 값을 `operand`로 저장, `pendingOperator` 설정. 이미 `pendingOperator` 있으면 마지막 것으로 교체 (PRD 7장 정책) |
| `toggleSign` | `() => void` | 현재 표시값 부호 반전 |
| `inputPercent` | `() => void` | 현재 표시값을 `/100` 처리 |
| `equals` | `() => { expression: string, result: string } \| { error: string }` | `pendingOperator`와 `operand`로 계산 실행. 성공 시 history용 결과 반환, 실패 시 에러 반환 |
| `clearAll` (AC) | `() => void` | 전체 상태 초기화 |
| `clearEntry` (CE) | `() => void` | 현재 입력값만 초기화 (`operand`, `pendingOperator` 유지) |
| `backspace` | `() => void` | `display` 마지막 글자 제거 |
| `getState` | `() => EngineState` | 현재 상태 조회 (Display 렌더링용) |

### 계산 규칙

- 연속 계산: `2 + 3 + 4 =` → `equals` 없이 `+` 재입력 시 직전 연산을 즉시 실행하고 결과를 새 `operand`로 사용.
- 0으로 나누기: `÷` 연산에서 두 번째 피연산자가 0이면 `equals`가 `{ error: "0으로 나눌 수 없습니다" }` 반환, 엔진 상태는 `clearAll`과 동일하게 초기화.
- 오버플로우: 계산 결과가 `Number.MAX_SAFE_INTEGER` 초과 또는 `!Number.isFinite(result)` → `{ error: "오류" }` 반환, 상태 초기화.
- 연산자 두 번 연속 입력(`inputOperator` 연달아 호출): 에러 아님, 마지막 연산자로 교체.

## 2. HistoryStore (main process)

### 저장 위치

`app.getPath('userData')/history.json` — JSON 배열, 없으면 최초 호출 시 `[]`로 생성.

### 데이터 타입

```ts
type HistoryEntry = {
  id: string;         // uuid v4
  expression: string; // 예: "2 + 3"
  result: string;
  timestamp: string;  // ISO-8601
}
```

### API (main process 내부 함수, IPC 핸들러가 호출)

| 함수 | 시그니처 | 동작 |
|---|---|---|
| `loadHistory` | `() => HistoryEntry[]` | 파일 read. 파일 없음/파싱 실패 시 `[]` 반환 + 콘솔 에러 로그 (PRD 7장: 치명적 실패 아님) |
| `appendHistory` | `(expression: string, result: string) => HistoryEntry` | 새 엔트리 생성 후 파일에 append, write 실패 시 콘솔 에러 로그만 남기고 반환값은 정상 제공 |

## 3. IPC 계약

| 채널 | 방향 | payload | 응답 |
|---|---|---|---|
| `history:add` | renderer → main (invoke) | `{ expression: string, result: string }` | `HistoryEntry` (추가된 항목) |
| `history:get` | renderer → main (invoke) | 없음 | `HistoryEntry[]` (전체 기록, 최신순) |

## 4. KeyboardHandler (renderer)

키 → `CalculatorEngine` 메서드 매핑. `keydown` 이벤트 리스너로 등록.

| 키 | 동작 |
|---|---|
| `0-9` | `inputDigit(key)` |
| `.` | `inputDecimal()` |
| `+` `-` `*` `/` | `inputOperator()` (`*`→`×`, `/`→`÷` 매핑) |
| `Enter` `=` | `equals()` 호출 후 결과 IPC로 `history:add` |
| `Escape` | `clearAll()` |
| `Delete` | `clearEntry()` |
| `Backspace` | `backspace()` |
| `%` | `inputPercent()` |

## 5. Display (renderer)

`CalculatorEngine.getState()` 결과를 렌더링만 하는 순수 뷰. 자체 상태 없음.

- `state.error`가 있으면 에러 메시지 표시, 다른 값 무시.
- 없으면 `state.display` 표시.

## 6. HistoryPanel (renderer)

- 앱 시작 시 `history:get`으로 초기 목록 로드.
- `equals()` 성공 후 `history:add` 호출 결과(반환된 `HistoryEntry`)를 목록 맨 위에 추가 (재조회 없이 로컬 갱신).
- 항목 클릭 시 해당 `result`를 `CalculatorEngine`에 새 입력값으로 반영 (`display`에 로드, 연산자/상태는 초기화).

## 7. 테스트 대상 매핑

| 모듈 | 테스트 종류 | 커버 대상 |
|---|---|---|
| `CalculatorEngine` | Jest 단위 테스트 | 사칙연산, 소수점, 연속계산, 0으로 나누기, 오버플로우, 연산자 교체 |
| `HistoryStore` | Jest 단위 테스트 | read/write, 파일 없음, 파싱 실패 fallback |
| 전체 앱 | Playwright E2E | 숫자 입력 → 연산 → `=` → 결과 확인 → history 반영 확인 |

# 계산기 모듈 PRD

## 1. 개요

Windows 기본 계산기 수준의 데스크탑 계산기 앱. Electron 기반, HTML/CSS/JS로 UI 구현.
전체 비전은 기본/공학용/프로그래머용 모드 + 계산 기록(history)을 모두 갖춘 계산기이나,
본 PRD의 구현 대상은 **Phase 1 (기본 모드 + history)** 로 한정한다. 공학용/프로그래머용은
Phase 2/3으로 범위만 명시하고 이번 구현에 포함하지 않는다.

## 2. 목표 / 성공 기준

- 사칙연산을 실수 입력값에 대해 정확히 계산한다.
- 계산 기록이 앱 재실행 후에도 유지된다.
- 마우스 클릭과 키보드 입력 모두로 조작 가능하다.
- 잘못된 입력(0으로 나누기 등)에서 앱이 죽지 않고 에러 상태로 복구된다.

## 3. 범위

### Phase 1 (이번 구현 대상)

- 사칙연산: `+`, `-`, `×`, `÷`
- 소수점 입력
- 부호 변경 (`±`)
- 퍼센트 (`%`)
- 전체 지우기 (`AC`) / 현재 입력 지우기 (`CE`)
- 연속 계산 (예: `2 + 3 + 4 =`)
- 계산 기록(history): 계산식 + 결과 목록, JSON 파일로 영구 저장, 클릭 시 결과 재사용
- 키보드 입력: 숫자키, `+ - * /`, `Enter`(=), `Esc`(AC), `Backspace`(한 글자 지우기)

### Phase 2 (범위만 명시, 미구현)

- 공학용 모드: `sin/cos/tan`, `log/ln`, 거듭제곱, 제곱근, 괄호, 상수(`π`, `e`)

### Phase 3 (범위만 명시, 미구현)

- 프로그래머용 모드: 2/8/10/16진수 변환, 비트 연산(`AND/OR/XOR/NOT`, shift)

### 비범위 (Out of scope)

- 다국어 UI
- 클라우드 동기화
- 모바일/웹 배포

## 4. 아키텍처

Electron 2-프로세스 구조:

- **Main process**: 앱 lifecycle 관리, `HistoryStore`를 통한 history JSON 파일 read/write, IPC 핸들러 제공.
- **Renderer process**: HTML/CSS/JS UI. `CalculatorEngine`(순수 계산 로직), `Display`, `HistoryPanel`, `KeyboardHandler`로 구성.
- 통신: Renderer가 `=` 입력 시 IPC(`history:add`)로 Main에 계산 결과 전달 → Main이 JSON 파일에 append → Main이 최신 history를 Renderer에 전달(`history:updated`) → `HistoryPanel` 갱신.

```
[KeyboardHandler / 버튼 클릭]
        │
        ▼
[CalculatorEngine] ──(결과 갱신)──▶ [Display]
        │ (= 입력 시)
        ▼ IPC: history:add
[Main process: HistoryStore] ──▶ history.json (userData 경로)
        │ IPC: history:updated
        ▼
   [HistoryPanel]
```

## 5. 컴포넌트 정의

| 컴포넌트 | 위치 | 역할 | 의존성 |
|---|---|---|---|
| `CalculatorEngine` | renderer | 사칙연산, 소수점, 부호, %, 연속계산, 에러 판정(0으로 나누기 등) 처리. UI와 독립된 순수 로직 | 없음 |
| `Display` | renderer | 현재 입력값/계산 결과/에러 메시지 표시 | `CalculatorEngine` |
| `HistoryPanel` | renderer | history 목록 표시, 항목 클릭 시 해당 결과를 `Display`에 재사용 | IPC |
| `KeyboardHandler` | renderer | 키보드 입력을 `CalculatorEngine` 호출로 매핑 | `CalculatorEngine` |
| `HistoryStore` | main | history.json read/write, IPC 핸들러(`history:add`, `history:get`) | Node `fs` |

## 6. 데이터 모델

history 항목 (JSON 배열, `userData/history.json`):

```json
{
  "id": "uuid",
  "expression": "2 + 3",
  "result": "5",
  "timestamp": "ISO-8601"
}
```

## 7. 에러 처리

- 0으로 나누기 → Display에 "0으로 나눌 수 없습니다" 표시, 입력 상태 초기화(AC와 동일 효과).
- 숫자 오버플로우(자바스크립트 `Number` 안전 정수 범위 초과) → Display에 "오류" 표시, 초기화.
- 잘못된 연산자 순서 입력(예: `+` 두 번 연속) → 마지막 연산자만 유효한 것으로 대체 처리(에러 아님).
- history.json 파일 read/write 실패(권한, 손상) → 콘솔 에러 로그만 남기고 앱은 history 없이 정상 동작 계속(치명적 실패 아님).

## 8. 테스트 전략

- `CalculatorEngine`: Jest 단위 테스트. 사칙연산, 소수점, 연속계산, 0으로 나누기, 오버플로우 케이스 커버.
- `HistoryStore`: 단위 테스트로 JSON read/write, 파일 손상 시 fallback 동작 검증.
- E2E: Playwright로 골든패스(숫자 입력 → 연산 → 결과 확인 → history 반영 확인) 검증.

## 9. 미해결 질문 (Open Questions)

- 없음 — 논의된 범위 내 모든 항목 확정.

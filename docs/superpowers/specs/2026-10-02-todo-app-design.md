# 할 일 관리 앱 — PRD + 설계 명세

- 작성일: 2026-10-02
- 상태: 사용자 검토 대기
- 문서 구성: 1부 제품 요구사항(PRD), 2부 설계 명세. 이 문서 하나로 구현할 수 있도록 정리한다.
- 작성 방식: Superpowers `brainstorming` 스킬 (질문 → 설계 승인 → 문서화)

---

# 1부. 제품 요구사항 (PRD)

## 1. 개요

하루 10~20개의 할 일을 관리하는 개인용 웹 앱이다. HTML 파일 하나를 브라우저로 열면 바로 쓸 수 있고, 새로고침하거나 브라우저를 다시 켜도 데이터가 유지된다. 순수 JavaScript로 만들며 외부 라이브러리와 빌드 도구를 쓰지 않는다.

### 1.1 사용자

- 사용자 1명, 브라우저 1개
- 매일 그날 할 일 10~20개를 적고, 하나씩 끝내며 진행 상황을 확인한다.
- 서버, 로그인, 기기 간 동기화는 필요 없다.

### 1.2 사용자 요구사항

| # | 요구사항 |
|---|---|
| R1 | 할 일 추가, 수정, 삭제 |
| R2 | 완료 체크 |
| R3 | 카테고리 분류: 업무 / 개인 / 공부 |
| R4 | 진행률 보기 |
| R5 | 브라우저에서 바로 실행 |
| R6 | 새로고침해도 데이터 유지 |
| R7 | 순수 JavaScript, 기술적으로 단순하게 |

### 1.3 브레인스토밍에서 정한 사항

| 항목 | 결정 |
|---|---|
| 목록 방식 | 날짜별 목록. 할 일마다 날짜가 있고, 화면에는 선택한 날짜의 항목만 보인다 |
| 미완료 항목 | 자동 이월하지 않는다. 지난 날짜의 미완료 항목에 [오늘로] 버튼을 둔다 |
| 파일 구성 | `index.html` 단일 파일 (HTML·CSS·JS 모두 포함) |
| 진행률 기준 | 선택한 날짜에서 현재 필터에 걸린 항목 |
| 삭제 보호 | 브라우저 기본 확인창 |
| 테스트 | 자동 테스트 없음. 수동 확인 목록으로 검증 |

### 1.4 성공 기준

1. 할 일 하나를 5초 안에 추가할 수 있다 (입력 → Enter).
2. 새로고침, 탭 닫기, 브라우저 재시작 후에도 할 일·완료 상태·카테고리·날짜가 그대로다.
3. 화면 상단에서 진행률(완료 수/전체 수, 퍼센트)이 항상 보인다.
4. 인터넷 연결 없이 `index.html`을 더블클릭해 열어도 모든 기능이 동작한다.
5. 외부 라이브러리 0개, 파일 1개.

## 2. 범위

### 2.1 넣을 것
- 할 일 추가 / 수정 / 삭제
- 완료 체크
- 카테고리 3종 (업무 / 개인 / 공부)
- 카테고리 필터 (전체 / 업무 / 개인 / 공부)
- 진행률
- 날짜 이동 (이전 날 / 다음 날 / 오늘)
- 지난 날짜의 미완료 항목 [오늘로] 이동
- localStorage 저장

### 2.2 넣지 않을 것
마감 시각, 우선순위, 드래그 정렬, 검색, 반복 할 일, 하위 항목, 로그인, 서버 동기화, 알림, 미완료 자동 이월.

요청에 없던 기능이고, 하루 10~20개 규모에서는 없어도 불편하지 않다.

## 3. 화면

세로 한 화면으로 구성한다.

```
┌──────────────────────────────────────────┐
│  할 일                                    │
│  ◀  2026-10-02 (금)  ▶          [오늘]    │
│  ███████████░░░░░░░░   7/12 (58%)        │
├──────────────────────────────────────────┤
│  [할 일 입력...............] [업무▾] [추가] │
├──────────────────────────────────────────┤
│  [전체] [업무] [개인] [공부]                 │
├──────────────────────────────────────────┤
│  ☐ 보고서 초안 작성   [업무]  수정 삭제      │
│  ☐ 영어 단어 30개     [공부]  수정 삭제      │
│  ☑ 장보기             [개인]  수정 삭제      │
└──────────────────────────────────────────┘
```

1. **헤더:** 앱 제목, 날짜 이동(◀ 날짜(요일) ▶), [오늘] 버튼, 진행률 막대와 `완료 수/전체 수 (퍼센트)` 표시
2. **입력줄:** 텍스트 입력, 카테고리 선택(기본값 업무), 추가 버튼
3. **필터:** 전체 / 업무 / 개인 / 공부 탭 (기본값 전체)
4. **목록:** 체크박스, 할 일 텍스트, 카테고리 배지, 수정 버튼, 삭제 버튼
   - 완료 항목은 취소선과 흐린 색으로 표시한다.
   - 오늘보다 이전 날짜를 볼 때는 미완료 항목에 [오늘로] 버튼이 추가로 보인다.
5. **빈 상태:** 선택한 날짜·필터에 항목이 없으면 "할 일이 없습니다"를 보여 준다.

## 4. 기능 명세

### F1. 추가
- 텍스트를 입력하고 [추가]를 누르거나 Enter를 누르면 추가된다.
- 추가한 항목의 날짜는 지금 보고 있는 날짜, 카테고리는 입력줄에서 고른 값이다.
- 추가 후 입력창은 비우고 포커스를 유지한다. 카테고리 선택값은 유지한다.

### F2. 수정
- [수정]을 누르면 그 줄이 편집 모드로 바뀐다. 그 자리에 텍스트 입력과 카테고리 선택이 나타나고, [저장] / [취소] 버튼이 보인다.
- Enter는 저장, Esc는 취소다.
- 한 번에 한 줄만 편집 모드가 된다. 다른 줄의 [수정]을 누르면 앞의 편집은 저장하지 않고 닫힌다.

### F3. 삭제
- [삭제]를 누르면 `confirm("삭제할까요?")` 확인창이 뜬다. 확인을 눌렀을 때만 삭제한다.

### F4. 완료 체크
- 체크박스로 완료 / 미완료를 바꾼다. 진행률이 즉시 갱신된다.

### F5. 카테고리 필터
- 탭을 누르면 해당 카테고리 항목만 목록에 보인다. 선택된 탭은 강조 표시한다.
- 필터는 저장하지 않는다. 앱을 열면 항상 "전체"다.

### F6. 진행률
- 대상: 선택한 날짜이면서 현재 필터에 걸린 항목
- 표시: `완료 수/전체 수 (퍼센트)` + 막대. 퍼센트는 반올림한 정수다.
- 대상 항목이 0개면 `0/0 (0%)`, 막대는 비어 있다.
- 추가·수정·삭제·체크·필터 변경·날짜 이동·[오늘로] 직후 즉시 갱신된다.

### F7. 날짜 이동
- ◀ / ▶는 하루씩 이동한다. [오늘]은 오늘 날짜로 돌아온다.
- 앱을 열면 오늘 날짜가 보인다.
- "오늘"은 사용자 PC의 로컬 날짜이고, 버튼을 누른 시점을 기준으로 한다. 앱을 켜 둔 채 자정이 지나도 보고 있는 날짜는 자동으로 바뀌지 않는다.

### F8. 오늘로 이동
- 오늘보다 이전 날짜의 미완료 항목에만 [오늘로] 버튼이 보인다.
- 누르면 그 항목의 `date`만 오늘로 바뀐다. 텍스트·카테고리·완료 상태는 그대로다.
- 항목은 보고 있던 날짜의 목록에서 사라지고 오늘 목록에 나타난다.

### F9. 저장
- 모든 변경 직후 localStorage에 자동 저장한다. 저장 버튼은 없다.
- 앱을 열 때 localStorage에서 불러온다.

---

# 2부. 설계 명세

## 5. 기술 구조

### 5.1 원칙
- **파일 1개:** `index.html` 안에 `<style>`과 `<script>`를 모두 넣는다. `<script type="module">`은 쓰지 않는다. 파일을 더블클릭해 `file://`로 열면 브라우저가 모듈 로딩을 막기 때문이다.
- **원본 1개:** 메모리의 `state` 객체가 유일한 원본이다. 화면(DOM)에서 데이터를 읽어 오지 않는다.
- **전체 다시 그리기:** 상태가 바뀌면 ① 저장하고 ② 화면 전체를 다시 그린다. 항목이 수십 개라 성능 문제가 없고, 화면과 데이터가 어긋나지 않는다.
- **안전한 출력:** 사용자 입력은 항상 `textContent`나 `value`로 넣는다. `innerHTML`에 넣지 않는다.

### 5.2 파일 내부 구성

```
index.html
├─ <head>
│   ├─ <meta charset="utf-8">, <meta name="viewport" ...>
│   ├─ <title>할 일</title>
│   └─ <style>   … 8장 스타일
└─ <body>
    ├─ <main class="app">  … 5.3 마크업 (고정 뼈대)
    └─ <script>            … 7장 함수를 아래 순서로 작성
        1) 상수
        2) 날짜 함수
        3) 저장 함수
        4) 상태
        5) 데이터 변경 함수
        6) 조회 함수
        7) 렌더링 함수
        8) 이벤트 연결 + 시작
```

### 5.3 마크업 뼈대

HTML에는 변하지 않는 뼈대만 두고, 목록 항목은 스크립트가 만든다.

```html
<main class="app">
  <header>
    <h1>할 일</h1>
    <div class="date-nav">
      <button id="prev-day" aria-label="이전 날">◀</button>
      <span id="current-date"></span>
      <button id="next-day" aria-label="다음 날">▶</button>
      <button id="today-btn">오늘</button>
    </div>
    <div class="progress">
      <div class="progress-bar"><div id="progress-fill"></div></div>
      <span id="progress-text"></span>
    </div>
  </header>

  <form id="add-form">
    <input id="new-text" type="text" placeholder="할 일 입력" aria-label="할 일">
    <select id="new-category" aria-label="카테고리">
      <option value="work">업무</option>
      <option value="personal">개인</option>
      <option value="study">공부</option>
    </select>
    <button type="submit">추가</button>
  </form>

  <nav id="filters">
    <button data-filter="all">전체</button>
    <button data-filter="work">업무</button>
    <button data-filter="personal">개인</button>
    <button data-filter="study">공부</button>
  </nav>

  <ul id="task-list"></ul>
  <p id="empty-message">할 일이 없습니다</p>
</main>
```

추가는 `<form>`의 `submit` 이벤트로 처리한다. 버튼 클릭과 Enter가 같은 경로로 들어온다.

## 6. 데이터 설계

### 6.1 할 일 항목

```js
{
  id: "lq3k9a-x7f2",                          // Date.now().toString(36) + "-" + 난수 문자열
  text: "보고서 초안 작성",                     // 앞뒤 공백 제거, 빈 문자열 불가
  category: "work" | "personal" | "study",    // 업무 | 개인 | 공부
  done: false,
  date: "2026-10-02",                         // 사용자 PC 기준 로컬 날짜 (YYYY-MM-DD)
  createdAt: 1790000000000                    // 생성 시각(ms), 목록 순서에 사용
}
```

- `id`에 `crypto.randomUUID()`를 쓰지 않는다. `file://`에서는 동작하지 않는 브라우저가 있다.
- `date`는 `YYYY-MM-DD` 문자열이라 문자열 비교(`<`)로 날짜 앞뒤를 판단할 수 있다.

### 6.2 저장 형식

- localStorage 키: `todoApp.v1`
- 값: 할 일 항목 배열을 `JSON.stringify`한 문자열
- 용량: 하루 20개 × 365일 ≈ 7,300개 × 약 150B ≈ 1.1MB. 일반적인 한도(약 5MB) 안이다.

### 6.3 화면 상태

```js
const state = {
  tasks: [],               // 할 일 항목 배열 (저장 대상은 이것뿐)
  selectedDate: "",        // 보고 있는 날짜 "YYYY-MM-DD", 시작 시 오늘
  filter: "all",           // "all" | "work" | "personal" | "study"
  editingId: null          // 편집 중인 항목 id, 없으면 null
};
```

`selectedDate`, `filter`, `editingId`는 저장하지 않는다.

## 7. 함수 구성

### 7.1 상수
| 이름 | 값 |
|---|---|
| `STORAGE_KEY` | `"todoApp.v1"` |
| `CATEGORY_LABELS` | `{ work: "업무", personal: "개인", study: "공부" }` |
| `WEEKDAYS` | `["일", "월", "화", "수", "목", "금", "토"]` |

### 7.2 날짜 함수
| 함수 | 하는 일 |
|---|---|
| `toDateString(date)` | `Date` → 로컬 기준 `"YYYY-MM-DD"` (`toISOString`은 UTC라 쓰지 않는다) |
| `todayString()` | `toDateString(new Date())` |
| `addDays(dateStr, n)` | `"YYYY-MM-DD"`에 n일을 더한 문자열. `new Date(y, m - 1, d + n)`로 계산해 월말·연말을 처리한다 |
| `formatDate(dateStr)` | `"2026-10-02 (금)"` 형식의 표시용 문자열 |

### 7.3 저장 함수
| 함수 | 하는 일 |
|---|---|
| `loadTasks()` | localStorage에서 읽어 배열을 돌려준다. 값이 없거나, JSON 파싱에 실패하거나, 배열이 아니면 `[]` |
| `saveTasks(tasks)` | 배열을 JSON으로 저장한다 |

### 7.4 데이터 변경 함수
모두 `state.tasks`를 바꾼 뒤 `commit()`을 호출한다.

| 함수 | 하는 일 |
|---|---|
| `addTask(text, category)` | 텍스트를 다듬어 비어 있으면 아무것도 하지 않고 `false`. 아니면 `selectedDate`로 항목을 만들어 추가하고 `true` |
| `updateTask(id, text, category)` | 다듬은 텍스트가 비어 있으면 `false`(편집 모드 유지). 아니면 수정하고 `editingId = null`, `true` |
| `deleteTask(id)` | 항목 제거. 확인창은 호출하는 쪽(이벤트 처리)에서 띄운다 |
| `toggleTask(id)` | `done` 반전 |
| `moveToToday(id)` | `date = todayString()` |
| `commit()` | `saveTasks(state.tasks)` 후 `render()` |

### 7.5 조회 함수
| 함수 | 하는 일 |
|---|---|
| `getVisibleTasks()` | `date === selectedDate`이고 필터에 맞는 항목을 `createdAt` 오름차순으로 |
| `getProgress(tasks)` | `{ done, total, percent }`. `total`이 0이면 `percent`는 0, 아니면 `Math.round(done / total * 100)` |

### 7.6 렌더링 함수
| 함수 | 하는 일 |
|---|---|
| `render()` | 아래 세 함수를 차례로 호출 |
| `renderHeader()` | 날짜 표시, 진행률 막대 너비(`%`)와 텍스트 |
| `renderFilters()` | `state.filter`와 같은 탭에 `active` 클래스, `aria-pressed` 설정 |
| `renderList()` | `<ul>`을 비우고 `getVisibleTasks()`로 `<li>`를 새로 만든다. 비어 있으면 빈 상태 문구를 보인다 |
| `createTaskItem(task)` | 일반 모드 `<li>` 생성 |
| `createEditItem(task)` | 편집 모드 `<li>` 생성. 렌더링 후 텍스트 입력에 포커스 |

일반 모드 `<li>` 구조:

```html
<li class="task done" data-id="...">
  <input type="checkbox" data-action="toggle" checked aria-label="완료">
  <span class="task-text">장보기</span>
  <span class="badge badge-personal">개인</span>
  <button data-action="move-today">오늘로</button>   <!-- task.date < 오늘 && !task.done 일 때만 -->
  <button data-action="edit">수정</button>
  <button data-action="delete">삭제</button>
</li>
```

편집 모드 `<li>` 구조:

```html
<li class="task editing" data-id="...">
  <input type="text" class="edit-text" value="..." aria-label="할 일 수정">
  <select class="edit-category" aria-label="카테고리">…</select>
  <button data-action="save">저장</button>
  <button data-action="cancel">취소</button>
</li>
```

### 7.7 이벤트 연결

목록은 다시 그릴 때마다 새로 만들어지므로, 항목마다 이벤트를 붙이지 않고 `#task-list` 한 곳에서 받는다(이벤트 위임). 클릭된 요소의 `data-action`과 가장 가까운 `<li>`의 `data-id`로 동작을 고른다.

| 대상 | 이벤트 | 동작 |
|---|---|---|
| `#add-form` | `submit` | 기본 동작 막기 → `addTask()` → 성공 시 입력창 비우고 포커스 |
| `#prev-day` / `#next-day` | `click` | `selectedDate = addDays(selectedDate, ∓1)`, `editingId = null`, `render()` |
| `#today-btn` | `click` | `selectedDate = todayString()`, `editingId = null`, `render()` |
| `#filters` | `click` | 누른 버튼의 `data-filter`로 `state.filter` 변경, `editingId = null`, `render()` |
| `#task-list` | `change` | `data-action="toggle"` → `toggleTask(id)` |
| `#task-list` | `click` | `edit` → `editingId = id`, `render()` / `cancel` → `editingId = null`, `render()` / `save` → `updateTask()` / `delete` → `confirm` 후 `deleteTask()` / `move-today` → `moveToToday()` |
| `#task-list` | `keydown` | 편집 입력에서 Enter → 저장, Esc → 취소 |

### 7.8 시작 순서

1. `state.tasks = loadTasks()`
2. `state.selectedDate = todayString()`
3. 이벤트 연결
4. `render()`

### 7.9 데이터 흐름

```
사용자 동작
  → 이벤트 처리 (7.7)
  → 데이터 변경 함수 (7.4)가 state.tasks 수정
  → commit(): saveTasks() → render()
  → 화면 갱신 (진행률·목록·필터 모두)

화면만 바뀌는 동작 (날짜 이동, 필터, 편집 시작/취소)
  → state의 화면 상태만 수정 → render()   (저장하지 않음)
```

## 8. 스타일

- 최대 폭 640px, 가운데 정렬. 360px 폭에서도 가로 스크롤이 생기지 않는다.
- 긴 텍스트는 `overflow-wrap: anywhere`로 줄바꿈한다.
- 카테고리 배지 색: 업무 파랑, 개인 초록, 공부 주황
- 완료 항목: 텍스트에 취소선, 흐린 색
- 선택된 필터 탭: 배경색으로 강조
- 진행률 막대: 회색 바탕 위에 채움 막대, `width`를 퍼센트로 지정
- 시스템 글꼴 사용 (외부 글꼴을 불러오지 않는다)

## 9. 예외 처리

| 상황 | 처리 |
|---|---|
| 빈 텍스트, 공백만 있는 텍스트 | 추가·저장하지 않는다 (F1, F2 공통) |
| 삭제 | 확인창으로 한 번 더 묻는다. 되돌리기 기능이 없으므로 실수를 막는 최소 장치다 |
| 저장된 데이터가 깨져 있음 (JSON 파싱 실패 또는 배열이 아님) | 빈 목록으로 시작한다 |
| 배열 안 개별 항목의 형식 | 검증하지 않는다. 이 앱만 쓰는 데이터이기 때문이다 |
| 저장 용량 초과 | 처리하지 않는다. 6.2의 추정대로 이 규모에서는 생기지 않는다 |

## 10. 지원 환경

- 최신 Chrome, Edge, Firefox, Safari
- `file://`로 직접 열기, 로컬 서버로 열기 모두 동작
- 인터넷 연결 불필요

## 11. 검증 (수동 확인 목록)

| # | 확인할 것 | 방법 | 기대 결과 |
|---|---|---|---|
| 1 | 저장 유지 | 할 일 추가 → 새로고침 | 추가한 항목, 완료 상태, 카테고리가 그대로 있다 |
| 2 | 진행률 | 4개 추가 후 1개씩 체크 | `1/4 (25%)` → `2/4 (50%)` → … → `4/4 (100%)` |
| 3 | 필터 | 업무 2개·개인 1개 추가, 업무 1개 체크 후 탭 전환 | 전체 `1/3 (33%)`, 업무 `1/2 (50%)`, 개인 `0/1 (0%)`, 공부 `0/0 (0%)` |
| 4 | 수정·삭제 | 텍스트와 카테고리 수정, 다른 항목 삭제 → 새로고침 | 수정 내용이 남고 삭제한 항목은 없다. 삭제 확인창에서 취소하면 지워지지 않는다 |
| 5 | 빈 입력 | 빈칸·공백만 입력 후 추가, 수정에서 텍스트를 지우고 저장 | 추가되지 않고, 수정도 저장되지 않는다 |
| 6 | 오늘로 이동 | ◀로 어제 이동 → 항목 추가 → [오늘로] → [오늘] | 어제 목록에서 사라지고 오늘 목록에 나타난다. 완료 항목에는 [오늘로] 버튼이 없다 |

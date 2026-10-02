# 할 일 관리 앱 구현 프롬프트

[PRD.md](PRD.md)를 Claude Code에 단계별로 붙여 넣어 구현하기 위한 프롬프트 모음이다. Superpowers `writing-plans` 스킬로 구현 계획을 세운 뒤, 바로 붙여 넣을 수 있는 5단계 형식으로 정리했다.

## 쓰는 법

1. 아래 **공통 규약**을 먼저 붙여 넣는다. 대화가 길어져 이름이 어긋나기 시작하면 이 절만 다시 붙여 넣는다.
2. 1단계부터 순서대로 프롬프트를 붙여 넣는다.
3. 단계가 끝나면 **브라우저 확인** 항목을 직접 해 본다. 모두 맞으면 **커밋** 명령을 실행하고 다음 단계로 넘어간다.
4. 맞지 않는 항목이 있으면 다음 단계로 가지 말고, 어떤 항목이 어떻게 다른지 그대로 Claude에게 알려 고치게 한다.

## 단계 한눈에 보기

| 단계 | 내용 | PRD 항목 | 확인하는 완료 기준 |
|---|---|---|---|
| 1 | 뼈대와 저장 계층 | F-9 | 1 (저장 유지) |
| 2 | 할 일 추가 | F-1 | 1, 6 |
| 3 | 완료 체크와 삭제 | F-2, F-4 | 5 |
| 4 | 필터, 진행률, 날짜 이동 | F-5, F-6, F-7 | 2, 3, 8 |
| 5 | 수정, 오늘로 옮기기, 마무리 | F-3, F-8 | 4, 7, 그리고 1~8 전체 |

### 이렇게 나눈 이유

- **저장을 맨 앞에 뒀다.** 화면부터 만들고 저장을 나중에 붙이면 "새로고침하면 사라진다"는 문제를 마지막에 만난다. 1단계에서 콘솔로 데이터를 넣고 새로고침해 보이는지 확인해 두면, 이후 단계에서는 저장을 의심할 일이 없다.
- **필터, 진행률, 날짜 이동을 한 단계로 묶었다.** 셋 다 "지금 화면에 무엇을 보여 줄지"를 정하는 같은 함수(`getVisibleTasks`)를 쓴다. 진행률은 날짜와 필터 기준으로 계산해야 해서, 따로 만들면 반드시 다시 고치게 된다. 4단계 확인 항목에 탭별 기대값을 표로 박아 두었다. 이 앱에서 가장 틀리기 쉬운 곳이다.
- **이벤트 위임은 3단계에서 잡는다.** 체크와 삭제를 붙일 때 `data-action` 구조를 정해 두면, 5단계에서 수정·저장·취소·오늘로 버튼을 더할 때 `data-action` 값만 늘리면 된다.
- **수정을 맨 뒤에 뒀다.** 편집 모드는 목록 그리기, 이벤트 처리, 포커스를 모두 건드린다. 나머지가 굳은 뒤에 붙여야 덜 흔들린다.

---

## 공통 규약

모든 단계 프롬프트보다 먼저 붙여 넣는다.

```text
이 프로젝트에서 계속 지킬 규약이야. 모든 단계에서 이 이름과 규칙을 그대로 써 줘.
이름을 바꾸거나 새 이름으로 같은 일을 하는 함수를 만들지 마.

## 파일과 기술
- index.html 파일 하나. CSS는 <style>, 자바스크립트는 <script>로 같은 파일 안에 둔다.
- 외부 라이브러리, 프레임워크, 빌드 도구, ES 모듈(import/export, type="module")은 쓰지 않는다.
- 사용자 입력은 textContent나 value로만 화면에 넣는다. innerHTML은 쓰지 않는다.
- PRD.md의 "이번 버전에서 하지 않을 것"은 만들지 않는다.

## 데이터
- localStorage 키: "todoApp.v1", 값: { version: 1, tasks: [...] }
- 항목: { id, text, category: "work"|"personal"|"study", done, date: "YYYY-MM-DD", createdAt }
- 날짜는 로컬 연·월·일로 만든다. toISOString()은 쓰지 않는다.

## 상수와 상태
- STORAGE_KEY, CATEGORY_LABELS = { work: "업무", personal: "개인", study: "공부" }, WEEKDAYS
- const state = { tasks: [], selectedDate: "", filter: "all", editingId: null }

## 함수 이름 (만드는 단계)
- 1단계: todayString(), addDays(dateStr, n), formatDate(dateStr), loadTasks(), saveTasks(),
         commit(), render(), renderHeader(), renderList(), createTaskItem(task), init()
- 2단계: addTask(text, category), bindEvents()
- 3단계: toggleTask(id), deleteTask(id)
- 4단계: getVisibleTasks(), getProgress(tasks), renderFilters(), changeDate(dateStr), changeFilter(filter)
- 5단계: updateTask(id, text, category), moveToToday(id), createEditItem(task)

## 동작 규칙
- 데이터를 바꾸는 함수는 state.tasks를 바꾼 뒤 commit()을 부른다. commit()은 saveTasks() 후 render().
- 날짜 이동, 필터 변경, 편집 시작·취소는 저장하지 않고 render()만 부른다.
- 목록 클릭·체크·키 입력은 #task-list 한 곳에서 이벤트 위임으로 받는다.
  버튼과 체크박스는 data-action 값으로 구분하고, 줄(<li>)에는 data-id를 붙인다.

## 화면 요소 id
#prev-day, #current-date, #next-day, #today-btn, #progress-fill, #progress-text,
#add-form, #new-text, #new-category, #filters(버튼마다 data-filter), #task-list, #empty-message
```

---

## 1단계. 뼈대와 저장 계층 (F-9)

### 붙여 넣을 프롬프트

```text
PRD.md를 끝까지 읽고 1단계를 진행해 줘. 공통 규약을 지켜.

## 목표
화면 뼈대와 저장 계층을 만든다. 데이터를 저장소에 넣고 새로고침하면 화면에 보이는 데까지.

## 할 일
1. index.html을 만든다. PRD 6장 순서대로 고정 마크업을 둔다:
   헤더(제목, ◀ 날짜 ▶, [오늘], 진행률 막대와 숫자), 입력줄(<form>, 입력창, 카테고리 선택 기본값 업무, [추가]),
   필터 탭 4개, 빈 <ul id="task-list">, <p id="empty-message">.
   아이콘 버튼과 입력에는 aria-label을 붙인다.
2. 날짜 함수: todayString(), addDays(dateStr, n), formatDate(dateStr)
   - addDays는 new Date(y, m - 1, d + n)으로 계산해 월말·연말을 넘긴다.
   - formatDate("2026-10-02")는 "2026-10-02 (금)".
3. 저장 함수: loadTasks()는 값이 없거나, JSON이 깨졌거나, tasks가 배열이 아니면 빈 배열을 돌려준다.
   saveTasks()는 { version: 1, tasks: state.tasks }를 저장한다.
4. commit(), render(), renderHeader()(날짜 표시만), renderList(), createTaskItem(task), init()
   - 이 단계의 renderList는 selectedDate와 같은 날짜의 항목을 createdAt 순으로 그린다.
   - createTaskItem은 지금은 텍스트(.task-text)와 카테고리 배지(.badge)만 넣는다.
5. 스타일은 최대 폭 640px, 가운데 정렬, 시스템 글꼴 정도만. 꾸미기는 5단계에서 한다.
6. 할 일 추가, 체크, 삭제, 필터, 진행률, 날짜 이동은 아직 만들지 않는다.

끝나면 만든 것과 아래 확인 결과를 보고해 줘.
```

### 브라우저 확인

| # | 해 볼 것 | 맞는 결과 |
|---|---|---|
| 1 | `index.html` 더블클릭 | 뼈대가 보이고 헤더에 오늘 날짜와 요일. 콘솔 오류 없음 |
| 2 | 콘솔에 `addDays("2026-12-31", 1)` / `addDays("2026-03-01", -1)` | `"2027-01-01"` / `"2026-02-28"` |
| 3 | 콘솔에서 저장 후 새로고침 (아래 코드) | "테스트 항목"과 "업무" 배지가 목록에 보임 |
| 4 | 콘솔에 `localStorage.setItem("todoApp.v1", "abc")` 후 새로고침 | 오류 없이 빈 목록 |

3번에 쓸 코드:

```js
localStorage.setItem("todoApp.v1", JSON.stringify({ version: 1, tasks: [{ id: "t1", text: "테스트 항목", category: "work", done: false, date: todayString(), createdAt: Date.now() }] }))
```

### 커밋

```bash
git add index.html
git commit -m "feat: 1단계 뼈대와 저장 계층"
```

---

## 2단계. 할 일 추가 (F-1)

### 붙여 넣을 프롬프트

```text
PRD.md와 현재 index.html을 읽고 2단계를 진행해 줘. 공통 규약을 지켜.

## 목표
입력창에 적고 Enter나 [추가]로 할 일을 등록한다. (PRD F-1)

## 할 일
1. addTask(text, category)
   - 앞뒤 공백을 잘라 내고, 비어 있으면 아무것도 하지 않고 false.
   - id는 Date.now().toString(36) + "-" + Math.random().toString(36).slice(2, 8). crypto.randomUUID()는 쓰지 않는다.
   - date는 state.selectedDate, done은 false, createdAt은 Date.now(). state.tasks에 넣고 commit(), true.
2. bindEvents()를 만들고 init()에서 부른다. #add-form의 submit 이벤트 하나로 버튼과 Enter를 함께 처리한다.
   - 기본 동작을 막는다. 성공하면 입력창을 비운다. 성공이든 실패든 입력창에 포커스를 둔다.
   - 카테고리 선택값은 그대로 둔다.
3. createTaskItem에 체크박스와 [수정]·[삭제] 버튼 자리를 만든다. 동작은 3·5단계에서 붙인다.
   - 체크박스: data-action="toggle", 버튼: data-action="edit" / "delete", <li>: data-id

끝나면 바꾼 것과 아래 확인 결과를 보고해 줘.
```

### 브라우저 확인

| # | 해 볼 것 | 맞는 결과 |
|---|---|---|
| 1 | "보고서" 입력 후 Enter | 목록에 추가, 입력창 비고 커서 유지 |
| 2 | 카테고리를 개인으로 바꾸고 두 개 추가 | 둘 다 "개인" 배지, 선택은 개인 그대로 |
| 3 | 3개 추가 후 새로고침 (완료 기준 1) | 3개가 그대로 있음 |
| 4 | 빈칸, 공백만 입력하고 추가 (완료 기준 6) | 아무 일도 일어나지 않음 |
| 5 | `<b>굵게</b>` 입력 | 굵은 글씨가 아니라 글자 그대로 보임 |

### 커밋

```bash
git add index.html
git commit -m "feat: 2단계 할 일 추가"
```

---

## 3단계. 완료 체크와 삭제 (F-2, F-4)

### 붙여 넣을 프롬프트

```text
PRD.md와 현재 index.html을 읽고 3단계를 진행해 줘. 공통 규약을 지켜.

## 목표
체크박스로 완료를 표시하고, [삭제]로 지운다. 목록 이벤트 구조를 여기서 정한다. (PRD F-2, F-4)

## 할 일
1. toggleTask(id): done을 뒤집고 commit().
2. deleteTask(id): 항목을 빼고 commit(). 확인창은 이벤트 쪽에서 띄운다.
3. #task-list에 이벤트 위임을 단다. 항목마다 이벤트를 달지 않는다.
   - change: data-action="toggle" → toggleTask
   - click: data-action으로 분기하는 switch를 만든다. 지금은 "delete"만:
     confirm("삭제할까요?")에서 확인한 경우에만 deleteTask. 되돌리기는 만들지 않는다.
     5단계에서 "edit", "save", "cancel", "move-today"를 이 switch에 더할 것이다.
4. 완료 항목: <li>에 done 클래스, 텍스트에 취소선과 흐린 색.

끝나면 바꾼 것과 아래 확인 결과를 보고해 줘.
```

### 브라우저 확인

| # | 해 볼 것 | 맞는 결과 |
|---|---|---|
| 1 | 체크 → 새로고침 | 취소선과 흐린 글자가 남아 있음 |
| 2 | 체크 해제 | 원래 모습으로 돌아옴 |
| 3 | [삭제] → 확인창에서 취소 (완료 기준 5) | 항목이 남음 |
| 4 | [삭제] → 확인 → 새로고침 (완료 기준 5) | 항목이 사라지고 다시 나타나지 않음 |

### 커밋

```bash
git add index.html
git commit -m "feat: 3단계 완료 체크와 삭제"
```

---

## 4단계. 필터, 진행률, 날짜 이동 (F-5, F-6, F-7)

### 붙여 넣을 프롬프트

```text
PRD.md와 현재 index.html을 읽고 4단계를 진행해 줘. 공통 규약을 지켜.

## 목표
보고 있는 날짜와 필터에 맞는 항목만 보여 주고, 진행률을 그 기준으로 계산한다. (PRD F-5, F-6, F-7)

## 할 일
1. getVisibleTasks(): date === state.selectedDate 이고 필터에 맞는 항목을 createdAt 오름차순으로.
   renderList와 renderHeader가 모두 이 함수 결과를 쓴다. 다른 곳에서 따로 거르지 않는다.
2. getProgress(tasks): { done, total, percent }. total이 0이면 percent 0, 아니면 Math.round(done / total * 100).
3. renderHeader: 진행률 막대 width를 퍼센트로, #progress-text는 "완료 수 / 전체 수 (퍼센트%)" 예: "1 / 4 (25%)".
4. renderFilters(): 선택된 탭에 active 클래스와 aria-pressed="true", 나머지는 "false". render()에서 부른다.
5. changeFilter(filter), changeDate(dateStr): state를 바꾸고 editingId = null, render(). 저장하지 않는다.
   - ◀ / ▶: addDays(selectedDate, ∓1), [오늘]: todayString()
   - 필터는 저장하지 않으므로 새로고침하면 항상 "전체", 날짜는 오늘.
6. 빈 상태: 보여 줄 항목이 0개면 #empty-message를 보이고, 아니면 hidden.
   - 전체 탭: "이 날의 할 일이 없습니다. 위에서 추가해 보세요."
   - 카테고리 탭: "이 카테고리에는 할 일이 없습니다."

끝나면 바꾼 것과 아래 확인 결과를 보고해 줘.
```

### 브라우저 확인

**진행률 (완료 기준 2):** 4개 추가 후 하나씩 체크

| 체크 수 | 맞는 결과 |
|---|---|
| 0 | `0 / 4 (0%)`, 막대 비어 있음 |
| 1 | `1 / 4 (25%)`, 막대 4분의 1 |
| 1 → 해제 | `0 / 4 (0%)`로 돌아옴 |

**필터와 진행률 (완료 기준 3):** 업무 3개(1개 완료), 개인 2개(2개 완료)를 넣고 탭을 바꿔 가며 확인

| 탭 | 진행률 | 목록 줄 수 |
|---|---|---|
| 전체 | `3 / 5 (60%)` | 5 |
| 업무 | `1 / 3 (33%)` | 3 |
| 개인 | `2 / 2 (100%)` | 2 |
| 공부 | `0 / 0 (0%)` | 0, 카테고리용 안내 문구 |

**날짜와 빈 상태 (완료 기준 8)**

| # | 해 볼 것 | 맞는 결과 |
|---|---|---|
| 1 | ◀ 누르기 | 어제 날짜(요일 포함), 오늘 항목은 안 보임, 전체용 안내 문구 |
| 2 | ◀ 상태에서 항목 추가 → [오늘] → ◀ | 추가한 항목은 어제 목록에만 있음 |
| 3 | 업무 탭 선택 후 새로고침 | 필터는 "전체", 날짜는 오늘 |

### 커밋

```bash
git add index.html
git commit -m "feat: 4단계 필터, 진행률, 날짜 이동"
```

---

## 5단계. 수정, 오늘로 옮기기, 마무리 (F-3, F-8)

### 붙여 넣을 프롬프트

```text
PRD.md와 현재 index.html을 읽고 마지막 5단계를 진행해 줘. 공통 규약을 지켜.

## 목표
편집 모드와 [오늘로]를 붙이고, 화면을 PRD 6장대로 다듬은 뒤 완료 기준 1~8을 모두 확인한다. (PRD F-3, F-8)

## 할 일
1. 편집 모드 (F-3)
   - [수정](data-action="edit") → state.editingId = id, render(). 한 번에 한 줄만 편집 상태.
   - renderList는 editingId와 같은 줄을 createEditItem(task)로 그린다:
     텍스트 입력(.edit-text, 현재 값) + 카테고리 선택(.edit-category, 현재 값) + [저장](save) + [취소](cancel).
     그린 직후 .edit-text에 포커스.
   - updateTask(id, text, category): 앞뒤 공백을 잘라 비어 있으면 저장하지 않고 편집 상태 유지(false).
     아니면 바꾸고 editingId = null, commit().
   - 편집 입력에서 Enter = 저장, Esc = 취소. keydown도 #task-list 위임으로 받는다.
     한글 조합 중(e.isComposing)인 Enter는 무시한다.
   - [취소]나 Esc → editingId = null, render().
2. 오늘로 옮기기 (F-8)
   - task.date < todayString() 이고 !task.done 인 줄에만 [오늘로](data-action="move-today").
   - moveToToday(id): date만 todayString()으로 바꾸고 commit(). 텍스트, 카테고리, 완료 상태, createdAt은 그대로.
3. 포커스 유지: 목록을 다시 그리기 전에 포커스가 있던 줄(data-id)과 버튼(data-action)을 기억했다가 되돌린다.
   저장·취소 뒤에는 그 줄의 [수정] 버튼으로 돌아간다.
4. 스타일 (PRD 6장)
   - 배지: 업무 파랑, 개인 초록, 공부 주황 / 선택된 필터 탭은 배경색으로 강조
   - 진행률 막대: 회색 바탕 위 채움 막대 / 긴 텍스트는 overflow-wrap: anywhere
   - 줄의 버튼들은 한 묶음(.actions)으로 감싸 함께 줄바꿈되게 한다
   - 폭 360px에서도 가로 스크롤 없음 / :focus-visible로 키보드 포커스 표시
5. 점검: innerHTML, console.log, 쓰이지 않는 코드가 없는지. "하지 않을 것" 기능이 없는지.
6. localStorage를 비우고 PRD 10장 완료 기준 1~8을 처음부터 모두 확인한다. 실패하면 고치고 1번부터 다시.
7. README.md의 현재 상태를 "구현 완료"로 바꾸고, 실행 방법을 실제 안내로 고친다.

끝나면 완료 기준 1~8 각각의 결과(통과/실패와 원인), 이번에 고친 것,
직접 확인하지 못한 항목과 사용자가 확인하는 방법을 보고해 줘.
```

### 브라우저 확인

| # | 해 볼 것 | 맞는 결과 |
|---|---|---|
| 1 | 텍스트와 카테고리를 함께 바꿔 [저장] → 새로고침 (완료 기준 4) | 목록과 배지에 반영되고 유지됨 |
| 2 | 편집 중 Esc | 원래대로 |
| 3 | 편집 중 텍스트를 다 지우고 Enter | 저장되지 않고 편집 상태 유지 |
| 4 | 한글로 고치고 바로 Enter | 마지막 글자까지 저장되고, 두 번 저장되지 않음 |
| 5 | A 편집 중 B의 [수정] | A는 원래 값, B만 편집 상태 |
| 6 | ◀로 어제 → 2개 추가, 1개 체크 → [오늘로] → [오늘] (완료 기준 7) | 미완료 항목에만 [오늘로], 누르면 어제 목록에서 사라지고 오늘 목록에 있음 |
| 7 | 키보드로 체크박스에서 Space | 체크 후에도 포커스가 그 체크박스에 남음 |
| 8 | 창 폭을 360px로 줄이고 긴 텍스트 입력 | 가로 스크롤 없이 줄바꿈 |
| 9 | localStorage 비우고 완료 기준 1~8 처음부터 | 모두 통과 |

### 커밋

```bash
git add index.html README.md
git commit -m "feat: 5단계 수정, 오늘로 옮기기, 마무리"
```

---

## 놓치기 쉬운 곳

PRD에 직접 적혀 있지 않지만 쓰는 사람이 바로 부딪히는 것들이다. 해당 단계의 확인 항목에 넣어 두었다.

| 상황 | 기대하는 동작 | 확인하는 단계 |
|---|---|---|
| 한글 입력 중 Enter로 저장 | 마지막 글자까지 한 번만 저장 | 5단계 확인 4 |
| 체크·저장 후 키보드 포커스 | 같은 줄에 남음 | 5단계 확인 7 |
| 띄어쓰기 없는 긴 텍스트 | 좁은 화면에서도 가로 스크롤 없이 줄바꿈 | 5단계 확인 8 |
| 저장소 값이 깨짐 | 오류 없이 빈 목록 | 1단계 확인 4 |
| HTML처럼 생긴 입력 | 글자 그대로 표시 | 2단계 확인 5 |

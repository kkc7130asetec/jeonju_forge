---

name: organize-single-page-code
description: 단일 페이지 소스 파일을 분리하지 않고 공통 선언과 업무 기능 단위로 재배치하며, 지정된 구분선 주석을 추가한다. 각 선언은 역할과 의존성에 따라 type, api, hook, container, helper, lib로 분류하고, 코드 블록은 container, helper, api, hook, lib, type 순으로 정렬한다. 코드 동작, API, 변수명, 함수명은 변경하지 않는다.
---

# 단일 페이지 코드 구조 정리

## 목적

현재 단일 페이지 파일을 유지하면서 코드의 선언과 구현을 다음 기준에 따라 주석으로 구분한다.

이 작업은 코드의 구조와 가독성만 개선한다. 코드의 동작, 공개 인터페이스, 실행 순서 및 프레임워크 생명주기를 변경하지 않는다.

선언은 문법적 형태가 아니라 실제 책임과 의존성을 기준으로 분류한다.

예를 들어 함수라는 이유만으로 `lib`에 배치하지 않는다. 함수가 상태를 읽거나 변경하거나, API·라우터·모달·번역기 등의 외부 의존성을 사용한다면 해당 책임에 맞는 다른 레이어로 분류한다.

## 필수 입력

* 정리할 단일 페이지 소스 파일
* 프로젝트의 로컬 지침 및 `AGENTS.md`
* 프로젝트에서 사용하는 포맷 및 타입 검사 명령

## 절대 제약

* 파일을 새로 만들거나 기존 코드를 다른 파일로 분리하지 않는다.
* 동작을 변경하지 않는다.
* API 요청 방식, 엔드포인트, 파라미터, 반환값 처리를 변경하지 않는다.
* 변수명, 함수명, 타입명, 컴포넌트명, hook 이름을 변경하지 않는다.
* 함수 본문을 리팩터링하지 않는다.
* 조건문, 반복문, 실행 순서, hook 호출 순서를 변경하지 않는다.
* import 경로나 export 형태를 변경하지 않는다.
* 타입을 새로 설계하거나 기존 타입의 의미를 변경하지 않는다.
* 사용되지 않는 코드처럼 보여도 삭제하지 않는다.
* 포맷터가 요구하는 변경 이외의 표현식 변경을 하지 않는다.
* 단순히 함수라는 이유만으로 `lib`에 배치하지 않는다.
* 분류를 맞추기 위해 안전하지 않은 코드 이동을 수행하지 않는다.
* 하나의 선언을 억지로 여러 레이어로 분해하거나 함수 본문을 나누지 않는다.

## 레이어 정의

선언은 다음 기준에 따라 분류한다.

분류가 애매한 경우 선언의 이름이나 문법보다 다음 항목을 우선해서 확인한다.

1. 어떤 외부 의존성을 사용하는가
2. 상태를 읽거나 변경하는가
3. 부수 효과를 발생시키는가
4. 현재 컴포넌트에 종속되는가
5. 여러 기능에서 재사용되는가
6. 동일 입력에 대해 항상 동일 출력을 반환하는가

### `type`

컴파일 타임 타입 선언을 배치한다.

포함 대상:

* `type`
* `interface`
* 타입 전용 generic alias
* 컴포넌트 내부에서 정의한 props, emit, row, request, response 관련 타입
* 타입 추론을 보조하기 위한 타입 선언

예:

```ts
type RowData = Record<string, unknown>;

interface SearchParams {
  keyword: string;
  page: number;
}
```

다음은 `type`으로 분류하지 않는다.

* 런타임 상수
* validation schema
* 객체 리터럴
* enum과 유사하게 사용하는 런타임 객체

### `api`

API 요청을 직접 수행하거나 특정 API 호출을 캡슐화한 선언을 배치한다.

포함 대상:

* HTTP 요청을 직접 호출하는 함수
* API client를 통해 서버 요청을 수행하는 함수
* endpoint별 요청 함수
* 요청 결과를 반환하는 API 전용 함수

예:

```ts
const fetchItems = (params: SearchParams) =>
  $api.get("/items", { params });
```

다음은 `api`로 분류하지 않는다.

* API 호출 결과를 상태에 반영하는 전체 업무 흐름
* loading 상태 변경
* 성공·실패 메시지 처리
* 모달 닫기
* API 호출 전후 여러 작업을 조합하는 이벤트 핸들러

이러한 코드는 `container`에 배치한다.

예:

```ts
const handleSearch = async () => {
  loading.value = true;

  try {
    rows.value = await fetchItems(searchParams.value);
  } finally {
    loading.value = false;
  }
};
```

위 코드에서 `fetchItems`는 `api`, `handleSearch`는 `container`이다.

API 호출이 별도 함수로 분리되어 있지 않고 container 함수 내부에 직접 포함되어 있다면 함수 본문을 분리하지 않는다. 해당 함수 전체를 `container`에 유지한다.

### `hook`

React hook, Vue composable, Nuxt composable 또는 프레임워크 컨텍스트를 획득하는 호출을 배치한다.

포함 대상:

* React의 `useState`, `useMemo`, `useEffect`, `useCallback` 등
* Vue/Nuxt의 `ref`, `reactive`, `computed`, `watch`, `useRoute`, `useRouter`, `useI18n`, `useNuxtApp` 등
* 프로젝트 custom hook 또는 composable 호출
* hook/composable 호출을 통해 얻은 의존성
* hook 호출 결과를 구조 분해하여 받는 선언

예:

```ts
const { t, te } = useI18n();
const { $api, $appModal } = useNuxtApp();
const { compkey } = useCompkey();
```

주의:

* React hook은 호출 순서를 절대 변경하지 않는다.
* hook 호출을 레이어별로 모으기 위해 조건문 밖이나 안으로 이동하지 않는다.
* hook 결과를 사용하는 일반 함수까지 모두 `hook`으로 분류하지 않는다.
* hook에서 얻은 값을 이용하는 이벤트 함수나 업무 로직은 `container`, 단순 보조 함수는 `helper`에 배치한다.

### `container`

페이지나 기능의 상태, 상태 파생값, 이벤트 처리, 업무 흐름 및 부수 효과를 담당하는 선언을 배치한다.

포함 대상:

* `ref`, `reactive`, state 선언
* `computed`, memoized state
* 상태를 읽거나 변경하는 함수
* 사용자 이벤트 처리 함수
* API 호출과 상태 반영을 조합하는 함수
* 모달, 알림, 라우팅, 저장, 삭제, 검색 등의 실행 함수
* lifecycle 처리
* watcher 또는 effect
* 여러 하위 함수를 순서대로 조합하는 orchestration 함수
* 화면 또는 기능의 주요 업무 규칙

예:

```ts
const selectedWarekey = ref("");
const loading = ref(false);

const selectedRow = computed(() =>
  rows.value.find(row => row.warekey === selectedWarekey.value),
);

const handleSave = async () => {
  loading.value = true;

  try {
    await $api.post("/items", rows.value);
    $appModal.close();
  } finally {
    loading.value = false;
  }
};
```

다음과 같은 특성이 하나라도 있으면 우선 `container` 가능성을 검토한다.

* 상태를 변경한다.
* API를 호출한다.
* router, modal, notification, storage 등을 호출한다.
* 실행 순서 자체가 업무 의미를 가진다.
* 사용자의 이벤트에 직접 연결된다.
* 여러 로직을 조합해서 하나의 기능을 실행한다.

함수라는 이유만으로 `lib`에 배치하지 않는다.

### `helper`

현재 페이지 또는 현재 기능의 구현을 보조하는 작은 함수를 배치한다.

`helper`는 독립적인 범용 라이브러리라기보다 현재 컴포넌트의 가독성, 중복 제거 및 표현 단순화를 위한 함수다.

포함 대상:

* 번역 key prefix를 감싸는 함수
* 현재 컴포넌트 상태를 읽는 단순 판별 함수
* 현재 기능 전용 데이터 접근 함수
* 요청 파라미터나 옵션 객체를 만드는 함수
* 반복되는 표현을 줄이는 wrapper
* 현재 페이지에 종속된 formatting 또는 mapping 함수
* hook/composable에서 얻은 의존성을 사용하는 작은 보조 함수

예:

```ts
const wr = (key: string) => t(`wrcvit.${key}`);

const hasSelectedWarehouse = () =>
  selectedWarekey.value !== "";

const createSearchParams = () => ({
  compkey: compkey.value,
  warekey: selectedWarekey.value,
});
```

다음은 `helper`가 아니라 `container`에 가깝다.

```ts
const openWarehouseModal = () => {
  $appModal.open({
    warekey: selectedWarekey.value,
  });
};
```

위 함수는 모달 호출이라는 부수 효과를 발생시키므로 `container`로 분류한다.

다음은 `helper`가 아니라 `lib`에 가깝다.

```ts
const normalizeWarekey = (value: string) =>
  value.trim().toUpperCase();
```

위 함수는 외부 상태에 의존하지 않고 동일 입력에 대해 동일 출력을 반환하므로 `lib`로 분류할 수 있다.

### `lib`

외부 상태와 현재 컴포넌트에 의존하지 않는 독립적인 계산·변환·검증 로직을 배치한다.

`lib`는 모든 일반 함수를 의미하지 않는다.

다음 조건을 원칙적으로 모두 만족해야 한다.

* 입력값만으로 결과를 계산한다.
* 동일 입력에 대해 동일 출력을 반환한다.
* 컴포넌트의 state, ref, props, context를 직접 읽지 않는다.
* API, router, modal, notification, storage를 호출하지 않는다.
* 현재 locale이나 런타임 설정에 따라 결과가 달라지지 않는다.
* 외부 상태를 변경하지 않는다.
* 별도 파일로 옮겨도 함수 자체의 의미가 유지된다.
* 독립적인 단위 테스트가 가능하다.

포함 대상:

* 순수 계산
* 문자열 또는 숫자 변환
* 정렬과 필터링
* 입력값 검증
* 외부 상태에 의존하지 않는 formatting
* 데이터 구조 변환
* immutable mapping

예:

```ts
const normalizeWarekey = (value: string) =>
  value.trim().toUpperCase();

const calculateTotal = (rows: RowData[]) =>
  rows.reduce((total, row) => total + Number(row.amount ?? 0), 0);
```

다음은 `lib`로 분류하지 않는다.

```ts
const wr = (key: string) => t(`wrcvit.${key}`);
```

위 함수는 외부에서 획득한 번역 함수와 현재 locale에 의존하므로 `helper`이다.

```ts
const isSelectedRow = (row: RowData) =>
  row.warekey === selectedWarekey.value;
```

위 함수는 컴포넌트 상태를 읽으므로 `helper` 또는 기능 책임에 따라 `container`이다.

```ts
const saveRows = async () =>
  $api.post("/rows", rows.value);
```

위 함수는 API 호출이라는 부수 효과가 있으므로 `api` 또는 `container`이다.

## 레이어 분류 우선순위

하나의 선언이 여러 레이어의 성격을 가지는 경우 다음 순서로 판단한다.

1. API 요청만 직접 캡슐화하면 `api`
2. 상태 변경이나 부수 효과, 업무 흐름을 수행하면 `container`
3. hook/composable 자체 호출과 반환값 획득은 `hook`
4. 현재 페이지에 종속된 작은 보조 함수는 `helper`
5. 외부 의존성이 없는 순수 계산·변환 함수만 `lib`
6. 타입 선언은 `type`

함수 이름에 `get`, `set`, `handle`, `create`, `format`, `use` 등이 포함되어 있다는 이유만으로 분류하지 않는다. 함수 본문과 의존성을 직접 확인한다.

## 작업 절차

### 1. 파일 분석

파일 전체를 먼저 읽고 다음 항목을 분류한다.

* 여러 기능이 공동으로 사용하는 선언
* 업무 기능별로 종속된 선언과 구현
* 화면 전체 설정 및 페이지 조립 코드
* 선언 간 참조 관계
* 실행 순서에 영향을 받을 수 있는 코드
* React hook의 호출 순서에 영향을 받을 수 있는 코드
* 상태를 읽거나 변경하는 선언
* API, router, modal, notification 등 부수 효과를 발생시키는 선언
* 현재 컴포넌트에 종속된 helper
* 외부 의존성이 없는 순수 lib

코드를 이동하기 전에 참조 관계와 실행 순서를 확인한다.

각 함수는 다음 질문으로 분류한다.

1. 이 함수가 상태를 변경하는가?
2. API, modal, router, notification 등을 호출하는가?
3. hook 또는 composable에서 얻은 값에 의존하는가?
4. 현재 컴포넌트의 ref, props, state를 읽는가?
5. 동일 입력에 대해 항상 동일 출력을 반환하는가?
6. 별도 파일로 이동해도 독립적으로 사용할 수 있는가?

1번 또는 2번이 참이면 일반적으로 `container` 또는 `api`이다.

3번 또는 4번이 참이고 부수 효과가 없다면 일반적으로 `helper`이다.

5번과 6번을 모두 만족하고 외부 의존성이 없다면 `lib`이다.

### 2. 공통기능 블록 구성

파일 최상단의 import 및 프레임워크상 반드시 먼저 위치해야 하는 선언 다음에, 여러 업무 기능에서 공동으로 사용하는 선언을 `공통기능` 블록으로 모은다.

다음 항목만 공통기능으로 판단한다.

* 둘 이상의 기능이 함께 사용하는 type
* 둘 이상의 기능이 함께 사용하는 api
* 둘 이상의 기능이 함께 사용하는 hook 또는 composable 결과
* 둘 이상의 기능이 함께 사용하는 container 상태 또는 실행 로직
* 둘 이상의 기능이 함께 사용하는 helper
* 둘 이상의 기능이 함께 사용하는 순수 lib

하나의 기능에서만 사용하는 선언은 공통기능으로 올리지 않는다.

단순히 파일 상단에 있거나 여러 줄에서 참조된다는 이유만으로 공통기능으로 판단하지 않는다. 실제로 둘 이상의 업무 기능이 해당 선언을 공유하는지 확인한다.

구분선은 정확히 다음 형식을 사용한다.

```ts
/* -------------------------------------------------------------------------- */
/* 공통기능                                                                    */
/* -------------------------------------------------------------------------- */
```

공통기능 내부에서는 실제 코드가 존재하는 레이어에만 다음 주석을 사용한다.

```ts
/* container */
/* helper */
/* api */
/* hook */
/* lib */
/* type */
```

빈 레이어 주석을 만들지 않는다.

### 3. 업무 기능 블록 구성

공통기능 아래의 코드를 업무 기능 단위로 구분한다.

기능명은 구현 세부사항이 아니라 사용자의 업무 행위를 나타내도록 작성한다.

예:

* 품목 검색 기능
* 그리드 초기화 기능
* 행 추가 및 삭제 기능
* 저장 기능
* 화면 설정
* 페이지 렌더링

각 기능 블록은 다음 형식을 사용한다.

```ts
/* -------------------------------------------------------------------------- */
/* 품목 검색 기능                                                              */
/* -------------------------------------------------------------------------- */
```

하나의 함수가 특정 기능의 이벤트, 상태 변경, API 호출 흐름을 담당하면 해당 기능 블록의 `container`에 배치한다.

해당 기능에서만 사용하는 보조 함수는 공통기능으로 올리지 않고 같은 기능 블록의 `helper`에 배치한다.

### 4. 기능 내부 레이어 구분

각 기능 블록 내부에서는 필요한 레이어만 다음 순서를 기준으로 구분한다.

1. `container`
2. `helper`
3. `api`
4. `hook`
5. `lib`
6. `type`

예:

```ts
/* -------------------------------------------------------------------------- */
/* 저장 기능                                                                   */
/* -------------------------------------------------------------------------- */

/* container */

/* helper */

/* api */
```

단, 해당 레이어의 코드가 없다면 주석도 추가하지 않는다.

레이어보다 코드의 실행 순서, 초기화 순서, React hook 호출 순서가 우선한다. 레이어 순서를 맞추기 위해 동작상 위험한 이동을 하지 않는다.

안전하게 이동할 수 없는 경우에는 기존 상대적 순서를 유지하면서 가장 가까운 위치에 적절한 레이어 주석을 배치한다.

### 5. 코드 이동 원칙

다음 조건을 모두 만족할 때만 선언을 이동한다.

* 이동 전후 참조가 유효하다.
* 초기화 시점이 변하지 않는다.
* side effect 실행 순서가 변하지 않는다.
* React hook 호출 순서가 변하지 않는다.
* 함수 선언과 함수 표현식의 hoisting 차이로 인한 문제가 없다.
* 모듈 초기화 순서가 변하지 않는다.
* computed, watcher, effect의 등록 순서가 변경되어 동작에 영향을 주지 않는다.
* closure가 참조하는 값의 초기화 시점이 변하지 않는다.

안전하게 이동할 수 없는 코드는 원래 위치에 유지하고, 가장 적절한 기능 블록과 레이어 주석만 배치한다.

분류를 맞추기 위해 다음 작업을 수행하지 않는다.

* API 호출을 별도 함수로 추출
* 함수 본문 일부를 helper나 lib로 분리
* 여러 함수를 하나로 병합
* 상태 선언 방식 변경
* hook 호출 위치 변경
* 함수 선언문과 함수 표현식 상호 변환
* inline callback을 별도 함수로 추출
* 기존 함수를 순수 함수 형태로 리팩터링

### 6. 분류 예시

다음 코드가 있다고 가정한다.

```ts
type RowData = Record<string, unknown>;

const { t } = useI18n();
const { $api, $appModal } = useNuxtApp();

const rows = ref<RowData[]>([]);
const selectedWarekey = ref("");

const wr = (key: string) =>
  t(`wrcvit.${key}`);

const normalizeWarekey = (value: string) =>
  value.trim().toUpperCase();

const fetchRows = (params: Record<string, unknown>) =>
  $api.get("/rows", { params });

const createSearchParams = () => ({
  warekey: selectedWarekey.value,
});

const handleSearch = async () => {
  rows.value = await fetchRows(createSearchParams());
};

const openDetail = () => {
  $appModal.open({
    warekey: selectedWarekey.value,
  });
};
```

다음과 같이 분류한다.

```ts
/* container */

const rows = ref<RowData[]>([]);
const selectedWarekey = ref("");

const handleSearch = async () => {
  rows.value = await fetchRows(createSearchParams());
};

const openDetail = () => {
  $appModal.open({
    warekey: selectedWarekey.value,
  });
};

/* helper */

const wr = (key: string) =>
  t(`wrcvit.${key}`);

const createSearchParams = () => ({
  warekey: selectedWarekey.value,
});

/* api */

const fetchRows = (params: Record<string, unknown>) =>
  $api.get("/rows", { params });

/* hook */

const { t } = useI18n();
const { $api, $appModal } = useNuxtApp();

/* lib */

const normalizeWarekey = (value: string) =>
  value.trim().toUpperCase();

/* type */

type RowData = Record<string, unknown>;
```

단, 실제 코드에서는 선언 간 참조와 초기화 순서를 먼저 확인한다. 위 순서로 이동하는 것이 안전하지 않다면 기존 순서를 유지한다.

### 7. 자체 검토

수정 후 다음을 확인한다.

* 단일 파일이 유지되었는가
* 모든 기존 import와 export가 보존되었는가
* 변수명과 함수명이 변경되지 않았는가
* API 호출이 변경되지 않았는가
* hook 호출 순서가 유지되었는가
* 실행문의 상대적 순서가 유지되었는가
* 빈 레이어 주석이 없는가
* 공통기능에는 실제 공통 선언만 포함되었는가
* 기능 블록 이름이 업무 기능을 명확히 나타내는가
* 함수라는 이유만으로 `lib`에 배치된 코드가 없는가
* 상태를 변경하거나 부수 효과를 발생시키는 함수가 `lib`에 포함되지 않았는가
* 컴포넌트 상태나 hook 결과에 의존하는 함수가 `lib`에 포함되지 않았는가
* 현재 컴포넌트 전용 보조 함수가 `helper`로 분류되었는가
* 독립적인 순수 계산·변환 함수만 `lib`로 분류되었는가
* API 호출만 캡슐화한 함수와 전체 업무 흐름 함수가 구분되었는가
* 안전하지 않은 코드 이동이 발생하지 않았는가

## 출력

이 단계에서는 다음 결과를 남긴다.

1. 정리된 단일 페이지 파일
2. 생성한 기능 블록 목록
3. 각 기능 블록에 포함된 레이어 목록
4. 이동하지 못한 코드와 그 이유
5. 분류가 애매하여 기존 위치를 유지한 선언과 판단 근거
6. 2단계 검증에서 실행해야 할 포맷 및 타입 검사 후보

포맷 및 타입 검사의 최종 판정은 검증 단계에서 수행한다.

---
name: frontend-test-writer
description: write frontend test code for the `jeonju_front` project from an approved frontend test plan. Use when implementing Vitest, @testing-library/vue, and MSW tests after a test plan exists.
---
---

name: frontend-test-writer
description: write frontend test code for the `jeonju_front` project from an approved frontend test plan. Use when implementing Vitest, @testing-library/vue, and MSW tests after a test plan exists.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Frontend Test Code Writer

이 스킬은 `jeonju_front`의 승인된 테스트 계획을 바탕으로 실제 테스트 코드를 작성하거나 개정할 때 사용하는 전용 스킬이다.

목표는 테스트 계획의 TC를 `Vitest`, `@testing-library/vue`, `MSW` 기반 테스트 코드로 정확히 옮기고, `TEST_RULE.md`와 기존 테스트 패턴에 맞게 유지보수 가능한 테스트를 만드는 것이다.

## 사용할 문서

1. `docs/frontend/TEST_RULE.md`
2. 승인된 테스트 계획 문서 `docs/frontend/test_plans/*.md`
3. 관련 소스 코드
4. 필요 시 기존 테스트 코드

## 역할

* 테스트 계획의 TC를 실제 테스트 코드로 구현한다.
* 각 `it` 또는 `test`가 어떤 TC ID를 구현하는지 드러낸다.
* `required_assertions`를 실제 assertion으로 옮긴다.
* `forbidden_assertions`와 `TEST_RULE.md`의 금지 규칙을 위반하지 않는다.
* 테스트 파일은 `test/` 하위에 `*.test.ts`로 작성한다.
* 테스트 계층은 테스트 계획의 `계층` 값을 따른다.

## 기본 원칙

* [critical] 테스트를 위해 구현된 코드에 손을 대지 않는다.
* 테스트의 위치는 tests/ 하위에 계층별로 나누어 작성한다.
* 테스트 계획 없이 테스트 코드를 작성하지 않는다.
* 구현 대상 TC ID와 테스트 파일 범위를 먼저 고정한 뒤 작성한다.
* 테스트 계획에 없는 동작을 임의로 추가 검증하지 않는다.
* 하나의 테스트는 하나의 정책 또는 계약만 검증한다.
* 테스트 대상 계층보다 아래에 있는 의존성은 fake, stub, mock으로 대체한다.
* 테스트 대상 계층보다 위의 흐름까지 검증하지 않는다.
* mock, fixture, handler, helper는 테스트를 성립시키는 최소 범위로만 구성한다.
* 구현 세부사항이 아니라 업무 정책, 공개 계약, 반환값 중심으로 검증한다.

## 작성 절차

1. `TEST_RULE.md`를 확인한다.
2. 테스트 계획 문서를 읽고 구현할 TC ID를 고른다.
3. TC의 `계층`, `테스트 대상`, `검증 관점`을 확인한다.
4. 테스트 계층에 맞는 테스트 파일 위치를 정한다.
5. 관련 소스와 기존 테스트 패턴을 확인한다.
6. 필요한 fake, stub, mock, MSW handler, render helper만 최소 구성한다.
7. `required_assertions`를 코드 assertion으로 구현한다.
8. `forbidden_assertions`와 공통 금지 규칙을 위반하지 않는지 확인한다.
9. 테스트를 실행한다.
10. 실패하면 테스트 의도와 실제 구현을 비교해 수정한다.
11. 통과 후 과도한 mock, 구현 세부사항 검증, 중복 assertion을 정리한다.

## 계층별 기준

### lib

* 순수 함수의 입력과 출력을 검증한다.
* 외부 의존성을 사용하지 않는다.
* Vue runtime, DOM, Nuxt composable, API, grid instance를 끌어오지 않는다.

### hook

* composable의 반환 계약, 상태 계산, 파생값을 검증한다.
* 필요한 composable은 mock한다.
* 실제 API 호출과 실제 렌더링을 하지 않는다.
* DOM visible/disabled를 직접 검증하지 않는다.

### container

* 이벤트 핸들러, 정책 로직, 의존성 호출 계약을 검증한다.
* 실제 외부 의존성 대신 fake, stub, mock을 사용한다.
* 의존성의 내부 구현을 검증하지 않는다.
* 실제 PqGrid 렌더링이나 pqGrid 라이브러리 내부 이벤트 발생은 검증하지 않는다.

### page

* 하위 컴포넌트에 전달되는 props 계약을 검증한다.
* 하위 컴포넌트는 stub 처리한다.
* 내부 함수 구현을 검증하지 않는다.
* source code 문자열 비교를 사용하지 않는다.

### integration

* 단위 테스트와 page 계약 테스트만으로 확인하기 어려운 핵심 연동 흐름만 검증한다.
* 단위 테스트에서 이미 검증한 세부 케이스를 반복하지 않는다.
* 실제 렌더링 비용을 감수할 가치가 있는 경우에만 작성한다.
* 현재 프로젝트에 E2E 환경이 없으므로 E2E 테스트를 작성하지 않는다.

## 파일 구성 기준

소스 파일 구조와 테스트 파일 구조를 1:1로 맞추지 않는다. 테스트 대상의 책임 기준으로 파일을 나눈다.

권장 파일명:

```txt
{feature}.lib.test.ts
{feature}.hook.test.ts
{feature}.container.test.ts
{feature}.page.test.ts
{feature}.integration.test.ts
```

작은 페이지는 하나의 테스트 파일에서 `describe("lib")`, `describe("container")`, `describe("page")`처럼 구분해도 된다.

다음 중 하나라도 해당하면 테스트 파일을 분리한다.

* 테스트 대상 함수가 많다.
* mock 설정이 계층마다 다르다.
* container 테스트와 page 테스트가 섞이기 시작한다.
* 테스트 파일 길이가 과도하게 증가한다.

## 구현 규칙

* 테스트 이름 또는 주석에서 TC ID가 식별 가능해야 한다.
* 하나의 `it` 또는 `test`는 가능하면 하나의 TC를 구현한다.
* `required_assertions`는 누락 없이 assertion으로 옮긴다.
* `forbidden_assertions`가 있으면 해당 금지 항목을 코드에 넣지 않는다.
* 공용 setup은 재사용하되, 테스트마다 의도가 흐려질 정도로 추상화하지 않는다.
* 기존 프로젝트의 render helper, factory, MSW 유틸이 있으면 우선 재사용한다.
* `Behavior`와 `Contract`는 테스트 계획의 분리 원칙을 따른다.
* 같은 TC 안에서 여러 계층을 동시에 검증하지 않는다.
* API 계약 검증이 필요한 경우 MSW 또는 프로젝트 표준 API mock을 사용한다.

## 금지 사항

* 테스트 계획 없이 테스트 코드를 작성하지 않는다.
* 테스트 계획에 없는 동작을 임의로 추가 검증하지 않는다.
* 여러 계층을 한 테스트에 섞지 않는다.
* CSS class, DOM 구조 전체, snapshot 전체 비교를 사용하지 않는다.
* 내부 state/ref/reactive 구조를 직접 검증하지 않는다.
* source code 문자열 비교를 사용하지 않는다.
* mock 호출 순서나 호출 횟수를 과도하게 고정하지 않는다.
* 테스트 하네스 안에 실제 비즈니스 로직을 다시 구현하지 않는다.
* 실제 PqGrid 연동이 목적이 아닌 테스트에서 PqGrid를 렌더링하지 않는다.

## 출력 기준

* 생성/수정한 테스트 파일 경로를 명시한다.
* 구현한 TC ID 목록을 명시한다.
* 계층별로 어떤 테스트를 작성했는지 명시한다.
* 실행한 테스트 명령과 결과를 명시한다.
* 실행하지 못한 경우 이유를 명시한다.

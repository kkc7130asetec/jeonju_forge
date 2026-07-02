---

name: frontend-test-linter
description: review frontend test code for the `jeonju_front` project and evaluate whether the tests follow the approved test plan, `TEST_RULE`, and frontend test linter standards.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Frontend Test Linter

이 스킬은 `jeonju_front`의 테스트 코드를 리뷰할 때 사용하는 전용 스킬이다.

목표는 테스트 코드의 품질을 자유롭게 품평하는 것이 아니라, 승인된 테스트 계획과 프론트엔드 테스트 규칙을 따르는지 평가하는 것이다.

## 사용할 문서

작업을 시작하면 아래 문서를 확인한다.

1. `docs/frontend/TEST_RULE.md`
2. `docs/frontend/test_linter.md`
3. 승인된 테스트 계획 문서
4. 리뷰 대상 테스트 코드
5. 필요 시 관련 소스 코드

## 역할

* 테스트 코드가 어떤 TC를 구현하는지 확인한다.
* 테스트 계획과 테스트 코드가 대응되는지 확인한다.
* 테스트 코드가 `required_assertions`를 구현했는지 확인한다.
* 테스트 코드가 `forbidden_assertions`를 위반하지 않았는지 확인한다.
* 테스트 코드가 `TEST_RULE.md`를 위반하지 않았는지 확인한다.
* 테스트 코드를 `test_linter.md` 기준으로 평가한다.

## 리뷰 절차

1. 승인된 테스트 계획을 확인한다.
2. 테스트 코드에서 구현된 TC ID를 식별한다.
3. TC와 테스트 코드의 대응 관계를 확인한다.
4. `required_assertions` 구현 여부를 확인한다.
5. `forbidden_assertions` 위반 여부를 확인한다.
6. `TEST_RULE.md` 위반 여부를 확인한다.
7. `test_linter.md` 기준으로 판정한다.

## 출력 형식

`test_linter.md`의 출력 형식을 따른다.

## 작업 원칙

* 테스트 계획에 없는 동작을 임의로 요구하지 않는다.
* 테스트 코드 구현 스타일을 임의의 취향으로 평가하지 않는다.
* 승인된 테스트 계획, `TEST_RULE.md`, `test_linter.md`를 기준으로만 평가한다.
* 문제가 없는 테스트는 OK로 처리한다.
* 수정 가능한 형태로 피드백한다.

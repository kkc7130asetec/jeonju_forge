---
name: format-grid-object-properties
description: searchlist 또는 colModel 배열 안의 한 줄 객체 리터럴을 속성별 여러 줄 형식으로 정리한다. 그리드 컬럼 정의, 검색 목록 설정, TypeScript 또는 JavaScript 객체의 가독성만 개선하는 포맷 변경 요청에 사용한다.
---

# 그리드 객체 속성 여러 줄 정리

한 줄 객체 리터럴을 프로젝트의 기존 들여쓰기와 쉼표 스타일에 맞춰 속성별 여러 줄로 펼친다. 동작 변경 없이 가독성만 개선한다.

## 작업 절차

1. 대상이 `searchlist`, `colModel` 또는 사용자가 지정한 객체 배열인지 확인한다.
2. 한 줄로 작성된 객체만 속성별 줄바꿈으로 변환한다. 이미 여러 줄인 객체와 대상 밖 코드는 변경하지 않는다.
3. 객체의 속성 이름, 값, 배열 내 순서, 속성 순서, `summary` 같은 중첩 객체, 쉼표 유무를 그대로 보존한다.
4. 대상 파일의 주변 코드에서 들여쓰기와 후행 쉼표 규칙을 확인해 동일하게 적용한다.
5. 변경 후 diff를 검토해 줄바꿈·공백 외 의미 있는 수정이 없는지 확인한다. 프로젝트에서 가벼운 포맷 또는 타입 검사가 가능하면 실행한다.

## 변환 예시

다음을:

```ts
{ title: wtakit("fordqty"), dataType: "double", dataIndx: "fordqty", editable: false, summary: { type: "sum" } },
```

다음처럼 만든다:

```ts
{
  title: wtakit("fordqty"),
  dataType: "double",
  dataIndx: "fordqty",
  editable: false,
  summary: { type: "sum" },
},
```

## 제약

* 속성을 추가·삭제·이름 변경·재정렬하지 않는다.
* 문자열 따옴표, 함수 호출, 중첩 객체와 배열의 내부 표현을 변경하지 않는다.
* `editable`, `summary`, `dataIndx` 등 특정 속성의 의미를 해석하거나 수정하지 않는다.
* 포맷터가 대상 외 코드를 수정하려 하면 대상 객체 변경만 남기도록 되돌린다.

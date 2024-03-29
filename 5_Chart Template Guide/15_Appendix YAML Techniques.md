## Scalars and Collections
YAML 스펙에 따르면 2개의 collection 타입, 수 많은 scalar 타입이 존재한다.

collection 타입의 두 종류는 map과 sequence다:

``` yaml
map:
  one: 1
  two: 2
  three: 3

sequence:
  - one
  - two
  - three
```

scalar 값은 위 collection과는 다르게 각 개별 값을 나타낸다.

### Scalar Types in YAML
Helm의 YAML에서 scalar 데이터 타입은 k8s resource 정의를 포함한 수많은 규칙에 의해 결정된다. 그러나 타입을 유추할 떄, 다음 규칙들은 대부분 잘 적용된다.

integer, float 타입이 쿼팅되지 않으면 보통 숫자 타입으로 다뤄진다:

``` yaml
count: 1
size: 2.34
```

하지만 쿼팅되면 문자열로 다뤄진다:

``` yaml
count: "1" # <-- string, not int
size: '2.34' # <-- string, not float
```

boolean에 대해서도 동일하다:

``` yaml
isGood: true   # bool
answer: "true" # string
```

빈 값은 null로 표현한다(nil이 아님)

port: "80"은 유효한 YMAL이기 때문에 template engine, YAML parser에서 오류가 발생하지만 k8s에서는 port 필드에 대해 정수를 예상하기 떄문에 오류가 발생한다.

때때로 YAML node tag를 사용해 특정 타입임을 강제할 수 있다:

``` yaml
coffee: "yes, please"
age: !!str 21
port: !!int "80"
```

위에서 !!str은 age가 문자열임을 강제한다. port는 정수형으로 강제한다.

## Strings in YAML
YAML 문서에 사용하는 대부분의 데이터는 문자열이다. YAML에는 문자열을 나타내는 방법이 여러가지다.

아래는 문자열을 정의하는 "inline" 방식이다.

``` yaml
way1: bare words
way2: "double-quoted strings"
way3: 'single-quoted strings'
```

line 스타일은 한 줄에 표현돼야 한다.

- Bare words는 quote되지 않으며 이스케이프 처리되지 않는다. 이러한 이유로 어떤 문자를 사용하는지 유의해야 한다.
- Double-quoted strings은 \로 이스케이프된 특정 문자를 포함할 수 있다. \n을 사용해 줄바꿈을 이스케이프할 수 있다.
- Single-quoted strings은 "literal" 문자열이며 \를 사용해 이스케이프 하지 않는다. 유일한 escape sequnce는 ''이며 이는 '로 디코딩된다.

아래는 multi-line 방식이다:

``` yaml
coffee: |
  Latte
  Cappuccino
  Espresso
```

coffe의 값은 단일 문자열 `Latte\nCappuccino\nEspresso\n`와 같다.

| 이후 첫 번째 줄은 정확하게 들여쓰기 돼야 한다. 아래는 올바르지 않은 예시다:

``` yaml
coffee: |
                  Latte
  Cappuccino
  Espresso
```

template에서는 위 오류를 피하기 위해 아래와 같은 방식으로 사용한다:

``` yaml
coffee: |
  # Commented first line
         Latte
  Cappuccino
  Espresso
```

첫 번째 줄이 무엇이든 문자열의 내용으로 인식된다는 점은 유의해야 한다. 따라서 이러한 기술을 사용해 k8s cm에 내용을 작성하는 경우 모든 사용자가 예상할 수 있는 유형의 주석이 있어야 한다.

### Controlling Spaces in Multi-line Strings
위에서 |를 사용해 multi-line 문자열을 사용했다. 이 때 문자열의 마지막에 \n가 있었다. YAML 프로세서에서 이를 제거하기 위해 `|-`를 사용할 수 있다:

``` yaml
coffee: |-
  Latte
  Cappuccino
  Espresso
```

coffe의 값은 다음과 같다: `Latte\nCappuccino\nEspresso`

이와 반대로 문자열 마지막의 모든 공백을 보존하길 원할 수도 있다. 이를 위해 `|+`를 사용할 수 있다:

``` yaml
coffee: |+
  Latte
  Cappuccino
  Espresso  


another: value
```

coffee의 값은 다음과 같다: `Latte\nCappuccino\nEspresso\n\n\n`

문자열 내 들여쓰기는 유지되고 줄바꿈도 유지된다:

``` yaml
coffee: |-
  Latte
    12 oz
    16 oz
  Cappuccino
  Espresso
```

coffee의 값은 다음과 같다: `Latte\n 12 oz\n 16 oz\nCappuccino\nEspresso`

### Indenting and Templates
template를 작성할 때 다른 tempalte의 내용을 삽입하는 경우도 있다. 이를 위한 두 가지 방법이 있다:

- {{ .Files.Get "FILENAME" }}: chart 내 파일의 내용을 가져온다.
-  {{ include "TEMPLATE" . }}: template을 렌더링한 다음 해당 내용을 chart에 포함한다.

When inserting files into YAML, it's good to understand the multi-line rules above. Often times, the easiest way to insert a static file is to do something like this:

``` yaml
myfile: |
{{ .Files.Get "myfile.txt" | indent 2 }}
```

Note how we do the indentation above: indent 2 tells the template engine to indent every line in "myfile.txt" with two spaces. Note that we do not indent that template line. That's because if we did, the file content of the first line would be indented twice.

### Folded Multi-line Strings
때때로 YAML 에서는 여러 줄로 문자열을 표현하고 해석될 때 하나의 긴 문자열로 처리되기 원하는 경우가 있다. 이는 "folding"이라고 부르며 `|` 대신 `>`를 이용해 folded block을 선언한다:

``` yaml
coffee: >
  Latte
  Cappuccino
  Espresso
```

coffe의 값은 `Latte Cappuccino Espresso\n`다. 마지막 개행 문자를 제외한 모든 개행문자는 공백으로 변환된다. `>-`를 사용해 개행 문자를 변환하거나 삭제한다.

folded syntax에서 들여쓰기된 텍스트 라인은 보존된다.

``` yaml
coffee: >-
  Latte
    12 oz
    16 oz
  Cappuccino
  Espresso
```

coffee의 값은 `Latte\n 12 oz\n 16 oz\nCappuccino Espresso`이다.

## Embedding Multiple Documents in One File
단일 파일 내 여러 YAML 문서를 작성할 수 있다. 이를 위해 문서 접두사로 `---`, 접미사로 `...`를 사용하면 된다.

``` yaml
---
document: 1
...
---
document: 2
...
```

대부분의 경우 --- 또는 ...을 생략한다.

helm에서는 일부 파일의 경우 여러 문서를 포함할 수 없다. 예를 들어 values.yaml 파일 내에서는 첫 번째 문서만 사용한다.

하지만 template 파일의 경우 여러 문서를 포함할 수 있다.

필요한 경우에만 한 파일에 여러 문서를 작성하는 것을 권장한다. 왜냐하면 디버깅하기가 어렵기 때문이다.

## YAML is a Superset of JSON

## YAML Anchors
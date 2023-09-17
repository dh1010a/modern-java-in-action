# Chapter 5 - 스트림 활용(1)
전 챕터에서는 스트림을 이용해서 외부 반복을 내부 반복으로 바꾸는 방법을 살펴보았다. 이번 챕터에서는 스트림 API가 지원하는 다양한 연산을 이용해서 필터링, 슬라이싱, 매핑, 검색, 매칭, 리듀싱 등 다양한 데이터 처리 질의를 표현하는 법을 살펴본다.

## 5.1 필터링
이 절에서는 프레디케이트 필터링과 고유 요소만 필터링 하는 방법을 살펴본다.
### 5.1.1 프레디케이트로 필터링
스트림 인터페이스는 filter메서드를 지원한다. 이 메서드는 **프레디케이트**(불리언을 반환하는 함수)를 인수로 받아서 일치하는 모든 요소를 포함하는 스트림을 반환한다.
```java
List<Dish> vegeterianMenu = menu.stream()
                                .filter(Dish::isVegeterian)
                                .collect(toList());
```

### 5.1.2 고유 요소 필터링
스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원한다. 이때 고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정된다.
```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct()
        .forEach(System.out::println);
// 짝수선택 -> 중복 필터링
```

## 5.2 스트림 슬라이싱
스트림의 요소를 선택하거나 스킵하는 다양한 방법을 설명한다. 프레디케이트를 이용, 스트림의 처음 몇 개의 요소를 무시하는 방법, 특정 크기로 스트림을 줄이는 방법 등 다양한 방법을 이용해 효율적으로 이런 작업을 수행할 수 있다.

### 5.2.1 프레디케이트를 이용한 슬라이싱
자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두가지 새로운 메서드를 지원한다.

**TAKEWHILE 활용**

filter 메서드는 전체 스트림을 반복하며 각 요소에 프레디케이트를 적용하며 참이 되는 것만 다음 단계로 넘어간다. 하지만 리스트가 정렬되어 있는 경우에 뒤에것을 더 안보고 중단할 수는 없을까?

* 리스트가 정렬되어있는 경우, 크기가 큰 스트림에 유용하게 사용할 수 있다.

이것을 takeWhile을 활용하여 조건이 참이 아닐 경우 바로 거기서 멈추도록 할 수 있다.

**DRIOWHILE 활용**

DROPWHILE은 TAKEWHILE과 반대로 동작한다. 나머지 연산이라고도 볼 수 있으며, 참인 부분을 버리다가 처음으로 거짓인 부분이 나오면 그 뒤 남은 부분을 모두 반환해준다.

### 5.2.2 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n)메서드를 지원한다. 스트림이 정렬되어 있으면 최대 요소 n개를 반환할 수 있다.

### 5.2.3 요소 건너뛰기
스트림을 처음 n개 요소를 제외한 스트림을 반환하는 skip() 메서드를 지원한다. n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환된다. offset과 비슷하다고도 볼 수 있다.

* limit(n)과 skip(n)은 상호 보완적인 연산을 수행한다.

## 5.3 매핑
특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산이다. 스트림 API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.

### 5.3.1 스트림의 각 요소에 함수 적용하기
스트림은 함수를 인수로 받는 map 메서드를 지원한다. 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.(기존 값을 고치는 것 보다는 새로운 버전을 만든다는 개념에 가까움) 다른 map 메서드를 연결(chaning) 할 수도 있다.
```java
List<Integer> dishNameLengths = menu.stream()
                                    .map(Dish::getName)
                                    .map(String::length)
                                    .collect(toList());
```

### 5.3.2 스트림 평면화

["Hello", "world"] 리스트를 ["H","e","l","o","W","r","d"] 리스트로 만들어 보자. 이것을 해결하기 위해 distinct로 해결하려고 한다면 잘못된 풀이이다.
```java
words.stream()
        .map(word -> word.split(""))
        .distinct()
        .collect(toList());
```
위와 같이 풀게되면 Stream\<String\>이 아닌 `Stream<String[]>`이 된다. 이 것을 해결하기 위해 나온 것이 flatMap 메서드이다.

```java
words.stream()
      .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
      .flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화
      .distinct()
      .collect(toList());
```

flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다. 즉, map(Arrays::stream)과 달리 flatMap은 하나의 평면화된 스트림을 반환한다.

요약하면 flatMap 메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

## 5.4 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다.

### 5.4.1 프레디케이트가 적어도 한 요소와 일치하는지 확인
이때는 anyMatch 메서드를 이용한다.
```java
if (menu.stream().anyMatch(Dish::isVegetarian)) {
  System.out.println("The menu is (somewhat) vegetarian friendly!");
}
```
anyMatch는 불리언을 반환하므로 최종 연산이다.

### 5.4.2 프레디케이트가 모든 요소와 일치하는지 검사
allMatch 메서드는 anyMatch와 달리 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사한다.
```java
boolean isHealthy = menu.stream()
                        .allMatch(dish -> dish.getCalories() < 1000);
```
#### NONEMATCH
nonematch는 allMatch와 반대 연산을 수행한다. 위의 구문을 다시 재구성 하면 아래와 같다.
```java
boolean isHealthy = menu.stream()
                        .noneMatch(dish -> dish.getCalories() >= 1000);
```

anyMatch, noneMatch, allMatch 세 메서드는 스트림 **쇼트 서킷** 기법, 즉 자바의 &&, || 와 같은 연산을 활용한다.

> 쇼트서킷은 전체 스트림을 처리하지 않았더라도 결과를 반환할 수 있다. and 연산의 경우 하나만 거짓이면 나머지는 처리할 필요 없이 전체 결과가 거짓이 되는 상황을 말한다. 위 메서드들은 모든 스트림의 요소를 처리하지 않고도 결과를 반환할 수 있다. 마찬가지로 스트림의 모든 요소를 처리할 필요 없이 주어진 크기의 스트림을 생성하는 limit도 쇼트서킷 연산이다.

### 5.4.3 요소 검색
findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다. findAny 메서드를 다른 스트림 연산과 연결해서 사용할 수 있다.
```java
Optional<Dish> dish =
    menu.stream()
        .filter(Dish::isVegetarian)
        .findAny();
```
스트림 파이프라인은 내부적으로 단일 과정으로 실행할 수 있도록 최적화 된다. 즉, 쇼트서킷을 이용해서 결과를 찾는 즉시 실행을 종료한다.

#### Optional이란?
Optional<T> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다. findAny와 같은 경우 아무것도 찾지 못해 null을 반환하여 에러를 발생시킬 수 있어 만들어 졌다.

* isPresent()는 Optional이 값을 포함시 true, 아니면 false를 리턴
* ifPresent(Consumer<T.> block)은 값이 있으면 주어진 블록을 실행. Consumer 함수 인터페이스에는 T형식의 인수를 받으며 void를 반환하는 람다를 전달할 수 있음
* T get()은 값이 존재하면 값을 반환, 없으면 NoSuchElementException을 반환
* T orElse(T other)는 값이 있으면 값을 반환하고, 없으면 기본값을 반환
  
결론적으로, Optional을 사용하면 null인지 검사하는 로직을 사용하지 않아도 된다.

### 5.4.4 첫 번째 요소 찾기
리스트 또는 정렬된 연속 데이터로부터 생성된 스트림처럼 일부 스트림에는 **논리적인 아이템 순서**가 정해져 있을 수 있다.

첫 번째 요소를 찾으려면 findFirst()를 사용하여 찾을 수 있다.

```java
Optional<Integer> firstSquareDivisibleByThree = 
    someNumbers.stream()
                .map(n -> n * n)
                .filter(n -> n % 3 == 0)
                .findFirst();
```
> findFirst와 findAny는 언제 사용할까?
>
> findFirst와 findAny 메서드가 모두 필요한 이유는 병렬성 때문이다. 병렬 실행에서는 첫 번째 요소를 찾기 어렵다. 따라서 요소의 반환 순서가 상관 없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

## 5.5 리듀싱
지금까지 살펴본 연산은 불리언, void, Optional을 반환했다. 또한 collect로 모든 스트림의 요소를 리스트로 모으는 방법을 살펴봤다.

이 절에서는 리듀스 연산을 이용해서 스트림 요소를 조합해서 더 복잡한 질의를 표현하는 방법을 살펴본다. 이러한 질의를 수행하려면 Integer 같은 결과가 나올때 까지 스트림의 모든 요소를 반복적으로 처리해야 한다. 이러한 질의를 **리듀싱 연산**(모든 스트림 요소를 처리해서 값으로 반환)이라고 한다.

함수형 프로그래밍 언어 용어로는 이 과정이 마치 종이를 작은 조각이 될 때 까지 반복해서 접는 것과 비슷하다는 의미로 **폴드**라고 부른다.

### 5.5.1 요소의 합
아래 코드 예제를 통해 살펴본다.
```java
// for-each 루프를 이용
int sum = 0;
for (int x :numbers) {
    sum += x;
}
// sum 변수의 초기값 0
// 리스트의 모든 요소를 조합하는 연산(+)

// stream reduce를 이용한 연산
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
//초기값 0
// 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T>
// 위에서는 람다 표현식 (a, b) -> a + b 사용

```
메서드 참조를 이용해서 좀 더 간결하게 만들 수 있다.
```java
int sum = numbers.stream().reduce(0, Intger::sum);
// Intgeger 클래스에서 지원하는 정적 sum 메서드 사용
```

#### 초깃값 없음
초깃값을 받지 않도록 오버로드된 reduce도 있다. 그러나 이 reduce는 Optional 객체를 반환한다. 스트림에 아무 요소도 없는 상황이 있다면 초기값이 없으므로 reduce는 합계를 반환할 수 없다. 따라서 합계가 없음을 가리킬 수 있도록 Optional 객체로 감싼 결과를 변환한다.
```java
Optional<Integer> sum = numbers.stream().reduce((a, b) -> a + b);
```

### 5.5.2 최댓값과 최솟값
최댓값과 최솟값을 찾을 때도 reduce를 활용할 수 있다. reduce는 아래 두 인수를 받는다
* 초깃값
* 스트림의 두 요소를 합쳐서 하나의 값으로 만드는데 사용할 람다

reduce연산은 새로운 값을 이용해서 스트림의 모든 요소를 소비할 떄 까지 람다를 반복 수행하면서 최댓값을 생산한다. 다음처럼 reduce를 이용해서 스트림의 최댓값을 찾을 수 있다.
```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);

Optional<Integer> min = numbers.stream.reduce(Integer::min);

// Intger::min 대신 (x, y) -> x < y ? x : y를 사용해도 무방하지만 가독성이 더 좋음
```
map과 reduce를 연결하는 기법을 map-reduce 패턴이라고 하며, 쉽게 병렬화 하는 특징이 있다.

> REDUCE 메서드의 장점과 병렬화
>
> * reduce를 이용하면 내부 반복이 추상화 되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 반복적인 합게에서는 sum 변수를 공유해야 하므로 쉽게 병렬화하기 어렵다. 강제적으로 동기화시키더라도 결국 병렬화로 얻어야 할 이득이 스레드 간의 소모적인 경쟁 때문에 상쇄되어 버린다. (이를 가변 누적자 패턴이라고 한다.)
>
> * reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. (포크/조인 프레임워크를 이용하는데 7장에서 깊게 살펴본다.)
>
> * stream()을 parallelStream()으로 바꿔 스트림 작업을 병렬로 수행할 수 있다. 단, 람다의 상태가 바뀌지 말아야 하며, 연산이 어떤 순서로 실행되더라도 결과가 바뀌지 않아야 한다.

#### 스트림 연산: 상태 없음과 상태 있음
* map, filter  등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다. 따라서 사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다는 가정 하에 이들은 보통 상태가 없는, 즉 내부 상태를 갖지않는 연산이다.
* reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다. 스트림이 처리하는 요소 수와 관계없이 내부 상태의 크기는 **한정**(bound)되어 있다.
* 반면 sorted, distinct 같은 연산은 filter나 map처럼 스트림을 입력으로 받아 다른 스트림을 출력하는 것 처럼 보일 수 있다. 그러나  스트림의 요소를 정렬하거나 중복을 제거하려면 과거의 이력을 알고 있어야 한다. 어떤 요소를 출력 스트림으로 추가하려면 **모든 요소가 버퍼에 추가되어 있어야 한다.** 데이터 스트림의 크기가 크거나 무한(모든 소수를 포함하는 스트림을 역순 정렬)이라면 문제가 생길 수 있음 -> 첫번째 요소로 가장 큰 소수, 즉 세상에 존재하지 않는 수를 반환해야함. 이러한 연산을 **내부 상태를 갖는 연산**이라고 한다.
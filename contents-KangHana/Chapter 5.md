# Chapter 5 - 스트림 활용

이번 챕터에서는 스트림의 활용에에 대해 중점적으로 다룰 것입니다. 해당 챕터의 대주제는 다음과 같습니다.

- 필터링, 슬라이싱, 매칭
- 검색, 매칭, 리듀싱
- 특정 범위의 숫자와 같은 숫자 스트림 사용
- 다중 소스로부터 스트림 만들기
- 무한 스트림

<br>

지난 챕터에서는 스트림읕 통해 외부반복을 내부반복으로 바꾸는 방법을 알아보았습니다. 다음은 데이터 컬렉션 반복을 명시적으로 반복하는 **외부반복코드(for문 사용)** 입니다.

```java
List<Dish> vegetarianDishes = new ArrayList<>();
for (Dish d : menu) {
  if (d.isVegetarian()) {
    vegetarianDishes.add(d);
  }
}
```

명시적 반복 (for문을 사용한 외부반복) 대신 filter와 collect 연산을 지원하는 스틑림 API를 이용하여 데이터 컬렉션 반복을 내부적으로 처리할 수 있습니다.

다음처럼 filter 메소드에 필터링 연산을 인수로 넘기면 됩니다.

```java
import static java.util.stream.Collectors.toList;
List<Dish> vegetarianDishes = menu.stream()
                                  .filter(Dish::isVegetarian)
                                  .collect(toList());
```

데이터 처리 방법은 스트림 API가 관리하기 떄문에 편리하게 작업을 할 수 있습니다. 따라서 스트림 API 내부적으로 다양한 최적화가 이루어질 수 있습니다.

스트림 API는 내부 반복 뿐만 아니라 코드를 병렬로 실행할지 여부도 결정할 수 있습니다. 이런 일은 **순차적인 반복을 단일 스레드로 구현하는 외부 반복으로는 달성할 수 없습니다.**

<br>

해당 챕터에서는 스트림 API가 지원하는 다양한 연산을 살펴봅니다.

<br>
<hr>
<br>

## ■ 1. 필터링

<br>

스트림의 요소를 선택하는 방법 즉, 프레디케이트 필터링 방법과 고유 요소만 필터링하는 방법을 설명할 예정입니다.

<br>

### ▷ 프레디케이트로 필터링

스트림 인터페이스는 filter 메소드를 지원합니다. filter 메소드는 **프레디케이트(불리언을 반환하는 함수)** 를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환합니다.

예시는 다음과 같습니다.

```java
Lsit<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarin) // 프레디케이트를 인수로 받음
                                .collect(toList());
```

<br>

### ▷ 고유 요소 필터링

스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메소드도 지원합니다. 이떄 고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정됩니다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct() // 필터링 된 요소들을 중복없이 뽑아냄 (고유 요소)
        .forEach(System.out::println);
```

<br>
<hr>
<br>

## ■ 2. 스트림 슬라이싱

<br>

스트림의 요소를 선택하고 스킵하는 메소드가 있습니다. 프레디케이트를 이용하는 방법, 스트림의 처음 몇 개의 요소를 무시하는 방법, 특정 크기로 스트림을 줄이는 방법 등 다양한 방법을 이용해 효율적으로 작업을 수행할 수 있습니다.

<br>

### ▷ 프레디케이트를 이용한 슬라이싱

자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 새로운 메소드를 지원합니다.

<br>

### **TAKEWHILE** 활용

takeWhile연산을 이용하면 조건에 맞지 않는 요소가 들어왔을 떄, 반복 작업을 중단하는 역할을 하는 연산입니다. 이는 **이미 정렬되어있는 리스트에서만 사용**할 수 있습니다!

<br>

다음과 같은 특별한 요리 목록을 갖고 있다고 가정해봅시다.

```java
List<Dish> specialMenu = Arrays.asList(
  new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
  new Dish("prawns", false, 300, Dish.Type.FISH),
  new Dish("rice", true, 350, Dish.Type.OTHER),
  new Dish("chicken", false, 400, Dish.Type.MEAT)
);
```

위 목록에서 칼로리 부분은 이미 정렬이 되어있습니다. 리스트가 이미 정렬되어 있다는 사실을 이용하여 300칼로리보다 크거나 같은 요리가 나왔을 떄 takeWhile연산을 이용하여 반복 작업을 중단할 수 있습니다.

```java
List<Dish> slicedMenu1 = specialMenu.stream()
                                    .takeWhile(dish -> dish.getCalories() < 300)
                                    .collect(toList());
```

<br>

### **DROPWHILE** 활용

나머지 요소를 선택할때, 즉 300 칼로리보다 큰 요소를 탐색할 때에는 dropWhile을 이용해주면 됩니다.

```java
List<Dish> slicedMenu2 = specialMenu.stream()
                                    .dropWhile(dish -> dish.getCalories() < 300)
                                    .collect(toList());
```

dropWhile은 takeWhile과 정반대의 작업을 수행합니다. dorpWhile은 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버립니다. 거짓이 되면 그 지점에서 _작업을 중단_ 하고, **남은 모든 요소를 반환**합니다.

이는 무한한 남은 요소를 가진 무한 스트림에서도 동작합니다!

<br>

### ▷ 스트림 축소

<br>

### **limit(n)**

limit은 스트림 크기를 지정해주는 메소드입니다. 스트림은 주어진 값 이하의 크기를 갖게 됩니다.

```java
List<Dish> dishes = specialMenu.stream()
                                .filter(dish -> dish.getCalories() > 300)
                                .limit(3)
                                .collect(toList());
```

프레디케이트와 일치하는 처음 세 요소를 선택한 후 즉시 결과를 반환합니다. 소스가 정렬되어 있지 않았다면 limit의 결과도 정렬되지 않은 상태로 반환됩니다.

<br>

### ▷ 요소 건너뛰기

<br>

### **skip(n)**

skip은 처음 n개 요소를 제외한 스트림을 반환하는 메소드입니다. n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환됩니다.

```java
List<Dish> dishes = specialMenu.stream()
                                .filter(dish -> dish.getCalories() > 300)
                                .skip(2)
                                .collect(toList());
```

<br>
<hr>
<br>

## ■ 3. 매핑

<br>

특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행됩니다. 스트림 API의 map과 flatMap 메소드는 특정 데이터를 선택하는 기능을 제공합니다.

<br>

### □ 스트림의 각 요소에 함수 적용하기

스트림은 **함수를 인수로** 받는 **map 메소드**를 지원합니다. 인수로 제공된 함수는 각 요소에 적용되고 함수를 적용한 결과가 새로운 요소로 매핑됩니다.

```java
List<String> dishNames = menu.stream()
                            .map(Dish::getName)
                            .collect(toList());
```

getName은 문자열을 반환하므로 map 메소드의 출력 스트림은 Stream<String> 형식을 갖습니다.
각 요리명의 길이를 알고싶으면 다음처럼 다른 map 메소드를 연결할 수도 있습니다.

```java
List<Integer> dishNameLengths = menu.stream()
                                    .map(Dish::getName)
                                    .map(String::length)
                                    .collect(toList());
```

<br>

### □ 스트림 평면화

위에서 했던 것들을 응용하여 리스트에서 고유 문자로 이루어진 리스트를 반환해봅시다. 예를 들어 ["Hello", "World"] 리스트가 있다면 결과로 ["H","e","l","o","W","r","d"] 를 포함하는 리스트가 반환되어야 합니다.

<br>

### **- 잘못된 풀이**

리스트에 있는 각 단어를 문자로 매핑한 다음 distinct로 중복된 문자를 필터링해 풀 수 있을거라고 생각했을수도 있습니다. (제가 그랬습니다) 즉, 코드로 작성하면 다음과 같습니다.

```java
words.stream()
      .map(word -> word.split(""))
      .distinct()
      .collect(toList());
```

하지만 위 코드는 각 단어의 Stirng[]을 반환한다는 문제점이 있습니다. 즉, map 메소드가 반환한 스트림의 형식은 Stream<String[]> 입니다. 반면 우리가 원하는 반환 형식은 **Stream<String.>** 입니다.

<br>

또 다른 경우로는 map 과 Arrays.stream을 활용한 경우가 있습니다. 다음 코드처럼 문자열을 받아 스트림을 만드는 Arrays.stream 메소드를 활용합니다.

```java
String[] arrayOfwords = {"Goodbye", "World"};
Stream<String> streamOfwords = Arrays.stream(arrayOfwords);

words.stream()
      .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
      .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성
      .distinct()
      .collect(toList());
```

이렇게 써도 결국 스트림 리스트(List<Stream<String>>)이 만들어지므로 문제는 해결되지 않습니다.

문제를 해결하기 위해서는 각 단어를 개별 문자열로 이루어진 배열로 만든 후 각 배열을 별도의 스트림으로 만들어야 합니다.

<br>

### **-flatMap 사용**

flatMap을 사용하면 다음처럼 문제를 해결할 수 있습니다.

```java
List<String> uniqueCharacters = words.stream()
                                      .map(word -> word.split("")) // 배열변환
                                      .flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화
                                      .distinct()
                                      .collect(toList())
```

**flatMap**은 각 배열을 스트림이 아니라 **_스트림의 콘텐츠로 매핑_** 합니다. 즉, 하나의 평면화된 스트림을 반환합니다.

요약하면 flatMap 메소드는 스트림의 각 값을 다른 스트림으로 만든 후, 모든 스트림을 하나의 스트림으로 만드는 기능을 수행합니다.

<br>
<hr>
<br>

## ■ 4. 검색과 매칭

<br>

특성 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용됩니다. 스트림 API는 allMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 유틸리티 메소드를 제공합니다.

<br>

### ▷ 프레디케이트가 적어도 한 요소와 일치하는지 확인 \_ **anyMatch** 메소드

적어도 한 요소와 일치하는지 확인할 때에는 anyMatch 메소드를 활용합니다.

```java
if (menu.stream().anyMatch(Dish::isVegetarian)) {
  System.out.println("The menu is (somewhat) vegetarian friendly!");
}
```

anyMatch는 **불리언을 반환**합니다.

<br>

### ▷ 프레디케이트가 모든 요소와 일치하는지 확인

<br>

### **allMatch** 메소드

스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사합니다. 예를 들어 메뉴가 건강식(모든 요리가 1000칼로리 이하면 건강식)인지 확인할 수 있습니다.

```java
boolean isHealthy = menu.stream()
                        .allMatch(dish -> dish.getCalories() < 1000);
```

<br>

### **noneMatch** 메소드

noneMatch는 allMatch와 반대되는 연산을 수행합니다. 즉, noneMatch는 주어진 프레디케이트와 일치하는 요소가 없는지 확인합니다. 위 예제를 noneMatch를 이용해 다시 구현할 수 있습니다.

```java
boolean isHealthy = menu.stream()
                        .noneMatch(dish -> dish.getCalories() >= 1000);
```

anyMatch, allMatch, noneMatch 세 메소드는 스트림 **쇼트서킷 기법**, 즉 자바의 &&, ||와 같은 연산을 활용합니다.

> **쇼트서킷**
>
> <br>
>
> 모든 스트림의 요소를 처리하지 않고도 결과를 반환할 수 있는 상황을 쇼트서킷이라고 합니다. 즉, 원하는 요소를 찾았으면 즉시 결과를 반환할 수 있습니다.
>
> 마찬가지로 스트림의 모든 요소를 처리할 필요 없이 주어진 크기의 스트림을 생성하는 limit도 쇼트서킷 연산입니다.
> <br>
>
> \_

<br>

### ▷ 요소 검색

findAny 메소드는 현재 스트림에서 임의의 요소를 반환합니다. 이 메소드를 다른 스트림 연산과 연결해서 사용할 수 있습니다.

**스트림 파이프라인**은 내부적으로 **_단일 과정으로 실행할 수 있도록 최적화_** 됩니다. 즉, 쇼트서킷을 이용해서 결과를 찾는 즉시 실행을 종료합니다.

```java
Optional<Dish> dish =
    menu.stream()
        .filter(Dish::isVegetarian)
        .findAny();
```

위 코드에서 Optional은 무엇일까요?

<br>

### **Optional** 이란?

Optional<T.> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스입니다. findAny처럼 null요소를 반환할 수 있는 메소드가 쉽게 에러를 발생시킬 수 있기 때문에 만들어졌다고 합니다.

즉, Optional은 값이 존재하는지 확인하고, 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공합니다.

- isPresent() : Optional이 값을 포함하면 true 반환, 아니면 false 반환
- ifPresent(Consumer<T.> block) : 값이 있으면 주어진 블록을 실행합니다. 이때 Consumer 함수형 인터페이스는 void를 반환하는 람다를 전달할 수 있습니다.
- T get() : 값이 존재하면 값을 반환, 없으면 NoSuchElementException을 일으킵니다.
- T orElse(T other) : 값이 있으면 값을 반환하고 없으면 기본값을 반환합니다.

Optional을 사용하면 값이 null인지 검사할 필요가 없습니다.

### ▷ 첫번째 요소 찾기

리스트나 정렬된 연속 데이터로부터 생성된스트림처럼 일부 스트림에는 **논리적인 아이템 순서가 정해져 있을 수도** 있습니다.

이런 스트림에서 첫 번째 요소를 찾고싶으면 findFirst를 이용할 수 있습니다.

```java
List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                            .map(n -> n * n)
                                                            .filter(n -> n % 3 == 0)
                                                            .findFirst();
```

> **findFirst**와 **findAny**는 언제 활용할까 ?
>
> <br>
>
> 이 둘이 모두 필요한 이유는 **병렬성** 때문입니다. 병렬실행에서는 첫 번째 요소를 찾기 쉽지 않기 때문에, 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용합니다.
>
> <br>
> _

<br>
<hr>
<br>

## ■ 5. 리듀싱

<br>

지금까지 살펴본 최종 연산은 아래와 같은 타입을 반환했습니다.

- 불리언 (allMatch 등)
- void (forEach)
- Optional 객체 (findAny 등)
- collect로 모든 스트림의 요소를 리스트로

지금부터는 리듀스 연산을 이용해서 스트림 요소를 조합해 더 복잡한 질의를 표현하는 방법을 설명하려고 합니다.

이러한 질의를 수행하기 위해서는 Integer 같은 결과가 나올 때까지 스트림의 모든 요소를 반복적으로 처리해야합니다. 이러한 질의를 **리듀싱 연산**이라고 합니다. (함수형 프로그래밍 언어 용어로는 **폴드**라고도 합니다.)

<br>

### **▷ 요소의 합**

다음은 for-each 루프를 이용해 리스트의 숫자 요소를 더하는 코드입니다.

```java
int sum = 0;
for (int x : numbers) {
  sum += x;
}
```

해당 코드에서 파라미터를 두 개 사용했습니다.

- sum 변수의 초깃값 0
- 리스트의 모든 요소를 조합하는 연산

위와 같은 상황에서 reduce를 이용하면 애플리케이션의 반복된 패턴을 추상화 할 수 있습니다.다음처럼 스트림의 모든 요소를 더할 수 있습니다.

```java
int sum = numbers.stream().reduce(0, (a,b) -> a + b);
```

reduce는 두 개의 인수를 갖습니다.

- 초깃값 \_ 초깃값 0으로 설정
- 함수 \_ 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T.>, 예제에서는 람다식 사용

reduce로 다른 람다, 예를 들면 (a,b) -> a\*b를 넘겨주면 모든 요소에 곱셈을 적용할 수 있습니다.

### **_예시_**

숫자 스트림에 4,5,3,9가 있고, reduce(0, (a, b) -> a + b) 라면,

0 + 4 = 4

4 + 5 = 9

9 + 3 = 12

12 + 9 = 21

순서로 연산됩니다.

이 과정은 메소드를 이용해 더 간결하게 만들 수 있습니다. 자바 8에서는 Integer 클래스에 **정적 sum 메소드**를 제공합니다.

```java
int sum = numbers.stream().reduce(0, Integer::sum);
```

<br>

### **초깃값 없음**

초깃값을 받지 않게 오버로드된 reduce도 있습니다. 이 reduce는 스트림에 아무 요소도 없는 상황이 있을 수 있고, 그로 인해 합계가 없을 수 있기 때문에 **Optional 객체를 반환**합니다.

```java
Optional<Integer> sum = numbers.stream().reduce((a, b) -> a + b);
```

<br>

### **▷ 최댓값과 최솟값**

reduce는 두 인수를 받는다고 했습니다.

- 초깃값
- 스트림의 두 요소를 합쳐서 하나의 값으로 만드는 데 사용할 람다

따라서 두 요소에서 최댓값을 반환하는 람다만 있으면 최댓값을 구할 수 있습니다. 이때 Integer의 max, min 메소드를 사용합니다.

이 경우가 초깃값이 없는 경우에 해당 됩니다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> max = numbers.stream().reduce((x, y) -> x > y ? x : y);

Optional<Integer> min = numbers.stream().reduce(Integer::min);
Optional<Integer> min = numbers.stream().reduce((x, y) -> x < y ? x : y);
```

### + **맵 리듀스 패턴**

map과 reduce를 연결하는 기법을 이르는 말로 쉽게 병렬화가 가능하다는 특징이 있습니다.

예시코드는 아래와 같습니다.

```java
int cnt = menu.stream()
              .map(d -> 1)
              .reduce(0, (a,b) -> a + b);
```

<br>

> ### **reduce 메소드의 장점과 병렬화**
>
> <br>
>
> reduce를 이용하면 내부 반복이 추상화 되면서 내부구현에서 병렬로 reduce를 실행할 수 있게 됩니다. 반복적 합계에서는 sum 변수를 공유해야 하기 때문에 병렬화가 어렵습니다.
>
> 추가로 7장에서 스트림의 모든 요소를 더하는 코드를 병렬로 만드는 방법인 parallelStream()을 설명할 예정입니다. 이를 이용하는 것만으로도 별다른 노력 없이 병렬성을 얻을 수 있습니다.
>
> \_

<br>
<hr>
<br>

## ■ 6. 숫자형 스트림

<br>

reduce를 이용해서 다음처럼 합계를 구할 수 있습니다.

```java
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .reduce(0, Integer::sum);
```

사실 위 코드에는 박싱비용이 숨어있습니다. 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야합니다.

```java
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .sum();
```

이렇게 쓰는건 잘못된 코드입니다. map 메소드는 Stream<T.>를 생성하기 때문에 Stream<Dish.>같은 것이 오면 sum 연산을 할 수 없어 메소드 자체가 생성되어 있지 않습니다.

다행히 스트림 API 숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림**을 제공합니다.

<br>

### **▷ 기본형 특화 스트림**

자바8에서는 세 가지 기본형 특화 스트림을 제공합니다. 스트림 API는 박싱 비용을 피할 수 있도록 세 가지 스트림을 제공합니다.

- IntStream : int 요소에 특화
- DoubleStream : double 요소에 특화
- LongStream : long 요소에 특화

특화 스트림은 필요할 때 다시 **객체 스트림으로 복원하는 기능도 제공**합니다. 그렇다고 특별한 추가 기능을 제공하지는 않고, 오직 **_박싱 과정에서 일어나는 효율성과 관련_** 있습니다.

<br>

### **숫자 스트림으로 매핑**

스트림을 특화 스트림으로 변활할 때는 다음 세 개의 메소드를 가장 많이 사용합니다.

- mapToInt
- mapToDouble
- mapToLong

이들은 map과 정확히 같은 기능을 수행하지만, **Stream 대신 특화 스트림을 반환**합니다.

```java
int calories = menu.stream()
                    .mapToInt(Dish::getCalories)
                    .sum();
```

sum 메소드는 스트림이 비어있으면 기본값 0을 반환합니다.

<br>

### **객체 스트림으로 복원하기**

**boxed 메소드를** 이용하면 특화 스트림을 일반 스트림으로 변환할 수 있습니다.

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intStream.boxed(); // 숫자 스트림을 스트림으로 변환
```

<br>

### **기본값 : OptionalInt**

스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 구별하기 위해 Optional 클래스를 활용합니다.

**Optional을 Integer, String 등의 참조 형식으로 파라미터화** 할 수 있습니다. 또, **OptionalInt, OptionalDouble, OptionalLong** 세 가지 기본형 특화 스트림 버전도 제공합니다.

```java
OptionalInt maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

int max = maxCalories.orElse(1); // 최댓값이 없는 상황에 사용할 기본값을 명시적으로 정의 가능
```

<br>

### **▷ 숫자 범위**

1에서 100사이 숫자를 생성하는 등 특정 범위의 숫자를 이용해야 하는 상황이 종종 발생합니다. 자바 8의 **IntStream과 LongStream**에서는 **range와 rangeClosed라는 정적 메소드를 제공**합니다.

- **_range_** : 시작값과 종료값을 인수로 포함하지 않음
- **_rangeClosed_** : 시작값과 종료값을 인수로 포함

다음 코드처럼 1부터 100까지 숫자를 만들 수 있습니다.

```java
IntStream evenNumbers = IntStream.rangeClosed(1,100)
                                  .filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count());
```

rangeClosed 대신 range를 쓰면 100이 포함되지 않기 때문에 49개를 반환합니다.

<br>

### **▷ 숫자 스트림 활용 : 피타고라스 수**

<br>

### **피타고라스의 수**

a*a + b*b = c\*c 를 만족하는 세 개의 정수 (a,b,c) 를 의미합니다.

<br>

### **세 수 표현하기**

세 수는 int 배열을 사용하는 것이 좋습니다.

<br>

### **좋은 필터링 조합**

세 수 중에서 a,b 두 수만 제공했다면, 두 수가 피타고라스 수의 일부가 될 수 있는 좋은 조합인지 확인하려면, **(a*a + b*b)의 제곱근이 정수인지 확인**해야 합니다. 이를 코드로 작성하면 다음과 같습니다.

```java
Math.sqrt(a*a + b*b) % 1 == 0;
```

이를 filter에 적용하면 다음과 같이 활용이 가능합니다.

```java
filter(b -> Math.sqrt(a*a + b*b) % 1 == 0);
```

위 코드에서 a가 주어진다면 피타고라스 수를 구성하는 모든 b를 필터링할 수 있습니다.

<br>

### **집합 생성**

필터로 좋은 조합을 갖는 a,b를 선택할 수 있게 되었습니다. 이제 세 번째 수를 찾으면 됩니다!

다음처럼 map을 이용해 각 요소를 피타고라스 수로 변환할 수 있습니다.

```java
stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0) // 적당한 b를 필터링
      .map(b -> new int[] {a, b, (int) Math.sqrt(a*a + b*b)}); // 피타고라스 수를 배열로 만들기
```

<br>

### **b값 생성**

필터를 만들었으니, 필터에 넣을 b값을 생성하면 됩니다. 이때는 Stream.rangeClosed를 이용해줍니다.

```java
IntStream.rangeClosed(1,100)
          .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
          .boxed() // Stream<Integer> 로 복원
          .map(b -> new int[] {a, b, (int) Math.sqrt(a*a + b*b)});
```

위 코드처럼 boxed로 바꿀 수도 있고, mapToObj로 재구현할 수도 있습니다.

```java
IntStream.rangeClosed(1,100)
          .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
          .mapToObj(b -> new int[] {a, b, (int) Math.sqrt(a*a + b*b)});
```

<br>

### **a값 생성과 최종 완성**

마찬가지로 a를 생성할 수 있습니다. 아래 코드는 최종 완성 코드입니다.

```java
Stream<int[]> pythagoreanTriples
    = IntStream.rangeClosed(1,100).boxed()
                .flatMap(a ->
                    IntStream.rangeClosed(a, 100) // a가 아닌 1을 쓰면 중복 발생 가능
                              .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
                              .mapToObj(b ->
                                  new int[]{a, b, (int)Math.sqrt(a * a + b * b)})
                );
```

여기서 **flatMap**은 a를 이용해 만든 세 수의 스트림과 스트림 a의 값을 매핑하면서 만들어지는 스트림을 **하나의 평준화된 스트림으로 만들어주는 역할**을 합니다.

<br>

### **개선할 점**

위 코드에서는 제곱근을 두 번 계산합니다. 따라서 (a*a, b*b, a*a+b*b) 형식을 만족하는 세 수를 만든 후 우리가 원하는 조건에 맞는 결과만 필터링하는 것이 더 최적화 된 방법입니다.

```java
Steam<double[]> pythagoreanTriples
    = IntStream.rangeClosed(1,100).boxed()
                .flatMap(a -> IntStream.rangeClosed(a, 100)
                .mapToObj(
                  b -> new double[]{a, b, Math.sqrt(a*a + b*b)})  // 세 수 만들어서 배열에
                .filter(t -> t[2] % 1 == 0)); // 세번째 요소는 정수여야함
```

<br>
<hr>
<br>

## ■ 7. 스트림 만들기

<br>

스트림을 얻거나 만들 수 있는 방법은 다양했습니다.

- stream 메소드로 컬렉션에서 스트림 얻을 수 있음
- 범위숫자에서 스트림 만들 수 있음

이 외에도 일련의 값, 배열, 파일, 함수를 이용한 무한 스트림 만들기 방법을 설명할 것입니다.

<br>

### **▷ 값으로 스트림 만들기**

**임의의 수를 인수로** 받는 정적 메소드 **Stream.of**를 이용해 스트림을 만들 수 있습니다. 아래 코드는 스트림의 모든 문자열을 대문자로 변환한 후 문자열을 출력하는 코드입니다.

```java
Stream<String> steam = Stream.of("Modern","Java","in","Action");
stream.map(String::toUpperCase).forEach(System.out::println);
```

아래 코드처럼 **empty 메소드**를 이용해 **스트림을 비울 수** 있습니다.

```java
Stream<String> emptyStream = Stream.empty();
```

<br>

### **▷ null이 될 수 있는 객체로 스트림 만들기**

자바9에서는 null이 될 수 있는 객체를 스트림으로 만들 수 있는 메소드가 추가되었습니다. 어쩔때는 null이 될 수 있는 객체를 스트림으로 만들어야할 떄도 있습니다.

null이 발생할 수 있는 메소드를 스트림에 활용하려면 다음처럼 null을 명시적으로 확인해야 했습니다.

```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);
```

**Stream.ofNullable**을 통해 다음처럼 코드를 구현할 수 있습니다.

```java
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

null이 될 수 있는 객체를 포함하는 스트림값을 flatMap과 함께 사용하는 상황에서는 이 패턴을 더 유용하게 사용할 수 있습니다.

```java
Stream<String> value = Stream.of("config", "home", "user")
                              .flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```

<br>

### **▷ 배열로 스트림 만들기**

배열을 인수로 받는 정적 메소드 **Arrays.stream**을 이용해 스트림을 만들 수 있습니다. 예를 들어 다음처럼 기본형 int로 이루어진 배열을 IntStream으로 변환할 수 있습니다.

```java
int[] numbers = {2,3,5,7,11,13};
int sum = Arrays.stream(numbers).sum();
```

<br>

### **▷ 파일로 스트림 만들기**

파일 처리 등 I/O 연산에 사용하는 자바의 NIO API도 스트림 API를 활용할 수 있도록 업데이트 되었습니다. **java.nio.file.Files**의 많은 정적 메소드가 스트림을 반환합니다.

예를 들어 Files.lines는 주어진 파일의 행 스트림을 문자열로 반환합니다.

<br>

지금까지 배운 스트림 연산을 이용해 다음 코드처럼 파일에서 고유한 단어 수를 찾는 프로그램을 만들 수 있습니다.

```java
long uniqueWords = 0;
try(Stream<String> lines = Files.lines(Paths.get("data.txt", Charset.defaultCharset()))) {
  uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                      .distinct()
                      .count();
} catch(IOException e) {

}
```

**Files.lines**로 **파일의 각 행 요소를 반환**하는 스트림을 얻을 수 있습니다. 스트림 소스가 I/O 자원이라 메모리 누수를 막기 위해서는 자원을 닫아야 합니다.

기존에는 finally 블록에서 자원을 닫았지만, **Stream 인터페이스는 AutoCloseable 인터페이스를 구현**하였기 때문에 **try 블록 내 자원은 자동으로 관리**됩니다.

<br>

### **▷ 함수로 무한 스트림 만들기**

스트림 API는 **함수에서 스트림을 만들 수 있는 두 정적 메소드 Stream.iterate와 Stream.generate를 제공**합니다. 두 연산을 이용해 무한 스트림을 만들 수 있습니다.

- **_무한 스트림_** 이란, 크기가 고정되지 않은 스트림을 의미합니다.

iterate와 generate로 만든 스트림은 요청할 때마다 주어진 함수를 이용해 값을 만듭니다.

<br>

### **iterate 메소드**

```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```

**iterate 메소드**는 초깃값과 람다를 인수로 받아서 요청할 때마다 값을 생산하여 무한 스트림을 만들어냅니다.

이러한 스트림을 **언바운드 스트림**이라고 합니다.

일반적으로 연속된 일련의 값을 만들때는 iterate를 사용합니다.

<br>

자바9의 iterate 메소드는 프레디케이트를 지원합니다. 두 번째 인수로 프레디케이트를 받아 언제까지 작업을 수행할 것인지의 기준을 설정할 수 있습니다.

```java
IntStream.iterate(0, n -> n < 100, n -> n + 4)
          .forEach(System.out::println);
```

위처럼 쓰는 것도 좋지만, **takeWhile**을 이용하는 것도 좋습니다.

```java
IntStream.iterate(0, n -> n + 4)
          .takeWhile(n -> n < 100)
          .forEach(System.out::println);
```

<br>

### **generate 메소드**

**generate**는 iterate와 달리 생산된 각 값을 **연속적으로 계산하지 않습니다**. generate는 Supplier<T>를 인수로 받아서 새로운 값을 생산합니다.

```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```

위 코드에서 사용한 발행자(메소드 참조 Math.random)은 상태가 없는 메소드, 즉 나중에 계산에 사용할 어떤 값도 저장해두지 않습니다. **병렬코드에서는 발행자에 상태가 있으면 안전하지 않다**고 합니다. 따라서 상태를 갖는 발행자는 피해야합니다.

IntStream을 이용하면 박싱 연산 문제를 피할 수 있습니다. IntStream의 generate 메소드는 Supplier<T.> 대신에 IntSupplier을 인수로 받습니다.

```java
IntStream ones = IntStream.generate(() -> 1);
```

아래 코드처럼 IntSupplier 인터페이스에 정의된 getAsInt를 구현하는 객체를 명시적으로 전달할 수도 있습니다. **getAsInt 메소드**의 연산을 커스터마이즈 할 수 있는 **상태 필드를 정의**할 수 있다는 점이 위 코드와의 차이점입니다.

그러나 상태를 바꾼다면 **부작용이 생길 수 있습니다**.

아래는 다음 피보나치 요소를 반환하도록 IntSupplier를 구현한 코드입니다.

```java
IntSupplier fib = new IntSupplier() {
  private int previous = 0;
  private int current = 1;
  public int getAsInt() {
    int oldPrevious = this.previous;
    int nextValue = this.previous + this.current;
    this.previous = this.current;
    this.current = nextValue;
  }
};

IntStream.generate(fib).limit(5).forEach(System.out::println);
```

위 코드에서는 IntSupplier 인스턴스를 만들었습니다. 만들어진 객체는 기존 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어있는지 추적하므로 **가변 상태 객체**입니다.

하지만 iterate는 새로운 값을 생성하면서도 기존 상태를 바꾸지 않는 순수한 불변상태를 유지합니다.

스트림을 **병렬로 처리**할 때는 **_불변 상태 기법을 고수_** 해야합니다.

<br>
<hr>
<br>

## ■ 8. 마치며

- 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
- **filter, distinct, takeWhile, dropWhile, skip, limit 메소드로 스트림을 필터링하거나 자를 수 있다.**
- 소스가 정렬되어 있다는 사실을 알고 있을 때 takeWhile과 dropWhile 메소드를 효과적으로 사용할 수 있다.
- map, flatMap 메소드로 스트림의 요소를 추출하거나 변환할 수 있다.
- findFirst, findAny 메소드로 스트림의 요소를 검색할 수 있다. allMatch, noneMatch, anyMatch 메소드를 이용해 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색할 수 있다.
- 이들 메소드는 **쇼트서킷**, 즉 **결과를 찾는 즉시 반환**하며, 전체 스트림을 처리하지는 않는다.
- reduce 메소드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다. 예를 들어 reduce로 스트림의 최댓값이나 모든 요소의 합계를 계산할 수 있다.
- filter, map 등은 상태를 저장하지 않는 상태 없는 연산이다. reduce같은 연산은 값을 계산하는 데 필요한 상태를 저장한다. sorted, distinct 등의 메소드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야한다. 이런 메소드를 상태 있는 연산이라고 부른다.
- IntStream, DoubleStream, LongStream은 기본형 특화 스트림이다. 이들 연산은 각각의 기본형에 맞게 특화되어 있다.
- 컬렉션 뿐 아니라 값, 배열, 파일, iterate와 generate 같은 메소드로도 스트림을 만들 수 있다.
- 무한한 개수의 요소를 가진 스트림을 무한스트림이라고 한다.

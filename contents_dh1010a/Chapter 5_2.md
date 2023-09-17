# Chapter 5 - 스트림 활용(2)

## 5.6 실전 연습
```java
 // 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리
 List<Transaction> tr2011 = 
    transactions.stream()
                .filter(transaction -> transaction.getYear() == 2011)
                .sorted(comparing(Transacaction::getValue))
                .collect(toList());
//거래자가 근무하는 모든 도시를 중복 없이 나열
List<String> cities = 
    transactions.stream()
                .map(transaction -> transaction.getTrader().getCity())
                .distinct() // .collect(toSet())
                .collect(toList());
//케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬
List<Trader> traders = 
    transactions.stream()
                .map(Transaction::getTrader)
                .filter(trader -> trader.getCity().equals("Cambridge"))
                .distinct()
                .sorted(comparing(Trader::getName))
                .collect(toList());
// 모든 거래자의 이름을 알파벳 순으로 정렬해서 반환
 String traderStr =
                transactions.stream()
                            .map(Transaction::getTrader)
                            .map(Trader::getName)
                            .distinct()
                            .sorted()
                            .reduce("", (a, b) -> a + b); // .collect(joining());

// Miilan에 거래자가 있는가?
boolean existed =
        transactions.stream()
                    .map(Transaction::getTrader) // .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
                    .map(Trader::getCity)
                    .anyMatch(city -> city.equals("Milan"));

// Cambridge에 거주하는 거래자의 모든 트랜잭션 값을 출력하시오.
transactions.stream()
            .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
            .map(Transaction::getValue)
            .forEach(System.out::println);

// 전체 트랜잭션 중 최댓값은 얼마인가?
Optional<Integer> max = transactions.stream()
                                    .max(Comparator.comparing(Transaction::getValue))
                                    .map(Transaction::getValue);

// 전체 트랜잭션 중 최솟값은 얼마인가?
Optional<Integer> min = transactions.stream()
                                    .map(Transaction::getValue)
                                    .reduce(Integer::min);
```

## 5.7 숫자형 스트림
다음처럼 메뉴의 칼로리 합계를 계산할 수 있다.
```java
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .reduce(0, Integer::sum);
```
위 코드에는 사실 박싱 비용이 숨어 있다. 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱 해야한다. 

아래처럼 직접 sum 메서드를 호출하면 더 좋지만, 그럴수는 없다. menu 메서드가 Stream<T\>를 생성하기 때문이다. menu처럼 Stream<Dish\> 형식의 요소만 있다면 sum이란 연산을 수행할 수 없기 때문이다. 하지만 스트림 API숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림**을 제공한다.
```java
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .sum();
```

### 5.7.1 기본형 특화 스트림
자바 8에서는 세 가지 기본형 특화 스트림을 제공한다.
1. IntStream: int요소에 특화
2. DoubleStream: Double요소에 특화
3. LongStream: long요소에 특화

각각의 인터페이스는 숫자 스트림의 합꼐를 계산하는 sum, 최댓값 요소를 검색하는 max 같이 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공한다. 또한 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공한다. **특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련** 있으며 스트림에 추가 기능을 제공하지 않는다.

#### 숫자 스트림으로 매핑
mapToInt, mapToDouble, mapToLong 세가지 메서드를 가장 많이 사용한다. 이들 메서드는 map과 정확히 같은 기능을 수행하지만, Stream<T> 대신 특화된 스트림을 반환한다.
```java
int calories = menu.stream()
                    .mapToInt(Dish::getCalories)
                    .sum();
```
스트림이 비어있으면 0을 반환하며, max/min/average 등 다양한 유틸리티 메서드도 지원한다.

#### 객체 스트림으로  복원하기
원상태인 특화되지 않은 스트림으로 복원하고 싶다면, boxed를 이용하여 일반 스틀미으로 변환할 수 있다.
```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intStream.boxed(); // 숫자 스트림을 스트림으로 변환
```

#### 기본값: OptionalInt
기본값이 0일때 스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 구별하기 위해 Optional클래스를 활용할 수 있다. Optional을 Integer, String등의 참조 형식으로 파라미터화 할 수 있다. 또한 OptionalInt, OptionalDouble, OptionalString 세 가지 기본형 특화 스트림 버전도 제공한다.
```java
OptionalInt maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

int max = maxCalories.orElse(1); // 값이 없을 때 기본 최댓값을 명시적으로 정의 가능
```

### 5.7.2 숫자 범위
특정 범위의 숫자를 이용하기 위해서 IntStream과 LongStream에서 range, rangeClosed라는 두 가지 정적 메서드를 제공한다. range 메서드는 시작값과 종료값이 결과에 포함되지 않는 반면, rangeClosed는 시작값과 종료값이 결과에 포함된다는 점이 다르다.
```java
IntStream evenNumbers = IntStream.rangeClosed(1,100)
                                  .filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count()); //50개의 짝수
```
위 코드를 이용해 1부터 100까지의 숫자를 만들 수 있다. range()의 경우라면 100이 포함되지 않아 49개의 결과를 반환할 것이다.

### 5.7.3 숫자 스트림 활용 : 피타고라스 수

#### 피타고라스 수
aa + bb = c*c 를 만족하는 세 개의 정수 (a,b,c) 를 의미한다.

#### 세 수 표현하기
세 수를 표현할 클래스를 정의하는 것보다는 세 요소를 갖는 int배열을 사용하는 것이 좋다.

#### 좋은 필터링 조합
세 수 중에서 a,b 두 수만 제공했다면, 두 수가 피타고라스 수의 일부가 될 수 있는 좋은 조합인지 확인하려면, (a\*a + b\*b)의 제곱근이 정수인지 확인해야 한다.
```java
Math.sqrt(a*a + b*b) % 1 == 0;
//이를 filter에 적용
filter(b -> Math.sqrt(a*a + b*b) % 1 == 0);
```
위 코드에서 a가 주어진다면 피타고라스 수를 구성하는 모든 b를 필터링할 수 있다.

#### 집합 생성
map을 이용해서 각 요소를 피타고라스 수로 변환할 수 있다.
```java
stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0) =
      .map(b -> new int[] {a, b, (int) Math.sqrt(a*a + b*b)});
```

#### b값 생성
이제 b값을 생성한다. Stream.rangeClosed를 활용한다.
```java
IntStream.rangeClosed(1,100)
          .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
          .boxed() // Stream<Integer> 로 복원
          .map(b -> new int[] {a, b, (int) Math.sqrt(a*a + b*b)});
//mapToObj를 이용한 재구현
IntStream.rangeClosed(1,100)
          .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
          .mapToObj(b -> new int[] {a, b, (int) Math.sqrt(a*a + b*b)});
```

#### a값 생성
마지막으로 a를 생성할 수 있으며, 아래는 최종 코드이다.
```java
Stream<int[]> pythagoreanTriples
    = IntStream.rangeClosed(1,100).boxed()
                .flatMap(a ->
                    IntStream.rangeClosed(a, 100)
                              .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
                              .mapToObj(b ->
                                  new int[]{a, b, (int)Math.sqrt(a * a + b * b)})
                );
```
여기서 b의 범위가 1부터 100이라면 중복된 세 수가 생성될 수도 있으니, a,100으로 하였다.

#### 개선할 점
* 제곱근을 두 번 계산하는 로직을 개선한다.
* (aa, bb, aa + bb) 형식을 만족하는 세 수를 만든 다음 원하는 조건에 맞는 결과만 필터링하도록 최적화한다.

```java
Stream<double[]> pythagoreanTriples =
                IntStream.rangeClosed(1, 100).boxed()
                         .flatMap(a -> IntStream.rangeClosed(a, 100)
                                          .mapToObj(b ->
                                                  new double[]{a, b, Math.sqrt(a * a + b * b)}) // 만들어진 세 수
                                 .filter(t -> t[2] % 1 == 0)); // 세 수의 세 번째 요소는 반드시 정수여야 한다
```

## 5.8 스트림 만들기
일련의 값, 배열, 파일, 심지어 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로 스트림 만드는 법을 설명한다.

### 5.8.1 값으로 스트림 만들기
임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있다.
```java
// 스트림의 모든 문자열을 대문자로 변환 후 출력
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
stream.map(String::toUpperCase)
	    .forEach(System.out::println);
// empty 메서드를 이용해 스트림을 비울 수 있음
Stream<String> emptyStream = Stream.empty();
```

### 5.8.2 null이 될 수 있는 객체로 스트림 만들기
자바 9에서는 null이 될 수 있는 객체를 스트림으로 만들 수 있는 새로운 메서드가 추가되었다. null이 될 수 있는 객체를 스트림으로 만드려면 다음처럼 null을 명시적으로 확인해야 했다.
```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);
```
Stream.ofNullable을 이용해 다음처럼 사용할 수 있다.
```java
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```
null이 될 수 있는 객체를 포함하는 스트림값을 flatMap과 함께 사용하는 상황에서는 이 패턴을 더 유용하게 사용 가능하다.
```java
Stream<String> value = Stream.of("config", "home", "user")
                              .flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```
### 5.8.3 배열로 스트림 만들기
Array.stream을 이용해서 스트림을 만들 수 있다.
```java
int[] numbers = {2,3,5,7,11,13};
int sum = Arrays.stream(numbers).sum();
```

### 5.8.4 파일로 스트림 만들기
파일을 처리하는 등의 I/O 연산에 사용하는 자바의 NIO API(비블록 I/O)도 스트림 API를 활용할 수 있도록 업데이트 되었다.
```java
long uniqueWords = 0;
try (Stream<String> lines = Files.lines(Path.get("data.txt"), Charset.defaultCharset())) {
		uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                            .distinct()
                            .count();
}
catch (IOException e) {
}
```
Stream 인터페이스는 자원을 자동으로 해제할 수 있는 AutoCloseable 인터페이스를 구현하므로 finally에서 자원을 닫을 필요가 없다.

### 5.8.5 함수로 무한 스트림 만들기
스트림 API는 함수에서 스트림을 만들 수 있는 두 정적 메서드 Stream.iterate와 Stream.generate를 제공한다. 두 연산을 이용해서 **무한 스트림** 즉, 고정된 컬렉션에서 고정된 크기로 스트림을 만들었던 것과 달리 크기가 고정되지 않은 스트림을 만들 수 있다.

#### iterate 메서드
```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```
iterate 메서드는 초기값과 람다를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있다. iterate는 요청할 때 마다 값을 생산할 수 있으며 끝이 없으므로 **무한 스트림**을 만든다.이러한 스트림을 **언바운드 스트림**이라고 표현한다. 

일반적으로 연속된 일련의 값을 만들 때는 iterate를 사용한다. 자바 9의 iterate 메서드는 프레디케이트를 지원한다. 두 번째 인수로 프레디케이트를 받아 언제까지 작업을 수행할 것인지의 기준으로 사용한다.
```java
IntStream.iterate(0, n -> n < 100, n -> n + 4)
          .forEach(System.out::println);
```
filter로 조건을 걸어주는 것으로는 같은 결과를 얻을 수 없다.filter 메서드는 언제 이 작업을 중단해야 하는지를 알 수 없기 때문이다. 그러므로 takewhile을 이용하는것이 해답이다.
```java
IntStream.iterate(0, n -> n + 4)
          .takeWhile(n -> n < 100)
          .forEach(System.out::println);
```

#### generate 메서드
iterate와 비슷하게 generate도 요구할 때 값을 계산하는 무한 스트림을 만들 수 있다. 하지만 iterate와 다르게 생산된 각 값을 연속적으로 계산하지 않는다. generate는 Supplier<T>를 인수로 받아서 새로운 값을 생선한다.
```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```
위 코드에서 사용한 발행자(메소드 참조 Math.random)는 상태가 없는 메소드, 즉 나중에 계산에 사용할 어떤 값도 저장해두지 않는다. 병렬코드에서는 발행자에 상태가 있으면 안전하지 않기 때문에, 상태를 갖는 발행자는 피하는게 좋다.

IntStream을 이용하면 박싱 연산 문제를 피할 수 있다. IntStream의 generate 메서드는 Supplier<T> 대신에 IntSupplier을 인수로 받는다.
`IntStream ones = IntStream.generate(() -> 1);` 이 코드는 무한 스트림을 생성하는 코드이다.

IntSupplier 인터페이스에 정의된 getAsInt를 구현하는 객체를 명시적으로 전달할 수도 있다. 여기서 사용한 익명 클래스와 람다는 비슷한 연산을 수행하지만 익명 클래스에서는 getAsInt 메서드의 연산을 커스터마이즈할 수 있는 상태 필드를 정의할 수 있다는 점이 다르다. 바로 부작용이 생길 수 있음을 보여주는 예제다.

즉, 람다는 상태를 바꾸지 않는다.

아래는 다음 피보나치 요소를 반환하도록 IntSupplier를 구현한 코드이다.
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
위 코드에서는 IntSupplier 인스턴스를 만들었다. 만들어진 객체는 기존 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어있는지 추적하므로 **가변**상태 객체다.getAsInt를 호출하면 객체 상태가 바뀌며 새로운 값을 생산한다.

하지만, iterate를 사용했을 때는 값을 새로 생성하면서도 기존 상태를 바꾸지 않는 순수한 **불변**상태를 유지했다.

스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 **불변 상태 기법**을 고수해야한다.

## 5.8 마치며
* 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
* filter, distinct, takeWhile, dropWhile, skip, limit 메서드로 스트림을 필터링하거나 자를 수 있다.
* 소스가 정렬되어 있다는 사실을 알고 있을 때 takeWhile과 dropWhile 메서드를 효과적으로 사용할 수 있다.
* map, flatMap 메서드로 스트림의 요소를 추출하거나 변환할 수 있다.
* findFirst, findAny 메서드로 스트림의 요소를 검색할 수 있다. allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색할 수 있다.
* 이들 메서드는 쇼트서킷, 즉 결과를 찾는 즉시 반환하며, 전체 스트림을 처리하지는 않는다.
* reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다. 예를 들어 reduce로 스트림의 최댓값이나 모든 요소의 합계를 게산할 수 있다.
*  filter, map 등은 상태를 저장하지 않는 상태 없는 연산이다. reduce 같은 연산은 값을 계산하는  데 필요한 상태를 저장한다. sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야 한다. 이런 메서드를 상태 있는 연산 이라고 부른다.
* IntStream, DoubleStream, LongStream은 기본형 특화 스트림이다. 이들 연산은 각각 기본형에 맞게 특화되어 있다.
* 컬렉션뿐 만 아니라 값, 배열, 파일, iterator과 generate 같은 메서드로도 스트림을 만들 수 있다.
* 무한한 개수의 요소를 가진 스트림을 무한 스트림이라 한다.
# Chapter 6 - 스트림으로 데이터 수집

이번 챕터에서는 collect가 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행하는 것을 다룰 것입니다. 해당 챕터의 대주제는 다음과 같습니다.

- Collectors 클래스로 컬렉션 만들고 사용하기
- 하나의 값으로 데이터 스트림 리듀스
- 특별한 리듀싱 요약 연산
- 데이터 그룹화와 분할
- 자신만의 커스텀 컬렉터 개발

collect, 컬렉션, 컬렉터를 헷갈리지 않도록 주의해야 합니다!!

다음은 어떤 트랜잭션 리스트를 통화벼로 그룹화한 코드입니다.

```java
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();

for (Transaction ts : transactions) {
  Currency currency = ts.getCurrency();
  List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);

  if (transactionsForCurrency == null) {
    transactionsForCurrency = new ArrayList<>();
    transactionsForCurrencies.put(currency, transactionsForCurrency);
  }
  transactionsForCurrency.add(ts);
}
```

위 코드는 무엇을 실행하는지 한눈에 파악하기 어려운 코드입니다. Stream에 toList를 사용하는 대신 더 범용적인 컬렉터 파라미터를 collect 메소드에 전달함으로써 원하는 연산을 간결하게 할 수 있음을 배울 것입니다.

다음 코드는 collect 메소드를 이용한 코드입니다.

```java
Map<Currency, List<Transaction>> transactionsByCurrencies =
transactions.stream().collect(groupBy(Transaction::getCurrency));
```

<br>
<hr>
<br>

## ■ 1. 컬렉터란 무엇인가?

<br>

위 코드에서 보면 알 수 있듯이, 함수형 프로그래밍에서는 무엇을 원하는지 직접 명시할 수 있기 때문에, 어떤 방법으로 이를 얻을지는 신경 안 써도 됩니다.

collect 메소드로 Collector 인터페이스를 구현할 수 있고, Collector 인터페이스 구현은 스트림의 요소를 어떤식으로 도출할지 지정합니다.

다수준(multilevel)으로 그룹화를 수행할 때는 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 두드러집니다.

- 명령형 코드 : 문제해결 과정에서 다중루프와 조건문을 추가하며 가독성, 유지보수성 떨어짐
- 함수형 코드 : 필요한 컬렉터를 쉽게 추가 가능

<br>

### ▷ 고급 리듀싱 기능을 수행하는 컬렉터

<br>

함수형 API의 또 다른 장점은 높은 수준의 조합성과 재사용성을 들 수 있습니다. 스트림에서 collect를 호출하면 스트림의 요소에 리듀싱 연산이 수행됩니다.

이는 명령형 프로그래밍에서는 직접 구현해야했던 작업이지만 자동으로 수행됩니다.

<br>

이렇게 Collector 인터페이스의 메소드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정됩니다.

Colllectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 쉽게 생성할 수 있는 정적 팩토리 메소드를 제공합니다. ex ) toList

<br>

### ▷ 미리 정의된 컬렉터

<br>

Collectors 클래스에서 제공하는 메소드의 기능은 다음 세가지로 구분할 수 있습니다.

- 스트림 요소를 하나의 값으로 리듀스 & 요약
  - ex ) 트랜잭션 리스트에서 트랜잭션 총합을 찾는 등의 다양한 계산을 수행할 떄 활용
- 요소 그룹화
  - 다수준으로 그룹화 하거나 서브그룹에 추가로 리듀싱 연산을 적용할 수 있게 다양한 컬렉터를 조합
- 요소 분할

  - 프레디케이트를 그룹화 함수로 사용

  6.2부터 6.4까지 순차적으로 설명할 예정입니다.

<br>
<hr>
<br>

## ■ 2. 리듀싱과 요약

<br>

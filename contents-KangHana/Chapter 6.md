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

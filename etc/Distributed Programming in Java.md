# Distributed Programming in Java

## Intro to Map-Reduce

![Imgur](https://i.imgur.com/imkZiBC.png)

### map

- 하나의 K,V pair를 취해서 여러 개의 K, V pair 생산
- 병렬로 실행 가능

```
(KA, VA) -> map -> {(KA1, VA1), (KA2, VA2), (KA1, VA2), ...}
(KB, VB) -> map -> {(KB1, VB1), (KB2, VB2), (KB3, VB3), ...}
...
```

### group

- MapReduce 프레임워크에 의해 자동으로 수행
- map 의 결과인 중간 K, V pair들의 K 기준으로 그룹핑

```
{(KA1, {VA1, VA2}), (KA2, {VA2}), ..., (KB1, {VB1}), (KB2, {VB2}), (KB3, {KB3})}

```

### reduce

- grouping 결과인 여러 개의 K, V pair를 K를 기준으로 집계(sum, multiply, ...)

```
{(KA1, VA1 + VA2), (KA2, VA2), ..., (KB1, VB1), (KB2, VB2), (KB3, KB3)}
```

## Hadoop Framework

![Imgur](https://i.imgur.com/RMl6Uqk.png)

- 대규모 병렬 프로그래밍을 쉽게 작성할 수 있게 해주는 프레임워크
- 각 노드는 멀티코어와 스토리지 보유
- Input 데이터에 따라 MR을 위한 자원 배분, 스케줄링
- MR은 함수형 프로그래밍 모델을 따르므로 병렬 연산이 가능해서 분산 수행 가능
- 분산 수행 중 특정 노드가 죽더라도 해당 노드에 배분된 작업만 재 실행하면 복구 가능하므로 Fault Tolerant

## Spark Framework

![Imgur](https://i.imgur.com/Af8MhMK.png)

- 노드에는 멀티코어와 스토리지 외에 메모리도 있으니 느린 스토리지는 적게 쓰고 빠른 메모리를 많이 쓰자

Hadoop | Spark
---- | ----
KV pair | Resilient Distributed Dataset
MR | Transforms(Intermediary - map, filter, join, ...), Actions(Terminal - reduce, collect, ...) 

- Lazy Evaluation
- Caching(Memory): 이 덕분에 성능 대폭 향상. 하지만 디스크보다 비싼 메모리가 충분해야

## TF-IDF Example

![Imgur](https://i.imgur.com/W51sTFg.png)

- Term Frequency(용어 빈도), Inverse Document Frequency(역 문서 빈도)
- 문서간의 유사도 측정에 사용되는 개념
- N개의 문서 D1, D2, ..., DN이 있고 각 문서마다(문서별로 가 아니라) 나오는 용어의 집합을 TERM1, TERM2, ... 라고 하면
  - TFij는 문서 Dj에서 용어 집합의 원소인 용어 Ti이 발견되는 빈도를 의미
- DF1, DF2, ... 는 각 용어 Ti를 포함하는 문서의 갯수를 의미
- 역 문서 빈도 IDFi = N / DFi 는 어떤 용어가 자주 사용되고 어떤 용어가 자주 사용되지 않는지 결정 가능
- 덜 사용되는 용어에게 주는 가중치 Weight(TERMi, Dj) = TFij * log(N / DFi)
- 문서 Dj에서 사용되는 모든 용어 TERMi의 빈도를 추출하는 데 MapReduce 활용 가능
  - ((Dj, TERMi), TFij)
- 1차 MR의 결과를 DFi를 구하는 데 사용 가능

## Page Rank Example

![Imgur](https://i.imgur.com/PKRiboF.png)

- page B의 순위 `RANK(B) = Sum a-SRC(B) (RANK (A) / DEST_COUNT(A))`
  - a-SRC(B)는 B로의 링크를 가지고 있는 모든 페이지의 집합
  - DEST_COUNT(A)는 A에 포함된 모든 외부로의 링크의 수

- page rank 알고리듬을 만들고
- 슈도 코드로 작성한 후
- Spark 문장으로 자연스럽게 작성할 수 있다.
  - flatMapToPair(), reduceByKey(), mapValue()






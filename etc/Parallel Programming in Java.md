# Parallel Programming in Java

## Future

Future 태스크(꼭 자바의 Future를 지칭하는 것이 아님)는 반환값을 가진 태스크이며, Future 객체(Promise 객체라고도 부른다)는 태스크의 반환값에 접근할 수 있게 해주는 handle이다.

Future 객체는 기본적으로 다음 두 가지 연산이 가능하다.

1. 할당: future 객체의 참조를 변수에 할당할 수 있다. future 객체에 내용물은 한 번만 할당 가능하며, future 태스크가 완료된 후에는 수정할 수 없다.

1. 블로킹 읽기: Future 태스크의 반환값을 얻으려면 Future 객체의 get() 메서드를 호출해야 하는데, 반환값이 나올 때까지 blocking 한다. 따라서 `future.get()` 다음에 나오는 statement는 get()이 Future 태스크가 실행 완료되고 값을 반환할 때까지 실행되지 않는다.

Future는 Data Race 상황을 피하도록 신경써서 정의해야 하며, 이것이 Future가 functional parallelism에 적합한 이유다.



## Fork/Join Framework

### Fork

별도의 스레드를 생성해서 태스크를 병렬 실행

### Join

별도의 스레드에서 병렬 실행되던 모든 태스크 실행이 종료될 때까지 대기

### RecursiveAction, RecursiveTask

- 둘 모두 compute() 내에서 연산 실행
- RecursiveAction 는 반환값이 없고
- RecursiveTask 는 반환값이 있다.

## Computation Graph

Work: CG 상에 있는 모든 노드의 실행 시간의 합
Span: 최장 경로(critical path) 상에 있는 노드의 실행 시간의 합

Tp를 프로세서의 수가 p개 일 때의 실행 시간이라고 하면

Tinf <= Tp <= T1

동일한 병렬 작업에 대해서도 Tp는 스케줄링 방식에 따라 달라질 수 있다.

parallel speedup = T1 / Tp
parallel speedup <= p
parallel speedup <= Work / Span

## Amdahl's law

직렬로 실행되어야 햐는 프로그램의 비율을 q라고 하면, SpeedUp의 한계는 1 / q 다.

따라서 병렬 프로그램 중 직렬로 실행되는 프로그램ㅁ의 비율이 10%라고 하면, SpeedUp의 한계는 1 / 0.1 = 10 이며, 이는 프로세서의 수를 10을 초과해 늘릴 필요 없음을 의미한다.




## Determinism

### Fuctional Determinism

입력값이 같으면 결괏값은 항상 같다.

### Structural Determinism

입력값이 같으면 처리 경로가 항상 같다.

> Data Race가 없다고 해서 Deterministic 하다고 할 수는 없다.
> 
> Data Race가 있다고 해서 항상 Non-deterministic 하다고 할 수는 없다?
> 
> - 값을 찾으면 플래그를 true로 세팅하고 다시 false로 세팅하지 않는 프로그램이 있다면 true로 세팅할 때는 Data Race가 있을 수 있으나 입력값이 같으면 true/false 여부는 항상 같게 나온다.


## Parallel Loop

`n*n` 매트릭스 곱 연산의 예

### Sequential

```
for (i: n) {
  for (j: n) {
    c[i][j] = 0
    for (k: n) {
      c[i][j] += a[i][k] + b[k][j]
    }
  }
}
```

### Parallel

k 루프를 병렬화하면 `c[i][j]`에 Data Race 발생하므로 병렬화하면 안됨

    ```
    for (i: n) {
      for (j: n) {
        c[i][j] = 0
        for (k: n) {
          c[i][j] += a[i][k] + b[k][j]
        }
      }
    }
    ```
    
따라서 i나 j 루프만 병렬화 가능

## Barrier

```
parallelFor (i: n) {
  print 'Hello ' + i
  print 'Bye ' + i
}
```

아래와 같이 Hello 0은 항상 Bye 0 앞에 오지만 Hello 2가 Hello 0 보다 앞에 있을 수도 있다.

```
Hello 2
Hello 0
Bye 2
Hello 1
Bye 1
Bye 0
```

모든 Hello 가 끝난 후에 Bye 를 출력하게 하려면?

```
parallelFor (i: n) {
  print 'Hello ' + i
  
  barrier: 여기에 도달하면 i에 대한 반복이 끝나기 전까지 다음 행으로 진행하지 않는다. 일종의 waiting point
  
  print 'Bye ' + i
}
```

Barrier를 쓰면 결과는 아래처럼 나온다.

```
Hello 2
Hello 0
Hello 1
Bye 2
Bye 1
Bye 0
```

### Stencil Computation

Xi = Avg(Xi-1, Xi+1) with X0 = 0 and X1 = 1


### Java Phaser

Barrier는 자바에서는 `java.util.concurrent.Phaser`로 구현되어 있다.

```
Phaser ph = new Phaser(n);

forall (i : [0:n-1]) {
  print HELLO, i;

  int phase = ph.arrive();  // arrive가 뭘까?

  myId = lookup(i);

  ph.awaitAdvance(phase);  // awaitAdvance가 뭘까?
  
  print BYE, myId;
}

```

### Point-to-Point Sync

![Imgur](https://i.imgur.com/WqEArYb.png)

- Critical Path Length(Span)는 6이지만, 실제 의존 관계를 살펴보면 3이 소요되는 Task2와 3이 소요되는 D(X, Y)에는 의존 관계가 없다.
- 따라서 Phaser를 적절히 사용하면 실질적인 CPL은 6이 아니라 Task1-D 또는 Task2-E로 이어지는 5가 된다.

Task 0 | Task 1 | Task 2
----|----|----
1a: X = A(); //cost = 1 | 1b: Y = B(); //cost = 2 | 1c: Z = C(); //cost = 3
2a: ph0.arrive(); | 2b: ph1.arrive(); | 2c: ph2.arrive();
3a: ph1.awaitAdvance(0); | 3b: ph0.awaitAdvance(0); | 3c: ph1.awaitAdvance(0);
4a: D(X, Y); //cost = 3 | 4b: ph2.awaitAdvance(0); | 4c: F(Y, Z); //cost = 1
done | 5b: E(X, Y, Z); //cost = 2 | done

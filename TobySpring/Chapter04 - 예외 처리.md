# Chapter 04 - 예외 처리

본 내용은 토비의 스프링 3 책의 내용을 정리한 것입니다.

토비의 스프링은 스프링 뿐아니라 객체 지향의 기본 원리, 테스팅, 예외 처리, 디자인 패턴 등 Java 개발자라면 반드시 알아야 하는 내용을 스토리 전개 방식으로 점진적으로 친절하게 설명해주는 명저입니다. 

똑같은 내용으로 미국에서 영어로 출간되었다면 Jolt상을 받기에도 충분한 책이라고 생각합니다.

책의 절대적인 가격은 높아보이지만 웬만한 책 두어권 보는 것보다 이 한 권의 책을 여러번 보는 것이 비용 대비 효과가 훨씬 좋을 것입니다. 
이 책을 읽고 아낄 수 있는 많은 시간, 높아질 몸값을 생각하면 충분히 투자할 가치가 있는 책입니다.

http://www.acornpub.co.kr/book/toby-spring3-1-set

## Unchecked

- 시스템 오류나 에러에 가까운 예외
- RuntimeException을 상속한 예외
- 명시적으로 try/catch나 throws를 해줄 필요 없음
- 항상 복구할 수 있는 에러가 아니면 Unchecked로 하고 바로 통보되게 하는 것이 좋음 

## Checked

- 비즈니스 흐름 상에서 예상할 수 있는 예외, 또는 애플리케이션 로직으로 해결할 수 있는 예외
- RuntimeException을 상속하지 않은 예외
- 명시적으로 try/catch나 throws를 해줘야함


## 예외 처리 전략

### 예외 회피

- 발생한 예외를 외부로 그냥 던져서 외부에서 처리하도록 함
- 메서드에 throws 문 선언으로 처리
- 처리해야할 곳으로 제어를 이전할 수 있게 해주나, 
- 처리해야할 곳에서도 throws를 쓰는 부작용

### 예외 복구

- try/catch로 예외를 잡아서 처리 후 정상 흐름으로 복귀시킨다.

    ```java
    try {
        file.open()
    } catch (IOException e) {
        errorCount++;
    }
    ```     

### 예외 전환

- try/catch로 예외를 잡아서 적절한 처리와 함께 다른 예외로 전환해서 외부로 던진다.

    ```java
    try {
        file.open()
    } catch (IOException e) {
        logger.warn(e);
        throw new UncheckedException(e);
    }
    ```     
- Checked 예외를 잡아서 Unchecked 예외로 전환해서 던지면 예외 발생 메서드를 호출한 외부 메서드에서는 try/catch나 throws가 필요없어진다.

## DataAccessException

- SQLException은 JDBC에서 사용되는 Checked 예외로, 
- 대부분의 경우 복구가 불가능하므로 Unchecked 예외로 감싸서 던지는 것이 바람직하다.
- DBMS 벤더 별로 예외 코드도 다를 수 있다.
- JDO, JPA, Hibernate 등은 SQLException 대신 Unchecked 예외를 사용한다.
- 따라서 Checked 예외로서 소스에 명시해줘야 하는 SQLException에 직접 의존하면, DAO 클래스에서 JDO, JPA 등 SQLException을 사용하지 않는 다른 datasource로 변경할 때 SQLException을 제거해야하므로 불필요한 작업이 발생한다.
- 스프링에서는 DB에 독립적으로 적용할 수 있는 추상화된 Unchecked 예외인 DataAccessException 제공
- JdbcTemplate에서는 Unchecked 예외인 DataAccessException을 이용해서 호스트 코드에서 try/catch를 쓸 필요 없게 만들어준다. 

## JdbcTemplate의 3대 기능

> - 자원 반납 코드를 DAO 코드에서 제거
> - 예외 처리 코드를 DAO 코드에서 제거
> - 트랜잭션 관리 코드를 DAO 코드에서 제거(5장 참고) 



----
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>

<a href='https://www.facebook.com/hanmomhanda' target='_blank'>HomoEfficio</a>가 작성한 이 저작물은

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">크리에이티브 커먼즈 저작자표시-비영리-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.

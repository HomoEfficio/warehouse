# Chapter 05 - 서비스 추상화

본 내용은 토비의 스프링 3 책의 내용을 정리한 것입니다.

토비의 스프링은 스프링 뿐아니라 객체 지향의 기본 원리, 테스팅, 예외 처리, 디자인 패턴 등 Java 개발자라면 반드시 알아야 하는 내용을 스토리 전개 방식으로 점진적으로 친절하게 설명해주는 명저입니다. 

똑같은 내용으로 미국에서 영어로 출간되었다면 Jolt상을 받기에도 충분한 책이라고 생각합니다.

책의 절대적인 가격은 높아보이지만 웬만한 책 두어권 보는 것보다 이 한 권의 책을 여러번 보는 것이 비용 대비 효과가 훨씬 좋을 것입니다. 
이 책을 읽고 아낄 수 있는 많은 시간, 높아질 몸값을 생각하면 충분히 투자할 가치가 있는 책입니다.

http://www.acornpub.co.kr/book/toby-spring3-1-set


## 서비스

- 3장까지는 별다른 비즈니스 로직 없이 단순히 데이터의 CRUD만 처리
- 실제 비즈니스 로직을 담아보자

### 사용자 등급(Level) 서비스

- 특정 조건을 만족하면 등급을 변경해주는 서비스
- DAO의 update에서 처리할 수도 있으나 SRP에 위배
- 비즈니스 로직은 UserService로 분리

![](http://i.imgur.com/MGtxMWA.png)

- 등급 변경 판별 로직은 UserService에 두기보다 Level에 두는 것이 좋다.

> 객체지향적인 코드는 다른 오브젝트의 데이터를 가져와서 스스로 작업하는 대신,
> 데이터를 갖고 있는 다른 오브젝트에게 작업을 해달라고 요청한다.
> - **오브젝트에게 데이터를 요구하지 말고, 작업을 요청하라**

## 트랜잭션 처리

- 1장에서 Data Source 분리, 3장에서 자원 반납 분리 및 에외 처리
- 트랜잭션 처리가 남아있다.
- 트랜잭션으로 묶는다는 것은 다수의 SQL 실행에 대해 AllOrNothing으로 적용되게 하겠다는 것을 의미하므로,
- 다수의 SQL을 실행시키는 UserService에서 트랜잭션을 관리하는 것이 타당
- UserService에서 트랜잭션을 관리하려면, Connection 생성 위치를 기존의 JdbcTemplate내에서 UserService로 이동해야 한다.
- 이렇게 하면 UserService 메서드 간, 또 UserService > UserDao 와 같이 호출할 때 Connection을 계속 전달해줘야 하므로 불편

### 트랜잭션 동기화

- UserService에서 Connection을 생성해서 트랜잭션 동기화 저장소에 저장
- 이후 UserDao, JdbcTemplate에서는 UserService에서 넘겨받은 Connection을 사용하는 것이 아니라, 트랜잭션 동기화 저장소에 저장된 Connection 사용
- JdbcTemplate은 트랜잭션 동기화 저장소에 등록된 Connection이 있으면 해당 Connection 사용, 없으면 새로운 Connection을 만들어서 별도의 트랜잭션을 시작해서 DB 작업 진행
- JdbcTemplate 덕분에 UserDao에서 트랜잭션 관리 관련 코드가 제거됨

![](http://i.imgur.com/EJPrWii.png)


### 글로벌 트랜잭션 처리

- 하나 이상의 DB가 첨여하는 트랜잭션 생성 시 JTA를 사용해야 함
- JTA를 사용하는 Tx 처리를 하려면 UserService 코드가 또 바뀌어야 함

## 트랜잭션 추상화

- UserService에서 JDBC나 JTA, Hibernate 등 기술에 특정한 Tx 관리 코드에 의존하는 대신 PlatformTransactionManager 인터페이스에 의존하도록 변경
- DataSourceTransactionManager, HibernateTransactionManager 등을 PlatformTransactionManager에 할당해서 사용

## DAO 추상화

- Dao도 JDBC, JTA, Hibernate 등에 의존하지 않도록 인터페이스 추출
- UserDao 인터페이스에 UserDaoJdbc, UserDaoJta, UserDaoHibernate등을 필요에 따라 할당하여 사용

![](http://i.imgur.com/VbXMH6v.png)



----
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="크리에이티브 커먼즈 라이선스" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>

<a href='https://www.facebook.com/hanmomhanda' target='_blank'>HomoEfficio</a>가 작성한 이 저작물은

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">크리에이티브 커먼즈 저작자표시-비영리-동일조건변경허락 4.0 국제 라이선스</a>에 따라 이용할 수 있습니다.

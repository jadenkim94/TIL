# 테코톡#8 Transaction

---

해당 포스팅은 우아한Tech의 테크토크 [트랜잭션 매커니즘 - 에이든](https://www.youtube.com/watch?v=ImvYNlF_saE) , [트랜잭션 - 예지니어스](https://www.youtube.com/watch?v=e9PC0sroCzc) 영상을 참고하여 재구성 하였습니다.
참고자료 - [Naver D2 - DBMS는 어떻게 트랜잭션을 관리할까?](https://d2.naver.com/helloworld/407507)

---

## 트랜잭션이란? 
```text
하나의 논리적 작업 단위를 구성하는 일련의 연산들의 집합 
즉, 여러 쿼리를 논리적으로 하나의 작업으로 묶어주는 것

논리적 단위가 필요한 이유
1. 구매자 계좌에서 10,000 원 출금 (update 구매자 계좌 잔고 -10,000)
2. 판매자 계좌에 10,000 원 입금 (update 판매자 계좌 잔고 +10,000) 
만약에 1번 작업이 진행된 후 2번 작업 수행 전에 예상치 못한 오류로 DB가 종료된다면..? 
이러한 불상사를 막기 위해 트랜잭션이란 논리적 단위로 구성하여 ACID 를 지킬 수 있도록 한다.  
```

### 트랜잭션의 성질 
```text
ACID 성질
1. Atomicity(원자성) : 트랜잭션은 DB에 모두 반영되거나, 전혀 반영되지 않아야 한다. (= 완료되지 않은 트랜잭션의 중간 상태를 DB에 반영허지 않는다)
2. Consistency(일관성) : 트랜잭션 작업처리결과는 항상 일관성이 있어야한다. (트랜잭션이 성공적으로 완료되면 DB의 제약조건에 맞는 상태를 보장해야함)
3. Isolation(독립성) : 둘 이상의 트랜잭션이 동시에 실행되고 있을 떄, 어떤 트랜잭션도 다른 트랜잭션 연산에 끼어들 수 없다.
4. Durability(지속성) : 트랜잭션이 성공적으로 완료되면 결과는 영구히 반영 
```

### 트랜잭션 성질과 성능 
```text
독립성을 완벽하게 보장하려면 동일 데이터에 100개의 트랜잭션이 접근하는 경우,
100개의 트랜잭션을 순차적으로 처리해야함으로 동시성이 매우 떨어져 성능이 떨어지게 된다.
그래서 트랜잭션의 격리 수준(isolation level)을 여러 단계로 두어 동시에 DB에 접근할 떄 
어떻게 제어할지에 대한 설정을 제공한다. 

표준적으로 
READ-UNCOMMITTED
READ-COMMITTED
REPEATABLE-READ
SERIALIZABLE
(4가지 isolation level 이 있으며 위로 갈수록 격리 수준이 낮다) 
 
데이터 특성에 따라 isolation level 을 적절히 설정하는 것이 중요하다.
```

### 트랜잭션 격리수준 (Isolation level)
```text
READ-UNCOMMITTED : 커밋 전의 트랜잭션의 데이터 변경 내용을 다른 트랜잭션이 읽는 것을 허용 
-> Dirty Read ( 1번 트랜잭션이 커밋 전 내용을 2번 트랜잭션이 읽어왔는데, 1번이 롤백하면 2번 트랜잭션이 읽은 값은 잘못된 값 ), Non-Repeatable, PhantomRead

READ-COMMITTED : 커밋이 완료된 트랜잭션의 변경사항만 다른 트랜잭션에서 조회 가능 
-> Non-Repeatable ( 1번 트랜잭션이 값을 바꾸는 도중 2번 트랜잭션이 값을 가져오면, 변경되기 전 값을 읽어온다. 그리고 1번 트랜잭션이 커밋 후 2번 트랜잭션이 다시 값을 읽으면 변경된 값을 가져온다, 즉 같은 트랜잭션내에서 동일한 값을 조회했는데 다른 값이 나오는 상황), PhantomRead

REPEATABLE-READ : 트랜잭션 범위 내에서 조회한 내용이 항상 동일함을 보장 
-> READ-COMMITTED 에서 발생할 수 있는 Non-Repeatable 을 방지하기 위해 한 트랜잭션에서 조회한 값의 동일성을 보장함 
-> 하지만 역시나 phantomRead 는 발생할 수 있는데, 이부분은 좀 더 찾아볼것 

SERIALIZABLE : 한 트랜잭션에서 사용하는 데이터를 다른 트랜잭션에서 접근 불가 
```

### 트랜잭션 복구
```text
ReDo 로그 : 변경 후의 값을 로그 
UnDo 로그 : 변경 전의 값을 로그 

1. Rollback 발생시 UnDo 로그를 바탕으로 데이터 복원 
2. 예상치 못한 오류 발생시 ReDo 로그를 바탕으로 데이터를 일관성있게 만든 후, UnDo 로그를 통해 데이터 복원
```

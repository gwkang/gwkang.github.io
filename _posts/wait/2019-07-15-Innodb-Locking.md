---
layout: post
title: InnoDB Locking
tags: [MySQL, InnoDB, Lock]
---
## InnoDB Locking

InnoDB 에서 사용하는 락의 종류는 아래와 같다.

* Shared 와 Exclusive Lock
* Intention Lock
* Record Lock
* Gap Lock
* Next-Key Lock
* Insert Intention Lock
* AUTO-INC Lock
* Predicate Locks for Spatial Indexes

## Shared 와 Exclusive Locks

InnoDB 는 Shared Lock 과 Exclusive Lock 을 구현한다.

* Shared Lock : Row 를 읽는 것에 Lock 을 건다.
* Exclusive Lock : Row 를 업데이트 하거나 삭제는 것에 Lock 을 건다.

만약 트랜잭션 T1 이 어떤 Row r 에 Shared Lock 을 건다면, 그 Row 에 대한 다른 트랜잭션 T2 의 요청은 다음과 같이 처리된다.

* T2 의 Shared Lock 요청은 즉시 승인된다. 그 결과, T1 과 T2 모두 r 에 Shared Lock 을 건다.
* T2 의 Exclusive Lock 요청은 즉시 승인되지 않는다.

만약 트랜잭션 T1 이 로우 r 에 Exclusive Lock 을 건다면 다른 트랜잭션 T2 의 두 어떤 종류의 Lock 도 즉시 승인되지 않는다. T2 는 T1 이 Row r 에 대한 Lock 을 해제할 때까지 대기해야 한다.

## Intention Locks

InnoDB 는 Row Lock 과 Table Lock 의 공존을 허용하는 다중 입도 락킹(multiple granularity locking)을 지원한다. 예를 들어  LOCK TABLES ... WRITE 같은 구문은 특정 테이블에 Exclusive Lock 을 건다. InnoDB 는 다중 입도 레벨로 Locking 을 실용화하기 위해서 Intention Lock 을 사용한다. Intention Lock 은 트랜잭션이 테이블의 한 행에 대해 나중에 필요로하는 락의 유형(공유 또는 독점)을 나타내는 Table-level Lock이다.

* Intention Shared Lock(IS)은 트랜잭션이 테이블의 개별 Row 에 Shared Lock 을 설정하려는 것을 가리킨다.
* Intention Exclusive Lock(IX)은 트랜잭션이 테이블의 개별 Row 에 Exclusive Lock 을 설정하려는 것을 가리킨다.

예를 들어, SELECT ... FOR SHARE 는 Intention Shared Lock(IS)을 설정한다. 그리고 SELECT ... FOR UPDATE  는 Intention Exclusive Lock(IX)을 설정한다.

Intention Lock 프로토콜은 다음을 따른다.

* 트랜잭션이 테이블의 한 행에 대해 공유락을 획득할 수 있기 전에, 그 테이블에 IS 락이나 그보다 강한 것을 먼저 획득해야 한다.
* 트랜잭션이 테이블의 한 행에 대해 독점락을 획득할 수 있기 전에, 그 테이블에 IX 를 먼저 획득해야 한다.

Table-level Lock 유형별 호환성은 다음의 테이블로 요약할 수 있다.

|      | X    | IX   | S    | IS   |
| ---- | ---- | ---- | ---- | ---- |
| X    | 충돌 | 충돌 | 충돌 | 충돌 |
| IX   | 충돌 | 호환 | 충돌 | 호환 |
| S    | 충돌 | 충돌 | 호환 | 호환 |
| IS    | 충돌 | 호환 | 호환 | 호환 |

Lock 은 존재하는 Lock 과 호환된다면 트랜잭션 요청이 승인되지만 충돌하면 승인되지 않는다. 트랜잭션은 존재하는 Lock 이 해제될 때까지 대기한다. Lock 요청이 존재하는 Lock 과 충돌하고 Dead Lock 의 이유로 승인될 수 없다면 에러가 발생한다.

Intention Lock은 전체 테이블 요청(예를 들어 LOCK TABLES ... WRITE)을 제외하고 어떤 다른 것도 블록하지 않는다. Intention Lock 의 주 목적은 **누군가가 Row 를 Locking 하고 있음을 또는 그 테이블의 어떤 Row 에 Lock 을 걸 것이라는 것**을 보여주는 것이다.

Intention Lock 에 대한 트랜잭션 데이터는 **SHOW ENGINE INNODB STATUS** 명령과 InnoDB 모니터로 볼 수 있다.

```sql
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

## Record Locks

Record Lock 은 Index Record 에 거는 Lock 이다. 

```sql
SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;
```

위 쿼리는 t.c1 이 10 인 Row 를 추가, 업데이트, 삭제를 못하게 막는다.

Record Lock 은 항상 Index Record 에 락을 건다.  테이블에 인덱스가 없다면 숨겨진 [Clustered Index](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html) 를 만들고 여기에 Lock 을 건다.

Record Lock 에 대한 트랜잭션 데이터는 **SHOW ENGINE INNODB STATUS** 명령과 InnoDB 모니터로 볼 수 있다.

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` 
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
2: len 7; hex b60000019d0110; asc        ;;
```

## Gap Locks

Gap Lock 은 Index Record 들 사이의 Gap 에 거는 Lock 이다. 또는 첫 Index Record 앞이나 마지막 Index Record 뒤의 Gap 에 거는 Lock 이다.

```sql
SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;
```

위 쿼리는 t.c1 이 15 인 값을 추가할 수 없게 한다. 왜냐하면, 10 과 20 사이에 있는 모든 값들에 Lock 이 걸리기 때문이다.

Gap 은 하나의 Index 값이나 여러 Index 값, 심지어 빈 값도 범위에 포함된다.

Gap Lock 은 성능과 동시성 사이의 균형을 맞춘다,  그리고 일부 Transaction Isolation Level 에서 사용된다.

Gap Locking 은 Unique Row 을 찾기 위해 Unique Index 를 사용하는 Row 에 Lock 을 거는 쿼리문에는 필요 없다. (검색 조건에 Multi-column Unique Index 의 일부 Column 만 사용한 경우에는 Gap Lock 이 걸린다.)

child 테이블의 id 컬럼이 Unique Index 를 갖고 있다면, 아래 쿼리는 id 가 100 인 Row 에 Index Record Lock 만 걸게된다.

```sql
SELECT * FROM child WHERE id = 100;
```

만약, id 가 Index 를 갖고 있지 않거나 Nonunique Index 를 갖고 있다면, 이 쿼리는 앞선 Gap 에 락을 건다.

다른 트랜잰션들이 어떤 한 Gap 을 동시에 잡고 있을 수 있다. 예를 들어, B 트랜잭션이 Exclusive Gap Lock 을 잡고 있을 때, A 트랜잭션도 같은 Gap 에 대해서 Shared Gap Lock 을 잡을 수 있다. 이런 것이 허용되는 이유는, 만약 어떤 레코드가 Index 에서 제거되면, 그 레코드에 대해 다른 트랜잭션이 잡은 Gap Lock 은 반드시 병합돼야 하기 때문이다.

InnoDB 의 Gap Locks 은 "순전히 억제"한다. Gap Lock 의 목적은 단지 Gap 사이에 레코드를 삽입하는 것을 막는 것이다. Gap Lock 은 공존할 수 있다. 어떤 트랜잭션이 취한 Gap Lock 은 같은 Gap 에 대해 다른 트랜잭션이 Gap Lock 을 얻지 못하도록 막지는 않는다. 그것이 Shared Gap Lock 이든 Exclusive Gap Lock 이든 차이가 없다. 그것들은 서로 충돌하지 않고 같은 기능을 수행한다.

Gap Locking 은 명확히 비활성될 수 있다. Transaction Isolcation Level 을 READ_COMMITED 로 바꾸면 비활성된다.  이 환경에서 Gap Locking 은 검색할 때와 Index 를 스캔할 때 사용하지 않는다. 그리고 Foreign-key 제약 검사와 중복 Key 검사를 위해서만 사용된다.

READ_COMMITTED Isolation Level 을 사용하면 또 다른 효과가 있다. MYSQL 이 WHERE 조건을 평가한 이후에 일치하지 않는 Row 에 대한 Record Lock 은 풀린다. UPDATE 구문에  대해서, InnoDB 는 "semi-consistent" 읽기를 한다. 그렇게 해서 최신 커밋 된 버전을 MySQL로 반환하여 MySQL이 Row가 UPDATE의 WHERE 조건과 일치하는지 확인할 수 있도록 한다.

## Next-Key Locks

Next-key Lock 은 Index Record에 대한 Record Lock과 Index Record 앞의 Gap에 대한 Gap Lock의 조합이다.

테이블 Index를 스캔하거나 검색할 때 접하는 Index Record에 Shared 또는 Exclusive Lock을 설정하는 방식으로 InnoDB는 Row-level Locking을 수행한다. 그래서, Row-level Lock은 실제로 Index Record Lock이다. Index Record에 걸리는 Next-key Lock 역시 Index Record 앞의 Gap에 영향을 준다. Next-Key Lock은 Index Record Lock과 Index Record 앞의 갭에 대한 Gap Lock을 합한 것이다. 만약 한 세션이 Index에 있는 Record R에 Shared 또는 Exclusive Lock을 걸었다면, 다른 세션에서는 Index 순서로 R 앞에 있는 Gap 안에 새 Index Record를 삽입할 수 없게된다.

어떤 인덱스가 10, 11, 13, 20을 포함한다고 가정하자. 따르는 간격을 커버하는 인덱스에 대한 가능한 Next-Key Lock

## 참고

* [InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
* [MySQL InnoDB lock & deadlock 이해하기](https://www.letmecompile.com/mysql-innodb-lock-deadlock/)
* [잠금에 관한 고찰(1) - 잠금(Lock) 매커니즘에 대하여](https://kuaaan.tistory.com/97)
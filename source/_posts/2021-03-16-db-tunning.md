---
layout: post
title:  "oracle 튜닝하기"
date:   2021-03-16 00:0054 +0900
categories: java
---

튜닝의 시작 

XPLAN을 통해서 실행 계획 일기.

```SQL
SELECT  *
FROM    HR.EMPLOYEES
WHERE   EMPLOYEE_ID LIKE '2%';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(null, null, 'ALLSTATS LAST')); -> 이거가 XPLAN 활용해서 실행계획을 보여주는 쿼리
```

실행 계획을 수행하고 나면 Id, Operation, Name, Starts, E-Rows, A-Rows, A-Time, Buffers... 

이렇게 테이블이 나타나는데. 읽을 때 Operation의 개행이 가장 안쪽인 것 부터 읽는다.

인덱스를 사용하고 나서 인덱스의 자료형을 준수해주지 않으면 내부적으로 해당 인덱스 컬럼 값을  변환하기 떄문에 인덱스를 제대로 활용할 수 없다.
즉 FULL SCAN이 일어난다.

튜닝이 정상적으로 되었는지를 알려면 기본적으로 Buffers 수가 줄었는지를 봐야 한다.

FULL SCAN
INDEX RANGE SCAN
INDEX UNIQUE SCAN

NESTED LOOPS -> 프로그래밍 언어에서 for문이 중첩되어 있는 구조와 동일.

FULL SCAN -> INDEX SCAN 으로 변경하여 성능 튜닝.


실행 계획을 봐야 우리가 작성한 쿼리가 정말 효율적인지. 아니라면 비효율적인 구간이 어딘지를 자세히 살펴볼 수 있다.
이런것들을 모두 알려면 사실 공부해야할 게 많음으로 모두 습득하고 튜닝된 쿼리를 작성할 수 있도록 하자.

오라클 힌트
-> 힌트를 옵티마이저에게 주는 것.
힌트에 오류가 있어도 Syntax를 뱉지는 않는다.
실행 계획에 영향을 미친다. 쿼리를 이런 방식으로 풀어라 라고 힌트를 주는 것이다.
즉 힌트를 주기 전에 그 이전에 쿼리가 어떻게 동작하는지를 알아야 한느데 이를 알기 위해서는 실행 계획을 봐야한다.

JOIN 순서를 조절할 수도 있음. A->B를 조인하는게 더 성능이 좋을 경우 ORDERED 이런 힌트를 주어서 해결 가능.

기본적으로는 힌트를 주지 않는게 좋다.
즉 옵티마이저가 최적화된 쿼리를 동작할 것이다 라고 생각해야한다.

최종적으로 작성한 쿼리가 성능이 낮다면 그때 힌트를 줘야함. 튜닝 단계에서.


조인방식도 엄청 다양하다.

NESTED LOOP JOIN, HASH JOIN
..

### NETSTED LOOP JOIN
-> 중첩 for문과 같은 원리.

IDOL_GROUP 과 IDOL_MEMBER 조인을 한다해보자.

소녀시대 -> 태연
        -> 티파니
        -> 윤아
2ne1    -> 박봄
        -> 산다라박

이런 식으로 조인되며 IDOL_GROUP 테이블이 OUTER_TABLE 이 되고(for문에서 밖에있는 for문의 역할과 같다)
IDOL_MEMBER 테이블이 INNER_TABLE 이 된다.

이런 식으로 조인하는 방법이 NESTED LOOP JOIN 이라고 한다.

보통 이럴떄에는 INNER TABLE에 해당하는 영역을 INDEX로 걸어서 성능을 높인다.

건건이 테이블을 조인하기 때문에 대량의 테이블을 조인하는 방법으로는 적절하지 않다.

아이돌 그룹 - 아이돌 멤버 처럼 1:M 관계를 가질 떄 1에 해당하는 테이블이 OUTER TABLE로 가야 성능에 유리하다.


### SORT MERGE JOIN
NESTED LOOP JOIN 방법과 유사하나. 다른점이 있다면 조인 이전에 먼저 조인 테이블 기준으로 정렬을 하고 조인을 시작한다.

INNER TABLE 에 인덱스가 걸려있지 않아서 NESTED LOOP JOIN을 사용하기가 너무 비효율적이다 할 떄 사용할 수 있다.

### 오라클 메모리 (PGA, SGA)
오라클이 사용하는 메모리는 크게 두가지가 존재.

SGA(System Global Area) - 모든 사용자가 공유 가능하여 사용

PGA(Program Global Area) - 사용자마다 공유하지 않고 개별적으로 사용

데이터베이스를 사용하는 구조를 그려보면 아래와 같다.

유저 프로세스 -> 서버 프로세스 + PGA -> 데이터 베이스

유저 프로세스에서 쿼리를 날리면 서버 프로세스가 이를 받고 데이터 베이스에서 데이터들을 가져오는 작업을 한다.
이떄 요청 받은 내용 및 기타 정보들을 저장하기 위해 서버 프로세스는 자신만의 메모리 공간인 PGA를 사용한다.

PGA - 데이터베이스에 접속하는 모든 유저에게 할당되는 각각의 서버 프로세스가 독자적으로 사용하는 오라클 메모리 영역.


### HASH JOIN
배치에서 ㅈ쓰면 좋은 수행 원리다.

PGA 영역 중 sorted area 부분을 hash key로 두고 조인을 하는 방법?

데이터 크기가 메모리 크기보다 커지면 디스크를 써야하기 떄문에 오히려 성능이 떨어질 수 있다.

### 서브쿼리의 종류

1. select 절에 사용하는 scalar subquery

2. from 절에 사용하는 inline view

3. where 절에 사용하는 중첩 서브쿼리.


서브쿼리 자체가 성능에 무조건 안좋다는 것은 선입견

```SQL
/* 서브쿼리를 활용한 경우 */
SELECT * FROM HR.EMPLOYEES A
 WHERE A.DEPARTMENT_ID IN (SELECT B.DEPARTMENT_ID
                            FROM HR.DEPARTMENTS B
                            WHERE B.LOCATION_ID=1700);

/* FROM 절에서 조인 */
SELECT *
  FROM HR.EMPLOYEES A,
       HR.DEPARTMENTS B
 WHERE A.DEPARTMENT_ID = B.DEPARTMENT_ID
   AND B.LOCATION_ID = 1700;          

/* NO_UNNEST 힌트문 적용한 경우 (성능이 더 떨어짐 원래 서브쿼리 방식)*/
SELECT * FROM HR.EMPLOYEES A
 WHERE A.DEPARTMENT_ID IN (SELECT /*+NO_UNNEST*/
                                 B.DEPARTMENT_ID
                            FROM HR.DEPARTMENTS B
                            WHERE B.LOCATION_ID=1700);                
```

실행계획을 살펴보면 위 두 쿼리가 둘다 HASH JOIN을 사용해서 동일한 형태로 동작하고 있음을 알수 있다.

원래 서브쿼리는 NO_UNNEST 이 방식을 동작하는데 서브쿼리를 푼 두번째 쿼리가 더 효과적으로 데이터를 가져오기 때문에
옵티마이저가 이 방식으로 HASH JOIN을 해서 데이터를 가져온다.


### 서브쿼리를 반드시 쓰면 안되는 경우.

```SQL
SELECT A.EMPLOYEE_ID,
       A.FIRST_NAME,
       A.LAST_NAME,
       A.SALARY
  FROM HR.EMPLOYEES A
 WHERE A.SALARY = (SELECT MIN(SALARY) FROM HR.EMPLOYEES)
    OR A.SALARY = (SELECT MAX(SALARY) FROM HR.EMPLOYEES);
```

위의 케이스를 잘 보면 HR.EMPLOYEES 이 동일한 테이블을 다른 차이 없이 3번 접근하고 있다,
그러면 이 테이블을 3번 액세스 한다. 비효율적이겠찌..

어떻게 수정해야 할까.

```SQL
SELECT B.EMPLOYEE_ID,
       B.FIRST_NAME,
       B.LAST_NAME,
       B.SALARY
  FROM (                
            SELECT A.EMPLOYEE_ID,
                   A.FIRST_NAME,
                   A.LAST_NAME,
                   A.SALARY,
                   ROW_NUMBER() OVER(ORDER BY SALARY) MINSAL,
                   ROW_NUMBER() OVER(ORDER BY SALARY DESC) MAXSAL
              FROM HR.EMPLOYEES A
       ) B
 WHERE B.MINSAL = 1 OR B.MAXSAL = 1;
```

이렇게 하면 같은 내용의 쿼리지만. EMPLOYEE_ID 이 테이블을 한번만 액세스 하고 있다. 훨씬 더 효율적

실행계획으로 살펴봐라.

테이블을 여러번 액세스 한다는 것이 성능이 떨어지는 이유는 기본적으로 테이블이 디스크에 존재하기 때문이다. 성능 튜닝을 할 때 우선적으로 포커스를 두는 것은 디스크 I/O 발생을 줄이는 것인데.
이 부분이 속도가 가장 느리기 때문이다.

그렇기 떄문에 테이블을 여러번 접근하는 것은 디스크 I/O 발생빈도가 늘어난다는 의미 임으로 이를 개선할 필요성이 있따.

#### 동일한 테이블을 바라보는 방법으로 서브쿼리를 작성하는 것은 안된다.



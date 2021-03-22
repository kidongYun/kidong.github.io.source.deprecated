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


### 4.1 B-TREE INDEX 구조

B-TREE INDEX 구조는 ROOT BLOCK, BRANCH BLOCK, LEAF BLOCK 으로 우선 구분된다. 각 BLOCK 들은 아래와 같은 데이터를 가진다.

ROOT BLOCK = BRANCH BLOCK 주소, 인덱스 컬럼 값
BRANCH BLOCK = LEAF BLOCK 주소, 인덱스 컬럼 값
LEAF BLOCK = 인덱스 컬럼 값, TABLE의 ROW ID

여기서 인덱스 컬럼 값은 테이블에서 인덱스를 설정한 컬럼 값을 의미한다.

```SQL
SELECT COL1, COL2, COL3, COL4 FROM TAB WHERE COL1 BETWEEN :VAL1 AND :VAL2;
```

위와 같은 SQL에서 만약 COL1 값에만 인덱스가 걸려 있다면 SELECT 절에 있는 COL2, COL3, COL4 데이터들은 인덱스 테이블을 통해서 가져올 수 없다.
이런 경우에는 LEAF BLOCK 이 가지고 있는 TABLE ROW ID 값을 통해 접근해서 가져온다.


### 4.2 INDEX RANGE SCAN

INDEX RANGE SCAN은 말그대로 INDEX 테이블에서 특정 범위 내 BLOCK 순차적으로 탐색하는 것이다.

```SQL
SELECT COL1, COL2, COL3, COL4 FROM TAB WHERE COL1 BETWEEN :VAL1 AND :VAL2;
```

위와 같은 쿼리가 있고 COl1 값이 인덱스로 설정되어 있다면. 우선 COL1 컬럼 기준으로 :VAL1 값을 찾아간다.
이 값을 찾기 위해서 ROOT BLOCK -> BRANCH BLOCK -> LEAF BLOCK 순으로 탐색할 것이다.
:VAL1 에 해당하는 LEAF BLOCK을 찾았다면. :VAL2에 해당하는 값이 나올때 까지 LEAF BLOCK을 선형적으로, 순차적으로 탐색한다.

인덱스를 순차적으로 읽으면서 한 건씩 ROWID를 가지고 TABLE BLOCK을 한 개씩 ACCESS 하기 때문에 이를 RANDOM SINGLE BLOCK I/O 라고 부르며
TABLE ACCES BY INDEX ROWID 라는 실행계획으로 전시된다.

이 방법은 최악의 경우 조회 건수 만큼(즉 인덱스를 사용한 이유가 없이 FULL SCAN을 한다는 의미) I/O가 발생할 수 있다.
-> 아니네. FULL TABLE SCAN 방법은 멀티로 동작하기 때문에 오히려 최악의 케이스 일때는 이 방법이 더 빠르다고 언급함.

위에서 계속 언급한 BLOCK 이라는 것은 저장된 데이터의 묶음 단위로 보면 된다

ROOT BLOCK과 BRANCH BLOCK에 들어가는 주소값 들은 가상으 ㅣ값을 활용 한다.

만약 SELECT * 이 절에 인덱스에서 사용되는 컬럼만 조회하고 다른 컬럼들이 없다면 TABLE BLOCK을 ACCESS 할 필요가 없다.

XPLAN 실행 계획 테이블에서 Name 필드는 테이블 명이 들어간다. 인덱스 테이블이나, 일반 테이블 등.

```SQL
IX_ORDERS_N1 : ORDER_DATE, ORDER_MODE, EMPLOYEE_ID

SELECT /*+ INDEX(A IX_ORDERS_N1) */
       ORDER_DATE, CUSTOMER_ID,
       ORDER_MODE, ORDER_TOTAL
  FROM ORDERS A
 WHERE ORDER_DATE BETWEEN TO_DATE('20210101', 'YYYYMMDD')
                      AND TO_DATE('20210102', 'YYYYMMDD');


// 실행 계획                      
SELECT STATEMENT
  TABLE ACCESS BY INDEX ROWID
    INDEX RANGE SCAN
```

위와 같이 실행계획이 나타난다면 TABLE ACCESS BY INDEX ROWID 는 위에서 설명했듯이 INDEX BLOCK -> TABLE BLOCK 으로 ACCESS하는 RANDOM SINGLE BLOCK I/O를 나타내는 OPERATION이다.

```
/*+ INDEX(A IX_ORDERS_N1) */
```
와 같인 힌트를 주면 INDEX RANGE SCAㅜ, INDEX FULL SCAN 으로 수행한다.

### 4.3 INDEX RANGE SCAN DESCENDING

INDEX RANGE SCAN DESCENING 실행계획의 특징은 INDEX RANGE SCAN과 동일하다. 다른 점이 있다면 INDEX를 역순으로 읽는 다는 점이다.

```SQL
IX_ORDERS_N1 : ORDER_DATE

SELECT /*+ INDEX_DESC(A IX_ORDERS_N1) */
       ORDER_DATE, CUSTOMER_ID,
       ORDER_MODE, ORDER_TOTAL
  FROM ORDERS A
 WHERE ORDER_DATE BETWEEN TO_DATE('20120101', 'YYYYMMDD')
                      AND TO_DATE('20210102', 'YYYYMMDD')
 ORDER BY ORDER_DATE DESC;                    


// 실행 계획
SELECT STATEMENT
  TABLE ACCESS BY INDEX ROWID
    INDEX RANGE SCAN DESCENDING
```

위 쿼리를 실행하게 되면 아래와 같은 실행 계획이 발생할 것이다. 여기서 INDEX RANGE SCAN DESCENDING 이 나타나는 걸 볼 수 있는데. INDEX_DESC() 힌트를 줘서 그런것도 있지만.
기본적으로 ORDER BY DESC 정렬을 이처럼 역순으로 하게 되면 이처럼 동작한다. 즉 힌트문을 제거해도 동일하게 동작할 것이다.

OPTIMIZER는 인덱스를 역순으로 읽으면서 ORDER BY에 의한 SORT(ORDER BY) OPERATION 작업을 생략할 수 있다. -> 더 성능이 좋아진다.

```SQL
IX_ORDERS_N1 : ORDER_DATE

SELECT MAX(ORDER_DATE)
  FROM ORDERS
 WHERE ORDER_MODE = 'direct';


// 실행 계획
SELECT STATEMENT
  SORT AGGREGATE
    TABLE ACCESS FULL
```

ORDER_DATE 기준으로 인덱스가 설정되어 있기 때문에 만약 ORDER_MODE 조건이 없다면. INDEX FULL SCAN(MIN/MAX) OPERATION 이 발생해서 Bufffers 수치가 낮을 것이다. 하지만 저 조건으로 인해 FULL TABLE SCAN 이 발생한다.

이를 튜닝할 수 있는 방법 중 하나는 INDEX DESCENDING을 이용한 TOP N 쿼리를 사용하는 방법이다.

```SQL
SELECT ORDER_DATE
  FROM (
        SELECT /*+ INDEX_DESC(A IX_ORDERS_N1) */
               ORDER_DATE
          FROM ORDERS A
         WHERE ORDER_DATE > TO_DATE('10000101', 'YYYYMMDD')
           AND ORDER_MODE = 'direct'
         ORDER BY ORDER_DATE DESC   
  )
 WHERE ROWNUM <= 1;


// 실행 계획
SELECT STATEMENT
  COUNT STOPKEY
    VIEW
      TABLE ACCESS BY INDEX ROWID
        INDEX RANGE SCAN DESCENDING
```

인덱스는 컬럼 기준으로 정렬이 되어있음을 이용하는 테크닉 같다. 서브 쿼리 부분을 보면 ORDER_DATE 컬럼을 조회하고, 이 값을 DESC 형태로 정렬하게끔 보여지도록 하는데.
이렇게 하면 INDEX RANGE SCAN DESCENDING OPERATION 이 수행되고 인덱스 테이블의 가장 큰쪽에 존재하는 값이 가장 큰 값임을 바로 알 수 있다 이 곳에서 순차적으로 탐색하면서 ORDER_MODE 조건에 맞는 데이터가 나타나면 바로 반환한다. ROWNUM <= 1 조건이 하나의 값만 반환하라고 명시하고 있기 때문이다.

실행 계획을 보면 COUNT STOPKEY OPERATION이 있는데 이부분이 ROWNUM <= 1 조건으로 인해 발생한 동작이다.

만약 ORDER_MODE 와 같이 조건에 해당하는 부분이 PK 처럼 구분되는 값들이 많다면. 큰쪽부터 검색한다 해도. 조건에 맞는 데이터를 찾는 데에 오래 걸리기 때문에 성능이 매우 저하된다.
조건이 선택지가 적고 자주 발견되는 데이터인지를 우선 파악해야 한다.

### 4.4 INDEX RANGE SCAN (MIN/MAX)

```SQL
IX_ORDERS_N2 : EMPLOYEE_ID, ORDER_DATE

SELECT /*+ INDEX(A IX_ORDERS_N2) */
       MAX(ORDER_DATE)
  FROM ORDERS A
 WHERE EMPLOYEE_ID = 'E402';


 // 실행 계획
SELECT STATEMENT
  SORT AGGREGATE
    FIRST ROW
      INDEX RANGE SCAN (MIN/MAX)
```

INDEX 컬럼이 EMPLOYEE_ID, ORDER_DATE 와 같이 구성되어 있고 INDEX 선두 컬럼이 조건 절에 들어오며 두 번째 컬럼인 ORDER_DATE가 MAX 처리 되었을 때 나타나는 실행계획이
INDEX RANGE SCAN (MIN/MAX) 이다. 조건에 해당하는 EMPLOYEE_ID = 'E402' 에 대한 많은 ORDER_DEATE 중에서 가장 끝에 있는 인덱스 만 읽고 빠져나온다.
인덱스는 정렬되어 있기 때문에 가능하다.

BUFFER가 3회로 나타날 텐데 그 이유는 ROOT BLOCK, BRANCH BLOCK, LEAF BLOCK을 각각 한번씩 총 3번 읽었기 때문이다.

### 4.5 INDEX 컬럼 가공

인덱스로 사용한 컬럼을 TRIM(COLUMN_NAME), TO_DATE(COLUMN_NAME, 'YYYYMMDD') 와 같이 가공하게 되면. OPTIMIZER가 인덱스를 인지하지 못하고 FULL TABLE SCAN을 수행한다.
주의해야하는 부분이다.

```SQL
IX_ORDERS_N3 : CUSTOMER_ID, ORDER_DATE

SELECT ORDER_DATE, CUSTOMER_ID,
       ORDER_MODE, ORDER_TOTAL
  FROM ORDERS A
 WHERE RTRIM(CUSTOMER_ID) = 'C01492'
   AND ORDER_DATE BETWEEN TO_DATE('20120101', 'YYYYMMDD')
                      AND TO_DATE('20130101', 'YYYYMMDD');

// 실행 계획
SELECT STATEMENT
  TABLE ACCESS FULL
```

CUSTOMER_ID 스키마가 CHAR(7) 이라고 한다면 위처럼 6자리인 데이터를 조회할 수 없다. 이는 근본적으로는 스키마의 설계가 잘못된 것이지만. 만약 레거시로 존재한다고 했을 때
쿼리의 성능을 높이기 위해서는 어떤 방법이 있을까. 

RTRIM을 써서 CUSTOMER_ID 인덱스 컬럼을 직접 수정하기 보다는 RPAD 를 사용해서 들어오는 데이터를 변환시키면 인덱스를 사용하 ㄹ수 있다.

```SQL
SELECT ORDER_DATE, CUSTOMER_ID
       ORDER_MODE, ORDER_TOTAL
  FROM ORDERS A
 WHERE CUSTOMER_ID = RPAD('C01492', 7, ' ')
   AND ORDER_DATE BETWEEN TO_DATE('20120101', 'YYYYMMDD')
                      AND TO_DATE('20130101', 'YYYYMMDD');


// 실행 계획
SELECT STATEMENT
  TABLE ACCESS BY INDEX ROWID
    INDEX RANGE SCAN
```

기존 실행 게획에서는 TABLE ACCESS FULL 동작을 하다가 위처럼 변경하게 되면 INDEX RANGE SCAN 동작으로 변경됨을 알 수 있다.

### 4.6 CLUSTERING FACTORER

CLUSTERING FACTOR 라는 것은 무엇일까? 이것은 데이터가 모여 있는 정도 라고 볼 수 있다.

인덱스 테이블에서 LEAF BLOCK이 가지고 있는 ROWID를 가지고 테이블 데이터를 하나씩 찾아갈 때 계속 동일한 BLOCK를 액세스 한다면 이는 CF(CLUSTERING FACTOR)가 잘되어 있다고 볼 수 있고 반대의 경우로 서로 다른 BLOCk을 찾아갈 확률이 높다면 CF는 나쁜 것이다.

CF가 좋으면 BLOCK I/O 가 감소하고, 반대의 경우 증가한다.

위 4.2 에서 살펴본 INDEX RANGE SCAN은 DATA BLOCK의 위착 RANDOM하게 왔다 갔다하기 때문에 CF가 좋지 않은 예시로 볼 수 있다.

PS. BLOCK 이라는 것은 데이터 묶음 단위로 생각해야 한다. 하나의 데이터를 가지고 있는 것이 아니다.

CF가 가장 좋은 경우는 INDEX에 저장된 데이터의 순서가 완전히 일치하는 경우이며. 반대로 좋지 않은 경우는 INDEX의 데이터 블록 순서와는 상관 없이 흩어져 있는 경우가 된다.

중요한 점은. CF를 활용할 때 이 데이터 블록을 한번 접근했다고 해서 그 블록을 영구히 캐싱하는 것은 아니다. 바로 다음 로우를 조회하는 시점에만 이 블록을 캐싱으로 활용할 수 있기 때문에 만약 Data Block들이 밀집되어 있다고 해도 이를 연속적으로 접근하지 않는다면 CF가 좋지 않은 경우가 된다.


인덱스 테이블 순으로 정렬된 테이블을 만들고 이를 활용해 인덱스 테이블 순서와 동일한 새로운 테이블을 생성해보자. 즉 CF가 가장 좋은 경우를 얻어보는 거다.

```SQL
-- 인덱스 테이블과 일치하는 테이블 ORDERS_CF 생성
CREATE TABLE ORDERS_CF
AS
SELECT *
  FROM ORDERS
 ORDER BY ORDER_DATE;

-- ORDER_DATE 기준으로 인덱스 생성
CREATE INDEX IX_ORDERS_CF_N1
ON ORDERS_CF(ORDER_DATE);
```

위와 같이하게 되면 인덱스 테이블 블록과 데이터 테이블 블록이 일치하기 때문에 CF는 최상이다.

CF를 조회하는 방법은 아래와 같다.

```SQL
SELECT A.TABLE_NAME, A.INDEX_NAME, A.CLUSTERING_FACTOR, A.NUM_ROWS, B.BLOCKS TABLE_BLOCKS
  FROM DBA_INDEXES A, DBA_TABLES B
 WHERE A.TABLE_NAME = B.TABLE_NAME
   AND A.OWNER = B.OWNER
   AND A.INDEX_NAME IN ('IX_ORDERS_N1', 'IX_ORDERS_CF_N1');
```

좋은 CF의 수치는 TABLE BLOCK 수에 근접하고 반대의 경우 테이블 전체 건수에 근접하게 된다.

의문) PK 같은 값을 인덱싱하면 효과가 없다고 배웠는데 잘 생각해보면.
데이터 테이블 탐색 방법은 선형 탐색 방법이고, 인덱스는 B-TREE 탐색 방법이기 때문에 
PK 같은걸 인덱싱해도 성능이 좋지 않을까?? -> 맞는 것 같다 확인해보니까 기본적으로 PK는 인덱스가 적용되어 있는거 같다.

일반적으로 인덱스 컬럼이 추가도리 수록 CF는 나빠진다. 추가도니 컬럼으로 정렬이 되어 INDEX의 저장 순서와 테이블의 저장 순서가 달라지게 되기 때문이다.
그러나 만약 선행적으로 적용된 인덱스 컬럼이 날짜 정보처럼 (시, 분, 초 까지 가지는) 거의 유니크한 값이라면. 다른 인덱스 컬럼과 같은 값을 가지는 경우가 거의 없어서
정렬 순서가 크게 달라지지 안흔다.

결론적으로 CF가 매우 좋은 INDEX의 컬럼이 UNIQUE에 가까울 경우에는 그 뒤에 컬럼이 추가 되더라도 CF가 나빠지지 않는다.


#### FUNCTION BASED INDEX
컬럼에 TO_CHAR() 함수와 같은 것을 적용시켜서 그 데이터를 인덱스 컬럼으로 하는 인덱스 테이블을 말한다.

```SQL
CREATE INDEX IX_ORDERS_CF_N3
ON ORDERS_CF(TO_CHAR(ORDER_DATE, 'YYYYMMDD'));
```


### 4.7 INDEX RANGE SCAN VS FULL TABLE SCAN

FULL TABLE SCAN은 TABLE BLOCK 전체를 DB_FILE_MULTIBLOCK_READ_COUNT 파라미터 값에 비례해서 수십 BLOCK씩 MULTI BLOCK으로 읽는다.

이렇게 I/O를 하는 방법이 RANDOM SIGNLE BLOCK I/O, MULTI BLOCK I/O 로 상이한데. 그렇기 때문에 테이블 전체 건수의 1/20 이상이라면
FULl  TABLE SCAN을 사용하는 것이 유리하다.

즉 테이블의 많은 부분을 차지하는 row를 가져올 때에는 오히려 INDEX RANGE SCAN 방법이 역효과가 난다는 의미이다. SINGLE I/O 이기 때문에 그렇다.

이거는 아무래도 실행 계획을 보면서 Buffers, A-Times 등을 보면서 판단을 해야하지 않을까 싶다.

아무튼 결론은. 테이블 전체 만큼의 데이터를 가져 오려고 하면 INDEX를 사용하지 말고 그냥 FULL SCAN을 하라.

### 4.8 INDEX ACCESS 조건, FILTER 조건

INDEX ACCESS 조건이라는 것은 INDEX SCAN 시에 SCAN 범위를 줄여주는데에 참여한 조건을 의미한다.

INDEX FILTER 조건이라는 것은 실제 SCAN 범위는 줄여주지 못하고 말 그대로 데이터의 FILTER 역할 만을 한다는 의미이다.

인덱스 탐색량을 줄이기 위해서는 INDEX ACCESS 조건이 더 많이 발생하도록 하는게 좋다. SCAN 범위가 줄기 때문이다.

범위 조건 연산(>=, <=, BETWEEN, LIKE 등) 이 수행되기 이전에는 ACCESS 조건이 나타나지만 그 이후에는 FILTER 조건으로 사용되는게 인덱스의 특징이다.


COL1, COL2, COL3, COL4 컬럼으로 구성된 INDEX가 있다고 해보자.

```SQL
WHERE COL1 = '1' AND COL2 = 'A' AND COL3 = '나' AND COL4 = 'a'
```

이렇게 모든 조건 연산들이 범위 조건 연산이 아니라면 모두 INDEX ACCESS 조건으로 사용되어 범위를 줄여줄 수 있다.

```SQL
WHERE COL1='1' AND COL2 = 'A' AND COL3 BETWEEN '가' AND '다' AND COL4 = 'a'
```
COL1, COL2 는 범위 조건 연산인 COL3 이전에 발생했기 때문에 INDEX ACCESS 조건으로 사용되지만.
COL4 는 이 후이기 때문에 FILTER 조건으로 사용된다.

```SQL
WHERE COL1='1' AND COL2 = 'A' AND COL4 = 'a'
```
COL3 컬럼에 대한 조건이 생략되어 있는데 생략된 것도 범위 조건 연산으로 보며. 그렇기 떄문에 COL1, COL2는 INDEX ACCESS 조건, COL4 는 INDEX FILTER 조건이 된다.

예시를 보자.

```SQL
IX_ORDERS_N1 : ORDER_DATE, ORDER_MODE, EMPLOYEE_ID
IX_ORDERS_N2 : EMPLOYEE_ID, ORDER_MOD, ORDER_DATE

SELECT TO_CHAR(ORDER_DATE, 'YYYYMM') ORDER_MM
       , COUNT(*) ORDER_CNT
       , SUM(ORDER_TOTAL) ORDER_AMT
  FROM ORDERS A
 WHERE ORDER_DATE BETWEEN TO_DATE('20110101', 'YYYYMMDD')
                      AND TO_DATE('20120101', 'YYYYMMDD')
   AND EMPLOYEE_ID = 'E123'
   AND ORDER_MODE = 'direct'
 GROUP BY TO_CHAR(ORDER_DATE, 'YYYYMM');
```

IX_ORDERS_N2 의 조건이 범위 연산이 후위에 있기 때문에 INDEX_ACCESS 조건이 더 많이 생성되어서 더 좁은 범위를 스캔해도 되어 성능이 좋아진다.

### 4.9 INDEX SKIP SCAN

INDEX SKIP SCAN은 조회 조건이 INDEX의 ACCESS 조건이 아니라 FILTERING 조건으로 들어오는 경우 ACCESS 범위에 해당하는 모든 INDEX BLOCK을 순차적으로 SCAN하는 것이 아니라 (이 방법이 기존의 INDEX 방법) 불필요한 LEAF BLOCK들은 SKIP 해서 인덱스 범위를 줄여주는 동작이다.

이건 좋지는 않다고 했음. 왜냐하면 결국 선두 인덱스가 생략된 상태로 조회되었을때가 가정인데. 이 가정 자체가 사실 좋지 않은 가정임.

### 4.10 INDEX FULL SCAN

INDEX FULL SCAN은 INDEX LEAF BLOCK 전체를 순차적으로 ACCESS 하는 동작을 말한다. 모든 LEAF BLOCK을 탐색하기 때문에 성능은 당연히 떨어지고, INDEX 건수가 많을 수록 더 느리다.

### 실장님의 가르침

인덱스 스킵스캔을 타는 것은 결국 인덱스 설정이 적절하지 않는다는 의미이기 때문에 다시한번 재고 해봐야 한다.

nested loop join 방법은 outer table, inner table 둘다 디스크에 존재하며 outer table이 루프를 돌면서 inner table을 조인한다.

hash join 방법은 inner table 을 메모리에 올려두고 조인하기 때문에 보다 더 빠르다.
inner table을 메모리에 올린다의 전제조건은 inner table을 full로 사용할 경우이다.

ctrl + e -> 실행계획이 보인다.

```SQL
/*+ use_hash(aa, bb) */
```
이 방법을 쓰면 HASH JOIN을 한다.

쿼리를 짤때에는 기본적으로 인덱스를 확인해야 한다. full 스캔이 나오면 안된다.

인덱스 컬럼을 가공 시 해당 컬럼을 인덱스로 사용이 안되는데 조인되는 테이블이 아니고 피 조인 테이블에 적용하면 가능하다.

실행 계획을 볼때에는 상수가 아니라 변수로 두고 해야지 더 정확하다.
상수가 있다면 기본적으로 오라클은 해당 데이터를 먼저 기준으로 두고 스캔한다.

변수 설정하는 방법 :aaa

스칼라 서브쿼리 vs 조인
만약 스칼라 서브쿼리 되는 곳에 작성한 쿼리가 충분히 캐쉬를 활용할 수 있는 쿼리라면 사용해도 좋지만 그렇지 않다면 그런 쿼리는 좋지 않다.
대신 조인을 써라.

CASE 키워드 보다는 DECODE 키워드가 좀더 빠르다. 구문을 파싱하는 작업이 덜하다.

80% 이상을 가져온다고 하면 인덱스 조회보다 FULL 조회가 더 빠르다.

캐싱된다고 하지만 결국 이거는 메모리에 올라가는 거기 때문에 TTL이 존재하지는 않는다 다만. 다른 쿼리에 의해 올라간 데이터에 의해 밀린다.

hello

# Table Full Scan VS Index Range Scan

<br/>

> **Table Full Scan**

- 테이블에 속한 블록 전체를 읽어서 사용자가 원하는 데이터를 찾는다.
- Table Full scan은 지양해야 하지만, 오히려 인덱스가 성능을 떨어뜨리는 경우가 있다.
- 캐시에서 찾지 못하면 한 번의 I/O Call로 인접한 수십-수백 개의 블록을 한꺼번에 불러오는 것이 좋다.


<br/>

> **Index Range Scan**

- 인덱스 선두 컬럼이 가공되지 ㅇ낳은 상태로 조건절에 있어야 Index Range Scan이 가능하다.
- 인덱스에서 일정량을 스캔하면서 얻은 ROWID로 테이블 레코드를 찾는다.
- 

<br/><br/>

> **참고**

[[SQL 튜닝] Table Full Scan VS Index Range Scan VS Index Full Scan](https://imnkj.tistory.com/49)

<br/>

_2022.12.27_

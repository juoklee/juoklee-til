# Software Architecture

<br/>

> **Architecture**

소프트웨어 시스템의 아키텍처는 비즈니스 요구 사항을 만족하는 시스템을 구축하기 위한 구조이다.

<br/><br/>

> **좋은 아키텍처**

좋은 아키텍처는 유지보수나 개발, 배포를 쉽게 할 수 있도록 한다.
시스템의 핵심요소를 식별하고, 세부사항은 이 정책에 무관하게 만들 수 있는 형태로 구축한다.
필요한 시스템을 만들고 유지보수하는 데에 투입되는 인력을 최소화하는 것이다.


<br/><br/>


> **아키텍처 패턴**

소프트웨어 구조에서 발생하는 문제점을 해결하기 윈한 일반화된 재사용 가능한 솔루션이다.

1. Layer Pattern
   - 수평적 계층화 (UI, Service, Domain, Persistence)
2. Client-Server Pattern
  - 하나의 서버와 다수의 클라이언트로 구성
3. Master-slave Pattern
  - 마스터 컴포넌트는 동등한 구조를 지닌 슬레이브 컴포넌트들로 작업을 분산
4. Pipe-filter Pattern
  - 버퍼링 또는 동기화 목적으로 사용되는 파이프를 통해 데이터를 이동시키고, 처리과정은 필터 컴포넌트에서 이루어진다.
5. Broker Pattern
 - 분리된 컴포넌트로 분산 시스템에서 사용된다.
6. Peer-to-peer Pattern
  - 컴포넌트를 peers라고 부르며, 이 피어들은 역할이 유동적으로 바뀔 수 있다.
7. Event-bus Pattern
  - 이벤트 소스, 이벤트 리스너, 채널, 이벤트 버스 4가지 컴포넌트
  - 알림 서비스 
8. Model-View-Controller Pattern
  - 대화형 애플리케이션을 model, view, controller 3부분으로  나눈다.
  - 효율적인 재사용
9. Blackboard Pattern
  - 해결 전략이 알려지지 않은 문제에 유용
  - black board, knowledge source, control component
10. Interpreter Pattern
  - 프로그램의 각 라인을 수행하는 방법을 지정한다.




> **참고**

[[Clean Architecture] 아키텍처란?](https://share-factory.tistory.com/28))
[아키텍처 패턴과 종류](https://the-boxer.tistory.com/26)
[[번역] 10가지 소프트웨어 아키텍처 패턴 요약](https://mingrammer.com/translation-10-common-software-architectural-patterns-in-a-nutshell/#%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90-%ED%8C%A8%ED%84%B4-%EB%B9%84%EA%B5%90)

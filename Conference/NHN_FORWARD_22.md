# [NHN FORWARD 22](https://forward.nhn.com/2022)
<br/>

## **편안한 휴식 시간을 지켜줄 안정적인 백엔드 운영과 개발 기법**

1. 개발담당자가 서비스를 운영한다.
    1. 장점: 빠른 장애대응 / 최적의 서버 구성 / 설계,개발 단계에서 운영 고려
    2. 단점: 불안: 언제 문제가 발생할 지 모른다. / 피로: 운영이슈 해결 + 개발 / 비 전문성 : 개발과 운영 둘 다 전문성이 필요하다.
2. 사용자가 많아지면서 장애가 발생하기 시작
    1. 자동 재시작(self-healing)
        - 서버의 재시작이 필요한 경우?
            - garbage collection death spiral(죽음의 소용돌이)
                - 원인: 제한이 없는 요청 응답 레코드 수, static class 변수 또는 싱글턴 패턴의 class 변수에 과도한 데이터 적재
            - 스레드 차단이 필요한 경우
            - 서버에서 데드락이 발생한 경우
        - 해결 작업 무한루프: 장애발생 → 전파 → CPU, Memory, disk 등 리소스 확인, network 확인 → 로그 확인(OutOfMemoryError체크, 주요서비스 연동 체크) → 힙덤프 스레드 덤프 남기기 → 서버 재시작
            - JVM 프로세스는 OutOfMemoryError에도 죽지 않습니다.
            - 자동 재시작의 구성 - _is_process_alive.sh_
              ```bash
               #!/usr/bin/env bash

               function 실행중인가요
               {
                if [-s $DOORAY_PID_FILE ]
                then
                  ps -p $(cat $DOORAY_PID_FILE) > /dev/null
                  return $?
                else
                  return 1
                fi
               }
              ```
            - JVM 실행 옵션 _-XX:+ExitOnOutOfMemoryError_
              ```bash
               #!/usr/bin/env bash

               java -Xmx10m \
                -XX:+HeapDumpOnOutOfMemoryError \
                -XX: +ExitOnOutOfMemoryError \
                -jar may_cause_oom.jar
              ```
            - JVM 실행의 옵션 _-XX:OnOutOfMemoryError (Introducedin 1.4.2 update 12, 6)_
              ```bash
               #!/usr/bin/env bash

               java -Xmx10m \
                -XX:+HeapDumpOnOutOfMemoryError \
                -jar may_cause_oom.jar
              ```    
        - 쿠버네티스의 livenessProbe 사용
            - pod - service, kubelet이라는 데몬이 livenessProbe의 pod를 계속확인 → 문제가 발생하면 pod 삭제하고 새로운 pod를 올린다.
        - springboot actuator healthcheck endpoint
            - 주요기능 점검 기능
              ```bash
                #!/usr/bin/env bash
                
                funcion 건강한가요
                {
                  health=$(curl -I http://localhost:9999/actuator/health 2>/dev/null | head -n 1 | cut -d$'' -f2)
                  if [ $health -eq "200" ]
                  then
                    return 0
                  else
                    return 1
                  fi
                }
              ```
        - 효과: 무중단 서비스 유지 
3. 과부하를 처리하는 방법   
    1. cascading faiures
        - 특정서버 하나가 문제인데, 연결된 서버까지 전달되는
        - 원인: 일부 서버 과부하, 자원의 부족
        - 시나리오: 게이트웨이를 통해 → 백엔드 버서 → 계정관리 서버, 계정관리 서버 과부하 백엔드서버가 아웃, 두번째 백엔드 서버 아웃, 게이트웨이 죽음
        - 자원의 추가 투입으로 해결할 수 있지만..?
    2. 처리방법
        - Speing boot의 actuator 활용: metric과 prometheus endpoint 활용
    3. 대응방안
        - scale up/out: 최번시 사용량을 위한 리소스를 항상 확보하기엔 비용 이슈
        - message broker로 지연처리
        - laas,k8s의 auto scale 사용: 단시간에 폭증하는 요청을 커버할 수 없음, managed k8s가 아니면 scale out과 다를 바 없는 비용이슈
        - circuit breaker pattern: 이미 서버가 부하가 발생한 후에 동작, 장애가 일시적으로 사용자에 노출
        - 수신거부
    4. 429 적용(Too many Request)
        - 기준을 초과한 상황에서 응답 거부
        - 사용자에게 노출되지 않는 api에 적용
          - <image width="300" src="https://user-images.githubusercontent.com/72849620/203883237-52e4fcd7-8b48-40b8-aff5-790322cbedd6.png">
    5. 기준적용
        - 초당요청량 / CPU 사용량:응답 지연으로 백엔드에서 병목이 발생하면 cascase failure 유발 / Active thread수
        - active thread 사용: Spring boot
          ```java
            /*ThrottlingByThread.java*/
  
            private static final double MAX_BUSY_THREAD_RATIO_FOR_MAIL_ RECEIVE = 85;
            private final MetricsEndpoint metricsEndpoint;
  
            public void checkBusyThread() {
              MetricResponse busyThreadMetric = metricsEndpoint .metric("tomcat. threads busy", null);
              MetricResponse maxThreadMetric = metricsEndpoint.metric("tomcat . threads. config-max", null);
            
              //생략 ..
        
              double busyRatio = busyThreadCount / maxThreadCount * 100;
  
              if (busyRatio > NAX_ BUSY_ THREAD_ RATIO_FOR MAIL_ RECEIVE) {
                throw new TooManyRequestException("busy: " + busythreadCount + ", ratio: " + busyRatio);
              }
            }
          ```
4. http cache
    1. 브라우저의 http cache
        - 네트워크 리소스를 줄여 빠르게 페이지를 로딩하는
    2. 이거를 백엔드 서버간 통신에 사용?
        - cache-control header: 캐시는 사용하기 이전에 기존 리소스의 상태를 반드시 확인
        - etag header: 응답하는 리소스의 버전을 제공
        - if-none-match header: etag를 비교하여 다른 경우에만 응답을 제공
        - eTag 구현: webRequest 사용
          ```java
            // ETag를 응답에 포함, 검증
            @GetMapping (value = "/members/{memberId}/resources")
            public ResponseEntity<List<Resource>> list(@PathVariable("memberId") final Long memberId, WebRequest webRequest) {
                // ... Long updatedAt = ...
                if (webRequest. checkNotModified (genETag (updatedAt))) (
                  // shortcut exit - no further processing necessary
                  return null;
                }
                // ...List‹Resource> resources = ...
            }
          ```
    3. apache storage backend
        - 캐시한 response body를 저장할 공간이 필요
    4. http cache 주의
        - shared cache 주의
        - method: get에서만 적용됨
        - storage backend: 데이터를 많이 보유할수록 hit 할 확률이 높아짐
    5. 코드
        - 효율적인 HTTPCache 서비스 구현
          ```java
            // Cache Control 속성 설정
  
            @GetMapping(value = "/members/{memberId}/resources")
            public ResponseEntity<List<Resource»> list(@PathVariable("memberId") final Long memberId, WebRequest webRequest) {
              
              // ...List<Resource> resources = ...
  
              return ResponseEntity.ok()
                .cacheControl(CacheControl.maxAge(0, TimeUnit.MINUTES)
                .cachePrivate()
                .mustRevalidate())
                .body (resources);
          ```
        - 서버 간 HTTPCache 클라이언트 구현
          ```java
            CachingHttpClients.custom()
                .setHttpCacheStorage(
                    new EhcacheHttpCacheStorage (httpClientCache, CacheConfig.custom().setMaxUpdateRetries(2).build(),
                          new DoorayHttpCacheEntrySerializer()
                    )
                )
                .setCacheConfig(
                    CacheConfig.custom().setMaxCacheEntries(Integer.MAX_VALUE)
                        .setMaxObjectSize(20L # 1024L # 1024L)
                        .setsharedCache(false)
                        .build()
                )
          ```
          ```java
            ## ehcache. xml
            ‹cache name="httpClientCache"
                  maxEntriesLocalHeap="20"
                  maxBytesLocalDisk="10g" 
                  eternal="false" 
                  timeTold leSeconds="1800" 
                  timeToLiveSeconds="1800" 
                  diskExpiryThreadIntervalSeconds="5" 
                  memoryStoreEvictionPolicy="LRU" 
                  transactionalMode="off" 
                  overflowTo0ffHeap="false" 
                  statistics-"true">
                  ‹persistence strategy="localTempSwap"/>
            </cache>
          ```
5. netty writing backpressure
    1. netty로 구현한 소켓서버
        - 기능: 사용자 클라이언트용 소켓서비스
        - TCP 기반
    2. 코드리뷰
        ```java
          public class DoorayWriteHandler extends ChannelInboundHandlerAdapter {
  
              @Override
              public void channelActive(ChannelHandlerContext ctx) throws Exception {

                Iterator<Content> contents = getcontents(); // 10,000 건 이상...
  
                while (contents.hasNext()) {
                  ctx.channel().writeAndFlush(contents.next());
                }
              }
          }
        ```
  
        ```java
          public class DoorayWriteHandler extends ChannelInboundHandlerAdapter {
                                  
            @Override
            public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                  
                Iterator<Content> contents = getcontents(); // 10,000 건 이상...
                
                while(contents.hasNext()) {
                  Thread.sleep(10L): // 이건 좀... 그래도 OutOfMemoryError는 막아봅시다.
                  ctx.channel().writeAndFlush(contents.next());
                }
            }
          }
        ```
  
        ```java
          public class DoorayWriteHandler extends ChannelInboundHandlerAdapter {
    
            @Override
            public void channelActive (ChannelHandlerContext ctx) throws Exception {
              processIfPossible(ctx. channel());
            }
  
            @Override
            public void channeWritabLilityChanged(ChannelHandlerContext ctx) throws Exception {
              processIfPossible(ctx. channeL());
            }
  
            //다음 페이지..
        ```
  
        ```java
          public class DoorayWriteHandler extends ChannelInboundHandlerAdapter {
  
            private volatile int id = 0
  
            //필요하면 멈추거나 다시 시작할 수 있어야 한다.
  
            protected void processIfPossible(Channel channel) {
              Iterator‹Content> contents = getContentsFrom(id);
  
              while(contents.hasNext() && channel.isWritable()) {
                Content content = contents.next;
                this.id = content .getId();
                ctx.channel() .writeAndFlush(content);
              }
            }
          }
        ```
  
        ```java
          public class BootStrap {
  
            public static void main(String[] arg) {
              ServerBootstrap b = new ServerBootstrap);
  
              //MATER_ MARK 를 Low: 1024, high: 8192 로 설정합니다.
              b.childoption( ChanneLOption.MRITE_BUFFER_WATER_MARK, new WriteBufferWaterMark(1*1024, 8*1024) );
            }
          }
        ```

6. 마무리
    1. 지속적인 작업: 테스트 코드 작성
    2. 가시화: 지표확보, 성과와 개선방향을 얻을 수 있음
    3. 자동화: 반복적인 작업은 자동화 하라
        
<br/><br/>



## **분산 시스템에서 데이터를 전달하는 효율적인 방법**

1. 분산시스템
    1. 목표를 달성하기 위해 여러 개의 컴퓨터 리소스를 사용하는 시스템
    2. ex) 엔터프라이즈, 마으크로 서비스 아키텍처, 모놀리식 아키텍처 + 검색엔진
    3. 네트워크를 사용하여 컴포넌트 간의 기능을 통합
2. 데이터 전달방법
    1. remote api
        - 서버-클라이언트
        - 케이트웨이: 사용자 요청에 즉각 응답하는 api 주로 사용하는 방식
        - 비교적 간단한 개발
    2. messeagequeue
        - publisher - consumer
            - consumer가 데이터 조작(crud)
            - messageQueue
        - 배치작업, 비동기 작업에서 주로 사용
        - 비교적 복잡한 개발
3. 효율적방법
    1. 모든 컴포넌트들은 네트워크로 연결
    2. 문제는? 네트워크는 신뢰할 수 없는 매체
        1. packet loss
        2. latency
        3. network down
    3. 데이터 유실에 대비
4. 데이터 전달 보장 방법
    1. 신뢰성 가장 낮은 at-most once delivery
        - fire and forget
        - 장점: 간단, 단점: 메세지 유실
    2. at-least-once delivery
        - 최소 한번 이상 발송, 수신
        - 장범: 효과 대비 쉬운 개발, 단점: 멱등성을 보장해야 하는 consumer
    3. exactly-once dlivery
5. RDB를 사용하는 애플리케이션에서 전달 방법
    1. @transactionalEventilistner
    2. @Retryable
      ```java
        @Service
        public class EventHandler {
  
          @Retryable(
            maxAttempts = 3,
            backoff = @Backoff (delay = 100L)
          )
          @TransactionalEventListener(phase = TransactionPhase. AFTER. COMMIT)
          public void propagate (CreateTaskEvent event) {
            // 이벤트 발생 로직
            // + restTemplate.execute(...);
            // + rabbit Template. send(...)
          }
        }
      ```
  
      |필드명|데이터타입|설명
      |-------------|---------|-------------|
      |event_id|BIGINT|이벤트의 순서를 보장|
      |created_at|Datetime(3~6)|이벤트 발생 시간|
      |status|smallint|Ready(0) /Done(1)|
      |payload|jsonb|JSON 타입의 Message payload|
      
    3. 마이크로 아키텍처 패턴
        - Transactional OutboxPattern
        - Polling publisher pattern
        - 장점: REST-API에서 al-leat-once를 구현할 수 있다.
        - 단점: 데이터베이스 부하
6. RabbitMQ를 사용한 전달방법
    1. publich/subscibe 방식 지원
    2. ack를 메시지 응답 처리 메커니즘
    3. AMQP
    4. correlationData를 사용한 코드
      ```java
        @Service
        public class MessagePublisher {
  
          public void sendMessage (CreateTaskEvent createTaskEvent) throws JsonProcessingException
            String ison = objectMapper.writeValueAsString (createTaskEvent);
            rabbitTemplate.send(EXCHANGE_NAME,
                                 ROUTING_KEY,
                                 new Message(i son. getBytes (StandardCharsets. UTF_ 8) new CorrelationData(UUID. randomUUIDO. toString()));
      ```
    5. confirmCallback을 사용한 코드
        ```java
            @Bean
            public RabbitTemplate rabbitTemplate (ConnectionFactory rabbitConnectionFactory) {
                RabbitTemplate rabbitTemplate = new RabbitTemplate(rabbitConnectionFactory);
    
                //  ...
                rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
                    if (lack) {
                        Message message = correlationData. getReturned () • getMessage ();
                        byte[] body = message.getBody);
                        log.error("Fail to produce. ID: {}", correlationData. getId());
                    }
                });
                return rabbitTemplate;
            }
        ```
    6. Cunsumer Ack
    7. @RabbitListener
7. Kafka를 사용한 전달 방법
    1. projucer confirm
        ```java
            @Slf4j
            @Component
            @RequiredArgsConstructor
            public class Producer {
    
                public void sendEvent (CreateTaskEvent event){
                    ListenableFuture<SendResult<String, CreateTaskEvent>> future kafkaTemplate.send (TOPIC_TASK, event);
                    future.addCallback(
                        result -> log.info("offset : {)", result. getRecordMetadata() .offset ()), throwable -> log.error("fail to publish", throwable)
                    );
                }
            }
        ```
    2. consumer ACK
8. 마무리
    1. event driven architecture의 기본은 데이터 전달
    2. 최소 At leadt once 설정
    3. producer confirm, consumer ack 고려


<br/><br/>

## **로그인에 사용하는 OAuth: 과거, 현재 그리고 미래**

1. OAuth 1.0
    1. 구성
        - resource Owner
        - OAuth client
        - OAuth Server
    2. 문제점
        - scope 개념 없음
        - 토큰 유효기간 문제
        - 역할이 확실히 나누어 지지 않음
        - client 구현 복잡성
        - 제한적인 사용 환경
2. OAuth 2.0 인가 프레임워크
    1. scopre 기능 추가: 토큰이라는 인가를 나타내는 값만 있으면 사용자에게 접근 가능, 해탕 토근을 통해 접근 범위를 나타냄
    2. 복잡성 간소화: 1.0은 보안성을 가져가기 위해 암호화적 보호 → 매번 서명 논문을 만들어야 함 http method, base url, parameter 에 대한 분류가 필요했다.  / bearer token+TLS로 해결했다. 토큰을 소유하고 있는 것만으로도 권한을 부여했다.
    3. 토큰 탈취 문제 개선: refresh token이라는 개념을 새롭게 만듬. 접근할 때는 access token을 사용해 접근하고(굉장히 짧은 기간)
    4. 제한적인 사용 환경: grant
    5. 현재 가장 널리 쓰이는 인가 프로토콜
3. OAuth 2.1 인가 프레임워크
    1. 보완 RFC
    2. PKCE : 시작요청+hash 보내고, 완료 요청 + 확인값 원문
    3. BCP: refresh token 회전 방식 
4. 인가를 사용한 Pseudo - 인증
    1. Open ID Connect: 2.0 토큰을 발급받고, jwt key id token을 발급
5. GNAP
    1. 프로토콜 핵심 개념 Grant → Interaction
    2. 자연스러운 PKI 연동 및 client 보안
6. 마무리
    1. 1.0: oauth 인가 프로토콜 개념 제시, 브라우저 환경 동작
    2. 2.0:  client 간소화, refresh 토큰 등장, 여러가지 grant 추가
    3. 2.1: 보안성 개선, IOT 기기 지원 및 모범 사례의 스펙화
    4. Open ID: 인증계층 추가, API 호출 없는 사용자 정보 확인
    5. IGNAP: nteraction 확장, 현/미래 기술들과 호환


<br/><br/>

## **클린 아키텍처 애매한 부분 정해 드립니다.**

1. 소프트웨어 아키텍처의 중요성
    1. 아키텍처? : 소프트웨어가 제공하는 가치 중 구조
2. 좋은 아키텍처가 중요한가?
    1. 유지보수
    2. 코드읽기, 코드이해, 코드짜기
    3. 구조가 좋다 = 수정하기 쉽다 = 수정하는데 비용이 적게 든다.
3. 좋은 아키텍처를 구성하는 방법
    1. 아키텍처 원칙을 잘 지켜서 코딩하자
        - 패러다임
        - 설원칙(SOLID)
        - 컴포넌트 응집성 원칙
        - 컴포넌트 결합 원칙
    2. 아키텍쳐 패턴: 좋은 아키텍처를 잡기위한 레시피
        - 계층형 아키텍처
            - 특징: 수평적 계층화
            - 장점: 구조 단순, 보편적
            - 단점: 도메인에 대해 아무것도 말해주지않음, 커지고 복잡해지면 조직화에 도움안됨, 데이터베이스 설계유도
        - 클린 아키텍처
            - 특징: 도메인이 중심(의존성 역전이용)
            - 장점: 도메인이 세부사항에 의존하지 않음, ddd 적용 용이 , 비즈니스 규칙에 집중 쉽다.
            - 단점: 계층형보다 패키지 구조가 복잡, 레퍼런 수가 적음
4. 클린 아키텍처
    1. 여러 아키텍처를 하나로 통합 시도.
        - 관심사의 분리, 의존성 방향은 안쪽과 고수준을 향함
    2. 바이블 쿸북
        1. 클린 아키텍처(로버트 마틴), 만들면서 배우는 클린 아키텍처
5. 헥사고날 아키텍처
    ![image](https://user-images.githubusercontent.com/72849620/203903995-ed8302db-71c8-474e-a839-e5e43854dacb.png)
6. 아키텍처별 패키지
    1. 계층형 아키텍처
    2. 기능기반 패키지
    3. 포트와 어댑터(헥사고날)
7. 클린 아키텍처는 애매하다
    1. 규칙이 단순하다. 핵심규칙 2가지 외에는 케바케
    2. 애매한 것을 판단할 기준
        1. 필요한 시스템을 만들고 유지보수하는데 투입되는 인력 최소화에 유리한가?
        2. 소스코드 의존성이 안쪽으로, 고수준의 정책을 향하는가?
        3. 세부사항이 변경되어도 도메인에 변경이 없을 것인가
        4. 테스트 쉬움?
        5. 각각의 원칙을 잘 지키고 있는가
8. 이럴 땐 쓰지마라
    1. 소규모 프로젝트
    2. 개발자 모두가 클린 아키텍처를 이해하고 있지 않을때
    3. 모두가 사용하기로 합의하지 않았을 때
9. 코드가 얼마나 늘어나나?
    1. 라인수나 파일 수보다 패키지수가 많이 늘어난다,
10. JPA Entity와 domain entity 분리해야 하나?
    1. 분리하지 않으면 클린 아키텍처가 아님.
    2. 업무규칙 뿐 아니라 연관관계 매칭을 고려해야 한다.
    3. JPA Repository는 출력 포트가 아니라 어댑터
11. 유스케이스가 다른 유스케이스를 호출해야 할떄?
    1. 컴포넌트를 별도 코드 베이스로 분리하려는 게 아니라면 직접 호출
12. 유스케이스를 꼭 인터페이스로 뽑아야 하나?
    1. 인터페이스로 뽑아야 한다
    2. controller가 service의 구현에 대해 너무 많이 알지 못하도록 막기 위해 존재한다.
    3. 사고의 경계를 그어주는 역할
13. 마무리
    1. 소프트웨어에서 아키텍처는 중요하다: 코드를 읽고 이해하고 수정하기 때문에 
    2. 계층을 나눈다음에 더 중요한 계층에 초점

<br/><br/>

## **Spring Cloud 기반 MSA 환경을 쿠버네티스로 전환하기**

1. 샵바이
    1. 클라우드 이커머스 플랫폼
    2. 마이크로 서비스 구조
        ![image](https://user-images.githubusercontent.com/72849620/203904064-82d7d8ca-6037-47e8-99d5-a7516d6c0c34.png)

2. 쿠버네티스로 전환
    1. 마이크로서비스가 너무 많다
        - 고정적인 스케줄링, 잦은 스케일링, 관리의 어려움 등
    2. 폭등하는 트래픽 양은 오토 스케일링으로 해결불가
    3. 쉽지 않은 서버 증설 + 스케일아웃(private cloud)
        - 서버증설 → 서버 세팅(배포환경 세팅) → 배포 설정 변경 → 배포
3. 쿠버네티스 전환 준비하기
    1. 코드변경 없이 쿠버네티스로 전환
    2. 쿠버네티스에서 제공하는 기능을 적극 활용
    3. dependency
        - API Gateway = Spring Cloud Gateway → Ingress
        - 내부통신 = Spring Cloud Openfeign → Spring Cloud OpenFeign
        - 서비스 레지스트리 = Spring Cloud Zookeeper 
        - 서비스 레지스트리 = Zookeeper → Service
        - 프로퍼티 파일 관리 = Spring Cloud Config → ConfigMap, Secret
        - MySQL, Mongo, Kafka, Redis등 = 사용중
    4. 서비스 레지스트리
        - spring cloud openfeign: 어노테이션 기반으로 작성되는 선언적 REST client
        - zooleeper? 쿠버네티스 service
        - spring cloud kubernetes: spring cloud에서 제공하는 인터페이스 중 몇가지에 대해 쿠버네티스 리소스를 활용한 구현체를 제공
        - spring cliud gateway: spring framework5 기반, host&path 기반 라우팅 지원, custom filter 구현 가능, path rewrite
        - API  gateway : ingress VS spring cliud gateway
    5. 프로퍼티 파일 관리
4. 쿠버네티스에 배포하기
    1. kubernetes objects
    2. helm으로 패키징 : 쿠버네티스 리소스 패키지 도구. chart로 관리


<br/><br/>


## **DDD 뭣이 중한디**

1. DDD에 대한 오해
    1. ~~DDD는 전술적 패턴이다.~~
        - 전략적 설계가 중요하다.
    
        |설계|전략적 설계|전술적 설계|
        |-----|-----|-----|
        |범위|전반적|특정 Bounded-Context|
        |목적|문제 도메인을 해결영역으로|풍부한 도메인 모델 적용|
        |메타포|전쟁에서 전략|전투에서 전술|
        |주요 패턴|Bounded-Context, Ubiquitous-Language, Aggregate, Domain-Event|
        |수행 방식|접근법|상대적으로 방법론에 가까움|
    
    2. 단순한 서비스에는 적용하지 않는 것이 좋다.
2. Domain Driven Design 도메인 주도 설계
3. 전략적 설계
    1. 비즈니스 도메인
    2. 문제공간: 하위 도메인 추출
        ex) fasticket.com의 문제공간: 인터넷 예매 /  하위 도메인: 예매, 도면, 상품, 회원
    3. 문제 도메인에서 핵심 하위 도메인 식별하기
        ex) 핵심: 예매 / 지원: 도면, 상품 / 일반: 회원
    4. 하위 도메인 유형: 핵심(난이도 최상, in-house), 지원(난이도 하, in-house), 일반(난이도 상, 솔루션구매)
    5. 해결공간: 개발 영역의 설계가 시작된다.
    
        ![image](https://user-images.githubusercontent.com/72849620/203905570-96a61d6e-4654-4095-b0b0-c3641f2eb3a9.png)
    
4. 브라운 필드 전략적 설계
    1. big ball of mud: 복잡성 증가
5. 전략적 설계시 유용한 도구
    1. event storming
    2. 사용사례 분석
    3. bussiness model 분석
    4. 지식탐구와 커뮤니케이션 (중요)
6. 전략적 설계 과정
    1. 비즈니스 도메인에서 문제 도메인추출
    2. 하위 도메인 추출
    3. 핵심 하위 도메인 식별
7. bounded-Context
    1. 모델 무결성
    
       ![image](https://user-images.githubusercontent.com/72849620/203905982-8758f032-8c7a-4880-bc9d-1d2eebb03440.png)

8. 콘웨이 법칙
    1. 소프트웨어의 구조는 해당 소프트웨어를 개발하는 조직의 구조를 따라간다.
9. 역 콘웨이 법칙
    1. 개발하는 조직의 구조를소프트웨어의 구조에 맞춘다.
10. 진화하는 설계
    1. Legacy(bbom) → 문제공간 식별하기 → 핵심도메인을 식별하고 분리 → DDD로 고도화 → 전술적 패턴 선택 → 
11. 마무리
    1. 일종의 접근법이자 철학이다. 방법론적처럼 명확하지 않고 추상적
    2. 기술보단 예술
    3. 전략적 설계가 중요하다
    4. 커뮤니케이션 바탕으로 지식탐구
    5. 언제 어디서나 유비쿼터스 언어로 소통한다.


<br/><br/>

_2022.11.24_

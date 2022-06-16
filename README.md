# oneqtour

하나1Q투어 (oneQtour ) 
 - 목돈/잔돈/포인트 모아 MD가 추천한 여행상품(적금+여행할인예약)을 구독하는 서비스 

본 킹스톤은 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 실습하여 구성한 예제입니다. 이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.

체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW
Table of contents
킹스톤과제 - 하나1Q투어 

서비스 시나리오
체크포인트
분석/설계
구현:
DDD 의 적용
폴리글랏 퍼시스턴스
폴리글랏 프로그래밍
동기식 호출 과 Fallback 처리
비동기식 호출 과 Eventual Consistency
운영
CI/CD 설정
동기식 호출 / 서킷 브레이킹 / 장애격리
오토스케일 아웃
무정지 재배포
신규 개발 조직의 추가
서비스 시나리오

![image](https://user-images.githubusercontent.com/58426170/173987544-023b846a-2ca9-4340-adea-25cb005799ff.png)



<b>기능적 요구사항</b>

여행MD가 여행상품을 만들어 등록/수정/삭제한다.
만들어진 여행상품은 1Q Store페이지에 진열/수정/삭제된다.
고객이 1Q Store페이지에서 마음에드는 여행상품을 선택하여 예약한다.
예약과 동시에 결제가 진행된다.
예약이 되면 예약 내역이 업데이트된다.
고객이 예약을 취소할 수 있다.
예약 사항이 취소될 경우 취소 내역이 업데이트된다.
//전체적인 여행상품(적금+여행상품)대한 정보 및 예약 상태 등을 한 화면에서 확인 할 수 있다.(viewpage)

<b>비기능적 요구사항</b>

 <b>트랜잭션</b>
 결제가 되지 않은 예약 건은 성립되지 않아야 한다. (Sync 호출)

 <b>장애격리</b>
 MD상품 등록 기능이 수행되지 않더라도 예약은 365일 24시간 받을 수 있어야 한다 Async (event-driven), Eventual Consistency
예약 시스템이 과중되면 사용자를 잠시동안 받지 않고 잠시 후에 하도록 유도한다 Circuit breaker, fallback

 <B>성능</B>
 모든 여행상품에 대한 정보 및 예약 상태 등을 한번에 확인할 수 있어야 한다 (CQRS)
 예약의 상태가 바뀔 때마다 MD에게 업데이트 해야 한다 (Event driven)

체크포인트
1. Saga
2. CQRS   <--- view page 
3. Compensation / Correlation
4. Request / Response
7. Circuit Breaker
5. Gateway


<B>분석 설계</B>

이벤트스토밍:

스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?
서브 도메인, 바운디드 컨텍스트 분리

팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
적어도 3개 이상 서비스 분리
폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
컨텍스트 매핑 / 이벤트 드리븐 아키텍처

업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?
헥사고날 아키텍처

설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?
구현

[DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?

Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
[헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
Request-Response 방식의 서비스 중심 아키텍처 구현

마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
서킷브레이커를 통하여  장애를 격리시킬 수 있는가?
이벤트 드리븐 아키텍처의 구현

카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
Correlation-key: 각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?
폴리글랏 플로그래밍

각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?
API 게이트웨이

API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
운영

SLA 준수
셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
모니터링, 앨럿팅:
무정지 운영 CI/CD (10)
Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명
Contract Test : 자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?

분석/설계


TO-BE 조직 (Vertically-Aligned)
image

Event Storming 결과
MSAEz 로 모델링한 이벤트스토밍 결과: http://www.msaez.io/#/storming/QtpQtDiH1Je3wad2QxZUJVvnLzO2/share/6f36e16efdf8c872da3855fedf7f3ea9
이벤트 도출
- 도메인 서열 분리 
   - Core Domain:  reservation, store : 없어서는 안될 핵심 서비스이며, 연간 Up-time SLA 수준을 99.999% 목표, 배포주기는 reservation 의 경우 1주일 1회 미만, room 의 경우 1개월 1회 미만
    - Supporting Domain:   viewpage : 경쟁력을 내기위한 서비스이며, SLA 수준은 연간 60% 이상 uptime 목표, 배포주기는 각 팀의 자율이나 표준 스프린트 주기가 1주일 이므로 1주일 1회 이상을 기준으로 함.
    - General Domain:   payment : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 
폴리시 부착 (괄호는 수행주체, 폴리시 부착을 둘째단계에서 해놔도 상관 없음. 전체 연계가 초기에 드러남)
폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)

<B>완성된 모형</B>
- View Model 추가

여행MD가 여행상품을 만들어 등록/수정/삭제한다.(ok)
만들어진 여행상품은 1Q Store페이지에 진열/수정/삭제된다.(ok)
고객이 1Q Store페이지에서 마음에드는 여행상품을 선택하여 예약한다.(ok)
예약과 동시에 결제가 진행된다.(ok)
예약이 되면 예약 내역이 업데이트된다.(ok)
고객이 예약을 취소할 수 있다.(ok)
예약 사항이 취소될 경우 취소 내역이 업데이트된다.(ok)
//전체적인 여행상품(적금+여행상품)대한 정보 및 예약 상태 등을 한 화면에서 확인 할 수 있다.(viewpage - 만들지 여부는 고민 )

![image](https://user-images.githubusercontent.com/58426170/173989073-4b128663-9830-4d77-8b81-17ed7fead393.png)



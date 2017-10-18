### Monolithic Architecture
> The traditional unified model for the design of a software program   
> Web Service에서의 Monolithic Architecture란 최종 application을 구성하는데 필요한 모든 컴포넌트들을 `통합`하여 개발 및 배포를 하는 설계를 말한다.

![이미지 이름]()
#### Monolithic Architecture의 장단점
>장점   

- 여러 컴포넌트들의 통합을 지향하기 때문에 application을 배포하는 과정이 간단하다.   
- End point의 사용자가 최종적으로 Web Service를 이용하기 까지의 모든 컴포넌트들이 한개의 묶음으로 이루어져 있기 때문에 복사본을 만들기가 쉽다.   
즉, 로드 밸런싱을 하기 위한 application의 복사과정을 쉽게 할 수 있다.
- application형태로 통합되어 있어 application 단위의 Test가 용이하다.   
- 트랜잭션 처리가 용이하다.   

>단점   

- 모든 컴포넌트들을 하나로 통합하기 때문에 Application의 규모가 커짐에 있어서 확장성이 매우 떨어진다.    
- 일반적으로 Monolithic한 설계는 각 컴포넌트간 결합도가 높다.   
즉, 작은부분에 대한 수정이 많은 곳에 영향을 미친다.   
- 배포를 하는 과정은 쉬우나 배포의 단위가 application 전체이므로 수정한 양이 작을지라도 전체 application을 다시 배포해야한다.   
확장성문제와도 맞물려 application 규모에 때라 이 문제 또한 심각해진다.   

#### 해결 방안
> Loosely Coupling   

Monolithic으로 설계 및 구현한 검색 및 주문 Application을 생각해보자.    
개발 초기 단계에서는 검색 서비스와 주문 서비스, 2개의 서비스만 필요하여   
검색기능과 주문기능이 밀접하게 연관이 되어 있었다.   
하지만 이후 추가적인 서비스들을 추가시키면서 이 서비스들도 검색 서비스와 연동을 시키고자하는 Needs가 생겼다.   
현재 검색 서비스는 주문을 위한 목적으로 설계 및 구현을 한 상태이기 때문에 다른 서비스에서도 검색 서비스를 이용할 수 있도록 수정하였다.   
수정한 결과, 검색 서비스와 다른 서비스는 잘 연동되었지만 기존에 잘 작동하던 주문 서비스에서 상품을 검색하는 부분에 문제가 생겼다.   

위의 예는 Monolithic 설계를 통해서 서비스(모듈)간 결합도가 높게 구성된 경우 흔히 발생하는 문제이다.   
이러한 문제는 확장성이 높은 환경에서 자주 발생하게 된다.   

이를 해결하기 위해서는 서비스간 결합도를 낮추는 방법으로 구현을 해야한다.   
Web Framework인 Spring Framework도 웹 어플리케이션을 개발함에 있어서 모듈간 결합도를 떨어뜨리는 설계방법을 지향하고 있다.   Spring MVC가 대표적인 예라고 할 수 있다.   

> REST를 통한 View의 분리

Spring MVC를 통해서 크게 Model, View, Controller 이렇게 3개의 영역으로 나누어 서로의 결합도를 낮추었다.   
또한, 서버를 통해 Data를 담은 HTML페이지를 랜더링해서 보여주어야 했던 과거의 동작방식과 달리, 웹 클라이언트의 기술이 눈에 띄게 발전하면서 Script를 이용한 비동기 통신이 널리 사용되게 되었다.    
이를 통해 서버는 구조화된 Data만 Response하여도 클라이언트 측에서 Page 랜더링을 담당할 수 있게 되었다.   
더욱이 REST API의 등장으로 기존의 서버의 규모에서 View를 담당하는 정적 파일과 데이터부터 비지니스 로직까지를 담당하는 서버파트로의 분리가 가속화 되었다.   
물론, 서버로 부터 View를 완전히 분리하는 것이 모든상황에서 절대적으로 좋은 것은 아니지만 분리를 하는 것이 가능해졌다는 것이 중요하게 생각해야할 부분이라고 생각한다.   

### Micro Service Architecture
> Micro Service Architecture는 앞에서 얘기한 해결방안의 논리적인 분리에서 더 나아가 물리적인 분리를 통해 Application을 Service단위로 나누어 설계하는 방법을 말한다.   

![이미지 이름]()

> 배경

- 전통적인 Monolithic설계방법의 단점을 극복하기 위해 각 모듈간 결합도를 낮추어 구현하는 방법들이 많이 적용되게 되었다.   

- REST API를 통해 API를 통한 DATA 통신방법이 잘 사용되고 있다.

> 특징

- 일반적으로 war파일을 통한 Monolithic한 배포를 한다는 것은 다르지 않지만 배포의 단위가 Application전체에서 Service로 줄어들었다는 점이다.   
즉, Monolithic 설계로 부터 얻을 수 있는 배포파일 복제와 배포속도의 장점만을 취하고 Service를 확장하는데 있어서 발생하는 Overhead는 없애버리는 설계방법이다.   

- 서비스가 확장됨에 따라 데이터베이스도한 확장되게 되는데 Micro Service 설계방식은 데이터베이스 또한 서비스의 관점에서 분리를 한다.
- 각 서비스를 호출하는 클라이언트는 API를 통해 모든 서비스와 통신을 한다.   
(클라이언트 입장에서는 각각의 서비스의 End Point를 알아야한다는 문제가 있다.)   

#### API Gateway
> Micro Service Architecture의 문제 중 하나는 서비스 단위로 물리적 분리가 되면서 클라이언트가 알아야할 서비스의 EndPoint가 많아 졌다는 점이다.

API Gateway는 모든 Api 서버들의 EndPoint를 단일화 하여 클라이언트로 부터 실제 서비스들의 End Point를 숨겨주는 기능을 한다.   

> 주요 기능

- 인증/인가   

해당 API를 호출하는 클라이언트에 대한 신분 확인과 접근 권한을 확인하는 기능을 한다.

- API token   

같은 API에 대한 반복되는 인증를 피하기 위해 최초 인증 시 Token발급한다.   

- API 라우팅

  1. 1개의 End Point에 트래픽이 집중되지 않게 부하분산기능을 한다.   
  2. 여러 End Point로 부터 Data를 받아오는 요청의 경우 노출 시키는 URI를 원하는 규격에 맞게 변경할 수 있다.
  3. HTTP 헤더를 통한 라우팅   

- 공통 로직 처리

여러 End Point에서 공통적으로 처리되는 부분의 경우 API Gateway를 통해 공통로직처리를 할 수 있다.   

- Text format Translation  
JSON 이나 XML 등 메세지 포맷을 원하는 포맷으로 바꾸는 기능

- Message Exchange Pattern MEP
Sync to Async, Async to Sync 등 호출 방식을 변환하는 기능

- Aggregation
SOA에서의 Orchestration과 유사한 것으로 여러 API를 통합하여 새로운 API로 만드는 기능
Aggregation작업은 API GateWay에게 많은 부하를 주는 기능이 때문에 다른 End Point 서버 처럼 Aggregation작업을 하는 서버를 별도로 추가해서 사용한다.   

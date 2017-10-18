[넷플릭스 마이크로 서비스 가이드 영상](https://www.youtube.com/watch?v=OczG5FQIcXw&feature=youtu.be)
### NetFlix 의 마이크로 서비스
> 과거 2000년도 NetFlix가 DVD data 센터를 운영하던 시절의 시스템 구성도는 다음과 같다.   
> 과거의 시스템은 Monolithic한 구조였다.    

![이미지 이름]()

> 특징

- 1개의 WAS에 모든 서비스가 집약되어 있다.   
- 복사본을 여러개 생성하여 앞에 로드밸런서를 두고 부하분산을 하였다.
- DataBase의 경우 Oracle을 사용하였고 Oracle의 DB Link 기능을 이용해
다른 DataBase와 연결하였다.

>문제

- 운영하는 모든 팀들이 한개의 프로젝트에 모두 붙어서 개발 및 운영을 해야했다.
- DataBase가 장애가 생기면 전체 시스템이 중단되어야 했다.   
- 휴가철 같이 평소와 달리 트래픽이 많아지는 날 이 트래픽을 견뎌낼 수 있는 하드웨어를 찾아다녀야 했다.

#### Micro Service Architecture
> An Approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweigth mechanisms, often an HTTP resource API.   
-Martin Fowler   

- 시스템이 동작하면서 발생할 수 있는 여러 위험을 잘게 쪼개어 기존의 Monolithic 설계의 문제점을 보완하는 설계방법

- 특정한 서비스를 담당하는 하나의 Application의 구조는 다음과 같다.

![이미지 이름]()   

이러한 구조들을 조합한 것이 마이크로 서비스 아키텍처이다.   

#### Micro Service 시스템의 문제
1. Dependency
2. Scale
3. Variance
4. Change

> Dependency

![이미지 이름]()

- Service A가 Service B를 의존하는 경우 MSA에서는 네트워크를 통한 요청/응답을 해야한다.
- 이러한 과정에서 발생할 수 있는 잠재적인 문제들은 상당히 많다.   
- 서비스간 의존성 때문에 발생할 수 있는 가장 큰 문제는 `Cascading Failure`인데, 특정 서비스에서 발생한 장애가 연결된 여러 Service들에게 까지 영향을 미치게 되어 전체적으로 시스템전체가 장애가 생기는 문제이다.   

> Dependency의 문제 해결

- 의존성으로 인해 한 서비스에서 발생한 장애가 Cascading되는 것을 막기 위해서는 서비스 자체적으로 장애를 처리할 수 있는 방어 시스템이 필요하다.   
- NetFlix에서는 이러한 방어 시스템을 구축했는데 이 것이 `Hystrix` 이다.

![Hystrix]()

- 타임아웃이나 재시도와 같은 동작을 처리할 수 있다.   
- 문제가 발생했을 경우, Fallback을 통해 static 한 응답을 내보내고 자체적으로 발생한 문제를 외부로 보내지 않는다.  

> Scale   
1. stateless Service
2. stateful Service
3. hybrid Service  


[Stateless/Stateful Service란?](https://nordicapis.com/defining-stateful-vs-stateless-web-services/)   
클라이언트로 부터 어떠한 요청을 처리함에 있어서 `Stored Data`로 부터 참조를 통해 결과값이 변경되는 서비스를 Stateful Service라고 한다.   
즉, Stored Data에서 State의 참조를 통해 결과값을 구하기 때문에 DB나 캐시 등에 의존하는 정도가 강한 Service들을 Stateful하다고 할 수 있다.    
Stateless Service는 Stored Data에 의존성이 약한 Service를 말한다.   
즉 서버의 장애가 서비스의 장애로 연결이 될 필요가 없는 Service이다.   

- Stateless Service의 경우 문제가 발생하는 경우 특정 서버에 종속되지 않으므로 문제가 발생한 서버가 복구 될때까지 복사본을 만들어 간단하게 해결할 수 있다.   
- Stateful Service의 경우 캐시나 DB에 대해 의존성이 강하므로 서버 1대의 장애에 따른 부작용이 심각해질 수 있다.

> Scale문제의 해결

- Stateless Service의 경우 auto scaling을 통해 쉽게 확장을 할 수 있다.
- Stateful Service의 경우 DB나 Cache에 대한 의존성이 강한데 MSA에서는 각 서비스 별로 DB와 Cache또한 분할 하므로 각각의 DB와 Cache는 서로 전혀 다른 데이터를 가지고 있게 된다.   
 이렇기 때문에 특정 캐시의 노드한개에 대한 문제가 전체 시스템의 문제로 이어질 수 있다.
 - 이를 해결하기 위해 memcahed를 wrapping한 EVcache를 사용하였다.   
 즉 같은 데이터를 같은 네트워크상에만 위치하지 않고 다른 네트워크의 DB나 캐시에도 위치시킴으로써 문제를 해결하였다.    

 - Stateful Service의 경우 DB나 Cache에 의존성이 강하므로 EVcache를 통해서 이를 보완하여 NetFlix의 시스템에 EVcache를 많이 사용하게 되었다.  
 - 하지만 캐시의 본래목적은 DB로 부터의 작업을 빠르게 처리하는데 있었는데
 캐시 miss가 발생하는 경우 DB로 부터 작업을 처리해야 한다.   
 - 대용량의 데이터를 EVcache를 통해서 빠른 속도로 처리하던 서비스들이 갑자기 DB를 통해 서비스를 처리하려고 하면 DB는 캐시 수준으로 처리속도가 빠르지 않기 때문에 문제가 발생하였다.   
 - 이를 해결하기 위해 캐시를 사용하는 서비스, 즉 Statful service들 중에서  실시간으로 캐시를 사용해야하는 서비스와 배치를 위해 캐시를 사용하는 서비스로 분리를 하였고 한개의 요청에서는 동일한 캐시의 참조가 한번만 발생하도록 했다.

 > Variance

 - 여러개의 서비스가 물리적으로 분리되면서 서비스별로 다양한 아키텍처를 적용할수도 있고 상이한 기술스펙을 사용할 수도있다.
 - 다양한 접근을 할 수 있다는점에서는 좋지만 시스템 전체적으로는 복잡성이 증가하는 문제가 있고 이는 곧 서비스의 관리가 힘들어지는 이유이다.   
 - MSA는 UI로부터 요청을 전담하는 API Gateway를통해 서버 End Point가지의 중개를 담당하는데 분산된 End Point서버들이 사용하는 아키텍처와 기술스펙들이 다양해 지는 경우 API Gateway의 부하가 커지게 되고 이는 곧 또다른 형태의 Monolithic 설계가 되버리는 문제가 생기게 된다.   

 > Variance의 해결

 ![apiGatewaybefore]() ![apiGatewayafter]()

 - 새로운 기술과 연동하는 UI의 End Point를 ApiGateway 바깥으로 빼내어 각각의 서비스로 만들었다.


> Change

- 서비스 단위로 분리가 되면서 기존의 것을 변화 시키는 것에 대한 비용적인 부분은 줄어들었다.
- 하지만 변화를 적용시킨 부분에 대한 검증작업이 필요하다.   
- 즉, 반복적으로 발생하는 배포에 대해 유연하고 설정이 가능한 단계들을 제공해야한다.   

> Change의 해결

- 기존에 사용해왓던 Asgard라는 배포플랫폼을 대체하는 Spinnaker를 만들었다.
- 각각의 변경사항들을 간단하게 배포할 수 있도록 도와준다.   
- 새로운 버전이 배포되면 자동으로 트래픽의 일부가 새버전으로 흐르도록 한다.   

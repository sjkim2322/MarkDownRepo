1. spring boot jdbc dependency 추가시 실행안되는 문제
2. @SpringBootApplication과 다른 Component 사이의 디렉토리 구조에 따른 component scan문제
3. API 키값 보안과 성능 사이의문제
4. Jaxb 관련 문제
    - List형태의 데이터 unmarshal링 관련
    - Date 관련 변수들 mapping관련
    - Jaxb오류
5. try-catch-finally 문에서 close문제




#7

- @SpringbootApplication은 Spring을 구동하는데 자주사용되는 어노테이션들을 모아논 어노테이션이다.
-  자주 사용하는 어노테이션은 크게 `@Configuration`, '@EnableAutoConfiguratioin` , `@ComponentScan` 이다.
- @SpringbootApplication은 위의 3개의 어노테이션을 디폴트로 사용하는 것과 동일하게 동작한다.   


#### DB관련 모듈 Dependency 추가만으로 실행이 되지 않는 문제
> @EnableAutoConfiguration은 pom.xml에 의해 dependency가 추가된 모듈들이 필요로하는 설정파일을 자동으로 읽어드린다.
> 이 과정에서 JDBC 관련 모듈이 dependency에 추가되어 있는 경우 `DataSourceAutoConfiguration`라는 @Configuration 클래스를 자동으로 읽어드리는데 초기에 Spring Boot 구동시에 Datasource에 대한 설정정보를 따로 작성하지 않았기 때문에 이부분에서 에러가 났다.   

#### 해결 방법
1. Dependency 제거
    - 지금은 JDBC 관련 작업을 하지 않으므로 관련 Dependency를 주석처리하면 간단하게 해결된다.

2. @EnableAutoConfiguration의 exclude 속성사용
    - @EnableAutoConfiguration은 default로 dependency모듈이 필요로하는 설정 클래스를 읽으려고하지만 exclude 속성을 사용해서 사용하지 않는 설정클래스를 로드하지 않을 수 있다.
    - @SpringBootApplication은 @EnableAutoConfiguration을 포함하고 있으므로 해당속성을 똑같이 사용할 수 있다.

    ```
    @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
    public class APISearchApplication {
	       public static void main(String[] args) {
		          SpringApplication.run(APISearchApplication.class, args);
	       }
    }
    ```
#### JDBC와 Mybatis의 의존성
- Spring boot 초기설정에서 JDBC와 Mybatis를 둘다 추가하여 작성된 pom.xml은 다음과 같다.

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>1.3.1</version>
</dependency>
```
- 하지만 Mybatis모듈도 결국은 jdbc모듈에 의존하는 형태일 것이기 때문에 다로 Jdbc모듈을 추가하지 않아도 될것이라고 생각했다.
- [Maven repository](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/1.3.1)에 찾아보니 `mybatis-spring-boot-starter` 모듈은 `org.springframework.boot » spring-boot-starter-jdbc`모듈에 의존성이 있어서 굳이 두개의 모듈을 따로 추가해줄 필요는 없어보인다.



#### 참고링크

[Spring Boot - 프로젝트 구성하기](http://opennote46.tistory.com/124)   

[Configuration using annotation @SpringBootApplication](https://stackoverflow.com/questions/33619532/configuration-using-annotation-springbootapplication)   

[Why Spring Boot Application class needs to have @Configuration annotation?](https://stackoverflow.com/questions/39247487/why-spring-boot-application-class-needs-to-have-configuration-annotation)


#8
- ComponentScan은 root를 따로 설정하지 않고 Default로 사용하면 이 어노테이션을 사용한 클래스가 있는 package가 root가 된다.
```
com.apiserver
    |- config
        |- APISearchApplication.java//@ComponentScan
    |- controller
        |- NewsController.java //@Controller
```
- 문제가 방생한 코드의 디렉토리 구조는 위와 같이 되있었는데 ComponentScan이 동작할 때 config 디렉토리가 root가 된다.
- 하지만 controller 디렉토리는 config 디렉토리의 하위 패키지가 아니므로 Scan되지 않았다.

#### 해결방법
- `APISearchApplication`을 root 디렉토리로 옮겨주었다.
```
com.apiserver
    |- APISearchApplication.java//@ComponentScan
    |- controller
        |- NewsController.java //@Controller
```

#11
- JAXB를 이용해서 java Object에 XML을 mapping(unmarshal)시킬때 JAXB가 Object에 접근하는 정도를 조절할 수 있는데 이때 @XmlAccessorType을 사용한다.
- @XmlAccessorType과 @XmlElement, @XmlAttribute를 Object에 적절히 매칭시켜서 xml이 Object로 매핑되게해야한다.
- 위의 어노테이션을 잘못사용하여 매핑하는데 문제가 생기는 경우 `JAXBException`이 발생한다.
- XmlAccessorType은 크게 4개의 속성을 사용할 수 있다.
> 1. PUBLIC_MEMBER
> 2. FIELD
> 3. PROPERTY
> 4. NONE

**1. PUBLIC_MEMBER**
- 이 속성을 사용할 경우 JAXB가 접근하는 정보들은 다음과 같다.
    - public fields
    - annotated fields
    - properties
- Domain객체를 구현할 때, WebContainer내부에서 사용하는 일반적인 Domain처럼 field는 private으로 선언하고 이 필드를 mutate할 수 있도록 public getter/setter를 구현하였다.

- 그렇기 때문에 PUBLIC_MEMBER를 사용한다고 할지라도 public field는 존재하지 않기 때문에 고려되지 않는다.

- PUBLIC_MEMBER를 사용했을 때 발생한 문제는 이미 getter/setter를 모든 필드에 구현해놓은 상태에서 필드에
@XmlElement를 선언하는 경우 발생한다.
```
@XmlRootElement(name = "item")
@XmlAccessorType(XmlAccessType.PUBLIC_MEMBER)
public static class Item {

        @XmlElement
	private String title;
	private String originallink;
	private String link;
	private String description;
	private Date pubDate;

	public String getTitle() {
	    return title;
	}

	public void setTitle(String title) {
	    this.title = title;
	}

  ...
```
> 위 코드는 프로젝트를 구현하면서 실제로 사용한 클래스의 일부인데 AccessType을 PUBLIC_MEMBER로 설정한 상태에서 필드자체에 @XmlElement를 추가한 경우, JAXBException이 발생한다.
```
com.sun.xml.internal.bind.v2.runtime.IllegalAnnotationsException: 1 counts of IllegalAnnotationExceptions
클래스에 동일한 이름 "title"을(를) 사용하는 속성이 두 개 있습니다.
```
- annotated fields인 `private String title`과 property로 설정한 `public void setTitle(String title)`두 요소가 모두 xml의 `title`태그를 매핑하려하기 때문에 발생한 Exception이다.

- PUBLIC_MEMPER를 사용하고 property를 생성한 경우에는 필드에 별도로 어노테이션을 선언할 필요가없다.

**2. FIELD**
- 이 속성을 사용할 경우 JAXB가 접근하는 정보들은 다음과 같다.
    - fields
    - annotated properties
- FIELD 속성은 fields의 AccessModifier의 정도에 상관없이 해당 필드와 매핑한다.
- 즉, private으로 선언된 필드인 경우에도 이 필드에 data를 매핑시켜주는데 여기서 중요한점은 getter/setter 메소드를 사용하지 않고 자체적으로 값을 주입한다.(Reflection을 사용한 것으로 보인다.)
```
@XmlRootElement(name = "item")
@XmlAccessorType(XmlAccessType.FIELD)
public static class Item {

	private String title;
	private String originallink;
	private String link;
	private String description;
	private Date pubDate;

	public String getTitle() {
	    return title;
	}

	public void setTitle(String title) {
      System.out.println("setter method Invoked!");
	    this.title = title;
	}

  ...
```
- 위와 같은 형태의 클래스에 JAXB로 매핑을 하는 과정을 Test한결과 "setter method Invoked!"는 출력되지 않았지만 title변수에 값은 매핑되었다.

- FIELD 속성의 경우, WebContainer의 특성상 getter/setter메소드를 이용해 데이터를 주고받는 경우가 많은데 이를 만들어두고도 사용하지 않기 때문에 부적절하다고 판단했다.

**3. PROPERTY**
- 이 속성을 사용할 경우 JAXB가 접근하는 정보들은 다음과 같다.
    - annotated fields
    - properties
- 기본적으로 getter/setter메소드를 이용한 프로퍼티들에만 접근하여 매핑을하고 추가적인 필드는 어노테이션을 통해서 매핑을 할 수 있다.
- 제일 적절한 속성이라고 생각해서 이 속성으로 구현하였다.

**4. NONE**
- 이 속성을 사용할 경우 JAXB가 접근하는 정보들은 다음과 같다.
    - annotated fields
    - annotated properties
- 이 속성은 오로지 어노테이션에만 의존하여 매핑을 한다.
- 실제 Object가 하는 일과 JAXB와 연동하는 부분이 완전히 분리되는 장점이 있다.
- 하지만 매핑하고자하는 변수들이 많아지는 경우 모든 변수들에 어노테이션을 추가해야하는 불편함이 있으므로 변수들이 많아지는 경우 이속성을 사용하기 까다롭다.

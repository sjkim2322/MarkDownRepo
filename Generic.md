### Generic
> 선언부에 Type Parameter가 포함된 클래스나 인터페이스를 Generic 클래스/인터페이스라고한다.  
> 어떤 클래스가 들어있는지 모르는 컬렉션에서 객체를 하나 가져오면 적절한 형변환을 필요로한다.    
> List\<String\>과같은 자료형을 Generic Type이라고 한다.   
> \< \> 안의 "String"은 actual type parameter라고한다.  

#### Raw Type
> Java의 Generic은 1.5버전 부터 지원하기 시작했다.   
> Java Collection Framework은 Generic을 잘활용하고 있는데 Generic이 도입 되기 전부터 Collection은 존재하고 있었고   
> 그 당시에는 Generic Type을 사용하지 않고 Collection을 사용하였다.  
```
//before 1.5 ver
public List mylist = ...;
``` 
List에 사용되는 공식적인 자료형을 선언시에 알려주지 않기 때문에 모든 Object를 list에 넣을 수 있었고,   
element들을 꺼낼 때는 Casting을 통해서 원하는 자료형에 다시 넣는 과정으로 사용했다.   
하지만 ClassCastException의 경우 컴파일타임에 잡기가 어려운 exception이고 이 것이 발생했을때 그 원인을 찾는 것 또한
많은 시간이 소요되었다.   
위와 같은 RawType을 사용했을 때 발생하는 문제가 Generic이 Java에 적용되게된 핵심적인 이유지만   
Generic이 적용된 이후로도 raw type을 사용하는 방법은 여전히 사용할 수 있다.   
그 이유는 Migration Compatibility(이전 호환성)때문이다.   
Generic이 도입되기 전에 작성된 코드들이 1.5이후의 버전에서 컴파일이 안되서는 안되기 때문이다.   
이전 버전에 작성된 코드와의 호환성을 위해서 아직 Raw Type을 지원한다는 것만 알고 새롭게 코드를 작성할 때는 RawType을 사용하는 것을 지양해야한다.   

#### \<Object\>   

Raw Type을 사용하지 않으면서 아무 Object나 넣을 수 있는 방법은 ```List<Obejct>```처럼 아무객체를 사용할 수 있다는 것을 컴파일러에게 알려주는 방법이 있다.   

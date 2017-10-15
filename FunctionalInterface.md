### Collection Framework
> Java의 Collection Framwork(JCF)은 클래스들의 모음을 관리하는 클래스들을 표준화한 Framework이다.   
> 1.5버전에서 Generic이 JCF에 적용되면서 이제는 JCF는 java를 개발하는데 있어서 아주 중요한 프레임워크가 되었다.   
> 그래서인지 자바의 버전이 올라감에 따라 새롭게 추가되는 스펙들은 거의 모두 JCF에 반영되었다.   
> 8버전에서 Stream과 Lambda 가 대표적인데 모두 JCF에도 적용되었다.   
> Strem은 Collection만 지원하는 것은 아니지만 기존의 iterator기반으로 작성되었던 Collection 연산들이 Stream을 이용해 짧은 작성만으로도 iterator 기반과 같은 기능을 구현할 수 있도록 추상화되었다.   

### Stream
[씹고 뜯고 맛보고 즐기는 스트림 API](https://www.slideshare.net/arawnkr/api-42494051)

> Collection은 클래스들의 모음을 관리한다.   
그러한 모음들이 중복을 허용하는지, 순서를 갖는지, 자료를 저장하는 형태가 무엇인지 등에 따라 세부적으로 구체화된 여러 하위 클래스들을 가지고 있다.   
> 어쩌면 `JCF는 Collection을 저장하는 방식에 초점이 맞추어져 있다고 보는 것이 맞는 것 같다.`   
> Collection에 클래스들을 모아놓고 Collection에 대한 연산을 통해 어떠한 처리를 하는 과정은 개발자의 각각 다른 구현방식에 따라 달라졌다.   
> Collection이 가지고 있는 데이터양이 많아질 수록 이러한 Collection을 이용한 연산하는 로직은 많은 시간을 투자해야하는 부분이었다.   

- Stream은 기존의 `Iterator`를 기반으로하는 순차적인 연산과 달리 `Spliterator`를 이용한 병렬 처리를 지원한다.    
- Collection이 많은 데이터를 포함하고 있을 경우 `for(Element e : Collection c)`처럼 Iterator로 변환한 이후 순차적으로 모든 Element들에 대한 탐색과정을 거친다.    
- Stream의 경우 내부적인 동작방식을 완벽하게 이해하지는 못했지만 `stream()`메소드가 `Spliterator`라는 클래스를 이용하는것으로 보아 많은 Element들을 여러개의 조각으로 나눈뒤에 각각 처리 후 다시 합치는 과정을 통해 보다 빠르게 연산을 진행하는 것 같다.

```
//User(id, name, grade)
List<User> userList = new ArrayList<>();
userList.add(new User(1,"kim",1));
userList.add(new User(2,"lee",1));
userList.add(new User(3,"kim",2));
userList.add(new User(4,"lee",2));
userList.add(new User(5,"kang",1));
userList.add(new User(6,"kang",1));

//list with name "kim"
List<User> kimList;
kimList = userList.parallelStream().filter(u-> u.getName().equals("kim")) // A Line
        .collect(()-> new ArrayListWrapper<>()
                ,(left,user)-> {
                    left.add(user);
                    System.out.println("Expect Twice");
                }
                ,(left,right)-> {
                    right.addAll(left);
                    System.out.println("combiner");
                });
```
위의 코드는 User클래스를 담고 있는 List를 Stream을 이용해 name이 "kim"인 user만을 모아서 다시 List로 Return해주는 코드이다.   
- userList를 Stream화 시키는 A Line에 그냥 `stream()`이 아니고 `parallelStream()`이기 떄문에 Spliterator에 의해 병렬로 처리될 것이다.   
- 중간 연산인 `filter()`를 이용해 name이 "kim"인 element만 filtering하였다.
- 이후 `collect()`를 이용해서 최종 연산을 진행했다.   
- 이름이 "kim"인 element들을 찾은 이후에는 단순히 새로운 List를 만들어서 그안에 각각의 element들을 `add(e)`하면 될 것이라고 생각했는데 collect함수의 파라미터는 총 3개나 된다.   
```
<R> R collect(Supplier<R> supplier,
              BiConsumer<R, ? super T> accumulator,
              BiConsumer<R, R> combiner);
```
위의 코드는 Stram 클래스에 선언된 collect 함수이다.   
- 파리미터로, supplier, accumulator, combiner를 갖는다.   
- supplier는 collect된 결괴(Result)를 담을 새로운 공급자(R)이다.
- accumulater는 공급자와 element를 파리미터로 받고(R,T) element를 공급자에게 어떤식으로 처리할 것인지를 정의한다.
- 두개의 공급자를 조합하여 새로운 한개의 공급자를 만들어낸다.

> 예상

최종결과는 1개의 공급자를 갖고 있지만 3번째 파라미터에서 2개의 공급자를 파리미터를 받기 때문에 Collect연산의 기반은 Collection안의 element들을 세부적으로 쪼개어서 각각 병렬로 처리한 이후에 다시 합치는 방식으로 동작할 것이라고 예상하였다.   

> Test

내 예상이 맞다면 첫번째 파라미터에서 생성하는 공급자의 경우는 Collection이 fiter를 거처서 총 2개의 element가 선택되고 각각 분할되어 있기때문에 2개 생성이 될 것이고, 두번째 accumulater에서는 총 2개이므로 "Expect twice"라는 출력은 2번 출력될 것이고 3번째 파라미터의 "combiner"는 분할된 요소들을 오른쪽에서 부터 왼쪽으로 합치는 작업을 하기 떄문에 첫번째 파라미터에서 생성된 공급자 수 - 1 만큼 출력이 될 것이라고 생각했다.   

- 첫번째 파라미터에서 공급자인 ArrayList가 생성될 때 새로운 Instance가 만들어진 것을 확인하기 위해 다음과 같이 기존의 ArrayList를 extends하여 생성자가 실행될 때 log를 찍도록 했다.
```
public class ArrayListWrapper<E> extends ArrayList<E> {
    public ArrayListWrapper() {
        super();
        System.out.println("instance created!!!");
    }
}
```
- 위의 코드의 실행 결과는 다음과 같다.   
```
instance created!!!
instance created!!!
instance created!!!
Expect Twice
instance created!!!
instance created!!!
instance created!!!
Expect Twice
combiner
combiner
combiner
combiner
combiner
[User{id=1, name='kim', grade=1}, User{id=3, name='kim', grade=2}]
```

> 평가

- `parallelStream()`을 사용하였기 때문에 총 6개의 List가 여러개의 스레드에서 조금씩 할당되어 각각 처리가 될것이라고 생각했는데 생성된 공급자의 Instance가 최초의 Element개수와 같은 것으로 보아 parallelStream()의 분할단위는 1개가 될때 까지 진행이 되는 것 같다.
- 하지만 이러한 점은 만약 1만개의 Element를 가지고 있는 List에 대해서 parallelStream()을 적용한다면 collect하는 단계에서 총 1만개의 빈 Instance가 생성될 것이다. 생성된 이후 전혀 사용되지 않는 instance들이 대부분일 수도 있고 사용되었다고 하더라도 3번째 파라미터의 combiner를 통해서 최종적으로 한개의 instance로 합쳐져 나머지 Instance들은 참조를 잃고 GC의 대상이 될것이다.
- 즉, paralleStream()의 경우에는 필요이상의 OverHead가 발생할 수 있다는 것을 생각해두어야한다.    
[관련 링크](https://stackoverflow.com/questions/23170832/java-8s-streams-why-parallel-stream-is-slower)
- `paralleStream()`부분을 `stream()`으로 수정한다면 결과는 다음과 같다.
```
instance created!!!
Expect Twice
Expect Twice
[User{id=1, name='kim', grade=1}, User{id=3, name='kim', grade=2}]
```
- 공급자로 생성되는 instance가 1개뿐이므로 3번째 파라미터에서 Combiner가 합칠 요소가 없어서 실행조차 되지 않는다.

- 공급자 역할을 하는 Instance가 총 2개만 생성될 것이라고 생각했는데 stream이 생성되기 전에 총 6개의 Element개수 만큼 생성되었다.
- 즉, `filter()`와 같은 중간연산을 하는 부분은 그 위치에서 처리되지 않고 최종연산을 할때 일괄적으로 처리된다. 이를 통해 stream을 통한 연산이 `Lazy Evaluation`된다는 것을 확인할 수 있었다.
- combiner의 경우 공급자로 생성된 instance들을 맨끝에서 부터 앞쪽으로 한개씩 addAll하므로 생성된 instance 개수 보다 1개적은만큼 실행되었다.



Predicate
Function
ToIntFunction
ToLongFunction
ToDoubleFunction
Comparator
Consumer
BinaryOperator
BiFunction
BiConsumer

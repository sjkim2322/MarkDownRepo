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
- 공급자로 생성된 instance가 총 6개가 생성되었는데 그렇다면 최초의 List에서 가지고 있는 element의 개수만큼 Thread가 생성되어 6개의 Thread가 각각 1개의 instance를 만든 것인지를 확인하기 위해 다음과 같이 코드를 수정하였다.
```
public class ArrayListWrapper<E> extends ArrayList<E> {

	public ArrayListWrapper() {
		super();
		String threadName = Thread.currentThread().getName();
		System.out.println("instance created By " + threadName + "thread");
	}

}
```
공급자가 생성될 때 이를 생성한 Thread의 이름을 확인하고자 하였다.
실행결과는 다음과 같다.
```
instance created By ForkJoinPool.commonPool-worker-2thread
instance created By ForkJoinPool.commonPool-worker-1thread
instance created By ForkJoinPool.commonPool-worker-2thread
instance created By mainthread
Expect Twice
instance created By ForkJoinPool.commonPool-worker-3thread
combiner
instance created By ForkJoinPool.commonPool-worker-1thread
Expect Twice
combiner
combiner
combiner
combiner
[User [id=1, name=kim, grade=1], User [id=3, name=kim, grade=2]]
```
생성된 instance의 개수는 6개이고 이를 생성한 Thread들의 개수는 mainthread를 포함하여 총 4개이다. 4개는 이 프로그램을 실행한 컴퓨터의 논리프로세서의 개수인데 paralleStream은 Runtime환경의 논리 프로세서의 개수만큼 Thread를 할당한다.   
```
The parallel streams use the default ForkJoinPool
which by default has one less threads as you have processors,
as returned by Runtime.getRuntime().availableProcessors()
(This means that parallel streams use all your processors because they also use the main thread):
```
[관련링크](https://stackoverflow.com/questions/21163108/custom-thread-pool-in-java-8-parallel-stream)


#### Functional interface
> Collection처럼 여러 Element들의 모음이 있는 자료구조를 통해 원하는 연산을 하는 경우가 있다.   
> 특정 조건에 맞는 element만을 취하는 연산이나 element를 다른 타입의 element로 매핑하는 연산, 특성 property를 기준으로 정렬하는 연산 등이 있다.
> 위의 코드 처럼 UserList에서 name이 "kim"인 element만 취하는 연산을 생각해보면 특정 조건에 맞는 element만을 필터링하는 연산인 것을 알 수 있다.
> Functional Interface는 이처럼 어떠한 연산을 하는데 있어서 연산에 대한 행위에 대한 추상화를 제공한다.   

UserList에서
- name이 "kim" 인 Element만 고른다.
- grade가 1보다 큰 Element만 고른다.   

위 처럼 특정 조건이 맞는 경우에만 element를 취하는 연산들을 코드로 짜보면 아래와 같은 형태가 될 것이다.
```
if(user.getName().equals("kim")) {
  result.add(user);
}

if(user.getGrade() > 1) {
  result.add(user);
}
```
위의 코드를 추상화 시켜보면 다음과 같다.   
`어떤 조건이 True일 경우 그 Element를 고른다.`
Stream API는 위와 같은 연산을 위해 filter메소드를 제공하는데 Stream 입장에서는 사용자가 작성해야하는 `조건`에는 신경쓰지 않고 사용자로 부터 받은 어떤 조건이 `True`일 경우 그 Element를 고르는 형태로 추상화 되어 있다.   
즉, Functional Interface는 어떤 연산에 있어서 그 행위만을 정의해 놓은 Interface이다. Stream은 사용자로 부터 전달받은 함수를 실행하여 그값이 True인 경우에만 특정 작업을 진행할 수 있다.    
`java.util.function`에는 특정 input과 output을 갖고 자주사용되는 순수함수들의 Interface를 제공하고 있다.   
Stream은 이러한 Interface들 중 Stream이 연산을 하는데 필요한 interface들을 파라미터로 받아 연산을 처리한다.   
Stream을 사용하는 사용자 입장에서는 최소한의 행위의 규약(Input과 output)만 정해진 함수형 인터페이스에 대한 구현체를 Stream의 파라미터로 넘겨주면 그에 대한 결과를 받을 수 있다. 함수형 인터페이스를 구현하는 일은 익명클래스를 사용하여 구현하지만 java8에서는 함수를 하나만 갖는 인터페이스의 경우 람다식을 사용할 수 있으므로 매우 간결하게 구현체를 작성할 수 있다.    
이는 마치 C언어의 함수포인터처럼 보이고 실제로도 그와 비슷하게 동작하지만 내부적으로는 함수형인터페이스의 구현체가 파라미터로 전달되는 형태로 보인다.   
Stream이 중간연산과 최종연산에서 사용하는 메소드들이 파라미터로 사용하는 함수형인터페이스들이 무엇인지 살펴보았다.

#### filter(Predicate<? super T> predicate);
- filter 메소드는 사용자로부터 Predicate 인터페이스의 구현체를 받아 구현체 안의 사용자가 Override한 `test()` 메소드를 실행시켜 그값이 True인경우 그 Element를 취한다.(stream은 Lazy Evaluation 되므로 filter메소드가 바로 그 element를 취하는 것은 아니고 위의 처리를 할 수 있는 작업이 추가된 새로운 Stream을 return한다.)
- 위의 코드에서 filter를 사용한 코드는 다음과 같다.
```
filter(u-> u.getName().equals("kim"))
```
람다식을 사용했는데 이를 풀어본다면 다음과 같은 익명클래스 생성코드가 될 것이다.
```
filter(new Predicate<User>() {
  @Override
  public boolean test(User u) {
    return u.getName().equals("kim");
  }
  })
```
하지만 위의 두코드가 완벽히 같은 형태로 컴파일 되는 것은 아닌 것 같다.   
[람다 표현식의 내부 구현](http://d2.naver.com/helloworld/4911107)에 의하면
람다로 작성한 코드는 익명클래스로 작성한 코드와 달라 this를 참조하는 부분에 있어서 차이가 있다고 한다.
- Stream 입장에서는 filter메소드를 통해 클라이언트로부터 Prdicate 객체를 받아 그 객체의 `test()`함수의 파라미터로 각각의 Element(여기서는 User)를 넣어 실행결과가 True인 경우에만 추려낸다.   
- 클라이언트 입장에서는 Predicate의 구현체를 filter의 파라미터로 전달해주기만 하면된다.   

Predicate의 구현체를 전달하는 방법은 크게 3가지가 있다.   
Predicate 뿐 아니라 일반적인 함수형 인터페이스를 구현하는 방법은 위 3가지에서 크게 벗어나지 않는 것 같다.

> **1.익명클래스 구현**   
> **2.Lambda Expression**   
> **3.Method Reference**   

**1. 익명클래스 구현**
- 구현하고자하는 인터페이스를 익명클래스 형태로 구현할 수 있다.
```
Predicate<T> predicate = new Predicate<T>() {
  @Override
  public boolean test(T t) {
    return  "conditional statement"
  }
}
```
위 코드처럼 Predicate를 구현한 구현체를 filter의 파라미터로 넘겨주면 filter는 각각의 Element T를 test메소드에 넣어 실행시킨 결과를 바탕으로 필터링한다.

**2. Lambda Expression**
- 1번의 익명클래스를 구현하는 방법과 거의 유사한 방식으로 동작하지만 람다식을 활용하면 익명클래스의 구현을 매우깔끔하게 작성할 수 있다.
- 1번의 익명클래스를 람다식으로 바꾸면 다음과 같다.
```
Predicate<T> predicate = t -> "conditional statement" ;
```
Predicate의 경우에는 return값이 boolean 으로 고정되어 있으므로 boolean 값을 return하도록 구현하면된다. 파라미터로는 각각의 element가 전달된다.  

**Method Reference**
[참고 링크](http://www.baeldung.com/java-8-double-colon-operator)
- Lambda와 익명클래스는 함수형 인터페이스가 요구하는 파라미터와 리턴값이 일치하는 구현체를 새로만든다.   
- 이 경우 구현체를 구현하는 과정에서 이미있는 메소드를 1번만 호출해서 값을 얻는 경우에는 람다식으로 구현하더라도 불필요한 코드가 많아지게 된다. 예를들면, 어떤 객체가 Null인지를 확인해서 Null인 객체만 True를 반환하는 람다식은 다음과 같을 것이다.
```
Predicate<T> predicate = t -> Objects.isNull(t);
```
- `isNull()`메소드는 Objects 클래스의 Static Method로 파라미터로 받은 객체가 Null인지를 boolean 값으로 return 한다.   
- 즉 이 함수는 Predicate<T> 인터페이스가 요구하는 signature와 동일하다.
- 이러한 경우 `::`을 이용한 메소드 참조를 통해 이 메소드에 대한 함수형 인터페이스의 구현을 다음과 같이 구현할 수 있다.
```
Predicate<T> predicate = Objects::isNull;
```
- 메소드 참조를 통한 함수형인터페이스 구현은 실제로 method에 대한 참조를 가지고 있지는 않는다.
- 하지만 코드를 작성할대는 마치 메소드의 참조가 전달되는 것과 같은 형태로 구현을 할 수 있다.
- java에서 메소드 참조를 사용할 수 있는 메소드는 다음 조건 중 1개를 만족해야한다.
- 다음조건을 만족한다면 컴파일러에 의해서 적절히 변경되는 것으로 이해하였다.
> 1. Static Method

- Static method의 경우에는 instance가 없이 ClassName으로 참조가 가능하므로 `ClassName::methodName` 형태로 작성하면
함수형 인터페이스 내부의 메소드의 파라미터가 그대로 method의 파라미터로 전달된다.
Predicate의 경우 <T> 값을 파라미터로 전달해주게되고 이는 곧 다음과 같다.
`ClassName.methodName(t)`
위의 `isNull` 메소드의 경우를 람다식으로 표현하면 다음과 같을 것이다.   
`t -> Objects.isNull(t)`   
> 2. An Instance Method of an Existing Object

- Static method가 아니더라도 instance를 넘겨준다면 instance 메소드도 참조형으로 사용할 수 있다.
즉, `instanceName::methodName(t)`를 만족한다면 Static method와 동일하게 동작한다.
> 3. An Instance Method of an Arbitary Object of a Particular Type

- 이 경우는 instance 메소드이기는 하지만
> 4. A Super Method of a Particular Object
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

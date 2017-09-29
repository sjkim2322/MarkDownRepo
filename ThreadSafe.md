## Thread Safe
Thread Safe라는 용어는 Method에도 사용되고 Class에도 사용된다.   
어떤 Class가 Thread Safe하다는 것은 그 클래스 내부의 모든 함수들과 변수에 대한 연산들이 Thread Safe하는 경우를 의미한다.   

#### Thread Safe의 전제 조건
Thread safe/non-safe를 생각하기 전에 다음과 같은 조건을 만족하지 않는다면 반드시 Safe하다고 판단해도 좋다.
> 1. 일련의 처리를 하는 과정(함수내부일 수도 있고 단순 연산자일 수도 있다.)에 있어서 현재 Thread뿐만 아니라 다른 Thread에서도 공통적으로 접근할 수 있는 SharedData가 사용된다.   

> 2. 1을 만족할 때 SharedData는 Mutable한 상태이고 처리과정의 결과는 해당 data의 상태를 변화시킨다.   

#### String
위의 조건을 바탕으로 String Class가 Thread Safe인지를 판단해보았다.
```
public final class String implements ...{
    private final char value[];
    ...

    public char charAt(int index) {
      ...
      return value[index];
    }
}
```
String Class는 기본적으로 char형 배열인 value라는 변수를 가지고 있다.   
`charAt`Method도 value변수를 return하면서 사용하고 있다.   
이 Method 뿐만 아니라 내부의 여러 Method들은 value라는 변수를 사용하고 있다.   
그리고 이 Class를 이용하여 생성된 1개의 Instance는 여러 Thread에서 접근하여 method의 실행이나 연산자를 통해 value라는 변수를 공통적으로 사용할 수 있다.
`즉, String Class는 Thread Safe의 전제조건 중에서 1을 만족한다.`   
하지만 value라는 SharedData를 담고있는 이 변수는 final로 선언되었기 때문에 Mutable한 상태가 아니다.   
String 연산에서 사용하는 '+'연산자 또한 value변수에 data를 추가하는 것이 아니라 기존의 문자열에 새로운 문자열을 추가한 새로운 instance를 만들고 그 Instance를 return 해준다.
결과적으로 String Class의 value변수는 Immutable하기 때문에 Thread Safe의 전제조건을 만족하지 못하기 때문에 `String Class는 Thread Safe하다고 할 수 있다.`   


#### Method의 Thread Safe
Method입장에서 Thread Safe란 위의 전제 조건을 만족하지 않거나,
전제조건을 만족하면서도 여러 Thread에서 해당 Method의 수행을 마친 결과를 예측할 수 있는 경우를 말한다.
`여기서 결과를 예측한다는 것은 각Thread의 실행순서에따른 결과를 예측하는 것이 아니고 순서와 상관없는 결과를 예측할 수 있다는 것이다.   
예를 들어 문자열을 담는 변수에 1번Thread에서 문자"1"을 총 10번, 2번Thread에서 문자"2"를 총 10번 쓰는 작업을 진행하면 변수의 값은 예측할 수 없지만 변수의 길이가 20이라는 것과 변수 내부에서 "1"과 "2"의 개수가 각각 10개라는것은 예측할 수 있다.`    
2가지의 전제조건을 모두 만족하는 Multi Thread환경에서는 SharedData에 대한 Race Condition이 생기게 된다.  
전제조건중에서 1번만 만족해도 Race Condition은 생기게 되지만 sharedData가 Immutable하거나, Mutable하지만 값을 수정하는 작업을 하지 않는 Method라면 Race Condition이 Thread Non-Safe로 이어지지 않는다.   
즉, Mutable한 data라도 쓰기 연산이 없다면 그 결과는 예측가능하다.   

#### StringBuilder 와 StringBuffer
Race Condition이 Thread Non-Safe로 이어지는 경우를 잘보여주는 StringBuilder클래스가 있다.   
```
abstract class AbstractStringBuilder implements ... {
    char[] value;
    ...
```
위 코드는 StringBuilder클래스가 상속하고 있는 StringBuilder의 추상클래스이다.   
이 클래스 또한 char형 배열인 value변수를 갖는다는 점에서는 String Class와 유사하지만 이 변수가 final이 아니라는 점 즉, Mutable하다는 큰 차이가 있다.
StringBuilder는 +를 통해서 문자열을 합친 새로운 Instance를 생성하는 String Class와 달리, 자체적으로 Capacity를 가지고 value변수에 길이를 늘리면서 값을 직접 수정한다. String Class의 +연산에 대응되는 Method로는 `append` Method가 있다.   
```
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count); // A
    count += len; // B
    return this;
}
```   
위 코드는 StringBuilder가 사용하는 append함수의 구현부이다.   
A라인의 `getChars`Method를 통해 str변수에 있는 값을 value변수의 뒷부분에 추가하고
B라인에서 StringBuilder가 가지고있는 Count값을 추가한 str의 길이만큼늘려준다.   
즉, MultiThread환경에서 append함수는 Mutable한 count값과 value값을 공유하는 상태에 놓인다.   
count값과 value값에 대한 Race Condition이 발생하게 되고 1번Thread와 2번Thread가 동시(서로 Switching하면서)에 실행될 때 1번Thread의 A 2번Thread의 A\`,1번 Thread 의 B,2번 Thread의 B\`순서로 실행된다면 결과를 예측할 수 없게 된다.
정리하자면 `일련의 처리흐름이 한단위로 진행되어야하는 원자성을 보장받아야하는 상황에서 Race Condition에 의해 그 원자성이 깨져버리면 그 결과를 예측할 수 없게 되고 Thread Non-Safe한 상태가 된다.`   
Race Condition이 발생했을때 원자성을 보장해주는 가장 간단한 방법은 `synchronized`키워드를 사용해 Race Condition을 회피하는 것이다.   
StringBuffer 클래스는 StringBuilder와 같은 추상클래스를 상속받아 구현된 클래스이다. 하지만 추상클래스의 모든 함수들을 Overriding할 때 synchronized키워드를 사용해서 SharedData에 대한 접근과 사용에 순서를 보장한다.
```
public final class StringBuffer extends AbstractStringBuilder implements ...
  ...

  @Override
      public synchronized StringBuffer append(String str) {
        ...
```

#### Testing
[Unit Test 코드]()   
1.String Test   

String의 +연산이 Thread Safe한지 Test하기 위해 다음과 같은 class를 작성하였다.   
```
public class MyThreadUsingString extends Thread{

	static String string;
	final String input;
	static final int expectValueCount = 50;
	public MyThreadUsingString(String input) {
		this.input = input;
	}
	@Override
	public void run() {
		for(int i =0; i<expectValueCount;i++) {
			 string+= input; // A
		}
	}
}
```
Test는 2개의 서로다른 Thread에서 1번Thread에서는 "1"을 2번Thread 에서는"2"값을 static으로 가지고 있는 string 변수에 각각 50번씩 + 한 후에 최종적으로 반환된 값에 "1"의 개수가 50개, "2"의 개수가 50개, 총길이기 100인지를 판단하는 방법으로 진행하였다.  
결과는 Thread Non-Safe였는데 String은 Thread Safe라고 예상햇던것과는 다른이유를 찾아보았다.   
이유는 A라인에 있었는데 A라인은 '2개의 문자열을 합친 새로운 Instance 생성' 과 '생성된 Instance의 참조값을 string 변수에 할당' 2개의 작업이 이루어진다.
즉, String Class 자체는 Immutable하기 때문에 Thread Safe하지만 새로 생성된 instance의 참조값을 SharedData에 지속적으로 변경하는 작업은 또다른 의미의 Race Condition이 발생하게 되어 Test가 실패했다.

2.StringBuilder, StringBuffer Test   
StringBuilder의 경우 2개의 Thread에서 append함수를 여러번 실행시킨뒤 결과값의 길이와 각각 append한 data의 개수가 예측가능한지 test하였다.   
결과는 Non Safe 였다.   
다음과 같은 2가지 이유로 Test가 실패하였다.
1. capacity가 꽉차서 Capacity를 늘리는 작업이 실행되기전에 다른 Thread가 값을 추가하게 되는 경우 OutOfIndexError가 발생했다.   
2. 값을 append하는 작업과 전체 데이터의 길이를 늘리는 작업 사이에 다른 Thread가 값을 append하게 되면 같은 자리에 다른 Data가 두번 append 되어 최종적으로 append 하고자하는 데이터의일부가 누락되었다.   

StringBuffer의 경우 StringBuilder와 같은 방법으로 Test했을 때    
모두 성공하였다.

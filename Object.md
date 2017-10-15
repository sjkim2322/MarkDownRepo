### Object
Java의 모든 Class는 Object Class를 Extend하는 형태로 구현된다.   
Object 내부에 여러 메서드들이 있는데 그 중 Extend시에 재정의가 가능한 비Final메서드들이 존재한다.   

#### equals 메서드
Object Class의 equals함수는 기본적으로 객체의 동일성(Object equality)를 판단할 수 있다.   
즉 서로다른 두개의 객체가 같은 주소를 가지고 있는 완벽하게 일치하는 객체일 경우 True를 반환한다.   
Object를 상속하여 만들어진 여러 Class들 중에서는 객체의 동일성 보다는 논리적 동일성(Logical equality)를 판단해야하는 Class가 있다.   
String Class가 대표적인 예인데, String 클래스는 객체의 동일성보다 class가 가지고있는 문자열의 값이 같은 가에 대한 논리적 동일성을 판단하기 위해 equals함수를 Overriding해서 사용하고 있다.  
즉, 객체간 논리적 동일성을 판단해야할 필요가 있다면 Obejct Class의 equals메서드를 재정의 해야한다.   

equals메서드를 재정의 할때 지켜야할 규약은 다음과 같다.
> 1. 반사성
> 2. 대칭성
> 3. 추이성
> 4. 일관성

1.반사성(Reflexivity)    
모든 객체는 자기 자신과는 같아야 한다.   
즉 x.equals(x)는 반드시 True를 리턴해야한다.   
2.대칭성(Symmetry)
x.equals(y)가 True라면 y.equals(x) 도 True여야한다.   
x와 y가 서로 다른 클래스이지만 논리적 동일성을 판단할 수 있는 형태일 때 어기기 쉬운 규약이다.   
각 클래스의 재정의 된 equals함수가 서로의 클래스에 대해 같은 값을 리턴할 수 있어야한다.   
3.추이성(Transitivity)
x.equals(y)가 True이고 y.equals(z)가 True라면 x.equals(z) 또한 True여야한다.   
추이성의 경우 어떤 클래스를 상속한 여러 클래스들 간에서 깨지기 쉽다.   
4.일관성(Consistency)   
x.equals(y)의 값이 일단 정해지면 x와 y가 변경되기 전까지는 그 값이 동일해야한다.   
즉, 신뢰성이 보장되는 자원에 대해서 equals를 구현하는 것이 좋다.   

대칭성과 추이성에 대해서는 다음 코드를 통해 자세히 살펴봤다.
```
class T {
 int a;
 int b;
 ...
 @Override
 public boolean equals(Object o) {
    if(!(o instanceof T)) {
        return false;
    }
    T t = (T) o;
    return t.a == a && t.b == b;
 }
```
위와 같은 T라는 클래스가 있고 이 클래스는 a,b라는 int변수를 갖는다.   
T클래스의 경우 a값과 b값을 비교한 논리적 동일성이 필요하여 equals함수를 위 코드와 같이 재정의했다.   
T 클래스의 eqauls메서드는 반사성, 대칭성, 추이성 모두 만족한다.   
하지만 T클래스를 상속한 다음과 같은 P클래스가 있다면 가정해보자.   
```
class P extends T {
    int c;
    ...
}
```
P 클래스 또한 equals 메소드를 재정의해야하므로 다음과 같은 형식으로 구현해야할 것이다.   
```
@Override
public boolean equals(Object o) {
    if(!(o instanceof P)) {
        return false; //A
    }
    P p = (P) o;
    return super.equals(o) && c == p.c;
}
```
하지만 이런식의 구현은 대칭성의 규약이 깨지게 된다. 
T클래스의 객체 t와 P클래스의 객체 p사이에서 각각의 a,b값이 같다면 t.equals(p)는 True일 것이다.   
하지만 p.equals(t)의 경우 A라인에서 false를 반환하게 되므로 대칭성 규약이 깨진다.   
클래스를 상속하여 또다른 클래스를 만들어도 equals가 올바르게 작동하기 위해서 하위 클래스인 P클래스의 equals메서드를 수정해야한다.   

```
@Override
public boolean equals(Object o) {
    if(!(o instanceof T)) {
        return false;
    } else if(!(o instanceof P)){ // o is T
        return o.equals(this);
    } else { // o is P
        return super.equals(o) && (P) o.c == c;
    }
}
```
위 코드처럼 하위클래스.equals(상위클래스) 의 형태의 경우 상위클래스의 equals함수만을 실행해 대칭성을 지킬 수 있다.   
하지만 다음의 예를 생각해보자. 각 클래스의 생성자들은 a,b,c순서로 값을 할당한다고 하자.   
```
p1 = new P(1,2,3);
t = new T(1,2);
p2 = new P(1,2,5);
```
p1.equals(t)의 경우 T클래스의 equals메서드를 통해 True라는 값을 얻을 수 있다.
t.equals(p2)의 경우에도 a,b값이 같기 때문에 True이다.   
추이성의 규약을 지킨다면 이런 경우, p1.equals(p2)는 True여야하지만    
추이성의 규약이 깨져 False값을 return하게 되고 실제로도 false이다.   

위의 예제 코드처럼 상속을 통해 클래스를 확장시키는 경우 모든 규약을 만족하는 것이 어렵다.   
instanceof의 사용대신 getClass를 사용하여 부모자식간의 관계를 생각하지 않고 오직 같은 클래스에서만 규약을 만든다면 쉽게 해결될 수도 있다.    
하지만 이러한 구현은 기존의 리스코프 대체 원칙을 따르고 있는 다른 API함수의 사용에 있어서 잘못된 결과를 반환하게 한다.   
즉, 부모클래스를 통해 추상화된 구현부에 자식 클래스가 들어가게 된다면 부모클래스의 equals메소드를 활용한 다른 메소드들은 자식 클래스에 대한 올바른 로직을 수행하지 못한다.   

이를 해결하는 방법으로는 Extends를 통한 상속을 사용하지 않고 자식클래스에 해당하는 P클래스가 T클래스를 포함(Composition)하는 형태로 구현한다면 해결할 수 있다.   
 
 
##### equals메서드를 재정의할 때 따라야할 지침
> 1.반사성을 반드시 만족해야 한다. 즉, 객체를 비교하는 equals 내부의 연산들이 오버헤드가 크다면 "==" 연산을 통해 같은 객체의 경우 바로 True를 리턴하는 것이 좋다.   
> 2.instanceof를 통해 파라미터로 적절한 Type을 받았는지 equals 내부에서 처리해야한다. ClassCastException에 대한 책임을 equals메소드가 지도록하여야한다.   
> 3.타입 체크가 성공한 파라미터는 반드시 정확한 타입으로 Casting한다.
> 4.hashcode도 반드시 재정의한다.   

#### hashcode 메서드
equals메소드를 재정의해서 비록 물리적으로 다른객체라고 할지라도 논리적으로 같다고 판단되는 객체라면 그 객체들의 Hashcode값도 같아야한다.    

#### toString 메서드
Object 클래스에 정의된 toString 메소드는 "className@hascode"와 같은 format으로 값을 return한다.   
그렇기 때문에 여러 property를 포함하는 새로운 클래스를 만드는 경우 그 property들을 다 포함할 수 있는 toString 함수를 재정의하는 것이 바람직하다.   
toString메서드를 재정의 하는 경우 어떤 format으로 재정의하였는지 문서로 남겨야한다.   
그리고 다른 사용자가 이 클래스를 사용할 때 property를 취하는 방법이 toString으로 반환된 문자열을 파싱해서 사용하지 않도록 각 property에 대한 accessor들을 만들어 두어야한다.   
getter/setter.   

#### clone 메서드
Object 클래스의 clone메서드를 재정의하기 위해서는 반드시 Cloneable 인터페이스를 구현해야한다.   
clone메서드는 protected이므로 이를 하위 클래스에서 public으로 재정의하고 외부에서 clone을 호출할 수 있도록 해주어야한다. 
하위클래스의 재정의한 clone메서드 내부에서 super.clone()을 통해 상위 클래스의 clone메소드를 실행시켜야하는데 여기서 Cloneable 인터페이스를 구현하지 않는 클래스라면   
CloneNotSupportedException이 발생한다. super.clone()을 사용하지 않고 생성자를 통해서 객체의 복사본을 만들 수 있다면 굳이 Cloneable을 구현하지 않아도된다.   
즉, 만들고자하는 클래스가 final 클래스여서 더이상 확장될 여지가 없고, 생성자를 통해서 완벽한 복제본을 만들 수 있는 경우에는 super.clone을 사용하지 않아도 된다.   

clone을 재정의하고자하는 클래스 내부에 property가 참조형 변수인 경우 깊은 복사를 이용해서 참조하는 객체들또한 모두 clone되어야한다.   

   

# 🔗 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클래스의 인스턴스를 생성하는 방법은 (1) public 생성자 (2) 정적 팩터리 메서드가 있다.

클래스는 생성자와 별도로 **정적 팩터리 메서드**(static factory method)를 제공할 수 있다.

&nbsp;

java개발을 하다보면 정적 팩터리 메서드를 사용하는 경우들이 있다.  
대표적으로 **Arrays, Collections** 클래스에서 사용하는 메서드들이다.
```java
int[] nums = {1,2,3,4,5};
List<String> list = Arrays.**asList**(nums);
Collections.sort(list);
```

&nbsp;

### 정적 팩터리 메서드란?

static으로 선언된 메서드이며, new Object()와 같이 객체생성을 하지 않고 사용할 수 있는 메서드이다.  
실제로 Boolean객체의 `valueOf()`가 정적 팩터리 메서드인데 내부를 보면 아래와 같다.  
##### 이 메서드는 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해준다.
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

&nbsp;

## 💎 정적 팩터리 메서드가 생성자보다 좋은 이유

### 1. 이름을 가질 수 있다.

> **한 클래스에 생성자가 여러 개 필요하다면, 생성자를 정적 팩터리 메서드로 바꾸고 이름을 지어주자.**

한 객체의 생성자 여러개로 오버로딩하면, 모든 생성자의 이름은 같지만 매개변수만 다르다.  
이러한 경우 **단순히 매개변수만 보고 의미하는 바를 정확하게 알기 힘든 경우**가 많다.  

생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.  
**입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식은 좋지 않은 발상이다.**  
각 생성자가 어떤 역할을 하는지 정확히 기억하기 어렵고, 코드를 읽는 사람도 의미를 알지 못할 것이다.

반면 **정적 팩터리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.**

ex) `BigInteger.probablePrime`: '값이 소수인 `BigInteger`를 반환한다.'

&nbsp;

### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

> **불변 클래스를 만들 수 있다.**

정적 팩터리 메서드 호출을 위해 새로운 인스턴스를 생성하지 않기 때문에, 불변 클래스(immutable class)는 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.  
불변 클래스(Immutable)는 인스턴스를 미리 만들어 놓거나, 새로 생성한 인스턴스를 캐싱하여 재활용해 불필요한 객체 생성을 막아 성능을 끌어올려 준다.  

> - **불변 클래스 (Immutable)**  
: 변경이 불가능한 클래스  
	- 레퍼런스 타입의 객체 (heap 영역에서 생성)  
	- Thread-safe (재할당은 가능. 값 복사 X)   
	Ex. String, Boolean, Integer, Float, Long ↔ (StringBuilder는 가변 클래스)  
##### `Boolean.valueOf()` 메서드는 객체를 생성하지 않고, Boolean 객체 참조를 반환해 준다.
```java
@DisplayName("Boolean is an immutable class")
@Test
void booleanTest() {
	Boolean boolean1 = Boolean.valueOf(true);
	Boolean boolean2 = Boolean.valueOf("true");
	Boolean boolean3 = new Boolean(true);

	assertThat(boolean1).isSameAs(boolean2);
	assertThat(boolean1).isNotSameAs(boolean3);
}
```

&nbsp;

ex) 
```java
class Singleton {
    private static Singleton singleton = null;

    private Singleton() {}

    static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }

        return singleton;
    }
}
```
생성자를 private 으로 선언하여 `new` 키워드를 통한 객체 생성을 막아두고, `getInstance` 메서드를 이용하여 인스턴스를 없다면 생성하고, 있다면 반환하도록 하였다.

```java
public class Main {
    public static void main(String[] args) {
        Singleton a = Singleton.getInstance();
        Singleton b = Singleton.getInstance();
        System.out.println(a == b); // true
    }
}
```

`getInstance`를 해서 비교해보면, 객체가 하나만 생성되었음을 알 수 있다.
	
&nbsp;

#### 이와 비슷한 기법  
> - **플라이웨이트 패턴 (Flyweight pattern)**  
: '공유'를 통하여 다양한 객체들을 효과적으로 지원하는 방법  
	- 객체 내부에서 참조하는 객체를 직접 만들지 않고, 있다면 객체를 공유하고 없다면 만들어 줌 (Pool로 관리)  
	Ex. String Pool (Heap 영역의 permanant 영역에 할당): 같은 내용의 String 객체가 선언된다면 기존의 String 객체를 참조  
→ 데이터를 공유하여 메모리를 절약하는 패턴

#### 인스턴스 통제(instance-controlled) 클래스

이와 같이 반복되는 요청에 같은 객체를 반환하는 정적 팩터리 방식의 클래스는 인스턴스를 통제할 수 있다.  
*인스턴스를 통제하는 이유는?*  
1. 클래스를 싱글턴(singleton)으로 만들 수 있다.  
싱글턴: 인스턴스를 오직 하나만 생성할 수 있는 클래스  
2. 클래스를 인스턴스화 불가(noninstantiable)로 만들 수 있다.
3. 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.  
      `a==b`일 때만 `a.equals(b)` 성립

&nbsp;

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

> **반환할 객체의 클래스를 자유롭게 선택할 수 있는 '유연성'을 부여해준다.**

정적 메서드로부터 인터페이스 자체를 반환하여, 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있다.  
구현 클래스의 상세를 숨길 수 있고, 사용자는 객체를 인터페이스만으로 다룰 수 있다.

API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어, API를 작게 유지할 수 있다.  
이는 인터페이스를 정적 팩토리 메서드의 반환 타입으로 사용하는 **인터페이스 기반 프레임워크**를 만드는 핵심 기술이기도 하다.

이 예로 java.util.Collections가 있다.  
프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 된다.  
나아가 정적 팩토리 메서드를 사용하는 클라이언트는 얻은 객체를 (그 구현 클래스가 아닌) 인터페이스만으로 다루게 된다.

&nbsp;

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

> **반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체도 반환할 수 있다.**  

이를 통해서 사용자는 팩토리 메서드가 반환하는 인스턴스가 어떤 클래스인지 알 수도 없고, 알 필요도 없어진다.  
→ 철저히 인터페이스 기반의 구현이 이루어진다.

&nbsp;

ex)  

```java
public interface MyInt {
    static MyInt of(int v) {
        MyInt instance;

        if (v > 100) {
            instance = new MyBigInt(v);
        } else {
            instance = new MySmallInt(v);
        }

        return instance;
    }

    String getValue();

    class MySmallInt implements MyInt {
        private int value;
        MySmallInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MySmallInt: " + this.value;
        }
    }

    class MyBigInt implements MyInt {
        private int value;

        MyBigInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MyBigInt: " + this.value;
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        MyInt mySmallInt = MyInt.of(10);
        MyInt myBigInt = MyInt.of(200);

        System.out.println(mySmallInt.getValue());
        System.out.println(myBigInt.getValue());
    }
}
```

클라이언트는 이 두 클래스의 존재를 모른다. 만약 정수가 100 이하라면 `MySmallInt` 인스턴스를, 그렇지 않다면 `MyBigInt`인스턴스를 반환한다.  
만약 두 클래스를 사용할 이점이 없어진다면 하나를 삭제 해도 아무 문제가 없다. 새 클래스를 추가할 수도 있다.

클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다. `MyInt`의 하위 클래스이기만 하면 되는 것이다.

&nbsp;

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이런 유연한 점을 이용하여 **서비스 제공자 프레임워크(Service Provider Framework)**를 만드는 근간이 된다.

서비스 제공자 프레임워크는 3가지의 핵심 컴포넌트로 이뤄진다.

1. 서비스 인터페이스(Service Interface): 구현체의 동작을 정의
2. 제공자 등록 API(Provider Registration API): 제공자가 구현체를 등록할 때 사용
3. 서비스 접근 API(Service Access API): 클라이언트가 서비스의 인스턴스를 얻을 때 사용

3개의 핵심 컴포넌트와 더불어 종종 네번째 컴포넌트가 사용되기도 한다.

1. 서비스 제공자 인터페이스(Service Provider Interface): 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명

대표적인 서비스 제공자 프레임워크로는 `JDBC(Java Database Connectivity)`가 있다.  
MySql, OracleDB, MariaDB등 다양한 Database를 JDBC라는 프레임워크로 관리할 수 있다.

&nbsp;

## 💎 정적 팩터리 메서드의 단점

### 1. 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

상속을 하게되면 `super()` 를 호출하며 부모 클래스의 함수들을 호출하게된다. 그러나 부모 클래스의 생성자가 private이라면 상속이 불가능하다.  
보통 정적 팩토리 메서드만 제공하는 경우 생성자를 통한 인스턴스 생성을 막는 경우가 많다.

ex) `Collections`는 생성자가 private으로 구현되어 있기 때문에 상속할 수 없다. 

그러나, 이 제약은 상속보다 컴포지션을 사용하도록 유도하고, 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들이기도 한다.  
(컴포지션: 기존 클래스를 확장하는 대신에 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하는 방식)

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

어떤 라이브러리를 사용하기 위해 API문서를 보면 정적 팩터리 메서드에 대해서 확인하기가 쉽지 않다.

아래 Boolean의 API문서를 보면, 생성자에 대한 내용은 별도로 작성하여 구분이 잘 된다.  
그러나 정적 팩터리 메서드에 대해서는 일반 메서드와 같이 작성이 되어있어 구분하기 쉽지않다.

<img width="1183" alt="스크린샷 2021-01-05 오전 11 40 31" src="https://user-images.githubusercontent.com/45806836/103600471-d4206880-4f4a-11eb-9af3-51d13007c0c9.png">

&nbsp;

## 💎 정적 팩터리 메서드 명명 방식

- **from**: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드  
`Date d = Date.from(instant);`
- **of**: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드  
`Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
- **instance** 혹은 **getInstance**: 매개변수로 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하진 않는다.  
`StackWalker luke = StackWalker.getInstance(options);`
- **create** 혹은 **newInstance**: **instance** 혹은 **getInstance**와 같지만, 매번 새로운 인스턴스 생성을 보장  
`Object newArray = Array.newInstance(classObject, arrayLen);`
- **get*Type***: **getInstance**와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용  
`FileStore fs = Files.getFileStore(path);`  ("***Type***"은 팩터리 메서드가 반환할 객체의 타입)
- **new*Type***: **newInstance**와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용  
`BufferedReader br = Files.newBufferedReader(path);`
- ***type***: **get*Type***과 **new*Type***의 간결한 버전   
`List<Complaint> litany = Collections.list(legacyLitany);`

&nbsp;

## 💎 결론

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.  

그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.

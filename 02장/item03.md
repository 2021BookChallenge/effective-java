# 🔗 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

> **싱글턴 (singleton)**  
: 인스턴스를 오직 하나만 생성할 수 있는 클래스  
ex) 함수(무상태 객체), 설계상 유일해야 하는 시스템 컴포넌트

생성자를 private으로 만들어 new를 통해 밖에서 호출 못하게, static method로 사용 가능.

### 싱글턴의 한계

1. **private 생성자를 가지고 있기 때문에 상속할 수 없다.**  
private 생성자는 오직 싱글톤 클래스 자신만이 자기 오브젝트를 만들도록 제한한다.  
문제는 private 생성자를 가진 클래스는 다른 생성자가 없다면 객체지향의 장점인 상속과 다형성 사용이 불가능하다는 점이다.
2. **싱글턴은 테스트하기가 어렵다.**  
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트 하기가 어려울 수 있다.  
만들어지는 방식이 제한적이기 떄문에 테스트에서 사용될 때 가짜 구현(mock Object)으로 대체하기가 어렵다.
3. **서버환경에서는 싱글턴이 하나만 만들어지는 것을 보장하지 못한다.**  
서버에서 클래스 로더를 어떻게 구성하고 있느냐에 따라서, 혹은 여러 개의 JVM에 분산되어 설치가 되는 경우에 싱글턴 클래스임에도 불구하고 하나 이상의 오브젝트가 만들어질 수 있다.
4. **싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.**  
아무 객체나 자유롭게 접근하고 수정하고 공유할 수 있는 전역 상태를 갖는 것은 객체지향 프로그래밍에서는  권장되지 않는 프로그래밍 모델이다. 대신 static 필드와 메소드로만 구성된 클래스를 사용하는 것을 권장한다.

&nbsp;

### 싱글턴 사용 이유

한번의 객체 생성으로 재 사용이 가능하기 때문에 메모리 낭비를 방지할 수 있다.

또한 싱글톤으로 생성된 객체는 무조건 한번 생성으로 전역성을 띄기에 다른 객체와 공유가 용이하다.

&nbsp;

## 💎 싱글턴을 만드는 방식

싱글턴을 만드는 방식은 두 가지가 있다. 두 방식 모두 생성자는 `private`으로 감춰두고,  
유일한 인스턴스에 접근할 수 있는 수단으로 `public static` 멤버를 하나 마련해 둔다.

### 1. public static 멤버가 final 필드인 방식

##### public static final 필드 방식의 싱글턴

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding()
}
```

private 생성자는 `public static final`필드인 `Elvis.INSTANCE`를 초기화할 때 딱 한 번만 호출된다.  
public이나 protected 생성자가 없으므로 `Elvis`클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다. 

```java
@Test
	public void singletonTest(){
		Elvis elvis1 = Elvis.INSTANCE;
		Elvis elvis2 = Elvis.INSTANCE;

		assertSame(elvis1, elvis2); // SUCCESS 
	}
```
##### 장점
1. 해당 클래스가 싱글턴임이 API에 명백히 드러난다.  
public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
2. 간결하다.

##### 예외
*리플렉션 API인 `AccessibleObject.setAccessible`을 사용해 private 생성자를 호출할 수 있다.*  
리플렉션 API: `java.lang.reflect`, class 객체가 주어지면, 해당 클래스의 인스턴스를 생성하거나 메소드를 호출하거나, 필드에 접근할 수 있다.

```java
Constructor<Elvis> constructor = (Constructor<Elvis>) elvis2.getClass().getDeclaredConstructor();
constructor.setAccessible(true);

Elvis elvis3 = constructor.newInstance();
assertNotSame(elvis2, elvis3); // FAIL
```

##### 해결 방법
*생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.*

```java
private Elvis() {
	if(INSTANCE != null){
		throw new RuntimeException("생성자를 호출할 수 없습니다!");
	}
}
```

&nbsp;

### 2. 정적 팩터리 메서드를 public static 멤버로 제공하는 방식

##### 정적 팩터리 방식의 싱글턴

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding()
}
```

`Elvis.getInstance()`는 항상 같은 객체의 참조를 반환하므로 제2의 `Elvis`인스턴스는 만들어지지 않는다.  
(리플렉션을 통한 예외는 똑같이 적용된다.)

```java
@Test
	public void getInstance() {
		Elvis elvis1 = Elvis.getInstance();
		Elvis elvis2 = Elvis.getInstance();

		assertSame(elvis1, elvis2); // SUCCESS
	}
```

##### 장점

1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.  
(getInstance()호출부 수정 없이 내부에서 private static 이 아닌 새 인스턴스를 생성해주면 된다)  
유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드 별로 다른 인스턴스를 넘겨주게 할 수 있다.    
2. 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.  
`Supplier`: `get`메서드 만을 가지고 아무 type이나 리턴할 수 있는 인터페이스.

    ```java
    Supplier<Elvis> elvisSupplier = Elvis::getInstance;
    Elvis elvis = elvisSupplier.get();
    ```

    메서드에 대한 레퍼런스를 `Supplier`타입으로 만듦.

&nbsp;

### 두 방식의 문제점

**각 클래스를 직렬화한 후 역직렬화할 때 새로운 인스턴스를 만들어서 반환한다.  
역직렬화는 기본 생성자를 호출하지 않고 값을 복사해서 새로운 인스턴스를 반환한다. 그때 통하는게 readResolve() 메서드이다.**  
이를 방지하기 위해 `readResolve` 에서 싱글턴 인스턴스를 반환하고, 모든 필드에 `transient`(직렬화 제외) 키워드를 넣는다.

싱글턴 클래스를 직렬화하려면 단순히 `Serializable`을 구현하고 선언하는 것만으로 부족하다.  
모든 인스턴스 필드를 일시적(`transient`)라고 선언하고 `readResolve` 메서드를 제공해야 한다.

이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

가짜 `Elvis`탄생을 예방하고 싶다면 `Elvis` 클래스에 다음의 `readResolve` 메서드를 추가하자

##### 싱글턴임을 보장해주는 `readResolve` 메서드

```java
private Obejct readResolve() {
	return INSTANCE;
}
```
'진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.

&nbsp;

### 3. 원소가 하나인 열거 타입을 선언하는 방식

대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

```java
public enum Elvis {
	INSTANCE; 

	public void leaveTheBuilding() { ... }
}
```

복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.

단, 만들려는 싱글턴이 Enum 이외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

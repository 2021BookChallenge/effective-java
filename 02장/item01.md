# 🔗 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클래스의 인스턴스를 생성하는 방법은 (1) public 생성자 (2) 정적 팩터리 메서드가 있다.

클래스는 생성자와 별도로 **정적 팩터리 메서드**(static factory method)를 제공할 수 있다.

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

생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.  
**입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식은 좋지 않은 발상이다.**  
각 생성자가 어떤 역할을 하는지 정확히 기억하기 어렵고, 코드를 읽는 사람도 의미를 알지 못할 것이다.

반면 **정적 팩터리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.**

ex) `BigInteger.probablePrime`: '값이 소수인 `BigInteger`를 반환한다.'

&nbsp;

### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

> **불변 클래스를 만들 수 있다.**

불변 클래스(Immutable)는 인스턴스를 미리 만들어 놓거나, 새로 생성한 인스턴스를 캐싱하여 재활용해 불필요한 객체 생성을 막아 성능을 끌어올려 준다.  

> - **불변 클래스 (Immutable)**  
: 변경이 불가능한 클래스  
	- 레퍼런스 타입의 객체 (heap 영역에서 생성)  
	- Thread-safe (재할당은 가능. 값 복사 X)   
	Ex. String, Boolean, Integer, Float, Long ↔ (StringBuilder는 가변 클래스)  
##### `Boolean.valueOf()` 메서드는 객체를 생성하지 않는다.
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
2. 클래스를 인스턴스화 불가(noninstantiable)로 만들 수 있다.
3. 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.  
      `a==b`일 때만 `a.equals(b)` 성립

&nbsp;

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.


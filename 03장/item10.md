# 🔗 아이템 10. equals는 일반 규약을 지켜 재정의 하라
`equals` 메서드를 재정의하지 않고 그냥 두면, 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

&nbsp;

## 💎 `equals`를 재정의 하면 안 되는 경우

1. 각 인스턴스가 본질적으로 고유할 때  
값 클래스(`Integer`나 `String`처럼 값을 표현하는 클래스)가 아닌 동작하는 개체를 표현하는 클래스  
ex) `Thread`

2. 인스턴스의 '논리적 통치성'을 검사할 일이 없을 때  
ex) `java.util.regax.Pattern`은 `equals`를 재정의해 두 `Pattern`의 정규표현식을 비교

3. 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞을 때  
ex) `Set`은 `AbstractSet`이 구현한 `equals`를 상속, `List`는 `AbstractList`, `Map`은 `AbstractMap`

4. 클래스가 *private*이나 *package-private*이고 `equals`를 호출할 일이 없을 때  
아래와 같이 구현해 `equals`가 실수로라도 호출되는 걸 막을 수 있다.  
```java
@Override public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

&nbsp;

## 💎 `equals`를 재정의 해야 하는 경우

객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아닌 '논리적 동치성'을 확인해야 하는데,  
상위 클래스의 `equals`가 논리적 동치성을 비교하도록 재정의 되지 않았을 때 (주로 값 클래스)

ex) 두 값 객체를 `equals`로 비교하는 경우, 객체가 같은지가 아니라 값이 같은지를 알고싶을 것이다.  
`equals`가 논리적 동치성을 확인하도록 재정의하면, 값 비교는 물론 `Map`의 키와 `Set`의 원소로 사용 가능.  
*but,* 값 클래스여도, 같은 인스턴스가 둘 이상 만들어지지 않는 *인스턴스 통제 클래스*라면 재정의하지 않아도 됨.

&nbsp;

## 💎 `equals` 메서드 재정의 일반 규약: 동치관계

**동치 클래스(equivalent class): 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산**  
→ `equals` 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 교환이 가능해야 한다.

- **반사성(reflexivity)**  
: `null`이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 `true`다.
- **대칭성(symmetry)**  
: `null`이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`가 `true`면 `y.equals(x)`도 `true`다.
- **추이성(transitivity)**  
: `null`이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)`가 `true`이고, `y.equals(z)`도 `true`면 `x.equals(z)`도 `true`다.
- **일관성(consistency)**  
: `null`이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`를 반복해서 호출하면 항상 `true`이거나 `false`다.
- `**null`-아님**  
: `null`이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 `false`다.

### 반사성(reflexivity)

> 객체가 자기 자신과 같아야 한다.

```java
public class ProgrammingLanguage{
  private String name;

  public Fruit(String name){
    this.name = name;
  }

  public static void main(){
    Set<ProgrammingLanguage> set = new HashSet<>();
    ProgrammingLanguage language = new ProgrammingLanguage("java");
    set.add(language);
    System.out.println(set.contains(language)); // false일 경우, 반사성을 만족하지 못하는 경우이다.
  }
}
```

### 대칭성(symmetry)

> 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

##### 잘못된 코드 - 대칭성 위배!

```java
// 대칭성을 위반한 클래스
public final class CaseInsensitiveString{
  private final String s;

  public CaseInsensitiveString(String s){
    this.s = Obejcts.requireNonNull(s);
  }

	// 대칭성 위배!
  @Override public boolean equals(Object o){
    if(o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    if(o instanceof String) // 한방향으로만 작동한다.
      return s.equalsIgnoreCase((String) o);
    return false;
  }
}
```

문제는 `CaseInsensitiveString`의 `equals`는 `String`을 알고 있지만, `String`의 `equals`는 `CaseInsensitiveString`의 존재를 모른다는 데 있다. 대칭성을 명백히 위반한다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
s.equals(cis); // false
```

##### 해결 - `CaseInsensitiveString`끼리만 비교하도록 한다.

```java
//대칭성을 만족하게 수정
@Override public boolean equals(Object o){
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
}
```

### 추이성(transitivity)

> 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같아면, 첫 번째 객체와 세 번째 객체도 같아야 한다.

상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하며 `equals`를 재정의할 때 자주 발생하는 문제다.

```java
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if(!o instanceof Point)
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
```

```java
public class ColorPoint extends Point {
	private final Color color;
	
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}

	...
}
```

##### 1. 잘못된 코드 - 대칭성 위배!  

    ```java
    @Override public boolean equals(Object o) {
    	if(!o instanceof ColorPoint)
    		return false;
    	return super.equals(o) && ((ColorPoint) o).color == color;
    }
    ```

    ```java
    public static void main(){
      Point p = new Point(1,2);
      ColorPoint cp = new ColorPoint(1,2, Color.RED);
      p.equals(cp);    // true
      cp.equals(p);    // false
    }
    ```

    `ColorPoint`의 `equals`는 입력 매개변수의 클래스 종류가 다르다며 매번 `false`만 반환할 것이다. 

##### 2. 잘못된 코드 - 추이성 위배!  

```java
    @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof ColorPoint))
        return o.equals(this);
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

```java
    public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      Point p2 = new Point(1,2);
      ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
      p1.equals(p2);    // true 
      p2.equals(p3);    // true 
      p1.equals(p3);    // false
    }
```

`p1.equals(p2);`와 `p2.equals(p3);`는 `true`를 반환하는데, `p1.equals(p3);`는 `false`를 반환해 추이성에 위배된다.  
이 방식은 **무한 재귀**에 빠질 위험도 있다.  

```java
    //SmellPoint.java의 equals
    @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof SmellPoint))
        return o.equals(this);
      return super.equals(o) && ((SmellPoint) o).color == color;
    }
```

```java
    public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      SmellPoint p2 = new SmellPoint(1,2);
      p1.equals(p2);
      // 1. ColorPoint의 equals: 2번째 if문 때문에 SmellPoint의 equals로 비교
      // 2. SmellPoint의 equals: 2번째 if문 때문에 ColorPoint의 equals로 비교
      // 3. 1~2 무한 재귀로 인한 StackOverflow Error
    }
```

구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 규약을 만족시킬 방법은 존재하지 않는다.

##### 3. 잘못된 코드 - 리스코프 치환 원칙 위배!  

그렇다고 `instanceof` 검사 대신 `getClass` 검사를 하라는 것은 아니다.

```java
    @Override public boolean equals(Object o){
      if(o == null || o.getClass() != getClass())
        return false;
      Point p = (Point) o;
      return p.x == x && p.y == y;
    }
```

위의 코드는 같은 구현 클래스의 객체와 비교할 때만 `true`를 반환한다.  
리스코프 치환 원칙: 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.  
= `Point`의 하위 클래스는 여전히 `Point`이므로 어디서든 `Point`로써 활용될 수 있어야 한다.  

**해결 1 - 상속 대신 컴포지션을 사용하라 (`equals` 규약을 지키면서 값 추가하기)**

```java
public class ColorPoint{
  private final Point point;
  private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	/* 이 ColorPoint의 Point 뷰를 반환한다. */
  public Point asPoint(){ // view 메서드 패턴
    return point;
  }

  @Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

> **컴포지션**: 기존 클래스가 새로운 클래스의 구성 요소로 쓰인다.  
**기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 *private* 필드로 기존 클래스의 인스턴스를 참조하게 한다.**  
컴포지션을 통해 새 클래스의 인스턴스 메서드들은 **기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환**한다.

`Point`를 상속하는 대신 `Point`를 `ColorPoint`의 *private* 필드로 두고, `ColorPoint`와 같은 위치의 일반 `Point`를 반환하는 뷰(view 메서드)를 *public*으로 추가하는 식이다.  

- `ColorPoint` vs. `ColorPoint`: `ColorPoint`의 `equals`를 이용하여 color값까지 모두 비교
- `ColorPoint` vs. `Point`: `ColorPoint`의 `asPoint`를 이용하여 `Point`로 바꿔, `Point`의 `equals`를 이용해 x, y비교
- `Point` vs. `Point`: `Point`의 `equals`를 이용해 x, y값 모두 비교

ex) `java.sql.Timestamp`: `java.util.Date` 확장 후 `nanoseconds` 필드 추가.  
→ `Timestamp`의 `equals`는 대칭성을 위배하며, `Date`와 섞어 쓸 때 엉뚱하게 동작할 수 있다.

**해결 2 - 추상 클래스의 하위 클래스 사용하기**

추상 클래스의 하위 클래스에서는 `equals` 규약을 지키면서도 값을 추가할 수 있다.  
상위 클래스의 인스턴스를 직접 만드는 게 불가능하기 때문에, 하위 클래스끼리의 비교가 가능하다.

### 일관성(consistency)

> 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다.

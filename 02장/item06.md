# 🔗 아이템 6. 불필요한 객체 생성을 피하라

> 불필요한 객체 생성을 피하면서 자원을 절약해야 한다.

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다.  
특히, 불변 객체는 언제든 재사용할 수 있다.

##### 같은 값임에도 다른 레퍼런스인 경우 - 기존의 인스턴스를 재사용 하자.

```java
String s = new String("java");
```

이 문장은 실행될 때마다 String 인스턴스를 **새로 만든다.** 

```java
String s = "java";
```

이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 재사용 한다.  
같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.  
cf) String pool의 플라이웨이트 패턴: 같은 내용의 String 객체가 선언된다면 기존의 객체를 참조하게 한다.

생성자대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 불필요한 객체 생성을 피할 수 있다.  
생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.  
ex) `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩터리 메서드 사용

&nbsp;

##### 재사용 빈도가 높고 생성비용이 비싼 경우 - 캐싱하여 재사용 하자.

```java
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

이 코드의 문제점은 `String.matches` 메서드를 사용하는데 있다. `String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기 적합하지 않다.  
이 메서드가 내부에서 만드는 정규 표현식용 `Pattern`인스턴스는 **한 번 쓰고 버려저서 곧바로 가비지 컬렉션 대상이 된다**.
`Pattern`은 생성비용이 높은 클래스 중 하나이다. 만약 늘 같은 `Pattern`이 필요함이 보장되고 재사용 빈도가 높다면 아래와 같이 상수(`static final`)로  초기에 캐싱해놓고 재사용할 수 있다.

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

성능을 개선하려면 필요한 정규표현식을 표현하는 (불변인) `Pattern` 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 `isRomanNumeral`이 호출될 때마다 이를 재사용한다.

생성비용이 비싼 객체라면 "캐싱" 방식을 고려해야 한다.   
자주 쓰는 값이라면 `static final`로 초기에 캐싱해놓고 재사용 하자.

&nbsp;

##### 같은 인스턴스를 대변하는 여러 개의 인스턴스를 생성하지 말자

```java
Map<String, Object> map = new HashMap<>();
map.put("Hello", "World");

Set<String> set1 = map.keySet();
Set<String> set2 = map.keySet();

assertThat(set1).isSameAs(set2); // TRUE
```

Map 인터페이스의 `keySet` 메서드는 Map 객체 안의 키 전부를 담은 `Set` 뷰를 반환한다.  
하지만, 동일한 Map에서 호출하는 `keySet` 메서드는 같은 Map을 대변하기 때문에 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀐다.  
따라서 `keySet`이 뷰 객체 여러 개를 만들 필요도 없고 이득도 없다.


&nbsp;

##### 의도치않은 오토박싱이 숨어들지 않도록 주의하자
```java
private static long sum() {
	Long sum = 0L;
	for(long i=0; i<=Integer.MAX_VALUE; i++) {
		sum += i;
	}
	return sum;
}
```
오토박싱은 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.  
의미상으로는 별다를 것 없지만 성능에서는 그렇지 않다.

`sum`변수를 `long`이 아닌 `Long`으로 사용해서 불필요한 `Long`인스턴스가 약 231개나 만들어졌다.  
(`long` 타입인 `i`가 `Long` 타입인 `sum` 인스턴스에 더해질 때마다)

**박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

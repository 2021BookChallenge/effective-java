# 🔗 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> **사용하는 자원에 따라 동작이 달라지는 클래스**에는 ~정적 유틸리티 클래스~나 ~싱글턴 방식~이 적합하지 않다.  
이 조건을 만족하는 간단한 패턴은,  
**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**이다.

&nbsp; 

##### 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
	private SpellChecker() {} // 객체 생성 방지 

	public static boolean isValid(String word) { ... } 
	public static List<String> suggestions(String typo) { ... } 
}
```

##### 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.
```java
public class SpellChecker { 
	private final Lexicon dictionary = ...; 

	private SpellChecker(...) {} 
	public static SpellChecker INSTANCE = new SpellChecker(...); 

	public static boolean isValid(String word) { ... } 
	public static List<String> suggestions(String typo) { ... } 
}
```

두 방식 모두 사전을 단 하나만 사용한다고 가정해, 그리 훌륭하지 않다.   
또한 직접 명시되어 고정되어 있는 변수는 테스트를 하기 힘들게 만든다.  
**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**  

즉, **의존성을 바깥으로 분리하여 외부로부터 주입받도록 해야 한다**. (의존 객체 주입 패턴)

##### 의존 객체 주입은 유연성과 테스트 용이성을 높여준다.
```java
public class SpellChecker { 
	private final Lexicon dictionary; 

	private SpellChecker(Lexicon dictionary) { 
    		this.dictionary = Objects.requireNonNull(dictionary); 
 	} 

	public boolean isValid(String word) { return true; } 
	public List<String> suggestions(String typo) { return null; } 
}
```
불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.

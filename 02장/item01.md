# 🔗 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자다.  

하지만 클래스는 생성자와 별도로 **정적 팩터리 메서드** (static factory method)를 제공할 수 있다.  

**그 클래스의 인스턴스를 반환하는 단순한 정적 메서드** 말이다.

&nbsp;

```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```
##### 이 메서드는 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해준다.


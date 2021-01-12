# 🔗 아이템 9. try-finally 대신 try-with-resources를 사용하라

자바 라이브러리에는 `close` 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.  
ex) `InputStream`, `OutputStream`, `java.sql.Connection`

자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다.  
전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 *try-finally*가 쓰였다.

&nbsp;

## 💎 try-finally

**try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다!**

```java
public static String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

**자원이 둘 이상이면 try-finally 방식은 너무 지저분하다!**

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

앞의 두 코드 예제에는 결점이 있다.  
예외는 *try* 블록과 *finally* 블록 모두에서 발생할 수 있다. (자원을 해제하는 `close` 메서드를 호출할 때)  
*finally* 블록 내에서 다시 *try-finally* 를 사용하게 되면, 코드는 길어지고 가독성도 떨어진다.

&nbsp;

## 💎 try-with-resources

이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야한다.  
(`AutoCloseable`: 단순히 `void`를 반환하는 `close` 메서드 하나만 정의한 인터페이스다.)  
닫아야 하는 자원을 뜻하는 클래스를 작성한다면 `AutoCloseable`을 반드시 구현해야 한다.

다음은 *try-with-resources*를 이용해 위의 두 예제 코드를 재작성한 예다.

**try-with-resources - 자원을 회수하는 최선책!**

```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return defaultVal;
    }
}
```

*try-with-resources*에서도 *catch* 절을 쓸 수 있다.

**복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다!**

```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```

ex)

```java
public class MyResource implements AutoCloseable {
	public void doSomething() throws FirstError {
		System.out.println("doing something");
		throw new FirstError();
	}

	@Override
	public void close() throws SecondException {
		System.out.println("clean my resource");
		throw new SecondError();
	}
}
```

```java
public class FirstError extends RuntimeException { }
public class SecondError extends RuntimeException { }
```

```java
public class AppRunner {
	public static void main(String[] args) {
		MyResource myResource = new MyResource();
		try {
			myResource.doSomething();
		} finally {
			myResource.close();
		}
	}
}
```

이렇게 코드를 작성하면, try 블록과 finally 블록에서 모두 예외가 발생할 수 있는데,  
**두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.**  
그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서의 디버깅을 어렵게 한다.

```java
public class AppRunner {
	public static void main(String[] args) {
		try (MyResource myResource = new MyResource()){
			myResource.doSomething();
		}
	}
}
```

코드를 이렇게 *try-with-resources*로 고치면 `close`를 알아서 호출해주며,  
`close` 호출에서 예외가 발생했을 때, `close`에서 발생한 예외는 숨겨지고 **첫 번째 예외가 기록된다.**  
이렇게 실전에서는 예외 하나만 보존되고 여러 개의 다른 예외가 숨겨질 수 있다.  

**이렇게 숨겨진 예외들은 스택 추적 내역에 *suppressed* 꼬리표를 달고 출력된다.**  
또한 자바7에서 `Throwable`에 추가된 `getSuppressed` 메서드를 쓰면 프로그램 코드에서 가져올 수 있다.

&nbsp;

## 💎 결론

꼭 회수해야 하는 자원을 다룰 때는 *try-finally* 말고 *try-with-resources*를 사용하자.  
예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.  
*try-finally*로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, *try-with-resources*로는 정확하고 쉽게 자원을 회수할 수 있다. **

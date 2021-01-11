# 🔗 아이템 7. 다 쓴 객체 참조를 해제하라

## 💎 메모리 직접 관리, 메모리 누수의 주범

자바에 GC(가비지 콜렉터)가 있기 때문에, GC가 다 쓴 객체를 알아서 회수해간다고 해서 메모리 관리에 더 이상 신경쓰지 않아도 된다는 것은 오해다.

아래 Stack을 사용하는 프로그램을 오래 실행하다 보면 점차 GC 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.

과연 메모리 누수가 일어나는 위치는 어디일까?

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

이 코드에서는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.  
이 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.  
스택에 계속 쌓다가 많이 빼내도 스택이 차지하고 있는 메모리는 줄어들지 않는다.  
가용한 범위(유의미한 값들을 갖고 있는 부분)는 `elements`배열의 인덱스가  `size` 보다 작은 부분이고, 그 값보다 큰 부분에 있는 값들은 필요없이 메모리를 차지하고 있는 부분이다.

→ 해당 참조를 다 썼을 때 `**null` 처리(참조 해제)** 하면 된다.

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```

각 해당 원소의 참조가 더이상 필요 없어지는 시점(Stack에서 꺼낼 때, 사용이 완료됨)에 `null`로 설정하여 다음 GC가 발생할 때 레퍼런스가 정리되게 한다. 만약 `null` 처리한 참조를 실수로 사용하려 할 때 프로그램이 `NullPointerException`을 던지며 종료할 수 있다. (그 자리에 있는 객체를 비우지 않고 실수로 잘못된 객체를 돌려주는 것보다는 차라리 괜찮다. `null` 처리 하지 않았다면 잘못된 일을 수행할 것이다.)

### 왜 메모리 누수에 취약했을까?

> 일반적으로 **자기 메모리를 직접 관리하는 클래스라면 항시 메모리 누수에 주의해야 한다**.  
원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리해줘야 한다.

스택이 **자기 메모리를 직접 관리하기 때문이다.**   
이 스택은 `elements`배열로 저장소 풀을 만들어 원소들을 관리한다.  
배열의 "활성 영역"에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않는다.  
→ 하지만 가비지 컬렉터는 이 사실을 알 길이 없다. 가비지 컬렉터가 보기에는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체다. 비활성 영역의 객체가 더 이상 쓸모 없다는 건 프로그래머만 아는 사실이다.

**→ 프로그래머는 비활성 영역이 되는 순간 `null` 처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에게 알려야 한다.**

### 객체 참조를 `null` 처리하는 일은 예외적이어야 한다.

그렇다고 필요 없는 객체를 볼 때마다 `null` 처리하면, 오히려 프로그램을 필요 이상으로 지저분하게 만든다.  
**객체 참조를 `null` 처리하는 일은 예외적인 상황에서나 하는 것이지 평범한 일이 아니다.**

필요없는 객체 레퍼런스를 정리하는 최선책은 그 레퍼런스를 가리키는 변수를 특정한 범위(scope)안에서만 사용하는 것이다. 변수의 범위를 가능한 최소가 되게 정의했다면(item 57) 이 일은 자연스럽게 이뤄진다.

```java
Object pop() {

	Object age = 24;

	...

	age = null; // X
}
```

`Object age`는 scope이 `pop()`안에서만 형성되어 있으므로 scope 밖으로 나가면 무의미한 레퍼런스 변수가 되기 때문에 GC에 의해 정리가 된다. (굳이 `null` 처리 하지 않아도 됨)

### 언제 레퍼런스를 `null` 처리 해야 하는가?

> **메모리를 직접 관리하는 클래스는 메모리 누수를 조심해야 한다.**

**메모리를 직접 관리할 때**, `Stack` 구현체처럼 `elements`라는 배열을 관리하는 경우에 GC는 어떤 객체가 필요 없는 객체인지 알 수 없으므로, 해당 레퍼런스를 `null`로 만들어 GC한테 필요없는 객체들이라고 알려줘야 한다.

&nbsp;

## 💎 캐시, 메모리 누수의 주범

캐시를 사용할 때도 메모리 누수 문제를 조심해야 한다. 객체의 레퍼런스를 캐시에 넣어 놓고, 캐시를 비우는 것을 잊기 쉽다. 여러 가지 해결책이 있지만, **캐시의 키**에 대한 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 `WeakHashMap`을 쓸 수 있다.

**캐시 구현의 안 좋은 예 - 객체를 다 쓴 뒤로도 key를 정리하지 않음.**

```java
public class CacheSample {
	public static void main(String[] args) {
		Object key = new Object();
		Object value = new Object();

		Map<Object, List> cache = new HashMap<>();
		cache.put(key, value);
		...
	}
}
```

key의 사용이 없어지더라도 `cache`가 key의 레퍼런스를 가지고 있으므로, GC의 대상이 될 수 없다.

**해결 - `WeakHashMap`**

캐시 외부에서 key를참조하는 동안만 엔트리가 살아있는 캐시가 필요하다면 `WeakHashMap`을 이용한다.  
다 쓴 엔트리는 그 즉시 자동으로 제거된다. 단, `WeakHashMap`은 이런 상황에서만 유용하다.

```java
public class CacheSample {
	public static void main(String[] args) {
		Object key = new Object();
		Object value = new Object();

		Map<Object, List> cache = new WeakHashMap<>();
		cache.put(key, value);
		...
	}
}
```

캐시 값이 무의미해진다면 자동으로 처리해주는 `WeakHashMap`은 key 값을 모두 Weak 레퍼런스로 감싸 hard reference가 없어지면 GC의 대상이 된다.

즉, `WeakHashMap`을 사용할 때 key 레퍼런스가 쓸모 없어졌다면, (key - value) 엔트리를 GC의 대상이 되도록해 캐시에서 자동으로 비워준다.

또는 시간이 지나면 캐시값이 의미가 없어지는 경우에 백그라운드 쓰레드를 사용하거나 (`ScheduledThreadPoolExecutor`), 새로운 엔트리를 추가할 때 부가적인 작업으로 기존 캐시를 비우는 일을 할 것이다. (`LinkedHashMap` 클래스는 `removeEldestEntry`라는 메서드를 제공한다.)

### 참고) Weak 레퍼런스

객체 만들듯이 만드는 모든 레퍼런스는 전부 Strong 레퍼런스.

`WeakReference weakWidget = new WeakReference(widget);`
Strong 레퍼런스를 Weak 레퍼런스로 감싸면 Weak 레퍼런스가 된다.

Garbage Collection 대상이 되려면 해당 객체를 가리키는 레퍼런스가 전부 없어져야 한다.  

Weak 레퍼런스는 Strong한 레퍼런스가 없어지면 Weak 레퍼런스도 GC의 대상이 될 수가 있다.  
`widget`을 가리키던 레퍼런스가 없어지면 `weakWidget`도 가비지 컬렉션 대상이 된다. 

&nbsp;

## 💎 콜백, 메모리 누수의 주범

세번째로 흔하게 메모리 누수가 발생할 수 있는 지점으로 리스너와 콜백이 있다.

클라이언트 코드가 콜백을 등록할 수 있는 API를 만들고 콜백을 뺼 수 있는 방법을 제공하지 않는다면, 계속해서 콜백이 쌓이기만 할 것이다. 이것 역시 `WeakHashMap`을 사용해서 콜백을 Weak 레퍼런스로 저장하면 GC가 이를 즉시 수거해 해결할 수 있다.
ex) `WeakHashMap`에 키로 저장하면 된다.

&nbsp;

## 💎 결론

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.

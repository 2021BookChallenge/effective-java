# 🧷 3장 | 모든 객체의 공통 메서드

## 🔖 다루는 내용

- `Object` 메서드들을 **언제, 어떻게** **재정의**해야 하는지

*Object*는 객체를 만들 수 있는 구체 클래스지만 기본적으로는 상속해서 사용하도록 설계 되었다.  
*Object*에서 *final*이 아닌 메서드(`equals`, `hashCode`, `toString`, `clone`, `finalize`)는 모두 재정의(overriding)을 염두에 두고 설계된 것이라, 재정의 시 지켜야 하는 규약이 명확히 정의 되어 있다.  
그래서 `Object`를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의 해야 한다.

메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(`HashMap`과 `HashSet`등)을 오동작하게 만들 수 있다.


## 🔖 index

🔗 [아이템 10. equals는 일반 규약을 지켜 재정의 하라](https://github.com/2021BookChallenge/Effective-Java/blob/main/03%EC%9E%A5/item10.md)  
🔗 [아이템 11. equals를 재정의하려거든 hashCode도 재정의하라](https://github.com/2021BookChallenge/Effective-Java/blob/main/03%EC%9E%A5/item11.md)

# 🔗 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

## 💎 생성자와 정적 팩터리 메서드의 제약

생성자와 정적 팩터리 메서드에는 **선택적 매개변수가 많을 때 적절히 대응하기 어렵다**는 단점이 있다.

&nbsp;

## 💎 점층적 생성자 패턴 (telescoping constructor pattern)

**필수 매개변수를 받는 생성자 1개, 그리고 선택매개변수를 하나씩 늘여가며 생성자를 만드는 패턴**

필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자, ... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories,  fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories,  fat,  sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
}
```

### 점층적 생성자 패턴의 단점

1. 초기화하고 싶은 필드만 포함한 생성자가 없다면, 설정하길 원치 않는 필드까지 매개변수에 값을 지정해줘야 한다.

2. 복잡하고 읽기 어렵다.  
코드를 읽을 때 각 값의 의미가 무엇인지 헷갈릴 것이고, 매개변수가 몇 개인지도 주의해서 세어 보아야 할 것이다. 타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수 있다. 클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에 엉뚱한 동작을 하게 된다.

3. 매개변수의 수가 많아질 경우 걷잡을 수 없게 된다.

→ 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

&nbsp;

## 💎 자바빈즈 패턴 (JavaBeans pattern)

**매개변수가 없는 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수 값을 설정하는 방식**

```java
public class NutritionFacts {
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setFat(0);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

### 자바빈즈 패턴의 단점

1. 객체 하나를 만들려면 메서드를 여러 개 호출해야 한다.
2. 객체가 완성되기 전까지는 **일관성**이 무너진 상태에 놓이게 된다.

→ 클래스를 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 추가 작업(freeze 등)을 해줘야 한다.

&nbsp;

## 💎 빌더 패턴

**Builder를 이용해 필수 매개변수로 객체를 생성하고, 일종의 setter를 사용하여 선택 매개변수를 초기화한 뒤  
build() 메서드를 호출하여 완전한 객체를 생성하는 패턴**

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수 만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다.  
그런 다음 빌더 객체가 제공하는 일종의 세터 메스드들로 원하는 선택 매개변수들을 설정한다.  
마지막으로 매개변수가 없는 `build`메서드를 호출해 (보통은 불변적인) 객체를 얻는다.

점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비했다.

빌더는 생성할 클래스 안에 정적(static) 멤버 클래스로 만들어두는 게 보통이다. (lombok 사용 가능)

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // 필수 매개변수만을 담은 Builder 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택 매개변수의 setter, Builder 자신을 반환해 연쇄적으로 호출 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        // build() 호출로 최종 불변 객체를 얻는다.
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```

`NutritionFacts` 클래스는 불변이며, 모든 매개변수의 기본값을 한 곳에 모아 뒀다.  
이 빌더의 Setter 메서드는 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.(method chaining)

```
NutritionFacts cocaCola = new NutriFacts.Builder(240, 8)
      .calories(100).sodium(35).carbohydrate(30).build();
```

> 빌더 패턴과 자바 빈즈 패턴의 가장 큰 차이점은 앞에서도 말한, 불변성에 있다.  
자바 빈즈 패턴은 객체를 생성한 '후', 값을 setter 메서드를 통해 넣는다. 그렇기에 객체 사용 도중 실수로, 혹은 악의적인 목적으로 setter 메서드를 통해 유효하지 않은 값이나 null값, 혹은 정확하지 않은 값이 들어갈 수 있다.  
반면, 빌더 패턴은 객체 생성 '전', 값을 setter 메서드를 통해 넣는다. 그리고 다 넣었다면 마지막에 build 메서드를 호출하여 객체를 생성한다.  
그렇기 때문에 객체 사용 중에 값이 변경될 우려가 없으며, 불변성과 안정성이 올라간다.  
당연하지만, 빌더 패턴 사용시에는 public setter 메서드를 선언해서는 안된다.


### 계층적으로 설계된 클래스와 Builder

1. 각 계층의 클래스에 관련 빌더를 멤버로 정의한다.
2. 추상 클래스는 추상 빌더를 갖게한다.
3. 구체 클래스(concrete class)는 구체 빌더를 갖게 한다.

```java
public abstract class Pizza{
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   // 추상 클래스는 추상 Builder를 가진다. 서브 클래스에서 이를 구체 Builder로 구현한다.
   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();

      // 하위 클래스는 이 메서드를 overriding하여 this를 반환하도록 해야 한다.
      protected abstract T self();
   }

   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
   }
}

public class NyPizza extends Pizza {
   public enum Size { SMALL, MEDIUM, LARGE }
   private final Size size;

   public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
         this.size = Objects.requireNonNull(size);
      }

      @Override public NyPizza build() {
         return new NyPizza(this);
      }

      @Override protected Builder self() { return this; }
   }

   private NyPizza(Builder builder) {
      super(builder);
      size = builder.size;
   }
}

public class Calzone extends Pizza {
   private final boolean sauceInside;

   public static class Builder extends Pizza.Builder<Builder> {
      private boolean sauceInside = false;

      public Builder sauceInside() {
         sauceInside = true;
         return this;
      }

      @Override public Calzone build() {
         return new Calzone(this);
      }

      @Override protected Builder self() { return this; }
   }

   private Calzone(Builder builder) {
      super(builder);
      sauceInside = builder.sauceInside;
   }
}
```

`Pizza.Builder` 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다. **여기에 추상 메서드인 `self`를 더해 하위 클래스에서는 형변환 하지 않고도 메서드 연쇄를 지원할 수 있다.** self 타입이 없는 자바를 위한 이 우회 방법을 시뮬레이트한 셀프 타입(simulated self-type) 관용구라 한다.

`Pizza`의 각 하위 클래스 빌더가 정의한 `build`메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.  
`NyPizza.Builer`는 `NyPizza`를 반환하고, `Calzone.Builder`는 `Calzone`를 반환한다.  
하위 클래스 메서드가 상위 클래스의 메서드가 정의한 반환 타입(`Pizza`)이 아닌, 그 하위 타입을 반환하는 기능을 공변 반환 타이핑이라고 한다.  
이 기능을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다. 

##### 계층적 빌더를 사용하는 클라이언트 코드

```java
public class Main {
    public static void main(String[] args) {
        NYPizza pizza = new NYPizza.Builder(SMALL)
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();

        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)
                .sauceInside()
                .build();
    }
}
```
빌더를 이용하면 가변인수 매개변수를 여러 개 사용할 수 있다. `addTopping` 메서드가 이렇게 구현된 예다.  
빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.

### 빌더 패턴의 단점

**성능** 객체를 만들려면 그에 앞서 빌더부터 만들어야 한다. 물론 빌더 생성 비용이 크진 않지만, 성능에 민감한 상황에서는 문제가 될 수 있다.

**장황** 점층적 생성자 패턴보다는 코드가 장황해 매개변수가 4개 이상은 되어야 값어치를 한다.

&nbsp;

## 💎 결론

**생성자나 정적 팩터리가 처리해야 할 매개변수가 많다변 빌더 패턴을 선택하는 게 더 낫다.**

매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.

빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.

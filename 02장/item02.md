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

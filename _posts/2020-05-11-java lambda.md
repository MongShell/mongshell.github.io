---
title:  "Java Lambda"
toc: true
# toc_label: "Table of Contents"
toc_stick: true

categories:
  - Java
tags:
  - Language
---

### Java Lambda

람다에 대해 검색을 하다보면, 람다를 설명하기 전에 함수형 프로그래밍에 대해서 짚고 넘어가는 포스팅이 많습니다.
람다가 함수형 프로그래밍과 밀접한 관계를 가진 것은 맞습니다만, 람다로 시작해서 함수형 프로그래밍의 개념의 늪에 빠져버려 결국 람다 자체를 놓칩니다. 그래서 여기서는 람다부터 설명하겠습니다.

개인적으로 람다의 장점은 함수형 프로그래밍을 할 수 있다는 것이 아닙니다. 첫째도, 둘째도 가독성입니다. (물론 람다가 많이 얽히면 오히려 가독성이 떨어집니다.)

람다를 이해하기 가장 쉬운 방법은 함수형 인터페이스부터 차근차근 설명하는 것입니다. 보통 함수형 인터페이스 중 하나인 Comparator를 많이 사용하는데, Comparator 조차 복잡하게 느껴질 수 있으니 여기서는 직접 함수형 인터페이스를 하나 만들어보겠습니다.

-----------------------

**함수형 인터페이스**는 단일 추상 메소드(single abstract method)를 가진 인터페이스입니다. 일반적으로 다음과 같은 구조를 가집니다.

함수형 인터페이스 Math
```java
@FunctionalInterface
interface Math {
  double execute(double x, double y);
}
```

`@FunctionalInterface`는 이 인터페이스가 함수형 인터페이스라는 것을 나타내는 어노테이션입니다. 만일 아래의 인터페이스가 함수형 인터페이스가 아니면 컴파일 타임 에러를 던집니다.

함수형 인터페이스는 이것이 전부입니다.(함수형 이라는 의미에 발목잡히지 마세요. 천천히 이해하면 됩니다.) 람다를 이해하기 바로 전까지 온것입니다. 이제 이 함수형 인터페이스를 람다를 사용해서 활용해보도록 하겠습니다.

```java
// 함수형 인터페이스 Math
@FunctionalInterface
interface Math {
  double execute(double x, double y);
}

public class Test {
  public static void main(String[] args) {
    // 함수형 인터페이스 구현
    Math addition = new Math() {
      @Override
      public double execute(double x, double y) {
        return x + y;
      }
    };
    System.out.println(addition.execute(1, 2));

    // lambda 1
    Math subtraction = (double x, double y) -> {
      return x - y;
    };
    System.out.println(subtraction.execute(1, 2));

    // lambda 2
    Math multiplication = (x, y) -> {
      return x * y;
    };
    System.out.println(multiplication.execute(1, 2));

    // lambda 3
    Math divide = (x, y) -> x / y;
    System.out.println(divide.execute(1, 2));
  }
}
```

우선 제일 위에 익명 객체를 이용해서 '+'를 구현해본 예제를 살펴보겠습니다.
Math 인터페이스를 구현한 익명 객체로 1 + 2 를 만들었습니다.

아래 labmda 1, labmda 2, labmda 3은 위의 익명 객체를 람다식으로 나타낸 예제들입니다. 이 4개는 동작이 모두 같습니다.
labmda 1을 보면 `(double x, double y)` 가 있습니다. 이는 parameter를 의미합니다. 그리고 화살표 후에 `{ }` 로 나타나는 함수의 body 가 있습니다. 이는 기존의 익명 객체에서 선언한 함수의 body 와 동일합니다.


labmda 1은 labmda 2, labmda 3처럼 간략해질 수 있습니다.
lambda 2에서는 parameter의 타입이 생략되었습니다. compiler는 Math 함수형 인터페이스에 정의되어있는 함수의 parameter 타입을 알고 있기 때문에 생략이 가능합니다.
lambda 3에서는 body 의 괄호와 return 까지 생략되었습니다. 가장 보편적인 람다식입니다. parameter x, y 가 `->` 이후에 오는 연산을 하는 것으로 이해하면 됩니다.


그러면 여기서 하나 궁금한 점이 생깁니다. `double execute(double x, double y);` 에서 우리는 execute라는 이름을 전혀 사용하지 않았습니다. 그러면 이 함수의 이름은 다른 것으로 바뀌어도 동작이 같을까요? 네. 같습니다. Math 인터페이스의 함수는 어차피 하나밖에 없기 때문에 compiler는 함수의 이름을 궁금해하지 않습니다.(JVM 에 대한 추가적인 지식이 필요합니다.) 그래서 함수형 인터페이스 안의 함수 이름은 람다식에서 특별한 의미를 가지지 않습니다.

기존의 클래스를 단 하나도 사용하지 않고 스스로 만든 함수형 인터페이스를 통해서 람다식을 만들어봤습니다. 이제 실제 람다식들이 어떻게 쓰이는지 알아보도록 하겠습니다.


람다식을 처음 배울 때 가장 많이 인용되는 array sorting을 살펴보겠습니다.
여기서 expression 1, 2, 3은 모두 같은 동작을 합니다.
 ```java
public class Main {

  public static void main(String[] args) {
    String[] arr = new String[5];
    arr[0] = "fifth";
    arr[1] = "forth";
    arr[2] = "third";
    arr[3] = "second";
    arr[4] = "first";

    // expression 1
    Arrays.sort(arr, new Comparator<String>() {
      @Override
      public int compare(String s1, String s2) {
        return s1.compareToIgnoreCase(s2);
      }
    });

    // expression 2
    Arrays.sort(arr, (s1, s2) -> s1.compareToIgnoreCase(s2));

    // expression 3
    Arrays.sort(arr, String::compareToIgnoreCase);

    // print array
    for (String name : arr) {
      System.out.println(name);
    }
  }
}
 ```

우선 Arrays.sort부터 살펴보겠습니다. 아래는 실제 Arrays.sort의 선언부입니다. Arrays.sort는 제네릭 타입의 array와 Comparator를 parameter로 받습니다.
```java
public static <T> void sort(T[] a, Comparator<? super T> c) {
    ...
}
```

Main 예제의 expression 1에서는 익명 Comparator객체를 Arrays.sort에 넘겨줘서 정렬 조건을 전달했습니다.
```java
    // expression 1
    Arrays.sort(arr, new Comparator<String>() {
      @Override
      public int compare(String s1, String s2) {
        return s1.compareToIgnoreCase(s2);
      }
    });
```

그러면 두번째 인자로 넘겨주는 Comparator의 실제 코드를 확인해보겠습니다. Comparator도 `@FunctionalInterface` 어노테이션이 있는 함수형 인터페이스이고, compare라는 단일 추상 메소드가 있다는 것을 확인할 수 있습니다.
```java
@FunctionalInterface
public interface Comparator<T> {
  int compare(T var1, T var2);

  ...
}
```
Comparator도 `@FunctionalInterface` 어노테이션이 있는 함수형 인터페이스이고, compare라는 단일 추상 메소드가 있다는 것을 확인할 수 있습니다.

즉, Comparator의 compare 메소드는 람다식으로 표현이 가능합니다.(위에서 Math의 execute에 해당합니다.)
compare 메소드를 람다식으로 나타낸 것이 expression 2입니다. compare의 구현 부분이 `->` 이후인 `s1.compareToIgnoreCase(s2)` 라고 생각하면 됩니다.
expression 3에서는 람다보다도 더 낯선 형태가 나타나는데, 이러한 방식은 메소드 참조라고 합니다. 메소드 참조에 대해서는 다른 포스팅에서 다루도록 하겠습니다.

이번에는 배열이 아닌 ArrayList 정렬을 살펴보겠습니다.

```java
public class Main {

  public static void main(String[] args) {
    ArrayList<String> arr = new ArrayList<>();
    arr.add("fifth");
    arr.add("forth");
    arr.add("third");
    arr.add("second");
    arr.add("first");

    // expression 1
    arr.sort(new Comparator<String>() {
      @Override
      public int compare(String s1, String s2) {
        return s1.compareTo(s2);
      }
    });

    // expression 2
    arr.sort((s1, s2) -> s1.compareTo(s2));

    // expression 3
    arr.sort(String::compareTo);

    for (String name : arr) {
      System.out.println(name);
    }
  }
}
```

이번 예제에서는 ArrayList<String>을 정렬합니다. 여기서 사용되는 ArrayList.sort 함수의 선언부는 아래와 같습니다. 역시 Comparator를 parameter로 받습니다.

```java
  public void sort(Comparator<? super E> c) {
    ...
  }
```

ArrayList를 expression 1와 같은 방법으로 정렬합니다. 이는 위에서 배열을 정렬할 때와 같이 익명 객체를 활용한 방법입니다. 그리고 expression 2는 람다를 사용해서 정렬한 것인데, 훨씬 가독성이 좋아진 것을 볼 수 있습니다. expression 3은 메소드 참조로 나타낸 것입니다.

이번에는 사용자 정의 클래스의 인스턴스들을 정렬해보도록 하겠습니다.

```java
public class Main {

  public static void main(String[] args) {
    ArrayList<User> arr = new ArrayList<>();
    arr.add(new User(5, "fifth"));
    arr.add(new User(4, "forth"));
    arr.add(new User(3, "third"));
    arr.add(new User(2, "second"));
    arr.add(new User(1, "first"));

    // expression (Name) 1
    arr.sort(new Comparator<User>() {
      @Override
      public int compare(User u1, User u2) {
        return u1.getName().compareTo(u2.getName());
      }
    });

    // expression (Name) 2
    arr.sort((u1, u2) -> u1.getName().compareTo(u2.getName()));

    // expression (Name) 3
    arr.sort(Comparator.comparing(User::getName));

    // expression (Id) 1
    arr.sort(new Comparator<User>() {
      @Override
      public int compare(User u1, User u2) {
        return u1.getId().compareTo(u2.getId());
      }
    });

    // expression (Id) 2
    arr.sort((u1, u2) -> u1.getId().compareTo(u2.getId()));

    // expression (Id) 3
    arr.sort(Comparator.comparing(User::getId));
    arr.sort(Comparator.comparingInt(User::getId));

    for (User user : arr) {
      System.out.println(user.toString());
    }
  }
}
```

User 클래스는 int id와 String name을 가지는 클래스입니다.

expression (Name) 1,2,3 은 User 인스턴스들을 name을 기준으로 정렬했습니다.
expression (Id)) 1,2,3 은 User 인스턴스들을 id를 기준으로 정렬했습니다.

둘 다 앞의 예제의 활용입니다. Comparator에 정렬 기준이 getName이냐 getId이냐만 다릅니다. 그런데 expression (Id) 3이 두가지로 표현됩니다. 이는 메소드 참조로 Id를 정렬하는 두가지 방법을 모두 나타냅니다.
---
title:  "Java Method Reference"
toc: true
# toc_label: "Table of Contents"
toc_stick: true

categories:
  - Java
tags:
  - Language
---

### Java Method Reference

이전 포스트에서 람다를 보면서 메소드 참조라는 것을 언급했습니다. 이번 포스트에서는 메소드 참조에 대해서 살펴보겠습니다.

메소드 참조의 표현 방식은 3가지가 있습니다.
1. Class::instanceMethod
2. Class::staticMethod
3. object::instanceMethod

첫 번째 방식에서는 첫 번째 매개변수가 메소드의 수신자가 되고, 나머지 매개변수는 메소드에 전달됩니다. (예. String::compareTo 가 (s1, s2) -> s1.compareTo(s2)와 같음)
두 번째 방식에서는 모든 매개변수가 정적 메소드로 전달됩니다.
세 번째 방식에서도 모든 매개변수가 인스턴스 메소드로 전달됩니다.

지금부터 예제를 통해 3가지의 메소드 참조가 작동하는 방식을 알아보겠습니다.

Custom 클래스
```java
public class Custom {

  int value;

  public Custom(int value) {
    this.value = value;
  }

  public int customCompare(Custom e) {
    System.out.println("customCompare! value : " + this.value + ", parameter : " + e.value);
    return this.value - e.value;
  }

  public static void println(Custom e) {
    System.out.println("println! value : " + e.value);
  }

  public void printDecorator(Custom e) {
    System.out.println("printDecorator! value : ### " + e.value + " ###");
  }
}
```

동작 테스트 클래스
```java
public class Main {

  public static void main(String[] args) {

    ArrayList<Custom> test = new ArrayList<>();
    test.add(new Custom(10));
    test.add(new Custom(20));
    test.add(new Custom(30));

    // Class::instanceMethod
    test.sort(Custom::customCompare);
    System.out.println("=========================");
    test.sort((e1, e2) -> e1.customCompare(e2));
    System.out.println();

    // Class::staticMethod
    test.forEach(Custom::println);
    System.out.println("=========================");
    test.forEach(e -> Custom.println(e));
    System.out.println();

    // object::instanceMethod
    Custom custom = new Custom(0);
    test.forEach(custom::printDecorator);
    System.out.println("=========================");
    test.forEach(e -> custom.printDecorator(e));
    System.out.println();
  }
}
```

출력
```
customCompare! value : 20, parameter : 10
customCompare! value : 30, parameter : 20
=========================
customCompare! value : 20, parameter : 10
customCompare! value : 30, parameter : 20

println! value : 10
println! value : 20
println! value : 30
=========================
println! value : 10
println! value : 20
println! value : 30

printDecorator! value : ### 10 ###
printDecorator! value : ### 20 ###
printDecorator! value : ### 30 ###
=========================
printDecorator! value : ### 10 ###
printDecorator! value : ### 20 ###
printDecorator! value : ### 30 ###
```

---------------------------

첫 번째 방식(Class::instanceMethod)의 동작 테스트 코드입니다. 출력을 보면 `test.sort(Custom::customCompare);`와 `test.sort((e1, e2) -> e1.customCompare(e2));`의 동작이 같습니다.
```java
    // Class::instanceMethod
    test.sort(Custom::customCompare);
    test.sort((e1, e2) -> e1.customCompare(e2));
```

이유를 파악하기 전에, ArrayList.sort의 선언부를 살펴보겠습니다. ArrayList.sort는 Comparator의 구현을 parameter로 받습니다.
```java
  public void sort(Comparator<? super E> c) {
    ...
  }
```

Comparator도 살펴보겠습니다. Comparator는 추상 메소드 compare를 가진 함수형 인터페이스입니다.
```java
@FunctionalInterface
public interface Comparator<T> {
  int compare(T var1, T var2);
  ...
}
```

다시 람다식을 기억할 겸, ArrayList.sort에 익명 객체 형식을 넘기는 방식으로 Comparator를 구현하는 방법을 확인해보겠습니다. 이 방식은 `test.sort((e1, e2) -> e1.customCompare(e2));`과 동작이 같습니다.
```java
    test.sort(new Comparator<Custom>() {
      @Override
      public int compare(Custom e1, Custom e2) {
        return e1.customCompare(e2);
      }
    });
```

정리해보겠습니다. ArrayList.sort에는 함수형 인터페이스 Comparator의 compare 구현 부분을 `(e1, e2) -> e1.customCompare(e2)`로 넘겨주면 됩니다. 컴파일러는 전달받은 함수를 Comporator.compare 의 body로 컴파일합니다. T(Type)만 일치하면 컴파일이 가능합니다. 그래서 `(e1, e2) -> e1.customCompare(e2)` 부분은 compare의 body에 전달됩니다.
```java
  // Class::instanceMethod
  public int customCompare(Custom e) {
    System.out.println("customCompare! value : " + this.value + ", parameter : " + e.value);
    return this.value - e.value;
  }
```

그러면 `Custom::customCompare`은 어떻게 `(e1, e2) -> e1.customCompare(e2)`와 같은 람다식으로 변하는 것일까요? 다시 처음의 설명을 인용하겠습니다.

> 첫 번째 매개변수가 메소드의 수신자가 되고, 나머지 매개변수는 메소드에 전달됩니다.

그래서 `Custom::customCompare`의 출력을 보면 `this.customCompare(e)`이라고 볼 수 있습니다.
`int compare(T var1, T var2)`가 `var1.customCompare(var2)`로 되었다고 이해해도 좋습니다.

요약하면, `Custom::customCompare`가 `(e1, e2) -> e1.customCompare(e2)`로 될 때, 첫번째 매개변수가 String 원소가 되고, 두번째 매개변수가 메소드의 매개변수가 되는 것입니다.

-------------------

두번째 방식(Class::staticMethod)입니다.
동작 테스트 코드입니다. 출력을 보면 `test.forEach(Custom::println);`와 `test.forEach(e -> Custom.println(e));`의 동작이 같습니다.
```java
    // Class::staticMethod
    test.forEach(Custom::println);
    test.forEach(e -> Custom.println(e));
```

 ArrayList.forEach의 선언부를 확인해보겠습니다. ArrayList.forEach는 Consumer의 구현을 parameter로 받습니다.
```java
  public void forEach(Consumer<? super E> action) {
    ...
  }
```

Comsumer를 확인해보겠습니다. Consumer는 추상 메소드 accept를 가진 함수형 인터페이스입니다. (Consumer에 대해서는 추가 설명이 필요하나, 지금은 메소드 참조에 대해 집중하기 위해서 패스합니다.)
```java
@FunctionalInterface
public interface Consumer<T> {
  void accept(T var1);
  ...
}
```

다시 람다식을 기억할 겸 ArrayList.forEach를 익명 객체를 넘기는 방식으로 구현한 것을 보겠습니다. 이 방식은 `test.forEach(e -> Custom.println(e));`과 동작이 같습니다. ArrayList.forEach는 함수형 인터페이스 Consumer 의 accept 구현 부분을 넘겨주면 된다는 것을 알 수 있습니다.
```java
    test.forEach(new Consumer<Custom>() {
      @Override
      public void accept(Custom e) {
        Custom.println(e);
      }
    });
```

이제 메소드 참조가 동작하는 방식을 다시 확인해보겠습니다. `test.forEach(Custom::println);`은 `test.forEach(e -> Custom.println(e));`와 동일하게 동작합니다. 그리고 `Custom::println`은 static 메소드입니다. 다시 앞의 설명을 인용하겠습니다.

> 모든 매개변수가 정적 메소드로 전달됩니다.

그래서 `Custom::println`가 `e -> Custom.println(e)`와 동일하게 동작합니다.

왜 두번째 방식(Class::staticMethod)에서는 모든 매개변수가 정적 메소드로 전달될까요? 사실 생각해보면 Class::staticMethod에서는 인스턴스가 존재하지 않습니다. 그래서 this가 쓰일수도 없습니다. 컴파일러가 혼란스러울 일도 없습니다.

그러면, `void accept(T var1);`은 `Custom.println(var1)`이라고도 할 수 있습니다.

---------------------

세 번째 경우(object::instanceMethod)도 두번째 경우와 비슷합니다. (그런데 실제로 많이 보지는 못했습니다.) 두번째와의 유일한 차이는 인스턴스를 선언했기 때문에 instanceMethod를 호출할 수 있다는 것입니다.


지금까지는 예제로 메소드 참조를 확인해봤습니다. 그러면 실제로는 어떤지, 이전 포스트에서 나타났던 예제를 통해 이해해보도록 하겠습니다.

-------------------------

Arrays.sort로 배열을 정렬하는 예제입니다.
```java
    String[] arr = new String[5];
    arr[0] = "fifth";
    arr[1] = "forth";
    arr[2] = "third";
    arr[3] = "second";
    arr[4] = "first";

    Arrays.sort(arr, String::compareToIgnoreCase);
```

Arrays.sort를 확인해보도록 하겠습니다. parameter는 배열과 Comparator입니다.
```java
  public static <T> void sort(T[] a, Comparator<? super T> c) {
    ...
  }
```

`Arrays.sort(arr, String::compareToIgnoreCase);`의 의미를 파악해보겠습니다. Arrays.sort를 보면 첫번째 parameter로는 배열을 넘기고, 두번째 인자에는 함수형 인터페이스에 있는 함수의 구현부를 넘깁니다. Comparator를 넘겨줘야하는 곳에 String::compareToIgnoreCase를 넘겨줬습니다. 어떻게 동작하는지 살펴보겠습니다.

String.compareToIgnoreCase입니다. 이 함수가 Comparator의 `int compare(T var1, T var2);`의 구현부가 됩니다. 이는 위에서 말한 첫 번째 방식(Class::instanceMethod)입니다.
```java
  public int compareToIgnoreCase(String str) {
    return CASE_INSENSITIVE_ORDER.compare(this, str);
  }
```

----------------------

두번째 예시를 보겠습니다. ArrayList.sort로 ArrayList<String>를 정렬하는 예제입니다.
```java
    ArrayList<String> arr = new ArrayList<>();
    arr.add("fifth");
    arr.add("forth");
    arr.add("third");
    arr.add("second");
    arr.add("first");

    arr.sort(String::compareTo);
```

ArrayList.sort를 확인해보도록 하겠습니다. parameter는 Comparator입니다.
```java
  public void sort(Comparator<? super E> c) {
    ...
  }
```

`arr.sort(String::compareTo);`의 의미는 Comparator의 구현부에 `String::compareTo`를 넘긴다는 의미입니다. 역시 첫 번째 방식(Class::instanceMethod)입니다.

String.compareTo입니다. 이 함수가 Comparator의 `int compare(T var1, T var2);`의 구현부가 됩니다. 이는 위에서 말한 첫 번째 방식(Class::instanceMethod)입니다.
```java
  public int compareTo(String anotherString) {
    ...
  }
```

---------------------

마지막 예제를 보겠습니다. ArrayList.sort로 ArrayList<User>를 정렬하는 예제입니다.

```java
    ArrayList<User> arr = new ArrayList<>();
    arr.add(new User(5, "fifth"));
    arr.add(new User(4, "forth"));
    arr.add(new User(3, "third"));
    arr.add(new User(2, "second"));
    arr.add(new User(1, "first"));

    arr.sort(Comparator.comparing(User::getName));
```

ArrayList.sort에 Comparator가 넘어가는 것은 같은데 이번에는 조금 더 복잡한 함수가 parameter로 넘어갔습니다. 차근차근 확인해보겠습니다.

아래 예제에 나온 모든 표현은 동작이 같습니다. 하나하나 파악해보도록 하겠습니다.
```java
    // expression 1
    arr.sort(Comparator.comparing(User::getName));

    // expression 2
    arr.sort(Comparator.comparing(user1 -> user1.getName()));

    // expression 3
    arr.sort(Comparator.comparing(new Function<User, String>() {
      @Override
      public String apply(User user1) {
        return user1.getName();
      }
    }));
```

Comparator.comparing 선언부입니다. Function을 parameter로 받는 것을 확인할 수 있고 동작이 조금 복잡합니다.
```java
  static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor) {
    ...
  }
```

우선 Function을 확인해보겠습니다. `R apply(T var1);`라는 단일 추상 메소드를 가진 함수형 인터페이스입니다.
```java
@FunctionalInterface
public interface Function<T, R> {
  R apply(T var1);
    ...
}
```
여기서 R 은 Return Type, T는 (Input) Type를 의미합니다. 즉, Function 은 한 타입을 받아서 다른 타입으로 변환해주는 것을 알 수 있습니다. expression 3을 보면 `new Function<User, String>`으로 선언되어 있는 것을 확인할 수 있습니다. 즉, User 타입을 받아서 String 타입으로 변환해주는 역할을 합니다.

그렴 `arr.sort(Comparator.comparing(user1 -> user1.getName()));` 을 확인해보겠습니다.

여기서 Comparator 에 전달되는 `user1 -> user1.getName()` 이 부분은 Class::instanceMethod 형식입니다. 첫 번째 매개변수만 있고 getName()은 parameter가 없으니, `Comparator.comparing(User::getName)` 은 `Comparator.comparing(user1 -> user1.getName())`이 됩니다.

다시 Comparator.comparing를 살펴보겠습니다.

Comparator.comparing 구현부입니다.
```java
  static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor) {
    Objects.requireNonNull(keyExtractor);
    return (Comparator)((Serializable)((c1, c2) -> {
      return ((Comparable)keyExtractor.apply(c1)).compareTo(keyExtractor.apply(c2));
    }));
  }
```
return 부분에 보면 인자(keyExtractor)의 apply 결과를 compareTo로 비교하는 것을 확인할 수 있습니다. 여기서 `keyExtractor.apply(c1)`은 `c1.keyExtractor.getName()`으로 해석할 수 있습니다. 이렇게 `arr.sort(Comparator.comparing(User::getName));`은 `User::getName`의 값들을 비교하는 Comparator를 ArrayList.sort에 넘겨주는 것입니다.


마지막으로 `arr.sort(Comparator.comparingInt(User::getId));`를 확인해보겠습니다.

Comparator.comparingInt
```java
  static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
    Objects.requireNonNull(keyExtractor);
    return (Comparator)((Serializable)((c1, c2) -> {
      return Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
    }));
  }
```

아까 보았던 `Comparator.comparing`과 유사한데 ToIntFunction 부분이 다릅니다. ToIntFunction 도 확인해보겠습니다.

```java
@FunctionalInterface
public interface ToIntFunction<T> {
  int applyAsInt(T var1);
}
```

return type이 int라는 것 외에는 Function과 완전히 동일합니다.

즉 `return Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));`

return 부분에 보면 인자(keyExtractor)의 applyAsInt 결과를 compare로 비교하는 것을 확인할 수 있습니다.
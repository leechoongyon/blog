>  source 는 [Github](https://github.com/leechoongyon/JavaExamples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 10편] Functional Interface



# 함수형 인터페이스란?

- 1개의 추상 메소드 interface 입니다.

- Functional Interface 는 자바의 람다식에 접근하기 위함입니다.

```java
@Test
public void functionalInterfaceTest01() throws Exception {
    FunctionalInterfaceSample func = message -> System.out.println(message);
    func.helloWorld("HelloWorld...");
}

public interface FunctionalInterfaceSample {
    public abstract void helloWorld(String message);
}
```





# 자바에서 제공하는 함수형 인터페이스



### Supplier

- parameter 는 없으며, T 타입을 리턴합니다.



```java
public interface Supplier<T> {
    T get();
}

@Test
public void supplierFunctionalInterfaceTest() throws Exception {
    Supplier<String> getMessage = () -> "Hello World...";
    assertThat(getMessage.get());
}

```



### Consumer

- parameter 를 T 타입으로 받고, return 값은 없습니다.



```java
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}





@Test
public void consumerFunctionalInterfaceTest() throws Exception {
	Consumer<String> setMessage = message -> System.out.println(message);
	setMessage.accept("Hello World...");
}
```





### Function

- Parameter 로 T 타입을 받고, R 타입을 리턴합니다.



```java
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}



@Test
public void functionFuntionalInterfaceTest() throws Exception {
	Function<String, String> appendPrefixSuffix = (str) -> "###" + str + "###";
	Assertions.assertEquals("###Hello World###", appendPrefixSuffix.apply("Hello World"));
}

```





# Predicate

- parameter T 를 받아 boolean 을 return 합니다.



```java
public interface Predicate<T> {
    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}




@Test
public void predicateFunctionalInterfaceTest() throws Exception {
  Predicate<String> isEqualsHelloWorld = message -> message.equals("Hello World");
  Assertions.assertEquals(true, isEqualsHelloWorld.test("Hello World"));
  Predicate<String> isNotEqualsHelloWorld = isEqualsHelloWorld.negate();
  Assertions.assertEquals(true, isNotEqualsHelloWorld.test("Hello..."));
}

```



# BiPredicate

- parameter T, U 를 받아 boolean 을 return 합니다.



```java
@FunctionalInterface
public interface BiPredicate<T, U> {

    boolean test(T t, U u);

    default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) && other.test(t, u);
    }

    
    default BiPredicate<T, U> negate() {
        return (T t, U u) -> !test(t, u);
    }

    default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) || other.test(t, u);
    }
}


@Test
void biPredicateFunctionalInterfaceTest() {
    BiPredicate<String, String> biPredicate = (x, y) -> {
        return x.equals(y);
    };
    Assertions.assertEquals(true, biPredicate.test("Hello World", "Hello World"));
}
```





# Reference

- https://www.baeldung.com/java-8-functional-interfaces

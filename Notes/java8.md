## Java 8 

### 1. Lambda example

##### 1.1 Old Style
``` java
public class AppleComparator implements Comparator<Apple> {
  public int compare(Apple a1, Apple a2) {
    return a.getWeight().compareTo(a2.getWeight());
  }
}
inventory.sort(new AppleComparator());
```

##### 1.2 Using Anonymous Class
``` java
inventory.sort(new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a.getWeight().compareTo(a2.getWeight());
  }
});
```

##### 1.3 Using Lambda Expressions
``` java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

##### 1.4 Using Method Reference, with static import of java.util.Comparator.comparing
``` java
inventory.sort(comparing(Apple.getWeight));
```

###### Reversed Order
``` java
inventory.sort(comparing(Apple.getWeight).reversed());
```

###### Chaining
``` java
inventory.sort(comparing(Apple.getWeight).reversed().thenComparing(Apple.getCountry));
```

### 2. Predicate<T> and Function<T, R>

##### 2.1 Predicate<T> Composing
```java
@FunctionalInterface
public interface Predicate<T〉{
  boolean test(T t);
}

Predicate<Apple> redApple = a -> a.getColor().equals("red");
Predicate<Apple> notRedApple = redApple.negate();
Predicate<Apple> redAndHeaveApple = redApple.and(a -> a.getWeight() > 150);
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150).or(a -> "green".equals(a.getColor()));
```

##### 2.1 Function<T, R> Composing
```java
Function(Integer, Integer) f = x -> x+1;
Function(Integer, Integer) g = x -> x*2;
Function(Integer, Integer) h = f.andThen(g);
int result = h.apply(1);  // This returns 4. g(f(x))
```
##### Another example:
```java
Function(Integer, Integer) f = x -> x+1;
Function(Integer, Integer) g = x -> x*2;
Function(Integer, Integer) h = f.compose(g);
int result = h.apply(1);  // This returns 3. f(g(x))
```

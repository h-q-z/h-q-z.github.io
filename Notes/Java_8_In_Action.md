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

### 3. Stream API

##### 3.1 Old Java Example
```java
List<Dish> lowCalDishes = new ArrayList<>();
for(Dish d: menu) {
  if (d.getCalories() < 400) {
    lowCalDishes.add(d);
  }
}
Collections.sort(lowCalDishes, new Comparator<Dish>() {
  public int compare(int d1, int d2){
    return Integer.compare(d1.getCalories(), d2.getCalories());
  }
});
List<String> lowCaloriesDishesNames = new ArrayList<>();
for (Dish d : lowCalDishes) {
  lowCaloriesDishesNames.add(d.getName());
}
```

##### Using Stream API

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloriesDishesNames = menu.stream().filter(d -> d.getCalories() < 400)
                                                   .sorted(comparing(Dish.getCalories))
                                                   .map(Dish.getName)
                                                   .collect(toList());
```

##### Run in parallel
```java
List<String> lowCaloriesDishesNames = menu.parallelStream()
                                                   .filter(d -> d.getCalories() < 400)
                                                   .sorted(comparing(Dish.getCalories))
                                                   .map(Dish.getName)
                                                   .collect(toList());
```

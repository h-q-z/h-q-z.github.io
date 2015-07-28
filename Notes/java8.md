### Java 8 Lambda example

##### 1. Old Style
``` java
public class AppleComparator implements Comparator<Apple> {
  public int compare(Apple a1, Apple a2) {
    return a.getWeight().compareTo(a2.getWeight());
  }
}
inventory.sort(new AppleComparator());
```

##### 2. Using Anonymous Class
``` java
inventory.sort(new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a.getWeight().compareTo(a2.getWeight());
  }
});
```

##### 3. Using Lambda Expressions
``` java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

##### 4. Using Method Reference, with static import of java.util.Comparator.comparing
``` java
inventory.sort(comparing(Apple.getWeight));
```

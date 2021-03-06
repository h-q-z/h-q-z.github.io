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

##### 3.2 Using Stream API

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloriesDishesNames = menu.stream().filter(d -> d.getCalories() < 400)
                                                   .sorted(comparing(Dish.getCalories))
                                                   .map(Dish.getName)
                                                   .collect(toList());
```

##### 3.3 Run in parallel
```java
List<String> lowCaloriesDishesNames = menu.parallelStream()
                                                   .filter(d -> d.getCalories() < 400)
                                                   .sorted(comparing(Dish.getCalories))
                                                   .map(Dish.getName)
                                                   .collect(toList());
```

##### 3.4 Intermediate Operations
```java
List<String> names = menu.stream().filter(d -> {
                                                  System.out.println("Filtering " + d.getName());
                                                  return d.getCalories() > 300;
                                                })
                                  .map(d -> {
                                               System.out.println("Mapping " + d.getName());
                                               return d.getName();
                                            })
                                  .skip(2)
                                  .limit(3)
                                  .collect(toList());
System.out.println(names);
```

##### 3.5 Filter unique elements
```java
List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream().filter(i -> i % 2 == 0).distinct().forEach(System.out::println);
```

##### 3.6 FlatMap, reduce, & collect
```java
String[] arrayOfWords = {"Remember", "my", "sun-and-star", "remember", "come", "back", "to", "me"};
arrayOfWords.stream().map)w -> w.split("")).flatMap(Array::stream).distinct().collect(Collectors.toList());

int sum = numbers.reduce(0, (a,b) -> a+b);
Optional<Integer> min = numbers.reduce(Integer::min);
Optional<Dish> mostCaloriesDish = menu.stream().collect(reducing( (d1,d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2);

int totalCalories = menu.stream().collect(reducing(1, Dish::getCalories, Integer::sum));

Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
Map<Dish.Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));
```

##### Example: test if prime
```java
public boolean isPrime(int candidate) {
  int candidateRoot = (int)Math.sqrt((double)candidate);
  return IntStream.rangeClosed(2, candidateRoot).noneMatch(i -> candidate % i == 0);
}

public Map<Boolean, List<Integer>> partitionPrimes(int n) {
  return IntStream.rangeClosed(2,n).boxed().collect(partitioningBy(candidate -> isPrime(candidate)));
}

```

### 4. Parallel Processing

##### 4.1 Turn sequential into parallel
```java
public static long parallelSum(long n) {
  return Stream.iterate(1L, i -> i+1).limit(n).parallel().reduce(0L, Long::sum);
}
```

* Config Thread Pool: 
```java
  System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12"); // global setting
```

##### 4.2 Future: Async Example
```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
  new Thread( () -> { try {
                            double price = calculatePrice(product);
                            futurePrice.complete(price);
                          } catch (Exception ex) {
                            futurePrice.completeExceptionally(ex);
                          }
                    }).start();
  return futurePrice;
}

Shop s = new Shop("BestShop");
Future<Double> futurePrice = shop.getPriceAsync("DreamProduct");

doSomethingElse();

try {
  double price = futurePrice.get(); // this will read or block if not available yet
} catch (Exception ex) {
  throw new RuntimeException(ex);
}

```

##### Example of Finding all Prices in Shop
```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures = shops.stream()
          .map(shop -> 
               CompletableFuture.supplyAsync( () -> shop.getName() + " price is " + shop.getPrice(product)))
          .collect(Collectors.toList());
  return priceFutures.stream().map(CompletableFuture::join).collect(toList));
}
```

##### Using ExecutorService
```java
ExecutorService executor = Executors.newCachedThreadPool();
final Future<Double> futureRate = executor.submit(new Callable<Double>() {
    public Double call() {
      return exchangeService.getRate(Money.EUR, money.USD);
    }
  });
Future<Double> futurePriceInUSD = executor.submit(new Callable<Double>() {
    public double call() {
      double priceInEUR = shop.getPrice(product);
      return priceInEUR * futureRate.get();
    }
  });  
  
public Stream<CompletableFuture<String>> findPricesStream(String product) {
  return shops.stream().map(shop -> CompletableFuture.supplyAsync( () -> shop.getPrice(product), executor))
                       .map(future -> future.thenApply(Quote::parse))
                       .map(future -> future.thenCompose(
                              quote -> CompletableFuture.supplyAsync( 
                                          () -> Discount.applyDiscount(quote), executor)));
}

CompletableFuture[] futures = findPricesStream("myPhone")
      .map(f -> f.thenAccept(System.out::println))
      .toArray(size -> new CompletableFuture[size]);
CompletableFutures.allOf(futures).join();
```

### 5. New Date and Time API
```java
LocalDate date = LocalDate.of(2014, 3, 18);
int year = date.getYear();

LocalDate today = LocalDate.now();
LocalTime t1 = LocalTime.of(13, 45, 20); // 13:45:20

LocalDate d2 = LocalDate.parse("2014-03-18");
LocalTime t2 = LocalTime.parse("13:45:20");

LocalDateTime dt1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);

LocalDate d3 = dt1.toLocalDate();

Instant.ofEpochSecond(3,0);

Duration du1 = Duration.between(time1, time2);
Duration du2 = Duration.ofMinutes(3);

Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
Period p = Period.between(LocalDate.of(2014, 3, 1), LocalDate.of(2014, 7, 19));
```



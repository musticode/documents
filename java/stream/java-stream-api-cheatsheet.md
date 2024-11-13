# Java Stream API

### Create stream

```java
// From parameters
Stream<String> s = Stream.of("a", "b");

// From arrays
Arrays.stream(myArray)

// From collections
myCollections.stream()

// you also have specialized streams like IntStream, LongStream, DoubleStream, some example below:
IntStream ints = IntStream.range(0, 10_000);
IntStream rdInts = new Random().ints(1000);
```

### Common methods

- `filter()`
- `mapToInt`, `mapToObj`
- `map()`
- `collect()`
- `findAny()`, `findFirst()`, `noneMatch()`
- `forEach()`
- `distinct()`
- `sorted()` `sorted(Comparator<? super T>)`



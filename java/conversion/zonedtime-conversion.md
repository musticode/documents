# Zoned date time conversion and matching calculation

You can use method between from ChronoUnit.

This method converts those times to same zone (zone from the first argument) and after that, invokes until method declared in Temporal interface:
```java
static long zonedDateTimeDifference(ZonedDateTime d1, ZonedDateTime d2, ChronoUnit unit){
    return unit.between(d1, d2);
}
```
Since both ZonedDateTime and LocalDateTime implements Temporal interface, you can write also universal method for those date-time types:

```java
static long dateTimeDifference(Temporal d1, Temporal d2, ChronoUnit unit){
    return unit.between(d1, d2);
}
````

But keep in mind, that invoking this method for mixed LocalDateTime and ZonedDateTime leads to DateTimeException.
# MomentJS and Java ZonedDateTime class

javascript momentjs

```javascript
const currentDate = new Date();
const formattedDateMoment = moment(currentDate).format('YYYY-MM-DD HH:mm:ss');
```


2021-03-23T16:43:32.010069453-05:00 -> YYYY-MM-DDTHH:mm:ss.SSSZ

```java
final ZonedDateTime parsed = ZonedDateTime.parse("2018-11-17T11:43:13-05:00[America/Toronto]");
```
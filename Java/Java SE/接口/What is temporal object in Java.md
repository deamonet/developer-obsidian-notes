I was exploring [`TemporalQuery`](http://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalQuery.html) and [`TemporalAccessor`](http://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalAccessor.html) introduced in Java 8. `TemporalAccessor` seems to be specially designed for temporal objects such as date, time, offset or some combination of these . What are temporal objects? Some googling gave its meaning as

> An object that changes over time

But not able to relate this in the context of Java?

- [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
- [java-8](https://stackoverflow.com/questions/tagged/java-8 "show questions tagged 'java-8'")

[Share](https://stackoverflow.com/q/28229721 "Short permalink to this question")

Follow

[edited Jan 30, 2015 at 5:30](https://stackoverflow.com/posts/28229721/revisions "show all edits to this post")

[

![5gon12eder's user avatar](https://i.stack.imgur.com/QF6FZ.jpg?s=64&g=1)

](https://stackoverflow.com/users/1392132/5gon12eder)

[5gon12eder](https://stackoverflow.com/users/1392132/5gon12eder)

24.3k55 gold badges4545 silver badges9292 bronze badges

asked Jan 30, 2015 at 5:24

[

![Omkar Shetkar's user avatar](https://i.stack.imgur.com/QM3hX.jpg?s=64&g=1)

](https://stackoverflow.com/users/1441374/omkar-shetkar)

[Omkar Shetkar](https://stackoverflow.com/users/1441374/omkar-shetkar)

3,48855 gold badges3535 silver badges5050 bronze badges

- 1
    
    Whilst not language specific, this might help you understand the concept: [dsc.ufcg.edu.br/~jacques/cursos/map/recursos/fowler-ap/…](http://www.dsc.ufcg.edu.br/~jacques/cursos/map/recursos/fowler-ap/Analysis%20Pattern%20Temporal%20Object.htm)
    
    – [Luke Peterson](https://stackoverflow.com/users/472798/luke-peterson "8,584 reputation")
    
    [Jan 30, 2015 at 5:29](https://stackoverflow.com/questions/28229721/what-is-temporal-object-in-java#comment44822238_28229721)
    

- 1
    
    Temporal just means "time related". java.time Objects (like their joda time counterparts) are immutable, so do not change. Ever.
    
    – [Mikkel Løkke](https://stackoverflow.com/users/1485064/mikkel-l%c3%b8kke "3,710 reputation")
    
    [Jan 30, 2015 at 6:38](https://stackoverflow.com/questions/28229721/what-is-temporal-object-in-java#comment44823457_28229721)
    

[Add a comment](https://stackoverflow.com/questions/28229721/what-is-temporal-object-in-java# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 2 Answers

Sorted by:

10

[](https://stackoverflow.com/posts/28229817/timeline)

According to with _Joda-Time & JSR 310, Problems, Concepts and Approaches_ [[**PDF**]](http://file.ithome.com.tw/20130806/A1-1330%20-%201415.pdf), the `TemporalAccessor`:

> Defines **read-only** access to a temporal object, such as a date, time, offset or some combination of these

The [_JSR-310 Date and Time API Guide_](https://jcp.org/aboutJava/communityprocess/pfd/jsr310/JSR-310-guide.html) states:

> Fields and units work together with the abstractions `Temporal` and `TemporalAccessor` provide access to date-time classes in a **generic manner**.

The book [_Beginning Java 8 Fundamentals: Language Syntax, Arrays, Data Types, Objects, and Regular Expressions_](http://www.apress.com/9781430266525) says:

> All classes in the API that specify some kind of date, time, or both are `TemporalAccesor`. `LocalDate`, `LocalTime`, `LocalDateTime`, and `ZoneDateTime` are some examples of `TemporalAccesor`.

The next is a example code (based from some examples in the previous book):

> ```java
> public static boolean isFriday13(TemporalAccessor ta) {
>     if (ta.isSupported(DAY_OF_MONTH) && ta.isSupported(DAY_OF_WEEK)) {
>         int dayOfMonth = ta.get(DAY_OF_MONTH);
>         int weekDay = ta.get(DAY_OF_WEEK);
>         DayOfWeek dayOfWeek = DayOfWeek.of(weekDay);
>         if (dayOfMonth == 13 && dayOfWeek == FRIDAY) {
>             return true;
>         }
>     }
>     return false;
> }
> 
> public static void main(String[] args) {
>     DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM/dd/yyyy");
>     TemporalAccessor ta = formatter.parse("02/13/2015");
>     LocalDate ld = LocalDate.from(ta);
>     System.out.println(ld);
>     System.out.println(isFriday13(ld));
> }
> ```
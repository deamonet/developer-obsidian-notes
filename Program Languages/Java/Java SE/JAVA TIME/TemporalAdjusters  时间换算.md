Common and useful TemporalAdjusters.

Adjusters are a key tool for modifying temporal objects. They exist to externalize the process of adjustment, permitting different approaches, as per the strategy design pattern. Examples might be an adjuster that sets the date avoiding weekends, or one that sets the date to the last day of the month.

There are two equivalent ways of using a `TemporalAdjuster`. The first is to invoke the method on the interface directly. The second is to use [`Temporal.with(TemporalAdjuster)`](https://docs.oracle.com/javase/8/docs/api/java/time/temporal/Temporal.html#with-java.time.temporal.TemporalAdjuster-):

   // these two lines are equivalent, but the second approach is recommended
   temporal = thisAdjuster.adjustInto(temporal);
   temporal = temporal.with(thisAdjuster);
 

It is recommended to use the second approach, `with(TemporalAdjuster)`, as it is a lot clearer to read in code.

This class contains a standard set of adjusters, available as static methods. These include:

- finding the first or last day of the month
- finding the first day of next month
- finding the first or last day of the year
- finding the first day of next year
- finding the first or last day-of-week within a month, such as "first Wednesday in June"
- finding the next or previous day-of-week, such as "next Thursday"

# 主要方法：

## dayOfWeekInMonth - 某个月的第几或者倒数第几周

### signature

public static [TemporalAdjuster](https://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalAdjuster.html "interface in java.time.temporal") dayOfWeekInMonth(int ordinal, [DayOfWeek](https://docs.oracle.com/javase/8/docs/api/java/time/DayOfWeek.html "enum in java.time") dayOfWeek)

Returns the day-of-week in month adjuster, which returns a new date in the same month with the ordinal day-of-week. This is used for expressions like the 'second Tuesday in March'.

The ISO calendar system behaves as follows:  
The input 2011-12-15 for (1,TUESDAY) will return 2011-12-06.  
The input 2011-12-15 for (2,TUESDAY) will return 2011-12-13.  
The input 2011-12-15 for (3,TUESDAY) will return 2011-12-20.  
The input 2011-12-15 for (4,TUESDAY) will return 2011-12-27.  
The input 2011-12-15 for (5,TUESDAY) will return 2012-01-03.  
The input 2011-12-15 for (-1,TUESDAY) will return 2011-12-27 (last in month).  
The input 2011-12-15 for (-4,TUESDAY) will return 2011-12-06 (3 weeks before last in month).  
The input 2011-12-15 for (-5,TUESDAY) will return 2011-11-29 (4 weeks before last in month).  
The input 2011-12-15 for (0,TUESDAY) will return 2011-11-29 (last in previous month).  

For a positive or zero ordinal, the algorithm is equivalent to finding the first day-of-week that matches within the month and then adding a number of weeks to it. For a negative ordinal, the algorithm is equivalent to finding the last day-of-week that matches within the month and then subtracting a number of weeks to it. The ordinal number of weeks is not validated and is interpreted leniently according to this algorithm. This definition means that an ordinal of zero finds the last matching day-of-week in the previous month.

The behavior is suitable for use with most calendar systems. It uses the `DAY_OF_WEEK` and `DAY_OF_MONTH` fields and the `DAYS` unit, and assumes a seven day week.

### Parameters:

`ordinal` - the week within the month, unbounded but typically from -5 to 5

`dayOfWeek` - the day-of-week, not null

### Returns:

the day-of-week in month adjuster, not null

### Example:

```java
// 表示这个月的最后一个星期四
int weekOfMonth = -1 
DayOfWeek dayOfWeek = DayOfWeek.THURSDAY;  
TemporalAdjuster temporalAdjuster = TemporalAdjusters.dayOfWeekInMonth(weekOfMonth, dayOfWeek);  
date = now.with(temporalAdjuster);
```
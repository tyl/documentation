## Default Search Patterns

The default implementation of the FilterFactory interface is called `DefaultFilterFactory` and supports the following filters:

### Strings

String-valued fields (`java.lang.String`) support the following patterns to generate a `SimpleStringFilter`:
    - `foo`: every string that starts with `foo`
    - `foo*`: same as above
    - `*foo*`: every string that contains `foo`
    - `*foo`: for a limitation in `SimpleStringFilter`, this is equivalent to `*foo*`.

### Numbers

Number-valued fields support Less, LessOrEqual, Greater, GreaterOrEqual, using the patterns: "<N", "<=N", ">N", ">=N". Where N is a number. For instance `>=10` on the field for property "age" will produce a `Compare.GreaterOrEqual("age", 10)`

Numeric ranges are also allowed using the syntax `N..M`. For instance `2..100` will look for all the numbers within the range `[2, 100]` (i.e., endpoints 2 and 100 included).

### Dates

Date-valued fields support range searches and matches.

- DD-MM-YYYY..dd-mm-yyyy finds all the dates within the interval. For instance 06-07-1986..02-02-2014 finds all the dates between Jul 6th, 1986 and Feb 2nd, 2014. (Format is currently not configurable). The date separator may be either - / or .
- a range of the kind AA-BBBB..CC-DDDD is interpreted as a range within 01-AA-BBBB and *max_day(CC,DDDD)*-CC-DDDD (where  *max_day(CC,DDDD)* is either 30, 31, 28 or 29, depending on the value of CC and DDDD)

- a range of the kind AA..BB is interpreted as AA-MM-YYYY..BB-MM-YYYY where MM is the current month, and YYYY is the current year
- a range of the kind AA-BB..CC-DD is interpreted as a range within the current year
- AAAA..BBBB is interpreted as a range from 01-01-AAAA and 31-12-BBBB

- MM-YYYY is interpreted as a shorthand for 01-MM-YYYY and *max_day(MM,YYYY)*-MM-YYYY
- DD-MM-YYYY find all the exact matches for the given date
- DD-MM finds all the exact matches for the given date, in the current year
- DD finds all the exact matches for the given date, in current month, year

#### Examples

Suppose that today is Feb 9, 2015:

Pattern                | Equivalent
---------------------- | ----------------------
10-10-2010..12-12-2014 |
10-2010..12-2014       | 01-10-2010..31-12-2014
10..12                 | 10-02-2015..12-02-2015
10-10..12-12           | 10-10-2015..12-12-2015
2010..2014             | 01-01-2010..31-12-2014
10-2010..12-2014       | 01-10-2010..31-12-2014
10-10-2010             |  
10-10                  | 10-10-2015
10                     | 10-02-2015


For every other value, an exact match (`Equal`) is attempted.

The implementation of these conversions can be found in the `DefaultFilterFactory`, which implements a `FilterFactory`. This factory is used by a `FilterApplier`, which in turn applies filters over a container in a `DataNavigation`. The `FilterApplier` class is used by the default listeners that are created with `DataNavigation.withDefaultBehavior()`.





### Lookup events (experimental)

   * ClearToFind
   * OnFind

The `DataNavigation` has experimental support for the *ClearToFind* and *Find* events. The ClearToFind event, "cleans" the fields of a FieldBinder for input, and makes it possible to perform a "search by example" (the same is obtained in a table using a pop-up window). These events can be attached using `ClearToFind.Listener` and `Find.Listener`, or both at once using `FindBehavior`.


![ClearToFind](http://i.imgur.com/AfrRFlT.png)

The default behavior for the ClearToFind event is to clear the fields when no filter has been applied; if a filter has been applied, the first click on the button will show the patterns that have been applied, and a second click will actually "clean" the search. Clicking the Find button will perform the search with the given criteria. For instance, in the example window of the picture, writing "Cor*" in the "First Name" field will find any Person whose name starts with the string "Cor"  

![Find](http://i.imgur.com/1ls24gW.png)

When a filter has been applied (that is, `Container.getContainerFilters().isEmpty()` is false), the optional `NavigationLabel` displays an asterisk near the count.

The user can input a textual pattern, which will be translated into a Vaadin `Container.Filter` automatically. Supported filters are currently:

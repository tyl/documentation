## Grid Support

Experimental Grid Support is already baked-in for Zoom fields. In order to opt-in for Grids invoke the `withGridSupport()` API

```java
final FieldBinder<Person> fieldBinder = 
       new FieldBinder<Person>(Person.class, mainContainer)
       .withGridSupport()
       .withDefaultBehavior();
```

Currently the flag only affects `ZoomField` generation. Soon, alternate implementations for `BeanTable` and `ListTable` will be provided. 


## FilterableGrid

The addon comes with an extension to Grid called FilterableGrid. The FilterableGrid uses auto-generated FilterPatternFields to automatically prepend search patterns to the Grid header.

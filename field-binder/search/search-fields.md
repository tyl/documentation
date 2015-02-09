## Search Fields

Custom Search Fields are provided to provide default search features.

Currently, two kinds of search fields are available 

* FilterPatternTextField
* FilterPatternComboBox

For instance, the TextField can be instanced with

```java
final FilterPatternTextField searchField = new FilterPatternTextField(propertyId, propertyType);
```


Both extend the abstract class `FilterPatternField`. A `FilterPatternField` returns a value that represents its pattern; upon request, it may also return a `SearchPattern` instance, using the method `getPatternFromValue()`.

A `SearchPattern` is a very simple value class with two fields:

```java
public class SearchPattern {
    private final Object objectPattern;
    private final Container.Filter filter;
}
```

the `objectPattern` represents is value of the field, the `filter` is the `Container.Filter` instance to which the pattern corresponds. The pattern is generated from the pattern using a `FilterFactory` instance (currently, this instance is the `DefaultFilterFactory`).

When the `getPatternFromValue()` method is invoked, the factory is invoked with the `propertyId`, the `propertyType` and the `value` of the field. 



An alternative constructor is also available:

```java
    public FilterPatternTextField(
    	Object propertyId, 
    	Class<?> propertyType, 
    	Container.Filterable targetContainer)
```
This constructor applies a pattern automatically on the given container, as the field changes. It is roughly equivalent to:

```java
searchField.getBackingField().addTextChangeListener((e) -> {
	targetContainer.addContainerFilter(searchField.getPatternFromValue().getFilter());
});

```


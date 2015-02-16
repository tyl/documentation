## Search Fields

Custom Search Fields are provided to provide default search features.

Currently, two kinds of search fields are available 

* SearchPatternTextField
* SearchPatternComboBox

For instance, the TextField can be instanced with

```java
final SearchPatternTextField searchField = new SearchPatternTextField(propertyId, propertyType);
```


Both extend the abstract class `SearchPatternField`. A `SearchPatternField` returns a value that represents its pattern; upon request, it may also return a `SearchPattern` instance, using the method `getPatternFromValue()`.

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
    public SearchPatternTextField(
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


## SearchFieldManager

The `SearchFieldManager` (SFM) keeps track of several search fields; it maps a propertyId onto a SearchField instance. 

The SFM is meant to hold a collection of fields that should be displayed when users want to perform a search (e.g., filtering a container). The fields in a SFM may be displayed in a custom form, or in a popup Window (using the SearchWindow). 

If you have written a managed form using the FieldBinder, the search fields can replace the fields in your form, using the `replaceFields()` method of the `FieldBinderSearchFieldManager`:

![replaceFields()](http://i.imgur.com/m0xhPEe.png)

This is the default implementation of the `clearToFind()` action of the Navigator. An alternative implementation is available, using a popup form.




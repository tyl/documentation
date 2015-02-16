## Search


The `DataNavigation` interface supports the `clearToFind()` and `find()`  methods to initiate and process a search on a data set. The FieldBinder Add-on implements a mechanism to present a search user interface automatically.
A managed form generated through the FieldBinder API can be commuted into "Search Mode". In the Search Mode, each Vaadin input field is *automatically* replaced by a `SearchField`.

The default implementation of the `clearToFind()` events replace a managed field into a `SearchPatternField` (see Section).

The picture shows a form for `Person` instances (see the tutorials). Each `Person` has `firstName`, `lastName`, `birthDate` properties. When the "Clear to Find" button is clicked, the search forms is transformed into a search form.

![replaceFields()](http://i.imgur.com/m0xhPEe.png)

The purpose of a search field is to let users input a (usually textual) *search pattern* that can be translated into a valid Vaadin `Container.Filter`. For instance users may input the text `Ge` into the `firstName` field; the default configuration maps the string `"Ge"` onto  Vaadin's `SimpleStringFilter` that filters all the first names that begin with the prefix `"Ge"`. The mapping between input strings and the generated filter is given through a `FilterFactory` (see [Filter Factory](filter-factory.md) and [Default Search Patterns](search-patterns.md)).




## SearchFieldManager

The default implementation of the `clearToFind()` action of the Navigator is to replace the managed fields of a FieldBinder with `SearchPatternField` instances (see class `org.tylproject.vaadin.addon.fieldbinder.behavior.commons.FieldBinders.Find`). The search fields are held in a `SearchFieldManager` instance (SFM); in particular, this is a `FieldBinderSearchFieldManager`. The generic `SearchFieldManager` may be used to design custom, possibly stand-alone search forms.

An example of a stand-alone search form is the `SearchForm` class, which is displayed in a `SearchWindow`. You can use this alternative search form by *replacing* the usual behavior configuration line:

```java
binder.getNavigation().withDefaultBehavior()
```

with the alternative configurations:
```java
binder.getNavigation()
    // enable SearchWindow
    .withFindListenersFrom(new SearchWindowFindListeners(binder))
    // ... other listener configurations ...;
```

See the documentation on event listeners for further information on how to customize
default event listeners.



## Search Fields

Currently, two types of search field are available

* SearchPatternTextField
* SearchPatternComboBox

Both extend the abstract class `SearchPatternField`. Search Fields can be instanced and placed in a layout manually to design custom search forms; for instance, the TextField can be instanced with

```java
final SearchPatternTextField searchField = new SearchPatternTextField(propertyId, propertyType);
```


When the `getPatternFromValue()` method is invoked, the *factory* is invoked with the `propertyId`, the `propertyType` and the `value` of the field.



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

However, the usual usage pattern is through a `SearchFieldManager` (see Section).

## Search Patterns

A `SearchPatternField` returns a value that *represents* the inserted pattern. For instance, the value of a search field where the user has input the text «hello» is the String `"hello"`. The method `getPatternFromValue()`  returns a `SearchPattern` instance, a class that contains the input pattern *and* the `Filter` to which that pattern corresponds.

The `SearchPattern` class is a very simple value class with two fields:

```java
public class SearchPattern {
    private final Object objectPattern;
    private final Container.Filter filter;
}
```

the `objectPattern` represents the value of the field, the `filter` is the `Container.Filter` instance to which the pattern corresponds. The Filter is generated from the pattern using a `FilterFactory` instance (currently, this instance is the `DefaultFilterFactory`).

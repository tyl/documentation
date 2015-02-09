## Collection-valued Tables

In Vaadin, a `Table` is a `Field` of type `AbstractSelect`; the `value` of a Vaadin `Table` is its selection. Thus, when you invoke `Table#getValue()` the returned value is a (possibly `null`) `Object` instance, representing the current selection.

The FieldBinder addon provides convenient Custom Field wrappers for Vaadin's table:

* CollectionTable
* ListTable
* BeanTable

The `value` for this wrappers is the `collection` of beans that the Table shows. These wrappers all belong to the same class hierarchy, with slight differences.

<dl>
 <dt> CollectionTable<T, U extends Collection<T>>
   <dd> A Table wrapper whose value is a generic collection of beans of type T. It is generated automatically by the field-binder to build "detail" views for non-`List` collections (usually `Set`s)
 
<dt> ListTable<T> 
   <dd> Almost equivalent to `CollectionTable<T, List<T>>` (it would be an alias if Java supported type aliases). 
   
<dt> BeanTable<T>
  <dd> A ListTable<T> implementation designed to be stand-alone. In order to use
  
</dl>


## Tabular Forms with `BeanTable<T>`

Because having forms with a lone Table is not uncommon, we also provide a stand-alone wrapper for a Table called a `BeanTable<T>`; it works similarly to the `ListTable<T>`, except it does not override the standard behavior of a table, with respect to selection. Methods `getValue()` and `setValue()` maintain their current semantics of "current selection". However, the `BeanTable` is a thin wrapper around Vaadin's Table which augment it with a `DataNavigation` instance, making it possible to obtain the same level of conciseness of the FieldBinder, but for a "Multi-occurrency"-type form:

```java
@Theme("valo")
public class TutorialTable extends UI {

  @Override
  protected void init(VaadinRequest request) {
    final Container.Ordered container = new ListTable<Person>(Person.class);

    final BeanTable<Person> table = new BeanTable<Person>(Person.class, container);
    table.setVisibleColumns("firstName", "lastName", "age", "birthDate");

    final ButtonBar bar = new ButtonBar(table.getNavigation()
                                             .withDefaultBehavior());

    final VerticalLayout mainLayout = new MVerticalLayout(bar, table)
                                      .withFullWidth()
                                      .withMargin(true);

    setContent(mainLayout);
  }
}

```


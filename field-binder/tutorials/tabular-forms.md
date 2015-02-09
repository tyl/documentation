

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


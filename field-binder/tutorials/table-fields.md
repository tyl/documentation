## Table Field Managements

Among the others, the most relevant difference between Grid and Table is that Table uses a *FieldFactory* to construct the Fields that will be used in *editing mode*;  Grid uses a *FieldGroup* instance that binds the propertyIds of the container to user-defined field instances. Grid's inline editing mode is also per-line, while Table is (by default) on all lines.

One problem with the Table approach is that Fields are re-generated each time the Table enters the edit mode via `Table.setEditable(true)`. This makes it hard to statically reference and bind fields from outside the context of a field factory, requiring users to write all the custom logic inside the factory.

For instance, consider the `Address(state, city)` entity. Suppose you want to show a ComboBox for states and a ComboBox for cities. With the Table, you would write:


```java
  myTable.setTableFieldFactory(new TableFieldFactory() {
        @Override
        public Field<?> createField(Container container, Object itemId, Object propertyId, Component uiContext) {
            if (itemId != fieldBinder.getNavigation().getCurrentItemId()) return null;
            if ("state".equals(propertyId)) {
              Field<?> state = ... ;
              return state;
            }
            if ("city".equals(propertyId)) {
              Field<?> city = ... ;
              return city;
            }
            ...
});
```

A typical action for one such pair of Comboboxes in your regular application is to pre-fill the "city" ComboBox with a list of cities in the currently selected state.


![Connected ComboBoxes](http://i.imgur.com/ZCThPgV.png)


In a simple form, you would do that by attaching events to the "state" field. However, using a simple TableFieldFactory, the "state" and "city" fields are destroyed and re-created each time editing mode is enabled on the Table.

However, FieldBinder makes this easy. Let's continue from [the Extended Tutorial](extended.md). Consider the `addressList` field:

```java
    final ListTable<Address> addressList =
            binder.buildListOf(Address.class, "addressList");
```

Each `ListTable` includes a `FieldBinder` that you can pull using the `getFieldBinder()` method.

```java
final FieldBinder<Address> addressListBinder = addressList.getFieldBinder();
```

You can now easily build `Field` instances and attach events: they will be automatically used in the Table. For instance, let us build a field for each column in the `addressList` `ListTable`:

```java
final TextField street  = addressListBinder.build("street"),
                zipCode = addressListBinder.build("zipCode");

// let us specify that we want ComboBoxes here:
final ComboBox state   = addressListBinder.build("State", "state", ComboBox.class);
final ComboBox city    = addressListBinder.build("City", "city", ComboBox.class);
```

Now it is possible to attach event listeners to fields in the usual Vaadin-way. For instance:

```java
  // add event listeners to the state field
  state.addItems(Arrays.asList("England", "Scotland", "Wales", "Northern Ireland"));
  state.addValueChangeListener(event -> {
      city.removeAllItems();
      if ("England".equals(event.getProperty().getValue())) {
          city.addItems(Arrays.asList("London", "Liverpool", "Oxford"));
          city.select("London");
      }
  });
```

You can also refer to Table fields within navigation event listeners:

```java
  addressList.getNavigation().addOnCommitListener(event ->
    Notification.show("The street was: "+street.getValue());
  );
```


[For more information on Collection Tables see the related tutorial](collection-tables.md).

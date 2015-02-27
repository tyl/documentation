# Tables and FieldBinder

CollectionTables are a FieldBinder wrapper for Vaadin's regular Table (and, in the future, for the grid). A CollectionTable includes:

* A Navigator instance
* Either a Table instance or a Grid instance

Because Vaadin has recently introduced the new Grid, each CollectionTable instance interacts with the Table/Grid component indirectly through an *adaptor* interface (TabularViewAdaptor, implemented by TableAdaptor or GridAdaptor).

## Differences between Grid and Table

Among the other, the most relevant differences between Grid and Table Table uses a *FieldFactory* to construct fields to show in editing mode;  Grid uses a *FieldGroup* instance that bounds the propertyIds of the container to user-defined field instances. Grid's inline editing mode is also per-line, while Table is (by default) on all lines.

One problem with the Table approach is that Fields are re-generated each time the Table enters the edit mode via `Table.setEditable(true)`. This makes it hard to statically reference and bind fields from outside the context of a field factory, requiring users to write all the custom logic inside the factory. 

For instance, consider the `Address(state, city)` entity. Suppose you want to show a ComboBox for states and a ComboBox for cities. With the Table, you would write:


```java
  myTable.setTableFieldFactory(new TableFieldFactory() {
        @Override
        public Field<?> createField(Container container, Object itemId, Object propertyId, Component uiContext) {
            if (itemId != fieldBinder.getNavigation().getCurrentItemId()) return null;
});``` 
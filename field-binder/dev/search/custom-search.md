## Customizing Search

Suppose you want to customize the default search experience for a FieldBinder form. 
the class `FieldBinders.Find<T>` uses a `FieldBinderSearchFieldManager` which
automatically infer and replaces fields in a FieldBinder with SearchPatternFields.

```java
    public static class Find<T> implements FindListeners {
       // ... 
		public Find(FieldBinder<T> binder) {
            this.binder = binder;
            this.searchFieldManager = new FieldBinderSearchFieldManager(binder);
        }
		 @Override
        public void clearToFind(ClearToFind.Event event) {
            event.getSource().setCurrentItemId(null);
            ...
            ((Container.Filterable)event.getSource().getContainer()).removeAllContainerFilters();
            searchFieldManager.replaceFields();
        }
        @Override
        public void onFind(OnFind.Event event) {
			  ...
            for (SearchPattern sp : searchFieldManager.getPatternsFromValues().values()) {
                ((Container.Filterable)event.getSource().getContainer())
                    .addContainerFilter(sp.getFilter());
            }
            searchFieldManager.restoreFields();
            binder.getNavigation().first();
        }
    }
}```


You are free to take inspiration and customize this class to your likings. For instance,
suppose that for a given form, you want to omit a field called "organizationName" from
the search feature.

You might, for instance, disable the search field in the `ClearToFind` event:

```java
public class CustomFind<T> extends FieldBinders.Find<T> {
	...
	@Override
    public void clearToFind(ClearToFind.Event event) {
      searchFieldManager.getSearchPatternField("organizationName").setEnabled(false);
      super.clearToFind(event);
    }}
```

The same can be done for the popup search dialog `SearchWindowFindListeners`, which is also used as the default Search UI for tables.


  
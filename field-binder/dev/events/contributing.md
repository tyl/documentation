## Contributing CRUD implementations

In our experience during the development of the add-on, many actions could be describe in terms of some base behavior. This was true both for non-tabular forms (FieldBinder) and for tabular forms (CollectionTables). Package `org.tylproject.vaadin.addon.fieldbinder.behavior.commons` contains utility classes that you may want to reuse in your own implementations. 

* `FieldBinders.BaseCrud` is the base class for CRUD in FieldBinders (e.g., `JPAContainerCrud` extends `FieldBinders.BaseCrud`).  
* `Tables.BaseCrud` is the base class for CRUD in tabular forms (e.g., `JPAContainerTableCrud` extends `Tables.BaseCrud`).

We advise you to follow this convention unless you have specific reasons not to do so. In case you have concerns, or you would like the base classes to change their behavior, please open an issue or a pull request so other container implementations may benefit from these changes.


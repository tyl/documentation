# Technical Walkthrough

In this document we will revisit the previous tutorials by giving more technical insight.

> **NOTE** in this document DataNavigation and BasicDataNavigation may be used interchangeably although `DataNavigation` is the interface
and `BasicDataNavigation` is the class. At some point they might be merged.

## Relation with FieldGroup
The FieldBinder uses a FieldGroup internally to which binding and unbinding is delegated when necessary.
This may change in the future.

A FieldGroup auto-generates and binds fields to an Item that functions as a data source. This implies that
an Item must be set (`FieldGroup.setItemDataSource()`) before users are allowed to invoke `FieldGroup.buildAndBind()`
or `FieldGroup.bind()`; otherwise the FieldGroup will throw an exception.

FieldBinder's constructors require a `Class` object so that propertyIds are known even though an `Item` data source has
**not** been set (yet), consequently `FieldBinder.build()` can be invoked at any time.

`FieldBinder.bind(field, propertyId)` currently **delegates** to `FieldGroup.bind(field, propertyId)`; however,
this happens **if and only if** `FieldGroup.getItemDataSource() != null`, which is **not** allowed for FieldGroup,
but is in FieldBinder.

If `field` has been bound to some property `p`, then  `FieldGroup.unbind(field)` causes `FieldGroup.getField(field) == null`.
In `FieldBinder`   `getFields(pid)` will **always** return the field that is bound to property `pid`, unless NO field has been bound to that property.


## FieldBinder creation

When a FieldBinder is created a Bean class must always be provided; optionally a Container
instance may be also provided. The Container instance is **required** if you want to use
the pre-defined container implementations (`withDefaultBehavior()`).

When a FieldBinder is created with a Container, a `DataNavigation` instance is automatically generated as well.
This object **cannot** be set manually to the FieldBinder, but can only be retrieved with
`getNavigation()` (DataNavigation instances can be created for using in your own code, though).

When the Navigation instance is created, a `BehaviorFactory` is also created and set to the Navigation instance.
The BehaviorFactory is the object responsible for making the `withDefaultBehavior()` method work:

```java
public interface BehaviorFactory<T> {
    Behavior forContainerType(@Nonnull Class<? extends Container> containerClass);
}
```


### Default Behaviors

The method `withDefaultBehavior()` delegates the creation of a collection
of pre-defined event listeners to the `BehaviorFactory`; then sets all the
listeners at once, using the shorthand method `withBehavior(behaviorClass)`:

```java
binder.getNavigation().withDefaultBehavior();
...
// BasicDataNavigation:
public BasicDataNavigation withDefaultBehavior() {
  ...
  Class<? extends Container.Ordered> containerClass = getContainerType();
  Behavior behavior = behaviorFactory.forContainerType(containerClass);
  return this.withBehavior(behavior);
  ...
}
```

The `BehaviorFactory` in the `BasicDataNavigation` class must create an object (of type `Behavior`) that implements
the Crud, Find, ItemChange listeners that describe the basic behavior of the Navigation
for the current container. For instance, CRUD for a JPAContainer requires invoking
different methods from those of a MongoContainer; therefore, implementations must be pluggable.

But also, the behavior for a Table (e.g., a detail view) differs from the behavior
of a non-tabular form; thus, even these case must be differentiated. There exist **two**
pre-defined BehaviorFactory implementations, which differ in that one, indeed, deals
with tabular forms (CollectionTables), and the other with non-tabular forms (just simple FieldBinders).

* `DefaultBehaviorFactory` for FieldBinders
* `DefaultTableBehaviorFactory` for CollectionTables (and subclasses)

When you **create** a FieldBinder instance, a `DefaultBehaviorFactory` is created, with
a reference to the FieldBinder instance itself:

```java
public FieldBinder(Class<T> beanClass, Container.Ordered container) {
   ...
   BasicDataNavigation nav = new BasicDataNavigation(container);
   nav.setBehaviorFactory(new DefaultBehaviorFactory<>(this));
   ...
}
```


When you create a class of the  CollectionTable hierarchy (manually, e.g., `new BeanTable(...)`
or automatically, via `fieldBinder.buildListOf(...)`), a `DefaultTableBehaviorFactory`
is created, with a reference to the current CollectionTable

> Caveat: currently, it is not really a CollectionTable, but a `TabularViewAdaptor`
> This may change in the future. The `TabularViewAdaptor` interface is used
> to adapt the Grid interface to the Table interface, until Grid will be
> stable enough to migrate the code base to it.
> Because of this, the initialization of the CollectionTable is a bit more involved
> and we are omitting it from here for the sake of simplicity.


The reason why a reference to the FieldBinder or the tabular widget is given
to the factory, is that, **the generated Behavior classes** MUST be handed this reference
as well: this is because the behavior implementations (especially, with respect to CRUD)
must be able to refer these components to interact with them.



## Building Fields


```java
final TextField firstName = binder.build("firstName");
final TextField lastName  = binder.build("lastName");
final DateField birthDate = binder.build("birthDate");
final TextField age       = binder.build("age");

final ListTable<Address> addressList = binder.buildListOf(Address.class, "addressList")
                                             .withDefaultEditorBar();
```

....


### Caveats

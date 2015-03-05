# Technical Walkthrough

In this document we will revisit the previous tutorials by giving more technical insight.

> **NOTE**: in this document, DataNavigation and BasicDataNavigation may be used interchangeably although `DataNavigation` is the interface
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


## Building Fields


```java
final TextField firstName = binder.build("firstName");
final TextField lastName  = binder.build("lastName");
final DateField birthDate = binder.build("birthDate");
final TextField age       = binder.build("age");

final ListTable<Address> addressList = binder.buildListOf(Address.class, "addressList")
                                             .withDefaultEditorBar();
```

Fields are automatically built using one of the several<sup id="build-methods"><a href="#fn-build-methods">†</a></sup> `build()` methods. Currently:

* `build(propertyId): T extends Field<?>`
* `build(caption, propertyId): T extends Field<?>`
* `build(propertyId, fieldType: Class<?>): T extends Field<?>`
* `build(caption, propertyId, fieldType: Class<?>): T extends Field<?>`

where `propertyId` is the property as it occurs in the given JavaBean class.
For instance, if `Person(firstName, lastName, birthDate, age, addressList)`
is a JavaBean, that is a class
where each `firstName`, `lastName` is represented through the pairs `getFirstName()`,
`setFirstName()` etc., then `firstName`, `lastName` etc. are called *properties*
of the Java Bean, and each `firstName`, `lastName` etc. is called a *property id*.

A *caption* is a string describing the field. If it is omitted, then it is **automatically
generated** according to some rules that will be described later.

A field type is a `Class<T extends Field<?>>`. E.g. `build("foo", ComboBox.class)` means
you want the FieldBinder to generate a ComboBox for property `foo`.

The **return type** of the `build()` method is automatically *casted* at *runtime*.
A side-effect is that *unsafe assignments* are allowed. In other words:

```java
final TextField date = fieldBinder.build("birthDate");
```

may throw a `ClassCastException` if the type of the `birthDate` property causes
the `fieldBinder` to generate a `DateField`. It is therefore advisable to pay attention to
the type of the variable to which you are assigning the return value of `build()`.

It might be safer sometimes to use the alternative version:

```java
final DateField date = fieldBinder.build("birthDate", DateField.class);
```
Please notice that, however, even this alternative cannot be guaranteed to succeed
if the given fieldType is not compatible with the given propertyId; e.g, consider:

```java
final DateField firstName = fieldBinder.build("firstName", DateField.class);
```
this will throw an exception, because property `firstName` is a `String`.
Actual generation of fields based on data types is delegated  a
`FieldBinderFieldFactory` instance, which is a subclass of Vaadin's
`FieldGroupFieldFactory` with a few customizations.


### Binding Fields and Unsupported Fields

You are not free *not* to use the `build()` command to auto-generate and
bind fields to a data source, but you will probably find more convenient to do so.

Sometimes, you might want to declare and instantiate fields manually, and *yet*,
get FieldBinder to manage their binding automatically. For instance, third-party
fields (e.g., third-party add-ons) cannot be automatically generated
by the FieldBinder, because FieldBinder is unaware of their existance.

The `bind()` command makes it possible to bind these fields to a datasource.

```java
bind(field: T extends Field<?>, propertyId): T
```

The particular method signature lets you use the idiom

```java
final MyCustomField customField = fieldBinder.bind(new MyCustomField(...), "somePropertyId");
```

**instead** of the classic:

```java
final MyCustomField customField = new MyCustomField(...);
fieldGroup.bind(customField, "somePropertyId")
```

(which you can still do, though.)


### Collection Tables
* `buildCollectionOf(containedBeanClass: Class<?>, propertyId): CollectionTable`
* `buildListOf(containedBeanClass: Class<?>, propertyId): ListTable`

These methods build a CollectionTable or a ListTable for detail views. The special method
signature is required because you have to specify the *type* of the bean that the collection
contains. In other words, if the `Person` bean contains an `addressList` of type `List<Address>`
FieldBinder needs you to specify the type of the contained class `Address.class`:

```java
ListTable<Address> addressList = fieldBinder.buildListOf(Address.class, "addressList");
```

This is a limitation due to Java's type erasure.

### Zoom and DrillDown Fields
* `buildZoomField(propertyId, containerPropertyId, zoomedContainer): TextZoomField`
* `buildDrillDownField(propertyId, containerPropertyId, zoomedContainer): TextZoomField`

These methods build `ZoomFields`. The special signature is required a proper
configuration of these fields requires more parameters than other simpler fields.

In particular, beside the `propertyId` the **value** of the field is **bound to**,
ZoomFields require:

* the `containerPropertyId` that is **displayed** in the field
* the *container* instance on which the zooming will occur.

For instance, suppose you have a property `licensePlateNumber` for a `CarOwner` class,
and you want to zoom over a collection of `Vehicles(licensePlate, model, color, ...)`.

* the `propertyId` is `licensePlateNumber`
* the `containerPropertyId` is `licensePlate`
* the `container` instance would be some datasource from where you pull license plate numbers

### Other Shorthands

* `buildAll(): Collection<Field<?>>`
* `buildAll(propertyIds...): Collection<Field<?>>`
* `buildAll(propertyIdCollection): Collection<Field<?>>`


These methods build all the given property ids. In the first case, all the known
properties of the given Bean Class will be used.



### Caption Generation


## Default Behaviors

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
with tabular forms (CollectionTables <a id="collectiontables" href="#fn-collectiontables"> * </a>), and the other with non-tabular forms (just simple FieldBinders).

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


The reason why a reference to the FieldBinder or the tabular widget is given
to the factory, is that, **the generated Behavior classes** MUST be handed this reference
as well: this is because the behavior implementations (especially, with respect to CRUD)
must be able to refer these components to interact with them.

This is an implementation detail that may change in the future.



### Caveats

---------------------------------------------------------------------------------

#### Footnotes

<div id="fn-collectiontables">
* Caveat: currently, it is not really a CollectionTable, but a `TabularViewAdaptor`
This may change in the future. The `TabularViewAdaptor` interface is used
to adapt the Grid interface to the Table interface, until Grid will be
stable enough to migrate the code base to it.
Because of this, the initialization of the CollectionTable is a bit more involved
and we are omitting it from here for the sake of simplicity. [↩](#collectiontables)
</div>


<div id="fn-build-methods">
† This is basically because of the lack of default values in Java methods. This in, say, Scala or Kotlin, may be equivalently written as
`build(caption = null, propertyId: Any, fieldType: Class<?> = Field.class): Field<?>`
[↩](#build-methods)


</div>

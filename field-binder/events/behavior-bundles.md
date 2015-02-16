# Predefined Behavior Bundles

The FieldBinder add-on comes with a variety of pre-defined listener implementations
so that you have to write as little code as possible. These pre-defined implementations
are all find in the subpackages of `org.tylproject.vaadin.addon.fieldbinder.behavior`:

	  org.tylproject.vaadin.addon.fieldbinder.behavior
			|
			├── commons
			│   ├── FieldBinders.java
			│   ├── SearchWindowFindListeners.java
			│   └── Tables.java
			└── containers
			    ├── jpacontainer
			    │   ├── JPAContainerCrud.java
			    │   └── JPAContainerTableCrud.java
			    ├── listcontainer
			    │   ├── ListContainerCrud.java
			    │   └── ListContainerTableCrud.java
			    └── mongocontainer
			        ├── BufferedMongoTableCrud.java
			        └── MongoCrud.java

For instance, suppose you want to manually configure the behavior of a JPA-backed FieldBinder, using a JPAContainer, and you want the built-in search feature to display a popup window. You may write:



```java
JPAContainer<Person> container =
	JPAContainerFactory.makeContainer(Person.class, "my-persistence-unit");
FieldBinder<Person> binder = new FieldBinder<Person>(Person.class);
...
binder.getNavigation()
	.withCurrentItemChangeListenerFrom(new FieldBinders.CurrentItemChange(binder))
	.withCrudListenersFrom(new JPAContainerCrud(container))
	.withFindListenersFrom(new SearchWindowFindListeners(binder));
```


## BehaviorFacade

An alternative way to provide several listeners at once is through the `BehaviorFacade` class, which implements the `Behavior` interface  (see [DataNavigation Events](datanavigation-events.md)) by delegating all the methods to 3 objects that must be provided in its constructor:

	* an implementation of `CurrentItemChange.Listener`
	* an implementation of `CrudListeners`
	* an implementation of `FindListeners`

Using the `BehaviorFacade` class the configuration above becomes:

```java
Behavior listeners = new BehaviorFacade(
	new FieldBinders.CurrentItemChange(binder),
	new JPAContainerCrud(container),
	new SearchWindowFindListeners(binder)
);
binder.getNavigation().withBehavior(listeners);
```

Although from a behavioral stand-point this is **completely equivalent** to invoking the `with*` methods described in the previous section, the `BehaviorFacade` class is internally used by the [`BehaviorFactory`](predefined-behavior.md) to configure pre-defined listeners for the known container types.

In fact, when you invoke ```binder.getNavigation().withDefaultBehavior()``` the default factory implementation in fact generates a suitable configuration of a `BehaviorFacade` instance.


## Commons

FieldBinders and Tables contain default behavior for simple forms (FieldBinder-based) or tabular forms (Table-based—grid support is under development).

### FieldBinders

* `FieldBinders.CurrentItemChangeListener`. Updates the contents of the fields when the current item of the navigation changes (e.g, on first(), next(), etc.)
* `FieldBinders.BaseCrud`. Basic behavior for CRUD. This is only meant as a base class to be extended by more container-specific listeners. You can use it to define your own container-specific implementations, when one is not provided by this add-on.
* `FieldBinders.Find`. Default find behavior: it replaces the visible fields with `SearchPatternFields` up until the `Find` event is fired (see the [Search](../search) section)

### Tables

* `Tables.CurrentItemChangeListener`. Updates the current selection in the table when the current item of the navigation changes.
* `Tables.BaseCrud`. Basic behavior for table-based CRUD. Again, this is only meant as a base class to be extended by more container-specific listeners.

Default find behavior for Tables is to use a popup window. This is provided by a separate component, since you might want to use search windows with a FieldBinder as well as tables.

### Search Window

* `SearchWindowFindListeners`. Find listeners to display a popup search form. It internally uses a `SearchFieldManager`  (see the [Search](../search) section).


## Containers

Supported containers are all bean/entity-based:

* ListContainer
* JPAContainer
* MongoContainer

You can design your own classes for custom containers. the `containers` packages contains one package for each type of container, with variants, depending whether the CRUD listeners are designed to be used with a FieldBinder-based form, or a table.

Container Type | FieldBinder CRUD   | Table CRUD
---------------|--------------------|----------------------------
ListContainer  | ListContainerCrud  | ListContainerTableCrud
JPAContainer   | JPAContainerCrud   | JPAContainerTableCrud
MongoContainer | MongoCrud          | BufferedMongoContainerCrud

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

* `FieldBinders.CurrentItemChangeListener`
* `FieldBinders.BaseCrud`
* `FieldBinders.Find`

### Tables

* `Tables.CurrentItemChangeListener`
* `Tables.BaseCrud`

### Search Window

* `SearchWindowFindListeners`


## Containers

Supported containers are all bean/entity-based:

* JPAContainer
* ListContainer
* MongoContainer

You can design your own classes for custom containers.

## Loading Default Listeners


For instance, suppose you want a JPAContainer-backed FieldBinder to display a search window. You will write:

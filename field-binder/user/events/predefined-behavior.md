## Default Behavior Loading

The `DataNavigation.withDefaultBehavior()` method tries to guess the best predefined set of listeners for the container that the `DataNavigation` is currently pointing to.

We will now outline how the resolving mechanism works. 

The `Behavior` interface groups the following interfaces:

* `CrudBehavior` which extends:
	* ItemCreate.Listener
	* ItemEdit.Listener
	* ItemRemove.Listener
	* AfterCommit.Listener, OnCommit.Listener, BeforeCommit.Listener
	* OnDiscard.Listener
	
* `FindBehavior` which extends:
	* ClearToFind.Listener
	* OnFind.Listener
* `CurrentItemChange.Listener`


The `BehaviorFactory` interface (default implementation: `DefaultBehaviorFactory`) tries to guess which collection of built-in listeners suits better the current configuration, depending on whether the Navigation controls a FieldBinder or a ListTable, and depending on which container type the navigation is currently wrapping.

When the `DataNavigation.withDefaultBehavior()` method is invoked, the `BehaviorFactory` is given the current container (`DataNavigation.getContainer()`); if applicable, the factory returns an instance of the `Behavior` interface

### Examples
> The `DataNavigation` controls a `FieldBinder`; then the factory is usually a `FieldBinderBehaviorFactory`; when it controls a `ListTable`, then this factory is usually a `TableBehaviorFactory`. In the case of this tutorial, your FieldBinder was controlled by a DataNavigation connected to a `FilterableListContainer`; thus the collection of listeners is taken from the `ListContainerBehavior` class. 

Therefore, in order for the `withDefaultBehavior()` to work, the `DataNavigation` must be  able to know what type of container it is bound to. This is obvious if `getContainer() != null`. However, sometimes you might not be able to `setContainer()` before `withDefaultBehavior()` is called. For instance, the container you want to set might be generated as a consequence of user interaction.
In these cases, you might want to declare the container type, without passing a real container instance.


## Declaring a Container type (deprecated in v1.4)

If the DataNavigation is *not* bound to a Container instance when the ``.withDefaultBehavior()` the DataNavigation may throw an `IllegalStateException`; it is possible to avoid this situation by declaring beforehand the *expected* type of the container that the Navigation should hold.

For instance, suppose you instance a BasicDataNavigation that will host a `BeanContainer`, but the BeanContainer is not known yet (e.g., it will be constructed/generated later). You can declare this intention using the method 

```java
restrictContainerType(Class<? extends Container.Ordered> type)
```

now the `withDefaultBehavior()` method is able to pass the information that the `BehaviorFactory` requires to construct a `Behavior` instance.

Notice that the restriction also introduces a validity check: from now on, any container instance that you will set, must be a *subclass* of the given `type` (in the sense of `type.isAssignableFrom(newContainerClass)`)
# The DataNavigation interface and the Event System

The `DataNavigation` interface implements navigation, CRUD (and experimentally) lookup methods for scanning through a Container. For instance, you can point to the first record

```java
      binder.getNavigation().first()
```

and then you can remove it

```java
      binder.getNavigation().remove()
```

The `DataNavigation` object wraps a `Container.Ordered` instance, and it maintains a pointer to the `currentItemId`. The `currentItemId` is `null` only when the `Container` is empty (`Container.size() == 0`) or when no container has been associated to the Navigator.


The `DataNavigation` interface defines several events that you can listen to.


### Navigation Events
	* FirstItem
	* NextItem
	* PrevItem
	* LastItem
	* CurrentItemChange


Because the DataNavigation object maintains an internal state, that is, a pointer to the "current" item id, for each other navigation event,  the `CurrentItemChange` event always fires. Therefore, if you want to hook into *every* navigation event, then you should listen to `CurrentItemChange`.

### CRUD events
	* ItemCreate
	* ItemEdit
	* ItemRemove
	* AfterCommit, OnCommit, BeforeCommit
	* OnDiscard



For instance, in order to listen to the `CurrentItemChange` event on the `binder` component use:

```java
binder.getNavigation().addCurrentItemChangeListener(new CurrentItemChange.Listener(){
    public void currentItemChange(CurrentItemChange.Event event) {
       // display the updated current itemId in a notification
       Notification.show(event.getNewItemId());
    }
});
```

There is also a shorthand interface `CrudListeners` that implements all of the following listeners: `ItemCreate`, `ItemEdit`, `ItemRemove`, `OnCommit`, `OnDiscard`. If you need all of them you can just write `class MyController implements CrudListeners`. You can also tell a navigator to use all of the methods from a `CrudListeners` at once using:

```java
   navigation.withCrudListenersFrom(...)
```



### Lookup events

   * ClearToFind
   * Find

The `DataNavigation` supports the *ClearToFind* and *Find* events. The ClearToFind event, "cleans" the fields of a FieldBinder for input, and makes it possible to perform a "search by example" (the same is obtained in a table using a pop-up window). These events can be attached using `ClearToFind.Listener` and `Find.Listener`, or both at once using `FindBehavior`.


![ClearToFind](http://i.imgur.com/AfrRFlT.png)

The default behavior for the ClearToFind event is to clear the fields when no filter has been applied; if a filter has been applied, the first click on the button will show the patterns that have been applied, and a second click will actually "clean" the search. Clicking the Find button will perform the search with the given criteria. For instance, in the example window of the picture, writing "Cor*" in the "First Name" field will find any Person whose name starts with the string "Cor"  

![Find](http://i.imgur.com/1ls24gW.png)

When a filter has been applied (that is, `Container.getContainerFilters().isEmpty()` is false), the optional `NavigationLabel` displays an asterisk near the count.

The user can input a textual pattern, which will be translated into a Vaadin `Container.Filter` automatically. For more information of filters see the [search](../search) section of the documentation.

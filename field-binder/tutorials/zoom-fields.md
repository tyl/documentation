## Zoom and DrillDown Fields

A zoom field is a selection field with the same objective as a ComboBox, but with extended capabilities. When the user clicks the Zoom field, instead of a menu, a popup window is shown. Users may select an item from the popup window. 


![Zoom using a FilterableGrid](https://cloud.githubusercontent.com/assets/380791/5934173/ad9809b2-a6cb-11e4-9237-38a6334c667a.png)

### Example

Suppose you have a collection of `Person` objects, and that each object has a `Role` field

```java
public class Role {
	String name;
	...
}
```

You may build a Zoom field to select the role for each person. The Zoom field will open a table view that shows all the details of a `Role` collection. The collecton of roles should be accessible through a `container` (if it is a simple collection, you may use a `ListContainer`).

```java
final Container.Indexed mainContainer = ... ;
final FieldBinder<Person> fieldBinder = 
       new FieldBinder<Person>(Person.class, mainContainer).withDefaultBehavior();

final Container.Indexed zoomContainer = ... ;
final TextZoomField zoomField = 
       fieldBinder.buildZoomField(
            "role", "name", zoomContainer);
```

The auto-generated ZoomField is already pre-configured with a default `ZoomDialog` implementation. The default is to use a Vaadin table, but Grid support is already baked in.


ZoomFields can be instanced manually with `new TextZoomField()`. In this case the `ZoomDialog` has to be given explicitly:

```java
final ZoomDialog zoomDialog = /* instance of a zoomDialog impl. */ ; 
final TextZoomField zoomField = new TextZoomField();
zoomField.withZoomDialog(zoomDialog);
```

Available ZoomDialog implementations are `TableZoomDialog` and `GridZoomDialog`:

```java
final ZoomDialog zd1 = new TableZoomDialog(propertyId, zoomContainer);
final ZoomDialog zd2 = new GridZoomDialog(propertyId, zoomContainer);
```

The GridZoomDialog requires Vaadin 7.4+ and it supports extra filtering capabilities.

Users may supply their own implementation of the `ZoomDialog` interface.



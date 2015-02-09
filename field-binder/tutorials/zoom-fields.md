## Zoom and DrillDown Fields

A zoom field is a selection field with the same objective as a ComboBox, but with extended capabilities.

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
final FieldBinder fieldBinder = 
       new FieldBinder(mainContainer).withDefaultBehavior();

final Container.Indexed zoomContainer = ... ;
final TextZoomField zoomField = 
       fieldBinder.buildZoomField(
            "role", "name", zoomContainer);
```


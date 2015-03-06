## Custom Zoom Dialogs

The default behavior for ZoomFields (and DrillDowns) is to display a popup window on the click of the Zoom/DrillDown button.
The popup window contents is customizable by implementing the `ZoomDialog` interface. A common base class that you can use
in your own implementations is `AbstractZoomDialog`.

The pre-defined implementations are

  * `TableZoomDialog` (non-filterable)
  * `GridZoomDialog` which internally uses `FilterableGrid`, a simple extension to `Grid` that displays 
    an auto-generated `SearchPatternField` at the top of each column in the grid. In this case, SearchPatternFields
    apply filters as you type
    
You are free to provide your own implementation of a ZoomDialog and then configure your ZoomField:

```java
myTextZoomField.withZoomDialog(new MyCustomZoomDialog())
```

If you want to add custom logic to an existing Zoom dialog, it is advisable to implement your extensions 
in a custom class (e.g. `class MyCustomZoomDialog extends GridZoomDialog { ... }`). If your extensions
are very simple, you can also tell Java to override methods inline. Most often, in fact, you may probably
want to override the behavior of the ZoomDialog when the `dismiss()` or `show()` methods. For instance:

```java
myTextZoomField.withZoomDialog(new GridZoomDialog(...) {
  @Override
  public void show(Object value) {
	super.show(value);  	
    // your custom code  }}
```
 

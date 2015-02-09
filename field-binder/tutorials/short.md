
## Short Tutorial
We will create a simple Address Book, where each `Person` may have many `Address`es. 
For conciseness, we will also use the [Viritin](https://github.com/viritin/viritin) addon, its `ListContainer` and the shorthand classes for Layouts. You do not need to add any further dependencies to your `pom`, though. The `field-binder` add-on depends on `Viritin` already. Now, let us write the following bean class for the Addresses:

```java
public class Address {
    private String street;
    private String zipCode;
    private String city;
    private String state;
    public Address() {}
    /* ...getters and setters... */
}
```

and the Person entity:

```java
public class Person {
   private String firstName;
   private String lastName; 
   private Integer age;
   private Date birthDate;
   private List<Address> addressList = new ArrayList<Address>();
   public Person() {}
   public Person(String firstName, String lastName) {
     this.firstName = firstName;
     this.lastName = lastName;
   }
   
   /* ...getters and setters... */
}
```

Now you are ready to define your Vaadin `UI`

```java
@Title("Short Tutorial")
@Theme("valo")
public class ShortTutorial extends UI {
  
  // CONTAINER
  
  // initialize an empty container
  final FilterableListContainer<Person> container = 
                                    new FilterableListContainer<Person>(Person.class);
  
  // FIELD BINDER 
  // initialize the FieldBinder for the given container
  final FieldBinder<Person> binder = new FieldBinder<Person>(Person.class, container);

  // initialize the layout, building the fields at the same time
  final VerticalLayout mainLayout = new MVerticalLayout(

      // auto-generates a button bar with the appropriate behavior
      // for the underlying FilterableListContainer
      new ButtonBar(binder.getNavigation().withDefaultBehavior()),

      new MFormLayout(
          // automatically generate a Field from the property type
          binder.build("firstName"),
          binder.build("lastName"),
          binder.build("birthDate"),
          binder.build("age"),
          
          // optional: display the index of the currentItem in the container
          new NavigationLabel(binder.getNavigation()) 

      ).withFullWidth().withMargin(true),

      // initialize the addressList field with a built-in button bar
      binder.buildListOf(Address.class, "addressList").withDefaultEditorBar()

  ).withFullWidth().withMargin(true);


  @Override
  protected void init(VaadinRequest request) {
    setContent(mainLayout);
  }

}
```

Start the Vaadin application: you have already a fully-functional master/detail editor in about 10 lines of actual code! 

Keep reading to know what happened under the hood, or skip to the next section for the extended tutorial.

### What You Have Done

Most of the code you have written is your regular Vaadin UI code. You only wrote about 10 lines of code. But has happened under the hood?  Let us have a look at the `FieldBinder`-specific code.

As you will see, you wrote very little, because a lot has been done already on your behalf!


#### Automatic Binding to a data source 

```java
  final FieldBinder<Person> binder = new FieldBinder<Person>(Person.class, container);
```

With this code you are generating a FieldBinder for a `Person` bean. The *optional* container parameter tells the FieldBinder that you want to use the automatic navigation support. The `container` parameter is completely optional, and if you omit it, the `FieldBinder` will continue to work as an improved `FieldGroup`. In this case you must use the alternate constructor:

```java
  final FieldBinder<Person> binder = new FieldBinder<Person>(Person.class);
```

#### Automatic `Field` generation

Similarly to the `FieldGroup`, you can automatically `build()` fields for a given `Person` property. The `FieldBinder` will automatically infer the type of field. In our `Person` bean we have defined the properties `firstName`, `lastName`, `birthDate` (we will get back the `addressList` in a moment). Then we can write:

```java
          binder.build("firstName"),
          binder.build("lastName"),
          binder.build("birthDate"),
          binder.build("age"),       
```

More `build()` methods are available, along the lines of Vaadin's `FieldGroup`:

  * `build(Object propertyId)`
  * `build(String caption, Object propertyId)`
  * `build(String caption, Object propertyId, Class<T> fieldType)`

Refer to the JavaDoc and Vaadin's documentation to get more detailed information


Fields for `List` types must be generated using the method

```java
      binder.buildListOf(Address.class, "addressList")
```

the first argument is the type of the elements contained in the `List`. In our case, these are `Address`es; the second argument is the name of the property the contains the `List<Address>`, which, in our `Person` bean is called `addressList`. This method generates  `ListTable<Address>`, which is a thin wrapper around Vaadin's standard `Table`. The `ListTable<Address>` is a `Field` that differs slightly from Vaadin's `Table`. In  Vaadin's `Table`, `getValue()` returns the currently selected element, and `setValue(Object)` changes the currently selected item. In the `ListTable`, the  "value" is the *list of the values contained in the underlying Table*.

`ListTable<T>` uses Viritin's `ListContainer<T>` internally.


#### `DataNavigation` #####

Each `FieldBinder` that has been created with a `container` instance includes a `DataNavigation` instance that you can get using:

```java
      DataNavigation nav = binder.getNavigation()
```

The `DataNavigation` acts as a controller for scanning through a `Container`. For instance, you can use `nav.first()` to make the FieldBinder point to the first Item in the Container. The `DataNavigation` instance also fires events for each command that it defines. For instance, `nav.first()` issues a `FirstItem.Event` that you can listen to using `nav.addFirstItemListener(FirstItem.Listener listener)`. 

The `DataNavigation.withDefaultBehavior()` tries to guess the best predefined set of listeners for the container that you are currently using. In the case of this tutorial, you are using a `FilterableListContainer`. 

You are not required to delve into the details of this right now, but you can read more in the latter part of this document.

Each `ListTable` is also bound to a `DataNavigation` instance, which is kept in sync with the actual underlying `Table` selection (in other words, when you click an item in the `Table` that the `ListTable` wraps, the `currentItemId` of the `DataNavigation` instance is updated accordingly.



#### `ButtonBar` generation 

A `ButtonBar` instance must be attached to a DataNavigation instance, so it is enough to say:

```java
      DataNavigation nav = binder.getNavigation();
      ButtonBar buttonBar = new ButtonBar(nav);
```

or, on one line:

```java
      new ButtonBar(binder.getNavigation().withDefaultBehavior()),
```

The same can be done both for the navigation of a FieldBinder and the navigation of a ListTable. However, since the `Table` is also a Vaadin `Component` you can also make it generate an embedded button bar that is part of the `ListTable` itself. In this case you can write, as in the tutorial:

```java
      binder.buildListOf(Address.class, "addressList").withDefaultEditorBar()
```

Otherwise, you can create a button bar using code similar to the FieldBinder's:

 
```java
ListTable<Address> addressList = 
            binder.buildListOf(Address.class, "addressList");
ButtonBar addressListBar = new ButtonBar(addressList.getNavigation()
                                                    .withDefaultBehavior()),
```


The `ButtonBar` is a compound component that contains three distinct bars:

* `NavButtonBar`
* `CrudButtonBar`
* `FindButtonBar`

You can also choose to pick only one; for instance, if you just want the buttons for CRUD on the `addressList`:

```java
CrudButtonBar addressListBar = 
       new CrudButtonBar(addressList.getNavigation().withDefaultBehavior()),
```

#### NavigationLabel

This optional component shows the index of the current item and the total count of the items in the container that the `DataNavigation` is controlling.

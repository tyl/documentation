

## Extended Tutorial

In this version of the tutorial we will extend the regular editing behavior of the `DataNavigation` component with custom logic, using the built-in event listener system.

Let us start from the previous tutorial, in which we built a `Person` editor, with a detail view of the `Address` list. Each `Person` of the previous example contained an `age` field. As you have probably noticed, the `age` field is actually a function of the `birthDate` field: `age` is the difference in years between the current date and the `birthDate` of a `Person`. Let us implement a custom `BeforeCommit` listener that will update the `age` field with a computed value.
 
In order to simplify the writing of the custom events, it is advisable to assign the generated Fields to instance variables of the class; so, refactor them out as follows:

```java
  final TextField firstName = binder.build("firstName");
  final TextField lastName  = binder.build("lastName");
  final DateField birthDate = binder.build("birthDate");
  final TextField age       = binder.build("age");
  
  final ListTable<Address> addressList = binder.buildListOf(Address.class, "addressList")
                                               .withDefaultEditorBar();
```   

Then, the layout should be updated accordingly:

```java
  // initialize the layout
  final VerticalLayout mainLayout = new MVerticalLayout(

      new ButtonBar(binder.getNavigation().withDefaultBehavior()),

      new MFormLayout(
          firstName, lastName, birthDate, age,
          new NavigationLabel(binder.getNavigation())
      ).withFullWidth().withMargin(true),

      addressList

  ).withFullWidth().withMargin(true);
```


Now, let us write the custom actions as event listener. For convenience, we may define a separate class. 

```java

class MyController implements BeforeCommit.Listener {
  @Override
  public void beforeCommit(BeforeCommit.Event event) {
    // e.g., using JodaTime:
    DateTime birthDateValue = new DateTime(birthDate.getValue());
    int ageValue = Years.yearsBetween(birthDateValue, DateTime.now()).getYears();

    age.setConvertedValue(ageValue);
  } 
}

```

then, let us add the event listener to the `DataNavigation` of the `FieldBinder`

```java
  @Override
  protected void init(VaadinRequest request) {
    setContent(mainLayout);
    
    DataNavigation dataNav = binder.getNavigation();
    MyController controller = new MyController();
    dataNav.addBeforeCommitListener(controller);
  }
```

Start the application, and you're set!

### Final Touches

Since `age` is now a computed field you may want to set the *age* field to read-only to prevent users from modifying it. In this case, you may want to hook into the `ItemEdit` and `ItemCreate` events, and update the `BeforeCommit` listener accordingly:

```java
class MyController implements ItemEdit.Listener, ItemCreate.Listener, BeforeCommit.Listener {

  @Override
  public void itemEdit(ItemEdit.Event event) {
    age.setReadOnly(true);
  }

  @Override
  public void itemCreate(ItemCreate.Event event) {
    age.setReadOnly(true);
  }

  ...
  @Override
  public void beforeCommit(BeforeCommit.Event event) {
    // e.g., using JodaTime:
    DateTime birthDateValue = new DateTime(birthDate.getValue());
    int ageValue = Years.yearsBetween(birthDateValue, DateTime.now()).getYears();

    age.setReadOnly(false);
    age.setConvertedValue(ageValue);
    age.setReadOnly(true);
      
  }
}
```

don't forget to register the new event listeners:

```java
  @Override
  protected void init(VaadinRequest request) {
    setContent(mainLayout);
    DataNavigation dataNav = binder.getNavigation();

    MyController controller = new MyController();

    dataNav.addItemEditListener(controller);
    dataNav.addItemCreateListener(controller);
    dataNav.addBeforeCommitListener(controller);
  }
```

Now restart the application and see the result. In the next section we will give more details on the architecture of the FieldBinder and the related component. In particular, we will focus on the `DataNavigation` component. 



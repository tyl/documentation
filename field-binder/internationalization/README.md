# Internationalization

Instantiate FieldBinder with the ResourceBundle option:

```java
final FieldBinder<Person> binder = new FieldBinder<>(Person.class).withResourceBundle(...);
```

Each new Field caption will be configured automatically if the resource bundle contains a key that matches the given propertyId. For instance, suppose you create a simple `ResourceBundle` class : 

```java
package com.example;
public class MyFormStrings extends ListResourceBundle {
	private static final String[][] STRINGS = {
	    { "firstName", "Nome" } // "Nome" is "firstName" in Italian
	};
	@Override
	protected Object[][] getContents() {
		return STRINGS;
	}
};

```

or a `.properties` file

```
# com.example.MyFormStrings.properties
firstName = Nome
```

Then you may create the `firstName` as usual, with:


```java
final ResourceBundle resourceBundle = ResourceBundle.getBundle("MyFormStrings")
final FieldBinder<Person> binder = new FieldBinder<>(Person.class).withResourceBundle(...);
final TextField firstName = binder.build("firstName");
```

The caption for the `firstName` field will be "Nome".

![Translated firstName field](http://i.imgur.com/G7XKzzR.png)

**N.B.:** ResourceBundles are automatically propagated to the CollectionTables. Thus, if `Person` contains a `List` of `Address(street, city)`, you'll be able to write:

 
```java
	private static final String[][] STRINGS = {
	    { "firstName", "Nome" },
		{ "street", "Strada" },
		{ "city", "Città" }
	};
```

or, for example, using `properties`:

```# com.example.MyFormStrings.properties
firstName = Nome
street = Strada
city = Città
```

The columns in the table will be localized

![Translated Detail Fields](http://i.imgur.com/PrRq7oe.png)

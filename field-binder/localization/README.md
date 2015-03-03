# Internationalization



## Default Button Bars

Default button bars support internationalization through ResourceBundles as well. The default language bundle is implemented by class `org.tylproject.vaadin.addon.datanav.resources.Strings` you are free to add your own locales using the rules defined by the JDK [ResourceBundle](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html) API

For instance, you might add `org.tylproject.vaadin.addon.datanav.resources.Strings_it` for Italian strings, `org.tylproject.vaadin.addon.datanav.resources.Strings_es` for Spanish, and so on.


## Field Captions

Captions of the generated fields can be set as usual with `Field#setCaption()` method. When they are not specified explicitly at build-time — e.g., using FieldBinder APIs such as `FieldBinder.build(caption, propertyId)` — they are usually generated automatically using Vaadin's default utility method (`SharedUtil.propertyIdToHumanFriendly(propertyId)`), which splits and capitalizes a camelCased identifier. For instance `firstName` becomes "First Name". This is good for quick and dirty form creation, but insufficient for larger applications.

You can instantiate FieldBinder with the ResourceBundle option. When this option is activated, the FieldBinder, for each Field that you `build(propertyId)` will look  in the ResourceBundle for the given `propertyId` and assign the corresponding value as a caption. If the propertyId were missing in the ResourceBundle, then it will fall back again onto the original "automatic" behavior.

To instantiate a FieldBinder with the ResourceBundle option activated use:

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
final TextField lastName = binder.build("lastName");
```

The caption for the `firstName` field will be "Nome". The caption for `lastName` will be "Last Name": the string is missing from the ResourceBundle, thus causing the FieldBinder to fall back onto the usual behavior of inferring the caption from the camelcased propertyId.

![Translated firstName field](http://i.imgur.com/G7XKzzR.png)

### Collection Tables

ResourceBundles are automatically propagated to the CollectionTables. Thus, if `Person` contains a `List` of `Address(street, city)`, you'll be able to write:

 
```java
	private static final String[][] STRINGS = {
	    { "firstName", "Nome" },
		{ "street", "Strada" },
		{ "city", "Città" }
	};
```

or, for example, using `properties`:

```
# com.example.MyFormStrings.properties
firstName = Nome
street = Strada
city = Città
```

The columns in the table will be localized

![Translated Detail Fields](http://i.imgur.com/PrRq7oe.png)

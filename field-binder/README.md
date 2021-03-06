# FieldBinder Add-on

An advanced FieldGroup implementation with support for automated generation of Master/Detail forms.

![Master-Detail Demo](http://i.imgur.com/TZdfj5P.png)




## What is wrong with Good Ol' FieldGroup?

Vaadin's `FieldGroup` and `BeanFieldGroup` are non-visual components that build and automatically bind `Field`s to a data source. The data source may be an `Item` or a Java Bean. However, `FieldGroup`s have quite a few limitations:

* A FieldGroup only contains *bound* fields
* If a field is not bound to a datasource (using `unbind()`), then it is automatically *removed* from the FieldGroup. The FieldGroup is then unaware that the field exists. Trying to `getField()` returns `null`.
* A FieldGroup cannot `buildAndBind()` fields with unrecognized values, but it is extensible through a Factory; however, unrecognized values include `Collection` types, such as `List`s, which are typical of forms of the Master/Detail-type  
* There is no generic implementation of a standard Master/Detail editor (the JPAContainer `MasterDetailEditor` implementation is ad-hoc)
* There is no way to bind a FieldGroup to a Container: a FieldGroup is bound to an Item. Thus, there is no simple way to scan through a collection of records and display their contents in a form

## Meet the FieldBinder

The `FieldBinder` add-on is an advanced FieldGroup implementation. 

Features:

* A `FieldBinder<T>` binds to a Java Bean, like a BeanFieldGroup
* Managed Fields (built using `build()`) can be always `bind()`'ed and `unbind()`'ed; the FieldBinder will always keep track of them
* A FieldBinder can build() and bind() a component that displays a bean property even when it is a List<T>
* A FieldBinder can be bound to a Container. The `DataNavigation` interface provides commands to move an internal pointer to the next, previous, first and last Item in the Container (the Container must implement `Container.Ordered`: most `Container` implementations do, since it is required by `Table`)
* The `DataNavigation` interface provides commands to scan through a dataset and retrieving the `Item` that it points to.
* The `DataNavigation` interface provides standard behavior for performing CRUD and lookup operations, which can be extended through a regular, Vaadin-style listener mechanism
* `FieldBinder` comes with standard built-in CRUD for the ListContainer and the [Lazy Mongo Container](https://github.com/tyl/mongodbcontainer-addon). The ListContainer should already cover most Java Bean use-cases. Support for Vaadin's official JPAContainer has landed in v1.1.  
* The `ButtonBar` component (and its relatives, `NavButtonBar`, `CrudButtonBar`, `FindButtonBar`) may be bound  to a `DataNavigation` and automatically generate buttons for user interactions 
* ButtonBars are i18n compliant through the standard Java `ResourceBundle` mechanism 
* The `ListTable` and `BeanTable` wrappers augment Vaadin's regular Table with default behavior for basic CRUD.

![Detailed FieldBinder Taxonomy](http://i.imgur.com/RiFRfxJ.png)


## Maven


Add `field-binder` to your Maven dependencies

```xml
<dependency>
   <groupId>org.tylproject.vaadin.addon.fieldbinder</groupId>
   <artifactId>field-binder</artifactId>
   <version>1.0</version>
</dependency>

<repository>
   <id>vaadin-addons</id>
   <url>http://maven.vaadin.com/vaadin-addons</url>
</repository>
```

If you want to use the SNAPSHOT version, clone this repository and install the package locally with

```sh
  $ git clone https://github.com/tyl/field-binder/
  $ mvn install 
```


## Demo (for the impatient)

Want to give an immediate look at the FieldBinder? Try the demo:

```sh
  $ cd demo
  $ mvn package && java -jar target/field-binder-demo-1.0.jar  
```

or simply:

```sh
  $ cd demo
  $ ./run.sh
```

The demo uses [vaadin4spring](https://github.com/peholmst/vaadin4spring) for convenience.

## Documentation

The documentation is divided into two categories: [User](user) and [Dev](dev). User documentation contains less technical details. Dev documentation is more in-depth and it is suggested as an introductory reading before delving into the details of the advanced usage of the FieldBinder.

* [User](user)
  * [Tutorials](user/tutorials)
  * [Built-in Event System](user/events)
  * [Localization](user/localization)
* [Dev](dev)
  * [Technical Walkthrough](dev/tech-walkthrough.md) 
  * [Dev Details on the Built-in Event System](dev/events)
  * [Built-in Search Feature](dev/search)
  * [Customizing Zoom Fields](dev/zoomfields)

### Contributing

If you want to contribute to the development of the FieldBinder Add-on we suggest you to take some time read the **Dev** documentation
and get you confident with the source code. Among welcome extensions are contributions to Container CRUD implementations, both if they are already  supported (e.g., bug-fixes) and if they are completely new.

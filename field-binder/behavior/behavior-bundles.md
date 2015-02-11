# Default Behavior Bundles

The DefaultBehaviorFactory is configure to support the following subpackages of `org.tylproject.vaadin.addon.fieldbinder.behavior`


	├── commons
	│   ├── FieldBinders.java
	│   ├── SearchWindowFindListeners.java
	│   └── Tables.java
	└── containers
	    ├── jpacontainer
	    │   ├── JPAContainerCrud.java
	    │   └── JPAContainerTableCrud.java
	    ├── listcontainer
	    │   ├── ListContainerCrud.java
	    │   └── ListContainerTableCrud.java
	    └── mongocontainer
	        ├── BufferedMongoTableCrud.java
	        └── MongoCrud.java

## Commons

FieldBinders and Tables contain default behavior for simple forms (FieldBinder-based) or tabular forms (Table-based—grid support is under development).

## Containers

Supported containers are all bean/entity-based:

* JPAContainer
* ListContainer
* MongoContainer

You can design your own classes for custom containers.


## BehaviorFacade

The default implementation of the `Behavior` interface...
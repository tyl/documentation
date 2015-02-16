## FilterFactory

A `FilterFactory` is an abstract component that provides a method to generate a filter from a (usually textual) pattern.
It is an interface containing only one method:

```java
Container.Filter createFilter(
	Class<?> targetType,
	Object targetPropertyId,
	Object pattern);
```

For a given `targetType`, and a given `targetPropertyId` the factory should return a Filter instance that corresponds to the given pattern.

The default implementation of the FilterFactory interface is called `DefaultFilterFactory` and supports many filters. See the [related document for more information on the default filter patterns](search-patterns.md). Users may define their own implementation to provide alternative pattern translations, or to add support for more patterns.

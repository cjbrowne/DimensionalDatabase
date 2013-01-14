DimensionalDatabase
==================

The idea is to create a database that supports viewing data as discrete points on an arbitrary number of dimensions.  Each dimension is effectively a key/value pair whose key describe the point on the dimension, and the dimension itself has other properties that determine its relationship to other dimensions.

# Dimensions
A Dimension has the following properties:
 - Data Type: the type of discrete datapoints on the dimension.  This can be either a list of perpendicular Dimensions or a list of accepted primitive value types, or a mixture of the two.
 - Key Type: the type of the key used to access discrete datapoints on the dimension.  For example, a Dimension with a String key might have data-points at "name", "address" and "email".  Using a Dimension as a Key-Type is undefined (implementations may choose to allow it if they can see some use case that I'm as-yet unaware of).
 - Access Specifier: I'm still working on what it means to restrict access to a dimension.  At the moment it involves restricting how you can access a Dimension via related dimensions, but this may evolve into something entirely different.

# Dimensional Relationships
There are a number of possible relationships between Dimensions.  It's an implementation detail to an extent, but implementors should consider methods to express the relationships between Dimensions in the syntax of any exposed interfaces (including the User Interface and API of the implementation).
## Perpendicular Dimensions
A Dimension is said to be perpendicular if either its Data Type is another dimension (parent perpendicular) or if another dimension specifies this dimension as a Data Type (child perependicular).
## Parallel Dimensions
A Dimension is said to be parallel to another dimension if its parent dimension (it must also be perpendicular to another dimension) lists both dimensions as children.  So parallel dimensions are conceptually identical to sibling dimensions.
## Relationship example
If we have a 'root' Dimension A that has the Data Type "B, C, int, string" then:
 - B and C are child-perpendicular to A
 - A is parent-perpendicular to B and C
 - B is parallel to C

# Data
As should be obvious, data is stored as discrete points on one or more dimensions.  A well-designed Dimensional Database should never duplicate data (that's one of the advantages of this system), not even ID data.  Each 'record' from the database is described by its parent Dimensions and their Keys [meta-data] and the value of the record [data].  You can craft unique ids for every single item in the database using a "root" Dimension A with data-type "B" and by qualifying the key-type of A with the "unique" keyword.  You can use any datatype for your unique id, including names.

# Example Syntax
For the benefit of implementors, consider the following example syntax:
## Binary Operators

### `->`

 * lvalue: dimension
 * rvalue: dimension
 * semantics: The rvalue is a child dimension of the rvalue.

### `.`

 * lvalue: dimension
 * rvalue: key
 * semantics: The rvalue is a key from the lvalue dimension

### `:`

 * lvalue: qualified key, value, dimension
 * rvalue: command
 * semantics: issues command specified in rvalue using the lvalue as the first argument.  Implementors should type-check the lvalue and issue an error if the type does not match the expected type of the first argument to the command specified in rvalue.

### `=`

 * lvalue: qualified key [if key is unqualified, implementors are permitted to use an arbitrary 'default' orphaned dimension; thus, using an unqualified key as an lvalue will invoke implementation-defined behaviour only in the case of unorthodox database models]
 * rvalue: value
 * semantics: the data-point referred to by the lvalue is set to the rvalue

### `==`

 * lvalue: qualified key or value
 * rvalue: qualified key or value
 * semantics: compares the lvalue to the rvalue, returns the boolean values TRUE or FALSE depending on whether the rvalue and rvalue are equivalent.  If the types differ and are not comparable, implementors are free either to return either value or to consider the syntax invalid and issue an error.  Comparing data of two different types is implementation-defined.

### `!=`

 * lvalue: qualified key or value
 * rvalue: qualified key or value
 * semantics: see `==`, of which this operator is the precise inverse.

### `-`

 * lvalue: qualified key
 * rvalue: value
 * semantics: reduces the data point referred to by lvalue by the value specified in rvalue.  In the case of complex data types (dimensions, strings, etc.), the implementor is free to decide how the semantics of this operator should work (or to consider it an error).

### `+`

 * lvalue: qualified key
 * rvalue: value
 * semantics: see "-", this is the semantic inverse of that

### Keywords

### `unique`
- Applies to: Keys
- Effect: disallows a User Interface or API from inserting a duplicate key.  An error is produced in these circumstances and the record is not committed to the database.  Implementors should take advantage of the 'unique' keyword to implement efficient storage and retrieval algorithms (as as tries for string-based keys).

### `increment`
- Applies to: Keys, Values
- Effect: in the case of Keys, inserting to the special-case '!' key will increment the largest 'key' value by one and insert the new value at that point on the Dimension.  Implementor's note: you may provide an incrementation algorithm which takes note of 'gaps' to preserve contiguity.

### `orphan`
- Applies to: Dimensions
- Effect: creates the new Dimension with no parent-perpendicular dimensions.  There must be at least one Orphaned Dimension in every database, and there may be more than one.  It's not recommended to use the database in this way, however, and implementors are permitted to treat multiple orphaned dimensions as a special case.

## Commands
Commands can either be issued "procedurally", or can be considered methods on their first argument.

Each command gives an example of both uses, in case you're confused.

All commands return their first argument after modification.  So you can string commands together easily.

### `set`
- argument list: qualified key, (qualified key | value) [, boolean=true]
- semantics: sets the value of the first argument to the second argument or the value referred to by the second argument.  If the key doesn't exist, it will be created as long as the boolean argument is unprovided or 'true'.
- usage examples: 
 - set(character.yoda->data.gold,23)
 - character.yoda->data.gold:set(23,false)

### `delete`
- argument list: (qualified key | dimension) [, boolean=false]
- semantics: deletes the specified key or dimension.  If the second argument is false or not provided the command will preserve as much data as possible, if it is true it will purge any data associated with the first argument
- usage examples:
 - delete(character.yoda)
 - character.yoda->data:delete(true) // (will delete the 'data' dimension just from the 'yoda' data point, use 'character->data:delete()' to delete it entirely from the 'character' dimension)

### `print`
- argument list: (qualified key | dimension | value)
- semantics: prints the argument to stdout.  If it's a qualified key, the key and its associated value will be printed.  If it's a value, the value will be printed.  If it's a dimension, the dimension, all its keys and all data points will be printed in tabular format.  Any child dimensions will just be listed by name.
- usage examples:
 - print(character)
 - character.yoda->data:print

## Aliases
Command versions of operators are provided for convenience.
The API exposed from the reference implementation uses these as function names, usefully.

### `child`
Aliases the `->` operator

### `key`
Aliases the `.` operator

### `is`
Aliases the `=` operator

### `equals`
Aliases the `==` and (by extension) `!=` operators.  Just reverse the return value to get to the `!=` operator.

### `minus`
Aliases the `-` operator

### `plus`
Aliases the `+` operator

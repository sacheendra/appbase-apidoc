# appbase.io API Documentation

All operations can return an error, an isError check should be done before any processing.

## Identify

### connect
Requires an API key for the app.

# This section requires refinement if things like google auth are used.
## Auth

### authenticate
Requires a email and digest of the password and returns an object containing user information.

### signup
Requires a email and digest of the password. Any other information the app may want to store. Returns an object containing user info or an error.

### forgot password
An email is sent to the user containing a link to reset their password.
# End of Refinements required.

## User

### get current user
An object representing the current user.

## Datastore

### add set (name)
Returns a set.

### delete set (name)
Deletes the set.

### get set (name)
Returns the set.

### add ([name])
Adds an object to the datastore.

### on (path)
Listens on an object from the datastore.

## Object

### get linked object/set (path)
Returns the linked object/set.

### update object (path, array of changed property-value pairs)
Update the object in the database.

### delete object (path)
Deletes the object.

### off (obj_id)
Turns off listening.

## Set

### get linked object (path)
Returns the linked object.

### delete linked object (path)
Returns the linked object.

### append an object (path of the object)
Append the object to the set.

### set new linked object (path to set, path of the object)
Returns the old object.

### off (set_id)
Stops listening on the set.

## Counter

### increment (path)
Increments the counter.

### decrement (path)
Decrements the counter.

### listen to this counter (path to the counter)
Listens on the counter.






































## Appbase Datatypes
  * Integer
  * String
  * Appbase Object
  * Appbase Set
  * Appbase Counter

## Appbase Datastore
This is the primary interface to your Appbase database

### new Datastore
This constructor takes the URI of your database and returns an instance of the Datastore object which provides access to your database.

### Usage
```javascript
new Datastore(URI)
```
 - __URI__ `String` -
  The URI of your database provided by appbase.

#### Returns
An instance of the Datastore object, which is the main access point to your Appbase database.

### addSet
This function is used to add a new set to the global object. This function takes the name of set to be added to the global object. If a set of the same name already exists, an error is passed to the callback.

#### Usage
```javascript
datastore.addSet(name, callback)
```
 - __name__ `String` -
  Name of the set to be created.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is an empty set for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### onSet
This function is used to listen on an existing set of the global object. This function takes the name of set to retrieve from the global object.

#### Usage
```javascript
datastore.onSet(name, callback)
```
 - __name__ `String` -
  Name of the set to listen on.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is an Appbase Set set for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### add
This function is used to add a new object to the database. This function takes the name of a set in global object and/or a key, both of which are optional. If nothing is specified, a uuid will be assigned as the key of the object.

#### Usage
```javascript
datastore.add([globalSet,[key]], callback)
```
 - __globalSet__ _(optional)_ `String` -
	Name of the set in the global object.
 - __key__ _(optional)_ `String` -
	Key given to the new object.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is the newly made Appbase object for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### on
This function is used to listen on an existing object. This function takes a path to an object and an error occurs if the object does not exist.

#### Usage
```javascript
datastore.on(path, callback)
```
 - __path__ `String` -
	Path to the object in Appbase.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is the Appbase object which resides at the path for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null. 


## Appbase Object
This object is the first argument of callbacks passed to on and add functions of the Datastore objects.

### setProperty
This function is used to set a property of the Appbase object. The function takes the name of the property as the first argument and the value as the second argument. A callback is taken as the last argument.

#### Usage
```javascript
AppbaseObject.setProperty(name, value, callback)
``` 
 - __name__ `String` -
  Name of the property.
 - __value__ `String` -
  Value to be assigned to the property. A value can be any valid Appbase Datatype.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is the changed Appbase object for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### deleteProperty
This function is used to delete a property from an Appbase object. It takes the name of a property and a callback.

#### Usage
```javascript
AppbaseObject.deleteProperty(name, callback)
```
 - __name__ `String` -
  Name of the property to be deleted.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is the changed Appbase object for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### getProperty
This function is used to get the value of a property. It takes the name of the property.

#### Usage
```javascript
AppbaseObject.getProperty(name)
```
 - __name__ `String` -
  Name of the property to get.

#### Returns
An Appbase Datatype which is the value of the property. A promise if the Appbase Datatype contained is an Object or a Set. An error object if the property does not exist.
The promise has a then function which takes a callback of the following type.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is the Appbase Datatype(Set or Object) for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### getTree
This function is used to retrieve a tree of objects with the current Appbase Object as root.

#### Usage
```javascript
getTree(levels, [maxNumberOfElements], callback)
```
 - __levels__ `Integer` -
  Depth of levels by which nested objects will be returned
 - __maxNumberOfElements__ _(optional)_ `Integer` -
  Maximum number of node is the tree which is returned. This takes priority over levels. Nodes are counted in a breadth-first order from smaller to larger in the standard ordering.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is an Appbase Tree for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### off
This function is used to stop the reception of real-time updates to an object.

#### Usage
```javascript
AppbaseObject.off()
```


## Appbase Set
This object is the first argument of callbacks passed to onSet and addSet functions of the Datastore objects.

### getItem
Used to get an item of the set by index. The index is an integer.

#### Usage
```javascript
AppbaseSet.getItem(index, callback)
```
 - __index__ `Integer` -
  The index of the Appbase Object to get.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is an Appbase Object for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### setItem
Used to set an item of the set by index. The index is an integer.

#### Usage
```javascript
AppbaseSet.setItem(index, object, callback)
```
 - __index__ `Integer` -
  The index of the Appbase Object to set.
 - __index__ `Integer` -
  The Appbase Object to be placed at the index.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is an Appbase Set with the new item for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### appendItem
Used to append an item to the end of the set. The index is an integer.

#### Usage
```javascript
AppbaseSet.getItem(index, callback)
```
 - __index__ `Integer` -
  The index of the Appbase Object to get.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is an Appbase Object for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### setItem
Used to set an item of the set by index. The index is an integer.

### deleteItem
Used to delete an item of the set by index. The index is an integer. Them item itself is not deleted, only its reference is removed from the set.

#### Usage
```javascript
AppbaseSet.deleteItem(index, callback)
```
 - __index__ `Integer` -
  The index of the Appbase Object to delete.
 - __callback__ `function` -
  callback is a function of the form callback(result, error). The result is the modified Appbase Set for success and a null value for error. If an error occurs, error object is passed as second argument, otherwise it is null.

### off
Used to turn of real-time updates on this Appbase Set.

#### Usage
```javascript
AppbaseSet.off()
```


## Appbase Counter
One of the available Appbase datatypes. All operations performed using this are transactional in nature.

### increment
Used to increment the value of the counter by the specified integer. 1 if no value is passed.

#### Usage
```javascript
AppbaseCounter.increment(num)
```
 - __num__ `Integer` -
  Number to increment the counter by.

### decrement
Used to decrement the value of the counter by the specified integer. 1 if no value is passed.

#### Usage
```javascript
AppbaseCounter.decrement(num)
```
 - __num__ `Integer` -
  Number to decrement the counter by.

### read
Used to obtain the current value of the counter.

#### Usage
```javascript
AppbaseCounter.read()
```

#### Returns
Return an integer which is the current value of the counter.

### off
This turns of real-time updates on the current Appbase Counter object.

#### Usage
```javascript
AppbaseCounter.off()
```

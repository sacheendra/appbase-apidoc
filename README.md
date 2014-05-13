# appbase.io API Documentation

## Appbase class

### Appbase.new()

This function takes the name of a collection and creates a new object in that collection. A reference to the newly created object is returned. The collection is created if it does not already exist. An optional key can attached to the collection string, in which case the object will be created with given key.

#### Usage
```
Appbase.new(collection,[key])
```
 - __collection__ _String_ -
	Name of the collection

 - __key__ _(optional) String_ -
	Key given to the new object

#### Returns
An Appbase reference pointing to the new object 

#### Example
```javascript
var ab = Appbase.new('User','andy_dufresne');
```

### Appbase.ref()

This function takes a collection-key pair as a string of the format "collection-name:key" and returns a reference to the object if it exists. Returns an error otherwise.


#### Usage
```
Appbase.ref(path)
```
 - __path__ _String_ -
	path to the object in Appbase

#### Returns
An Appbase reference pointing to the object located at given path 

#### Example
```javascript
var ab = Appbase.ref('User:andy_dufresne/Tool:rock_hammer');
```
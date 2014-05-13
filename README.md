# appbase.io API Documentation

## Appbase class

### Appbase.new()

This function takes the name of a collection and creates a new object in that collection. A reference to the newly created object is returned. The collection is created if it does not already exist. An optional key can attached to the collection string, in which case the object will be created with given key.

#### Usage
```
Appbase.new(collection,[key])
```
 - __collection__ `String` -
	Name of the collection
 - __key__ _(optional)_ `String` -
	Key given to the new object

#### Returns
An _Appbase_ reference pointing to the new object 

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
 - __path__ `String` -
	path to the object in Appbase

#### Returns
An `Appbase` reference pointing to the object located at given path 

#### Example
```javascript
var ab = Appbase.ref('User:andy_dufresne/Tool:rock_hammer');
```


## Appbase reference

This object is a reference returned by the class functions - Appbase.new() and Appbase.ref(). It points to an object stored in the appbase.

### path()
To know what path this reference points to.

#### Usage
```
var path = ab.path();
```

#### Returns
`String` - The path.

#### Example
```
var ab = Appbase.ref('User:andy_dufresne/Tool:rock_hammer');
var path = ab.path();
// now, path = 'User:andy_dufresne/Tool:rock_hammer'
```

### set()
Sets value for a property

#### Usage
```
ab.set(property,value);
```
 - __property__ `String`
 - __value__ `String/Boolean/Number/null/undefined` - Giving null or undefined as the value removes the property

#### Returns
The same `Appbase` reference, to allow chaining of set methods

#### Example
```

```

### get()
Getting the value of a property
#### Usage
```
ab.get([property],callback);
```
 - __property__ _(optional)_ `String` - If no property is given, all the properties and their values are returned in a javascript object
 - __callback__ `Function` - This function will be called with arguement - value of the given property, or the object with all property/value pairs

#### Returns
`null`

#### Example
```

```

### getTree()
### on()
### off()
### link()
### unlink()


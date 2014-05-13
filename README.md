# appbase.io API Documentation

## Appbase class

### Appbase.new()

This function takes the name of a collection and creates a new object in that collection. A reference to the newly created object is returned. The collection is created if it does not already exist. An optional key can attached to the collection string, in which case the object will be created with given key.

#### Usage
```javascript
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
```javascript
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


## Appbase object

This object is returned by the class functions - Appbase.new() and Appbase.ref(). It is a reference to an object stored in Appbase.

### path()
To know what path this reference points to.

#### Usage
```javascript
path()
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
```javascript
set(property,value)
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
```javascript
get([property],callback)
```
 - __property__ _(optional)_ `String` - If no property is given, all the properties and their values are returned in a javascript object
 - __callback__ `Function` - This function will be called with arguement - value of the given property, or the object with all property/value pairs

#### Returns
`null`

#### Example
```

```

### getTree()
#### Usage
```javascript
getTree([levels],callback)
```
 - __levels__ _(optional)_ `Number` - Depth of levels by which nested objects will be returned
 - __callback__ `Function`

#### Returns
`null`

#### Example
```

```

### link()
#### Usage
```javascript
link([key],ab)
```
 - __key__ _(optional)_ `String` -
 - __ab__ `Appbase` reference - 

#### Returns
`null`

#### Example
```

```

### unlink()

#### Usage
```javascript
unlink(ab)
unlink(collection,key)
```

 - __ab__ `Appbase` reference - 
 - __collection__ `String` - 
 - __key__ `String` - 

#### Returns
`null`

#### Example
```

```


### on()
#### Usage
```javascript
on(event,callback)
```
 - __event__ `String` - 
 - __callback__ `Function` - 

#### Returns
`null`

#### Example
```

```


### off()
#### Usage
```javascript
off(event)
```
 - __event__ `String` -

#### Returns
`null`

#### Example
```

```

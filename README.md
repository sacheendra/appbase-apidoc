# appbase.io API Documentation

# Appbase JavaScript API

## Datatypes

  * Number
  * String
  * Object

## Appbase Global

### Appbase.add()

This function takes the name of a collection and creates a new object in that collection. A reference to the newly created object is returned. The collection is created if it does not already exist. An optional key can attached to the collection string, in which case the object will be created with given key.

#### Usage
```javascript
Appbase.add(collection,[key],callback)
```
 - __collection__ `String` -
  Name of the collection
 - __key__ _(optional)_ `String` -
  Key given to the new object
 - __callback__ `Function` - err
#### Returns
An _Appbase_ reference pointing to the new object 

#### Example
```javascript
var myDataRef = Appbase.new('User','andy_dufresne',callback);
```

### Appbase.ref()

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
var myDataRef = Appbase.ref('User/andy_dufresne/rock_hammer');
```

## Appbase Reference
Represents a reference to an object

### path()
To know what path this reference points to.

#### Usage
```javascript
path()
```

#### Returns
`String` - The path.

### insert()
Sets/inserts value for a property

#### Usage
```javascript
insert([property], value, [index], [callback])
```
 - __property__ _(optional)_ `String`
 - __value__ `String/Number/Appbase Reference` - Giving null or undefined as the value removes the property
 - __index__ `Number` - The property is inserted at the index. i.e., Things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err,new obj snapshot)

If no __property__ is given while inserting an _Appbase object_, object's __uuid__ will be set as the __property__ name

#### Returns
The same `Appbase` reference, to allow chaining of set methods

### remove()
removes a property

#### Usage
```javascript
remove(property[callback])
```
 - __property__ `String`
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of set methods

### destroy()
Delete the whole object, references to this object in other objects will be deleted on read.

#### Usage
```javascript
destroy()
```

### on('value')

#### Usage
```javascript
on('value',options,callback)
```
options is an object with properties.

 - __levels__ `Number` - include data of referenced objects up to this depth 
 - __limits__ `Array` - list of numbers, specifying how many objects should be included from each level. eg. levels:3, limits:[5,2,2] = total objects included: 5*2*2

callback will be passed an Appbase Snapshot Object.

### on('object_added')
get existing properties, listen to new ones

#### Usage
```javascript
on('object_added',options,callback)
```
options is an object with properties:

 - __levels__ `Number` - include data of referenced objects up to this depth 
 - __limits__ `Array` - list of numbers, specifying how many objects should be included from each level.
 - __startAt__ `Number` - index to start with
 - __overflow__ (ignore) `Function` - when level one limit is hit, objects from the bottom are as argument to this function. - This is used for removing objects from the view automatically

callback will be passed an Appbase Snapshot Object.

### on('object_removed')
listen to removal of objects

#### Usage
```javascript
on('child_removed',options,callback)
```

### on('object_changed')
listen to changes of objects

#### Usage
```javascript
on('child_changed',options,callback)
```

### off()
#### Usage
```javascript
off([event])
```
 - __event__ _(optional)_ `String` - All listeners will be cancelled if not specified.

### strongInsert()
A strongly consistent insert operation. No operations will take place on the property 

#### Usage
```javascript
insert([property], apply, [index], [callback])
```
 - __property__ _(optional)_ `String`
 - __apply__ `function` - The function should return which returns String/Number/Appbase Reference. The old value is passed in as an argument to the function.
 - __index__ `Number` - The property is inserted at the index. i.e., Things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err,new obj snapshot)

If no __property__ is given while inserting an _Appbase object_, object's __uuid__ will be set as the __property__ name

#### Returns
The same `Appbase` reference, to allow chaining of set methods


## Appbase Snapshot Object
This is will be passed to callbacks in all event firing.

### val()
Returns the data in the form of a JavaScript object.

#### Usage
```javascript
val()
```

#### Returns
The data of the object as a JavaScript object.

### prevVal()
Returns the data in the form of a JavaScript object as it was before this change was received.

#### Usage
```javascript
prevVal()
```

#### Returns
The data in the form of a JavaScript object as it was before this change was received.

### ref()
Returns the Appbase Reference for this object.

#### Usage
```javascript
ref()
```

#### Returns
The Appbase Reference for this object.

### getIndex()
Returns the index of this object .

#### Usage
```javascript
getIndex()
```

#### Returns
The Appbase Reference for this object.

### getPrevIndex()
Returns the index of this object before this change was received.

#### Usage
```javascript
getIndex()
```

#### Returns
The Appbase Reference for this object before this change was received.

### exportVal()
Returns the data in the form of a JavaScript object with ordering data.

#### Usage
```javascript
exportVal()
```

#### Returns
The value of the object as a JavaScript object with ordering data.

### name()
Returns the property name with which the object was stored in the current path.

#### Usage
```javascript
name()
```

#### Returns
The property name with which the object was stored in the current path.
# appbase.io API Documentation

# Appbase JavaScript API

## Datatypes

  * Number
  * String
  * Object

## Appbase Global

### Appbase.add()

Adds a new object in _Appbase_. This is the way to create a new object in a _namespace_. A _namespace_ is a collection of objects on which _security rules and permissions_ can be applied and all the objects belonging to this namespace will follow the rules. An existing object can also be added to another _namespace_. A link to the object is placed in the namespace, i.e., a new object is not created. All updates to an object will reflect when the object is accessed through any _namespace_.

#### Usage
```javascript
Appbase.add(namespace,[key/reference],callback)
```
 - __namespace__ `String` - Name of the namespace
 - __key/reference__ _(optional)_ `String/Appbase Reference` - key given to the new object/an Appbase Reference to an existing object
 - __callback__ `Function` - err

The _namespace_ is created if it does not already exist. Optionally, A _unique key_ can be given to the the object, otherwise a unique id will be given automatically. The key should not contain '/' character. 

If the object with the given key already exists, reference to the existing object will be returned.

The error may occur if this operation is not permitted, and there are two such cases:
 1. Namespace doesn't exist and creation of new namespaces is not permitted.
 2. For the given namespace, creating new objects is not permitted.

#### Returns
An _Appbase Reference_ pointing to the new/existing object.

#### Example
```javascript
var myDataRef = Appbase.add('User','andy_dufresne',function(error){
    if(!error){
        console.log('object created.')
    }
);
```

### Appbase.ref()
An _Appbase Reference_ allows operating on an object stored in _Appbase_ at some path. This method creates reference pointing to a path. 

#### Usage
```javascript
Appbase.ref(path)
```
 - __path__ `String` - path to the object in Appbase
A _Path_ in Appbase represents a chain of elements, separated with '/', and it finally points an object. The first element of the path represents a namespace, and following elements are objects. 

#### Returns
An `Appbase` reference pointing to the object located at the given path. 

#### Example
```javascript
var myDataRef = Appbase.ref('shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
```

The _path_, `'shawshank.api.appbase.io/user/andy_dufresne/rock_hammer'` points to an _object_, which is inserted as the `property : 'rock_hammer'` in the object of the `namespace : 'user'` with `key : andy_dufresne`.

## Appbase Reference
Operations, such as read/write on objects, located at a path can be done using Appbase References.

### path()
To know what path this reference points to.

#### Usage
```javascript
path()
```

#### Returns
`String` - The path.

### insert()
Sets/inserts value for a property. This operation creates a new accessible path, which can be used to create an _Appbase Reference_.

#### Usage
```javascript
insert([property], value, [index], [callback])
```
 - __property__ _(optional)_ `String`
 - __value__ `String/Number/Appbase Reference` - Giving null or undefined as the value removes the property
 - __index__ `Number` - The property is inserted at the index, things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err,new obj snapshot)

If no __property__ is given while inserting an _Appbase object_, object's __uuid__ will be set as the __property__ name

#### Returns
The same `Appbase` reference, to allow chaining of methods


#### Example
```javascript
var userRef = Appbase.ref('shawshank.api.appbase.io/user/andy_dufresne');
var toolRef = Appbase.add('tool'); // new object of the namespace 'tool'

toolRef.insert('size',12);
userRef.insert('rock_hammer',toolRef);

/* Now Dufresne's rock hammer can be accessed directly with 
 * the path: 'shawshank.api.appbase.io/user/andy_dufresne/rock_hammer', 
 * and its size with: 'shawshank.api.appbase.io/user/andy_dufresne/rock_hammer/size'
 */
```

### remove()
removes a property

#### Usage
```javascript
remove(property[callback])
```
 - __property__ `String`
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### destroy()
Delete the whole object, references to this object in other objects will be removed as well.

#### Usage
```javascript
destroy()
```

### on('value')
Reading of data from _Appbase_ happens through listeing to events on _Appbase References_. This event listens to changes in the value at a path. 

It immediately fires the event with existing value, when listening for the first time, then fires again whenever the value is changed. 

#### Usage
```javascript
on('value',[options],callback)
```
 - __options__ is an object with properties:
     - __levels__ `Number` - include data of referenced objects up to this depth 
     - __limits__ `Array` - list of numbers, specifying how many objects should be included from each level. eg. levels:3, limits:[5,2,2] = total objects included: 5 x 2 x 2 = 20.
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var toolRef = Appbase.ref('shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
/* Existing data at this location: {size:12}
 */

toolRef.on('value',function(snapshot){
    console.log(snapshot.val().size);
})

seTimeout(function(){
    toolRef.insert('size',13);
},2000);

/* Immediately logs '12' - the existing value.
 * After 2 secs, it logs '13'.
 */
```

### on('object_added')
Get existing objects/properties inserted at a location, and listen to new ones.

#### Usage
```javascript
on('object_added',[options],callback)
```

 - `options` is an object with properties:
     - __levels__ `Number` - include data of referenced objects up to this depth 
     - __limits__ `Array` - list of numbers, specifying how many objects should be included from each level.
     - __startAt__ `Number` - index to start with
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.


#### Returns
The same `Appbase` reference, to allow chaining of methods


#### Example
```javascript
var toolRef = Appbase.ref('shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
/* Existing data at this location: {size:12}
 */

toolRef.on('object_added',function(snapshot){
    console.log(snapshot.name(),':',snapshot.val());
})

seTimeout(function(){
    toolRef.insert('usage','shaping chess pieces');
},2000);

/* Immediately logs 'size : 12' - existing data.
 * After 2 secs, it logs 'usage : shaping chess pieces'.
 */
```

### on('object_removed')
Listen to removal of objects/properties. 

#### Usage
```javascript
on('child_removed',[options],callback)
```
 - __callback__ `Function` - with snapshot to the removed object. 
#### Returns
The same `Appbase` reference, to allow chaining of methods

### on('object_changed')
Whenever an the value/index of an existing property is changed, this event is called. It doesn't log the existing value.

#### Usage
```javascript
on('child_changed',[options],callback)
```

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var toolRef = Appbase.ref('shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');

// Existing data at this location: {size:12,usage:'shaping chess pieces'}

toolRef.on('object_changed',function(snapshot){
    var text = snapshot.name() + ' changed from ' + snapshot.prevVal() + ', to ' + snapshot.val();
    console.log(text);
})

toolRef.insert('usage','prison break');

// Logs 'usage changed from shaping chess pieces to prision break'.
```
### off()
#### Usage
```javascript
off([event])
```
 - __event__ _(optional)_ `String` - All listeners will be cancelled if not specified.

#### Returns
The same `Appbase` reference, to allow chaining of methods

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
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var toolRef = Appbase.ref('shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');

toolRef.insert('size',function(prevSize) {
  return prevSize + 1;
});

/* The size of Dufresne's rock hammer can be increased
 * consistently (no changes will be lost). If 3 people
 * the size by 1 each simultaneously, the size will
 * increase by 3.
 */
 
```


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

### index()
Returns the index of this object .

#### Usage
```javascript
index()
```

#### Returns
The Appbase Reference for this object.

### prevIndex()
Returns the index of this object before this change was received.

#### Usage
```javascript
prevIndex()
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
#### Usage
```javascript
name()
```

#### Returns
The property name with which the object was stored in the current path.
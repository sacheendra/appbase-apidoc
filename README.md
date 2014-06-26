# appbase.io API Documentation

# Appbase JavaScript API

## Datatypes

  * Number
  * String
  * Boolean
  * Object

## Appbase Global

### Appbase.new()

This is the way to create a new object in a _namespace_. A _namespace_ is a collection of objects on which _security rules and permissions_ can be applied and all the objects belonging to this namespace will follow the rules.

#### Usage
```javascript
Appbase.new(namespace,[key],callback)
```
 - __namespace__ `String` - Name of the namespace
 - __key__ _(optional)_ `String` - key given to the new object
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
var myDataRef = Appbase.new('User','andy_dufresne',function(error){
    if(!error){
        console.log('object added.')
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
A _Path_ in Appbase represents a chain of elements, separated with '/', and it finally points an object. The _base-url_ is a unique string for the Application, and the first element after the url represents a namespace, and following elements are objects. 

#### Returns
An `Appbase` reference pointing to the object located at the given path. 

#### Example
```javascript
var myDataRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
```

The _path_, `'https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer'` points to an _object_, which is inserted as the `linkname : 'rock_hammer'` in the object of the `namespace : 'user'` with `key : andy_dufresne`. The application's base url is `https://shawshank.api.appbase.io`.


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

### set()
Sets a property's value.

#### Usage
```javascript
set('prop',val,[callback])
```
 - __prop__ `String` - property name
 - __value__ `String/Number/Boolean` - value
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### strongSet()
A strongly consistent _set_ operation. It allows you create consistent aggregators, such as counters.

#### Usage
```javascript
strongSet([property], apply, [index], [callback])
```
 - __property__ _(optional)_ `String`
 - __apply__ `function` - The function should return which returns String/Number/Boolean. The old value is passed in as an argument to the function.
 - __index__ `Number` - The property is inserted at the index. i.e., Things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');

toolRef.strongSet('size',function(prevSize) {
  return prevSize + 1;
});

/* The size of Dufresne's rock hammer can be increased
 * consistently (no changes will be lost). If 3 people
 * the size by 1 each simultaneously, the size will
 * increase by 3.
 */
 
```

### unset()
Removes a property's value.

#### Usage
```javascript
unset('prop',[callback])
```
 - __prop__ `String` - property name
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### link()
Sets/inserts a link. This operation creates a new accessible path, which can be used to create an _Appbase Reference_.

#### Usage
```javascript
link([linkname], abRef, [index], [callback])
```
 - __linkname__ _(optional)_ `String`
 - __abRef__ `Appbase Reference` - The object where the link would point
 - __index__ `Number` - The property is inserted at the index, things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err)

If no __linkname__ is given while inserting an _Appbase object_, object's __uuid__ will be set as the __property__ name

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var userRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne');
var toolRef = Appbase.new('tool'); // new object of the namespace 'tool'

toolRef.set('size',12);
userRef.link('rock_hammer',toolRef);

/* Now Dufresne's rock hammer can be accessed directly with 
 * the path: 'https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer'
 */
```

### unlink()
removes a link

#### Usage
```javascript
unlink(linkname,[callback])
```
 - __linkname__ `String/Appbase Reference` - If while adding the link, no linkname was gievn and only the Appbase Reference was given, then an Appbase Reference can be used to remove the link 
 - __callback__ - with args: (err)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### destroy()
Delete the object from _Appbase_, references to this object in other objects will be removed as well. The appbase reference now turns invalid and listeners won't fire. Any data modification operation will fail.

#### Usage
```javascript
destroy([callback])
```
- __callback__ - with args: (err)

### on('value')
TODO: explain levels
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
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
/* Existing data at this location: {size:12}
 */

toolRef.on('value',function(snapshot){
    console.log(snapshot.val().size);
})

seTimeout(function(){
    toolRef.set('size',13);
},2000);

/* Immediately logs '12' - the existing value.
 * After 2 secs, it logs '13'.
 */
```

### on('link_added')
Get existing links inserted at a location, and listen to new ones.

#### Usage
```javascript
on('link_added',[options],callback)
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
TODO:
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
/* Existing data at this location: {size:12}
 */

toolRef.on('link_added',function(snapshot){
    console.log(snapshot.name(),':',snapshot.val());
})

seTimeout(function(){
    toolRef.set('usage','shaping chess pieces');
},2000);

/* Immediately logs 'size : 12' - existing data.
 * After 2 secs, it logs 'usage : shaping chess pieces'.
 */
```

### on('link_removed')
Listen to removal of objects/properties. 

#### Usage
```javascript
on('child_removed',[options],callback)
```
 - __callback__ `Function` - with snapshot to the removed object. 
#### Returns
The same `Appbase` reference, to allow chaining of methods

### on('link_changed')
Whenever an the value/index of an existing property is changed, this event is called. It doesn't log the existing value.

#### Usage
```javascript
on('link_changed',[options],callback)
```

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
TODO
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');

// Existing data at this location: {size:12,usage:'shaping chess pieces'}

toolRef.on('link_changed',function(snapshot){
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

### refDuplicate()
Get a new _Appbase Reference Object_ pointing to the same path. This is useful when you want to use the same object in a different context and attach different listeners.

#### Usage
```javascript
refDuplicate()
```

#### Returns
New `Appbase` reference

### refToLink()
Get an _Appbase Reference Object_ pointing to the object stored in the link. This is just string manipulation on the path, and the reference will be returned even if the link doesn't exist, but read/write operations will fail.

#### Usage
```javascript
refToLink(linkname)
```
#### Returns
`Appbase` reference

#### Example
```javascript
var userRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne');
var toolRef = userRef.refToLink('rock_hammer');

/* `toolRef` points to the the path:
 * 'https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer'
 */
```

### refToUplink()
Go up in path and get an _Appbase Reference Object_.

#### Usage
```javascript
refToUplink()
```
Throws an error of the object has no uplink.

#### Returns
`Appbase` reference

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/user/andy_dufresne/rock_hammer');
var userRef = toolRef.refToUplink();

/* `userRef` points to the the path:
 * 'https://shawshank.api.appbase.io/user/andy_dufresne'
 */
 
var newRef = userRef.refToUplink(); //Throws an error
```

## Appbase Snapshot Object
_Appbase Snapshot Object_ is an immutable copy of the data at a location in _Appbase_. It is passed to callbacks in all event firing. It can't be modified and will never change. To modify data, use an Appbase reference. 
TODO: Explain methods

### val()
Returns the data (prop-value pairs) in the form of a JavaScript object.

#### Usage
```javascript
val()
```

#### Returns
The data (prop-value pairs) of the object as a JavaScript object.

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
Returns the index of this object.

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

## Privileged Methods

These methods shouldn't be a necessity in the normal application working. The use of these methods can be controlled via security rules.

### Appbase.rename()

Allows renaming of namespaces, object primary keys and moving an object to a different namespace.


#### Usage
```javascript
Appbase.rename(old,new)
```
 - __old__ `String` - old /namespace, or /namespace/pk or Appbase Reference Object
 - __new__ `String` - new /namespace, or /namespace/pk

The old /namespace or /namespace/pk must exist and the new one must not.

The links pointing to the object being renamed will still work.

#### Returns
An `Appbase` reference pointing to the new path, if renaming of an object is happening. `undefined` otherwise.

#### Example
```javascript
Appbase.rename('/users', '/prisoners'); // Renaming a namespace

Appbase.rename('/users/abc', '/users/xyz'); //Renaming a primary key

Appbase.rename('/users/abc', '/prisoners'); // Moving an object to another namespace

Appbase.rename('/users/abc', '/prisoners/pqr'); // Moving an object to another namespace and renaming its primary key too

Appbase.rename('/user/pqr/xyz','/user/abc/klm'); //Throws an error, as the path should only be up to /namespace/pk

Appbase.rename('/users', '/prisoners/abc'); // Throws an error, if `old` is namespace, the `new` has to be namespace.

var abRef = ('/user/abc');
var abNewRef = Appbase.rename(abRef,'/user/pqr'); //Works. `abRef` will now turn invalid, and listeners won't work, until a new object at /user/abc is created. Use `abNewRef` instead.

var prisonerRef = Appbase.rename(abNewRef,'/prisoner'); // Moving an object to another namespace.


```
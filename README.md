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
Appbase.new({ns: namespace, pk: key},callback)
```
 - __ns__ _(optional)_ `String` - Name of the namespace
 - __pk__ _(optional)_ `String` - key given to the new object
 - __callback__ `Function` - err

The _namespace_ automatically is created if it does not already exist. 

Optionally, A unique _primary key_ can be given to the the object, otherwise a generate unique key will be given automatically. The key should not contain '/' character. 

If no _namespace_ given, the new object is goes to a precreated  namespace named _Default_.

If neither the _key_ nor the _namespace_ is given, an object with a generated unique key is created under the _Default_ namespace. 

If the object with the given key already exists, reference to the existing object will be returned.

The error may occur if this operation is not permitted, and there are two such cases:

 1. Namespace doesn't exist and creation of new namespaces is not permitted.
 2. For the given namespace, creating new objects is not permitted.

#### Returns
An _Appbase Reference_ pointing to the new/existing object.

#### Example
```javascript
var myDataRef = Appbase.new({ns:'prisoner',key:'andy_dufresne'},function(error){
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
var myDataRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
```

The _path_, `'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'` points to an _object_, which is inserted as the `linkname : 'rock_hammer'` in the object of the `namespace : 'prisoner'` with `key : andy_dufresne`. The application's base url is `https://shawshank.api.appbase.io`.


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
set(prop,val,[callback])
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
strongSet(property, apply, [callback])
```
 - __property__ `String`
 - __apply__ `function` - The function should return which returns String/Number/Boolean. The old value is passed in as an argument to the function
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');

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
Removes a property.

#### Usage
```javascript
unset(prop,[callback])
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
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
var toolRef = Appbase.new({ns:'tool'}); // new object of the namespace 'tool'

toolRef.set('size',12);
prisonerRef.link('rock_hammer',toolRef);

/* Now Dufresne's rock hammer can be accessed directly with 
 * the path: 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'
 */
```

### unlink()
Removes a link.

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

Reading of data from _Appbase_ happens through listeing to events on _Appbase References_. This event listens to changes in the value at a path. 

It immediately fires the event with existing value, when listening for the first time, then fires again whenever the value is changed. 

#### Usage
```javascript
on('value',[listenDepth],callback)
```
 - __listenDepth__ `Number` - The depth of links up to which listen for data changes. Default value is `1`, meaning data of the object itself.
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.

`listenDepth` allows listening and retrieving data of the links along with the actual object. If a depth is provided, the method also listens for changes in the links and fires the event when a link's data is changed.

In the background, listening with `listenDepth` happens through listening to `value` on the linked objects. It's a costly operation in terms of bandwidth if there are a huge number of links.

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
// Existing data at this location: {first_name:'Andy', last_name: 'Dufresne'}

var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
// Existing data : {size:12}

toolRef.on('value',function(toolSnapshot){
    console.log('tool-logger: ' + toolSnapshot.val().size);
});

prisonerRef.on('value',2,function(prisonerSnapshot){

    /* The `listenDepth` here is '2', so it would listen to changes in the links' data, 
     * The only link here is 'rock_hammer' and its snapshot is accessible via prisonerSnapshot.link('rock_hammer').
     * `prisonerSnapshot` itself contains prisoner's data, and prisonerSnapshot.val() would return {first_name:'Andy', last_name: 'Dufresne'}.
     * Take a look at the Appbase Snapshot Object in this document for details.
     */

    var toolSnapshot = prisonerSnapshot.link('rock_hammer');
    console.log('prisoner-logger: ' +  + toolSnapshot.val().size);
});

setTimeout(function(){
    toolRef.set('size',13);
},2000);

/* Both loggers would immediately log '12' - the existing value. 
 * After 2 secs, they would log '13'.
 */ 

setTimeout(function(){
    prisonerRef.set('prison_id', 37927);
},4000);

/* After 4 secs, as prisonerRef's data is changed, 'value' event would be fired on prisonerRef.
 * Prisoner-logger is logging rock_hammer's size, which is still '13'. 
 * So it would would log '13', the second time.
 * Obviously, this time prisoner-logger would log `prison_id` too.
 * 
 * stdout as a whole:
 *  |--------------------
 *  | tool-logger: 12
 *  | prisoner-logger: {first_name:'Andy', last_name: 'Dufresne'}  13
 *  | prisoner-logger: {first_name:'Andy', last_name: 'Dufresne', prison_id: 37927} 13
 *  |--------------------
 */

```

### on('link_added')
Get existing links inserted at a location, and listen to new ones.

#### Usage
```javascript
on('link_added',[fetchDepth],[orderingOptions],callback)
```
 - __fetchDepth__ `Number` - The depth of links up to which the data should be retrieved. Default is `1`, meaning data of the link itself.
 - `orderingOptions` is an object with properties:
     - __limit__ How many exsiting links to fetch
     - __startAt__ `Number` - index to start with
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.


`fetchDepth` here is different from `value` event's `listenDepth` parameter. Here it means that whenever a new link is added, along with its own data, its links' data should be retrieved, too. Fetching of links' data in depth happens only once and it doesn't keep listening to changes in the links' data in depth.

`startAt` and `limit` are only effective for retrieving the existing data. New links will be returned regardless of their index.


#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var redRef = Appbase.new({ns:'prisoner',pk:'ellis_boyd_red'}); // New prisoner
redRef.set('firstname','Ellis Boyd');
redRef.set('lastname','Redding');
redRef.set('nick','Red');

var capRef = Appbase.new({ns:'clothes'});
capRef.set('type','Newsboy Cap');
redRef.link('cap',capRef); // Red has a Newsboy Cap.

var andyRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');

andyRef.on('link_added',2,function(linkSnap){
    console.log('Name:', linkSnap.val().nick);
    
    // As the `fetchDepth` is `2`, cap's data is fetched too, and the snapshot is available
    var capSnap = linkSnap.link('cap'); 
    console.log('wears:', capSnap.val().type); 
});

setTimeout(function(){
    andyRef.link('best_friend',redRef);
},2000);

/* stdout
 * |-----------------
 * | Nick: Red
 * | wears: Newsboy Cap
 * |-----------------
 */
```

### on('link_removed')
Listen to removal of links. 

#### Usage
```javascript
on('link_removed',[fetchDepth],callback)
```
 - __fetchDepth__ `Number` - The depth of links up to which the data should be retrieved. Default is `1`, meaning data of the link itself
 - __callback__ `Function` - with snapshot to the removed object 

The `fetchDepth` here is similar to the one in 

#### Returns
The same `Appbase` reference, to allow chaining of methods

### on('link_changed')
If an existing link is changed, this event is fired.

#### Usage
```javascript
on('link_changed',[listenDepth],callback)
```
 - __listenDepth__ `Number` - The depth of links up to which listen for data changes. Default value is `0`, meaning no listening on the data.
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.

These are the cases, where a link is considered changed:
 1. A link's order is manually changed, i.e. by calling `link(linkname,order)` and providing a manual order for an existing link
 2. A link now points to a different object
 3. Data in the object where the link points, is changed. To listen to such changes, `listenDepth` should be kept `> 0`. 

When the `listenDepth` > `0`, in the background, all the links are being listened for `value` event, and this is a very costly operation if there are a huge number of links. This is the reason why `listenDepth` is kept `0` by default, where this event is fired only for the first two cases.
 
#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
TODO depth
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
// Existing data at this location: {size:12}

var andy = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');

andy.on('link_changed',1,function(snapshot){
    //As the `listenDepth` is '1', the event is fired when the link's data is changed
    console.log(snapshot.val());
})

toolRef.set('usage','prison break');

// Logs {size: 12, usage: 'prison break'}.
```
### off()
Stop listening to changes.

#### Usage
```javascript
off([event])
```
 - __event__ _(optional)_ `String` - All listeners will be cancelled if not specified.

#### Returns
The same `Appbase` reference, to allow chaining of methods

### once()
The callback function is called only once and then its turned off.

#### Usage
```javascript
once(event,..)
```
 - __event__ `String` - Any of the four events, 
 - Other arguments depend on `event`.

This is equivalent to:
```javascript
abRef.on('value',function(snap)){
    // do something with snap
    abRef.off('value');
}
```

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
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
var toolRef = prisonerRef.refToLink('rock_hammer');

/* `toolRef` points to the the path:
 * 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'
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
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
var prisonerRef = toolRef.refToUplink();

/* `prisonerRef` points to the the path:
 * 'https://shawshank.api.appbase.io/prisoner/andy_dufresne'
 */
 
var newRef = prisonerRef.refToUplink(); //Throws an error
```

## Appbase Snapshot Object
_Appbase Snapshot Object_ is an immutable copy of the data at a location in _Appbase_. It is passed to callbacks in all event firing. It can't be modified and will never change. To modify data, use an Appbase reference.

### val()
Returns the data (prop-value pairs) in the form of a JavaScript object.

#### Usage
```javascript
val()
```

### prevVal()
Returns the data in the form of a JavaScript object as it was before this change was received.

#### Usage
```javascript
prevVal()
```

### ref()
Returns an Appbase Reference for this object.

#### Usage
```javascript
ref()
```

### name()
The link name with which the object is stored in the current path.
#### Usage
```javascript
name()
```

### index()
Returns the index of this object

#### Usage
```javascript
index()
```


### prevIndex()
Returns the index of this object before this change was received.

#### Usage
```javascript
prevIndex()
```

### link()
Snapshot object of the link.  
Applicable only when the object is being listened with `depth` more than 1.

#### Usage
```javascript
link(linkname)
```

### links()
Array containing _Appbase Snapshot Objects_ for all the links, in the same order specified while adding the links.  
Applicable only when the object is being listened with `depth` more than 1.

#### Usage
```javascript
links()
```

### exportVal()
TODO
Returns the data in the form of a JavaScript object with ordering data.

#### Usage
```javascript
exportVal()
```

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

Appbase.rename('/users/abc', '/users/xyz'); //Renaming the primary key

Appbase.rename('/users/abc', '/prisoners'); // Moving an object to another namespace

Appbase.rename('/users/abc', '/prisoners/pqr'); // Moving an object to another namespace and renaming its primary key too

Appbase.rename('/user/pqr/xyz','/user/abc/klm'); //Throws an error, as the path should only be up to /namespace/pk

Appbase.rename('/users', '/prisoners/abc'); // Throws an error, if `old` is a namespace, the `new` has to a be namespace.

var abRef = ('/user/abc');
var abNewRef = Appbase.rename(abRef,'/user/pqr'); //Renaming the primary key. `abRef` will now turn invalid, and listeners won't work, until a new object at /user/abc is created. Use `abNewRef` instead.

var prisonerRef = Appbase.rename(abNewRef,'/prisoner'); // Moving an object to another namespace.


```
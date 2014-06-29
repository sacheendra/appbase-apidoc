# appbase.io API Documentation

# Appbase JavaScript API

## Appbase Datatypes

### Conceptual
  * Vertex
  * Edge
  * Property

### Primitive Javascript
  * Number
  * String

### Javascript Objects
 * Appbase Reference
 * Snapshot Object

## Appbase Global

### Appbase.new()

This is the way to create a new vertex in a _namespace_. A _namespace_ is a collection of objects on which _security rules and permissions_ can be applied and all the objects belonging to this namespace will follow the rules.

#### Usage
```javascript
Appbase.new(namespace,[key],callback)
```
 - __namespace__ `String` - Name of the namespace
 - __key__ _(optional)_ `String` - key given to the new vertex
 - __callback__ `Function` - err

The _namespace_ automatically is created if it does not already exist. 

Optionally, A unique _primary key_ can be given to the vertex, otherwise a generated unique key will be given automatically. The key should not contain '/' character. 

If the vertex with the given key already exists, reference to the existing vertex will be returned.

The error may occur if this operation is not permitted, and there are two such cases:

 1. Namespace doesn't exist and creation of new namespaces is not permitted.
 2. For the given namespace, creating new objects is not permitted.

#### Returns
An _Appbase Reference_ pointing to the new/existing vertex.

#### Example
```javascript
var abRef = Appbase.new('prisoner','andy_dufresne',function(error){
    if(!error){
        console.log('vertex added.')
    }
);
```

### Appbase.ref()
An _Appbase Reference_ allows operating on a vertex stored in _Appbase_ at some path. This method creates reference pointing to a path. 

#### Usage
```javascript
Appbase.ref(path)
```
 - __path__ `String` - path to the vertex in Appbase
A _Path_ in Appbase represents a chain of elements, separated with '/', and it finally points a vertex. The _base-url_ is a unique string for the Application, and the first element after the url represents a namespace, and following elements are objects. 

#### Returns
An `Appbase` reference pointing to the vertex located at the given path. 

#### Example
```javascript
var abRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
```

The _path_, `'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'` points to an _object_, which is inserted as the `edgename : 'rock_hammer'` in the vertex of the `namespace : 'prisoner'` with `key : andy_dufresne`. The application's base url is `https://shawshank.api.appbase.io`.


## Appbase Reference
Operations, such as read/write on objects, located at a path can be done using Appbase References.

### path()
To know what path this reference points to.

#### Usage
```javascript
abRef.path()
```

#### Returns
`String` - The path.

### properties.add()
Add a property into the vertex and give it a value, or set a value for an existing property.

#### Usage
```javascript
abRef.properties.add(prop,val,[callback])
```
 - __prop__ `String` - property name
 - __value__ `String/Number` - value
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### properties.commit()
A strongly consistent _set_ operation. It allows you create consistent aggregators, such as counters.

#### Usage
```javascript
abRef.properties.commit(property, apply, [callback])
```
 - __property__ `String`
 - __apply__ `function` - The function should return which returns String/Number. The old value is passed in as an argument to the function
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');

toolRef.properties.commit('size',function(prevSize) {
  return prevSize + 1;
});

/* The size of Dufresne's rock hammer can be increased
 * consistently (no changes will be lost). If 3 people
 * the size by 1 each simultaneously, the size will
 * increase by 3.
 */
 
```

### properties.remove()
Removes a property.

#### Usage
```javascript
abRef.properties.remove(prop,[callback])
```
 - __prop__ `String` - property name
 - __callback__ - with args: (err,new obj snapshot)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### edges.add()
Sets/inserts a edge. This operation creates a new accessible path, which can be used to create an _Appbase Reference_.

#### Usage
```javascript
abRef.edges.add([edgename], [abRef], [index], [callback])
```
 - __edgename__ _(optional)_ `String`
 - __abRef__ _(optional)_ `Appbase Reference` - The vertex where the edge would point
 - __index__ _(optional)_ `Number` - The property is inserted at the index, things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1. If no `index` is given, the edge would be enqueued.
 - __callback__ - with args: (err)

Either two of the first there optional arguments is necessary for the method to work.

If no `edgename` is given while inserting an _Appbase object_, vertex's __uuid__ will be set as the edge's name.

If some edge exists and points to a vertex, and now its passed as `edgename` with an Appbase Reference of some other vertex, the edge will be _replaced_. It is considered to be _removed_ and _added_ again, therefore, __edge_removed__ event will be fired, followed by __edge_added__ for the same edge. Take a look at the documentation of `abRef.on()` for more details on the events.

If an existing edgename, or abRef is passed with an index, the edge  is moved to that index, and all the following edges are  considered moved, too. __edge_moved__ event will be fired on all of them. 

#### Returns
The same `Appbase` reference, to allow chaining of methods

#### Example
```javascript
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
var toolRef = Appbase.new('tool'); // new vertex of the namespace 'tool'

toolRef.properties.add('size',12);
prisonerRef.edges.add('rock_hammer',toolRef);

/* Now Dufresne's rock hammer can be accessed directly with 
 * the path: 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'
 */
```

### edges.remove()
Removes a edge.

#### Usage
```javascript
abRef.edges.remove(edgename,[callback])
```
 - __edgename__ `String/Appbase Reference` - If while adding the edge, no edgename was gievn and only the Appbase Reference was given, then an Appbase Reference can be used to remove the edge 
 - __callback__ - with args: (err)

#### Returns
The same `Appbase` reference, to allow chaining of methods

### destroy()
Delete the vertex from _Appbase_, references to this vertex in other objects will be removed as well. The appbase reference now turns invalid and listeners won't fire. Any data modification operation will fail.

#### Usage
```javascript
abRef.destroy([callback])
```
- __callback__ - with args: (err)

### on('value')

Reading of data from _Appbase_ happens through listening to events on _Appbase References_. This event listens to changes in the value at a path. 

It immediately fires the event with existing value, when listening for the first time, then fires again whenever the value is changed. 

#### Usage
```javascript
abRef.on('value',[listenerName],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference Object` - points the path on which the event is fired
    - __snapObj__ `Appbase Snapshot Object` - Snapshot of the data stored at the path. Take a look at the documentation of Appbase Snapshot Object.

`listenerName` is a unique string, which can be used later on to turn this listener off using `offWithName()`. This is a way to keep track of listeners. If a `listenerName` is given again with a different callback function, the old callback function is replaced, and will no longer be called when the event is fired, instead the new function is called. If no `listenerName` is given, a unique string will be generated as the listener's name and returned.

#### Returns
A `string` which is the listener's name and can be used to turn the listener off.

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
// Existing data : {size:12}

toolRef.on('value',function(err,ref,snap){
   console.log(snap.properties().size); 
);

setTimeout(function(){
    toolRef.properties.add('size',13);
},2000);

/* It would immediately log '12' - the existing value. 
 * After 2 secs, It would log '13'.
 */ 
```

### on('edge_added')
Get existing edges inserted at a location, and listen to new ones.

#### Usage
```javascript
abRef.on('edge_added',[listenerName],[options],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`
 - __options__ `Object`
     - __limit__ How many existing edges to fetch
     - __startAt__ `Number` - Index to start with
     - __noData__ `Boolean` - Whether to include the data stored at the vertex where the edge points 
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference Object` - pointing to path of the edge
    - __[snapObj]__ `Appbase Snapshot Object` - Snapshot of the data stored at the vertex pointed by the edge

`snapObj` will not be passed if `{noData: true}` is passed as the options to the listener.

`startAt` and `limit` are only effective for retrieving the existing properties. New edges will be returned regardless of their index.

#### Returns
A `string` which is the listener's name and can be used to turn the listener off.

#### Example
```javascript
var redRef = Appbase.new('prisoner','ellis_boyd_red'); // New prisoner
redRef.properties.add('firstname','Ellis Boyd');
redRef.properties.add('lastname','Redding');
redRef.properties.add('nick','Red');

var andyRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');

andyRef.on('edge_added',function(edgeSnap){
    console.log('Name:', edgeSnap.val().nick);
});

andyRef.edges.add('best_friend',redRef);

/* stdout
 * |-----------------
 * | Name: Red
 * |-----------------
 */
```

### on('edge_removed')
Listen to removal of edges. 

#### Usage
```javascript
abRef.on('edge_removed',[listenerName],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`.
 - __fetchDepth__ `Number` - The depth of edges up to which the data should be retrieved. Default is `1`, meaning data of the edge itself
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference Object` - pointing to path of the edge
    - __snapObj__ `Appbase Snapshot Object` - Snapshot of the data stored at the vertex pointed by the edge

#### Returns
The same `Appbase` reference, to allow chaining of methods

###  on('edge_changed')
If the properties of the vertex, pointed by an existing edge is changed, this event is fired.

#### Usage
```javascript
abRef.on('edge_changed',[listenerName],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`.
 - __listenDepth__ `Number` - The depth of edges up to which listen for data changes. Default value is `0`, meaning no listening on the properties.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference Object` - pointing to path of the edge
    - __snapObj__ `Appbase Snapshot Object` - Snapshot of the data stored at the vertex pointed by the edge

For this event to fire, in the background the vertexes pointed by all the edges are listened for `value` event, and this would be a costly operation in terms of bandwidth if there are a huge number of edges.

#### Returns
A `string` which is the listener's name and can be used to turn the listener off.

#### Example
```javascript
var andy = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');

var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
// Existing data at this location: {size:12}


andy.on('edge_changed',function(snapshot){
    for (var prop in snapshot.properties()) {
        console.log(prop,':',snapshot.properties()[prop]);
    }
})

toolRef.properties.add('usage','prison break');

/* stdout
 * |-----------------
 * | size : 12
 * | usage : prison break
 * |-----------------
 */
```

###  on('edge_moved')
This event is fired, when the order of an edge is changed. 

When the order of an edge is manually changed by calling `abRef.edges.add()` on an existing edge, also the other edges are moved, either they are shifted upward or downward by '1' in the order. This event is fired for all the edges, moved manually or automatically.

#### Usage
```javascript
abRef.on('edge_moved',[listenerName],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`.
 - __listenDepth__ `Number` - The depth of edges up to which listen for data changes. Default value is `0`, meaning no listening on the properties.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference Object` - pointing to path of the edge
    - __snapObj__ `Appbase Snapshot Object` - Snapshot of the data stored at the vertex pointed by the edge
 
#### Returns
A `string` which is the listener's name and can be used to turn the listener off.


### off()
Turn off the listeners on an event.

#### Usage
```javascript
abRef.off([event])
```
 - __event__ _(optional)_ `String` - All the listeners on this event, will be turned off. If no event is given, all the listeners on all the events will be turned off.

#### Returns
`Array` of listeners' names, which have been turned off.

### offWithName()
Turn off a listener by its name.

#### Usage
```javascript
offWithName(listenerName)
```
 - __listenerName__ `String` - The unique name given to the listener while calling `on(event)`.

### refToEdge()
Get an _Appbase Reference Object_ pointing to a edge of the current vertex. This is just string manipulation on the path, and the reference will be returned even if the edge doesn't exist, but read/write operations will fail.

#### Usage
```javascript
abRef.refToEdge(edgename)
```

#### Returns
`Appbase` reference

#### Example
```javascript
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
var toolRef = prisonerRef.refToEdge('rock_hammer');

/* `toolRef` points to the the path:
 * 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'
 */
```

### refToUpedge()
Go up in path and get an _Appbase Reference Object_.

#### Usage
```javascript
abRef.refToUpedge()
```
Throws an error of the vertex has no upedge.

#### Returns
`Appbase` reference

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
var prisonerRef = toolRef.refToUpedge();

/* `prisonerRef` points to the the path:
 * 'https://shawshank.api.appbase.io/prisoner/andy_dufresne'
 */
 
var newRef = prisonerRef.refToUpedge(); //Throws an error
```

## Appbase Snapshot Object
_Appbase Snapshot Object_ is an immutable copy of the data at a location in _Appbase_. It is passed to callbacks in all event firing. It can't be modified and will never change. To modify data, use an Appbase reference.

### properties()
Returns prop-value pairs in the form of a JavaScript object.

#### Usage
```javascript
snapObj.properties()
```

### prevProperties()
Returns the data in the form of a JavaScript object as it was before this change was received.

#### Usage
```javascript
snapObj.prevProperties()
```

### namespace()
Returns the _namespace_ of the vertex. 

#### Usage
```javascript
snapObj.namespace()
```

### name()
The edge name with which the vertex is stored in the current path.
#### Usage
```javascript
snapObj.name()
```

### index()
Returns the index of this vertex

#### Usage
```javascript
snapObj.index()
```

### prevIndex()
Returns the index of this vertex before this change was received.

#### Usage
```javascript
snapObj.prevIndex()
```

## Privileged Methods

These methods shouldn't be a necessity in the normal application working. The use of these methods can be controlled via security rules.

### Appbase.rename()

Allows renaming of namespaces, vertex primary keys and moving a vertex to a different namespace.


#### Usage
```javascript
Appbase.rename(old,new)
```
 - __old__ `String` - old /namespace, or /namespace/pk or Appbase Reference Object
 - __new__ `String` - new /namespace, or /namespace/pk

The old /namespace or /namespace/pk must exist and the new one must not.

The edges pointing to the vertex being renamed will still work.

#### Returns
An `Appbase` reference pointing to the new path, if renaming of a vertex is happening. `undefined` otherwise.

#### Example
```javascript
Appbase.rename('/users', '/prisoners'); // Renaming a namespace

Appbase.rename('/users/abc', '/users/xyz'); //Renaming the primary key

Appbase.rename('/users/abc', '/prisoners'); // Moving a vertex to another namespace

Appbase.rename('/users/abc', '/prisoners/pqr'); // Moving a vertex to another namespace and renaming its primary key too

Appbase.rename('/user/pqr/xyz','/user/abc/klm'); //Throws an error, as the path should only be up to /namespace/pk

Appbase.rename('/users', '/prisoners/abc'); // Throws an error, if `old` is a namespace, the `new` has to a be namespace.

var abRef = ('/user/abc');
var abNewRef = Appbase.rename(abRef,'/user/pqr'); //Renaming the primary key. `abRef` will now turn invalid, and listeners won't work, until a new vertex at /user/abc is created. Use `abNewRef` instead.

var prisonerRef = Appbase.rename(abNewRef,'/prisoner'); // Moving a vertex to another namespace.


```
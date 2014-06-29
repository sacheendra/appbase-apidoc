# appbase.io API Documentation

# Appbase JavaScript API

## Primitive Datatypes

  * Number
  * String

## Appbase Conceptual Datatypes
  * Vertex
  * Edge
  * Property
  * Value

## Appbase Javascript Datatypes
 * Reference
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

### strongSet()
A strongly consistent _set_ operation. It allows you create consistent aggregators, such as counters.

#### Usage
```javascript
abRef.strongSet(property, apply, [callback])
```
 - __property__ `String`
 - __apply__ `function` - The function should return which returns String/Number. The old value is passed in as an argument to the function
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
abRef.edges.add([edgename], abRef, [index], [callback])
```
 - __edgename__ _(optional)_ `String`
 - __abRef__ `Appbase Reference` - The vertex where the edge would point
 - __index__ `Number` - The property is inserted at the index, things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err)

If no __edgename__ is given while inserting an _Appbase object_, vertex's __uuid__ will be set as the __property__ name

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
abRef.on('value',[listenerName],[listenDepth],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener.
 - __listenDepth__ `Number` - The depth of edges up to which listen for data changes. Default value is `1`, meaning data of the vertex itself.
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.

`listenerName` is a unique string, which can be used later on to turn this listener off using `offWithName()`. This is a way to keep track of listeners. If a `listenerName` is given again with a different callback function, the old callback function is replaced, and will no longer be called when the event is fired, instead the new function is called. If no `listenerName` is given, a unique string will be generated as the listener's name and returned.

`listenDepth` allows listening and retrieving data of the edges along with the actual vertex. If a depth is provided, the method also listens for changes in the edges and fires the event when a edge's data is changed.

In the background, listening with `listenDepth` happens through listening to `value` on the objects pointed by the edges. It's a costly operation in terms of bandwidth if there are a huge number of edges.

#### Returns
A `string` which is the listener's name and can be used to turn the listener off.

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

    /* The `listenDepth` here is '2', so it would listen to changes in the edges' data, 
     * The only edge here is 'rock_hammer' and its snapshot is accessible via prisonerSnapshot.edges.add('rock_hammer').
     * `prisonerSnapshot` itself contains prisoner's data, and prisonerSnapshot.val() would return {first_name:'Andy', last_name: 'Dufresne'}.
     * Take a look at the Appbase Snapshot Object in this document for details.
     */

    var toolSnapshot = prisonerSnapshot.edges.add('rock_hammer');
    console.log('prisoner-logger: ' +  + toolSnapshot.val().size);
});

setTimeout(function(){
    toolRef.properties.add('size',13);
},2000);

/* Both loggers would immediately log '12' - the existing value. 
 * After 2 secs, they would log '13'.
 */ 

setTimeout(function(){
    prisonerRef.properties.add('prison_id', 37927);
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

### on('edge_added')
Get existing edges inserted at a location, and listen to new ones.

#### Usage
```javascript
abRef.on('edge_added',[listenerName],[fetchDepth],[orderingOptions],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`.
 - __fetchDepth__ `Number` - The depth of edges up to which the data should be retrieved. Default is `1`, meaning data of the edge itself.
 - `orderingOptions` is a vertex with properties:
     - __limit__ How many exsiting edges to fetch
     - __startAt__ `Number` - index to start with
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.


`fetchDepth` here is different from `value` event's `listenDepth` parameter. Here it means that whenever a new edge is added, along with its own data, its edges' data should be retrieved, too. Fetching of edges' data in depth happens only once and it doesn't keep listening to changes in the edges' data in depth.

`startAt` and `limit` are only effective for retrieving the existing properties. New edges will be returned regardless of their index.

#### Returns
A `string` which is the listener's name and can be used to turn the listener off.

#### Example
```javascript
var redRef = Appbase.new('prisoner','ellis_boyd_red'); // New prisoner
redRef.properties.add('firstname','Ellis Boyd');
redRef.properties.add('lastname','Redding');
redRef.properties.add('nick','Red');

var capRef = Appbase.new('clothes');
capRef.properties.add('type','Newsboy Cap');
redRef.edges.add('cap',capRef); // Red has a Newsboy Cap.

var andyRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');

andyRef.on('edge_added',2,function(edgeSnap){
    console.log('Name:', edgeSnap.val().nick);
    
    // As the `fetchDepth` is `2`, cap's data is fetched too, and the snapshot is available
    var capSnap = edgeSnap.edges.add('cap'); 
    console.log('wears:', capSnap.val().type); 
});

setTimeout(function(){
    andyRef.edges.add('best_friend',redRef);
},2000);

/* stdout
 * |-----------------
 * | Nick: Red
 * | wears: Newsboy Cap
 * |-----------------
 */
```

### on('edge_removed')
Listen to removal of edges. 

#### Usage
```javascript
abRef.on('edge_removed',[listenerName],[fetchDepth],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`.
 - __fetchDepth__ `Number` - The depth of edges up to which the data should be retrieved. Default is `1`, meaning data of the edge itself
 - __callback__ `Function` - with snapshot to the removed vertex 

The `fetchDepth` here is similar to the one in 

#### Returns
The same `Appbase` reference, to allow chaining of methods

###  on('edge_changed')
If an existing edge is changed, this event is fired.

#### Usage
```javascript
abRef.on('edge_changed',[listenerName],[listenDepth],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation `on('value')`.
 - __listenDepth__ `Number` - The depth of edges up to which listen for data changes. Default value is `0`, meaning no listening on the properties.
 - __callback__ `Function` - will be passed an Appbase Snapshot Object.

These are the cases, where a edge is considered changed:
 1. A edge's order is manually changed, i.e. by calling `edge(edgename,order)` and providing a manual order for an existing edge
 2. A edge now points to a different vertex
 3. Data in the vertex where the edge points, is changed. To listen to such changes, `listenDepth` should be kept `> 0`. 

When the `listenDepth` > `0`, in the background, all the edges are being listened for `value` event, and this is a very costly operation if there are a huge number of edges. This is the reason why `listenDepth` is kept `0` by default, where this event is fired only for the first two cases.
 
#### Returns
A `string` which is the listener's name and can be used to turn the listener off.

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
// Existing data at this location: {size:12}

var andy = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');

andy.on('edge_changed',1,function(snapshot){
    //As the `listenDepth` is '1', the event is fired when the edge's data is changed
    console.log(snapshot.val());
})

toolRef.properties.add('usage','prison break');

// Logs {size: 12, usage: 'prison break'}.
```
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
### ref()
Returns an Appbase Reference for this vertex.

#### Usage
```javascript
snapObj.ref()
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

### edge()
Snapshot object of the edge.  
Applicable only when the vertex is being listened with `depth` more than 1.

#### Usage
```javascript
snapObj.edge(edgename)
```

### edges()
Array containing _Appbase Snapshot Objects_ for all the edges, in the same order specified while adding the edges.  
Applicable only when the vertex is being listened with `depth` more than 1.

#### Usage
```javascript
snapObj.edges()
```

### exportVal()
Returns the data in the form of a JavaScript object with ordering data.

#### Usage
```javascript
snapObj.exportVal()
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
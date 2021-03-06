# Appbase JavaScript API

The data model in Appbase can be best visualized as a graph. Each object in Appbase is represented as a __vertex__. _Vertices_ can store primitive values and are linked to other vertices by __edges__. Each _vertex_ belongs to a __namespace__. _Namespaces_ act as convenient entry points into the graph, and as we will see later, help with applying _security rules and permissions_.

## Appbase Datatypes

### Conceptual
  * Namespace
  * Vertex
  * Named Edge
  * Ordered Edge
  * Property

### Primitive Javascript
  * Number
  * String

### Javascript Objects
 * Appbase Reference
 * Vertex Snapshot

## Appbase Global

### Appbase.new()

Creates a new __vertex__ object and applies a _namespace_ to it.

#### Usage
```javascript
Appbase.new(namespace,[key],callback)
```
 - __namespace__ `String` - key of the namespace
 - __key__ _(optional)_ `String` - key given to the new vertex
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`

The _namespace_ is automatically created if it does not already exist.

A unique _key_ can be given to the vertex. Otherwise, a unique key will be generated automatically. The key should not contain any whitespace and '/' character.

An error occurs if a vertex already exists for the given namespace and key.

#### Returns
`Appbase Reference` - pointing to the new vertex.

#### Example
```javascript
var abRef = Appbase.new('prisoner', 'andy_dufresne', function(error) {
    if (!error) {
        console.log('Vertex created.')
    } else {
        console.log('The vertex already exists.')
    }
);
```

### Appbase.ref()
An _Appbase reference_ allows operating on a vertex stored in _Appbase_ at some path. This method creates a reference pointing to a path.

#### Usage
```javascript
Appbase.ref(path)
```
 - __path__ `String` - path to the vertex in Appbase
A _Path_ in Appbase consists of one or more linked vertices with the endpoint always being a vertex. '/' is used to demarcate between consequent vertices. The _base-url_ is a unique string for the Application, and the first element after the url represents a namespace, and following elements are objects.

#### Returns
`Appbase Reference` - pointing to the vertex located at the given path.

#### Example
```javascript
var abRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
```

The _path_, 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer' points to a _vertex_, which is inserted as the __edgename : 'rock_hammer'__ in the _vertex_ of the __namespace : 'prisoner'__ with __key : andy_dufresne__. The application's _base url_ is __https://shawshank.api.appbase.io__.

## Appbase Reference
Operations, such as read/write on vertex, located at a path can be done using an `Appbase Reference`.

### path()
To know what path this reference points to.

#### Usage
```javascript
abRef.path()
```

#### Returns
`String` - The path

### properties.add()
Add a property into the vertex and give it a value, or set a value for an existing property.

#### Usage
```javascript
abRef.properties.add(prop,val,[callback])
```
 - __prop__ `String` - property name
 - __value__ `String/Number` - value
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called
    - __snapObj__ `Vertex Snapshot` - Snapshot of the new data stored in the vertex.

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.

### properties.commit()
A strongly consistent _set_ operation. It allows you create consistent aggregators, such as counters.

#### Usage
```javascript
abRef.properties.commit(property, apply, [callback])
```
 - __property__ `String`
 - __apply__ `function` - The function should return which returns String/Number. The old value is passed in as an argument to the function
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called
    - __snapObj__ `Vertex Snapshot` - - Snapshot of the new data stored in the vertex.
    - __isCommitted__ `Boolean` - Whether the final value is committed or is still the new data returned from the server 

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.


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
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called
    - __snapObj__ `Vertex Snapshot` - - Snapshot of the new data stored in the vertex.

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.

### named_edges.add()
Sets/inserts a unidirectional edge to another vertex. This operation also creates a new accessible path, which can be used to create an `Appbase Reference`.

#### Usage
```javascript
abRef.named_edges.add(abRef, edgename, [callback])
```
 - __edgename__ `String`
 - __abRef__ `Appbase Reference` - The vertex where the edge would point
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called

Fo named edges, if some edge exists and points to a vertex, and now its passed as `edgename` with an `Appbase Reference` of some other vertex, the edge will be _replaced_. It is considered to be _removed_ and _added_ again, therefore, __edge_removed__ event will be fired, followed by __edge_added__ for the same edge. Take a look at the documentation of `abRef.on()` for more details on the events.

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.

#### Example
```javascript
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
var toolRef = Appbase.new('tool'); // new vertex of the namespace 'tool'

toolRef.properties.add('size',12);
prisonerRef.named_edges.add('rock_hammer',toolRef);

/* Now Dufresne's rock hammer can be accessed directly with 
 * the path: 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'
 */
```

### ordered_edges.add()
Sets/inserts a unidirectional edge to another vertex. This operation also creates a new accessible path, which can be used to create an `Appbase Reference`.

#### Usage
```javascript
abRef.edges.add(abRef, [priority], [callback])
```
 - __abRef__ `Appbase Reference` - The vertex where the edge would point
 - __priority__ _(optional)_ `Number` - A natural number, negative, positive or zero. If no priority is given, server timestamp will given as the priority, allowing you to fetch the edges in the same order as they were added
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called

If `abRef` for an existing ordered edge is passed with a different priority, the edge  is moved to that index, and all the following edges are  considered moved, too. __edge_moved__ event will be fired on all of them. 

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.

### named_edges.remove()
Removes a edge.

#### Usage
```javascript
abRef.named_edges.remove(edgename,[callback])
```
 - __edgename__ `String` - for named edges
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.


### ordered_edges.remove()
Removes a edge.

#### Usage
```javascript
abRef.ordered_edges.remove(abRef,[callback])
```
 - __abRef__ `Appbase Reference` - reference to the ordered edge
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the same path on which the method is called

#### Returns
`Appbase Reference` - pointing to the same path where the method is called. This allows chaining of methods.


### destroy()
Delete the vertex from _Appbase_, edges to this vertex in other vertexes will be removed as well. The appbase reference now turns invalid and listeners won't fire. Any data modification operation will fail.

#### Usage
```javascript
abRef.destroy([callback])
```
- __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`

### on('properties')

Reading of data from _Appbase_ happens through listening to events on _Appbase References_. This event listens to changes in the properties at a path. 

It immediately fires the event with existing properties, when listening for the first time, then fires again whenever the properties are changed. 

#### Usage
```javascript
abRef.on('properties',[listenerName],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - points to the path on which the event is fired
    - __snapObj__ `Vertex Snapshot` - Snapshot of the data stored in the vertex. Take a look at the documentation of `Vertex Snapshot`

`listenerName` is a unique string, which can be used later on to turn this listener off using `offWithName(listenerName)`. This is a way to keep track of listeners. If a `listenerName` is given again with a different callback function, the old callback function is replaced, and will no longer be called when the event is fired, instead the new function will be called. If no `listenerName` is given, a unique string will be generated as the listener's name and returned.

#### Returns
`String` - the listener's name and can be used to turn the listener off.

#### Example
```javascript
TODO: change according to the method signature
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
// Existing data : {size:12}

toolRef.on('properties',function(err,ref,snap){
   console.log(snap.properties().size); 
);

setTimeout(function(){
    toolRef.properties.add('size',13);
},2000);

/* It would immediately log '12' - the existing properties. 
 * After 2 secs, It would log '13'.
 */ 
```

### on('edge_added')
Get existing edges inserted at a location, and listen to new ones.

#### Usage
```javascript
abRef.on('edge_added',edgetype,[listenerName],[options],callback)
```
 - __edgetype__ `String` - 'ordered' or 'named'
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation of `on('properties')`
 - __options__ `Object`
     - __limit__ How many existing edges to fetch - only for ordered edges
     - __startAt__ `Number` - Priority to start with - only for ordered edges
     - __endAt__ `Number` - Priority to end with - only for ordered edges
     - __skip__  `Number` - Skip initial edges - only for ordered edges
     - __noData__ `Boolean` - Whether to include the data stored at the vertex where the edge points 
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - pointing to path of the edge
    - __[snapObj]__ `Vertex Snapshot` - Snapshot of the data stored in the vertex, where the edge points. Take a look at the documentation of `Vertex Snapshot`

`snapObj` will not be passed if `{noData: true}` is passed as the options to the listener.

`startAt` and `limit` are only effective for retrieving the existing properties. New edges will be returned regardless of their index.

#### Returns
`String` - the listener's name and can be used to turn the listener off.

#### Example
```javascript
TODO: change according to the method signature
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
abRef.on('edge_removed',edgetype,[listenerName],callback)
```
 - - __edgetype__ `String` - 'ordered' or 'named'
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation of `on('properties')`.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - pointing to path of the edge
    - __snapObj__ `Vertex Snapshot` - Snapshot of the data stored in the vertex, where the edge used to point. Take a look at the documentation of `Vertex Snapshot`

#### Returns
`String` - the listener's name and can be used to turn the listener off.

###  on('edge_changed')
If the properties of the vertex, pointed by an existing edge is changed, this event is fired.

#### Usage
```javascript
abRef.on('edge_changed',edgetype,[listenerName],callback)
```
 - __edgetype__ `String` - 'ordered' or 'named'
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation of `on('properties')`.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - pointing to path of the edge
    - __snapObj__ `Vertex Snapshot` - Snapshot of the data stored in the vertex, where the edge points. Take a look at the documentation of `Vertex Snapshot`

For this event to fire, in the background the vertexes pointed by all the edges are listened for __properties__ event, and this would be a costly operation in terms of bandwidth if there are a huge number of edges.

#### Returns
`String` - the listener's name and can be used to turn the listener off.


#### Example
```javascript
TODO: change according to the method signature
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
TODO: only on ordered edges
This event is fired, when the order of an edge is changed. 

When the order of an edge is manually changed by calling `abRef.edges.add()` on an existing edge, also the other edges are moved, either they are shifted upward or downward by '1' in the order. This event is fired for all the edges, moved manually or automatically.

#### Usage
```javascript
abRef.on('edge_moved',[listenerName],callback)
```
 - __listenerName__ _(Optional)_ `String` - Name given to the listener. For details, take a look at the documentation of `on('properties')`.
 - __listenDepth__ `Number` - The depth of edges up to which listen for data changes. Default value is `0`, meaning no listening on the properties.
 - __callback__ `Function` - will be passed these as arguments:
    - __error__ `Boolean/String`
    - __abRef__ `Appbase Reference` - pointing to path of the edge
    - __snapObj__ `Vertex Snapshot` - Snapshot of the data stored in the vertex, where the edge points. Take a look at the documentation of `Vertex Snapshot`
 
#### Returns
`String` - the listener's name and can be used to turn the listener off.



### off()
Turn off the listeners on an event.

#### Usage
```javascript
abRef.off([event])
```
 - __event__ _(optional)_ `String` - All the listeners on this event, will be turned off. If no event is given, all the listeners on all the events will be turned off.

#### Returns
`Array` - containing listeners' names, which have been turned off.

### offWithName()
Turn off a listener by its name.

#### Usage
```javascript
offWithName(listenerName)
```
 - __listenerName__ `String` - The unique name given to the listener while calling `on(event)`.

### out()
TODO: text
Get an _Appbase Reference_ pointing to a edge of the current vertex. This is just string manipulation on the path, and the reference will be returned even if the edge doesn't exist, but read/write operations will fail.

#### Usage
```javascript
abRef.out(edgename)
```

#### Returns
`Appbase Reference`

#### Example
```javascript
var prisonerRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne');
var toolRef = prisonerRef.refToEdge('rock_hammer');

/* `toolRef` points to the the path:
 * 'https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer'
 */
```

### in()
TODO: explain
Go up in path and get an _Appbase Reference_.

#### Usage
```javascript
abRef.in()
```
Throws an error of the vertex has no upedge.

#### Returns
`Appbase Reference`

#### Example
```javascript
var toolRef = Appbase.ref('https://shawshank.api.appbase.io/prisoner/andy_dufresne/rock_hammer');
var prisonerRef = toolRef.refToUpedge();

/* `prisonerRef` points to the the path:
 * 'https://shawshank.api.appbase.io/prisoner/andy_dufresne'
 */
 
var newRef = prisonerRef.refToUpedge(); //Throws an error
```

## Vertex Snapshot
_Vertex Snapshot_ is an immutable copy of the data at a location in _Appbase_. It is passed to callbacks in all event firing. It can't be modified and will never change. To modify data, use an Appbase reference.

### Snapshot Methods
TODO: index -> priority
Method | Returns
-|-
properties() | prop-value pairs in the form of a JavaScript object
prevProperties() | the previous version of prop-value pairs
namespace() | the _namespace_ of the vertex
name() | the edge name with which the vertex is stored in the current path
index() | index of this vertex in the current path
prevIndex() | the previous index of this vertex in the current path

The following table shows what exact data would be returned by the methods in different kind of events.

Method | value | edge_added | edge_removed | edge_changed | edge_moved
-|-|-|-|-|-
properties() | properties of the **vertex being listened** | properties of the vertex pointed by **the edge** | **null** | **new** properties of the vertex pointed by **the edge** | properties of the vertex pointed by **the edge**
prevProperties() | **null** when the event is fired for **the first time** , *and*, **previous version** of properties when they are **changed later on** | **null** | properties of the vertex pointed by **the edge being deleted** | **previous version** of properties of the vertex pointed by the edge | same as **properties()**
index() | **index** of the edge pointing to **this vertex** in the current path | **index** of **the edge** | **null** | **index** of **the edge** | **current** index of **the edge**
prevIndex() | same as **index()** | **null** | **index** of **the edge being deleted** | same as **index()**  | **previous** index of **the edge**



## Privileged Methods
These methods shouldn't be a necessity in the normal application working. The use of these methods can be controlled via security rules.

### Appbase.rename()

Allows renaming of namespaces, vertex primary keys and moving a vertex to a different namespace.


#### Usage
```javascript
Appbase.rename(old,new)
```
 - __old__ `String/Appbase Reference` - old '/namespace', or '/namespace/pk' or `Appbase Reference` poiting to '/namespace/pk'
 - __new__ `String` - new '/namespace', or '/namespace/pk'

The old /namespace or /namespace/pk must exist and the new one must not.

The edges pointing to the vertex being renamed will still work.

#### Returns
`Appbase Reference` - pointing to the new path, if renaming of a vertex is happening,  
`undefined` otherwise.

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

# appbase.io API Documentation

# Appbase clientside JS API

TODO: 

 1. How to change roles
 2. Transactions
 3. What happens while listning into levels

## Datatypes 
This doesn't change. Let the developer set Boolean/Number in a property. We can store it as string on the server, that's fine.

  * Boolean
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
var ab = Appbase.new('User','andy_dufresne',callback);
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
var ab = Appbase.ref('User/andy_dufresne/rock_hammer');
```

### Appbase.signup()

#### Usage
```javascript
Appbase.signup(mail,password,callback)
```
 - __mail__ `String`
 - __password__ `String`
 - __callback__ `Function` - with arg error

### Appbase.signin()

#### Usage
```javascript
Appbase.signin(email,password,callback)
```
 - __mail__ `String`
 - __password__ `String`
 - __callback__ `Function` - with arg error

### Appbase.signout()

#### Usage
```javascript
Appbase.signout()
```

### Appbase.onAuth()
Register a callback when the user is authenticated.

#### Usage
```javascript
Appbase.onAuth(callback)
```
argument given to callback is an object with following properties.

 - __user__ `Object` - with properties: email
 - __isNew__ `Boolean` - whether the user just signed up

### Appbase.onUnauth()
Register a callback when the user is unauthenticated.

#### Usage
```javascript
Appbase.onUnauth(callback)
```
argument given to callback is an object with following properties.

 - __isSessionExpired__ `Boolean` - whether the session expiration caused unauthentication
 - __isSignedOut__ `Boolean` - whether the Appbase.signout() was called
 - __isSigninRequired__ `Boolean` - whether the user was never logged in

### Appbase.sendPasswordResetMail()

#### Usage
```javascript
Appbase.sendPasswordResetMail(mail,callback)
```

### Appbase.changePassword()

#### Usage
```javascript
Appbase.changepassword(oldPwd,newPwd,callback)
```

### Appbase.transaction() //igonre for now
All operations either succeed, or fail.

#### Usage
```javascript
Appbase.transaction(function(){
  operations;
}, callback)
```

callback parameter is an object with these properties:

 - __error__ `String` 
 - __AbRef__ `Appbase Object`: Reference to Appbase object which caused the error

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
insert(property, value, [index], [callback])
```
 - __property__ `String`
 - __value__ `String/Boolean/Number/null/undefined/Appbase Reference` - Giving null or undefined as the value removes the property
 - __index__ `Number` - The property is inserted at the index. i.e., Things after that are moved back by 1. Push is same as inserting at 0. Enqueue is same as inserting at -1.
 - __callback__ - with args: (err,new obj snapshot)

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

### getRefToProperty() // what if the property is primitive and not an object?
Get reference to an object, by property

#### Usage
```javascript
getRefToProperty(property)
```

### destroy() //deleteSelf()
Delete the whole object, references to this object in other objects will be deleted on read.

#### Usage
```javascript
destroy()
```

### getRefToIndex()
Get reference to the object by index

#### Usage
```javascript
getRefToIndex(index)
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

### on('object_added') // need to discuss on the options
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

### consistant() // rename

#### Usage
[https://www.firebase.com/docs/javascript/firebase/transaction.html][1]



## Appbase Snapshot Object
This is will be passed to callbacks in all event firing.

[https://www.firebase.com/docs/javascript/datasnapshot/][2]


  [1]: https://www.firebase.com/docs/javascript/firebase/transaction.html
  [2]: https://www.firebase.com/docs/javascript/datasnapshot/
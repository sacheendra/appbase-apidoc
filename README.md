# appbase.io API Documentation

# Appbase clientside JS API

## Datatypes
  * Boolean
  * Number
  * String
  * Object

## Appbase Global

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

### Appbase.transaction()
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

## Appbase Object
Represents a reference to an object

### path()
To know what path this reference points to.

#### Usage
```javascript
path()
```

#### Returns
`String` - The path.


### set()
Sets value for a property

#### Usage
```javascript
set(property, value/object,options,[callback])
```
 - __property__ `String`
 - __value__ `String/Boolean/Number/null/undefined` - Giving null or undefined as the value removes the property
 - __options__ `Object` - with properties: index
 - __callback__ - with arg error

#### Returns
The same `Appbase` reference, to allow chaining of set methods


### move()
Move an item to an index, indices of others automatically change.
#### Usage
```javascript
move(property,index)
```

#### Returns
The same `Appbase` reference, to allow chaining of set methods

### child()
Get reference to a child object

#### Usage
```javascript
child(property)
```
Throws error if child doesn't exist.

### childAt()
Get reference to a child object

#### Usage
```javascript
childAt(index)
```
Throws error if index doesn't exist.

### parent()
Reference to parent object in the current path of the entity

#### Usage
```javascript
parent()
```

### on('value')

#### Usage
```javascript
on('value',options,callback)
```
options is an object with properties.

 - __levels__ `Number` - include data of referenced objects up to this depth 
 - __limits__ `Array` - list of numbers, specifying how many objects should be included from each level. eg. levels:3, limits:[5,2,2] = total objects included: 5*2*2

### on('child')
get existing children, listen to new ones

#### Usage
```javascript
on('value',options,callback)
```
options is an object with properties:

 - __levels__ `Number` - include data of referenced objects up to this depth 
 - __limits__ `Array` - list of numbers, specifying how many objects should be included from each level.
 - __startAt__ `Number` - index to start with
 - __overflow__ `Function` - when level one limit is hit, objects from the bottom are as argument to this function. - This is used for removing objects from the view automatically

### on('child_remove')
listen to removal of children

#### Usage
```javascript
on('child_remove',callback)
```

### off()
#### Usage
```javascript
off(event)
```
 - __event__ `String` -

### push()

#### Usage
```javascript
push(AbRef,callback)
```
 - __AbRef__ - `Appbase Object`: Object to push at the top, property name will be uuid of AbRef


### pushNew()

#### Usage
```javascript
pushNew(collection,callback)
```
 - __collection__ - `String`: create a new object of the collection,push it to the top and return its reference


### enqueue()

#### Usage
```javascript
enqueue(AbRef,callback)
```
 - __AbRef__ - `Appbase Object`: Object to put at the bottom, property name will be uuid of AbRef


### enqueueNew()

#### Usage
```javascript
enqueueNew(collection,callback)
```
 - __collection__ - `String`: create a new object of the collection,enqueue it at the bottom and return its reference

### atomic()

#### Usage
[https://www.firebase.com/docs/javascript/firebase/transaction.html][1]


  [1]: https://www.firebase.com/docs/javascript/firebase/transaction.html
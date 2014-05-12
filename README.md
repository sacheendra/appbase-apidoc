# http://appbase.io/ API Documentation

__Read the functions like this:__

__name(arg1: type, arg2: type, ..., ?optional-arg1: type): return-type__

__"string1:string2" = string1 + ":" + string2__

__?thing - means thing is optional__


# Functions in Global scope


## Appbase.new("collection:?key": String): Appbase reference
This function takes the name of a collection and creates a new object in that collection. A reference to the newly created object is returned. The collection is created if it does not already exist. An optional key can attached to the collection string, in which case the object will be created with given key.


## Appbase.ref("collection:key": String): Appbase reference
This function takes a collection-key pair as a string of the format "collection-name:key" and returns a reference to the object if it exists. Returns an error otherwise.


---


# Methods of an Appbase reference

# appbase.io API Documentation

# Appbase clientside JS API

IMPORTANT: Accepted values for index are -1 to (to decide wether to allow discontinuous ordering)

## Datatypes
  * String
  * Counter
  * Object

## Appbase Global

### Appbase.ref(PATH)

### Appbase.createCounter()

### Appbase.add(namespace, [primary_key/appbase reference/appbase object], [callback(err, reference)])
  NOTE: Returns Appbase Reference

## Appbase Reference

### Reference.on('value', [callback(err, Object)])
  NOTE: Fired only when self is changed.

### Reference.on('object_changed', options, [callback(err, Object which was changed)])

### Reference.on('object_added', options, [callback(err, Object which was added)])

### Reference.on('object_removed', options, [callback(err, Object which was removed)])

## Appbase Object

### Object.get(property_name)

#### Returns
  The string/counter if it is one of them. An Appbase Reference to the linked object.

### Object.insert(property_name, string/counter/appbase object/appbase ref, [index], [callback(err, changed_obj)])
  NOTE: The property is inserted at the index. i.e., Things after that are moved back by 1.
  NOTE 2: Push is same as inserting at 0. Enqueue is same as inserting at -1.

#### Returns
  The object with property changed.

### Object.getByIndex(index)
  NOTE: Use index -1 to get last object. 0 to get first.

#### Returns
  Same as Object.get

### Object.remove(property_name, [callback(err, changed_obj)])

### Object.deleteSelf([callback(err)])

## Appbase Counter

### Counter.increment()

### Counter.decrement()

### Counter.listen(callback(value))
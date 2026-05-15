1. Keys of hash functions are immutable. ie they can’t be change.
	1. If you try to assign a mutable  data type as a key (array) you will get a `Type Error: Unhashable type.` To solve this, convert the mutable datatype to immutable. ie array → tuple. 
	2. Strings are immutable. Any operation that appears to modify a string, such as concatenation or using methods like `.replace()`, actually creates a new string object rather than altering the original.
2. In order for a value to be assigned to a key, the key must first be initialized.
	1. To avoid initializing empty list for the keys every time, use `defaultdict()`. It has empty list assigned for keys by default
3. Retrieving data:
	1. `hashmap.values()`: returns a list of values 
	2. `hashmap.keys()`: returns a list of keys
	3. `hashmap.items()`: returns a list of key:value pairs as tuples
	4. Retrieve Values: 
		1. `hashmap.get()`: hashmap.get(key, default_value)
		2. `hashmap[key]`: value = hashmap[key]


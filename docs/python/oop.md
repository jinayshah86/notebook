# Object-oriented programming (OOP)

Singleton classes are not really used as often in Python as in other 
languages. The effect of a singleton is usually better implemented as a global 
variable in a module.

Everything in Python is an object.

Class attributes are shared among all instances, while instance attributes
 are not; therefore, you should use class attributes to provide the states and 
 behaviors to be shared by all instances, and use instance attributes for data 
 that belongs just to one specific object.

When you search for an attribute in an object, if it is not found, Python 
keeps searching in the class that was used to create that object (and keeps 
searching until it's either found or the end of the inheritance chain is 
reached)

Python class does not have a constructor but it has an initializer. It's 
called an initializer since it works on an already-created instance, and 
therefore it's called `__init__`.  It's a magic method, which is run right 
after the object is created. Python objects also have a `__new__` method, 
which is the actual constructor. In practice, it's not so common to have to 
override it though, it's a practice that is mostly used when coding 
metaclasses.

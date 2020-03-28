#### Pickling and Unpickling
The `pickle` module, on the other hand, is not human readable, translates to
bytes, is Python specific, and, thanks to the wonderful Python
introspection capabilities, it supports an extremely large amount of data
types.

Regardless of these differences, though, which you should know when you
consider whether to use one or the other, I think that the most important
concern regarding `pickle` lies in the security threats you are exposed to
when you use it. Unpickling erroneous or malicious data from an untrusted
source can be very dangerous, so if you decide to adopt it in your
application, you need to be extra careful.

**Code:**
```python
import pickle
from dataclasses import dataclass

@dataclass
class Person:
    first_name: str
    last_name: str
    id: int

    def greet(self):
        print(f'Hi, I am {self.first_name} {self.last_name}'
              f' and my ID is {self.id}'
        )

people = [
    Person('Obi-Wan', 'Kenobi', 123),
    Person('Anakin', 'Skywalker', 456),
]

# save data in binary format to a file
with open('data.pickle', 'wb') as stream:
    pickle.dump(people, stream)

# load data from a file
with open('data.pickle', 'rb') as stream:
    peeps = pickle.load(stream)

for person in peeps:
    person.greet()
```

**Output:**
```text
Hi, I am Obi-Wan Kenobi and my ID is 123
Hi, I am Anakin Skywalker and my ID is 456  
```
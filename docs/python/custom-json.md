# Custom JSON Encoder/Decoder

**Code:**

```python
import json

# using class

class ComplexEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, complex):
            return {
                '_meta': '_complex',
                'num': [obj.real, obj.imag],
            }
        return json.JSONEncoder.default(self, obj)

data = {
    'an_int': 42,
    'a_float': 3.14159265,
    'a_complex': 3 + 4j,
}

json_data = json.dumps(data, cls=ComplexEncoder)
print(json_data)

# using method

def object_hook(obj):
    try:
        if obj['_meta'] == '_complex':
            return complex(*obj['num'])
    except (KeyError, TypeError):
        return obj

data_out = json.loads(json_data, object_hook=object_hook)
print(data_out)
```


**Output:**
```bash
{
  "an_int": 42,
  "a_float": 3.14159265,
  "a_complex": {"_meta": "_complex", "num": [3.0, 4.0]}
}
{'an_int': 42, 'a_float': 3.14159265, 'a_complex': (3+4j)}
```

For more information visit [python docs](https://docs.python.org/3.4/library/json.html#json.JSONEncoder)
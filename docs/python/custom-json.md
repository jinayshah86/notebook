# Custom JSON Encoder/Decoder

**Code:**

```python
import json
import pytz
from datetime import datetime
from pprint import pprint

datetimeobj_key = "_datetime"

obj = {
    "date": datetime.now(),
    "datetz": datetime.now(pytz.utc),
    "string": "hello world!",
    "dict": {
        "_meta": datetimeobj_key
    },
    "list": list(range(5)),
}
# using classes
print("using classes".center(50, '-'))


class DateTimeJSONEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return {"_meta": datetimeobj_key, "value": obj.isoformat()}
        return super().default(obj)


dump_obj = json.dumps(obj, cls=DateTimeJSONEncoder, indent=2)
print(dump_obj)


class DateTimeJSONDecoder(json.JSONDecoder):
    def __init__(self, *args, **kwargs):
        super().__init__(object_hook=self.object_hook, *args, **kwargs)

    def object_hook(self, obj):
        try:
            if obj.get("_meta") == datetimeobj_key:
                value = obj["value"]
                return datetime.fromisoformat(value)
        except KeyError:
            return obj
        return obj


load_obj = json.loads(dump_obj, cls=DateTimeJSONDecoder)
pprint(load_obj)

del dump_obj
del load_obj

# using functions
print("using functions".center(50, '-'))


def datetime_json_encoder(obj):
    if isinstance(obj, datetime):
        return {"_meta": datetimeobj_key, "value": obj.isoformat()}
    return obj


dump_obj = json.dumps(obj, default=datetime_json_encoder, indent=2)
print(dump_obj)


def datetime_json_decoder(obj):
    try:
        if obj.get("_meta") == datetimeobj_key:
            value = obj["value"]
            return datetime.fromisoformat(value)
    except (KeyError, ):
        return obj
    return obj


load_obj = json.loads(dump_obj, object_hook=datetime_json_decoder)
pprint(load_obj)
```


**Output:**
```text
------------------using classes-------------------
{
  "date": {
    "_meta": "_datetime",
    "value": "2020-03-28T07:00:09.446641"
  },
  "datetz": {
    "_meta": "_datetime",
    "value": "2020-03-28T07:00:09.446651+00:00"
  },
  "string": "hello world!",
  "dict": {
    "_meta": "_datetime"
  },
  "list": [
    0,
    1,
    2,
    3,
    4
  ]
}
{'date': datetime.datetime(2020, 3, 28, 7, 0, 9, 446641),
 'datetz': datetime.datetime(2020, 3, 28, 7, 0, 9, 446651, tzinfo=datetime.timezone.utc),
 'dict': {'_meta': '_datetime'},
 'list': [0, 1, 2, 3, 4],
 'string': 'hello world!'}
-----------------using functions------------------
{
  "date": {
    "_meta": "_datetime",
    "value": "2020-03-28T07:00:09.446641"
  },
  "datetz": {
    "_meta": "_datetime",
    "value": "2020-03-28T07:00:09.446651+00:00"
  },
  "string": "hello world!",
  "dict": {
    "_meta": "_datetime"
  },
  "list": [
    0,
    1,
    2,
    3,
    4
  ]
}
{'date': datetime.datetime(2020, 3, 28, 7, 0, 9, 446641),
 'datetz': datetime.datetime(2020, 3, 28, 7, 0, 9, 446651, tzinfo=datetime.timezone.utc),
 'dict': {'_meta': '_datetime'},
 'list': [0, 1, 2, 3, 4],
 'string': 'hello world!'}
```

For more information visit [python docs](https://docs.python.org/3.4/library/json.html#json.JSONEncoder)
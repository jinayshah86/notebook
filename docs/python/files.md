### Opening and Reading from file
```python
file_fullpath = 'temp.txt'
filemode = 'rt'  # r: read, t: text
with open(file_fullpath, filemode) as fh:
    for line in fh.readlines():
        print(line.strip())  # remove whitespace and print
```

**"rt"** is the default mode

### Writing to files
```python
with open('print_example.txt', 'w') as fw:
    print('Hey I am printing into a file!!!', file=fw)
```

### File modes

- **r**: read file
- **w**: file is overwritten with an empty file, and the original content is
lost. 
- **x**: open the file for writing only if it doesn't exists, if it exists
raise `FileExistsError`

### Check existence

```python
import os

filename = 'temp.txt'
path = os.path.dirname(os.path.abspath(filename))

print(os.path.isfile(filename))  # True
print(os.path.isdir(path))  # True
print(path)  # /Users/mickey/tutorials/python/files
```

Should you ever need to work with paths in a different way, you can check
out `pathlib`. While `os.path` works with strings, `pathlib` offers classes
representing filesystem paths with semantics appropriate for different
operating systems. 


### Manipulating pathname
**Code:**
```python
# files/paths.py
import os

filename = 'fear.txt'
path = os.path.abspath(filename)

print(path)
print(os.path.basename(path))
print(os.path.dirname(path))
print(os.path.splitext(path))
print(os.path.split(path))

readme_path = os.path.join(
    os.path.dirname(path), '..', '..', 'README.rst')
print(readme_path)
print(os.path.normpath(readme_path))
```
**Output:**
```text
/Users/mickey/tutorials/python/files/fear.txt           # path
fear.txt                                                # basename
/Users/mickey/tutorials/python/files                    # dirname
('/Users/mickey/tutorials/python/files/fear', '.txt')   # splitext
('/Users/mickey/tutorials/python/files', 'fear.txt')    # split
/Users/mickey/tutorials/python/files/../../README.rst   # readme_path
/Users/mickey/tutorials/README.rst                      # normalized
```


### Temporary files and directories
Sometimes, it's very useful to be able to create a temporary directory or
file when running some code. For example, when writing tests that affect
the disk, you can use temporary files and directories to run your logic and
assert that it's correct, and to be sure that at the end of the test run
, the test folder has no leftovers.

**Example:**
```python
import os
from tempfile import NamedTemporaryFile, TemporaryDirectory

with TemporaryDirectory(dir='.') as td:
    print('Temp directory:', td)
    with NamedTemporaryFile(dir=td) as t:
        name = t.name
        print(os.path.abspath(name))
```
**Output:**
```text
Temp directory: ./tmpwa9bdwgo
/Users/mickey/tutorials/python/files/tmpwa9bdwgo/tmp3d45hm46
```
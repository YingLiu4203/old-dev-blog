---
layout: post
title: Beautiful Idiomatic Python
categories:
- Programming
tags:
- Python
---

# Writing Beautiful Idiomatic Python Code

This article summarizes the best practices of Python programing
based on this YouTube video [Transforming Code into Beautiful, Idiomatic Python](http://youtu.be/OSGv2VnC0go).

## Looping

```python

for index in xrange(6):
    print index

colors = ['red', 'green', 'blue', 'yellow']

for color in colors:
    print color

for color in reversed(colors):
    print color

for index, color in enumerate(colors):
    print index, '-->', color

names = ['raymond', 'rachel', 'matthew']

for name, color in zip(names, colors):
    print name, '-->', color

for name, color in izip(names, colors):
    print name, '-->', color
    
# use generator expression
print sum(i ** 2 for i in xrange(10))
```

## Sorting

```python
# reverse sorting
for color in sorted(colors, reverse=True):
    print color

# custom sorting
for color in sorted(colors, key=len):
    print color
    
# iterate until a sentinel value
blocks = []
for block in iter(partial(f.read, 32), ''):
    blocks.append(block)
```

## Dictionary

```python
d = {'matthew': 'blue', 'rachel': 'green', 'raymond': 'red'}

print d.keys()
print d.items()

for k. v in d.iteritems():
    print k, v

d2 = dict(izip(names, colors))
d3 = dict(enumerate(names))

d_color = defaultdict(int)
for color in colors:
    d[color] += 1

d_name1 = {}
for name in names:
    key = len(name)
    d.setdefault(key, []).append(name)

d_name2 = defaultdict(list)
for name in names:
    key = len(name)
    d[key].append(name)

# atomic operation
key, value = d.popitem()

# linking dictionaries
d = defaults.copy()
d.update(os.environ)
d.update(command_line_args)

# should be
d = ChainMap(command_line_args, os.environ, defaults)
```

## Improving Clarity

User keywords to call a function/method: 
`search('name', retweets=False, numtweets=20, popular=True)`

Use named tuples for multiple return values: 
`TestResults = namedtuple('TestResults', ['failed', 'attempted'])`

Un-package sequence in one statement: 
`fname, lname, age = ('Aice', 'Jay', 20)`

Updating multiple state variables: `x, y = y, x + y`

## Improving Efficiency

```python

# use join to concatenating strings
', '.join(names)

# use deque to delete from the frong
names = deque(names)
names.popleft()
names.appendleft('mark')
```

## Using Decorators and Context Managers

Use `with` when use a file, a lock,  exception ignorance.

```python

@contextmanager
def ignored(*exceptions):
    try:
        yield
    except exceptions:
        pass

with ignored(OSError):
    os.remove('temp.txt')
```

    



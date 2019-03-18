### redisco
---
https://github.com/iamteem/redisco

```py
from redisco import models
class Person(models.Model):
  name = models.Attribute(required=True)
  created_at = models.DateTimeField(auto_now_add=True)
  fave_colors = models.ListField(str)
  
person = Person(name="Conchita")
person.is_valid()
person.save()
conchita = Person.objects.filter(name='Conchita')[0]
conchita.name
conchita.created_at


def not_me(field_name, value):
  if value == 'Me':
    return ((field_name, 'it is me'),)

class Person(models.Model):
  name = models.Attributes(required=True, validator=not_me)
  age = models.IntegerField()
  
  def validate(self):
    if self.age and self.age < 21:
      self._errors.append(('age', 'below 21'))

person = Person(name='Me')
person.is_valid()
person.errors

person.objects.all()
Person.objects.filter(name='Conchita')
Person.objects.filter(name='Conchita').first()
Person.objects.all().order('name')
Person.objects.filter(fave_colors='Red')

Person.objects.zfilter(created_at__lt=datetime(2010, 4, 20, 5, 2, 0))
Person.objects.zfilter(created_at__gte=datetime(2010, 4, 20, 5,2, 0))
Person.objects.zfilter(created_at__in=(datetime(2010, 4, 20, 5, 2, 0), datetime(2010, 5, 1)))


from redisco.containers import Set
s = Set('myset')
s.add('apple')
s.add('orange')
s.members
t = Set('nset')
t.add('kiwi')
t.add('guava')
t.members
s.update(t)
s.members

import redis
from redisco.containers import List
l = List('alpha')
l.append('a')
l.append('b')
l.append('c')
'a' in l
'd' in l
len(l)
l.index('b')
l.members

zset = SortedSet('zset')
zset.members
'e' in zset
'a' in zset
zset.rank('d')
zset.rank('b')
zset[1]
zset.add('f', 200)
zset.members
zset.add('d', 99)
zset.members

h = cont.Hash('hkey')
len(h)
h['name'] = "Richard Cypher"
h['real_name'] = "Richard Rahl"
h
h.dict

l = List('mylist')
l.lrange(0, -1)
l.rpush('b')
l.rpush('c')
l.lpush('a')
l.lrange(0, -1)
h = Hash('hkey')
h.hset('name', 'Richard Rahl')
h

import redisco
redisco.connection_setup(host='localhost', port=6380, db=10)

import redis
r = redis.Redis(host='localhost', port=6381)
Set('someset', r)
```

```
```

```
// redisco/containers.py

import collections
from functools import partial

class Container(object):
  """
  """
  
  def __init__(self, key, db=None, pipeline=None):
    self._db

```



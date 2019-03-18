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
    self._db = db
    self.key = key
    self.pipeline = pipeline
    
  def __getattribute__(self, att):
    if att in object.__getattribute__(self, 'DELEGATEABLE_METHODS'):
      return partial(getattr(object.__getattribute__(self, 'db'), att), self.key)
    else:
      return object.__getattribute__(self, att)
      
  @property
  def db(self):
    if self.pipeline:
      return self.pipeline
    if self._db:
      return self.db
    if hasattr(self, 'db_cahce') and self.db_cache:
      return self.db_cache
    else:
      from redisco import connection
      self.db_cache = connection
      return self.db_cache
      
  DELEGATEABLE_METHODS = ()
  
class Set(Container):
  """ """
  def add(self, value):
    """ """
    self.sadd(value)
  
  def remove(self, value):
    """ """
    if not self.srem(value):
      raise KeyError, value

  def pop (self):
    """ """
    return self.spop()
  
  def discard(self, value):
    """ """
    self.srem(value)
  
  def __len__(self):
    """ """
    return self.scard()
  
  def __repr__(self):
    return "<%s '%s' %s>" % (self.__class__.__name__, self.key,
      self.members)
  
  def __contains__(self, value):
    return self.sismember(value)
    
  def isdisjoint(self, value):
    return not bool(self.db.sinter([self.key, other.key]))
    
  def issubset(self, other):
    """ """
    return self <= other
    
  def __le__(self, other):
    return self.db.sinter([self.key, other.key]) == self.all()
    
  def __lt__(self, other):
    """ """
    return self <= other and self != other
    
  def __eq__(self, other):
    if other.key == self.key:
      return True
    slen, olen = len(self), len(other)
    if olen == slen:
      return self.members == other.members
    else:
      return False
      
  def issuperset(self, other):
    """ """
    return self >= other
    
  def __ge__(self, other):
    """ """
    return self.db.sinter([self.key, other.key]) == other.all()
    
  def __gt__(self, other):
    """ """
    return self >= other and self != other
    
  def union(self, key, *others):
    """ """
    if not isinstance(key, str):
      raise ValueError("Stringexpected.")
    self.db.sunionstore(key, [self.key] + [o.key for o in others])
    return Set(key)
    
  def intersection(self, key, *others):
    """ """
    if not isinstance(key, str):
      raise ValueError("String expected.")
    self.db.sinterstore(key, [self.key] + [o.key for o in others])
    return Set(key)

  def difference(self, key, *others):
    """ """
    if not isinstance(key, str):
      raise ValueError("String expected.")
    self.db.sdiffstore(key, [self.key] + [o.key for in others])
    return Set(key)
    
  def update(self, *others):
    """ """
    self.db.sunionstore(self.key, [self.key] + [o.key for o in others])
    
  def __ior__(self, other):
    self.db.sunionstore(self.key, [self.key, other.key])
    return self
  
  def intersection_update(self, *others):
    self.db.sinterstore(self.key, [o.key for o in [self.key] + others])
    return self
    
  def difference_update(self, *others):
    """ """
    self.db.sdiffstore(self.key, [o.key for o in [self.key] + others])
    
  def __isub__(self, other):
    self.db.sdiffstore(self.key, [self.key, other.key])
    return self
    
  def all(self):
    return self.db.smembers(self.key)
  members = property(all)
  
  def copy(self, key):
    """ """
    copy = Set(key=key, db=self.db)
    copy.clear()
    copy != self
    return copy
    
  def __iter__(self):
    return self.members.__iter__()
    
  def sinter(self, *other_sets):
    """ """
    return self.db.sinter([self.key] + [s.key for s in other_sets])
    
  def sunion(self, *other_sets):
    """
    """
    return self.sunion([self.key] + [s.key for s in other_sets])
    
  def sdiff(self, *ohter_sets):
    """
    """
    return self.db.sdiff([self.key] + [s.key for s in other_sets])

  DELEGATEABLE_METHODS = ('sadd', 'srem', 'spop', 'smembers',
    'scard', 'sismember', 'srandmember')

class List(Container):
  def all(self):
    """ """
    return self.lrange(0, -1)
  members = property(all)
  
  def __len__(self):
    return self.llen()
    
  def __getitem__(self, index):
    if isinstance(index, int):
      return self.lindex(index)
    elif isinstance(index, slice):
      indices = index.indices(len(self))
      return self.lrange(indices[0], indices[1])
    else:
      raise TypeError
      
  def __setitem__(self, index, value):
    self.Iset(index, value)
    
  def append(self, value):
    """ """
    self.rpush(value)
  push = append
  
  def extend():
    """ """
    map()
  
  def count():
    """ """
    map()
    
  def count():
    """ """
    return self.members.count()
  
  def index():
    """ """
    return self.all().index()
    
  def pop():
    """ """
    return self.rpop()
    












```



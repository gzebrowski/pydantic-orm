# Pydantic-orm

Orm similar to django-orm (all concepts were based on this)
For now it works only with Sqlite3 - but in the future should also work with postgres, mysql and mongoDB.

How to use this simple orm engine...


```
import datetime
import sqlite3
from typing import Optional

from pydantic import Field

from orm import AbstractModel, register_model, prepare_db, Q
```

```
def connect_db():
    connection = sqlite3.connect('test_orm.db')
    connection.row_factory = sqlite3.Row
    return connection


conn = connect_db()


@register_model
class Threads(AbstractModel):
    gpt_model: str
    name: str
    created_at: datetime.datetime = Field(json_schema_extra={'auto_now_add': True})

    @ classmethod
    def _get_connection(cls):
        return conn


@register_model
class Messages(AbstractModel):
    thread_id: int = Field(json_schema_extra={'Relation': 'Threads'})  # creates foreign key
    created_at: datetime.datetime = Field(json_schema_extra={'auto_now_add': True})  # automatically adds timestamp
    message: str
    role: str
    cost: Optional[float]  # This field is automatically nullable

    @ classmethod
    def _get_connection(cls):
        return conn
```


```
connect_db()  # creates connection to sqlite3
prepare_db()  # creates tables in db for all models decorated by @register_model if they don't exist yet…
```

```
thread_id = Threads.create(name='my first conversation')
Messages.create(thread_id=thread_id, message='Hello', role='user')
Messages.create(thread_id=thread_id, message='My name is John Smith', role='user', cost=2.2)
Messages.create(thread_id=thread_id, message='What is value of PI?', role='user', cost=7)
Messages.create(thread_id=thread_id, message='Pi is 3.14', role='assistant', cost=5.6)
```

### Queryset.filter Queryset.count and slices (that are translated into LIMIT and OFFSET)
```
qset1 = Messages.objects().filter(
    message__contains='wynosi', role='user', cost__isnull=False, cost__gt=1)[0:3]
print(qset1.query, qset1.count())
```

### Queryset.first method

```
obj = Messages.objects().filter(
    message__contains='wynosi', role='user', cost__isnull=False, cost__gt=1).first()
print(obj)
```

### we

```
obj.cost = 12
obj.save()
```

### we can access fields that are foreign keys

```
print(obj.thread.name)
```

### object delete method

```
obj2 = Messages.objects().order_by('-id').first()
obj2.delete()
```

### Queryset delete method

```
Messages.objects().filter(id__gt=100).delete()
```

### Q object to make possible use OR-s in queries
```
qset3 = Messages.objects().filter(Q(cost=1) | Q(role='user')).exclude(id=2)
print(qset3.query)
```

### Queryset lookups

```
qset3 = Messages.objects().filter(thread__name__contains='conversation')
for obj in qset3:
    print(obj.message, obj.thread.name)
```

There are a few lookups supported:
- contains (as value - string)
- isnull (as value - True or False)
- notin  (as a value we pass list)
- in (as a value we pass list)
- startswith (as value - string)
- endswith (as value - string)
- gt (greater than)
- lt  (less than)
- gte (greater than or exact)
- lte (less than or exact)


### Queryset.raw method
```
for obj in Messages.objects().raw('Select * from messages left join threads On messages.thread_id=threads.id'):
    print(dict(obj))
```

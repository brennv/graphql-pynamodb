Please read [UPGRADE-v1.0.md](https://github.com/graphql-python/graphene/blob/master/UPGRADE-v1.0.md)
to learn how to upgrade to Graphene `1.0`.

---

# ![Graphene Logo](http://graphene-python.org/favicon.png) Graphene-PynamoDB [![Build Status](https://travis-ci.org/yfilali/graphql-pynamodb.svg?branch=master)](https://travis-ci.org/yfilali/graphql-pynamodb) [![Coverage Status](https://coveralls.io/repos/github/yfilali/graphql-pynamodb/badge.svg?branch=master)](https://coveralls.io/github/yfilali/graphql-pynamodb?branch=master)


A [PynamoDB](http://pynamodb.readthedocs.io/) integration for [Graphene](http://graphene-python.org/).

## Installation

For instaling graphene, just run this command in your shell

```bash
pip install graphene-pynamodb
```

## Examples

Here is a simple PynamoDB model:

```python
from uuid import uuid4
from pynamodb.attributes import UnicodeAttribute
from pynamodb.models import Model


class User(Model):
    class Meta:
        table_name = "my_users"
        host = "http://localhost:8000"

    id = UnicodeAttribute(hash_key=True)
    name = UnicodeAttribute(null=False)


if not User.exists():
    User.create_table(read_capacity_units=1, write_capacity_units=1, wait=True)
    User(id=str(uuid4()), name="John Snow").save()
    User(id=str(uuid4()), name="Daenerys Targaryen").save()

```

To create a GraphQL schema for it you simply have to write the following:

```python
import graphene
from graphene_pynamodb import PynamoObjectType


class UserNode(PynamoObjectType):
    class Meta:
        model = User
        interfaces = (graphene.Node,)


class Query(graphene.ObjectType):
    users = graphene.List(UserNode)

    def resolve_users(self, args, context, info):
        return User.scan()


schema = graphene.Schema(query=Query)
```

Then you can simply query the schema:

```python
query = '''
    query {
      users {
        name
      }
    }
'''
result = schema.execute(query)
```

To learn more check out the following [examples](examples/):

* **Full example**: [Flask PynamoDB example](examples/flask_pynamodb)


## Contributing

After cloning this repo, ensure dependencies are installed by running:

```sh
python setup.py install
```

After developing, the full test suite can be evaluated by running:

```sh
python setup.py test # Use --pytest-args="-v -s" for verbose mode
```

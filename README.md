# Marshmallow DataMapper For DynamoDB

This library is a providing a datamapper to store marshmallow objects in Amazon DynamoDB.

## Getting started

```python

from marshmallow import (
    fields,
    post_load
)

class Address:

    def __init__(self, address_id, address, address2, city, state, zipcode):
        self.address_id = address_id
        self.address = address
        self.address2 = address2
        self.city = city
        self.state = state
        self.zipcode = zipcode

    def __repr__(self):
        return AddresssSchema().dumps(self)
        


class AddressSchema(Schema):

    class Mapper:
        table_name = "address"
        hash_key = "address_id"
        indexes = {
            "city_index" : ["state", "city"]
        }

    address_id = fields.Str(required=True)
    address = fields.Str(required=True)
    address2 = Version(required=True)
    city = fields.Str(required=True)
    state = fields.Str(required=False)
    zipcode = fields.Str(required=False)

    @post_load
    def load_address(self, data=None, **kwargs):
        return Address(**data) if data else None

```

With domain classes defined, you can interact with records in DynamoDB via an instance of `data_mapper`:

```python

from marshmallow_datamapper import DataMapper

address_datamapper = DataMapper("us-east-1", AddressSchema())

```

### Supported operations

Using the `mapper` object and `AddressSchema` class defined above, you can perform the following operations:

#### `put`

Creates (or overwrites) an item in the table

```python

address = Address(
    address_id = "0208aa09-0b5f-439f-915d-db61a9cd8664",
    address = "123 Evergreen Terrace",
    address2 = "Apt B",
    city = "Springfield",
    state = "MO",
    zipcode = "43012"
)

address_datamapper.put(address)

```

#### `get`

Retrieves an item from DynamoDB

```python
address_id = "0208aa09-0b5f-439f-915d-db61a9cd8664",

address_datamapper.get({
    "address_id": address_id
})
```



#### `delete`

Removes an item from the table
```python

address_datamapper.delete({
    "address_id": address_id
})

```

#### `scan`

Lists the items in a table or index

```python

all_addresses = address_datamapper.get({})


addresses_by_city = address_datamapper.get({
    "index" : "city_index",
})

```

#### `query`

Finds a specific item (or range of items) in a table or index

```python

springfield_addresses = address_datamapper.query({
    "index" : "city_index",
    "city" : "Springfield",
    "state" : "MO"
})
```

#### Batch operations

The mapper also supports batch operations. Under the hood, the batch will automatically be split into chunks that fall within DynamoDB's limits (25 for `batchPut` and `batchDelete`, 100 for `batchGet`). The items can belong to any number of tables, and exponential backoff for unprocessed items is handled automatically.

##### `batch_put`

Creates (or overwrites) multiple items in the table

```python

addressses = [address1, address2]
address_datamapper.batch_put(addresses)

```

##### `batch_get`

Fetches multiple items from the table

```typescript
address_ids = [
     "0208aa09-0b5f-439f-915d-db61a9cd8664", "df430694-7677-4d1e-bbc7-53ec34f9a7f9"
]

addresses = address_datamapper.batch_get(address_ids)

```

**NB:** Only items that exist in the table will be retrieved. If a key is not found, it will be omitted from the result.

##### `batch_delete`

Removes multiple items from the table, returning them upon successful deletion

```
addresses_removed = address_datamapper.batch_delete(address_ids)
```


#### Operations with Expressions

##### Aplication example

```python

address_datamapper.update(key={
        "address_id" : "0208aa09-0b5f-439f-915d-db61a9cd8664"
    },
    update={
        "address": "234 Evergreen Terrace"
    },
    return_values="UPDATED_NEW"
}

``` 


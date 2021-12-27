# Entity

## What is Entity

Entity is a Kotlin MongoDB Interface.

## How does it work

To define an Entity data class:

1. Define a data class
2. Add `@Entity.DatabaseName` and `@Entity.CollectionName` annotations to the data class
3. Extend `Entity`

As such:

```kotlin
package entity

import Entity
import org.bson.types.ObjectId
import kotlin.collections.ArrayList

@Entity.DatabaseName("user")
@Entity.CollectionName("user")
data class User(
    val name: UserName,
    val age: Number,
    var status: UserStatus = UserStatus.ACTIVE,
    val countriesVisited: ArrayList<String> = arrayListOf(),
    override val _id: ObjectId = ObjectId()
) : Entity()

data class UserName(val firstName: String, val lastName: String)

enum class UserStatus {
    ACTIVE,
    INACTIVE
}
```

After defining an Entity, you can easily create and insert an Entity using:
```kotlin
var user = User(
    UserName("Emile", "Steenkamp"),
    23,
    UserStatus.ACTIVE,
    arrayListOf("ZA", "NL")
).insert()
```

The above defined `User` object will be saved in the database as:
```javascript
{
  "_id": ObjectId("6153263f8aedab15aa1f44d4"),
  "age": 23,
  "countriesVisited": [ "ZA", "NL" ],
  "name": {
    "firstName": "Emile",
    "lastName": "Steenkamp"
  },
  "status": 0
}
```

After inserting the Entity, we can find an Entity by its `_id`:
```kotlin
val user = EntityQuery.findById<User>(ObjectId("6153263f8aedab15aa1f44d4"))
```

Or find all Entities by a filter:
```kotlin
val filter = BasicDBObject()
val users = EntityQuery.find<User>(filter)
```

Update an Entity:
```kotlin
user.status = UserStatus.INACTIVE
user = user.update()
```

## Mapping

The Entity ORM uses code reflection to map Entity Classes to MongoDB Documents.
The Document will be generated by mapping the Class property name as the key
and the value of the Class property to the value. Supported Mapping values
types are:

Type                 |Info
---------------------|-------------
null                 |
Number               |
Boolean              |
Timestamp            |Timestamps are converted to and stored as a ISODate object, so the usage of Date is preferred over Timestamp
Symbol               |
Date                 |
ObjectId             |
Binary               |
Code                 |
BsonRegularExpression|
Enum<*>              |An Enum will be stored in the database using it's ordinal value
ArrayList<*>         |ArrayList will map all Items in the ArrayList using the same mapping, except nested ArrayLists are not possible yet
data class           |A data class Document will recursively be created by using the same mapping

When a Document is loaded, the primary constructor of the Entity Class is
called, and the document values are mapped to the Entity Class argument

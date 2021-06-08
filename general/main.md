---
theme: main 
paginate: true 
backgroundColor: #fff

marp: true
---

![bg vertical 70% 70%](https://raw.githubusercontent.com/roman-right/beanie/main/assets/logo/with_text.svg)

---

Asynchronous Python ODM for MongoDB

https://github.com/roman-right/beanie

---
![bg](#aaa)
![bg 50% 70%](https://raw.githubusercontent.com/samuelcolvin/pydantic/master/docs/logo-white.svg)
![bg 50% 70%](images/motor.png)

---

```python
class User(BaseModel):
    id: int
    name = 'John Doe'
    signup_ts: Optional[datetime] = None
    friends: List[int] = []
```

---

# Picture of the document

---
![bg left:40% 120%](images/figure.jpg)

```python
from typing import Optional
from beanie import Document, Indexed


class Person(Document):
    name: Indexed(str)
    twitter: Optional[str]
    karma: int = 0

    class Collection:
        name = "persons"
```

---
![bg right:40%](images/ironman.gif)

```python
person = Person(
    name="Tony Stark",
    twitter="@Iron_Man"
)
await person.create()
print(person)

```

Output:

```python
id = ObjectId('60be669a4938de2829846785')
name = 'Tony Stark'
twitter = '@Iron_Man'
karma = 0
```

---
![bg left 100%](images/id.jpg)

```python
from pydantic import Field
from uuid import UUID, uuid4


class Sample(Document):
    id: UUID = Field(default_factory=uuid4())
```

---

![bg right 100%](images/nested.gif)

```python
from pydantic import BaseModel


class Team(BaseModel):
    name: str
    description: str


class Person(Document):
    name: Indexed(str)
    twitter: Optional[str]
    team: Team
```

---

![bg](images/find.jpeg)

---

# Get

![bg left:40% 130%](images/pick_up.gif)

```python
person = await Person.get(ObjectId("60be6d2f71afa4600aaaaeb2"))
```

Result:

```python
id = ObjectId('60be6d2f71afa4600aaaaeb2')
name = 'Tony Stark'
twitter = '@Iron_Man'
team = Team(
    name='Avengers',
    description='A group of strong enough people'
)
```

---

# Find

![bg left:40% 130%](images/agents.jpeg)
async generator

```python
async for person in Person.find(
        Product.team.name == "Agents"
):
    print(person)
```

list

```python
agents = await Person.find(
    Product.team.name == "Agents"
).to_list()
```

---

# Find One

![bg left:40% 80%](images/neo.png)

```python
bar = await Person.find_one(
    Person.name == "Thomas Anderson",
    Person.team.name = "Resistance"
)
```

---

![bg left:40% 70%](images/comparison2.jpeg)

#### Supported comparison operatos:

```python
==, >, >=, <, <=, !=
```

#### Example

```python
good_guys = await Person.find(
    Person.karma > 10
).to_list()
```

---

```python
from beanie.operators import In

red_and_blue = await Person.find(
    In(Person.team.name, ["Blue", "Red"]))
).to_list()
```

---

![bg left:40% ](images/mongo_logo_1.jpeg)

```python
people = await Person.find(
    {"name": "Joe"}
).to_list()
```

---
![bg](images/sort.jpeg)

---

![bg left:40% 90%](images/plus_minus.png)

```python
people = await Person.find(
    Person.team.name = "Red"
).sort(
    -Person.karma
).to_list()
```



```python
people = await Person.find(
    Person.team.name = "Red"
).sort(
    "-karma"
).to_list()
```



```python
people = await Product.all().sort(
    [
        (Person.karma, pymongo.DESCENDING),
        (Person.name, pymongo.ASCENDING),
    ]
).to_list()
```

---

![bg right:30% 70%](images/pagination.png)

```python
people = await Person.find(
    Person.team.name == "Blue"
).skip(2).to_list()
```

```python
people = await Person.find(
    Person.team.name == "Red"
).limit(2).to_list()
```

---

# Projections

![bg left:40%](images/projection.jpeg)

```python
class PersonShortView(BaseModel):
    name: str
    karma: int


people = await Person.find(
    Person.team.name == "Red"
).project(PersonShortView).to_list()
```

---

# Update

pic here

---

# One

![bg left:40% 100%](images/batman.jpeg)

```python
await Person.find_one(
    Person.name == "Bruce Wayne"
).update(Set({Person.twitter: "@TheBatman"}))

```

---

# Many

![bg right:40% 110%](images/squad.jpeg)

```python
await Person.find(
    Person.team.name == "Villains"
).update(Inc({Person.karma: -10}))
```

---

# Preset methods

#### Methods

```python

set(), inc(), current_date()

```

#### Example

```python
await Person.find(
    Person.team.name == "Red"
).set({Person.karma: 100})

```

---

# Wrappers

```python
from beanie.operators import Inc

await Person.find(
    Person.team.name == "Blue"
).update(
    Inc({Person.karma: 15})
)
```

---

# Native MongoDB Syntax

![bg left:40%](images/mongo_logo_1.jpeg)

```python
await Person.find(
    Person.team.name == "Blue"
).update(
    {"$inc": {Person.karma: 15}
)
```

---

# Aggregations

---

```python
class OutputItem(BaseModel):
    id: str = Field(None, alias="_id")
    average_karma: int


result = await Person.aggregate(
    [
        {
            "$group": {
                "_id": "$team.name",
                "average_karma": {"$avg": "$karma"}
            }
        }
    ],
    projection_model=OutputItem
).to_list()
```

---

Result:

```python
[OutputItem(id='Blue', average_karma=4),
 OutputItem(id='Red', average_karma=8),
 OutputItem(id='Avengers', average_karma=11),
 OutputItem(id='Agents', average_karma=1),
 OutputItem(id='Resistance', average_karma=15)]
```

---

# Filter

```python
class OutputItem(BaseModel):
    id: str = Field(None, alias="_id")
    min_karma: int


result = await Person.find(
    Person.karma > 0
).aggregate(
    [{"$group": {"_id": "$team.name", "min_karma": {"$min": "$karma"}}}],
    projection_model=OutputItem
).to_list()
```

---

```python
result = await Person.aggregate(
    [
        {
            "$group": {
                "_id": "$team.name",
                "min_karma": {"$min": "$karma"}
            }
        }
    ]
).to_list()
```

---

```python
[{'_id': 'Blue', 'min_karma': 3.0},
 {'_id': 'Red', 'min_karma': 8.0},
 {'_id': 'Avengers', 'min_karma': 11.0},
 {'_id': 'Agents', 'min_karma': 1.0},
 {'_id': 'Resistance', 'min_karma': 15.0}]
```

---

#### Methods

```python
sum(), avg(), min(), max()
```

#### Example

```python

avg_karma: float = Person.find(
    Person.team.name == "RED"
).avg(Person.karma)

```

---

![bg 100%](images/fastapi.png)

---

```python
@router.post("/persons/", response_model=Person)
async def create(person: Person):
    await person.create()
    return person
```

---

```python
@router.get("/persons/", response_model=List[Person])
async def get_list():
    persons = await Person.all().to_list()
    return persons
```

---

```python
@router.put("/persons/{person_id}/update_karma", response_model=Person)
async def update_karma(person_id: ObjectId, karma: float):
    person = Person.get(person_id)
    await person.inc({Person.karma: karma})
    return person
```

---

```python
@router.delete("/persons/{person_id}/update_karma")
async def update_karma(person_id: ObjectId, karma: float):
    person = await Person.find_one(Person.id == person_id).delete()
    return {"status": "OK"}
```

---

# Plans

---

# Relations

```python

class Window(Document):
    width: int
    height: int


class House(Document):
    address: str
    windows: List[Window]
    favorite_window: Window

```

---
![bg](images/stickers.jpg)

---

# Contact me

- Twiter: @roman_the_right
- GitHub: roman-right
- Mail: roman-right@protonmail.com

![bg right:47% 80%](images/qr-1623078319448.png)

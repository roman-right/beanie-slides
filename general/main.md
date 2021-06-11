---
theme: main 
paginate: true 
backgroundColor: #fff

marp: true
---

![bg vertical 70% 70%](https://raw.githubusercontent.com/roman-right/beanie/main/assets/logo/with_text.svg)

---


![bg](images/mongo_db.jpeg)

---

![bg left:40% 100%](images/json.png)
```json
[
    {
        "_id": {
            "$oid": "60c1e0a8b889bff62d97b72d"
        },
        "name": "Tony Stark",
        "twitter": "@Iron_Man",
        "team": {
            "name": "Avengers",
            "description": "A group of strong enough people"
        },
        "created": {
            "$date": {
                "$numberLong": "1623268445596"
            }
        }
    },
    {
        "_id": {
            "$oid": "60c1e0cbb889bff62d97b72e"
        },
        "name": "Thomas Anderson",
        "twitter": null,
        "team": {
            "name": "Resistance",
            "description": ""
        },
        "created": {
            "$date": {
                "$numberLong": "1623269028565"
            }
        }
    }
]
```

---
![bg](#aaa)
![bg 50% 70%](https://raw.githubusercontent.com/samuelcolvin/pydantic/master/docs/logo-white.svg)
![bg 50% 70%](images/motor.png)

---

```python
class User(BaseModel):
    id: int
    name: str = 'John Doe'
    ts: Optional[datetime]
    friends: List[int] = []
```
```python
user_dict = {
    "id": 10,
    "name": "Anna"
}
user = User.parse_obj(user_dict)
print(user)
```
```python
User(id=10, name='Anna', ts=None, friends=[])
```

---
![bg left:40% 120%](images/figure.jpg)

```python
from typing import Optional
from beanie import Document, Indexed


class Person(Document):
    name: Indexed(str)
    twitter: Optional[str]
    rating: int = 0

    class Collection:
        name = "persons"
```

---

![bg right 100%](images/nested.gif)

```python
class Team(BaseModel):
    name: str
    description: str


class Person(Document):
    name: Indexed(str)
    twitter: Optional[str]
    rating: int = 0

    team: Team
    created: datetime = Field(
        default_factory=datetime.utcnow
    )

    class Collection:
        name = "persons"
```
---
![bg right:40%](images/ironman.gif)

```python
team = Team(
    name="Avengers", 
    description="A group of strong enough people"
)
tony = Person(
    name="Tony Stark", 
    twitter="@Iron_Man", 
    team=team
)
await tony.create()
print(tony)

```

Output:

```python
id=ObjectId('60c11c5dbce98d02f4eee8d7') 
name='Tony Stark' 
twitter='@Iron_Man' 
rating=0
team=Team(
    name='Avengers', 
    description='A group of strong enough people'
) 
created=datetime.datetime(
    2021, 6, 9, 19, 54, 5, 596329)
```

---
![bg left 100%](images/id.jpg)

```python
from pydantic import Field
from uuid import UUID, uuid4


class Sample(Document):
    id: UUID = Field(default_factory=uuid4())
    name: str
```
```python
sample = Sample(name="Test")
await sample.create()
print(sample)
```
```python
id=UUID('7b7fea05-7a47-42ba-a1f9-3bf432034737')
name="Test"
```

---

![bg](images/find.jpeg)

---

# Get

![bg left:40% 130%](images/pick_up.gif)

```python
person = await Person.get(ObjectId("60c11c5dbce98d02f4eee8d7"))
```

Result:

```python
id=ObjectId('60c11c5dbce98d02f4eee8d7') 
name='Tony Stark' 
twitter='@Iron_Man' 
rating=0
team=Team(
    name='Avengers', 
    description='A group of strong enough people'
) 
created=datetime.datetime(
    2021, 6, 9, 19, 54, 5, 596329)
```

---

# Find

![bg left:40% 130%](images/agents.jpeg)
async generator

```python
async for person in Person.find(
        Person.team.name == "Agents"
):
    print(person)
```

list

```python
agents = await Person.find(
    Person.team.name == "Agents"
).to_list()
```

---

# Find (The) One

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
people = await Person.find(
    Person.created > datetime.datetime(2021, 6, 9)
).to_list()
```
```python
[Person(..., created=datetime.datetime(2021, 6, 9, 20)),
Person(..., created=datetime.datetime(2021, 6, 9, 21)),
Person(..., created=datetime.datetime(2021, 6, 9, 23))
Person(..., created=datetime.datetime(2021, 6, 10, 10)),]
```

---
# Wrappers

![bg left:40%](images/wraps3.jpeg)

```python
In, Near, Exists, Nor...
```

```python
from beanie.operators import In

red_and_blue = await Person.find(
    In(Person.team.name, ["Blue", "Red"]))
).to_list()
```

---

# MongoDB syntax

![bg right:40% ](images/mongo_logo_1.jpeg)

```python
people = await Person.find(
    {"name": "Joe"}
).to_list()
```

---

![bg left:40% 100%](images/sorting_machine.gif)

```python
people = await Person.find(
    Person.team.name = "Red"
).sort(
    -Person.created
).to_list()
```



```python
people = await Person.find(
    Person.team.name = "Red"
).sort(
    "-created"
).to_list()
```



```python
people = await Product.all().sort(
    [
        (Person.created, pymongo.DESCENDING),
        (Person.name, pymongo.ASCENDING),
    ]
).to_list()
```

---

![bg right:30% 70%](images/pagination.png)

```python
people = await Person.find(
    Person.team.name == "Blue"
).sort(
    Person.created
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
    created: int


people = await Person.find(
    Person.team.name == "Red"
).project(PersonShortView).to_list()
```

---

# Projections

![bg left:40%](images/projection.jpeg)

```python
class PersonShortView(BaseModel):
    name: str
    created: int
    team: str

    class Settings:
        projection = {"name": 1, 
                      "created": 1, 
                      "team": "$team.name"}


people = await Person.find(
    Person.team.name == "Red"
).project(PersonShortView).to_list()
```

---

![bg 70%](images/update_splash.jpg)

---

![bg left:40% 130%](images/pick_up.gif)

```python
person = await Person.get(
    ObjectId("60c11c5dbce98d02f4eee8d7")
)
person.name = "Robert"
await person.replace()
```

---

![bg left:40% 130%](images/pick_up.gif)

```python
person = await Person.get(
    ObjectId("60c11c5dbce98d02f4eee8d7")
)
await person.update(
    Set({Person.twitter: "@RobertDowneyJr"})
)
```

---
# Many

![bg right:40% 110%](images/squad.jpeg)

```python
await Person.find(
    Person.team.name == "Villains"
).update(Inc({Person.rating: -10}))
```

---

# One

![bg left:40% 100%](images/batman.jpeg)

```python
await Person.find_one(
    Person.name == "Bruce Wayne"
).update(Set({Person.twitter: "@TheBatman"}))

```

---

# Preset methods

![bg right:40%](images/points.jpg)

#### Methods

```python

set(), inc(), current_date()

```

#### Example

```python
await Person.find(
    Person.team.name == "Gryffindor"
).inc({Person.rating: 100})

```

---

# Wrappers

![bg left:40%](images/wraps2.jpg)

```python
Set, Min, Max, Mul and etc..

```

```python
from beanie.operators import Inc

await Person.find(
    Person.team.name == "Blue"
).update(
    Inc({Person.rating: 15})
)
```

---

# Native MongoDB Syntax

![bg right:40%](images/mongo_logo_1.jpeg)

```python
await Person.find(
    Person.team.name == "Blue"
).update(
    {"$inc": {Person.rating: 15}
)
```

---

![bg 80%](images/aggregations.png)

---
![bg right:40%](images/pizza.jpeg)

```python
class Pizza(Document):
    name: str
    base: float
    cheese: float
    diameter: int
    decade: str
```
---

![bg left:40%](images/graps2.jpg)

```python
class PizzaMetrics(BaseModel):
    decade: str = Field(None, alias="_id")
    base: float
    cheese: float


result = await Pizza.aggregate(
    [
        {
            "$group": {
                "_id": "$decade",
                "base": {"$avg": "$base"},
                "cheese": {"$avg": "$cheese"},
            }
        }
    ],
    projection_model=PizzaMetrics
).to_list()
```

```python
[PizzaMetrics(decade='70s', base=10, cheese=2),
 PizzaMetrics(decade='80s', base=20, cheese=4),
 PizzaMetrics(decade='90s', base=25, cheese=20),
 PizzaMetrics(decade='00s', base=10, cheese=20),
 PizzaMetrics(decade='10s', base=2, cheese=2)]
```

---

# Filter

![bg left:40% 112%](images/margarita.jpeg)

```python
result = await Pizza.find(
    Pizza.name == "Margherita"
).aggregate(
    [
        {
            "$group": {
                "_id": "$decade",
                "base": {"$avg": "$base"},
                "cheese": {"$avg": "$cheese"},
            }
        }
    ],
    projection_model=PizzaMetrics
).to_list()
```

---

# Without projection

```python
result = await Pizza.aggregate(
    [
        {
            "$group": {
                "_id": "$decade",
                "base": {"$avg": "$base"},
                "cheese": {"$avg": "$cheese"},
            }
        }
    ]
).to_list()
```

```python
[{"_id":"70s", "base":10, "cheese":2},
 {"_id":"80s", "base":20, "cheese":4},
 {"_id":"90s", "base":25, "cheese":20},
 {"_id":"00s", "base":10, "cheese":20},
 {"_id":"10s", "base":2, "cheese":2}]
```

---

#### Methods

```python
sum(), avg(), min(), max()
```

#### Example

![bg left:40%](images/pizza2.jpg)

```python

max_margherita_diameter = await Pizza.find(
    Pizza.name == "Margherita"
).max(Pizza.diameter)

```

---

![bg 100%](images/fastapi.png)

---

![bg left:40% 100%](images/create.jpeg)

```python
@router.post(
    "/persons/", 
    response_model=Person)
async def create(person: Person):
    await person.create()
    return person
```
```json
{
    "name": "Thor",
    "team": {
        "name":"Avengers",
        "description":"A group of strong enough people"
    }
}
```
```json
{
    "id": "60c216c181efcce3bc01f0a4", 
    "name": "Thor", 
    "twitter": null, 
    "rating": 0, 
    "team": {
        "name": "Avengers", 
        "description": "A group of strong enough people"
    }, 
    "created": "2021-06-10T13:42:25.645244"}
```

---

![bg right:40% 100%](images/read.jpeg)

```python
@router.get(
    "/persons/", 
    response_model=List[Person])
async def get_list():
    persons = await Person.all().to_list()
    return persons
```
```python
@router.get(
    "/persons/{person_id}", 
    response_model=Person)
async def get():
    person = await Person.get(person_id)
    return person
```

---

![bg left:40% 100%](images/update2.jpeg)

```python
class Rating(BaseModel):
    value: float

@router.put(
    "/persons/{person_id}/update_rating", 
    response_model=Person)
async def update_rating(
    person_id: ObjectId, 
    rating: Rating
):
    person = await Person.get(person_id)
    await person.inc({Person.rating: rating.value})
    return person
```

---

![bg right:40% 100%](images/delete.jpeg)

```python
@router.delete("/persons/{person_id}")
async def delete(person_id: ObjectId):
    person = await Person.find_one(
        Person.id == person_id
    ).delete()
    return {"status": "OK"}
```
---

# Other features

- Migrations
- Indexes
- Sessions and transactions

Documentation: <https://roman-right.github.io/beanie/>

---

![bg](images/future.jpg)

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

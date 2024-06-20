**ORM Tutorial in Python**

Welcome to the Object-Relational Mapping (ORM) Tutorial in Python! In this tutorial, we'll explore how to use an ORM library to interact with databases using Python. ORM allows you to work with databases using Python objects, making database operations more intuitive and less error-prone. Let's dive in!

**1. Introduction to ORM:**
ORM (Object-Relational Mapping) is a technique that lets you query and manipulate data from a database using an object-oriented paradigm. It maps database tables to Python classes, rows to objects, and columns to attributes.

**2. Choosing an ORM Library:**
There are several ORM libraries available for Python, including SQLAlchemy, Django ORM, and Peewee. For this tutorial, we'll focus on SQLAlchemy, which is a powerful and widely-used ORM library.

**3. Installing SQLAlchemy:**
You can install SQLAlchemy using pip:

```
pip install sqlalchemy
```

**4. Creating a Database Connection:**
Before working with the database, you need to establish a connection. SQLAlchemy provides a `create_engine()` function to create a database engine.

```python
from sqlalchemy import create_engine

# SQLite database example
engine = create_engine('sqlite:///example.db')
```

**5. Defining Database Models:**
Next, you define Python classes to represent database tables. Each class corresponds to a table, and each attribute corresponds to a column.

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)
```

**6. Creating Tables:**
After defining models, you can create tables in the database using the `create_all()` method.

```python
Base.metadata.create_all(engine)
```

**7. Performing CRUD Operations:**
Now you can perform CRUD (Create, Read, Update, Delete) operations on the database.

**a. Create:**
```python
from sqlalchemy.orm import sessionmaker

# Create session
Session = sessionmaker(bind=engine)
session = Session()

# Create a new user
user = User(name='Alice', age=30)
session.add(user)
session.commit()
```

**b. Read:**
```python
# Retrieve all users
users = session.query(User).all()
for user in users:
    print(user.name, user.age)
```

**c. Update:**
```python
# Update user
user = session.query(User).filter_by(name='Alice').first()
user.age = 31
session.commit()
```

**d. Delete:**
```python
# Delete user
user = session.query(User).filter_by(name='Alice').first()
session.delete(user)
session.commit()
```

**8. Querying with Filters:**
You can filter query results using various criteria.

```python
# Filter users by age
users = session.query(User).filter(User.age > 25).all()
```

**9. Relationships:**
ORM allows you to define relationships between tables.

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Address(Base):
    __tablename__ = 'addresses'

    id = Column(Integer, primary_key=True)
    email = Column(String)
    user_id = Column(Integer, ForeignKey('users.id'))

    user = relationship('User', back_populates='addresses')
```

**10. Joins:**
You can perform joins to retrieve related data.

```python
# Perform a join
users_with_addresses = session.query(User).join(Address).all()
for user in users_with_addresses:
    print(user.name, user.addresses)
```

Congratulations! You've completed the ORM Tutorial in Python using SQLAlchemy. ORM simplifies database interactions by allowing you to work with Python objects instead of raw SQL queries. Practice implementing these concepts in your projects to become proficient in using ORM with Python. If you have any questions or need further clarification, feel free to ask. Happy coding!
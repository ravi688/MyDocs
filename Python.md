# Complete Python Learning Guide: Start to Finish

## Table of Contents
1. [Getting Started](#getting-started)
2. [Basic Syntax and Variables](#basic-syntax-and-variables)
3. [Data Types](#data-types)
4. [Operators](#operators)
5. [Control Structures](#control-structures)
6. [Functions](#functions)
7. [Data Structures](#data-structures)
8. [Object-Oriented Programming](#object-oriented-programming)
9. [File Handling](#file-handling)
10. [Error Handling](#error-handling)
11. [Modules and Packages](#modules-and-packages)
12. [Advanced Topics](#advanced-topics)
13. [Standard Library](#standard-library)
14. [Best Practices](#best-practices)
15. [Projects and Practice](#projects-and-practice)

---

## Getting Started

### What is Python?
Python is a high-level, interpreted programming language known for its simplicity and readability. It's used for web development, data analysis, artificial intelligence, automation, and more.

### Installing Python
1. Visit python.org and download the latest version
2. During installation, check "Add Python to PATH"
3. Verify installation: `python --version` in command line

### Your First Program
```python
print("Hello, World!")
```

---

## Basic Syntax and Variables

### Python Syntax Rules
- Python uses indentation (4 spaces) instead of braces
- Statements end with a newline (no semicolon needed)
- Case-sensitive language

### Variables and Assignment
```python
# Variable assignment
name = "Alice"
age = 25
height = 5.6
is_student = True

# Multiple assignment
x, y, z = 1, 2, 3

# Constants (by convention, use UPPER_CASE)
PI = 3.14159
MAX_SIZE = 100
```

### Naming Conventions
- Use snake_case for variables and functions
- Use PascalCase for classes
- Use UPPER_CASE for constants
- Avoid reserved keywords

---

## Data Types

### Numbers
```python
# Integers
age = 25
temperature = -10

# Floats
price = 19.99
pi = 3.14159

# Complex numbers
complex_num = 3 + 4j

# Type conversion
int_from_float = int(3.7)  # 3
float_from_int = float(5)  # 5.0
```

### Strings
```python
# String creation
single_quoted = 'Hello'
double_quoted = "World"
multiline = """This is a
multiline string"""

# String methods
text = "python programming"
print(text.upper())        # PYTHON PROGRAMMING
print(text.capitalize())   # Python programming
print(text.replace("python", "Python"))  # Python programming
print(text.split())        # ['python', 'programming']

# String formatting
name = "Alice"
age = 25
print(f"My name is {name} and I'm {age} years old")  # f-strings (recommended)
print("My name is {} and I'm {} years old".format(name, age))  # .format()
print("My name is %s and I'm %d years old" % (name, age))  # old style
```

### Booleans
```python
is_true = True
is_false = False

# Boolean operations
print(True and False)  # False
print(True or False)   # True
print(not True)        # False
```

---

## Operators

### Arithmetic Operators
```python
a, b = 10, 3

print(a + b)   # Addition: 13
print(a - b)   # Subtraction: 7
print(a * b)   # Multiplication: 30
print(a / b)   # Division: 3.333...
print(a // b)  # Floor division: 3
print(a % b)   # Modulus: 1
print(a ** b)  # Power: 1000
```

### Comparison Operators
```python
print(5 == 5)   # Equal: True
print(5 != 3)   # Not equal: True
print(5 > 3)    # Greater than: True
print(5 < 3)    # Less than: False
print(5 >= 5)   # Greater than or equal: True
print(5 <= 3)   # Less than or equal: False
```

### Logical Operators
```python
print(True and False)  # False
print(True or False)   # True
print(not True)        # False
```

### Assignment Operators
```python
x = 10
x += 5   # x = x + 5, x is now 15
x -= 3   # x = x - 3, x is now 12
x *= 2   # x = x * 2, x is now 24
x /= 4   # x = x / 4, x is now 6.0
```

---

## Control Structures

### If Statements
```python
age = 18

if age >= 18:
    print("You're an adult")
elif age >= 13:
    print("You're a teenager")
else:
    print("You're a child")

# Ternary operator
status = "adult" if age >= 18 else "minor"
```

### Loops

#### For Loops
```python
# Iterating over a range
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

# Iterating over a list
fruits = ["apple", "banana", "orange"]
for fruit in fruits:
    print(fruit)

# Enumerate for index and value
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# Range with start, stop, step
for i in range(1, 10, 2):
    print(i)  # 1, 3, 5, 7, 9
```

#### While Loops
```python
count = 0
while count < 5:
    print(count)
    count += 1

# While with else
i = 0
while i < 3:
    print(i)
    i += 1
else:
    print("Loop completed normally")
```

#### Loop Control
```python
# Break and continue
for i in range(10):
    if i == 3:
        continue  # Skip this iteration
    if i == 7:
        break     # Exit the loop
    print(i)
```

---

## Functions

### Basic Functions
```python
def greet(name):
    """This function greets someone"""
    return f"Hello, {name}!"

# Function call
message = greet("Alice")
print(message)
```

### Parameters and Arguments
```python
# Default parameters
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# Keyword arguments
print(greet("Bob"))                    # Hello, Bob!
print(greet("Charlie", "Hi"))          # Hi, Charlie!
print(greet(greeting="Hey", name="Dave"))  # Hey, Dave!

# Variable arguments
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3, 4))  # 10

# Keyword variable arguments
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="New York")
```

### Lambda Functions
```python
# Lambda function
square = lambda x: x ** 2
print(square(5))  # 25

# Using lambda with built-in functions
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]
```

### Scope
```python
# Global and local scope
global_var = "I'm global"

def my_function():
    local_var = "I'm local"
    global global_var
    global_var = "Modified global"
    print(local_var)
    print(global_var)

my_function()
```

---

## Data Structures

### Lists
```python
# Creating lists
fruits = ["apple", "banana", "orange"]
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]

# List methods
fruits.append("grape")      # Add to end
fruits.insert(1, "mango")   # Insert at index
fruits.remove("banana")     # Remove by value
popped = fruits.pop()       # Remove and return last item
fruits.extend(["kiwi", "pear"])  # Add multiple items

# List operations
print(len(fruits))          # Length
print("apple" in fruits)    # Membership test
print(fruits[0])            # Indexing
print(fruits[-1])           # Negative indexing
print(fruits[1:3])          # Slicing

# List comprehensions
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
```

### Tuples
```python
# Creating tuples
coordinates = (3, 4)
colors = ("red", "green", "blue")
single_item = (42,)  # Note the comma

# Tuple unpacking
x, y = coordinates
print(f"x: {x}, y: {y}")

# Tuples are immutable
# coordinates[0] = 5  # This would cause an error
```

### Dictionaries
```python
# Creating dictionaries
person = {
    "name": "Alice",
    "age": 25,
    "city": "New York"
}

# Dictionary operations
print(person["name"])           # Access value
person["age"] = 26             # Modify value
person["job"] = "Engineer"     # Add new key-value pair

# Dictionary methods
print(person.keys())           # Get all keys
print(person.values())         # Get all values
print(person.items())          # Get key-value pairs
print(person.get("height", "Not found"))  # Safe access

# Dictionary comprehensions
squares = {x: x**2 for x in range(5)}
```

### Sets
```python
# Creating sets
colors = {"red", "green", "blue"}
numbers = set([1, 2, 3, 3, 4])  # Duplicates removed

# Set operations
colors.add("yellow")
colors.remove("red")
colors.discard("purple")  # Won't error if not present

# Set mathematics
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}
print(set1.union(set2))        # {1, 2, 3, 4, 5, 6}
print(set1.intersection(set2)) # {3, 4}
print(set1.difference(set2))   # {1, 2}
```

---

## Object-Oriented Programming

### Classes and Objects
```python
class Person:
    # Class variable
    species = "Homo sapiens"
    
    def __init__(self, name, age):
        # Instance variables
        self.name = name
        self.age = age
    
    def introduce(self):
        return f"Hi, I'm {self.name} and I'm {self.age} years old"
    
    def have_birthday(self):
        self.age += 1
        return f"Happy birthday! Now I'm {self.age}"

# Creating objects
person1 = Person("Alice", 25)
person2 = Person("Bob", 30)

print(person1.introduce())
print(person1.have_birthday())
```

### Inheritance
```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

# Using inheritance
dog = Dog("Buddy")
cat = Cat("Whiskers")
print(dog.speak())  # Buddy says Woof!
print(cat.speak())  # Whiskers says Meow!
```

### Special Methods
```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages
    
    def __str__(self):
        return f"{self.title} by {self.author}"
    
    def __len__(self):
        return self.pages
    
    def __eq__(self, other):
        return self.title == other.title and self.author == other.author

book1 = Book("1984", "George Orwell", 328)
print(str(book1))  # 1984 by George Orwell
print(len(book1))  # 328
```

### Properties and Encapsulation
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius  # Protected attribute
    
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
    
    @property
    def area(self):
        return 3.14159 * self._radius ** 2

circle = Circle(5)
print(circle.area)  # 78.53975
circle.radius = 10
print(circle.area)  # 314.159
```

---

## File Handling

### Reading Files
```python
# Reading entire file
with open("example.txt", "r") as file:
    content = file.read()
    print(content)

# Reading line by line
with open("example.txt", "r") as file:
    for line in file:
        print(line.strip())

# Reading all lines into a list
with open("example.txt", "r") as file:
    lines = file.readlines()
```

### Writing Files
```python
# Writing to a file
with open("output.txt", "w") as file:
    file.write("Hello, World!\n")
    file.write("This is a new line.")

# Appending to a file
with open("output.txt", "a") as file:
    file.write("\nThis is appended text.")

# Writing multiple lines
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]
with open("output.txt", "w") as file:
    file.writelines(lines)
```

### Working with CSV
```python
import csv

# Reading CSV
with open("data.csv", "r") as file:
    reader = csv.reader(file)
    for row in reader:
        print(row)

# Writing CSV
data = [["Name", "Age", "City"],
        ["Alice", 25, "New York"],
        ["Bob", 30, "London"]]

with open("output.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerows(data)
```

---

## Error Handling

### Try-Except Blocks
```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
except Exception as e:
    print(f"An error occurred: {e}")
else:
    print("No error occurred")
finally:
    print("This always executes")
```

### Raising Exceptions
```python
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems unrealistic")
    return True

try:
    validate_age(-5)
except ValueError as e:
    print(f"Validation error: {e}")
```

### Custom Exceptions
```python
class CustomError(Exception):
    def __init__(self, message):
        self.message = message
        super().__init__(self.message)

def risky_function():
    raise CustomError("Something went wrong!")

try:
    risky_function()
except CustomError as e:
    print(f"Custom error: {e.message}")
```

---

## Modules and Packages

### Importing Modules
```python
import math
from datetime import datetime
import os as operating_system
from collections import Counter, defaultdict

# Using imported modules
print(math.pi)
print(datetime.now())
print(operating_system.getcwd())
```

### Creating Your Own Module
```python
# File: my_module.py
def greet(name):
    return f"Hello, {name}!"

def calculate_area(radius):
    return 3.14159 * radius ** 2

PI = 3.14159

# File: main.py
import my_module

print(my_module.greet("Alice"))
print(my_module.calculate_area(5))
print(my_module.PI)
```

### Packages
```python
# Package structure:
# mypackage/
#   __init__.py
#   module1.py
#   module2.py
#   subpackage/
#     __init__.py
#     module3.py

from mypackage import module1
from mypackage.subpackage import module3
```

---

## Advanced Topics

### Decorators
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@my_decorator
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
```

### Generators
```python
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Using the generator
for num in fibonacci(10):
    print(num)

# Generator expressions
squares = (x**2 for x in range(10))
print(list(squares))
```

### Context Managers
```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()

# Using the context manager
with FileManager("test.txt", "w") as f:
    f.write("Hello, World!")
```

### Threading and Multiprocessing
```python
import threading
import time

def worker(name):
    for i in range(5):
        print(f"Worker {name}: {i}")
        time.sleep(1)

# Creating threads
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"Thread-{i}",))
    threads.append(t)
    t.start()

# Wait for all threads to complete
for t in threads:
    t.join()
```

---

## Standard Library

### Common Modules
```python
# datetime
from datetime import datetime, timedelta
now = datetime.now()
future = now + timedelta(days=7)

# random
import random
print(random.randint(1, 100))
print(random.choice(["apple", "banana", "orange"]))

# json
import json
data = {"name": "Alice", "age": 25}
json_string = json.dumps(data)
parsed_data = json.loads(json_string)

# urllib for web requests
import urllib.request
response = urllib.request.urlopen("https://api.github.com")
data = response.read()

# collections
from collections import Counter, defaultdict, namedtuple
count = Counter("hello world")
dd = defaultdict(list)
Point = namedtuple("Point", ["x", "y"])
```

---

## Best Practices

### Code Style (PEP 8)
```python
# Good
def calculate_area(radius):
    """Calculate the area of a circle."""
    return 3.14159 * radius ** 2

# Variable naming
user_name = "alice"
MAX_CONNECTIONS = 100

# Function naming
def get_user_data():
    pass

# Class naming
class UserManager:
    pass
```

### Documentation
```python
def complex_function(param1, param2, param3=None):
    """
    Brief description of the function.
    
    Args:
        param1 (str): Description of param1
        param2 (int): Description of param2
        param3 (list, optional): Description of param3. Defaults to None.
    
    Returns:
        dict: Description of return value
    
    Raises:
        ValueError: If param1 is empty
        TypeError: If param2 is not an integer
    """
    pass
```

### Testing
```python
import unittest

class TestMathOperations(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(2 + 2, 4)
    
    def test_division(self):
        self.assertEqual(10 / 2, 5)
        with self.assertRaises(ZeroDivisionError):
            10 / 0

if __name__ == "__main__":
    unittest.main()
```

---

## Projects and Practice

### Beginner Projects
1. **Calculator**: Build a simple calculator with basic operations
2. **To-Do List**: Create a command-line to-do list manager
3. **Guess the Number**: Number guessing game with hints
4. **Password Generator**: Generate secure passwords
5. **Word Count**: Count words, characters, and lines in text files

### Intermediate Projects
1. **Web Scraper**: Extract data from websites
2. **Weather App**: Get weather data from an API
3. **File Organizer**: Organize files by type/date
4. **Contact Manager**: Store and manage contact information
5. **Simple Game**: Tic-tac-toe, hangman, or rock-paper-scissors

### Advanced Projects
1. **Web Application**: Build with Flask or Django
2. **Data Analysis**: Analyze datasets with pandas
3. **Machine Learning**: Simple ML models with scikit-learn
4. **API Development**: Create REST APIs
5. **Desktop GUI**: Build with tkinter or PyQt

### Learning Resources
- Python.org official documentation
- "Automate the Boring Stuff with Python" by Al Sweigart
- LeetCode and HackerRank for coding challenges
- Real Python website for tutorials
- Python Package Index (PyPI) for third-party libraries

### Next Steps
1. Choose a specialization (web development, data science, AI, etc.)
2. Learn relevant frameworks and libraries
3. Build portfolio projects
4. Contribute to open source projects
5. Join Python communities and forums

---

Remember: Programming is learned by doing. Start with small projects and gradually work your way up to more complex applications. Don't try to memorize everything at once - focus on understanding concepts and practicing regularly!

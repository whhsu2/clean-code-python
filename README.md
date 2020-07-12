# clean-code-python

## Table of Contents
  1. [Introduction](#introduction)
  2. [Variables](#variables)
  3. [Functions](#functions)
  4. [Comments](#comments)
  4. [Objects and Data Structures](#objects-and-data-structures)
  5. [Error Handling](#error-handling)
  6. [Boundary](#boundary)
  7. [Classes](#classes)
     1. [S: Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
     2. [O: Open/Closed Principle (OCP)](#openclosed-principle-ocp)
     3. [L: Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
     4. [I: Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
     5. [D: Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
  8. [System](#system)
  9. [Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)

## **Introduction**

Software engineering principles, from Robert C. Martin's book
[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adapted for Python. This is not a style guide. It's a guide to producing
readable, reusable, and refactorable software in Python.

Not every principle herein has to be strictly followed, and even fewer will be universally 
agreed upon. These are guidelines and nothing more, but they are ones codified over many 
years of collective experience by the authors of *Clean Code*.

The goal is to have well-designed software, here is Kent Beck's four rules of *Simple Design*:  
1. Runs all the tests
2. Contains no duplication
3. Expresses the intent of hte progammer
4. Minimizes the number of classes and methods  

The rules are given in order of importance.

Inspired from [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)

Targets Python3.7+

## **Variables**
### Use meaningful and pronounceable variable names

**Bad:**
```python
ymdstr = datetime.date.today().strftime("%y-%m-%d")
```

**Good**:
```python
current_date: str = datetime.date.today().strftime("%y-%m-%d")
```
**[⬆ back to top](#table-of-contents)**

### Use the same vocabulary for the same type of variable

**Bad:**
Here we use three different names for the same underlying entity:
```python
get_user_info()
get_client_data()
get_customer_record()
```

**Good**:
If the entity is the same, you should be consistent in referring to it in your functions:
```python
get_user_info()
get_user_data()
get_user_record()
```

**Even better**
Python is (also) an object oriented programming language. If it makes sense, package the functions together with the concrete implementation
of the entity in your code, as instance attributes, property methods, or methods:

```python
class User:
    info : str

    @property
    def data(self) -> dict:
        # ...

    def get_record(self) -> Union[Record, None]:
        # ...
```

**[⬆ back to top](#table-of-contents)**

### Use searchable names
We will read more code than we will ever write. It's important that the code we do write is 
readable and searchable. By *not* naming variables that end up being meaningful for 
understanding our program, we hurt our readers.
Make your names searchable.

**Bad:**
```python
# What the heck is 86400 for?
time.sleep(86400);
```

**Good**:
```python
# Declare them in the global namespace for the module.
SECONDS_IN_A_DAY = 60 * 60 * 24

time.sleep(SECONDS_IN_A_DAY)
```
**[⬆ back to top](#table-of-contents)**

### Use explanatory variables
**Bad:**
```python
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = r'^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$'
matches = re.match(city_zip_code_regex, address)

save_city_zip_code(matches[1], matches[2])
```

**Not bad**:

It's better, but we are still heavily dependent on regex.

```python
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = r'^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$'
matches = re.match(city_zip_code_regex, address)

city, zip_code = matches.groups()
save_city_zip_code(city, zip_code)
```

**Good**:

Decrease dependence on regex by naming subpatterns.
```python
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = r'^[^,\\]+[,\\\s]+(?P<city>.+?)\s*(?P<zip_code>\d{5})?$'
matches = re.match(city_zip_code_regex, address)

save_city_zip_code(matches['city'], matches['zip_code'])
```
**[⬆ back to top](#table-of-contents)**

### Avoid Mental Mapping
Don’t force the reader of your code to translate what the variable means.
Explicit is better than implicit.

**Bad:**
```python
seq = ('Austin', 'New York', 'San Francisco')

for item in seq:
    do_stuff()
    do_some_other_stuff()
    # ...
    # Wait, what's `item` for again?
    dispatch(item)
```

**Good**:
```python
locations = ('Austin', 'New York', 'San Francisco')

for location in locations:
    do_stuff()
    do_some_other_stuff()
    # ...
    dispatch(location)
```
**[⬆ back to top](#table-of-contents)**


### Don't add unneeded context

If your class/object name tells you something, don't repeat that in your
variable name.

**Bad:**

```python
class Car:
    car_make: str
    car_model: str
    car_color: str
```

**Good**:

```python
class Car:
    make: str
    model: str
    color: str
```

**[⬆ back to top](#table-of-contents)**

### Use default arguments instead of short circuiting or conditionals

**Tricky**

Why write:

```python
def create_micro_brewery(name):
    name = "Hipster Brew Co." if name is None else name
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

... when you can specify a default argument instead? This also makes it clear that
you are expecting a string as the argument.

**Good**:

```python
def create_micro_brewery(name: str = "Hipster Brew Co."):
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

**[⬆ back to top](#table-of-contents)**
## **Functions**
### Function arguments (2 or fewer ideally)
Limiting the amount of function parameters is incredibly important because it makes 
testing your function easier. Having more than three leads to a combinatorial explosion 
where you have to test tons of different cases with each separate argument.

Zero arguments is the ideal case. One or two arguments is ok, and three should be avoided. 
Anything more than that should be consolidated. Usually, if you have more than two 
arguments then your function is trying to do too much. In cases where it's not, most 
of the time a higher-level object will suffice as an argument.

**Bad:**
```python
def create_menu(title, body, button_text, cancellable):
    # ...
```

**Good**:
```python
class Menu:
    def __init__(self, config: dict):
        title = config["title"]
        body = config["body"]
        # ...

menu = Menu(
    {
        "title": "My Menu",
        "body": "Something about my menu",
        "button_text": "OK",
        "cancellable": False
    }
)
```

**Also good**
```python
class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False


def create_menu(config: MenuConfig):
    title = config.title
    body = config.body
    # ...


config = MenuConfig()
config.title = "My delicious menu"
config.body = "A description of the various items on the menu"
config.button_text = "Order now!"
# The instance attribute overrides the default class attribute.
config.cancellable = True

create_menu(config)
```

**Fancy**
```python
from typing import NamedTuple


class MenuConfig(NamedTuple):
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False


def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = config
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

**Even fancier**
```python
from dataclasses import astuple, dataclass


@dataclass
class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False

def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = astuple(config)
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

**[⬆ back to top](#table-of-contents)**

### Functions should do one thing
This is by far the most important rule in software engineering. When functions do more 
than one thing, they are harder to compose, test, and reason about. When you can isolate 
a function to just one action, they can be refactored easily and your code will read much cleaner. If you take nothing else away from this guide other than this, you'll be ahead of many developers.

**Bad:**
```python

def email_clients(clients: List[Client]):
    """Filter active clients and send them an email.
    """
    for client in clients:
        if client.active:
            email(client)
```

**Good**:
```python
def get_active_clients(clients: List[Client]) -> List[Client]:
    """Filter active clients.
    """
    return [client for client in clients if client.active]


def email_clients(clients: List[Client, ...]) -> None:
    """Send an email to a given list of clients.
    """
    for client in clients:
        email(client)
```

Do you see an opportunity for using generators now?

**Even better**
```python
def active_clients(clients: List[Client]) -> Generator[Client]:
    """Only active clients.
    """
    return (client for client in clients if client.active)


def email_client(clients: Iterator[Client]) -> None:
    """Send an email to a given list of clients.
    """
    for client in clients:
        email(client)
```


**[⬆ back to top](#table-of-contents)**

### Function names should say what they do

**Bad:**

```python
class Email:
    def handle(self) -> None:
        # Do something...

message = Email()
# What is this supposed to do again?
message.handle()
```

**Good:**

```python
class Email:
    def send(self) -> None:
        """Send this message.
        """

message = Email()
message.send()
```

**[⬆ back to top](#table-of-contents)**

Another example is functions should either do something or answer something, but not both. Either your function should change the state of an object, or it should return some information about that object.

### Functions should only be one level of abstraction

When you have more than one level of abstraction, your function is usually doing too 
much. Splitting up functions leads to reusability and easier testing.

**Bad:**

```python
def parse_better_js_alternative(code: str) -> None:
    regexes = [
        # ...
    ]

    statements = regexes.split()
    tokens = []
    for regex in regexes:
        for statement in statements:
            # ...

    ast = []
    for token in tokens:
        # Lex.

    for node in ast:
        # Parse.
```

**Good:**

```python

REGEXES = (
   # ...
)


def parse_better_js_alternative(code: str) -> None:
    tokens = tokenize(code)
    syntax_tree = parse(tokens)

    for node in syntax_tree:
        # Parse.


def tokenize(code: str) -> list:
    statements = code.split()
    tokens = []
    for regex in REGEXES:
        for statement in statements:
           # Append the statement to tokens.

    return tokens


def parse(tokens: list) -> list:
    syntax_tree = []
    for token in tokens:
        # Append the parsed token to the syntax tree.

    return syntax_tree
```

**[⬆ back to top](#table-of-contents)**

### Don't use flags as function parameters

Flags tell your user that this function does more than one thing. Functions 
should do one thing. Split your functions if they are following different code 
paths based on a boolean.

**Bad:**

```python
from pathlib import Path

def create_file(name: str, temp: bool) -> None:
    if temp:
        Path('./temp/' + name).touch()
    else:
        Path(name).touch()
```

**Good:**

```python
from pathlib import Path

def create_file(name: str) -> None:
    Path(name).touch()

def create_temp_file(name: str) -> None:
    Path('./temp/' + name).touch()
```

**[⬆ back to top](#table-of-contents)**

### Avoid side effects

A function produces a side effect if it does anything other than take a value in 
and return another value or values. For example, a side effect could be writing 
to a file, modifying some global variable, or accidentally wiring all your money
to a stranger.

Now, you do need to have side effects in a program on occasion - for example, like
in the previous example, you might need to write to a file. In these cases, you
should centralize and indicate where you are incorporating side effects. Don't have
several functions and classes that write to a particular file - rather, have one
(and only one) service that does it.

The main point is to avoid common pitfalls like sharing state between objects
without any structure, using mutable data types that can be written to by anything,
or using an instance of a class, and not centralizing where your side effects occur.
If you can do this, you will be happier than the vast majority of other programmers.

**Bad:**

```python
# This is a module-level name.
# It's good practice to define these as immutable values, such as a string.
# However...
name = 'Ryan McDermott'

def split_into_first_and_last_name() -> None:
    # The use of the global keyword here is changing the meaning of the
    # the following line. This function is now mutating the module-level
    # state and introducing a side-effect!
    global name
    name = name.split()

split_into_first_and_last_name()

print(name)  # ['Ryan', 'McDermott']

# OK. It worked the first time, but what will happen if we call the
# function again?
```

**Good:**
```python
def split_into_first_and_last_name(name: str) -> list:
    return name.split()

name = 'Ryan McDermott'
new_name = split_into_first_and_last_name(name)

print(name)  # 'Ryan McDermott'
print(new_name)  # ['Ryan', 'McDermott']
```

**Also good**
```python
from dataclasses import dataclass

@dataclass
class Person:
    name: str

    @property
    def name_as_first_and_last(self) -> list:
        return self.name.split() 

# The reason why we create instances of classes is to manage state!
person = Person('Ryan McDermott')
print(person.name)  # 'Ryan McDermott'
print(person.name_as_first_and_last)  # ['Ryan', 'McDermott']
```

**[⬆ back to top](#table-of-contents)**

## **Comments**

Whenever you feel like you need to write a comment, your code is probably not expressive enough or you're probably not naming a variable well clear enough.

## **Objects and Data Structures**

There's a reason that we keep our variables private. We don't want anyone else to depend on them. We want to keep the freedom to change their type or implementation on a whim or an impulse.

Hiding implementation is about abstraction! A class does not simply push its variables out through getter and setters. Rather it exposes abstract interfaces that allow its users to manipulate the essence of the data, without having to know its implementation.

**Bad:**

```python

class Point 
    x: double
    y: double
```

**Good:**

```python

class Point 
    
    def getX(self) -> double:
        return 
    
    def getY(self) -> double:
        return

    def setCartesian(self, double x, double y):
        return
```

The difference between the two is in the second example, there is no way you can tell whether the implementation is in rectangular or polar coordinates.

### **Data/Object Anti-Symmetry**

*Coming soon*

**[⬆ back to top](#table-of-contents)**

## **Error Handling**

I think any discussion about error handling should include mention of the things we do that invite errors.

### Dont Return Null

```python
Employees = getEmployees()
if Employees != Null:
    for e in Employees:
        totalPay += e.getPay()
```
By changing getEmployees to return empty list we can clean up the code


```python
Employees = getEmployees()
    for e in Employees:
        totalPay += e.getPay()
```



*Coming soon*

**[⬆ back to top](#table-of-contents)**

## **Boundary**


Code at boundaries needs clear seperation and test that define expectations. We should avoid letting too much of our code know about the third-party particulars. It's better to depend on something you control than something you don't control, lest it end up controlling you.

### Using Third Pary Code

```python
class Sensors 
    def __init__(self, name):  
        sensors = new HashMap()

    def getById(self, id):
        return sensors.get(id)
```

The interface at the boundary (Map) is hidden. It is able to evolve with very little impact on the rest of the application. The use of generics is no longer a big issue because the casting and type management is handled inside the Sensor class. 




## **Classes**

The first rule of classes is that they should be small. The second rule of classes is that they should be smaller than that. With functions we measure size by counting physical lines. With classes we use a different measure. We count responsibilities. This brings us to first priciple of class implementation:

### **Single Responsibility Principle (SRP)**

The Single Responsibility Principle (SRP) states that a class or module should have one, and only one, reason to change. This principle gives us both a definition of responsibility and a guideline for class sizes. Classes should have one responsibility - one reason to change.

Many developers fear that a large number of small, single-purpose classes make it more difficult to understand the bigger picture. However, a system with many small classes has no more moving parts than a system with a few large classes. So the question is do you want your tools organized into toolboxes with many small drawers each containing well-defined and well-labeled components ? Or do you want a few drawers that you just toss everything into ? 

One of the metrics to view if your class follows the SRP principle is cohesion. Classes should have small number of variables. Each of the methods of a class should manipulate one or more of those variables.

From the below implementation of Stack. This is a very cohesive class. Of the three methods only size() fails to use both the variables.

```python
class Stack
    def __init__(self, name):  
        topOfStack: int
        elements: List[int]

    def size(self):
        return self.topOfStack
    
    def push(self, element):
        topOfStack += 1
        elements.append(element)

    def pop(self):
        if topOfStack == 0:
            return None
        return elements.pop()
```

Maintaining Cohesion Results in Many Small Classes

One of the most important decisions regarding responsiblities is where to put code. For example, where should PI constant go ? Should it be in the Math class ? Perhaps it belongs in the Triogonometry class ? Or maybe in the Circle class ? 

The principle of least surprise comes into play here. Code should be placed where a reader would naturally expect it to be. Ask yourself this question, where will you want this code to be found ? 

*Coming soon*

**[⬆ back to top](#table-of-contents)**


### **Open/Closed Principle (OCP)**

Classes should be open for extension but closed for modification. In an ideal system, we incorporate new features by extending the system, not by making modifications to existing code.

Here's a [Example](https://www.stevebrownlee.com/open-closed-principle-practical-example/) of Open/Closed Principle.

```python
class SQL 
    
```


### **Liskov Substitution Principle (LSP)**
### **Interface Segregation Principle (ISP)**
### **Dependency Inversion Principle (DIP)**



*Coming soon*

**[⬆ back to top](#table-of-contents)**

## **System**

Seperating Constructing a System from Using it




## **Don't repeat yourself (DRY)**

*Coming soon*

**[⬆ back to top](#table-of-contents)**

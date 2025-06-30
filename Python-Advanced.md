# Python Advanced Concepts - Complete Guide

## Table of Contents
1. [Object-Oriented Programming Advanced](#object-oriented-programming-advanced)
2. [Functional Programming](#functional-programming)
3. [Memory Management & Performance](#memory-management--performance)
4. [Concurrency & Parallelism](#concurrency--parallelism)
5. [Advanced Data Structures](#advanced-data-structures)
6. [Metaprogramming](#metaprogramming)
7. [Advanced Language Features](#advanced-language-features)
8. [Performance Optimization](#performance-optimization)
9. [Error Handling & Debugging](#error-handling--debugging)
10. [Testing & Quality Assurance](#testing--quality-assurance)

---

## Object-Oriented Programming Advanced

### Metaclasses

Metaclasses are "classes that create classes" - they define how classes are constructed.

```python
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Singleton(metaclass=SingletonMeta):
    def __init__(self, value):
        self.value = value

# Usage
s1 = Singleton("first")
s2 = Singleton("second")
print(s1 is s2)  # True - same instance
```

### Descriptors

Descriptors control attribute access through `__get__`, `__set__`, and `__delete__` methods.

```python
class ValidatedAttribute:
    def __init__(self, validator):
        self.validator = validator
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)
    
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value for {self.name}: {value}")
        obj.__dict__[self.name] = value

class Person:
    age = ValidatedAttribute(lambda x: isinstance(x, int) and x >= 0)
    name = ValidatedAttribute(lambda x: isinstance(x, str) and len(x) > 0)
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

### Multiple Inheritance & MRO

Method Resolution Order determines which method gets called in multiple inheritance scenarios.

```python
class A:
    def method(self):
        print("A.method")

class B(A):
    def method(self):
        print("B.method")
        super().method()

class C(A):
    def method(self):
        print("C.method")
        super().method()

class D(B, C):
    def method(self):
        print("D.method")
        super().method()

# MRO: D -> B -> C -> A -> object
print(D.__mro__)
d = D()
d.method()  # Prints: D.method, B.method, C.method, A.method
```

### Abstract Base Classes

Define interfaces and enforce implementation contracts.

```python
from abc import ABC, abstractmethod, abstractproperty

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass
    
    @property
    @abstractmethod
    def name(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)
    
    @property
    def name(self):
        return "Rectangle"
```

---

## Functional Programming

### Advanced Decorators

```python
import functools
import time
from typing import Callable, Any

# Parameterized decorator
def retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

# Class decorator
def add_repr(cls):
    def __repr__(self):
        attrs = ', '.join(f"{k}={v!r}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"
    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Decorator factory with state
def counting_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.call_count += 1
        return func(*args, **kwargs)
    wrapper.call_count = 0
    return wrapper
```

### Closures and Factory Functions

```python
def make_multiplier(factor):
    """Factory function that creates multiplier functions"""
    def multiplier(value):
        return value * factor
    return multiplier

# Create specialized functions
double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15

# Closure with mutable state
def make_counter(initial=0):
    count = [initial]  # Mutable container to avoid nonlocal
    
    def counter():
        count[0] += 1
        return count[0]
    
    def reset():
        count[0] = initial
    
    counter.reset = reset
    return counter
```

### Advanced Higher-Order Functions

```python
from functools import reduce, partial
from operator import add, mul
import itertools

# Complex functional operations
def compose(*functions):
    """Compose multiple functions"""
    return reduce(lambda f, g: lambda x: f(g(x)), functions, lambda x: x)

# Pipeline processing
def pipeline(data, *operations):
    """Apply a series of operations to data"""
    return reduce(lambda result, op: op(result), operations, data)

# Example usage
numbers = [1, 2, 3, 4, 5]
result = pipeline(
    numbers,
    partial(filter, lambda x: x % 2 == 0),  # Filter even numbers
    partial(map, lambda x: x ** 2),         # Square them
    list                                     # Convert to list
)
print(result)  # [4, 16]

# Advanced partial application
def log_message(level, message, timestamp=None):
    ts = timestamp or time.time()
    return f"[{level}] {ts}: {message}"

# Create specialized loggers
info_logger = partial(log_message, "INFO")
error_logger = partial(log_message, "ERROR")
```

---

## Memory Management & Performance

### Memory Views and Buffers

```python
import array
import sys

# Memory-efficient operations on large data
def process_large_array():
    # Create a large array
    data = array.array('i', range(1000000))
    
    # Create memory view without copying data
    view = memoryview(data)
    
    # Slice without copying
    slice_view = view[100:200]
    
    # Convert back to array for modification
    slice_array = array.array('i', slice_view)
    
    print(f"Original size: {sys.getsizeof(data)}")
    print(f"View size: {sys.getsizeof(view)}")
    print(f"Slice view size: {sys.getsizeof(slice_view)}")

# Working with bytes efficiently
def bytes_operations():
    data = b"Hello, World!" * 1000
    mv = memoryview(data)
    
    # Efficient slicing
    chunk = mv[0:100]
    
    # Convert to different formats
    as_bytes = chunk.tobytes()
    as_list = chunk.tolist()
```

### Slots for Memory Optimization

```python
class RegularClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class SlottedClass:
    __slots__ = ['x', 'y']
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Memory comparison
import sys

regular = RegularClass(1, 2)
slotted = SlottedClass(1, 2)

print(f"Regular class size: {sys.getsizeof(regular.__dict__)}")
print(f"Slotted class size: {sys.getsizeof(slotted)}")

# Slots with inheritance
class Point3D(SlottedClass):
    __slots__ = ['z']
    
    def __init__(self, x, y, z):
        super().__init__(x, y)
        self.z = z
```

### Weak References

```python
import weakref
import gc

class Node:
    def __init__(self, value):
        self.value = value
        self.children = []
        self.parent = None
    
    def add_child(self, child):
        child.parent = weakref.ref(self)  # Weak reference to parent
        self.children.append(child)

# Callback when object is garbage collected
def cleanup_callback(ref):
    print(f"Object {ref} was garbage collected")

class Resource:
    def __init__(self, name):
        self.name = name

resource = Resource("Important Resource")
weak_ref = weakref.ref(resource, cleanup_callback)

# Object still exists
print(weak_ref().name)  # "Important Resource"

# Delete strong reference
del resource
gc.collect()  # Force garbage collection

# Weak reference is now None
print(weak_ref())  # None
```

---

## Concurrency & Parallelism

### Advanced Threading

```python
import threading
import queue
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

# Producer-Consumer pattern
class ProducerConsumer:
    def __init__(self, max_queue_size=10):
        self.queue = queue.Queue(max_queue_size)
        self.shutdown = threading.Event()
    
    def producer(self, items):
        for item in items:
            if self.shutdown.is_set():
                break
            self.queue.put(item)
            time.sleep(0.1)
        self.queue.put(None)  # Sentinel value
    
    def consumer(self, name):
        while not self.shutdown.is_set():
            try:
                item = self.queue.get(timeout=1)
                if item is None:
                    break
                print(f"Consumer {name} processed: {item}")
                self.queue.task_done()
            except queue.Empty:
                continue

# Thread-safe singleton
class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

# Advanced thread pool usage
def parallel_processing():
    def process_item(item):
        # Simulate processing time
        time.sleep(0.1)
        return item ** 2
    
    items = range(100)
    
    with ThreadPoolExecutor(max_workers=4) as executor:
        # Submit all tasks
        future_to_item = {executor.submit(process_item, item): item for item in items}
        
        # Process completed tasks
        results = []
        for future in as_completed(future_to_item):
            item = future_to_item[future]
            try:
                result = future.result()
                results.append(result)
            except Exception as exc:
                print(f'Item {item} generated an exception: {exc}')
    
    return results
```

### Multiprocessing Patterns

```python
import multiprocessing as mp
from multiprocessing import Pool, Queue, Process, Value, Array
import ctypes

# Shared memory between processes
def worker_with_shared_memory(shared_array, shared_value, lock):
    with lock:
        shared_value.value += 1
        for i in range(len(shared_array)):
            shared_array[i] += 1

def multiprocessing_shared_memory():
    # Shared array and value
    shared_array = Array(ctypes.c_double, [1.0, 2.0, 3.0, 4.0, 5.0])
    shared_value = Value(ctypes.c_int, 0)
    lock = mp.Lock()
    
    processes = []
    for _ in range(4):
        p = Process(target=worker_with_shared_memory, 
                   args=(shared_array, shared_value, lock))
        p.start()
        processes.append(p)
    
    for p in processes:
        p.join()
    
    print(f"Shared value: {shared_value.value}")
    print(f"Shared array: {list(shared_array[:])}")

# Process pool with different patterns
def cpu_intensive_task(n):
    """Simulate CPU-intensive work"""
    result = 0
    for i in range(n * 1000000):
        result += i
    return result

def multiprocessing_patterns():
    # Pattern 1: map
    with Pool() as pool:
        results = pool.map(cpu_intensive_task, [10, 20, 30, 40])
        print("Map results:", results)
    
    # Pattern 2: starmap for multiple arguments
    def multiply(x, y):
        return x * y
    
    with Pool() as pool:
        results = pool.starmap(multiply, [(2, 3), (4, 5), (6, 7)])
        print("Starmap results:", results)
    
    # Pattern 3: async operations
    with Pool() as pool:
        async_results = [pool.apply_async(cpu_intensive_task, (i,)) for i in range(1, 5)]
        results = [ar.get() for ar in async_results]
        print("Async results:", results)
```

### Asyncio Advanced Patterns

```python
import asyncio
import aiohttp
import aiofiles
from asyncio import Queue, Semaphore
import time

# Async context manager
class AsyncTimer:
    async def __aenter__(self):
        self.start = time.time()
        return self
    
    async def __aexit__(self, *args):
        self.end = time.time()
        print(f"Elapsed time: {self.end - self.start:.2f} seconds")

# Producer-consumer with asyncio
async def async_producer(queue: Queue, items):
    for item in items:
        await queue.put(item)
        await asyncio.sleep(0.1)
    await queue.put(None)  # Sentinel

async def async_consumer(queue: Queue, name):
    while True:
        item = await queue.get()
        if item is None:
            queue.task_done()
            break
        print(f"Consumer {name} processed: {item}")
        await asyncio.sleep(0.05)
        queue.task_done()

# Rate limiting with semaphore
async def rate_limited_request(session, url, semaphore):
    async with semaphore:
        async with session.get(url) as response:
            return await response.text()

async def concurrent_requests_with_rate_limiting():
    urls = [f"https://httpbin.org/delay/1" for _ in range(10)]
    semaphore = Semaphore(3)  # Limit to 3 concurrent requests
    
    async with aiohttp.ClientSession() as session:
        tasks = [rate_limited_request(session, url, semaphore) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)
    
    return results

# Async iterator
class AsyncRange:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.start >= self.stop:
            raise StopAsyncIteration
        current = self.start
        self.start += 1
        await asyncio.sleep(0.1)  # Simulate async operation
        return current

# Usage
async def use_async_iterator():
    async for i in AsyncRange(1, 5):
        print(f"Got: {i}")
```

---

## Advanced Data Structures

### Custom Collections

```python
from collections.abc import MutableMapping, MutableSequence
import bisect

class SortedDict(MutableMapping):
    """Dictionary that maintains sorted order of keys"""
    
    def __init__(self):
        self._keys = []
        self._values = []
    
    def __getitem__(self, key):
        index = bisect.bisect_left(self._keys, key)
        if index < len(self._keys) and self._keys[index] == key:
            return self._values[index]
        raise KeyError(key)
    
    def __setitem__(self, key, value):
        index = bisect.bisect_left(self._keys, key)
        if index < len(self._keys) and self._keys[index] == key:
            self._values[index] = value
        else:
            self._keys.insert(index, key)
            self._values.insert(index, value)
    
    def __delitem__(self, key):
        index = bisect.bisect_left(self._keys, key)
        if index < len(self._keys) and self._keys[index] == key:
            del self._keys[index]
            del self._values[index]
        else:
            raise KeyError(key)
    
    def __iter__(self):
        return iter(self._keys)
    
    def __len__(self):
        return len(self._keys)

# Trie data structure
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_word = True
    
    def search(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end_word
    
    def starts_with(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
    
    def words_with_prefix(self, prefix):
        """Return all words that start with the given prefix"""
        node = self.root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        
        def dfs(node, current_word):
            words = []
            if node.is_end_word:
                words.append(current_word)
            for char, child_node in node.children.items():
                words.extend(dfs(child_node, current_word + char))
            return words
        
        return dfs(node, prefix)
```

### Advanced Generators

```python
def chunked(iterable, size):
    """Yield successive chunks of specified size from iterable"""
    iterator = iter(iterable)
    while True:
        chunk = list(itertools.islice(iterator, size))
        if not chunk:
            break
        yield chunk

def sliding_window(iterable, window_size):
    """Yield sliding windows of specified size"""
    iterator = iter(iterable)
    window = collections.deque(itertools.islice(iterator, window_size), maxlen=window_size)
    if len(window) == window_size:
        yield tuple(window)
    for item in iterator:
        window.append(item)
        yield tuple(window)

def fibonacci_generator():
    """Infinite Fibonacci generator"""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

def pipeline_generator(*operations):
    """Create a generator pipeline"""
    def generator(iterable):
        for operation in operations:
            iterable = operation(iterable)
        yield from iterable
    return generator

# Usage examples
numbers = range(1, 21)

# Create pipeline
process_numbers = pipeline_generator(
    lambda x: filter(lambda n: n % 2 == 0, x),  # Even numbers
    lambda x: map(lambda n: n ** 2, x),         # Square them
    lambda x: filter(lambda n: n > 50, x)       # Greater than 50
)

result = list(process_numbers(numbers))
print(result)  # [64, 100, 144, 196, 256, 324, 400]

# Generator with send()
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

acc = accumulator()
next(acc)  # Prime the generator
print(acc.send(10))  # 10
print(acc.send(20))  # 30
print(acc.send(5))   # 35
```

### Context Managers

```python
import contextlib
import tempfile
import os
from typing import Generator, Any

# Custom context manager class
class DatabaseConnection:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None
    
    def __enter__(self):
        print(f"Connecting to {self.connection_string}")
        # Simulate connection
        self.connection = f"Connection to {self.connection_string}"
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing database connection")
        if exc_type is not None:
            print(f"Exception occurred: {exc_type.__name__}: {exc_val}")
            # Return True to suppress exception, False to propagate
            return False
        self.connection = None

# Context manager using contextlib
@contextlib.contextmanager
def temporary_file(content: str) -> Generator[str, None, None]:
    """Create a temporary file with content and clean up after use"""
    fd, path = tempfile.mkstemp()
    try:
        with os.fdopen(fd, 'w') as f:
            f.write(content)
        yield path
    finally:
        os.unlink(path)

# Multiple context managers
@contextlib.contextmanager
def multiple_resources():
    """Manage multiple resources"""
    resources = []
    try:
        for i in range(3):
            resource = f"Resource {i}"
            print(f"Acquiring {resource}")
            resources.append(resource)
            yield resource
    finally:
        for resource in reversed(resources):
            print(f"Releasing {resource}")

# Exception handling context manager
@contextlib.contextmanager
def ignore_errors(*exceptions):
    """Context manager that ignores specified exceptions"""
    try:
        yield
    except exceptions as e:
        print(f"Ignored exception: {e}")

# Usage examples
with DatabaseConnection("sqlite:///example.db") as conn:
    print(f"Using connection: {conn}")

with temporary_file("Hello, World!") as temp_path:
    with open(temp_path, 'r') as f:
        print(f"File content: {f.read()}")

with ignore_errors(ValueError, TypeError):
    int("not a number")  # This would normally raise ValueError
```

---

## Metaprogramming

### Dynamic Class Creation

```python
def create_class_with_methods(class_name, method_dict):
    """Dynamically create a class with specified methods"""
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)
    
    # Add __init__ method
    method_dict['__init__'] = __init__
    
    # Create class dynamically
    return type(class_name, (object,), method_dict)

# Create methods dictionary
methods = {
    'greet': lambda self: f"Hello, I'm {self.name}",
    'calculate': lambda self, x, y: x + y + self.bonus
}

# Create class dynamically
Person = create_class_with_methods('Person', methods)

# Use the dynamically created class
person = Person(name="Alice", bonus=10)
print(person.greet())  # "Hello, I'm Alice"
print(person.calculate(5, 3))  # 18

# Advanced metaclass for automatic registration
class RegisteredMeta(type):
    """Metaclass that automatically registers classes"""
    registry = {}
    
    def __new__(mcs, name, bases, attrs):
        cls = super().__new__(mcs, name, bases, attrs)
        if name != 'RegisteredBase':  # Don't register the base class
            mcs.registry[name] = cls
        return cls
    
    @classmethod
    def get_registered_classes(mcs):
        return mcs.registry

class RegisteredBase(metaclass=RegisteredMeta):
    pass

class ServiceA(RegisteredBase):
    pass

class ServiceB(RegisteredBase):
    pass

print(RegisteredMeta.get_registered_classes())
# {'ServiceA': <class '__main__.ServiceA'>, 'ServiceB': <class '__main__.ServiceB'>}
```

### Advanced Introspection

```python
import inspect
import types
from typing import get_type_hints

def analyze_function(func):
    """Comprehensive function analysis"""
    sig = inspect.signature(func)
    
    analysis = {
        'name': func.__name__,
        'module': func.__module__,
        'doc': inspect.getdoc(func),
        'source': inspect.getsource(func) if inspect.getsourcefile(func) else None,
        'parameters': {},
        'return_annotation': sig.return_annotation,
        'type_hints': get_type_hints(func),
        'is_coroutine': inspect.iscoroutinefunction(func),
        'is_generator': inspect.isgeneratorfunction(func)
    }
    
    for param_name, param in sig.parameters.items():
        analysis['parameters'][param_name] = {
            'kind': param.kind.name,
            'default': param.default if param.default != param.empty else None,
            'annotation': param.annotation if param.annotation != param.empty else None
        }
    
    return analysis

def dynamic_method_calling():
    """Demonstrate dynamic method resolution and calling"""
    
    class Calculator:
        def add(self, x, y):
            return x + y
        
        def multiply(self, x, y):
            return x * y
        
        def divide(self, x, y):
            if y == 0:
                raise ValueError("Cannot divide by zero")
            return x / y
    
    calc = Calculator()
    
    # Get all methods dynamically
    methods = inspect.getmembers(calc, predicate=inspect.ismethod)
    
    for method_name, method in methods:
        if not method_name.startswith('_'):
            try:
                result = method(10, 2)
                print(f"{method_name}(10, 2) = {result}")
            except Exception as e:
                print(f"{method_name} failed: {e}")

# Monkey patching with tracking
class PatchTracker:
    _patches = {}
    
    @classmethod
    def patch_method(cls, target_class, method_name, new_method):
        """Patch a method and keep track of original"""
        original_method = getattr(target_class, method_name, None)
        cls._patches[(target_class, method_name)] = original_method
        setattr(target_class, method_name, new_method)
    
    @classmethod
    def restore_method(cls, target_class, method_name):
        """Restore original method"""
        key = (target_class, method_name)
        if key in cls._patches:
            original_method = cls._patches[key]
            if original_method is not None:
                setattr(target_class, method_name, original_method)
            else:
                delattr(target_class, method_name)
            del cls._patches[key]
    
    @classmethod
    def restore_all(cls):
        """Restore all patched methods"""
        for (target_class, method_name) in list(cls._patches.keys()):
            cls.restore_method(target_class, method_name)
```

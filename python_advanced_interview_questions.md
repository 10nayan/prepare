# 100 Advanced Python Interview Questions and Answers

## Object-Oriented Programming

1. **Q: What is method resolution order (MRO) in Python?**
   - A: MRO is the order in which Python looks for methods and attributes in a class hierarchy. Python uses the C3 linearization algorithm to determine MRO. You can view a class's MRO using `ClassName.__mro__` or `ClassName.mro()`.

2. **Q: Explain metaclasses in Python.**
   - A: Metaclasses are classes for classes - they define how classes behave. They're used to modify class creation behavior. The most common use is creating APIs and frameworks.
   ```python
   class MyMetaclass(type):
       def __new__(cls, name, bases, attrs):
           # Modify class creation here
           return super().__new__(cls, name, bases, attrs)
   ```

3. **Q: What are abstract base classes?**
   - A: Abstract base classes (ABCs) are classes that contain one or more abstract methods. They can't be instantiated and require subclasses to provide implementations for the abstract methods.
   ```python
   from abc import ABC, abstractmethod
   class MyABC(ABC):
       @abstractmethod
       def my_method(self):
           pass
   ```

4. **Q: Explain descriptor protocol in Python.**
   - A: Descriptors define how attribute access is intercepted. They implement at least one of `__get__`, `__set__`, or `__delete__` methods. Property decorators use descriptors internally.

5. **Q: What is the difference between `__new__` and `__init__`?**
   - A: `__new__` is called to create a new instance, while `__init__` initializes the created instance. `__new__` is a static method that returns the instance, while `__init__` returns None.

## Memory Management

6. **Q: How does Python's memory management work?**
   - A: Python uses reference counting and garbage collection. When an object's reference count reaches zero, it's deallocated. The garbage collector handles circular references.

7. **Q: What is reference counting?**
   - A: Python keeps track of how many references point to an object. When the count reaches zero, the object is deallocated. You can check reference count using `sys.getrefcount(object)`.

8. **Q: Explain Python's garbage collection mechanism.**
   - A: Python's GC uses a generational garbage collection algorithm with three generations. New objects start in generation 0 and move to older generations if they survive collection cycles.

9. **Q: What is memory interning in Python?**
   - A: Python interns some strings and small integers for optimization. This means multiple references to the same value point to the same object in memory.

10. **Q: How can you manually manage memory in Python?**
    - A: While Python handles memory automatically, you can use:
    - `del` statement to remove references
    - `gc.collect()` to force garbage collection
    - Context managers (`with` statement) for resource management

## Multithreading and Multiprocessing

11. **Q: What is the Global Interpreter Lock (GIL)?**
    - A: The GIL is a mutex that protects access to Python objects, preventing multiple native threads from executing Python bytecode simultaneously. It makes multi-threaded CPU-bound programs slower than single-threaded ones.

12. **Q: How do you achieve true parallelism in Python?**
    - A: True parallelism can be achieved using:
    - multiprocessing module
    - concurrent.futures.ProcessPoolExecutor
    - Distributed computing libraries like Dask
    - Using non-CPython implementations (Jython, IronPython)

13. **Q: Explain the difference between multiprocessing and multithreading.**
    - A: 
    - Multithreading: Multiple threads share the same memory space but are limited by GIL
    - Multiprocessing: Multiple processes with separate memory spaces, not limited by GIL
    - Threads are better for I/O-bound tasks, processes for CPU-bound tasks

14. **Q: What is asyncio in Python?**
    - A: asyncio is a library for writing concurrent code using async/await syntax. It's used for I/O-bound and high-level structured network code.
    ```python
    async def main():
        await asyncio.sleep(1)
        return 'done'
    ```

15. **Q: How do you share data between processes?**
    - A: Data can be shared using:
    - multiprocessing.Value and Array
    - multiprocessing.Queue
    - multiprocessing.Pipe
    - Manager objects
    - Shared memory with mmap

## Decorators and Generators

16. **Q: What are function decorators?**
    - A: Decorators are functions that modify other functions' behavior. They use the @syntax and are commonly used for logging, timing, access control, etc.
    ```python
    def my_decorator(func):
        def wrapper():
            print("Before")
            func()
            print("After")
        return wrapper
    ```

17. **Q: What are class decorators?**
    - A: Class decorators are similar to function decorators but are applied to classes. They can modify class creation and instance creation behavior.
    ```python
    def singleton(cls):
        instances = {}
        def get_instance(*args, **kwargs):
            if cls not in instances:
                instances[cls] = cls(*args, **kwargs)
            return instances[cls]
        return get_instance
    ```

18. **Q: Explain generator functions and expressions.**
    - A: Generators are functions that use yield to return values one at a time. Generator expressions are like list comprehensions but create a generator object.
    ```python
    def gen_func():
        yield 1
        yield 2
    
    gen_expr = (x**2 for x in range(10))
    ```

19. **Q: What is the yield from statement?**
    - A: yield from delegates the generation of values to a sub-generator, enabling better composability of generators.
    ```python
    def sub_gen():
        yield 1
        yield 2
    
    def main_gen():
        yield from sub_gen()
    ```

20. **Q: What are the advantages of generators over lists?**
    - A: Generators:
    - Use less memory (values generated on demand)
    - Can represent infinite sequences
    - Better for large datasets
    - More memory efficient in loops

## Advanced Data Structures

21. **Q: Explain collections.defaultdict.**
    - A: defaultdict is a dict subclass that calls a factory function to supply missing values.
    ```python
    from collections import defaultdict
    d = defaultdict(list)
    d['key'].append(1)  # No KeyError if key doesn't exist
    ```

22. **Q: What is collections.Counter used for?**
    - A: Counter is a dict subclass for counting hashable objects. It provides methods for arithmetic operations on counts.
    ```python
    from collections import Counter
    c = Counter(['a', 'b', 'a'])  # Counter({'a': 2, 'b': 1})
    ```

23. **Q: How does collections.deque work?**
    - A: deque is a double-ended queue with O(1) append/pop operations from either end. It's more efficient than lists for these operations.
    ```python
    from collections import deque
    d = deque([1, 2, 3])
    d.appendleft(0)
    d.append(4)
    ```

24. **Q: What is heapq used for?**
    - A: heapq implements a min heap queue algorithm. It's useful for maintaining a priority queue or finding N smallest/largest items.
    ```python
    import heapq
    heap = []
    heapq.heappush(heap, 1)
    smallest = heapq.heappop(heap)
    ```

25. **Q: Explain bisect module's functionality.**
    - A: bisect provides functions for binary search and insertion into sorted sequences.
    ```python
    import bisect
    sorted_list = [1, 3, 5, 7]
    bisect.insort(sorted_list, 4)  # Maintains sort order
    ```

## Design Patterns

26. **Q: What is the Singleton pattern in Python?**
    - A: Singleton ensures a class has only one instance. It can be implemented using:
    ```python
    class Singleton:
        _instance = None
        def __new__(cls):
            if cls._instance is None:
                cls._instance = super().__new__(cls)
            return cls._instance
    ```

27. **Q: Explain the Factory pattern.**
    - A: Factory pattern provides an interface for creating objects without specifying their exact classes.
    ```python
    class Factory:
        def create_animal(self, animal_type):
            if animal_type == "dog":
                return Dog()
            elif animal_type == "cat":
                return Cat()
    ```

28. **Q: What is the Observer pattern?**
    - A: Observer pattern allows objects to be notified when state changes occur in other objects.
    ```python
    class Subject:
        def __init__(self):
            self._observers = []
        
        def notify(self):
            for observer in self._observers:
                observer.update()
    ```

29. **Q: How do you implement the Strategy pattern?**
    - A: Strategy pattern enables selecting an algorithm at runtime.
    ```python
    class Strategy:
        def execute(self):
            pass
    
    class Context:
        def __init__(self, strategy):
            self._strategy = strategy
        
        def execute_strategy(self):
            self._strategy.execute()
    ```

30. **Q: What is the Decorator pattern?**
    - A: Decorator pattern adds behavior to objects dynamically.
    ```python
    class Component:
        def operation(self):
            pass
    
    class Decorator(Component):
        def __init__(self, component):
            self._component = component
        
        def operation(self):
            return self._component.operation()
    ```

## Performance Optimization

31. **Q: How can you profile Python code?**
    - A: Python provides several profiling tools:
    - cProfile for function-level profiling
    - line_profiler for line-by-line profiling
    - memory_profiler for memory usage
    - timeit for quick timing

32. **Q: What are list comprehensions and when should they be used?**
    - A: List comprehensions provide a concise way to create lists. They're often more readable and faster than loops.
    ```python
    squares = [x**2 for x in range(10)]
    filtered = [x for x in range(100) if x % 2 == 0]
    ```

33. **Q: How can you optimize dictionary access?**
    - A: Optimize dictionary access by:
    - Using dict.get() with default value
    - Using collections.defaultdict
    - Using dict comprehension
    - Avoiding repeated key lookups

34. **Q: What is the difference between range and xrange (Python 2)?**
    - A: In Python 2:
    - range creates a list in memory
    - xrange creates an iterator object
    In Python 3, range is like xrange, creating an iterator.

35. **Q: How can you reduce memory usage in Python?**
    - A: Techniques include:
    - Using generators instead of lists
    - Using __slots__ to restrict attributes
    - Using array module for numeric data
    - Proper garbage collection
    - Using itertools for efficient iteration

## Python Internals

36. **Q: How does Python's import system work?**
    - A: Python's import system:
    1. Searches for the module in sys.modules
    2. Looks in various locations (sys.path)
    3. Loads and compiles the code
    4. Creates a module object
    5. Executes the module code

37. **Q: What are Python's data model methods?**
    - A: Data model methods (magic/dunder methods) define how objects behave:
    - `__init__` for initialization
    - `__str__` for string representation
    - `__len__` for length
    - `__call__` for callable objects
    - Many others

38. **Q: How does Python handle closures?**
    - A: Closures are functions that remember values from their enclosing scope:
    ```python
    def outer(x):
        def inner():
            return x  # x is from enclosing scope
        return inner
    ```

39. **Q: What is the difference between .py and .pyc files?**
    - A: 
    - .py files contain source code
    - .pyc files contain compiled bytecode
    - .pyc files are created for performance
    - Python checks if .pyc is newer than .py

40. **Q: How does Python's bytecode work?**
    - A: Python compiles source to bytecode which is then executed by the Python Virtual Machine (PVM). You can view bytecode using:
    ```python
    import dis
    dis.dis(function_name)
    ```

## Exception Handling

41. **Q: What is the difference between raise and raise from?**
    - A: raise from explicitly chains exceptions, preserving the traceback:
    ```python
    try:
        raise ValueError("Original")
    except ValueError as e:
        raise RuntimeError("New") from e
    ```

42. **Q: How do you create custom exceptions?**
    - A: Custom exceptions inherit from Exception or its subclasses:
    ```python
    class MyCustomError(Exception):
        def __init__(self, message, code):
            self.message = message
            self.code = code
    ```

43. **Q: What is the purpose of finally clause?**
    - A: finally ensures code execution regardless of exceptions:
    - Always executes after try/except
    - Used for cleanup operations
    - Runs even if return/break/continue in try

44. **Q: How do context managers work?**
    - A: Context managers implement `__enter__` and `__exit__` methods:
    ```python
    class MyContext:
        def __enter__(self):
            print("Entering")
            return self
        
        def __exit__(self, exc_type, exc_val, exc_tb):
            print("Exiting")
    ```

45. **Q: What is the difference between except and except Exception?**
    - A:
    - except: catches all exceptions (not recommended)
    - except Exception: catches all built-in exceptions
    - Best practice is to catch specific exceptions

## Advanced Standard Library Features

46. **Q: How does functools.lru_cache work?**
    - A: lru_cache implements memoization, caching function results:
    ```python
    from functools import lru_cache
    
    @lru_cache(maxsize=None)
    def fibonacci(n):
        if n < 2:
            return n
        return fibonacci(n-1) + fibonacci(n-2)
    ```

47. **Q: What is itertools.chain used for?**
    - A: chain combines multiple iterables into a single iterator:
    ```python
    from itertools import chain
    combined = chain([1, 2], [3, 4])  # 1, 2, 3, 4
    ```

48. **Q: Explain functools.partial.**
    - A: partial creates a new function with pre-filled arguments:
    ```python
    from functools import partial
    basetwo = partial(int, base=2)
    basetwo('10')  # 2
    ```

49. **Q: How does contextlib.contextmanager work?**
    - A: contextmanager decorator simplifies creating context managers:
    ```python
    from contextlib import contextmanager
    
    @contextmanager
    def temp_file():
        f = open('temp.txt', 'w')
        try:
            yield f
        finally:
            f.close()
    ```

50. **Q: What is operator.itemgetter used for?**
    - A: itemgetter creates a callable that fetches items from sequences:
    ```python
    from operator import itemgetter
    data = [(1, 'one'), (2, 'two')]
    get_second = itemgetter(1)
    [get_second(item) for item in data]  # ['one', 'two']
    ```

## Advanced Python Features

51. **Q: What are slots in Python classes?**
    - A: __slots__ restricts attribute creation and reduces memory usage:
    ```python
    class Point:
        __slots__ = ['x', 'y']
        def __init__(self, x, y):
            self.x = x
            self.y = y
    ```

52. **Q: How do you use weak references?**
    - A: Weak references don't prevent garbage collection:
    ```python
    import weakref
    class Cache:
        def __init__(self):
            self._cache = weakref.WeakValueDictionary()
    ```

53. **Q: What are property decorators?**
    - A: property decorators create managed attributes:
    ```python
    class Circle:
        def __init__(self, radius):
            self._radius = radius
        
        @property
        def area(self):
            return 3.14 * self._radius ** 2
    ```

54. **Q: Explain the difference between is and ==.**
    - A:
    - is checks identity (same object)
    - == checks equality (same value)
    - Use is for None comparisons
    - Use == for value comparisons

55. **Q: How do you use __call__ method?**
    - A: __call__ makes instances callable:
    ```python
    class Multiplier:
        def __init__(self, factor):
            self.factor = factor
        
        def __call__(self, x):
            return x * self.factor
    ```

## Advanced Concepts

56. **Q: What are Python's descriptors?**
    - A: Descriptors control attribute access:
    ```python
    class Descriptor:
        def __get__(self, obj, owner):
            return obj._value
        
        def __set__(self, obj, value):
            obj._value = value
    ```

57. **Q: How does the with statement work internally?**
    - A: with uses context manager protocol:
    1. Calls __enter__
    2. Executes block
    3. Calls __exit__ (even if exception occurs)

58. **Q: What is monkey patching?**
    - A: Monkey patching is runtime modification of classes/modules:
    ```python
    def new_method():
        pass
    
    class MyClass:
        pass
    
    MyClass.new_method = new_method
    ```

59. **Q: How do you implement custom containers?**
    - A: Implement container methods:
    ```python
    class CustomList:
        def __len__(self):
            pass
        
        def __getitem__(self, index):
            pass
        
        def __setitem__(self, index, value):
            pass
    ```

60. **Q: What are metaclass conflicts?**
    - A: Metaclass conflicts occur when classes with different metaclasses are combined:
    - Can happen in multiple inheritance
    - Must ensure metaclass compatibility
    - Can be resolved with proper metaclass hierarchy

## Advanced Libraries and Tools

61. **Q: How does unittest.mock work?**
    - A: mock replaces parts of system for testing:
    ```python
    from unittest.mock import Mock
    thing = Mock()
    thing.method.return_value = 3
    thing.method()  # Returns 3
    ```

62. **Q: What is the purpose of typing module?**
    - A: typing provides type hints:
    ```python
    from typing import List, Dict
    
    def process_items(items: List[str]) -> Dict[str, int]:
        return {item: len(item) for item in items}
    ```

63. **Q: How do you use dataclasses?**
    - A: dataclasses automatically add methods:
    ```python
    from dataclasses import dataclass
    
    @dataclass
    class Point:
        x: float
        y: float
    ```

64. **Q: What is asyncio.gather used for?**
    - A: gather runs coroutines concurrently:
    ```python
    async def main():
        results = await asyncio.gather(
            coroutine1(),
            coroutine2()
        )
    ```

65. **Q: How do you use pathlib?**
    - A: pathlib provides object-oriented filesystem paths:
    ```python
    from pathlib import Path
    p = Path('.')
    [x for x in p.glob('**/*.py')]
    ```

## Advanced Debugging

66. **Q: How do you use pdb?**
    - A: pdb is Python's debugger:
    ```python
    import pdb; pdb.set_trace()
    # Commands: n (next), s (step), c (continue)
    ```

67. **Q: What is sys.settrace used for?**
    - A: settrace sets a trace function for debugging:
    ```python
    def trace_calls(frame, event, arg):
        if event == 'call':
            print(f"Calling {frame.f_code.co_name}")
        return trace_calls
    
    sys.settrace(trace_calls)
    ```

68. **Q: How do you profile memory usage?**
    - A: Use memory_profiler:
    ```python
    from memory_profiler import profile
    
    @profile
    def my_function():
        pass
    ```

69. **Q: What is logging best practices?**
    - A: Logging best practices include:
    - Using proper log levels
    - Structured logging
    - Configuring handlers appropriately
    - Including context information

70. **Q: How do you debug multiprocessing?**
    - A: Debug multiprocessing using:
    - logging module with process name
    - remote debugger
    - print statements with process ID
    - multiprocessing.log_to_stderr()

## Advanced Testing

71. **Q: How do you use pytest fixtures?**
    - A: Fixtures provide test setup/teardown:
    ```python
    import pytest
    
    @pytest.fixture
    def setup_data():
        data = {"key": "value"}
        yield data
        # cleanup code here
    ```

72. **Q: What is test parameterization?**
    - A: Parameterization runs tests with different inputs:
    ```python
    @pytest.mark.parametrize("input,expected", [
        (1, 2),
        (2, 4),
        (3, 6)
    ])
    def test_double(input, expected):
        assert double(input) == expected
    ```

73. **Q: How do you mock database connections?**
    - A: Use unittest.mock or pytest-mock:
    ```python
    @patch('module.Database')
    def test_db(mock_db):
        mock_db.return_value.query.return_value = ['result']
    ```

74. **Q: What is property-based testing?**
    - A: Property-based testing generates test cases:
    ```python
    from hypothesis import given
    from hypothesis.strategies import integers
    
    @given(integers())
    def test_abs_is_positive(x):
        assert abs(x) >= 0
    ```

75. **Q: How do you measure test coverage?**
    - A: Use coverage.py:
    ```bash
    coverage run -m pytest
    coverage report
    coverage html
    ```

## Advanced Packaging

76. **Q: How do you create a Python package?**
    - A: Package structure:
    ```
    mypackage/
        setup.py
        mypackage/
            __init__.py
            module1.py
    ```

77. **Q: What is wheel format?**
    - A: Wheel is a built-package format:
    - Faster installation
    - Pre-built binaries
    - Created using `python setup.py bdist_wheel`

78. **Q: How do you use setuptools?**
    - A: setuptools configures package installation:
    ```python
    from setuptools import setup
    
    setup(
        name='mypackage',
        version='1.0',
        packages=['mypackage']
    )
    ```

79. **Q: What is pip's dependency resolution?**
    - A: pip resolves dependencies by:
    - Reading requirements.txt
    - Checking version constraints
    - Building dependency graph
    - Resolving conflicts

80. **Q: How do you use virtual environments?**
    - A: Virtual environments isolate dependencies:
    ```bash
    python -m venv myenv
    source myenv/bin/activate  # Unix
    myenv\Scripts\activate.bat  # Windows
    ```

## Advanced Security

81. **Q: How do you handle sensitive data?**
    - A: Best practices:
    - Use environment variables
    - Use secure configuration
    - Encrypt sensitive data
    - Use secure storage solutions

82. **Q: What is SQL injection prevention?**
    - A: Prevent SQL injection:
    ```python
    # Bad
    cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")
    
    # Good
    cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
    ```

83. **Q: How do you implement rate limiting?**
    - A: Rate limiting example:
    ```python
    from functools import wraps
    from time import time
    
    def rate_limit(calls_per_second):
        min_interval = 1.0 / calls_per_second
        last_called = [0.0]
        
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                now = time()
                elapsed = now - last_called[0]
                if elapsed < min_interval:
                    time.sleep(min_interval - elapsed)
                result = func(*args, **kwargs)
                last_called[0] = time()
                return result
            return wrapper
        return decorator
    ```

84. **Q: What is CSRF protection?**
    - A: CSRF protection methods:
    - Use tokens in forms
    - Validate request origin
    - Use secure session handling
    - Implement proper headers

85. **Q: How do you implement authentication?**
    - A: Authentication implementation:
    ```python
    from functools import wraps
    
    def require_auth(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            auth = request.authorization
            if not auth or not check_auth(auth.username, auth.password):
                return authenticate()
            return f(*args, **kwargs)
        return decorated
    ```

## Advanced Web Development

86. **Q: How do you implement websockets?**
    - A: Using websockets:
    ```python
    import websockets
    
    async def handler(websocket, path):
        async for message in websocket:
            await websocket.send(f"Got {message}")
    
    start_server = websockets.serve(handler, 'localhost', 8765)
    ```

87. **Q: What is ASGI?**
    - A: ASGI (Asynchronous Server Gateway Interface):
    - Successor to WSGI
    - Handles async requests
    - Supports WebSocket
    - Used by FastAPI, Starlette

88. **Q: How do you implement caching?**
    - A: Caching implementation:
    ```python
    from functools import lru_cache
    from django.core.cache import cache
    
    @lru_cache(maxsize=100)
    def expensive_computation():
        pass
    ```

89. **Q: What is GraphQL in Python?**
    - A: GraphQL implementation:
    ```python
    import graphene
    
    class Query(graphene.ObjectType):
        hello = graphene.String()
        
        def resolve_hello(self, info):
            return 'World'
    ```

90. **Q: How do you handle file uploads?**
    - A: File upload handling:
    ```python
    def handle_upload(request):
        file = request.files['file']
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
    ```

## Advanced Data Processing

91. **Q: How do you use numpy efficiently?**
    - A: Numpy optimization:
    ```python
    import numpy as np
    
    # Vectorized operations
    arr = np.array([1, 2, 3])
    result = arr * 2  # Instead of loop
    ```

92. **Q: What is pandas optimization?**
    - A: Pandas performance tips:
    ```python
    import pandas as pd
    
    # Use vectorized operations
    df['new'] = df['col'].apply(lambda x: x*2)  # Slower
    df['new'] = df['col'] * 2  # Faster
    ```

93. **Q: How do you handle large datasets?**
    - A: Large dataset handling:
    ```python
    # Use chunks
    for chunk in pd.read_csv('large.csv', chunksize=10000):
        process_chunk(chunk)
    ```

94. **Q: What is parallel processing in pandas?**
    - A: Parallel processing:
    ```python
    import pandas as pd
    import multiprocessing as mp
    
    def parallel_apply(df, func):
        with mp.Pool(mp.cpu_count()) as pool:
            return pd.concat(pool.map(func, np.array_split(df, mp.cpu_count())))
    ```

95. **Q: How do you optimize database queries?**
    - A: Query optimization:
    ```python
    # Use select_related for foreign keys
    queryset = Model.objects.select_related('foreign_key').filter(condition=value)
    ```

## System Integration

96. **Q: How do you implement RPC?**
    - A: RPC implementation:
    ```python
    import xmlrpc.client
    import xmlrpc.server
    
    class RPCHandler:
        def handle_request(self, data):
            return process_data(data)
    ```

97. **Q: What is message queuing?**
    - A: Message queue usage:
    ```python
    import pika
    
    connection = pika.BlockingConnection()
    channel = connection.channel()
    channel.queue_declare(queue='task_queue')
    ```

98. **Q: How do you implement microservices?**
    - A: Microservices implementation:
    ```python
    from flask import Flask
    
    app = Flask(__name__)
    
    @app.route('/api/v1/service')
    def microservice():
        return jsonify({'status': 'ok'})
    ```

99. **Q: What is service discovery?**
    - A: Service discovery:
    ```python
    import consul
    
    c = consul.Consul()
    c.agent.service.register('my_service',
                           service_id='my_service_1',
                           port=8080)
    ```

100. **Q: How do you implement API versioning?**
     - A: API versioning:
     ```python
     @app.route('/api/v1/resource')
     def resource_v1():
         return 'v1'
     
     @app.route('/api/v2/resource')
     def resource_v2():
         return 'v2'

# Dart Comprehensive Cheatsheet

## Table of Contents
1. [Data Types](#data-types)
2. [Variables](#variables)
3. [Constants](#constants)
4. [Lists](#lists)
5. [Maps](#maps)
6. [Other Data Structures](#other-data-structures)
7. [Functions](#functions)
8. [OOP (Object-Oriented Programming)](#oop)
9. [Late Keyword](#late-keyword)
10. [Null Assertion Operator (!)](#null-assertion-operator)
11. [Arguments (Optional, Required, Positional, Named, Default)](#arguments)
12. [Exceptions](#exceptions)
13. [Generics](#generics)
14. [Null Safety](#null-safety)
15. [Required Keyword](#required-keyword)
16. [Static Keyword](#static-keyword)
17. [Ternary Operator](#ternary-operator)

---

## Data Types

### Purpose
Dart is a strongly-typed language. Data types define what kind of values a variable can hold.

### Core Types
- **int**: Integer numbers (whole numbers)
- **double**: Floating-point numbers (decimals)
- **num**: Supertype for both int and double
- **String**: Text/characters
- **bool**: Boolean (true/false)
- **dynamic**: Can hold any type (loses type safety)
- **Object**: Base class for all Dart objects
- **void**: Represents no value (used for functions that don't return)

### Syntax & Examples

```dart
// Integer
int age = 25;
int hexValue = 0xDEADBEEF;

// Double
double price = 19.99;
double exponent = 1.42e5; // 142000.0

// Num (can be int or double)
num temperature = 98.6;
temperature = 100; // Also valid

// String
String name = 'John';
String greeting = "Hello, World!";
String multiline = '''
  This is a
  multiline string
''';
String interpolation = 'My name is $name and I am $age years old';
String expression = 'Next year I will be ${age + 1}';

// Boolean
bool isActive = true;
bool isComplete = false;

// Dynamic (avoid when possible)
dynamic anything = 'text';
anything = 42;
anything = true;
```

### Use Cases
- **int**: Counters, IDs, quantities
- **double**: Prices, measurements, calculations
- **String**: Names, messages, text data
- **bool**: Flags, conditions, toggle states
- **dynamic**: JSON parsing, interop with untyped code

---

## Variables

### Purpose
Variables store data that can be changed during program execution.

### Syntax

```dart
// Explicit type declaration
int count = 0;
String message = 'Hello';

// Type inference using 'var'
var number = 42; // Inferred as int
var text = 'Dart'; // Inferred as String

// Can reassign variables
count = 10;
message = 'Goodbye';
```

### Use Cases
- Storing user input
- Accumulating values in loops
- Tracking state changes
- Temporary calculations

### Examples

```dart
var counter = 0;
for (int i = 0; i < 5; i++) {
  counter += i;
}
print(counter); // 10

String username = 'guest';
if (loggedIn) {
  username = 'john_doe';
}
```

---

## Constants

### Purpose
Constants are immutable values that cannot be changed after initialization. Dart has two types: `const` and `final`.

### const vs final
- **const**: Compile-time constant (value must be known at compile time)
- **final**: Runtime constant (can be set once at runtime)

### Syntax

```dart
// const - compile-time constant
const double pi = 3.14159;
const String appName = 'MyApp';
const int maxUsers = 100;

// final - runtime constant
final DateTime now = DateTime.now();
final String userId = generateUserId();
final List<int> numbers = [1, 2, 3];
```

### Use Cases
- **const**: Configuration values, mathematical constants, fixed strings
- **final**: Values computed at runtime that shouldn't change, API responses, user data

### Examples

```dart
// const for compile-time values
const String apiUrl = 'https://api.example.com';
const int timeout = 30;

// final for runtime values
final String timestamp = DateTime.now().toString();
final int randomNumber = Random().nextInt(100);

// Error: const requires compile-time constant
// const DateTime today = DateTime.now(); // Won't compile

// const collections are deeply immutable
const List<int> fixedList = [1, 2, 3];
// fixedList.add(4); // Error: Cannot modify const list
```

---

## Lists

### Purpose
Lists are ordered collections of items (arrays in other languages). Can be fixed or growable.

### Syntax

```dart
// List with type inference
var fruits = ['apple', 'banana', 'orange'];

// Explicit type
List<int> numbers = [1, 2, 3, 4, 5];

// Empty list
List<String> names = [];

// Fixed-length list
var fixedList = List.filled(5, 0); // [0, 0, 0, 0, 0]

// Growable list (default)
var growable = <int>[];
```

### Common Operations

```dart
List<String> colors = ['red', 'green', 'blue'];

// Add elements
colors.add('yellow');
colors.addAll(['purple', 'orange']);
colors.insert(0, 'pink'); // Insert at index

// Access elements
String first = colors[0];
String last = colors[colors.length - 1];
String lastAlt = colors.last;

// Remove elements
colors.remove('red');
colors.removeAt(0);
colors.removeLast();
colors.clear();

// Check and search
bool hasRed = colors.contains('red');
int index = colors.indexOf('blue'); // -1 if not found

// Length
int size = colors.length;
bool isEmpty = colors.isEmpty;

// Iteration
for (var color in colors) {
  print(color);
}

colors.forEach((color) => print(color));
```

### Advanced Features

```dart
// Spread operator
var list1 = [1, 2, 3];
var list2 = [0, ...list1, 4]; // [0, 1, 2, 3, 4]

// Null-aware spread
List<int>? nullableList;
var list3 = [...?nullableList]; // [] if null

// Collection if
var nav = [
  'Home',
  'About',
  if (loggedIn) 'Logout',
];

// Collection for
var numbers = [1, 2, 3];
var doubled = [
  for (var n in numbers) n * 2
]; // [2, 4, 6]

// List methods
var nums = [1, 2, 3, 4, 5];
var evens = nums.where((n) => n % 2 == 0).toList(); // [2, 4]
var squares = nums.map((n) => n * n).toList(); // [1, 4, 9, 16, 25]
var sum = nums.reduce((a, b) => a + b); // 15
```

### Use Cases
- Storing collections of items
- Managing ordered data
- Iterating over sequences
- Building UI lists (ListView in Flutter)

---

## Maps

### Purpose
Maps are key-value pair collections (dictionaries/hash maps in other languages).

### Syntax

```dart
// Map literal
var person = {
  'name': 'John',
  'age': 30,
  'city': 'New York'
};

// Explicit type
Map<String, int> scores = {
  'Alice': 95,
  'Bob': 87,
  'Charlie': 92
};

// Empty map
var emptyMap = <String, dynamic>{};
Map<int, String> ids = {};

// Using Map constructor
var map = Map<String, int>();
```

### Common Operations

```dart
Map<String, dynamic> user = {
  'username': 'john_doe',
  'email': 'john@example.com',
  'age': 25
};

// Access values
String? email = user['email'];
int age = user['age'] ?? 0; // With default

// Add or update
user['phone'] = '555-1234';
user['age'] = 26;

// Remove
user.remove('phone');

// Check keys
bool hasEmail = user.containsKey('email');
bool hasValue = user.containsValue('john_doe');

// Get keys and values
var keys = user.keys; // ('username', 'email', 'age')
var values = user.values; // ('john_doe', 'john@example.com', 26)

// Length
int size = user.length;
bool isEmpty = user.isEmpty;

// Iteration
user.forEach((key, value) {
  print('$key: $value');
});

for (var entry in user.entries) {
  print('${entry.key}: ${entry.value}');
}
```

### Advanced Features

```dart
// Spread operator
var defaults = {'theme': 'dark', 'language': 'en'};
var settings = {
  ...defaults,
  'notifications': true
};

// Collection if
var config = {
  'version': '1.0',
  if (isDevelopment) 'debug': true,
};

// putIfAbsent
Map<String, int> cache = {};
cache.putIfAbsent('key', () => expensiveCalculation());

// Map methods
var prices = {'apple': 1.50, 'banana': 0.75, 'orange': 2.00};
var doubled = prices.map((k, v) => MapEntry(k, v * 2));
```

### Use Cases
- Configuration settings
- JSON data representation
- Caching key-value pairs
- Lookup tables
- User profiles

---

## Other Data Structures

### Sets

#### Purpose
Sets are unordered collections of unique items (no duplicates).

#### Syntax

```dart
// Set literal
var uniqueNumbers = {1, 2, 3, 4, 5};

// Explicit type
Set<String> tags = {'dart', 'flutter', 'mobile'};

// Empty set (must specify type or use Set())
var emptySet = <int>{};
Set<String> names = {};

// From list (removes duplicates)
var list = [1, 2, 2, 3, 3, 3];
var set = list.toSet(); // {1, 2, 3}
```

#### Operations

```dart
Set<int> numbers = {1, 2, 3};

// Add elements
numbers.add(4);
numbers.addAll([5, 6]);

// Remove
numbers.remove(1);

// Check membership
bool hasTwo = numbers.contains(2);

// Set operations
var a = {1, 2, 3};
var b = {3, 4, 5};

var union = a.union(b); // {1, 2, 3, 4, 5}
var intersection = a.intersection(b); // {3}
var difference = a.difference(b); // {1, 2}
```

#### Use Cases
- Removing duplicates
- Membership testing
- Mathematical set operations
- Unique tags or categories

### Queues

#### Purpose
Queues are collections optimized for adding/removing elements at the ends (FIFO/LIFO).

#### Syntax

```dart
import 'dart:collection';

// Queue
Queue<int> queue = Queue();
queue.addLast(1);
queue.addLast(2);
queue.addFirst(0);
int first = queue.removeFirst();
```

#### Use Cases
- Task scheduling
- Breadth-first search
- Event processing

### Runes

#### Purpose
Runes represent Unicode code points of a string (for emoji and special characters).

#### Syntax

```dart
var heart = '\u2665'; // ♥
var runes = Runes('\u{1F600}'); // 😀
String emoji = String.fromCharCodes(runes);
```

---

## Functions

### Purpose
Functions are reusable blocks of code that perform specific tasks.

### Syntax

```dart
// Basic function
void greet() {
  print('Hello!');
}

// Function with parameters
void sayHello(String name) {
  print('Hello, $name!');
}

// Function with return value
int add(int a, int b) {
  return a + b;
}

// Arrow function (single expression)
int multiply(int a, int b) => a * b;

// Type inference
var divide = (int a, int b) => a / b;
```

### Advanced Function Features

```dart
// Optional positional parameters
String greet(String name, [String? title]) {
  if (title != null) {
    return 'Hello, $title $name';
  }
  return 'Hello, $name';
}
greet('John'); // "Hello, John"
greet('John', 'Dr.'); // "Hello, Dr. John"

// Named parameters
void createUser({String? name, int? age, String? email}) {
  print('Name: $name, Age: $age, Email: $email');
}
createUser(name: 'Alice', age: 25);

// Required named parameters
void register({required String email, required String password}) {
  // email and password must be provided
}
register(email: 'user@example.com', password: 'secret');

// Default parameter values
int power(int base, [int exponent = 2]) {
  return base * base; // simplified
}

// Named parameters with defaults
void configure({String host = 'localhost', int port = 8080}) {
  print('$host:$port');
}

// First-class functions (assign to variables)
var operation = add;
print(operation(5, 3)); // 8

// Higher-order functions (functions as parameters)
void executeOperation(int a, int b, int Function(int, int) op) {
  print(op(a, b));
}
executeOperation(10, 5, add);

// Closures
Function makeMultiplier(int factor) {
  return (int value) => value * factor;
}
var triple = makeMultiplier(3);
print(triple(5)); // 15

// Anonymous functions
var numbers = [1, 2, 3, 4];
numbers.forEach((n) {
  print(n * 2);
});
```

### Use Cases
- Code reusability
- Organizing logic
- Callbacks and event handlers
- Higher-order operations (map, filter, reduce)

---

## OOP (Object-Oriented Programming)

### Purpose
OOP organizes code into objects that contain data (properties) and behavior (methods).

### Classes and Objects

```dart
// Basic class
class Person {
  String name;
  int age;

  // Constructor
  Person(this.name, this.age);

  // Method
  void introduce() {
    print('Hi, I am $name and I am $age years old.');
  }
}

// Create object
var person = Person('Alice', 30);
person.introduce();
```

### Constructors

```dart
class User {
  String username;
  String email;
  int age;

  // Default constructor
  User(this.username, this.email, this.age);

  // Named constructor
  User.guest() : username = 'guest', email = '', age = 0;

  User.fromJson(Map<String, dynamic> json)
      : username = json['username'],
        email = json['email'],
        age = json['age'];

  // Factory constructor
  factory User.create(String username) {
    return User(username, '$username@example.com', 0);
  }
}

// Usage
var user1 = User('john', 'john@example.com', 25);
var user2 = User.guest();
var user3 = User.fromJson({'username': 'alice', 'email': 'alice@example.com', 'age': 30});
```

### Getters and Setters

```dart
class Rectangle {
  double width;
  double height;

  Rectangle(this.width, this.height);

  // Getter
  double get area => width * height;

  // Setter
  set dimensions(List<double> dims) {
    width = dims[0];
    height = dims[1];
  }
}

var rect = Rectangle(5, 10);
print(rect.area); // 50
rect.dimensions = [7, 3];
```

### Inheritance

```dart
class Animal {
  String name;

  Animal(this.name);

  void makeSound() {
    print('Some sound');
  }
}

class Dog extends Animal {
  String breed;

  Dog(String name, this.breed) : super(name);

  @override
  void makeSound() {
    print('Woof!');
  }

  void fetch() {
    print('$name is fetching!');
  }
}

var dog = Dog('Buddy', 'Golden Retriever');
dog.makeSound(); // "Woof!"
dog.fetch();
```

### Abstract Classes and Interfaces

```dart
// Abstract class (cannot be instantiated)
abstract class Shape {
  double getArea(); // Abstract method

  void describe() {
    print('This is a shape with area ${getArea()}');
  }
}

class Circle extends Shape {
  double radius;

  Circle(this.radius);

  @override
  double getArea() => 3.14 * radius * radius;
}

// Implicit interfaces (every class defines an interface)
class Printer {
  void printData(String data) {
    print('Printing: $data');
  }
}

class ConsolePrinter implements Printer {
  @override
  void printData(String data) {
    print('Console: $data');
  }
}
```

### Mixins

```dart
// Mixin (code reuse without inheritance)
mixin Flyable {
  void fly() {
    print('Flying!');
  }
}

mixin Swimmable {
  void swim() {
    print('Swimming!');
  }
}

class Duck with Flyable, Swimmable {
  String name;
  Duck(this.name);
}

var duck = Duck('Donald');
duck.fly();
duck.swim();
```

### Private Members

```dart
class BankAccount {
  String _accountNumber; // Private (prefix with _)
  double _balance = 0.0;

  BankAccount(this._accountNumber);

  double get balance => _balance;

  void deposit(double amount) {
    _balance += amount;
  }
}
```

### Use Cases
- Modeling real-world entities
- Creating reusable components
- Organizing complex applications
- Building frameworks and libraries

---

## Late Keyword

### Purpose
The `late` keyword indicates that a variable will be initialized later but before it's used. Useful for non-nullable variables that can't be initialized immediately.

### Syntax

```dart
// Late initialization
late String description;

void initialize() {
  description = 'This is initialized later';
}

void printDescription() {
  print(description); // Must be initialized before use
}

// Late final (initialize once, later)
late final String config;

void setup() {
  config = loadConfig(); // Can only be set once
}
```

### Use Cases

```dart
// 1. Circular dependencies
class A {
  late B b;

  A() {
    b = B(this);
  }
}

class B {
  A a;
  B(this.a);
}

// 2. Expensive initialization (lazy loading)
class HeavyObject {
  late List<int> data = _generateHugeList(); // Only computed when accessed

  List<int> _generateHugeList() {
    return List.generate(1000000, (i) => i);
  }
}

// 3. Dependency injection
class Service {
  late Database db; // Will be injected by framework
}

// 4. Non-nullable instance variables
class User {
  late String username;
  late int age;

  void initialize(String user, int userAge) {
    username = user;
    age = userAge;
  }
}
```

### Important Notes

```dart
// Runtime error if accessed before initialization
late int value;
// print(value); // Error: LateInitializationError

// Use with caution
class Example {
  late String data = computeData();

  String computeData() {
    // This runs only on first access
    return 'computed';
  }
}
```

### Use Cases
- Delaying expensive computations
- Breaking circular dependencies
- Non-nullable variables that can't be initialized in constructor
- Dependency injection frameworks

---

## Null Assertion Operator (!)

### Purpose
The `!` operator tells Dart that a nullable value is definitely not null at that point. It removes nullability from the type.

### Syntax

```dart
String? nullableString = 'Hello';
String nonNullable = nullableString!; // Assert it's not null

int? possiblyNull = getValue();
int definitelyNotNull = possiblyNull!; // Runtime error if null
```

### Use Cases

```dart
// 1. When you know a value isn't null
String? findUser(int id) {
  // ... search logic
  return 'John';
}

void processUser(int id) {
  String user = findUser(id)!; // We know it exists
  print(user.toUpperCase());
}

// 2. After null checking
String? name = getName();
if (name != null) {
  // Still nullable here in some contexts
  print(name!.length); // Assert non-null
}

// 3. With nullable objects
class Container {
  String? value;
}

void process(Container c) {
  if (c.value != null) {
    String val = c.value!;
    print(val);
  }
}

// 4. List/Map access
List<String>? items = getItems();
if (items != null && items.isNotEmpty) {
  String first = items.first!; // Though items.first already non-null
}
```

### Dangers

```dart
// Runtime error if actually null
String? nullValue = null;
// String result = nullValue!; // Error: Null check operator used on a null value

// Better alternatives
String? name = getName();

// Prefer: null-aware operators
String safeName = name ?? 'Unknown';

// Prefer: null-aware method calls
int? length = name?.length;

// Prefer: if-null checks
if (name != null) {
  print(name.toUpperCase()); // Type promotion, no ! needed
}
```

### Best Practices
- Use sparingly, only when absolutely certain value isn't null
- Prefer null-aware operators (??, ?., ...)
- Prefer type promotion through null checks
- Avoid in production code where possible

---

## Arguments

### Purpose
Dart supports multiple parameter types to make functions flexible and easy to use.

### Positional Arguments

```dart
// Required positional arguments
int add(int a, int b) {
  return a + b;
}

add(5, 3); // Must provide both in order
```

### Optional Positional Arguments

```dart
// Square brackets [] for optional positional
String greet(String name, [String? title, int? age]) {
  var greeting = 'Hello';
  if (title != null) greeting += ', $title';
  greeting += ' $name';
  if (age != null) greeting += ' (age $age)';
  return greeting;
}

greet('Alice'); // "Hello Alice"
greet('Bob', 'Dr.'); // "Hello, Dr. Bob"
greet('Charlie', 'Prof.', 45); // "Hello, Prof. Charlie (age 45)"
```

### Named Arguments

```dart
// Curly braces {} for named arguments
void createUser({String? name, int? age, String? email}) {
  print('Name: $name, Age: $age, Email: $email');
}

createUser(name: 'Alice', email: 'alice@example.com'); // Order doesn't matter
createUser(age: 30, name: 'Bob');
```

### Required Named Arguments

```dart
// Use 'required' keyword
void register({
  required String email,
  required String password,
  String? phone
}) {
  print('Email: $email');
  print('Password: $password');
  if (phone != null) print('Phone: $phone');
}

register(email: 'user@example.com', password: 'secret123');
// register(); // Error: Missing required parameters
```

### Default Parameter Values

```dart
// Default values for optional parameters
int power(int base, [int exponent = 2]) {
  var result = 1;
  for (int i = 0; i < exponent; i++) {
    result *= base;
  }
  return result;
}

power(5); // 25 (5^2)
power(5, 3); // 125 (5^3)

// Named parameters with defaults
void connectToServer({
  String host = 'localhost',
  int port = 8080,
  bool secure = false
}) {
  var protocol = secure ? 'https' : 'http';
  print('$protocol://$host:$port');
}

connectToServer(); // "http://localhost:8080"
connectToServer(host: 'api.example.com', secure: true); // "https://api.example.com:8080"
```

### Combining Argument Types

```dart
// Positional + Optional Positional
void example1(int required, [int? optional]) { }

// Positional + Named
void example2(int required, {String? name, int? age}) { }

// All together (positional first, then optional positional OR named)
void example3(int a, int b, [int? c, int? d]) { }
void example4(int a, {required String name, int? age}) { }

// Cannot mix optional positional and named
// void invalid(int a, [int? b], {String? name}) { } // Error!
```

### Real-World Examples

```dart
// Flutter-style widget constructor
class Button {
  final String text;
  final VoidCallback? onPressed;
  final Color? color;
  final double? width;
  final double? height;

  Button({
    required this.text,
    this.onPressed,
    this.color,
    this.width,
    this.height
  });
}

var btn = Button(
  text: 'Click Me',
  onPressed: () => print('Clicked'),
  color: Colors.blue
);

// API call function
Future<Response> apiCall(
  String endpoint, {
  String method = 'GET',
  Map<String, String>? headers,
  dynamic body,
  int timeout = 30
}) async {
  // ... implementation
}

apiCall('/users', method: 'POST', body: {'name': 'Alice'});
```

### Use Cases
- **Positional**: Simple, ordered parameters
- **Optional Positional**: Backwards compatibility, simple optional params
- **Named**: Many parameters, optional parameters, better readability
- **Required Named**: Mandatory params with clear names
- **Default Values**: Sensible defaults, configuration options

---

## Exceptions

### Purpose
Exceptions handle runtime errors and exceptional conditions gracefully.

### Throwing Exceptions

```dart
// Built-in exceptions
throw Exception('Something went wrong');
throw FormatException('Invalid format');
throw ArgumentError('Invalid argument');
throw StateError('Invalid state');

// Custom exceptions
class InvalidAgeException implements Exception {
  final String message;
  InvalidAgeException(this.message);

  @override
  String toString() => 'InvalidAgeException: $message';
}

void setAge(int age) {
  if (age < 0) {
    throw InvalidAgeException('Age cannot be negative');
  }
}

// Throw any object (though exceptions are preferred)
throw 'Error message'; // Valid but not recommended
throw 42; // Valid but not recommended
```

### Catching Exceptions

```dart
// Basic try-catch
try {
  int result = 10 ~/ 0; // Integer division by zero
} catch (e) {
  print('Error occurred: $e');
}

// Catch specific exception types
try {
  var data = jsonDecode(invalidJson);
} on FormatException catch (e) {
  print('Invalid JSON format: $e');
} on Exception catch (e) {
  print('Other exception: $e');
} catch (e) {
  print('Unknown error: $e');
}

// Catch with stack trace
try {
  riskyOperation();
} catch (e, stackTrace) {
  print('Error: $e');
  print('Stack trace: $stackTrace');
}

// Finally block (always executes)
try {
  openFile();
  processFile();
} catch (e) {
  print('Error: $e');
} finally {
  closeFile(); // Always runs
}
```

### Rethrowing Exceptions

```dart
void processData() {
  try {
    loadData();
  } catch (e) {
    print('Error in processData: $e');
    rethrow; // Pass exception to caller
  }
}

try {
  processData();
} catch (e) {
  print('Caught rethrown exception: $e');
}
```

### Common Exception Types

```dart
// FormatException - parsing errors
try {
  int.parse('not a number');
} on FormatException catch (e) {
  print('Parse error: $e');
}

// RangeError - index out of bounds
try {
  var list = [1, 2, 3];
  print(list[5]);
} on RangeError catch (e) {
  print('Index error: $e');
}

// ArgumentError - invalid arguments
void validateEmail(String email) {
  if (!email.contains('@')) {
    throw ArgumentError('Invalid email format');
  }
}

// StateError - invalid state
void withdraw(double amount) {
  if (balance < amount) {
    throw StateError('Insufficient balance');
  }
}

// TimeoutException - operations timeout
import 'dart:async';

try {
  await Future.delayed(Duration(seconds: 10))
      .timeout(Duration(seconds: 5));
} on TimeoutException catch (e) {
  print('Operation timed out');
}
```

### Real-World Example

```dart
class UserRepository {
  Future<User> getUser(int id) async {
    try {
      final response = await http.get('/api/users/$id');

      if (response.statusCode == 404) {
        throw NotFoundException('User not found');
      }

      if (response.statusCode != 200) {
        throw ApiException('Failed to load user');
      }

      return User.fromJson(jsonDecode(response.body));
    } on SocketException {
      throw NetworkException('No internet connection');
    } on FormatException {
      throw DataException('Invalid response format');
    } catch (e) {
      throw UnknownException('Unexpected error: $e');
    }
  }
}

// Usage
try {
  var user = await repository.getUser(123);
  print(user.name);
} on NotFoundException catch (e) {
  showError('User not found');
} on NetworkException catch (e) {
  showError('Please check your internet connection');
} on ApiException catch (e) {
  showError('Server error occurred');
} catch (e) {
  showError('An unexpected error occurred');
}
```

### Use Cases
- Input validation
- Network error handling
- File I/O errors
- Resource management
- User-friendly error messages

---

## Generics

### Purpose
Generics enable writing type-safe, reusable code that works with multiple types while maintaining type checking.

### Generic Classes

```dart
// Generic class
class Box<T> {
  T value;

  Box(this.value);

  T getValue() => value;
  void setValue(T newValue) => value = newValue;
}

// Usage
var intBox = Box<int>(42);
var stringBox = Box<String>('Hello');

intBox.setValue(100); // OK
// intBox.setValue('text'); // Error: wrong type

// Type inference
var autoBox = Box(3.14); // Box<double>
```

### Generic Functions

```dart
// Generic function
T getFirst<T>(List<T> items) {
  return items.first;
}

int first = getFirst<int>([1, 2, 3]); // Explicit type
String firstStr = getFirst(['a', 'b']); // Inferred type

// Multiple type parameters
Map<K, V> createMap<K, V>(K key, V value) {
  return {key: value};
}

var map = createMap('name', 'Alice'); // Map<String, String>
var map2 = createMap(1, true); // Map<int, bool>
```

### Generic Collections

```dart
// List with generics
List<int> numbers = [1, 2, 3];
List<String> names = ['Alice', 'Bob'];

// Map with generics
Map<String, int> scores = {'Alice': 95, 'Bob': 87};
Map<int, User> usersById = {1: user1, 2: user2};

// Set with generics
Set<String> uniqueTags = {'dart', 'flutter', 'mobile'};
```

### Generic Constraints

```dart
// Constrain type parameter
class NumberProcessor<T extends num> {
  T value;

  NumberProcessor(this.value);

  T add(T other) => (value + other) as T;
  T multiply(T other) => (value * other) as T;
}

var intProcessor = NumberProcessor<int>(10);
var doubleProcessor = NumberProcessor<double>(3.14);
// var stringProcessor = NumberProcessor<String>('text'); // Error

// Function with constraints
T max<T extends Comparable>(T a, T b) {
  return a.compareTo(b) > 0 ? a : b;
}

int maxInt = max(5, 10); // 10
String maxStr = max('apple', 'banana'); // 'banana'
```

### Generic Interfaces

```dart
abstract class Cache<T> {
  T? get(String key);
  void set(String key, T value);
  void remove(String key);
}

class MemoryCache<T> implements Cache<T> {
  final Map<String, T> _cache = {};

  @override
  T? get(String key) => _cache[key];

  @override
  void set(String key, T value) => _cache[key] = value;

  @override
  void remove(String key) => _cache.remove(key);
}

// Usage
var userCache = MemoryCache<User>();
userCache.set('user1', User('Alice'));
User? user = userCache.get('user1');
```

### Real-World Examples

```dart
// Result type (for error handling)
class Result<T, E> {
  final T? data;
  final E? error;

  Result.success(this.data) : error = null;
  Result.failure(this.error) : data = null;

  bool get isSuccess => data != null;
  bool get isFailure => error != null;
}

// Usage
Result<User, String> fetchUser(int id) {
  try {
    var user = database.getUser(id);
    return Result.success(user);
  } catch (e) {
    return Result.failure('Failed to fetch user: $e');
  }
}

var result = fetchUser(123);
if (result.isSuccess) {
  print('User: ${result.data!.name}');
} else {
  print('Error: ${result.error}');
}

// Repository pattern
abstract class Repository<T, ID> {
  Future<T?> findById(ID id);
  Future<List<T>> findAll();
  Future<T> save(T entity);
  Future<void> delete(ID id);
}

class UserRepository implements Repository<User, int> {
  @override
  Future<User?> findById(int id) async {
    // Implementation
  }

  @override
  Future<List<User>> findAll() async {
    // Implementation
  }

  @override
  Future<User> save(User entity) async {
    // Implementation
  }

  @override
  Future<void> delete(int id) async {
    // Implementation
  }
}
```

### Use Cases
- Creating reusable collections
- Type-safe APIs
- Generic data structures (Stack, Queue, Tree)
- Repository/DAO patterns
- Result/Option types
- Flutter widget builders

---

## Null Safety

### Purpose
Null safety prevents null reference errors by distinguishing nullable and non-nullable types at compile time.

### Nullable vs Non-Nullable Types

```dart
// Non-nullable (cannot be null)
int age = 25;
String name = 'Alice';
// age = null; // Error
// name = null; // Error

// Nullable (can be null, add ?)
int? nullableAge;
String? nullableName = null;
nullableAge = 30; // OK
nullableAge = null; // OK
```

### Null-Aware Operators

```dart
// 1. Null-aware access (?.)
String? name = getName();
int? length = name?.length; // null if name is null

// Chaining
String? city = user?.address?.city;

// 2. Null-coalescing operator (??)
String displayName = name ?? 'Unknown';
int userAge = age ?? 0;

// 3. Null-aware assignment (??=)
String? username;
username ??= 'guest'; // Assign only if null
print(username); // 'guest'

username ??= 'admin'; // No change, already has value
print(username); // Still 'guest'

// 4. Null assertion (!)
String? maybeString = 'Hello';
String definitelyString = maybeString!; // Assert non-null

// 5. Null-aware spread (...?)
List<int>? nullableList = null;
var list = [...?nullableList]; // Empty list if null

// 6. Null-aware index ([])
List<String>? items = getItems();
String? first = items?[0]; // null if items is null
```

### Type Promotion

```dart
String? name = getName();

// Before check: name is String?
// print(name.length); // Error

if (name != null) {
  // Inside if: name is promoted to String
  print(name.length); // OK
  print(name.toUpperCase()); // OK
}

// Negated check
if (name == null) {
  return;
}
// After early return: name is String
print(name.length);
```

### Late Variables

```dart
// Non-nullable but initialized later
late String description;
late final String config;

void initialize() {
  description = 'Initialized';
  config = 'Config value';
}

// Lazy initialization
late String data = expensiveComputation();
```

### Required Parameters

```dart
class User {
  final String name;
  final int age;

  // All parameters required by default
  User(this.name, this.age);

  // Named parameters with required
  User.create({
    required this.name,
    required this.age
  });
}
```

### Dealing with Nullable Values

```dart
// Example: Processing nullable user input
String? userInput = getUserInput();

// Option 1: Provide default
String input = userInput ?? 'default value';

// Option 2: Check before use
if (userInput != null) {
  process(userInput);
}

// Option 3: Null-aware call
userInput?.trim();

// Option 4: Transform safely
String? trimmed = userInput?.trim();

// Option 5: Assert if certain
String certain = userInput!; // Throws if null

// Working with nullable collections
List<String>? items = getItems();

// Safe length check
int count = items?.length ?? 0;

// Safe iteration
items?.forEach((item) {
  print(item);
});

// Default to empty
List<String> safeItems = items ?? [];
```

### Migrating to Null Safety

```dart
// Before null safety
String name; // Could be null
int age; // Could be null

// After null safety - explicit choice
String name; // Cannot be null
String? maybeName; // Can be null

int age; // Cannot be null
int? maybeAge; // Can be null
```

### Real-World Example

```dart
class UserProfile {
  final String id; // Always required
  final String email; // Always required
  String? phoneNumber; // Optional
  String? avatarUrl; // Optional
  DateTime? lastLogin; // Optional

  UserProfile({
    required this.id,
    required this.email,
    this.phoneNumber,
    this.avatarUrl,
    this.lastLogin
  });

  String getDisplayName() {
    return email.split('@').first; // Safe: email is non-nullable
  }

  bool hasPhone() {
    return phoneNumber != null;
  }

  String getPhoneOrDefault() {
    return phoneNumber ?? 'No phone provided';
  }

  void updateLastLogin() {
    lastLogin = DateTime.now();
  }

  String? getTimeSinceLogin() {
    if (lastLogin == null) return null;

    var difference = DateTime.now().difference(lastLogin!);
    return '${difference.inDays} days ago';
  }
}
```

### Use Cases
- Preventing null pointer exceptions
- Making code more explicit and self-documenting
- Catching errors at compile time instead of runtime
- Improving code reliability

---

## Required Keyword

### Purpose
The `required` keyword marks named parameters as mandatory, ensuring they must be provided at the call site.

### Syntax

```dart
// Without required (all optional)
void createUser({String? name, int? age}) {
  // name and age might be null
}

createUser(); // Valid
createUser(name: 'Alice'); // Valid

// With required
void registerUser({
  required String email,
  required String password,
  String? phone // Still optional
}) {
  // email and password are guaranteed non-null
}

// registerUser(); // Error: Missing required parameters
registerUser(email: 'user@example.com', password: 'secret'); // Valid
registerUser(
  email: 'user@example.com',
  password: 'secret',
  phone: '555-1234'
); // Valid
```

### In Constructors

```dart
class Product {
  final String id;
  final String name;
  final double price;
  final String? description;
  final String? category;

  Product({
    required this.id,
    required this.name,
    required this.price,
    this.description,
    this.category
  });
}

// Must provide required fields
var product = Product(
  id: 'p123',
  name: 'Laptop',
  price: 999.99
);

// Optionally provide others
var detailedProduct = Product(
  id: 'p124',
  name: 'Mouse',
  price: 29.99,
  description: 'Wireless mouse',
  category: 'Accessories'
);
```

### Real-World Examples

```dart
// Flutter-style widget
class Button {
  final String label;
  final VoidCallback onPressed;
  final Color? backgroundColor;
  final double? width;

  Button({
    required this.label,
    required this.onPressed,
    this.backgroundColor,
    this.width
  });
}

var button = Button(
  label: 'Submit',
  onPressed: () => print('Clicked'),
  backgroundColor: Colors.blue
);

// API configuration
class ApiConfig {
  final String baseUrl;
  final String apiKey;
  final int? timeout;
  final Map<String, String>? headers;

  ApiConfig({
    required this.baseUrl,
    required this.apiKey,
    this.timeout,
    this.headers
  });
}

var config = ApiConfig(
  baseUrl: 'https://api.example.com',
  apiKey: 'secret-key-123'
);

// Database connection
Future<void> connectDatabase({
  required String host,
  required int port,
  required String database,
  String? username,
  String? password,
  bool ssl = false
}) async {
  // Connection logic
}

await connectDatabase(
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  username: 'admin',
  ssl: true
);
```

### With Default Values

```dart
// Can combine required with other optional parameters
void configureApp({
  required String appName,
  required String version,
  String environment = 'production', // Optional with default
  bool debugMode = false, // Optional with default
  int? maxConnections // Optional, nullable
}) {
  print('$appName v$version running in $environment');
}

configureApp(
  appName: 'MyApp',
  version: '1.0.0'
); // Uses defaults

configureApp(
  appName: 'MyApp',
  version: '1.0.0',
  environment: 'development',
  debugMode: true
);
```

### Benefits

```dart
// Before required (easy to forget parameters)
void sendEmail({String? to, String? subject, String? body}) {
  if (to == null || subject == null || body == null) {
    throw ArgumentError('Missing required parameters');
  }
  // Send email
}

// After required (compile-time safety)
void sendEmail({
  required String to,
  required String subject,
  required String body
}) {
  // Guaranteed to have all parameters
  // Send email
}

// Compiler catches mistakes
// sendEmail(to: 'user@example.com'); // Error: Missing subject and body
```

### Use Cases
- Ensuring critical parameters are provided
- Making APIs self-documenting
- Preventing runtime errors from missing data
- Improving code clarity and intent
- Mandatory configuration values

---

## Static Keyword

### Purpose
The `static` keyword creates class-level members that belong to the class itself rather than to instances of the class.

### Static Variables

```dart
class Counter {
  static int count = 0; // Shared by all instances
  int id; // Unique to each instance

  Counter() {
    count++; // Increment shared counter
    id = count; // Assign unique ID
  }
}

var c1 = Counter();
var c2 = Counter();
var c3 = Counter();

print(Counter.count); // 3
print(c1.id); // 1
print(c2.id); // 2
print(c3.id); // 3
```

### Static Methods

```dart
class MathUtils {
  // Static method (no instance needed)
  static int add(int a, int b) {
    return a + b;
  }

  static double average(List<int> numbers) {
    if (numbers.isEmpty) return 0;
    return numbers.reduce((a, b) => a + b) / numbers.length;
  }

  // Constants
  static const double pi = 3.14159;
  static const double e = 2.71828;
}

// Call without creating instance
int sum = MathUtils.add(5, 3); // 8
double avg = MathUtils.average([1, 2, 3, 4, 5]); // 3.0
print(MathUtils.pi); // 3.14159
```

### Static Constants

```dart
class AppConstants {
  static const String appName = 'My App';
  static const String version = '1.0.0';
  static const int maxRetries = 3;
  static const Duration timeout = Duration(seconds: 30);

  // Cannot instantiate
  AppConstants._(); // Private constructor prevents instantiation
}

// Usage
print(AppConstants.appName);
print(AppConstants.maxRetries);
```

### Static in Classes

```dart
class DatabaseConnection {
  static DatabaseConnection? _instance;
  String connectionString;

  // Private constructor
  DatabaseConnection._(this.connectionString);

  // Static factory method (Singleton pattern)
  static DatabaseConnection getInstance(String connStr) {
    _instance ??= DatabaseConnection._(connStr);
    return _instance!;
  }

  static void resetConnection() {
    _instance = null;
  }
}

// Always returns same instance
var db1 = DatabaseConnection.getInstance('server1');
var db2 = DatabaseConnection.getInstance('server2');
print(identical(db1, db2)); // true
```

### Real-World Examples

```dart
// 1. Configuration class
class Config {
  static const String apiUrl = 'https://api.example.com';
  static const String apiKey = 'your-api-key';
  static const int timeout = 30;

  static Map<String, String> getHeaders() {
    return {
      'Authorization': 'Bearer $apiKey',
      'Content-Type': 'application/json'
    };
  }

  Config._();
}

// 2. Utility class
class StringUtils {
  static bool isValidEmail(String email) {
    return email.contains('@') && email.contains('.');
  }

  static String capitalize(String text) {
    if (text.isEmpty) return text;
    return text[0].toUpperCase() + text.substring(1);
  }

  static String truncate(String text, int maxLength) {
    if (text.length <= maxLength) return text;
    return '${text.substring(0, maxLength)}...';
  }

  StringUtils._();
}

// Usage
if (StringUtils.isValidEmail('user@example.com')) {
  print('Valid email');
}

// 3. Logger class
class Logger {
  static bool _debugMode = false;
  static List<String> _logs = [];

  static void setDebugMode(bool enabled) {
    _debugMode = enabled;
  }

  static void log(String message) {
    var timestamp = DateTime.now().toString();
    var logEntry = '[$timestamp] $message';
    _logs.add(logEntry);
    if (_debugMode) {
      print(logEntry);
    }
  }

  static List<String> getLogs() {
    return List.unmodifiable(_logs);
  }

  static void clearLogs() {
    _logs.clear();
  }
}

// Usage
Logger.setDebugMode(true);
Logger.log('Application started');
Logger.log('User logged in');

// 4. Factory tracking
class User {
  static int _nextId = 1;
  static final Map<int, User> _cache = {};

  final int id;
  final String name;

  User._(this.id, this.name);

  factory User(String name) {
    var id = _nextId++;
    var user = User._(id, name);
    _cache[id] = user;
    return user;
  }

  static User? findById(int id) {
    return _cache[id];
  }

  static int get totalUsers => _cache.length;
}
```

### Static vs Instance

```dart
class Example {
  static int staticVar = 0; // One per class
  int instanceVar = 0; // One per instance

  static void staticMethod() {
    print(staticVar); // OK
    // print(instanceVar); // Error: Can't access instance members
  }

  void instanceMethod() {
    print(staticVar); // OK
    print(instanceVar); // OK
    staticMethod(); // OK
  }
}

// Access static without instance
Example.staticMethod();
print(Example.staticVar);

// Access instance members through instance
var example = Example();
example.instanceMethod();
print(example.instanceVar);
```

### Use Cases
- Utility functions (math, string manipulation, validation)
- Configuration constants
- Singleton patterns
- Factory methods
- Counters and global state (use sparingly)
- Helper classes
- Logging and debugging

---

## Ternary Operator

### Purpose
The ternary operator provides a concise way to write simple if-else statements in a single expression.

### Syntax

```dart
// Basic syntax
// condition ? valueIfTrue : valueIfFalse

int age = 20;
String status = age >= 18 ? 'Adult' : 'Minor';
print(status); // "Adult"

// Equivalent to:
String status;
if (age >= 18) {
  status = 'Adult';
} else {
  status = 'Minor';
}
```

### Common Use Cases

```dart
// 1. Variable assignment
int score = 85;
String grade = score >= 90 ? 'A' :
               score >= 80 ? 'B' :
               score >= 70 ? 'C' :
               score >= 60 ? 'D' : 'F';

// 2. Return values
int max(int a, int b) => a > b ? a : b;
int min(int a, int b) => a < b ? a : b;

// 3. Display text
int itemCount = 5;
String message = itemCount == 1 ? '1 item' : '$itemCount items';

// 4. Default values
String? username = getUsername();
String display = username != null ? username : 'Guest';

// Better with null coalescing
String display2 = username ?? 'Guest';

// 5. Function calls
bool isLoggedIn = true;
isLoggedIn ? logout() : login();

// 6. List/Map values
var users = ['Alice', 'Bob'];
String firstUser = users.isNotEmpty ? users.first : 'No users';

// 7. Widget properties (Flutter)
Container(
  color: isDarkMode ? Colors.black : Colors.white,
  child: Text(
    'Hello',
    style: TextStyle(
      color: isDarkMode ? Colors.white : Colors.black
    )
  )
)
```

### Nested Ternary

```dart
// Can be nested but can become hard to read
int temperature = 25;
String description = temperature > 30 ? 'Hot' :
                    temperature > 20 ? 'Warm' :
                    temperature > 10 ? 'Cool' : 'Cold';

// Better alternative for complex conditions
String getTemperatureDescription(int temp) {
  if (temp > 30) return 'Hot';
  if (temp > 20) return 'Warm';
  if (temp > 10) return 'Cool';
  return 'Cold';
}
```

### With Null Safety

```dart
String? nullableString = getValue();

// Ternary with null check
String result = nullableString != null ? nullableString.toUpperCase() : 'EMPTY';

// Better with null-aware operators
String result2 = nullableString?.toUpperCase() ?? 'EMPTY';

// Complex example
int? age = getAge();
String message = age != null
    ? (age >= 18 ? 'Adult' : 'Minor')
    : 'Age unknown';
```

### Real-World Examples

```dart
// 1. API response handling
String getStatusMessage(int statusCode) {
  return statusCode == 200 ? 'Success' :
         statusCode == 404 ? 'Not Found' :
         statusCode == 500 ? 'Server Error' :
         'Unknown Error';
}

// 2. User permissions
bool canEdit = isOwner ? true : isAdmin ? true : false;
// Simplified
bool canEdit = isOwner || isAdmin;

// 3. Formatting
String formatPrice(double price) {
  return price == 0 ? 'Free' : '\$${price.toStringAsFixed(2)}';
}

// 4. Validation messages
String validateEmail(String email) {
  return email.isEmpty ? 'Email is required' :
         !email.contains('@') ? 'Invalid email format' :
         'Valid';
}

// 5. Icon selection (Flutter)
Icon getStatusIcon(String status) {
  return Icon(
    status == 'success' ? Icons.check_circle :
    status == 'error' ? Icons.error :
    status == 'warning' ? Icons.warning :
    Icons.info
  );
}

// 6. List filtering display
List<String> items = getItems();
Widget buildList() {
  return items.isEmpty
      ? Text('No items found')
      : ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) => Text(items[index])
        );
}
```

### Best Practices

```dart
// Good: Simple, readable
String status = isActive ? 'On' : 'Off';

// Good: Clear variable names
String userType = isPremium ? 'Premium User' : 'Free User';

// Acceptable: Short nested ternary
String size = width > 1200 ? 'large' :
              width > 800 ? 'medium' : 'small';

// Bad: Too complex, hard to read
var result = condition1 ? (condition2 ? value1 : (condition3 ? value2 : value3)) :
             (condition4 ? value4 : value5);

// Better: Use if-else or switch
var result;
if (condition1 && condition2) {
  result = value1;
} else if (condition1 && condition3) {
  result = value2;
} else if (condition4) {
  result = value4;
} else {
  result = value5;
}
```

### Ternary vs Alternatives

```dart
// Ternary
String message = count == 1 ? 'item' : 'items';

// If-else
String message;
if (count == 1) {
  message = 'item';
} else {
  message = 'items';
}

// Null coalescing (for null checks)
String name = username ?? 'Guest';

// If-null operator (for null default)
username ??= 'Guest';
```

### Use Cases
- Simple conditional assignments
- Short conditional logic
- Default values
- Formatting and display strings
- Widget properties in Flutter
- Return values in short functions

---

## Summary

This cheatsheet covers all fundamental Dart concepts:
- **Data Types**: Core types for storing different kinds of data
- **Variables & Constants**: Mutable and immutable data storage
- **Collections**: Lists, Maps, Sets, and other data structures
- **Functions**: Reusable code blocks with various parameter types
- **OOP**: Classes, inheritance, mixins, and abstraction
- **Late**: Delayed initialization for non-nullable variables
- **Null Assertion (!)**: Forcing nullable to non-nullable (use carefully)
- **Arguments**: Flexible parameter passing (positional, named, optional, required)
- **Exceptions**: Error handling and recovery
- **Generics**: Type-safe reusable code
- **Null Safety**: Preventing null reference errors
- **Required**: Mandatory named parameters
- **Static**: Class-level members and methods
- **Ternary**: Concise conditional expressions

Each section includes purpose, syntax, examples, and real-world use cases to help you master Dart programming.

# Singleton Design Pattern

## Definition

Singleton is a **Creational Design Pattern** that ensures:

1. Only one instance of a class exists.
2. A global access point is provided to access that instance.

---

# Why Do We Need Singleton?

Some resources should be shared across the entire application rather than creating multiple objects.

### Examples

* Logger
* Configuration Manager
* Cache Manager
* Database Connection Pool Manager
* Application Context
* Feature Flag Manager

If every class creates its own instance, it may lead to:

* Increased memory usage
* Inconsistent state
* Resource contention
* Unnecessary object creation

---

# Core Idea

A Singleton class:

1. Prevents external object creation.
2. Creates exactly one object.
3. Returns the same object every time.

---

# Why is Constructor Private?

```java
public class Singleton {

    private Singleton() {

    }
}
```

If the constructor were public:

```java
Singleton s1 = new Singleton();
Singleton s2 = new Singleton();
```

Multiple objects could be created.

Making the constructor private prevents object creation from outside the class.

---

# Why is Instance Variable Static?

```java
private static Singleton instance;
```

`static` means it belongs to the class itself.

Without static:

```java
private Singleton instance;
```

Every object would have its own copy of `instance`, defeating the purpose.

Since the Singleton instance must exist independently of any object, it must be static.

---

# Why is getInstance() Static?

```java
public static Singleton getInstance()
```

If it were not static:

```java
public Singleton getInstance()
```

You would first need an object to call it:

```java
Singleton s = new Singleton();
```

But object creation is blocked by the private constructor.

Therefore `getInstance()` must be static so it can be called directly:

```java
Singleton s = Singleton.getInstance();
```

---

# Eager Initialization Singleton

Instance is created when the class loads.

```java
public final class Singleton {

    private static final Singleton INSTANCE =
            new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### Advantages

* Simple
* Thread-safe

### Disadvantages

* Object created even if never used

---

# Lazy Initialization Singleton

Object is created only when needed.

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {

        if(instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

### Advantages

* Memory efficient

### Disadvantages

* Not thread-safe

---

# Problem in Multi-threaded Environment

Consider:

```java
if(instance == null) {
    instance = new Singleton();
}
```

### Step 1

Thread A checks:

```java
instance == null
```

Result:

```text
true
```

---

### Step 2

Before creating the object, Thread A pauses.

---

### Step 3

Thread B checks:

```java
instance == null
```

Still true.

Thread B creates **Object #1**.

---

### Step 4

Thread A resumes and creates **Object #2**.

### Result

Two Singleton objects are created.

Singleton guarantee is broken.

---

# Solution 1: Synchronized Method

```java
public static synchronized Singleton getInstance() {

    if(instance == null) {
        instance = new Singleton();
    }

    return instance;
}
```

## How it Works

* Only one thread can execute this method at a time.
* Thread A enters.
* Thread B waits.
* Thread A creates the instance and exits.
* Thread B enters and sees that the instance already exists.

Only one object is created.

---

# Why is synchronized Slow?

Consider:

```java
Singleton.getInstance();
Singleton.getInstance();
Singleton.getInstance();
```

After the first call:

```java
instance != null
```

No creation is needed anymore.

However, every call still acquires and releases the lock.

This creates unnecessary synchronization overhead.

The application remains correct but may lose performance under heavy concurrency.

---

# Understanding synchronized Block

### Syntax

```java
synchronized(lockObject) {

}
```

### Example

```java
synchronized(Singleton.class) {

}
```

### Meaning

> Only one thread can execute this block while holding the lock on `Singleton.class`.

Think of it as a room with one key.

Only one thread can hold the key and enter at a time.

---

# Solution 2: Double Checked Locking

```java
public class Singleton {

    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {

        if(instance == null) {

            synchronized(Singleton.class) {

                if(instance == null) {

                    instance = new Singleton();

                }
            }
        }

        return instance;
    }
}
```

---

# Why Check Twice?

### First Check

```java
if(instance == null)
```

Avoids synchronization after the instance is created.

### Second Check

```java
if(instance == null)
```

Prevents multiple threads from creating objects after entering the synchronized block.

This combination is called **Double Checked Locking**.

---

# Why volatile is Required?

```java
private static volatile Singleton instance;
```

`volatile` prevents instruction reordering.

Without `volatile`, another thread may see a partially constructed object.

### Interview Point

> Double Checked Locking requires `volatile` to work correctly.

---

# Modern Best Practice: Enum Singleton

```java
public enum Singleton {

    INSTANCE;

}
```

### Usage

```java
Singleton singleton = Singleton.INSTANCE;
```

### Advantages

* Thread-safe
* Reflection-safe
* Serialization-safe
* Simplest implementation

This is generally considered the safest Singleton implementation in Java.

---

# Advantages of Singleton

* Controlled access to a single instance
* Saves memory
* Shared state across the application
* Avoids repeated object creation
* Centralized resource management

---

# Disadvantages of Singleton

* Global state can make testing difficult
* Hidden dependencies
* Can violate Single Responsibility Principle
* Difficult to mock in unit tests
* Overuse leads to tightly coupled code

---

# Real-World Use Cases

## Logger

```java
Logger.getInstance().log("Application Started");
```

Only one logger should manage logs.

---

## Configuration Manager

```java
ConfigurationManager.getInstance()
                    .getProperty("db.url");
```

Configuration should be loaded once.

---

## Cache Manager

```java
CacheManager.getInstance()
            .put("user1", user);
```

Shared cache across the application.

---

## Database Connection Pool Manager

```java
ConnectionPool.getInstance()
              .getConnection();
```

One pool manages all connections.

---

## Feature Flag Manager

```java
FeatureManager.getInstance()
              .isEnabled("NEW_UI");
```

Single source of truth for feature toggles.

---

# Interview Summary

Singleton ensures:

1. Only one object exists.
2. Global access to that object.

## Implementation Steps

1. Private constructor
2. Static instance variable
3. Static `getInstance()` method

## Thread-safe Approaches

1. Eager Initialization
2. Synchronized Method
3. Double Checked Locking + `volatile`
4. Enum Singleton (**Recommended**)

## Most Common Interview Answer

> Singleton is a Creational Design Pattern that restricts object creation to a single instance and provides a global access point to that instance. It is implemented using a private constructor, a static instance variable, and a static `getInstance()` method.

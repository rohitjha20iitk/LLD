# Builder Design Pattern - Complete Revision Notes

# 1. Definition

Builder is a **Creational Design Pattern** that allows complex objects to be constructed step by step.

It helps:

1. Avoid telescoping constructors.
2. Handle optional parameters cleanly.
3. Improve readability of object creation.
4. Create immutable objects.
5. Reuse the same construction process to create different representations of a product (GoF Builder).

---

# 2. What Problem Does Builder Solve?

Suppose we have:

```java
public class Employee {

    private String name;
    private int age;
    private String department;
    private String address;
    private String phoneNumber;

    public Employee(
            String name,
            int age,
            String department,
            String address,
            String phoneNumber) {

        this.name = name;
        this.age = age;
        this.department = department;
        this.address = address;
        this.phoneNumber = phoneNumber;
    }
}
```

Creating an object:

```java
Employee employee =
    new Employee(
        "Rohit",
        28,
        "Engineering",
        "Bangalore",
        "9999999999");
```

This is manageable.

But imagine:

```java
Employee(
    String name,
    int age,
    String department,
    String address,
    String phoneNumber,
    String manager,
    String project,
    String country,
    boolean permanentEmployee,
    String bloodGroup
)
```

Now object creation becomes difficult to read and maintain.

---

# 3. Problems with Large Constructors

## 3.1 Problem 1: Poor Readability

```java
new Employee(
    "Rohit",
    28,
    "Engineering",
    "Bangalore",
    "9999999999",
    "Rajesh",
    "ProjectA",
    "India",
    true,
    "B+"
);
```

It is difficult to understand what each value represents.

---

## 3.2 Problem 2: Optional Fields

Suppose only:

```text
name
age
department
```

are mandatory.

Then we end up doing:

```java
new Employee(
    "Rohit",
    28,
    "Engineering",
    null,
    null,
    null,
    null,
    null,
    false,
    null
);
```

This is ugly and error-prone.

---

## 3.3 Problem 3: Telescoping Constructors

Trying to solve the problem using overloaded constructors:

```java
Employee(String name)

Employee(String name, int age)

Employee(String name, int age, String department)

Employee(String name, int age, String department, String address)
```

Eventually leads to many constructors.

This is called the **Telescoping Constructor Problem**.

---

# 4. Modern Java Builder Pattern

Instead of:

```java
new Employee(
    "Rohit",
    28,
    "Engineering",
    "Bangalore"
);
```

we write:

```java
Employee employee =
    new EmployeeBuilder()
        .name("Rohit")
        .age(28)
        .department("Engineering")
        .address("Bangalore")
        .build();
```

This reads almost like English.

---

# 5. Builder Class

```java
public class EmployeeBuilder {

    private String name;
    private int age;
    private String department;
    private String address;

    public EmployeeBuilder name(String name) {
        this.name = name;
        return this;
    }

    public EmployeeBuilder age(int age) {
        this.age = age;
        return this;
    }

    public EmployeeBuilder department(String department) {
        this.department = department;
        return this;
    }

    public EmployeeBuilder address(String address) {
        this.address = address;
        return this;
    }

    public Employee build() {
        return new Employee(
                name,
                age,
                department,
                address);
    }
}
```

---

# 6. Why Don't Builder Methods Return void?

Example:

```java
public EmployeeBuilder name(String name) {
    this.name = name;
    return this;
}
```

Builder methods return the Builder object itself.

This enables:

```java
new EmployeeBuilder()
    .name("Rohit")
    .age(28)
    .department("Engineering");
```

This style is called:

* Method Chaining
* Fluent API

---

# 7. What Does 'this' Mean?

Inside:

```java
public EmployeeBuilder name(String name) {
    this.name = name;
    return this;
}
```

`this` refers to the current Builder object.

It does NOT create a new Builder.

It returns the same Builder object.

Example:

```java
EmployeeBuilder builder = new EmployeeBuilder();

EmployeeBuilder b1 =
        builder.name("Rohit");

EmployeeBuilder b2 =
        builder.age(28);
```

Result:

```java
builder == b1   // true
builder == b2   // true
b1 == b2        // true
```

All references point to the same Builder object.

---

# 8. What Does build() Do?

Before:

```java
new EmployeeBuilder()
    .name("Rohit")
    .age(28);
```

Only a Builder object exists.

No Employee object exists.

When:

```java
.build()
```

is called:

```java
public Employee build() {

    return new Employee(
        name,
        age,
        department,
        address
    );
}
```

the actual Employee object gets created.

---

# 9. Builder vs Setters

Using setters:

```java
Employee employee = new Employee();

employee.setName("Rohit");
employee.setAge(28);
```

Problems:

1. Object can exist in an incomplete state.
2. Mandatory fields can be forgotten.
3. Difficult to create immutable objects.

Builder solves these problems.

---

# 10. Builder and Immutable Objects

Immutable object:

```java
public class Employee {

    private final String name;
    private final int age;

    public Employee(
            String name,
            int age) {

        this.name = name;
        this.age = age;
    }
}
```

No setters.

Once created, the object cannot be modified.

Builder collects values first and then creates the immutable object.

---

# 11. Nested Builder (Most Common in Real Java)

```java
Employee employee =
    Employee.builder()
            .name("Rohit")
            .age(28)
            .build();
```

Instead of:

```java
new EmployeeBuilder()
```

the Builder is placed inside the Employee class.

---

# 12. Why builder() is Static?

```java
public static Builder builder() {
    return new Builder();
}
```

Static allows:

```java
Employee.builder();
```

without creating an Employee first.

Exactly like Singleton's `getInstance()` is static because no object exists yet.

---

# 13. Builder is NOT a Singleton

```java
Builder b1 = Employee.builder();
Builder b2 = Employee.builder();
```

Result:

```java
b1 == b2
```

false

Because:

```java
return new Builder();
```

creates a new Builder every time.

---

# 14. Why Builder Should Not Be Shared

Suppose Builder was shared:

```java
b1.name("Rohit");
b2.name("Amit");
```

Builder state becomes:

```text
name = Amit
```

The last update wins.

State gets overwritten.

Each build process must get its own Builder object.

---

# 15. Passing Builder to Constructor

Common implementation:

```java
public static class Builder {

    private String name;
    private int age;

    public Employee build() {
        return new Employee(this);
    }
}
```

Constructor:

```java
private Employee(Builder builder) {

    this.name = builder.name;
    this.age = builder.age;
}
```

Advantages:

1. Constructor signature remains simple.
2. Easier maintenance.
3. Works well when class has many fields.
4. Adding fields does not require changing constructor parameters.

---

# 16. Validation Inside build()

```java
public Employee build() {

    if(name == null) {
        throw new IllegalStateException(
            "Name is required");
    }

    if(age <= 0) {
        throw new IllegalStateException(
            "Age must be positive");
    }

    return new Employee(this);
}
```

Validation happens when:

```java
.build()
```

is called.

Not before.

---

# 17. Why Validation Belongs in build()

Builder is allowed to be incomplete.

Example:

```java
Employee.builder()
        .name("Rohit");
```

No validation yet.

Only when:

```java
.build()
```

is called do we attempt to create the final object.

This is the perfect place to validate.

---

# 18. Lombok @Builder

Code:

```java
@Builder
public class Employee {

    private String name;
    private int age;
}
```

Usage:

```java
Employee employee =
    Employee.builder()
            .name("Rohit")
            .age(28)
            .build();
```

Lombok automatically generates:

* Builder class
* builder() method
* build() method
* Method chaining methods

Lombok uses the same Builder pattern internally.

---

# 19. Modern Java Builder Summary

Mental Model:

```text
Many Constructor Parameters
            ↓
         Builder
            ↓
Readable + Immutable Object
```

---

# 20. Interview Definition (Modern Java Builder)

Builder is a creational design pattern that allows complex objects to be constructed step by step. It improves readability, avoids telescoping constructors, handles optional parameters cleanly, and is commonly used to create immutable objects.

---

# 21. GoF / Refactoring Guru Builder

The original Builder pattern solves a different problem.

Modern Java Builder:

```text
One Builder
One Product
```

Example:

```text
EmployeeBuilder -> Employee
```

---

GoF Builder:

```text
Same Construction Process
Different Products
```

Example:

```text
CarBuilder -> Car

CarManualBuilder -> Manual
```

---

# 22. Easy Example - Report Generation

We want:

```text
PDF Report
HTML Report
Excel Report
```

Construction steps:

```text
Add Title
Add Summary
Add Charts
Add Footer
```

Same steps.

Different outputs.

---

# 23. Builder Interface

```java
public interface ReportBuilder {

    void addTitle();

    void addSummary();

    void addCharts();

    void addFooter();
}
```

Defines common construction steps.

---

# 24. Concrete Builders

PDF Builder:

```java
public class PdfReportBuilder
        implements ReportBuilder {

}
```

Creates:

```text
PDF Report
```

---

HTML Builder:

```java
public class HtmlReportBuilder
        implements ReportBuilder {

}
```

Creates:

```text
HTML Report
```

---

Excel Builder:

```java
public class ExcelReportBuilder
        implements ReportBuilder {

}
```

Creates:

```text
Excel Report
```

---

# 25. Why Use an Interface?

Allows code like:

```java
public void createStandardReport(
        ReportBuilder builder) {

    builder.addTitle();
    builder.addSummary();
    builder.addCharts();
    builder.addFooter();
}
```

The method works for all Builder implementations.

---

# 26. Open/Closed Principle

Suppose we add:

```java
ExcelReportBuilder
```

We do NOT modify:

```java
createStandardReport()
```

because it already works with:

```java
ReportBuilder
```

This follows:

```text
Open for Extension
Closed for Modification
```

---

# 27. Director

Director stores construction recipes.

Example:

```java
public class ReportDirector {

    public void buildStandardReport(
            ReportBuilder builder) {

        builder.addTitle();
        builder.addSummary();
        builder.addCharts();
        builder.addFooter();
    }
}
```

---

# 28. Director's Responsibility

Builder knows:

```text
HOW
```

Director knows:

```text
IN WHAT ORDER
```

Builder:

```text
How to add title
How to add charts
How to add summary
```

Director:

```text
Which sequence to execute
```

---

# 29. Cooking Analogy

Builder = Chef

```text
Can chop vegetables
Can fry paneer
Can cook rice
```

Director = Recipe

```text
Step 1
Step 2
Step 3
```

Chef knows HOW.

Recipe knows ORDER.

---

# 30. Director Does Not Create Products

Wrong:

```text
Director -> Product
```

Correct:

```text
Director -> Builder -> Product
```

Director only calls construction steps.

Builder creates the product.

---

# 31. Car Example from Refactoring Guru

Builder Interface:

```java
setSeats()

setEngine()

setGPS()
```

---

CarBuilder:

```text
Installs actual parts
```

Produces:

```text
Car
```

---

CarManualBuilder:

```text
Writes documentation
```

Produces:

```text
Manual
```

---

Director:

```java
buildSportsCar()
```

contains:

```java
builder.setSeats(2);
builder.setEngine(SPORT);
builder.setGPS(true);
```

---

Execution:

```java
director.buildSportsCar(
        new CarBuilder());
```

Produces:

```text
Sports Car
```

---

Execution:

```java
director.buildSportsCar(
        new CarManualBuilder());
```

Produces:

```text
Sports Car Manual
```

---

# 32. Why Builder Doesn't Expose Unfinished Product

Bad design:

```java
builder.getCar();
```

before construction completes.

Could return:

```text
Seats = 2
Engine = null
GPS = null
```

A partially built object.

---

Good design:

Builder hides the product until:

```java
builder.getProduct();
```

or

```java
.build();
```

is called.

This guarantees clients receive a fully constructed object.

---

# 33. Why Is This Good?

It prevents:

```text
Partially Constructed Objects
```

and guarantees:

```text
Finished Product Only
```

---

# 34. Modern Java Builder vs GoF Builder

## Modern Java Builder

Problem:

```text
Too many constructor parameters
Optional fields
Immutable objects
Readable object creation
```

Example:

```java
Employee.builder()
        .name("Rohit")
        .age(28)
        .build();
```

One Builder → One Product

---

## GoF Builder

Problem:

```text
Same construction process
Different product representations
```

Example:

```java
director.buildSportsCar(
        new CarBuilder());

director.buildSportsCar(
        new CarManualBuilder());
```

Same Steps → Different Products

---

# 35. Builder vs Singleton

## Singleton

Problem:

```text
Ensure only one object exists
```

Solution:

```java
Singleton.getInstance();
```

Returns the same object every time.

---

## Builder

Problem:

```text
Create complex objects cleanly
```

Solution:

```java
Employee.builder()
        .name("Rohit")
        .build();
```

Creates a new object when build() is called.

---

# 36. Most Common Interview Questions

## Q1. Why Builder?

Answer:

```text
Avoid telescoping constructors and improve readability.
```

---

## Q2. Why return this?

Answer:

```text
To enable method chaining.
```

---

## Q3. What does build() do?

Answer:

```text
Creates the final object.
```

---

## Q4. Why validate in build()?

Answer:

```text
Builder is allowed to be incomplete.
Validation should happen before creating the final object.
```

---

## Q5. Why builder() is static?

Answer:

```text
No Employee object exists yet.
We need a way to obtain a Builder first.
```

---

## Q6. Why is Builder often used with immutable objects?

Answer:

```text
Builder gathers all values first.
The final object is created once and cannot be modified.
```

---

## Q7. What is the difference between Modern Java Builder and GoF Builder?

Answer:

```text
Modern Builder:
One Builder -> One Product

GoF Builder:
Same Construction Process -> Different Products
```

---

# 37. Final One-Page Revision

## Modern Java Builder

```text
Too Many Parameters
        ↓
     Builder
        ↓
Method Chaining
        ↓
Validation
        ↓
Immutable Object
```

---

## GoF Builder

```text
Builder Interface
        ↓
Concrete Builders
        ↓
Director
        ↓
Same Steps
        ↓
Different Products
```

---

# Most Common Interview Answer

Builder is a Creational Design Pattern that constructs complex objects step by step. In modern Java it is mainly used to avoid telescoping constructors, improve readability, handle optional parameters, and create immutable objects. In the original GoF pattern, the same construction process can be reused to create different representations of a product through different Builder implementations.

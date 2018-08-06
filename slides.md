<!-- .slide: data-background="#5D6FA5" -->
<!-- .slide: data-state="terminal" -->
# Kotlin: a better Java

Semyon Samusev |  Zeptolab
<br>

----------

# What is Kotlin?

- JVM language <!-- .element: class="fragment" -->
- from Jetbrains <!-- .element: class="fragment" -->
- inspired by "Effective Java" <!-- .element: class="fragment" -->
- open sourced in 2011 (release 1.0 in feb 2015) <!-- .element: class="fragment" -->
- 100% java interop, costs nothing to adopt <!-- .element: class="fragment" -->
- in 2017 became first class citizen lang on Android <!-- .element: class="fragment" -->

----------

# What does it add?

1. Conciseness
2. Null safety
3. Lambdas and higher order functions
4. Extensions
5. Smart casts

---------v

## Find more at:
<a href="https://kotlinlang.org/docs/reference/comparison-to-java.html">https://kotlinlang.org/docs/reference/comparison-to-java.html</a>

----------

# 1. Conciseness

Kotlin:
```kotlin
data class Person (val firstName: String,
                   val lastName: String = "undefined",
                   var age: Int = 7)
```

---------v
Java:
```
public class Person {
    private final String firstName;
    private final String lastName;
    private int age;

    public Person(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    public Person(String firstName, String lastName) {
        this(firstName, lastName, 7);
    }

    public Person(String firstName) {
        this(firstName, "undefined");
    }

    public String getFirstName() { return firstName; }

    public String getLastName() { return lastName; }

    public int getAge() { return age; }

    public void setAge(int age) { this.age = age; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age &&
                Objects.equals(firstName, person.firstName) &&
                Objects.equals(lastName, person.lastName);
    }

    @Override
    public int hashCode() {
        return Objects.hash(firstName, lastName, age);
    }
}
```

----------

# 2. Null safety

- Nulls are part of the type system
- Types default to non-nullable
- Mark nullable with the addition of a `?`

---------v

# 2. Null safety

```kotlin
var nonNullType: Type
nonNullType = null // compilation error
```
<!-- .element: class="fragment" -->
` `
```kotlin
var nullableType: Type?
nullableType = null // ok
```
<!-- .element: class="fragment" -->

---------v

# 2. Null safety
```kotlin
var a: String = "abc"
print(a.length) // ok
```
<!-- .element: class="fragment" -->
` `
```kotlin
var b: String? = "abc"
print(b.length) // compilation error
```
<!-- .element: class="fragment" -->

----------

# Null safety helpers

```kotlin
var b: String? = "abc"
```
- <span>Explicit null checks</span>
- <span>Safe calls: `?` </span>
- <span>Elvis operator: `?:`</span>
- <span>Not-null assertion operator `!!`</span>

---------v

# Null safety helpers
```kotlin
var b: String? = "abc"
```
- ```kotlin
val l: Int = if (b != null) b.length else -1
```
- ```kotlin
val l: Int? = b?.length
```
- ```kotlin
val l: Int = b?.length ?: -1
```
- ```kotlin
val l: Int = b!!.length
```

----------
# 3. Higher order functions

- functions are first-class <!-- .element: class="fragment" -->
- <div>function type syntax: `(Int, String) -> List<String>`</div> <!-- .element: class="fragment" -->
- <div>`typealias ClickHandler = (Button, Event) -> Unit`</div> <!-- .element: class="fragment" -->
- could be inlined <!-- .element: class="fragment" -->

---------v
# 3. Higher order functions
```kotlin
fun <T, R> fold(
        col: Collection<T>,
        initial: R,
        combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in col) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

---------v
# 3. Lambdas
- <div>syntax: `val sum = { x: Int, y: Int -> x + y }`</div> <!-- .element: class="fragment" -->
- <div>`val product = fold(items, 1) { acc, e -> acc * e }`</div> <!-- .element: class="fragment" -->
- <div>`it`: implicit first parameter `ints.filter {it > 0}`</div> <!-- .element: class="fragment" -->
- <div>closures (clojure scope is less restrictive than java)</div> <!-- .element: class="fragment" -->


---------v
# 3. Inline
```kotlin
fun <T> withLock(lock: Lock, body: () -> T): T {
  lock.lock()
  try {
    return body()
  }
  finally {
    lock.unlock()
  }
}
```
```kotlin
withLock(lock) { foo() }
```
<!-- .element: class="fragment" -->
```kotlin
inline fun <T> withLock(lock: Lock, body: () -> T): T { ... }
```
<!-- .element: class="fragment" -->


----------
# 4. Extensions
### Motivation
<blockquote class="stretch">
In Java, we are used to classes named "*Utils": FileUtils, StringUtils and so on.
The famous java.util.Collections belongs to the same breed.
The code that uses them looks like this:
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list))
</blockquote>

----------
# 4. Extensions

- attaches to type (receiver)
- resolves statically
- has no access to private members
- could have nullable receiver
- <span>base for a huge part of "standard library"</span> <!-- .element: class="fragment highlight-blue" -->

---------v
# 4. Extensions
### example (our lock)
```kotlin
inline fun <T> Lock.withLock(body: () -> T): T {
  lock()
  try {
    return body()
  }
  finally {
    unlock()
  }
}
```
```kotlin
val l = ReentrantLock()
val result = l.withLock { someAction() }
```
<!-- .element: class="fragment" -->

---------v
# 4. Extensions
### example (from standard library)
```kotlin
public fun <T> Iterable<T>.forEach(action: (T) -> Unit) {
    for (element in this) action(element)
}
```

---------v
# 4. Extensions
### example (from standard library)
```kotlin
public inline fun <S, T : S> Iterable<T>.reduce(
        operation: (acc: S, T) -> S
): S {
    val iterator = this.iterator()
    if (!iterator.hasNext())
        throw UnsupportedOperationException("Empty collection can't be reduced.")
    var accumulator: S = iterator.next()
    while (iterator.hasNext()) {
        accumulator = operation(accumulator, iterator.next())
    }
    return accumulator
}

```

---------v
# 4. Extensions
### Nullable receiver
```kotlin
fun Any?.toString(): String {
    if (this == null) {
        return "null"
    } else {
        return this.toString()
    }
}
```
```kotlin
fun Any?.toString() = this?.toString() ?: "null"
```
<!-- .element: class="fragment" -->

----------
# 5. Smart casts
### and static typing

*Shared*: static + strong typing

*Different*: compiler tracked smart casting

---------v
# 5. Smart casts
```java
// java
JsonNode node = input.get("analytics");
if (node instanceof ObjectNode) {
    ((ObjectNode) node).put("counter", 1);
}
```
` `
```kotlin
// kotlin
val node: JsonNode = input.get("analytics")
if (node is ObjectNode) {
    node.put("counter", 1)
}
```
<!-- .element: class="fragment" -->

---------v
# 5. Smart casts
### to non-nullable type
```kotlin
val someVal: String? = foo() // type is String?

...

if (someVal != null) {
    print(someVal.length)    // type is String
}

```


---------v
# 5. Smart casts
### mutability protection
```kotlin
var globalVar: Type? = foo()

...

if (globalVar != null) {
    globalVar.doSmth()      // compilation error
}

```

----------
# that's all folks
## questions?

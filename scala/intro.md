# Introduction to Scala

From https://www.scala-lang.org/:

```
Scala combines object-oriented and functional programming in one concise, high-level language. Scala's static types help avoid bugs in complex applications, and its JVM and JavaScript runtimes let you build high-performance systems with easy access to huge ecosystems of libraries.

```

Scala was desgined as a better, more concise version of Java. Scala source code compiles to Java bytecode and runs on the Java Virtual Machine. 

Scala is flexible. It lets you add new types, collections, and control constructs that feel like they are built-in to the language. 

Scala is convenient. The standard library has a set of predefined types, collections and control constructs. 

Scala is **object-oriented** in that every value is an object and every operation is a method call. 

Scala is **functional** in that functions are first-class values and functions are pure. 

Operations get interpreted as method calls: 

```scala
val sumB = 2.+(4)
val sumA = 2 + 4
```

Two kinds of variables. `val` is immutable. `var` is mutable. 

```scala
val fourHearts: Int = 4
var aceSpades: Int = 1
//using type inference
val fourHearts = 4
var aceSpades = 1
```

Value types are `Double`, `Float`, `Long`, `Int`, `Short`, `Byte`, `Char`, `Boolean` and `Unit`. 

`Double` is the most precise type for floating point numbers. 

A type like `Int` refers a class, in this case `scala.Int`. `fourHearts` is an instance of class `Int`. 

**Scala scripts**. Are executed by wrapping it in a template and compiling and executing the resulting program. 

**Scala applications**. Must be combined explicitly and run explicitly. Consist of many source files that can be compiled individually.  They run fast because they are compiled only once. 

**Interpreter**. A program that directly executes instructions. 

**Compiler**. A program that translates source code from a high-level programming language to a lower lever language to create an executable program. 

The most popular tool for building Scala apps is `sbt`, simple build tool. Scala works also in Jupyter Notebook with a kernel called Almond.

Function: 

```scala
def bust(hand: Int): Boolean = {
    hand > 21
}

// Calculate hand values
var handPlayerA: Int = queenDiamonds + threeClubs + aceHearts + fiveSpades
var handPlayerB: Int = kingHearts + jackHearts

// Find and print the maximum hand value
println(maxHand(handPlayerA, handPlayerB))
```

Scala has mutable and immutable collections. 

`Array` is a mutable sequence of objects that share the same type. 

```scala
val players = Array("Alex", "Chen", "Marta")
//more precise
val players: Array[String] = new Array[String](3)
players(0) = "Alex"
players(1) = "Chen"
players(2) = "Marta"
//arrays are mutable, so you can update them 
players(0) = "Sindhu"
//if you update with a different type you get type mismatch error
players(0) = 500
//you can mix types by parametrizing with Any
val mixedTypes = new Array[Any](3)
```

It is recommended to use `val` with `Array`. Otherwise an entirely different array object could be assigned to the same variable. With `val`, the variable cannot be assigned another `Array`, though the `Array` itself is mutable. 

`Lists` are immutable sequence of objects that share the same type. 

```scala
val players = List("Alex", "Chen", "Marta")
//prepend a new element
val newPlayers = "Sindhu" :: players
//if you define players as a var, you can reassign it
var players = List("Alex", "Chen", "Marta")
players = "Sindhu" :: players
```

Calling a method on a List returns a new object. 

`Nil` indicates an empty list. This allows a common way to initialize new lists:

```scala
val players = "Alex" :: "Chen" :: "Marta" :: Nil
```

```scala
//concatenate lists
val playersA = List("Sindhu", "Alex")
val playersB = List("Chen", "Marta")
val allPlayers = playersA ::: playersB
```

**Type**. Restricts the possible values to which a variable can refer, or an expression can produce, at runtime. 
**Compile time**. When source code is translated into machine code. 
**Run time**. When the program is executing commands. 

A language is **statically typed** if the type of the variable is known at compile time. It is **dynamically typed** if types are checked on the fly, at run time. 

Conditionals. 

```scala
if (handA > handB) println(handA) else println(handB)
```

If/else expressions result in values:

```scala
// Point value of a player's hand
val hand = sevenClubs + kingDiamonds + threeSpades

// Inform a player where their current hand stands
val informPlayer: String = {
  if(hand > 21)
    "Bust! :("
  else if (hand == 21)
    "Twenty-One! :)"
  else
    "Hit or stay?"
}

// Print the message
print(informPlayer)
```

```scala
// Find the number of points that will cause a bust
def pointsToBust(hand: Int): Int = {
  // If the hand is a bust, 0 points remain
  if (hand > 21)
    0
  // Otherwise, calculate the difference between 21 and the current hand
  else
    21 - hand
}
```














## What is Java, and what makes it platform independent?
Java is a high-level, object-oriented, and statically typed programming language. What makes it platform independent is the underlying compilation and execution model. Java compiler converts the .java files to bytecode, which is platform independent. We have JVM specific to each OS, which converts this platform independent bytecode to OS specific instruction according to the underlying OS.

## How does Java achieve "write once, run anywhere"?
Java achieves platform independence through underlying execution and compiler model. Java compiler converts the .java files to platform independent bytecode. Each OS has its own JVM. This JVM converts the bytecode into native machine code instructions according to the underlying OS. That's how Java achieves platform independence.

## What are the main features of Java?
The main features of Java are, basically, it is platform independent. Instead of converting the Java code directly into machine code, it is converted into bytecode, which is platform independent code. Then it is executed by the JVM of the respective platform, I mean respective operating system, into native machine code according to underlying OS. And coming to the next one, it is object oriented programming. Everything is around objects and classes. And third one is robustness. Basically, the JVM basically handles the memory stuff and preventing the bugs. And fourth one is it provides security. It verifies the bytecode. And coming to fifth one, it is basically multi-threaded one. Basically, we can utilize the cores, we can efficiently utilize the cores.

## What is the exact difference between JDK, JRE, and JVM?
JDK basically it's Java Development Kit. It is used for developing Java applications. It provides compilers, debuggers, and JRE. JRE stands for Java Runtime Environment. Basically, it provides the standard libraries and JVM for running the Java application. JVM, Java Virtual Machine, is the one which executes the bytecode into native machine code.

## What is the difference between primitive types and reference types?
Primitive store our values directly in memory. Reference types store memory address pointing to the object in the heap. Primitive types are stored in stack, whereas objects are stored in the heap. Primitive types will always have some default values. It can never be null. Objects can be null.

## What is the difference between int, Integer, and autoboxing?
int is the primitive data type holding integer values. Its default value is zero and it can never be null. Coming to Integer, Integer is the wrapper class of the int primitive data type. It stores the int as a wrapped object in heap. It can be null. Coming to autoboxing, it is a way of converting the int to Integer. We are converting primitive data type to our wrapper class. That is done by the compiler. That is called autoboxing.

## What is the default value of instance variables?
The default value of instance variables is basically it depends on the type of the variable. Let's say if it's integer, it will be zero. If it's boolean, it will be false. If it's like object type, if there are object types, it will be null.

## What is the difference between local, instance, and static variables?
So basically, local variables are created whenever a method is called. These are stored in stack, and these are destroyed whenever a method ended. And coming to instance variables, these will get created whenever an object is created. Basically, they live in the heap along with the object, and whenever an object gets destroyed, these instance variables also get destroyed. And coming to static variables, these are class level. These variables are shared across the application, and whenever an application shuts down, these static variables also get destroyed.

## What is the scope and lifetime of a variable in Java?
Scope is basically the visibility of the variable, and lifetime is how long the variable lives in the memory. Coming to local variable, these will get created whenever a method is called. These are whenever a method is called, and this is shared in the method itself. This  cannot accessed by any other thing outside the method. And this will get destroyed whenever the method ends. Next one is instance variables. These are basically, this will get created whenever an object is created. And the scope is across the class, and this will be shared across the class. I mean, this can be accessed by everything in the class, in that class. And this will get destroyed whenever a, whenever an object gets destroyed. And coming to static variables, like these are, this is the class level one. These are created whenever the application starts up, whenever the application is starting, these will get created. And this will be shared across the all the files and classes in that application. It will be shared across all the classes in the application. This can be accessed. And coming to lifetime, this will get destroyed whenever the application shuts down.

## What are access modifiers, and why do they matter?
Access modifiers are the thing which controls the access and visibility of properties or behavior properties or methods of the class. Let's say if a method is private or method or property is private, that can be accessed within the class. It cannot be accessed from anywhere. Coming to default, this can be accessed within the same package. And coming to protected, this can be accessed via same package plus subclasses or through inheritance. A public, it can be accessed from anywhere.

## What is the difference between private, protected, package-private, and public?
What is the difference between private, protected, package private, and public? Basically, these are access modifiers which are used to control the access and visibility of properties or methods. If a method is private, it can be accessed within the class. It cannot be accessed even outside the class or the subclasses. Coming to protected, this can be accessed through inheritance. These methods or properties which are protected can be accessed through inheritance. And coming to package private, these are default basically. These can be accessed within the same package from the classes which are in the same package. Public can be accessed from anywhere.

## What is the difference between final, finally, and finalize()?
So basically, finally is a keyword used to restrict applied restrictions. A final variable cannot be reassigned, a final method cannot be overridden, and a final class cannot be inherited. Finally is a block used with try-catch. Its primary purpose is to close the resources regardless of whether an exception is thrown or not. Finalize is an object class method inherited from object class method. So historically, JVM called it right before the GC cleanup, but the GC cleanup is not predictable. So this led to performance issues. So this was deprecated in favor of try-with-resources. 

## What is the purpose of the main method? What happens when you run a Java class from the command line?
Basically, the main method is a starting point of any standard Java application. JVM uses this main method to start the application, to execute the application logic, to start executing the application logic. Basically, JVM calls the main method, and in main method, we will write our standard logic, write our business logic. So it gets executed. And coming to when we run the Java class from command line, basically the OS initializes JVM, and JVM loads the .class file and verifies the bytecode, and it looks for the main class and executes the class.

## What is this keyword used for?
So basically, we use this keyword when we want to reference the object of a class from class itself. And we primarily use this when our instance has same parameters as the method parameters. We use this.name = name to resolve the naming same name issue. We use like that and we use it for constructor chaining.

## What is super keyword used for?
super is the keyword used in inheritance to directly access the immediate parent class. We use it to call the parent's constructor using super(...), call the parent's implementation of an overridden method with super.method(), or read a parent field with super.field.

## What is the difference between == and .equals()?
== is a binary operator, whereas .equals() is a method defined in java.lang.Object.
1.For primitives, == compares the raw values.
2.For objects, == checks reference equality—whether two variables point to the exact same memory address on the Heap.
3..equals() is designed to check logical value equality. However, by default in Object, it just uses ==. It only compares actual values if the class overrides it—which standard classes like String and wrapper classes do.

## If you override .equals(), what other method must you always override?
If two objects are equal according to equals(), their hashCode() must also match. If their hash codes are different, the collection places them in different buckets, so it will never find the match.
A collision only happens when two unequal objects happen to produce the same hash code and end up in the same bucket

## What is the use of the transient keyword? What about volatile?
So basically, transient keyword is used when we want to ignore a property during serializing the object using object output stream. We use volatile keyword when you want a force thread to write into RAM and read from main heap instead of reading from local CPU caches so that all other threads which are reading stays updated what the other thread writes. This just guarantees visibility, this doesn't guarantee atomicity.

## What are Access Modifiers in Java? Explain their scope.
Access modifiers are the one which control the access and visibility of class properties or methods. Private methods or properties can be accessed within the class. It cannot be accessed outside that class. And coming to protected, it can be accessed within the same package. This class can be accessed, these methods or properties, private methods, protected methods or properties can be accessed from the class within the same package or like the subclass or through the subclass, I mean inheritance. And coming to default, private protected, these are, these methods or properties can be accessed from the same package class which is within the same package. And coming to public, this can be accessed from any class across the application.

## How do you prevent a class from being subclassed?
Okay, let's practice that subclassing. How do you prevent a class from being subclassed? So let me tell you. To prevent a class from being subclassed, there are majorly three ways. First one is making the class final. And the second one is making the constructor private so that subclass will not be able to call parent class constructor. And coming to third one, we can use sealed class. Here we basically permit the classes, I mean, permit only some subclasses. I mean, it's like basically giving the permission to only some set of subclasses instead of giving the permission for all subclasses, all classes. 

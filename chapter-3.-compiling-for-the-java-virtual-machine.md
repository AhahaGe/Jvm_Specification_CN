# Chapter 3. Compiling for the Java Virtual Machine

## 3.5. More Control Examples

Compilation of `for` statements was shown in an earlier section \([§3.2](chapter-3.-compiling-for-the-java-virtual-machine.md#jvms-3.2)\). Most of the Java programming language's other control constructs \(`if-then-else`, `do`, `while`, `break`, and `continue`\) are also compiled in the obvious ways. The compilation of `switch` statements is handled in a separate section \([§3.10](chapter-3.-compiling-for-the-java-virtual-machine.md#jvms-3.10)\), as are the compilation of exceptions \([§3.12](chapter-3.-compiling-for-the-java-virtual-machine.md#jvms-3.12)\) and the compilation of `finally` clauses \([§3.13](chapter-3.-compiling-for-the-java-virtual-machine.md#jvms-3.13)\).

As a further example, a `while` loop is compiled in an obvious way, although the specific control transfer instructions made available by the Java Virtual Machine vary by data type. As usual, there is more support for data of type `int`, for example:

```text
void whileInt() {
    int i = 0;
    while (i < 100) {
        i++;
    }
}

```

is compiled to:

```text
Method void whileInt()
0   iconst_0
1   istore_1
2   goto 8
5   iinc 1 1
8   iload_1
9   bipush 100
11  if_icmplt 5
14  return
```

Note that the test of the `while` statement \(implemented using the _if\_icmplt_ instruction\) is at the bottom of the Java Virtual Machine code for the loop. \(This was also the case in the `spin` examples earlier.\) The test being at the bottom of the loop forces the use of a _goto_ instruction to get to the test prior to the first iteration of the loop. If that test fails, and the loop body is never entered, this extra instruction is wasted. However, `while` loops are typically used when their body is expected to be run, often for many iterations. For subsequent iterations, putting the test at the bottom of the loop saves a Java Virtual Machine instruction each time around the loop: if the test were at the top of the loop, the loop body would need a trailing _goto_ instruction to get back to the top.

Control constructs involving other data types are compiled in similar ways, but must use the instructions available for those data types. This leads to somewhat less efficient code because more Java Virtual Machine instructions are needed, for example:

```text
void whileDouble() {
    double i = 0.0;
    while (i < 100.1) {
        i++;
    }
}

```

is compiled to:

```text
Method void whileDouble()
0   dconst_0
1   dstore_1
2   goto 9
5   dload_1
6   dconst_1
7   dadd
8   dstore_1
9   dload_1
10  ldc2_w #4      // Push double constant 100.1
13  dcmpg          // To compare and branch we have to use...
14  iflt 5         // ...two instructions
17  return
```

Each floating-point type has two comparison instructions: _fcmpl_ and _fcmpg_ for type `float`, and _dcmpl_ and _dcmpg_ for type `double`. The variants differ only in their treatment of NaN. NaN is unordered \([§2.3.2](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.3.2)\), so all floating-point comparisons fail if either of their operands is NaN. The compiler chooses the variant of the comparison instruction for the appropriate type that produces the same result whether the comparison fails on non-NaN values or encounters a NaN. For instance:

```text
int lessThan100(double d) {
    if (d < 100.0) {
        return 1;				
    } else {
        return -1;				
    }
}

```

compiles to:

```text
Method int lessThan100(double)
0   dload_1
1   ldc2_w #4      // Push double constant 100.0
4   dcmpg          // Push 1 if d is NaN or d > 100.0;
                   // push 0 if d == 100.0
5   ifge 10        // Branch on 0 or 1
8   iconst_1
9   ireturn
10  iconst_m1
11  ireturn
```

If `d` is not NaN and is less than `100.0`, the _dcmpg_ instruction pushes an `int` _-1_ onto the operand stack, and the _ifge_ instruction does not branch. Whether `d` is greater than `100.0` or is NaN, the _dcmpg_ instruction pushes an `int` _1_ onto the operand stack, and the _ifge_ branches. If `d` is equal to `100.0`, the _dcmpg_ instruction pushes an `int` _0_ onto the operand stack, and the _ifge_ branches.

The _dcmpl_ instruction achieves the same effect if the comparison is reversed:

```text
int greaterThan100(double d) {
    if (d > 100.0) {
        return 1;			
    } else {
        return -1;			
    }
}

```

becomes:

```text
Method int greaterThan100(double)
0   dload_1
1   ldc2_w #4      // Push double constant 100.0
4   dcmpl          // Push -1 if d is NaN or d < 100.0;
                   // push 0 if d == 100.0
5   ifle 10        // Branch on 0 or -1
8   iconst_1
9   ireturn
10  iconst_m1
11  ireturn
```

Once again, whether the comparison fails on a non-NaN value or because it is passed a NaN, the _dcmpl_ instruction pushes an `int` value onto the operand stack that causes the _ifle_ to branch. If both of the _dcmp_ instructions did not exist, one of the example methods would have had to do more work to detect NaN.

If _n_ arguments are passed to an instance method, they are received, by convention, in the local variables numbered _1_ through _n_ of the frame created for the new method invocation. The arguments are received in the order they were passed. For example:

```text
int addTwo(int i, int j) {
    return i + j;
}

```

compiles to:

```text
Method int addTwo(int,int)
0   iload_1        // Push value of local variable 1 (i)
1   iload_2        // Push value of local variable 2 (j)
2   iadd           // Add; leave int result on operand stack
3   ireturn        // Return int result
```

By convention, an instance method is passed a `reference` to its instance in local variable _0_. In the Java programming language the instance is accessible via the `this` keyword.

Class \(`static`\) methods do not have an instance, so for them this use of local variable _0_ is unnecessary. A class method starts using local variables at index _0_. If the `addTwo` method were a class method, its arguments would be passed in a similar way to the first version:

```text
static int addTwoStatic(int i, int j) {
    return i + j;
}

```

compiles to:

```text
Method int addTwoStatic(int,int)
0   iload_0
1   iload_1
2   iadd
3   ireturn
```

The only difference is that the method arguments appear starting in local variable _0_ rather than _1_.

The normal method invocation for a instance method dispatches on the run-time type of the object. \(They are virtual, in C++ terms.\) Such an invocation is implemented using the _invokevirtual_ instruction, which takes as its argument an index to a run-time constant pool entry giving the internal form of the binary name of the class type of the object, the name of the method to invoke, and that method's descriptor \([§4.3.3](chapter-4.-the-class-file-format/#jvms-4.3.3)\). To invoke the `addTwo` method, defined earlier as an instance method, we might write:

```text
int add12and13() {
    return addTwo(12, 13);
}

```

This compiles to:

```text
Method int add12and13()
0   aload_0             // Push local variable 0 (this)
1   bipush 12           // Push int constant 12
3   bipush 13           // Push int constant 13
5   invokevirtual #4    // Method Example.addtwo(II)I
8   ireturn             // Return int on top of operand stack;
                        // it is the int result of addTwo()
```

The invocation is set up by first pushing a `reference` to the current instance, `this`, on to the operand stack. The method invocation's arguments, `int` values `12` and `13`, are then pushed. When the frame for the `addTwo` method is created, the arguments passed to the method become the initial values of the new frame's local variables. That is, the `reference` for `this` and the two arguments, pushed onto the operand stack by the invoker, will become the initial values of local variables _0_, _1_, and _2_ of the invoked method.

Finally, `addTwo` is invoked. When it returns, its `int` return value is pushed onto the operand stack of the frame of the invoker, the `add12and13` method. The return value is thus put in place to be immediately returned to the invoker of `add12and13`.

The return from `add12and13` is handled by the _ireturn_ instruction of `add12and13`. The _ireturn_ instruction takes the `int` value returned by `addTwo`, on the operand stack of the current frame, and pushes it onto the operand stack of the frame of the invoker. It then returns control to the invoker, making the invoker's frame current. The Java Virtual Machine provides distinct return instructions for many of its numeric and `reference` data types, as well as a _return_ instruction for methods with no return value. The same set of return instructions is used for all varieties of method invocations.

The operand of the _invokevirtual_ instruction \(in the example, the run-time constant pool index _\#4_\) is not the offset of the method in the class instance. The compiler does not know the internal layout of a class instance. Instead, it generates symbolic references to the methods of an instance, which are stored in the run-time constant pool. Those run-time constant pool items are resolved at run-time to determine the actual method location. The same is true for all other Java Virtual Machine instructions that access class instances.

Invoking `addTwoStatic`, a class \(`static`\) variant of `addTwo`, is similar, as shown:

```text
int add12and13() {
    return addTwoStatic(12, 13);
}

```

although a different Java Virtual Machine method invocation instruction is used:

```text
Method int add12and13()
0   bipush 12
2   bipush 13
4   invokestatic #3     // Method Example.addTwoStatic(II)I
7   ireturn
```

Compiling an invocation of a class \(`static`\) method is very much like compiling an invocation of an instance method, except this is not passed by the invoker. The method arguments will thus be received beginning with local variable _0_ \([§3.6](chapter-3.-compiling-for-the-java-virtual-machine.md#jvms-3.6)\). The _invokestatic_ instruction is always used to invoke class methods.

The _invokespecial_ instruction must be used to invoke instance initialization methods \([§3.8](chapter-3.-compiling-for-the-java-virtual-machine.md#jvms-3.8)\). It is also used when invoking methods in the superclass \(`super`\) and when invoking `private` methods. For instance, given classes `Near` and `Far` declared as:

```text
class Near {
    int it;
    public int getItNear() {
        return getIt();
    }
    private int getIt() {
        return it;
    }
}

class Far extends Near {
    int getItFar() {
        return super.getItNear();
    }
}

```

the method `Near.getItNear` \(which invokes a `private` method\) becomes:

```text
Method int getItNear()
0   aload_0
1   invokespecial #5    // Method Near.getIt()I
4   ireturn
```

The method `Far.getItFar` \(which invokes a superclass method\) becomes:

```text
Method int getItFar()
0   aload_0
1   invokespecial #4    // Method Near.getItNear()I
4   ireturn
```

Note that methods called using the _invokespecial_ instruction always pass `this` to the invoked method as its first argument. As usual, it is received in local variable _0_.

To invoke the target of a method handle, a compiler must form a method descriptor that records the actual argument and return types. A compiler may not perform method invocation conversions on the arguments; instead, it must push them on the stack according to their own unconverted types. The compiler arranges for a `reference` to the method handle object to be pushed on the stack before the arguments, as usual. The compiler emits an _invokevirtual_ instruction that references a descriptor which describes the argument and return types. By special arrangement with method resolution \([§5.4.3.3](chapter-5.-loading-linking-and-initializing.md#jvms-5.4.3.3)\), an _invokevirtual_ instruction which invokes the `invokeExact` or `invoke` methods of `java.lang.invoke.MethodHandle` will always link, provided the method descriptor is syntactically well-formed and the types named in the descriptor can be resolved.

## 3.8. Working with Class Instances

Java Virtual Machine class instances are created using the Java Virtual Machine's _new_ instruction. Recall that at the level of the Java Virtual Machine, a constructor appears as a method with the compiler-supplied name. This specially named method is known as the instance initialization method \([§2.9](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.9)\). Multiple instance initialization methods, corresponding to multiple constructors, may exist for a given class. Once the class instance has been created and its instance variables, including those of the class and all of its superclasses, have been initialized to their default values, an instance initialization method of the new class instance is invoked. For example:

```text
Object create() {
    return new Object();
}

```

compiles to:

```text
Method java.lang.Object create()
0   new #1              // Class java.lang.Object
3   dup
4   invokespecial #4    // Method java.lang.Object.()V
7   areturn
```

Class instances are passed and returned \(as `reference` types\) very much like numeric values, although type `reference` has its own complement of instructions, for example:

```text
int i;                                  // An instance variable
MyObj example() {
    MyObj o = new MyObj();
    return silly(o);
}
MyObj silly(MyObj o) {
    if (o != null) {
        return o;
    } else {
        return o;
    }
}

```

becomes:

```text
Method MyObj example()
0   new #2              // Class MyObj
3   dup
4   invokespecial #5    // Method MyObj.()V
7   astore_1
8   aload_0
9   aload_1
10  invokevirtual #4    // Method Example.silly(LMyObj;)LMyObj;
13  areturn

Method MyObj silly(MyObj)
0   aload_1
1   ifnull 6
4   aload_1
5   areturn
6   aload_1
7   areturn
```

The fields of a class instance \(instance variables\) are accessed using the _getfield_ and _putfield_ instructions. If `i` is an instance variable of type `int`, the methods `setIt` and `getIt`, defined as:

```text
void setIt(int value) {
    i = value;
}
int getIt() {
    return i;
}

```

become:

```text
Method void setIt(int)
0   aload_0
1   iload_1
2   putfield #4    // Field Example.i I
5   return

Method int getIt()
0   aload_0
1   getfield #4    // Field Example.i I
4   ireturn
```

As with the operands of method invocation instructions, the operands of the _putfield_ and _getfield_ instructions \(the run-time constant pool index _\#4_\) are not the offsets of the fields in the class instance. The compiler generates symbolic references to the fields of an instance, which are stored in the run-time constant pool. Those run-time constant pool items are resolved at run-time to determine the location of the field within the referenced object.

Java Virtual Machine arrays are also objects. Arrays are created and manipulated using a distinct set of instructions. The _newarray_ instruction is used to create an array of a numeric type. The code:

```text
void createBuffer() {
    int buffer[];
    int bufsz = 100;
    int value = 12;
    buffer = new int[bufsz];
    buffer[10] = value;
    value = buffer[11];
}

```

might be compiled to:

```text
Method void createBuffer()
0   bipush 100     // Push int constant 100 (bufsz)
2   istore_2       // Store bufsz in local variable 2
3   bipush 12      // Push int constant 12 (value)
5   istore_3       // Store value in local variable 3
6   iload_2        // Push bufsz...
7   newarray int   // ...and create new int array of that length
9   astore_1       // Store new array in buffer
10  aload_1        // Push buffer
11  bipush 10      // Push int constant 10
13  iload_3        // Push value
14  iastore        // Store value at buffer[10]
15  aload_1        // Push buffer
16  bipush 11      // Push int constant 11
18  iaload         // Push value at buffer[11]...
19  istore_3       // ...and store it in value
20  return
```

The _anewarray_ instruction is used to create a one-dimensional array of object references, for example:

```text
void createThreadArray() {
    Thread threads[];
    int count = 10;
    threads = new Thread[count];
    threads[0] = new Thread();
}

```

becomes:

```text
Method void createThreadArray()
0   bipush 10           // Push int constant 10
2   istore_2            // Initialize count to that
3   iload_2             // Push count, used by anewarray
4   anewarray class #1  // Create new array of class Thread
7   astore_1            // Store new array in threads
8   aload_1             // Push value of threads
9   iconst_0            // Push int constant 0
10  new #1              // Create instance of class Thread
13  dup                 // Make duplicate reference...
14  invokespecial #5    // ...for Thread's constructor
                        // Method java.lang.Thread.()V
17  aastore             // Store new Thread in array at 0
18  return
```

The _anewarray_ instruction can also be used to create the first dimension of a multidimensional array. Alternatively, the _multianewarray_ instruction can be used to create several dimensions at once. For example, the three-dimensional array:

```text
int[][][] create3DArray() {
    int grid[][][];
    grid = new int[10][5][];
    return grid;
}

```

is created by:

```text
Method int create3DArray()[][][]
0   bipush 10                // Push int 10 (dimension one)
2   iconst_5                 // Push int 5 (dimension two)
3   multianewarray #1 dim #2 // Class [[[I, a three-dimensional
                             // int array; only create the
                             // first two dimensions
7   astore_1                 // Store new array...
8   aload_1                  // ...then prepare to return it
9   areturn
```

The first operand of the _multianewarray_ instruction is the run-time constant pool index to the array class type to be created. The second is the number of dimensions of that array type to actually create. The _multianewarray_ instruction can be used to create all the dimensions of the type, as the code for `create3DArray` shows. Note that the multidimensional array is just an object and so is loaded and returned by an _aload\_1_ and _areturn_ instruction, respectively. For information about array class names, see [§4.4.1](chapter-4.-the-class-file-format/#jvms-4.4.1).

All arrays have associated lengths, which are accessed via the _arraylength_ instruction.

Compilation of `switch` statements uses the _tableswitch_ and _lookupswitch_ instructions. The _tableswitch_ instruction is used when the cases of the `switch` can be efficiently represented as indices into a table of target offsets. The `default` target of the `switch` is used if the value of the expression of the `switch` falls outside the range of valid indices. For instance:

```text
int chooseNear(int i) {
    switch (i) {
        case 0:  return  0;
        case 1:  return  1;
        case 2:  return  2;
        default: return -1;
    }
}

```

compiles to:

```text
Method int chooseNear(int)
0   iload_1             // Push local variable 1 (argument i)
1   tableswitch 0 to 2: // Valid indices are 0 through 2
      0: 28             // If i is 0, continue at 28
      1: 30             // If i is 1, continue at 30
      2: 32             // If i is 2, continue at 32
      default:34        // Otherwise, continue at 34
28  iconst_0            // i was 0; push int constant 0...
29  ireturn             // ...and return it
30  iconst_1            // i was 1; push int constant 1...
31  ireturn             // ...and return it
32  iconst_2            // i was 2; push int constant 2...
33  ireturn             // ...and return it
34  iconst_m1           // otherwise push int constant -1...
35  ireturn             // ...and return it
```

The Java Virtual Machine's _tableswitch_ and _lookupswitch_ instructions operate only on `int` data. Because operations on `byte`, `char`, or `short` values are internally promoted to `int`, a `switch` whose expression evaluates to one of those types is compiled as though it evaluated to type `int`. If the `chooseNear` method had been written using type `short`, the same Java Virtual Machine instructions would have been generated as when using type `int`. Other numeric types must be narrowed to type `int` for use in a `switch`.

Where the cases of the `switch` are sparse, the table representation of the _tableswitch_ instruction becomes inefficient in terms of space. The _lookupswitch_ instruction may be used instead. The _lookupswitch_ instruction pairs `int` keys \(the values of the `case` labels\) with target offsets in a table. When a _lookupswitch_ instruction is executed, the value of the expression of the `switch` is compared against the keys in the table. If one of the keys matches the value of the expression, execution continues at the associated target offset. If no key matches, execution continues at the `default` target. For instance, the compiled code for:

```text
int chooseFar(int i) {
    switch (i) {
        case -100: return -1;
        case 0:    return  0;
        case 100:  return  1;
        default:   return -1;
    }
}

```

looks just like the code for `chooseNear`, except for the _lookupswitch_ instruction:

```text
Method int chooseFar(int)
0   iload_1
1   lookupswitch 3:
         -100: 36
            0: 38
          100: 40
      default: 42
36  iconst_m1
37  ireturn
38  iconst_0
39  ireturn
40  iconst_1
41  ireturn
42  iconst_m1
43  ireturn
```

The Java Virtual Machine specifies that the table of the _lookupswitch_ instruction must be sorted by key so that implementations may use searches more efficient than a linear scan. Even so, the _lookupswitch_ instruction must search its keys for a match rather than simply perform a bounds check and index into a table like _tableswitch_. Thus, a _tableswitch_ instruction is probably more efficient than a _lookupswitch_ where space considerations permit a choice.

## 3.11. Operations on the Operand Stack

The Java Virtual Machine has a large complement of instructions that manipulate the contents of the operand stack as untyped values. These are useful because of the Java Virtual Machine's reliance on deft manipulation of its operand stack. For instance:

```text
public long nextIndex() { 
    return index++;
}

private long index = 0;

```

is compiled to:

```text
Method long nextIndex()
0   aload_0        // Push this
1   dup            // Make a copy of it
2   getfield #4    // One of the copies of this is consumed
                   // pushing long field index,
                   // above the original this
5   dup2_x1        // The long on top of the operand stack is 
                   // inserted into the operand stack below the 
                   // original this
6   lconst_1       // Push long constant 1 
7   ladd           // The index value is incremented...
8   putfield #4    // ...and the result stored in the field
11  lreturn        // The original value of index is on top of
                   // the operand stack, ready to be returned
```

Note that the Java Virtual Machine never allows its operand stack manipulation instructions to modify or break up individual values on the operand stack.

## 3.12. Throwing and Handling Exceptions

Exceptions are thrown from programs using the `throw` keyword. Its compilation is simple:

```text
void cantBeZero(int i) throws TestExc {
    if (i == 0) {
        throw new TestExc();
    }
}

```

becomes:

```text
Method void cantBeZero(int)
0   iload_1             // Push argument 1 (i)
1   ifne 12             // If i==0, allocate instance and throw
4   new #1              // Create instance of TestExc
7   dup                 // One reference goes to its constructor
8   invokespecial #7    // Method TestExc.()V
11  athrow              // Second reference is thrown
12  return              // Never get here if we threw TestExc
```

Compilation of `try`-`catch` constructs is straightforward. For example:

```text
void catchOne() {
    try {
        tryItOut();
    } catch (TestExc e) {
        handleExc(e);
    }
}

```

is compiled as:

```text
Method void catchOne()
0   aload_0             // Beginning of try block
1   invokevirtual #6    // Method Example.tryItOut()V
4   return              // End of try block; normal return
5   astore_1            // Store thrown value in local var 1
6   aload_0             // Push this
7   aload_1             // Push thrown value
8   invokevirtual #5    // Invoke handler method: 
                        // Example.handleExc(LTestExc;)V
11  return              // Return after handling TestExc
Exception table:
From    To      Target      Type
0       4       5           Class TestExc
```

Looking more closely, the `try` block is compiled just as it would be if the `try` were not present:

```text
Method void catchOne()
0   aload_0             // Beginning of try block
1   invokevirtual #6    // Method Example.tryItOut()V
4   return              // End of try block; normal return
```

If no exception is thrown during the execution of the `try` block, it behaves as though the `try` were not there: `tryItOut` is invoked and `catchOne` returns.

Following the `try` block is the Java Virtual Machine code that implements the single `catch` clause:

```text
5   astore_1            // Store thrown value in local var 1
6   aload_0             // Push this
7   aload_1             // Push thrown value
8   invokevirtual #5    // Invoke handler method: 
                        // Example.handleExc(LTestExc;)V
11  return              // Return after handling TestExc
Exception table:
From    To      Target      Type
0       4       5           Class TestExc
```

The invocation of `handleExc`, the contents of the `catch` clause, is also compiled like a normal method invocation. However, the presence of a `catch` clause causes the compiler to generate an exception table entry \([§2.10](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.10), [§4.7.3](chapter-4.-the-class-file-format/#jvms-4.7.3)\). The exception table for the `catchOne` method has one entry corresponding to the one argument \(an instance of class `TestExc`\) that the `catch` clause of `catchOne` can handle. If some value that is an instance of `TestExc` is thrown during execution of the instructions between indices _0_ and _4_ in `catchOne`, control is transferred to the Java Virtual Machine code at index _5_, which implements the block of the `catch` clause. If the value that is thrown is not an instance of `TestExc`, the `catch` clause of `catchOne` cannot handle it. Instead, the value is rethrown to the invoker of `catchOne`.

A `try` may have multiple `catch` clauses:

```text
void catchTwo() {
    try {
        tryItOut();
    } catch (TestExc1 e) {
        handleExc(e);
    } catch (TestExc2 e) {
        handleExc(e);
    }
}

```

Multiple `catch` clauses of a given `try` statement are compiled by simply appending the Java Virtual Machine code for each `catch` clause one after the other and adding entries to the exception table, as shown:

```text
Method void catchTwo()
0   aload_0             // Begin try block
1   invokevirtual #5    // Method Example.tryItOut()V
4   return              // End of try block; normal return
5   astore_1            // Beginning of handler for TestExc1;
                        // Store thrown value in local var 1
6   aload_0             // Push this
7   aload_1             // Push thrown value
8   invokevirtual #7    // Invoke handler method:
                        // Example.handleExc(LTestExc1;)V
11  return              // Return after handling TestExc1
12  astore_1            // Beginning of handler for TestExc2;
                        // Store thrown value in local var 1
13  aload_0             // Push this
14  aload_1             // Push thrown value
15  invokevirtual #7    // Invoke handler method:
                        // Example.handleExc(LTestExc2;)V
18  return              // Return after handling TestExc2
Exception table:
From    To      Target      Type
0       4       5           Class TestExc1
0       4       12          Class TestExc2
```

If during the execution of the `try` clause \(between indices _0_ and _4_\) a value is thrown that matches the parameter of one or more of the `catch` clauses \(the value is an instance of one or more of the parameters\), the first \(innermost\) such `catch` clause is selected. Control is transferred to the Java Virtual Machine code for the block of that `catch` clause. If the value thrown does not match the parameter of any of the `catch` clauses of `catchTwo`, the Java Virtual Machine rethrows the value without invoking code in any `catch` clause of `catchTwo`.

Nested `try`-`catch` statements are compiled very much like a `try` statement with multiple `catch` clauses:

```text
void nestedCatch() {
    try {
        try {
            tryItOut();
        } catch (TestExc1 e) {
            handleExc1(e);
        }
    } catch (TestExc2 e) {
        handleExc2(e);
    }
}

```

becomes:

```text
Method void nestedCatch()
0   aload_0             // Begin try block
1   invokevirtual #8    // Method Example.tryItOut()V
4   return              // End of try block; normal return
5   astore_1            // Beginning of handler for TestExc1;
                        // Store thrown value in local var 1
6   aload_0             // Push this
7   aload_1             // Push thrown value
8   invokevirtual #7    // Invoke handler method: 
                        // Example.handleExc1(LTestExc1;)V
11  return              // Return after handling TestExc1
12  astore_1            // Beginning of handler for TestExc2;
                        // Store thrown value in local var 1
13  aload_0             // Push this
14  aload_1             // Push thrown value
15  invokevirtual #6    // Invoke handler method:
                        // Example.handleExc2(LTestExc2;)V
18  return              // Return after handling TestExc2
Exception table:
From    To      Target      Type
0       4       5           Class TestExc1
0       12      12          Class TestExc2
```

The nesting of `catch` clauses is represented only in the exception table. The Java Virtual Machine does not enforce nesting of or any ordering of the exception table entries \([§2.10](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.10)\). However, because `try`-`catch` constructs are structured, a compiler can always order the entries of the exception handler table such that, for any thrown exception and any program counter value in that method, the first exception handler that matches the thrown exception corresponds to the innermost matching `catch` clause.

For instance, if the invocation of `tryItOut` \(at index _1_\) threw an instance of `TestExc1`, it would be handled by the `catch` clause that invokes `handleExc1`. This is so even though the exception occurs within the bounds of the outer `catch` clause \(catching `TestExc2`\) and even though that outer `catch` clause might otherwise have been able to handle the thrown value.

As a subtle point, note that the range of a `catch` clause is inclusive on the "from" end and exclusive on the "to" end \([§4.7.3](chapter-4.-the-class-file-format/#jvms-4.7.3)\). Thus, the exception table entry for the `catch` clause catching `TestExc1` does not cover the _return_ instruction at offset _4_. However, the exception table entry for the `catch` clause catching `TestExc2` does cover the _return_ instruction at offset _11_. Return instructions within nested `catch` clauses are included in the range of instructions covered by nesting `catch` clauses.

\(This section assumes a compiler generates `class` files with version number 50.0 or below, so that the _jsr_ instruction may be used. See also [§4.10.2.5](chapter-4.-the-class-file-format/#jvms-4.10.2.5).\)

Compilation of a `try`-`finally` statement is similar to that of `try`-`catch`. Prior to transferring control outside the `try` statement, whether that transfer is normal or abrupt, because an exception has been thrown, the `finally` clause must first be executed. For this simple example:

```text
void tryFinally() {
    try {
        tryItOut();
    } finally {
        wrapItUp();
    }
}

```

the compiled code is:

```text
Method void tryFinally()
0   aload_0             // Beginning of try block
1   invokevirtual #6    // Method Example.tryItOut()V
4   jsr 14              // Call finally block
7   return              // End of try block
8   astore_1            // Beginning of handler for any throw
9   jsr 14              // Call finally block
12  aload_1             // Push thrown value
13  athrow              // ...and rethrow value to the invoker
14  astore_2            // Beginning of finally block
15  aload_0             // Push this
16  invokevirtual #5    // Method Example.wrapItUp()V
19  ret 2               // Return from finally block
Exception table:
From    To      Target      Type
0       4       8           any
```

There are four ways for control to pass outside of the `try` statement: by falling through the bottom of that block, by returning, by executing a `break` or `continue` statement, or by raising an exception. If `tryItOut` returns without raising an exception, control is transferred to the `finally` block using a _jsr_ instruction. The _jsr_ _14_ instruction at index _4_ makes a "subroutine call" to the code for the `finally` block at index _14_ \(the `finally` block is compiled as an embedded subroutine\). When the `finally` block completes, the _ret_ _2_ instruction returns control to the instruction following the _jsr_ instruction at index _4_.

In more detail, the subroutine call works as follows: The _jsr_ instruction pushes the address of the following instruction \(_return_ at index _7_\) onto the operand stack before jumping. The _astore\_2_ instruction that is the jump target stores the address on the operand stack into local variable _2_. The code for the `finally` block \(in this case the _aload\_0_ and _invokevirtual_ instructions\) is run. Assuming execution of that code completes normally, the _ret_ instruction retrieves the address from local variable _2_ and resumes execution at that address. The _return_ instruction is executed, and `tryFinally` returns normally.

A `try` statement with a `finally` clause is compiled to have a special exception handler, one that can handle any exception thrown within the `try` statement. If `tryItOut` throws an exception, the exception table for `tryFinally` is searched for an appropriate exception handler. The special handler is found, causing execution to continue at index _8_. The _astore\_1_ instruction at index _8_ stores the thrown value into local variable _1_. The following _jsr_ instruction does a subroutine call to the code for the `finally` block. Assuming that code returns normally, the _aload\_1_ instruction at index _12_ pushes the thrown value back onto the operand stack, and the following _athrow_ instruction rethrows the value.

Compiling a `try` statement with both a `catch` clause and a `finally` clause is more complex:

```text
void tryCatchFinally() {
    try {
        tryItOut();
    } catch (TestExc e) {
        handleExc(e);
    } finally {
        wrapItUp();
    }
}

```

becomes:

```text
Method void tryCatchFinally()
0   aload_0             // Beginning of try block
1   invokevirtual #4    // Method Example.tryItOut()V
4   goto 16             // Jump to finally block
7   astore_3            // Beginning of handler for TestExc;
                        // Store thrown value in local var 3
8   aload_0             // Push this
9   aload_3             // Push thrown value
10  invokevirtual #6    // Invoke handler method:
                        // Example.handleExc(LTestExc;)V
13  goto 16             // This goto is unnecessary, but was
                        // generated by javac in JDK 1.0.2
16  jsr 26              // Call finally block
19  return              // Return after handling TestExc
20  astore_1            // Beginning of handler for exceptions
                        // other than TestExc, or exceptions
                        // thrown while handling TestExc
21  jsr 26              // Call finally block
24  aload_1             // Push thrown value...
25  athrow              // ...and rethrow value to the invoker
26  astore_2            // Beginning of finally block
27  aload_0             // Push this
28  invokevirtual #5    // Method Example.wrapItUp()V
31  ret 2               // Return from finally block
Exception table:
From    To      Target      Type
0       4       7           Class TestExc
0       16      20          any
```

If the `try` statement completes normally, the _goto_ instruction at index _4_ jumps to the subroutine call for the `finally` block at index _16_. The `finally` block at index _26_ is executed, control returns to the _return_ instruction at index _19_, and `tryCatchFinally` returns normally.

If `tryItOut` throws an instance of `TestExc`, the first \(innermost\) applicable exception handler in the exception table is chosen to handle the exception. The code for that exception handler, beginning at index _7_, passes the thrown value to `handleExc` and on its return makes the same subroutine call to the `finally` block at index _26_ as in the normal case. If an exception is not thrown by `handleExc`, `tryCatchFinally` returns normally.

If `tryItOut` throws a value that is not an instance of `TestExc` or if `handleExc` itself throws an exception, the condition is handled by the second entry in the exception table, which handles any value thrown between indices _0_ and _16_. That exception handler transfers control to index _20_, where the thrown value is first stored in local variable _1_. The code for the `finally` block at index _26_ is called as a subroutine. If it returns, the thrown value is retrieved from local variable _1_ and rethrown using the _athrow_ instruction. If a new value is thrown during execution of the `finally` clause, the `finally` clause aborts, and `tryCatchFinally` returns abruptly, throwing the new value to its invoker.

Synchronization in the Java Virtual Machine is implemented by monitor entry and exit, either explicitly \(by use of the _monitorenter_ and _monitorexit_ instructions\) or implicitly \(by the method invocation and return instructions\).

For code written in the Java programming language, perhaps the most common form of synchronization is the `synchronized` method. A `synchronized` method is not normally implemented using _monitorenter_ and _monitorexit_. Rather, it is simply distinguished in the run-time constant pool by the `ACC_SYNCHRONIZED` flag, which is checked by the method invocation instructions \([§2.11.10](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.11.10)\).

The _monitorenter_ and _monitorexit_ instructions enable the compilation of `synchronized` statements. For example:

```text
void onlyMe(Foo f) {
    synchronized(f) {
        doSomething();
    }
}

```

is compiled to:

```text
Method void onlyMe(Foo)
0   aload_1             // Push f
1   dup                 // Duplicate it on the stack
2   astore_2            // Store duplicate in local variable 2
3   monitorenter        // Enter the monitor associated with f
4   aload_0             // Holding the monitor, pass this and...
5   invokevirtual #5    // ...call Example.doSomething()V
8   aload_2             // Push local variable 2 (f)
9   monitorexit         // Exit the monitor associated with f
10  goto 18             // Complete the method normally
13  astore_3            // In case of any throw, end up here
14  aload_2             // Push local variable 2 (f)
15  monitorexit         // Be sure to exit the monitor!
16  aload_3             // Push thrown value...
17  athrow              // ...and rethrow value to the invoker
18  return              // Return in the normal case
Exception table:
From    To      Target      Type
4       10      13          any
13      16      13          any
```

The compiler ensures that at any method invocation completion, a _monitorexit_ instruction will have been executed for each _monitorenter_ instruction executed since the method invocation. This is the case whether the method invocation completes normally \([§2.6.4](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.6.4)\) or abruptly \([§2.6.5](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.6.5)\). To enforce proper pairing of _monitorenter_ and _monitorexit_ instructions on abrupt method invocation completion, the compiler generates exception handlers \([§2.10](chapter-2.-the-structure-of-the-java-virtual-machine.md#jvms-2.10)\) that will match any exception and whose associated code executes the necessary _monitorexit_ instructions.

The representation of annotations in `class` files is described in [§4.7.16](chapter-4.-the-class-file-format/#jvms-4.7.16)-[§4.7.22](chapter-4.-the-class-file-format/#jvms-4.7.22). These sections make it clear how to represent annotations on declarations of classes, interfaces, fields, methods, method parameters, and type parameters, as well as annotations on types used in those declarations. Annotations on package declarations require additional rules, given here.

When the compiler encounters an annotated package declaration that must be made available at run time, it emits a `class` file with the following properties:

*  The `class` file represents an interface, that is, the `ACC_INTERFACE` and `ACC_ABSTRACT` flags of the `ClassFile` structure are set \([§4.1](chapter-4.-the-class-file-format/#jvms-4.1)\).
*  If the `class` file version number is less than 50.0, then the `ACC_SYNTHETIC` flag is unset; if the `class` file version number is 50.0 or above, then the `ACC_SYNTHETIC` flag is set.
*  The interface has package access \(JLS §6.6.1\).
*  The interface's name is the internal form \([§4.2.1](chapter-4.-the-class-file-format/#jvms-4.2.1)\) of _`package-name`_`.package-info`.
*  The interface has no superinterfaces.
*  The interface's only members are those implied by _The Java Language Specification, Java SE 8 Edition_ \(JLS §9.2\).
*  The annotations on the package declaration are stored as `RuntimeVisibleAnnotations` and `RuntimeInvisibleAnnotations` attributes in the `attributes` table of the `ClassFile` structure.


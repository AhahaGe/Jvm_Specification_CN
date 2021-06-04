# Chapter 6. The Java Virtual Machine Instruction Set

A Java Virtual Machine instruction consists of an opcode specifying the operation to be performed, followed by zero or more operands embodying values to be operated upon. This chapter gives details about the format of each Java Virtual Machine instruction and the operation it performs.

### 6.1. Assumptions: The Meaning of "Must"

The description of each instruction is always given in the context of Java Virtual Machine code that satisfies the static and structural constraints of [§4 \(The `class` File Format\)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html). In the description of individual Java Virtual Machine instructions, we frequently state that some situation "must" or "must not" be the case: "The _value2_ must be of type `int`." The constraints of [§4 \(The `class` File Format\)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html) guarantee that all such expectations will in fact be met. If some constraint \(a "must" or "must not"\) in an instruction description is not satisfied at run time, the behavior of the Java Virtual Machine is undefined.

The Java Virtual Machine checks that Java Virtual Machine code satisfies the static and structural constraints at link time using a `class` file verifier \([§4.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.10)\). Thus, a Java Virtual Machine will only attempt to execute code from valid `class` files. Performing verification at link time is attractive in that the checks are performed just once, substantially reducing the amount of work that must be done at run time. Other implementation strategies are possible, provided that they comply with _The Java Language Specification, Java SE 8 Edition_ and _The Java Virtual Machine Specification, Java SE 8 Edition_.

### 6.2. Reserved Opcodes

In addition to the opcodes of the instructions specified later in this chapter, which are used in `class` files \([§4 \(The `class` File Format\)](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)\), three opcodes are reserved for internal use by a Java Virtual Machine implementation. If the instruction set of the Java Virtual Machine is extended in the future, these reserved opcodes are guaranteed not to be used.

Two of the reserved opcodes, numbers 254 \(0xfe\) and 255 \(0xff\), have the mnemonics _impdep1_ and _impdep2_, respectively. These instructions are intended to provide "back doors" or traps to implementation-specific functionality implemented in software and hardware, respectively. The third reserved opcode, number 202 \(0xca\), has the mnemonic _breakpoint_ and is intended to be used by debuggers to implement breakpoints.

Although these opcodes have been reserved, they may be used only inside a Java Virtual Machine implementation. They cannot appear in valid `class` files. Tools such as debuggers or JIT code generators \([§2.13](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.13)\) that might directly interact with Java Virtual Machine code that has been already loaded and executed may encounter these opcodes. Such tools should attempt to behave gracefully if they encounter any of these reserved instructions.

### 6.3. Virtual Machine Errors

A Java Virtual Machine implementation throws an object that is an instance of a subclass of the class `VirtualMachineError` when an internal error or resource limitation prevents it from implementing the semantics described in this chapter. This specification cannot predict where internal errors or resource limitations may be encountered and does not mandate precisely when they can be reported. Thus, any of the `VirtualMachineError` subclasses defined below may be thrown at any time during the operation of the Java Virtual Machine:

* `InternalError`: An internal error has occurred in the Java Virtual Machine implementation because of a fault in the software implementing the virtual machine, a fault in the underlying host system software, or a fault in the hardware. This error is delivered asynchronously \([§2.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.10)\) when it is detected and may occur at any point in a program.
* `OutOfMemoryError`: The Java Virtual Machine implementation has run out of either virtual or physical memory, and the automatic storage manager was unable to reclaim enough memory to satisfy an object creation request.
* `StackOverflowError`: The Java Virtual Machine implementation has run out of stack space for a thread, typically because the thread is doing an unbounded number of recursive invocations as a result of a fault in the executing program.
* `UnknownError`: An exception or error has occurred, but the Java Virtual Machine implementation is unable to report the actual exception or error.

### 6.4. Format of Instruction Descriptions

Java Virtual Machine instructions are represented in this chapter by entries of the form shown below, in alphabetical order and each beginning on a new page.

#### mnemonic

**Operation**

Short description of the instruction

**Format**

  
_mnemonic_  
_operand1_  
_operand2_  
...  


**Forms**

_mnemonic_ = opcode

**Operand Stack**

..., _value1_, _value2_ →

..., _value3_

**Description**

A longer description detailing constraints on operand stack contents or constant pool entries, the operation performed, the type of the results, etc.

**Linking Exceptions**

If any linking exceptions may be thrown by the execution of this instruction, they are set off one to a line, in the order in which they must be thrown.

**Run-time Exceptions**

If any run-time exceptions can be thrown by the execution of an instruction, they are set off one to a line, in the order in which they must be thrown.

Other than the linking and run-time exceptions, if any, listed for an instruction, that instruction must not throw any run-time exceptions except for instances of `VirtualMachineError` or its subclasses.

**Notes**

Comments not strictly part of the specification of an instruction are set aside as notes at the end of the description.

Each cell in the instruction format diagram represents a single 8-bit byte. The instruction's _mnemonic_ is its name. Its opcode is its numeric representation and is given in both decimal and hexadecimal forms. Only the numeric representation is actually present in the Java Virtual Machine code in a `class` file.

Keep in mind that there are "operands" generated at compile time and embedded within Java Virtual Machine instructions, as well as "operands" calculated at run time and supplied on the operand stack. Although they are supplied from several different areas, all these operands represent the same thing: values to be operated upon by the Java Virtual Machine instruction being executed. By implicitly taking many of its operands from its operand stack, rather than representing them explicitly in its compiled code as additional operand bytes, register numbers, etc., the Java Virtual Machine's code stays compact.

Some instructions are presented as members of a family of related instructions sharing a single description, format, and operand stack diagram. As such, a family of instructions includes several opcodes and opcode mnemonics; only the family mnemonic appears in the instruction format diagram, and a separate forms line lists all member mnemonics and opcodes. For example, the Forms line for the _lconst\_&lt;l&gt;_ family of instructions, giving mnemonic and opcode information for the two instructions in that family \(_lconst\_0_ and _lconst\_1_\), is

_lconst\_0_ = 9 \(0x9\)

_lconst\_1_ = 10 \(0xa\)

In the description of the Java Virtual Machine instructions, the effect of an instruction's execution on the operand stack \([§2.6.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6.2)\) of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\) is represented textually, with the stack growing from left to right and each value represented separately. Thus,

..., _value1_, _value2_ →

..., _result_

shows an operation that begins by having _value2_ on top of the operand stack with _value1_ just beneath it. As a result of the execution of the instruction, _value1_ and _value2_ are popped from the operand stack and replaced by _result_ value, which has been calculated by the instruction. The remainder of the operand stack, represented by an ellipsis \(...\), is unaffected by the instruction's execution.

Values of types `long` and `double` are represented by a single entry on the operand stack.

In the First Edition of _The Java® Virtual Machine Specification_, values on the operand stack of types `long` and `double` were each represented in the stack diagram by two entries.

### 6.5. Instructions


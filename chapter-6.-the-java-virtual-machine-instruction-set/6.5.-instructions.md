# 6.5. Instructions \(a-d\)

#### _aaload_

**Operation**

Load `reference` from array

**Format**

  
_aaload_  


**Forms**

_aaload_ = 50 \(0x32\)

**Operand Stack**

..., _arrayref_, _index_ →

..., _value_

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `reference`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `reference` _value_ in the component of the array at _index_ is retrieved and pushed onto the operand stack.

**Run-time Exceptions**

If _arrayref_ is `null`, _aaload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _aaload_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### _aastore_

**Operation**

Store into `reference` array

**Format**

  
_aastore_  


**Forms**

_aastore_ = 83 \(0x53\)

**Operand Stack**

..., _arrayref_, _index_, _value_ →

...

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `reference`. The _index_ must be of type `int` and value must be of type `reference`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `reference` _value_ is stored as the component of the array at _index_.

At run time, the type of _value_ must be compatible with the type of the components of the array referenced by _arrayref_. Specifically, assignment of a value of reference type S \(source\) to an array component of reference type T \(target\) is allowed only if:

* If S is a class type, then:
  * If T is a class type, then S must be the same class as T, or S must be a subclass of T;
  * If T is an interface type, then S must implement interface T.
* If S is an interface type, then:
  * If T is a class type, then T must be `Object`.
  * If T is an interface type, then T must be the same interface as S or a superinterface of S.
* If S is an array type, namely, the type SC`[]`, that is, an array of components of type SC, then:
  * If T is a class type, then T must be `Object`.
  * If T is an interface type, then T must be one of the interfaces implemented by arrays \(JLS §4.10.3\).
  * If T is an array type TC`[]`, that is, an array of components of type TC, then one of the following must be true:
    * TC and SC are the same primitive type.
    * TC and SC are reference types, and type SC is assignable to TC by these run-time rules.

**Run-time Exceptions**

If _arrayref_ is `null`, _aastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _aastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

Otherwise, if _arrayref_ is not `null` and the actual type of _value_ is not assignment compatible \(JLS §5.2\) with the actual type of the components of the array, _aastore_ throws an `ArrayStoreException`.

#### _aconst\_null_

**Operation**

Push `null`

**Format**

  
_aconst\_null_  


**Forms**

_aconst\_null_ = 1 \(0x1\)

**Operand Stack**

... →

..., `null`

**Description**

Push the `null` object `reference` onto the operand stack.

**Notes**

The Java Virtual Machine does not mandate a concrete value for `null`.

#### _aload_

**Operation**

Load `reference` from local variable

**Format**

  
_aload_  
_index_  


**Forms**

_aload_ = 25 \(0x19\)

**Operand Stack**

... →

..., _objectref_

**Description**

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The local variable at _index_ must contain a `reference`. The _objectref_ in the local variable at _index_ is pushed onto the operand stack.

**Notes**

The _aload_ instruction cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the _astore_ instruction \([§_astore_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.astore)\) is intentional.

The _aload_ opcode can be used in conjunction with the _wide_ instruction \([§_wide_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.wide)\) to access a local variable using a two-byte unsigned index.

#### _aload\_&lt;n&gt;_

**Operation**

Load `reference` from local variable

**Format**

  
_aload\_&lt;n&gt;_  


**Forms**

_aload\_0_ = 42 \(0x2a\)

_aload\_1_ = 43 \(0x2b\)

_aload\_2_ = 44 \(0x2c\)

_aload\_3_ = 45 \(0x2d\)

**Operand Stack**

... →

..., _objectref_

**Description**

The &lt;_n_&gt; must be an index into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The local variable at &lt;_n_&gt; must contain a `reference`. The _objectref_ in the local variable at &lt;_n_&gt; is pushed onto the operand stack.

**Notes**

An _aload\_&lt;n&gt;_ instruction cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the corresponding _astore\_&lt;n&gt;_ instruction \([§_astore\_&lt;n&gt;_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.astore_n)\) is intentional.

Each of the _aload\_&lt;n&gt;_ instructions is the same as _aload_ with an _index_ of &lt;_n_&gt;, except that the operand &lt;_n_&gt; is implicit.

#### _anewarray_

**Operation**

Create new array of `reference`

**Format**

  
_anewarray_  
_indexbyte1_  
_indexbyte2_  


**Forms**

anewarray = 189 \(0xbd\)

**Operand Stack**

..., _count_ →

..., _arrayref_

**Description**

The _count_ must be of type `int`. It is popped off the operand stack. The _count_ represents the number of components of the array to be created. The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\), where the value of the index is \(_indexbyte1_ `<<` 8\) \| _indexbyte2_. The run-time constant pool item at that index must be a symbolic reference to a class, array, or interface type. The named class, array, or interface type is resolved \([§5.4.3.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.1)\). A new array with components of that type, of length _count_, is allocated from the garbage-collected heap, and a `reference` _arrayref_ to this new array object is pushed onto the operand stack. All components of the new array are initialized to `null`, the default value for `reference` types \([§2.4](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.4)\).

**Linking Exceptions**

During resolution of the symbolic reference to the class, array, or interface type, any of the exceptions documented in [§5.4.3.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.1) can be thrown.

**Run-time Exceptions**

Otherwise, if _count_ is less than zero, the _anewarray_ instruction throws a `NegativeArraySizeException`.

**Notes**

The _anewarray_ instruction is used to create a single dimension of an array of object references or part of a multidimensional array.

#### _areturn_

**Operation**

Return `reference` from method

**Format**

  
_areturn_  


**Forms**

_areturn_ = 176 \(0xb0\)

**Operand Stack**

..., _objectref_ →

\[empty\]

**Description**

The _objectref_ must be of type `reference` and must refer to an object of a type that is assignment compatible \(JLS §5.2\) with the type represented by the return descriptor \([§4.3.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.3.3)\) of the current method. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction \([§_monitorexit_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorexit)\) in the current thread. If no exception is thrown, _objectref_ is popped from the operand stack of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\) and pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

The interpreter then reinstates the frame of the invoker and returns control to the invoker.

**Run-time Exceptions**

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _areturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10) and if the first of those rules is violated during invocation of the current method, then _areturn_ throws an `IllegalMonitorStateException`.

#### _arraylength_

**Operation**

Get length of array

**Format**

  
_arraylength_  


**Forms**

_arraylength_ = 190 \(0xbe\)

**Operand Stack**

..., _arrayref_ →

..., _length_

**Description**

The _arrayref_ must be of type `reference` and must refer to an array. It is popped from the operand stack. The _length_ of the array it references is determined. That _length_ is pushed onto the operand stack as an `int`.

**Run-time Exceptions**

If the _arrayref_ is `null`, the _arraylength_ instruction throws a `NullPointerException`.

#### _astore_

**Operation**

Store `reference` into local variable

**Format**

  
_astore_  
_index_  


**Forms**

_astore_ = 58 \(0x3a\)

**Operand Stack**

..., _objectref_ →

...

**Description**

The _index_ is an unsigned byte that must be an index into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The _objectref_ on the top of the operand stack must be of type `returnAddress` or of type `reference`. It is popped from the operand stack, and the value of the local variable at _index_ is set to _objectref_.

**Notes**

The _astore_ instruction is used with an _objectref_ of type `returnAddress` when implementing the `finally` clause of the Java programming language \([§3.13](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.13)\).

The _aload_ instruction \([§_aload_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.aload)\) cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the _astore_ instruction is intentional.

The _astore_ opcode can be used in conjunction with the _wide_ instruction \([§_wide_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.wide)\) to access a local variable using a two-byte unsigned index.

#### _astore\_&lt;n&gt;_

**Operation**

Store `reference` into local variable

**Format**

  
_astore\_&lt;n&gt;_  


**Forms**

_astore\_0_ = 75 \(0x4b\)

_astore\_1_ = 76 \(0x4c\)

_astore\_2_ = 77 \(0x4d\)

_astore\_3_ = 78 \(0x4e\)

**Operand Stack**

..., _objectref_ →

...

**Description**

The &lt;_n_&gt; must be an index into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The _objectref_ on the top of the operand stack must be of type `returnAddress` or of type `reference`. It is popped from the operand stack, and the value of the local variable at &lt;_n_&gt; is set to _objectref_.

**Notes**

An _astore\_&lt;n&gt;_ instruction is used with an _objectref_ of type `returnAddress` when implementing the `finally` clauses of the Java programming language \([§3.13](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.13)\).

An _aload\_&lt;n&gt;_ instruction \([§_aload\_&lt;n&gt;_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.aload_n)\) cannot be used to load a value of type `returnAddress` from a local variable onto the operand stack. This asymmetry with the corresponding _astore\_&lt;n&gt;_ instruction is intentional.

Each of the _astore\_&lt;n&gt;_ instructions is the same as _astore_ with an _index_ of &lt;_n_&gt;, except that the operand &lt;_n_&gt; is implicit.

#### _athrow_

**Operation**

Throw exception or error

**Format**

  
_athrow_  


**Forms**

_athrow_ = 191 \(0xbf\)

**Operand Stack**

..., _objectref_ →

_objectref_

**Description**

The _objectref_ must be of type `reference` and must refer to an object that is an instance of class `Throwable` or of a subclass of `Throwable`. It is popped from the operand stack. The _objectref_ is then thrown by searching the current method \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\) for the first exception handler that matches the class of _objectref_, as given by the algorithm in [§2.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.10).

If an exception handler that matches _objectref_ is found, it contains the location of the code intended to handle this exception. The `pc` register is reset to that location, the operand stack of the current frame is cleared, _objectref_ is pushed back onto the operand stack, and execution continues.

If no matching exception handler is found in the current frame, that frame is popped. If the current frame represents an invocation of a `synchronized` method, the monitor entered or reentered on invocation of the method is exited as if by execution of a _monitorexit_ instruction \([§_monitorexit_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorexit)\). Finally, the frame of its invoker is reinstated, if such a frame exists, and the _objectref_ is rethrown. If no such frame exists, the current thread exits.

**Run-time Exceptions**

If _objectref_ is `null`, _athrow_ throws a `NullPointerException` instead of _objectref_.

Otherwise, if the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10), then if the method of the current frame is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _athrow_ throws an `IllegalMonitorStateException` instead of the object previously being thrown. This can happen, for example, if an abruptly completing `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10) and if the first of those rules is violated during invocation of the current method, then _athrow_ throws an `IllegalMonitorStateException` instead of the object previously being thrown.

**Notes**

The operand stack diagram for the _athrow_ instruction may be misleading: If a handler for this exception is matched in the current method, the _athrow_ instruction discards all the values on the operand stack, then pushes the thrown object onto the operand stack. However, if no handler is matched in the current method and the exception is thrown farther up the method invocation chain, then the operand stack of the method \(if any\) that handles the exception is cleared and _objectref_ is pushed onto that empty operand stack. All intervening frames from the method that threw the exception up to, but not including, the method that handles the exception are discarded.

#### _baload_

**Operation**

Load `byte` or `boolean` from array

**Format**

  
_baload_  


**Forms**

_baload_ = 51 \(0x33\)

**Operand Stack**

..., _arrayref_, _index_ →

..., _value_

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `byte` or of type `boolean`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `byte` _value_ in the component of the array at _index_ is retrieved, sign-extended to an `int` _value_, and pushed onto the top of the operand stack.

**Run-time Exceptions**

If _arrayref_ is `null`, _baload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _baload_ instruction throws an `ArrayIndexOutOfBoundsException`.

**Notes**

The _baload_ instruction is used to load values from both `byte` and `boolean` arrays. In Oracle's Java Virtual Machine implementation, `boolean` arrays - that is, arrays of type `T_BOOLEAN` \([§2.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.2), [§_newarray_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.newarray)\) - are implemented as arrays of 8-bit values. Other implementations may implement packed `boolean` arrays; the _baload_ instruction of such implementations must be used to access those arrays.

#### _bastore_

**Operation**

Store into `byte` or `boolean` array

**Format**

  
_bastore_  


**Forms**

_bastore_ = 84 \(0x54\)

**Operand Stack**

..., _arrayref_, _index_, _value_ →

...

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `byte` or of type `boolean`. The _index_ and the _value_ must both be of type `int`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `int` _value_ is truncated to a `byte` and stored as the component of the array indexed by _index_.

**Run-time Exceptions**

If _arrayref_ is `null`, _bastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _bastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

**Notes**

The _bastore_ instruction is used to store values into both `byte` and `boolean` arrays. In Oracle's Java Virtual Machine implementation, `boolean` arrays - that is, arrays of type `T_BOOLEAN` \([§2.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.2), [§_newarray_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.newarray)\) - are implemented as arrays of 8-bit values. Other implementations may implement packed `boolean` arrays; in such implementations the _bastore_ instruction must be able to store `boolean` values into packed `boolean` arrays as well as `byte` values into `byte` arrays.

#### _bipush_

**Operation**

Push `byte`

**Format**

  
_bipush_  
_byte_  


**Forms**

_bipush_ = 16 \(0x10\)

**Operand Stack**

... →

..., _value_

**Description**

The immediate _byte_ is sign-extended to an `int` _value_. That _value_ is pushed onto the operand stack.

#### _caload_

**Operation**

Load `char` from array

**Format**

  
_caload_  


**Forms**

_caload_ = 52 \(0x34\)

**Operand Stack**

..., _arrayref_, _index_ →

..., _value_

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `char`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The component of the array at _index_ is retrieved and zero-extended to an `int` _value_. That _value_ is pushed onto the operand stack.

**Run-time Exceptions**

If _arrayref_ is `null`, _caload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _caload_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### _castore_

**Operation**

Store into `char` array

**Format**

  
_castore_  


**Forms**

_castore_ = 85 \(0x55\)

**Operand Stack**

..., _arrayref_, _index_, _value_ →

...

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `char`. The _index_ and the _value_ must both be of type `int`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `int` _value_ is truncated to a `char` and stored as the component of the array indexed by _index_.

**Run-time Exceptions**

If _arrayref_ is `null`, _castore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _castore_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### _checkcast_

**Operation**

Check whether object is of given type

**Format**

  
_checkcast_  
_indexbyte1_  
_indexbyte2_  


**Forms**

_checkcast_ = 192 \(0xc0\)

**Operand Stack**

..., _objectref_ →

..., _objectref_

**Description**

The _objectref_ must be of type `reference`. The unsigned _indexbyte1_ and _indexbyte2_ are used to construct an index into the run-time constant pool of the current class \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\), where the value of the index is \(_indexbyte1_ `<<` 8\) \| _indexbyte2_. The run-time constant pool item at the index must be a symbolic reference to a class, array, or interface type.

If _objectref_ is `null`, then the operand stack is unchanged.

Otherwise, the named class, array, or interface type is resolved \([§5.4.3.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.1)\). If _objectref_ can be cast to the resolved class, array, or interface type, the operand stack is unchanged; otherwise, the _checkcast_ instruction throws a `ClassCastException`.

The following rules are used to determine whether an _objectref_ that is not `null` can be cast to the resolved type: if S is the class of the object referred to by _objectref_ and T is the resolved class, array, or interface type, _checkcast_ determines whether _objectref_ can be cast to type T as follows:

* If S is an ordinary \(nonarray\) class, then:
  * If T is a class type, then S must be the same class as T, or S must be a subclass of T;
  * If T is an interface type, then S must implement interface T.
* If S is an interface type, then:
  * If T is a class type, then T must be `Object`.
  * If T is an interface type, then T must be the same interface as S or a superinterface of S.
* If S is a class representing the array type SC`[]`, that is, an array of components of type SC, then:
  * If T is a class type, then T must be `Object`.
  * If T is an interface type, then T must be one of the interfaces implemented by arrays \(JLS §4.10.3\).
  * If T is an array type TC`[]`, that is, an array of components of type TC, then one of the following must be true:
    * TC and SC are the same primitive type.
    * TC and SC are reference types, and type SC can be cast to TC by recursive application of these rules.

**Linking Exceptions**

During resolution of the symbolic reference to the class, array, or interface type, any of the exceptions documented in [§5.4.3.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.4.3.1) can be thrown.

**Run-time Exception**

Otherwise, if _objectref_ cannot be cast to the resolved class, array, or interface type, the _checkcast_ instruction throws a `ClassCastException`.

**Notes**

The _checkcast_ instruction is very similar to the _instanceof_ instruction \([§_instanceof_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof)\). It differs in its treatment of `null`, its behavior when its test fails \(_checkcast_ throws an exception, _instanceof_ pushes a result code\), and its effect on the operand stack.

#### _d2f_

**Operation**

Convert `double` to `float`

**Format**

  
_d2f_  


**Forms**

_d2f_ = 144 \(0x90\)

**Operand Stack**

..., _value_ →

..., _result_

**Description**

The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\) resulting in _value_'. Then _value_' is converted to a `float` result using IEEE 754 round to nearest mode. The _result_ is pushed onto the operand stack.

Where an _d2f_ instruction is FP-strict \([§2.8.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.2)\), the result of the conversion is always rounded to the nearest representable value in the float value set \([§2.3.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.2)\).

Where an _d2f_ instruction is not FP-strict, the result of the conversion may be taken from the float-extended-exponent value set \([§2.3.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.2)\); it is not necessarily rounded to the nearest representable value in the float value set.

A finite _value_' too small to be represented as a `float` is converted to a zero of the same sign; a finite _value_' too large to be represented as a `float` is converted to an infinity of the same sign. A `double` NaN is converted to a `float` NaN.

**Notes**

The _d2f_ instruction performs a narrowing primitive conversion \(JLS §5.1.3\). It may lose information about the overall magnitude of _value_' and may also lose precision.

#### _d2i_

**Operation**

Convert `double` to `int`

**Format**

  
_d2i_  


**Forms**

_d2i_ = 142 \(0x8e\)

**Operand Stack**

..., _value_ →

..., _result_

**Description**

The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\) resulting in _value_'. Then _value_' is converted to an `int`. The result is pushed onto the operand stack:

* If the _value_' is NaN, the _result_ of the conversion is an `int` 0.
* Otherwise, if the _value_' is not an infinity, it is rounded to an integer value V, rounding towards zero using IEEE 754 round towards zero mode. If this integer value V can be represented as an `int`, then the result is the `int` value V.
* Otherwise, either the _value_' must be too small \(a negative value of large magnitude or negative infinity\), and the _result_ is the smallest representable value of type `int`, or the _value_' must be too large \(a positive value of large magnitude or positive infinity\), and the _result_ is the largest representable value of type `int`.

**Notes**

The _d2i_ instruction performs a narrowing primitive conversion \(JLS §5.1.3\). It may lose information about the overall magnitude of _value_' and may also lose precision.

#### _d2l_

**Operation**

Convert `double` to `long`

**Format**

  
_d2l_  


**Forms**

_d2l_ = 143 \(0x8f\)

**Operand Stack**

..., _value_ →

..., _result_

**Description**

The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\) resulting in _value_'. Then _value_' is converted to a `long`. The _result_ is pushed onto the operand stack:

* If the _value_' is NaN, the _result_ of the conversion is a `long` 0.
* Otherwise, if the _value_' is not an infinity, it is rounded to an integer value V, rounding towards zero using IEEE 754 round towards zero mode. If this integer value V can be represented as a `long`, then the _result_ is the `long` value V.
* Otherwise, either the _value_' must be too small \(a negative value of large magnitude or negative infinity\), and the _result_ is the smallest representable value of type `long`, or the _value_' must be too large \(a positive value of large magnitude or positive infinity\), and the _result_ is the largest representable value of type `long`.

**Notes**

The _d2l_ instruction performs a narrowing primitive conversion \(JLS §5.1.3\). It may lose information about the overall magnitude of _value_' and may also lose precision.

#### _dadd_

**Operation**

Add `double`

**Format**

  
_dadd_  


**Forms**

_dadd_ = 99 \(0x63\)

**Operand Stack**

..., _value1_, _value2_ →

..., _result_

**Description**

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and undergo value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value1_' and _value2_'. The `double` _result_ is _value1_' + _value2_'. The _result_ is pushed onto the operand stack.

The result of a _dadd_ instruction is governed by the rules of IEEE arithmetic:

* If either _value1_' or _value2_' is NaN, the result is NaN.
* The sum of two infinities of opposite sign is NaN.
* The sum of two infinities of the same sign is the infinity of that sign.
* The sum of an infinity and any finite value is equal to the infinity.
* The sum of two zeroes of opposite sign is positive zero.
* The sum of two zeroes of the same sign is the zero of that sign.
* The sum of a zero and a nonzero finite value is equal to the nonzero value.
* The sum of two nonzero finite values of the same magnitude and opposite sign is positive zero.
* In the remaining cases, where neither operand is an infinity, a zero, or NaN and the values have the same sign or have different magnitudes, the sum is computed and rounded to the nearest representable value using IEEE 754 round to nearest mode. If the magnitude is too large to represent as a `double`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `double`, we say the operation underflows; the result is then a zero of appropriate sign.

The Java Virtual Machine requires support of gradual underflow as defined by IEEE 754. Despite the fact that overflow, underflow, or loss of precision may occur, execution of a _dadd_ instruction never throws a run-time exception.

#### _daload_

**Operation**

Load `double` from array

**Format**

  
_daload_  


**Forms**

_daload_ = 49 \(0x31\)

**Operand Stack**

..., _arrayref_, _index_ →

..., _value_

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `double`. The _index_ must be of type `int`. Both _arrayref_ and _index_ are popped from the operand stack. The `double` value in the component of the array at _index_ is retrieved and pushed onto the operand stack.

**Run-time Exceptions**

If _arrayref_ is `null`, _daload_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _daload_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### _dastore_

**Operation**

Store into `double` array

**Format**

  
_dastore_  


**Forms**

_dastore_ = 82 \(0x52\)

**Operand Stack**

..., _arrayref_, _index_, _value_ →

...

**Description**

The _arrayref_ must be of type `reference` and must refer to an array whose components are of type `double`. The _index_ must be of type `int`, and value must be of type `double`. The _arrayref_, _index_, and _value_ are popped from the operand stack. The `double` _value_ undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value_', which is stored as the component of the array indexed by _index_.

**Run-time Exceptions**

If _arrayref_ is `null`, _dastore_ throws a `NullPointerException`.

Otherwise, if _index_ is not within the bounds of the array referenced by _arrayref_, the _dastore_ instruction throws an `ArrayIndexOutOfBoundsException`.

#### _dcmp&lt;op&gt;_

**Operation**

Compare `double`

**Format**

  
_dcmp&lt;op&gt;_  


**Forms**

_dcmpg_ = 152 \(0x98\)

_dcmpl_ = 151 \(0x97\)

**Operand Stack**

..., _value1_, _value2_ →

..., _result_

**Description**

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and undergo value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value1_' and _value2_'. A floating-point comparison is performed:

* If _value1_' is greater than _value2_', the `int` value 1 is pushed onto the operand stack.
* Otherwise, if _value1_' is equal to _value2_', the `int` value 0 is pushed onto the operand stack.
* Otherwise, if _value1_' is less than _value2_', the `int` value -1 is pushed onto the operand stack.
* Otherwise, at least one of _value1_' or _value2_' is NaN. The _dcmpg_ instruction pushes the `int` value 1 onto the operand stack and the _dcmpl_ instruction pushes the `int` value -1 onto the operand stack.

Floating-point comparison is performed in accordance with IEEE 754. All values other than NaN are ordered, with negative infinity less than all finite values and positive infinity greater than all finite values. Positive zero and negative zero are considered equal.

**Notes**

The _dcmpg_ and _dcmpl_ instructions differ only in their treatment of a comparison involving NaN. NaN is unordered, so any `double` comparison fails if either or both of its operands are NaN. With both _dcmpg_ and _dcmpl_ available, any `double` comparison may be compiled to push the same _result_ onto the operand stack whether the comparison fails on non-NaN values or fails because it encountered a NaN. For more information, see [§3.5](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.5).

#### _dconst\_&lt;d&gt;_

**Operation**

Push `double`

**Format**

  
_dconst\_&lt;d&gt;_  


**Forms**

_dconst\_0_ = 14 \(0xe\)

_dconst\_1_ = 15 \(0xf\)

**Operand Stack**

... →

..., &lt;_d_&gt;

**Description**

Push the `double` constant &lt;_d_&gt; \(0.0 or 1.0\) onto the operand stack.

#### _ddiv_

**Operation**

Divide `double`

**Format**

  
_ddiv_  


**Forms**

_ddiv_ = 111 \(0x6f\)

**Operand Stack**

..., _value1_, _value2_ →

..., _result_

**Description**

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and undergo value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value1_' and _value2_'. The `double` _result_ is _value1_' / _value2_'. The _result_ is pushed onto the operand stack.

The result of a _ddiv_ instruction is governed by the rules of IEEE arithmetic:

* If either _value1_' or _value2_' is NaN, the result is NaN.
* If neither _value1_' nor _value2_' is NaN, the sign of the result is positive if both values have the same sign, negative if the values have different signs.
* Division of an infinity by an infinity results in NaN.
* Division of an infinity by a finite value results in a signed infinity, with the sign-producing rule just given.
* Division of a finite value by an infinity results in a signed zero, with the sign-producing rule just given.
* Division of a zero by a zero results in NaN; division of zero by any other finite value results in a signed zero, with the sign-producing rule just given.
* Division of a nonzero finite value by a zero results in a signed infinity, with the sign-producing rule just given.
* In the remaining cases, where neither operand is an infinity, a zero, or NaN, the quotient is computed and rounded to the nearest `double` using IEEE 754 round to nearest mode. If the magnitude is too large to represent as a `double`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `double`, we say the operation underflows; the result is then a zero of appropriate sign.

The Java Virtual Machine requires support of gradual underflow as defined by IEEE 754. Despite the fact that overflow, underflow, division by zero, or loss of precision may occur, execution of a _ddiv_ instruction never throws a run-time exception.

#### _dload_

**Operation**

Load `double` from local variable

**Format**

  
_dload_  
_index_  


**Forms**

_dload_ = 24 \(0x18\)

**Operand Stack**

... →

..., _value_

**Description**

The _index_ is an unsigned byte. Both _index_ and _index_+1 must be indices into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The local variable at _index_ must contain a `double`. The _value_ of the local variable at _index_ is pushed onto the operand stack.

**Notes**

The _dload_ opcode can be used in conjunction with the _wide_ instruction \([§_wide_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.wide)\) to access a local variable using a two-byte unsigned index.

#### _dload\_&lt;n&gt;_

**Operation**

Load `double` from local variable

**Format**

  
_dload\_&lt;n&gt;_  


**Forms**

_dload\_0_ = 38 \(0x26\)

_dload\_1_ = 39 \(0x27\)

_dload\_2_ = 40 \(0x28\)

_dload\_3_ = 41 \(0x29\)

**Operand Stack**

... →

..., _value_

**Description**

Both &lt;_n_&gt; and &lt;_n_&gt;+1 must be indices into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The local variable at &lt;_n_&gt; must contain a `double`. The _value_ of the local variable at &lt;_n_&gt; is pushed onto the operand stack.

**Notes**

Each of the _dload\_&lt;n&gt;_ instructions is the same as _dload_ with an _index_ of &lt;_n_&gt;, except that the operand &lt;_n_&gt; is implicit.

#### _dmul_

**Operation**

Multiply `double`

**Format**

  
_dmul_  


**Forms**

_dmul_ = 107 \(0x6b\)

**Operand Stack**

..., _value1_, _value2_ →

..., _result_

**Description**

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and undergo value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value1_' and _value2_'. The `double` result is _value1_' \* _value2_'. The _result_ is pushed onto the operand stack.

The result of a _dmul_ instruction is governed by the rules of IEEE arithmetic:

* If either _value1_' or _value2_' is NaN, the result is NaN.
* If neither _value1_' nor _value2_' is NaN, the sign of the result is positive if both values have the same sign and negative if the values have different signs.
* Multiplication of an infinity by a zero results in NaN.
* Multiplication of an infinity by a finite value results in a signed infinity, with the sign-producing rule just given.
* In the remaining cases, where neither an infinity nor NaN is involved, the product is computed and rounded to the nearest representable value using IEEE 754 round to nearest mode. If the magnitude is too large to represent as a `double`, we say the operation overflows; the result is then an infinity of appropriate sign. If the magnitude is too small to represent as a `double`, we say the operation underflows; the result is then a zero of appropriate sign.

The Java Virtual Machine requires support of gradual underflow as defined by IEEE 754. Despite the fact that overflow, underflow, or loss of precision may occur, execution of a _dmul_ instruction never throws a run-time exception.

#### _dneg_

**Operation**

Negate `double`

**Format**

  
_dneg_  


**Forms**

_dneg_ = 119 \(0x77\)

**Operand Stack**

..., _value_ →

..., _result_

**Description**

The value must be of type `double`. It is popped from the operand stack and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value_'. The `double` _result_ is the arithmetic negation of _value_'. The _result_ is pushed onto the operand stack.

For `double` values, negation is not the same as subtraction from zero. If `x` is `+0.0`, then `0.0-x` equals `+0.0`, but `-x` equals `-0.0`. Unary minus merely inverts the sign of a `double`.

Special cases of interest:

* If the operand is NaN, the result is NaN \(recall that NaN has no sign\).
* If the operand is an infinity, the result is the infinity of opposite sign.
* If the operand is a zero, the result is the zero of opposite sign.

#### _drem_

**Operation**

Remainder `double`

**Format**

  
_drem_  


**Forms**

_drem_ = 115 \(0x73\)

**Operand Stack**

..., _value1_, _value2_ →

..., _result_

**Description**

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and undergo value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value1_' and _value2_'. The _result_ is calculated and pushed onto the operand stack as a `double`.

The result of a _drem_ instruction is not the same as that of the so-called remainder operation defined by IEEE 754. The IEEE 754 "remainder" operation computes the remainder from a rounding division, not a truncating division, and so its behavior is _not_ analogous to that of the usual integer remainder operator. Instead, the Java Virtual Machine defines _drem_ to behave in a manner analogous to that of the Java Virtual Machine integer remainder instructions \(_irem_ and _lrem_\); this may be compared with the C library function `fmod`.

The result of a _drem_ instruction is governed by these rules:

* If either _value1_' or _value2_' is NaN, the result is NaN.
* If neither _value1_' nor _value2_' is NaN, the sign of the result equals the sign of the dividend.
* If the dividend is an infinity or the divisor is a zero or both, the result is NaN.
* If the dividend is finite and the divisor is an infinity, the result equals the dividend.
* If the dividend is a zero and the divisor is finite, the result equals the dividend.
* In the remaining cases, where neither operand is an infinity, a zero, or NaN, the floating-point remainder _result_ from a dividend _value1_' and a divisor _value2_' is defined by the mathematical relation _result_ = _value1_' - \(_value2_' \* _q_\), where _q_ is an integer that is negative only if _value1_' / _value2_' is negative, and positive only if _value1_' / _value2_' is positive, and whose magnitude is as large as possible without exceeding the magnitude of the true mathematical quotient of _value1_' and _value2_'.

Despite the fact that division by zero may occur, evaluation of a _drem_ instruction never throws a run-time exception. Overflow, underflow, or loss of precision cannot occur.

**Notes**

The IEEE 754 remainder operation may be computed by the library routine `Math.IEEEremainder`.

#### _dreturn_

**Operation**

Return `double` from method

**Format**

  
_dreturn_  


**Forms**

_dreturn_ = 175 \(0xaf\)

**Operand Stack**

..., _value_ →

\[empty\]

**Description**

The current method must have return type `double`. The _value_ must be of type `double`. If the current method is a `synchronized` method, the monitor entered or reentered on invocation of the method is updated and possibly exited as if by execution of a _monitorexit_ instruction \([§_monitorexit_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorexit)\) in the current thread. If no exception is thrown, _value_ is popped from the operand stack of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\) and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value_'. The _value_' is pushed onto the operand stack of the frame of the invoker. Any other values on the operand stack of the current method are discarded.

The interpreter then returns control to the invoker of the method, reinstating the frame of the invoker.

**Run-time Exceptions**

If the Java Virtual Machine implementation does not enforce the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10), then if the current method is a `synchronized` method and the current thread is not the owner of the monitor entered or reentered on invocation of the method, _dreturn_ throws an `IllegalMonitorStateException`. This can happen, for example, if a `synchronized` method contains a _monitorexit_ instruction, but no _monitorenter_ instruction, on the object on which the method is synchronized.

Otherwise, if the Java Virtual Machine implementation enforces the rules on structured locking described in [§2.11.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10) and if the first of those rules is violated during invocation of the current method, then _dreturn_ throws an `IllegalMonitorStateException`.

#### _dstore_

**Operation**

Store `double` into local variable

**Format**

  
_dstore_  
_index_  


**Forms**

_dstore_ = 57 \(0x39\)

**Operand Stack**

..., _value_ →

...

**Description**

The _index_ is an unsigned byte. Both _index_ and _index_+1 must be indices into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value_'. The local variables at _index_ and _index_+1 are set to _value_'.

**Notes**

The _dstore_ opcode can be used in conjunction with the _wide_ instruction \([§_wide_](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.wide)\) to access a local variable using a two-byte unsigned index.

#### _dstore\_&lt;n&gt;_

**Operation**

Store `double` into local variable

**Format**

  
_dstore\_&lt;n&gt;_  


**Forms**

_dstore\_0_ = 71 \(0x47\)

_dstore\_1_ = 72 \(0x48\)

_dstore\_2_ = 73 \(0x49\)

_dstore\_3_ = 74 \(0x4a\)

**Operand Stack**

..., _value_ →

...

**Description**

Both &lt;_n_&gt; and &lt;_n_&gt;+1 must be indices into the local variable array of the current frame \([§2.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.6)\). The _value_ on the top of the operand stack must be of type `double`. It is popped from the operand stack and undergoes value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value_'. The local variables at &lt;_n_&gt; and &lt;_n_&gt;+1 are set to _value_'.

**Notes**

Each of the _dstore\_&lt;n&gt;_ instructions is the same as _dstore_ with an _index_ of &lt;_n_&gt;, except that the operand &lt;_n_&gt; is implicit.

#### _dsub_

**Operation**

Subtract `double`

**Format**

  
_dsub_  


**Forms**

_dsub_ = 103 \(0x67\)

**Operand Stack**

..., _value1_, _value2_ →

..., _result_

**Description**

Both _value1_ and _value2_ must be of type `double`. The values are popped from the operand stack and undergo value set conversion \([§2.8.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.8.3)\), resulting in _value1_' and _value2_'. The `double` _result_ is _value1_' - _value2_'. The _result_ is pushed onto the operand stack.

For `double` subtraction, it is always the case that `a-b` produces the same result as `a+(-b)`. However, for the _dsub_ instruction, subtraction from zero is not the same as negation, because if `x` is `+0.0`, then `0.0-x` equals `+0.0`, but `-x` equals `-0.0`.

The Java Virtual Machine requires support of gradual underflow as defined by IEEE 754. Despite the fact that overflow, underflow, or loss of precision may occur, execution of a _dsub_ instruction never throws a run-time exception.

#### _dup_

**Operation**

Duplicate the top operand stack value

**Format**

  
_dup_  


**Forms**

_dup_ = 89 \(0x59\)

**Operand Stack**

..., _value_ →

..., _value_, _value_

**Description**

Duplicate the top value on the operand stack and push the duplicated value onto the operand stack.

The _dup_ instruction must not be used unless _value_ is a value of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

#### _dup\_x1_

**Operation**

Duplicate the top operand stack value and insert two values down

**Format**

  
_dup\_x1_  


**Forms**

_dup\_x1_ = 90 \(0x5a\)

**Operand Stack**

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

**Description**

Duplicate the top value on the operand stack and insert the duplicated value two values down in the operand stack.

The _dup\_x1_ instruction must not be used unless both _value1_ and _value2_ are values of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

#### _dup\_x2_

**Operation**

Duplicate the top operand stack value and insert two or three values down

**Format**

  
_dup\_x2_  


**Forms**

_dup\_x2_ = 91 \(0x5b\)

**Operand Stack**

Form 1:

..., _value3_, _value2_, _value1_ →

..., _value1_, _value3_, _value2_, _value1_

where _value1_, _value2_, and _value3_ are all values of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

Form 2:

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

where _value1_ is a value of a category 1 computational type and _value2_ is a value of a category 2 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

**Description**

Duplicate the top value on the operand stack and insert the duplicated value two or three values down in the operand stack.

#### _dup2_

**Operation**

Duplicate the top one or two operand stack values

**Format**

  
_dup2_  


**Forms**

_dup2_ = 92 \(0x5c\)

**Operand Stack**

Form 1:

..., _value2_, _value1_ →

..., _value2_, _value1_, _value2_, _value1_

where both _value1_ and _value2_ are values of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

Form 2:

..., _value_ →

..., _value_, _value_

where _value_ is a value of a category 2 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

**Description**

Duplicate the top one or two values on the operand stack and push the duplicated value or values back onto the operand stack in the original order.

#### _dup2\_x1_

**Operation**

Duplicate the top one or two operand stack values and insert two or three values down

**Format**

  
_dup2\_x1_  


**Forms**

_dup2\_x1_ = 93 \(0x5d\)

**Operand Stack**

Form 1:

..., _value3_, _value2_, _value1_ →

..., _value2_, _value1_, _value3_, _value2_, _value1_

where _value1_, _value2_, and _value3_ are all values of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

Form 2:

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

where _value1_ is a value of a category 2 computational type and _value2_ is a value of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

**Description**

Duplicate the top one or two values on the operand stack and insert the duplicated values, in the original order, one value beneath the original value or values in the operand stack.

#### _dup2\_x2_

**Operation**

Duplicate the top one or two operand stack values and insert two, three, or four values down

**Format**

  
_dup2\_x2_  


**Forms**

_dup2\_x2_ = 94 \(0x5e\)

**Operand Stack**

Form 1:

..., _value4_, _value3_, _value2_, _value1_ →

..., _value2_, _value1_, _value4_, _value3_, _value2_, _value1_

where _value1_, _value2_, _value3_, and _value4_ are all values of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

Form 2:

..., _value3_, _value2_, _value1_ →

..., _value1_, _value3_, _value2_, _value1_

where _value1_ is a value of a category 2 computational type and _value2_ and _value3_ are both values of a category 1 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

Form 3:

..., _value3_, _value2_, _value1_ →

..., _value2_, _value1_, _value3_, _value2_, _value1_

where _value1_ and _value2_ are both values of a category 1 computational type and _value3_ is a value of a category 2 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

Form 4:

..., _value2_, _value1_ →

..., _value1_, _value2_, _value1_

where _value1_ and _value2_ are both values of a category 2 computational type \([§2.11.1](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.1)\).

**Description**

Duplicate the top one or two values on the operand stack and insert the duplicated values, in the original order, into the operand stack.


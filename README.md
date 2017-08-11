# Magik (programming language)
Magik is an object-oriented programming language that supports multiple inheritance, polymorphism and is dynamically typed. It was designed implemented in 1989 by Arthur Chance, of Smallworld Systems Ltd, as part of Smallworld Geographical Information System (GIS). Following Smallworld's acquisition in 2000, Magik is now is provided by GE Energy, still as part of its Smallworld technology platform.

## Hello World example
The following is an example of the Hello world program written in Magik:

 `write("Hello World!")`

## Comments
 Magik uses the # token to mark sections of code as comments:

`# This is a comment.`

## Assignments
 Magik uses the << operator to make assignments:
```
   a << 1.234
   b << b + a
   c << "foo" + "bar" # Concat strings
   ```
 For clarity, this notation is read as "a becomes 1.234" or "b becomes b plus a". This terminology separates assignment from comparison.

 Magik also supports a compressed variation of this operator that works in a similar way to those found in C:

`   b +<< a # Equivalent to b << b + a`
 To print a variable you can use the following command
```
  a << "hello"
  write(a)
  ```

## Symbols
 As well as conventional data types such as integers, floats and strings Magik also implements symbols. Symbols are a special token data type that are used extensively throughout Magik to uniquely identify objects. They are represented by a colon followed by a string of characters. Symbols can be escaped using the vertical bar character. For example:

```
   a << :hello  # whenever :hello is encountered, it is the same instance
   b << :|hello world|`
```

## Dynamic typing
 Magik variables are not typed as they are in say C# and can reference different objects at runtime. Everything in Magik is an object (there is no distinction between objects and primitive types such as integers):
```
   a << 1.2     # a floating point number is assigned to variable 'a'
   a << "1.2"   # later, a string is assigned to variable 'a'
```

## Objects
 Objects are implemented in Magik using exemplars. Exemplars have similarities to classes in other programming languages such as Java, but with important differences. Magik supports multiple inheritance, and mixins (which implement functionality with no data). New instances are made by cloning an existing instance (which will typically be the exemplar but does not have to be).

 New exemplars are created using the statement def_slotted_exemplar(), for example:
```
   def_slotted_exemplar(:my_object,
   {
     {:slot_a, 34},
     {:slot_b, "hello"}
   }, {:parent_object_a, :parent_object_b})
   ```

 This code fragment will define a new exemplar called my_object that has two slots (or fields) called slot_a (pre-initialized to 34) and slot_b (pre-initialised to "hello") that inherits from two existing exemplars called parent_object_a and parent_object_b.

## Comparison
 Magik implements all usual logical operators (=, <, <=, >, >=, ~=/<>) for comparison, as well as a few unusual ones. The `_is` and `_isnt` operators are used for comparing specific instances of objects, or object references rather than values.

 For example:
```
   a << "hello"
   b << "hello"

   a = b # returns True (_true) because the values of a and b are equal
   a _is b # returns False (_false) because a is not the same instance as b

   a << "hello"
   b << a = b # returns True (_true) because the values of a and b are equal
   a _is b # returns True (_true) because b was assigned the specific instance of the same object as a, rather than the value of a.
   ```
  If sentences
```
  x << 1
   y << 2
   z << _true
   ```
```
  _if x < y _then
      #do something
  _endif

  _elif x = y _then

  _else

  _endif
  ```
  Logical and, or inside if sentences are `_andif` and `_orif`
```
  _if z _is _true _andif z _isnt _unset _then

  _endif
  ```

## Methods
 Methods are defined on exemplars using the statements `_method` and `_endmethod`:
```
  _method my_object.my_method(a, b)
     _return a + b
   _endmethod
```
 It is convention to supply two methods new() (to create a new instance) and init() (to initialize an instance).
```
   # New method
   _method person.new(name, age)
     _return _clone.init(name, age)
   _endmethod

   # Initialize method.
   _private _method person.init(name, age)
      # Call the parent implementation.
      _super.init(name, age)
      # Initialise the slots.
      .name << name
      .age << age
     _return _self
   _endmethod
   ```

 The `_clone` creates a physical copy of the person object. The `_super` statement allows objects to invoke an implementation of a method on the parent exemplar. Objects can reference themselves using the `_self` statement. An object's slots are accessed and assigned using a dot notation.

 Methods that are not part of the public interface of the object can be marked private using the `_private` statement. Private methods can only be called by `_self`, `_super` and `_clone`.

 Optional arguments can be declared using the` _optional` statement. Optional arguments that are not passed are assigned by Magik to the special object `_unset` (the equivalent of null). The `_gather` statement can be used to declare a list of optional arguments.
```
   _method my_object.my_method(_gather values)     
   _endmethod
   ```

### Conflict Methods

If you define my_method() on the classes or mixins a and b, and you have a third class c inheriting from a and b, a conflict method on c is generated automatically.
```
_method a.my_method()
    # do something
_endmethod
$
_method b.my_method()
    # do something else
_endmethod
$
def_slotted_exemplar(:c,{},{:a,:b}
$
```

Calling my_method() on c will raise an error condition unless you implement my_method() on c as well. A typical implementation
```
for such a conflict resolution would be:
_method c.my_method()
    ## conflict resolution
    _super(a).my_method()
    _super(b).my_method()
_endmethod
$
```

### Abstract Methods
You implement some functionality on a mixin (or a slotted_exemplar), intended to be used by a subclass. It may happen, that in your code there are some calls to methods, which have to be implemented on the actual child class using your funcitionality, but it makes no sense to implement them on your mixin/slotted_exemplar. In this case you may implement them as `_abstract`. This gives you the possibility to give them a `_pragma` - Statement and a public comment, describing what that method should do. For example
```
_abstract _method my_class.my_method
    ## whatever this method should do...
_endmethod
$
```

Execpt for the conflict behaviour this implementation is equivalent to the following code
```
_method my_class.my_method
    ## whatever this method should do...
    condition.raise(:subclass_should_implement,
            :name,:my_method,
            :class,_self.class_name)
_endmethod
$
```

Note that `_self.class_name` normally is not :my_class, but is the class name of the object my_method is called on during runtime (normally a child class of :my_class).
In fact there is no technical need to implement abstract methods. But if you get used implementing them, you have a formal desciption of the implementation needs of child classes. And in case you have forgotten the implementation of the method on the child class, the raised condition :subclass_should_implement together with the public comment of your abstract method normally gives you a better hint for fixing a bug like that, than the condition :does_not_understand, which would have been raised otherwise.

## Iteration
 In Magik the `_for`, `_over`, `_loop` and `_endloop` statements allow iteration.
```
   _method my_object.my_method(_gather values)
     total << 0.0
     _for a _over values.elements()
     _loop
        total +<< a
     _endloop
     _return total
   _endmethod

   m << my_object.new()
   x << m.my_method(1.0, 2, 3.0, 4) # x = 10.0
   ```

 Here values.elements() is an iterator which helps to iterate the values.

 In Magik generator methods are called iterator methods. New iterator methods can be defined using the `_iter` and `_loopbody` statements:
```
   _iter _method my_object.even_elements()
     _for a _over _self.elements()
     _loop
       _if a.even? _is _true
       _then
          _loopbody(a)       
       _endif
     _endloop
   _endmethod
   ```

There is no while sentence in magik. To create a similar loop sentence could be used `_loop` and a `_leave` (is equivalent to break) when the condition happenned:

```
 x << 0
  _loop
    _if x < 5
      write(x)
      x +<< 1  # is equal to += in others programming languages
    _endif
  _endloop
```

## Global variables
To declared a global variables

`_global busy = _true`

But other method doesn't see this variable as a global. It is necessary declared inside the method because, if not, the magik language will see the variable as local variable

```
_method my_class.my_method()
	 _global busy
    write(busy)  #answer true
_endmethod

_method my_class.my_method1()
    write(busy)  #answer unset - is equivalent to creat a "_local busy"
_endmethod
```

## Procedures
 Magik also supports functions called procedures. Procedures are also objects and are declared using the` _proc` and `_endproc` statements. Procedures are assigned to variables which may then be invoked:
```
  my_procedure << _proc @my_procedure(a, b, c)
     _return a + b + c
   _endproc

   x << my_procedure(1, 2, 3) # x = 6
   ```

  The most common used of procedure is:
```
   _for i _over 1.upto(6)
    _loop
        _proc()
            foo()
        _endproc
    _endloop
  ```

## Threads
In computer science, a thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler, which is typically a part of the operating system.[1] The implementation of threads and processes differs between operating systems, but in most cases a thread is a component of a process. Multiple threads can exist within one process, executing concurrently and sharing resources such as memory, while different processes do not share these resources. In particular, the threads of a process share its executable code and the values of its variables at any given time.

In magik sentence a thread is created with a procedure in global variables

```
_for ut _over gis_program_manager.databases[:electric].user_tables()
_loop
    _proc()
        test!fcsi_test_queue()
    _endproc.fork_at(_thisthread.high_background_priority,_self)
_endloop
$

_pragma(classify_level=restricted, topic={fcsi,osm})
_global test!fcsi_test_queue <<
    _proc()
        #_for a _over ut.fast_elements()
        _for a _over 1.upto(2)
        _loop
            write(_thisthread)
            write(a)
            _thisthread.sleep(500)
        _endloop
    _endproc
$
```

or in method that is more similar of other langueges
```
def_slotted_exemplar(:threadpoc,
    {
    }, {})
$

_method threadpoc.startThreads()
    _for j _over 1.upto(6)
    _loop
        write(_thisthread)
        threadpoc.method(:|runnable()|).value.fork_at(_thisthread.high_background_priority, j)
    _endloop
_endmethod

_method threadpoc.runnable(arg)
    _for i _over 1.upto(10)
    _loop
        write("Hello world! " + arg.write_string + " - " + (i).write_string)
        _thisthread.sleep(500)
    _endloop
_endmethod
$
```
## Try 
```
_for i _over 1.upto(10)
_loop @name
  _try _with error
    # do something
  _when error
    write("message")
    _leave @name
  _endtry
_endloop
```

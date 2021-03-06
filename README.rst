Notice
======

The scope of the project has been narrowed down and renamed to programmable
tuple. The repository has been moved to
`here <https://github.com/tschijnmo/programmabletuple>`_.

immutableclass
==============

Python metaclass for making instances of user-defined classes immutable

.. image:: https://travis-ci.org/tschijnmo/immutableclass.svg?branch=master
    :target: https://travis-ci.org/tschijnmo/immutableclass

This module provides a metaclass for making the instances of user-defined
classes immutable. Its basic functionality is modelled after
:py:cls:`collections.namedtuple`, but it offers more object-orientation and
programmability. Basically, here instances of immutable classes are frozen
once they are initialized. Any attempt to mutate the state of the instance
will cause an error. Otherwise they are designed to behave as similar to
the instances of the plain mutable classes as possible.

The basic motivation for this is to make code more secure and less error-prone.
Frequently in Python code, we have got structures (any structure) holding
references to instances of user-defined classes, which by default are all
mutable. But sometimes the correctness of the behaviour of the structure would
depend on the assumption that the objects that these references point to will
not be mutated. A solution to this problem is to make copies of the instances
into the structure rather than just hold a reference and share the actual
object. In this way, other parts of the code could safely mutate the states of
the instances without any undesirable side effect. However, the copying comes
with cost. For cases where the the objects referenced will almost definitely be
mutated, this cost is necessary. But for cases where the objects are unlikely
to be mutated in the majority of cases, the copying might cease to be an
economical choice. So the basic idea of this metaclass is that we could have a
means to enforce the references to be pointing the the same value, and we can
still make new instances out of the old ones if they really need to be mutated.
It can be construed as an attempt to make Python more functional while
maintaining its object-orientated aspect. The resulted immutable class
resembles a hybrid of the Haskell record and the user-defined class in Python.

Basic usage
-----------

Fields
^^^^^^

Since all the information about instances of an immutable class needs to be
given to the initializer, the arguments of the initializer uniquely define
values of the immutable class. Hence they are called the defining fields of the
class. Besides the defining fields, additional fields can be added to the class
instances to hold some other essential data. These fields are going to be
termed the data fields. This can be achieved by assigning a list of names to
the ``__fields__`` attribute of the class, in the same way as the ``__slots__``
attribute is used. And the actual value for the data fields can be set in the
initializer in the same way as normal. For example, to define an immutable
class for people to store their first and last name, and we would like the
instances to carry the full name with comma separation for alphabetization, we
can just define

.. code:: python

    class Person(metaclass=ImmutableClass):
        __fields__ = ['full_name']
        def __init__(self, first_name, last_name):
            self.full_name = ', '.join([last_name, first_name])

Then in this way, if we make an instance by running ``Person('John',
'Smith')``, the values of all the fields, defining fields and data fields, can
all be able to be retrieved by using the dot notation, like ``p.full_name``.
Note that if some fields are desired to be hold private, the same underscore
convention of python could be used. Just it is not advised to keep defining
attributes private.

Methods
^^^^^^^

Methods can also be defined for immutable classes with exactly the same syntax
as the normal mutable class. Just here the only place where ``self`` could be
mutated is in the ``__init__`` method, any attempt to mutate ``self`` would
cause an error in any other method. So the methods here should be ones that
concentrates more on the return value rather than mutating the state of the
object. Due to this apparent deviation from the classical Smalltalk-style
object-orientated programming, the methods could be clearly defined outside the
class as a normal function, and then then we can forward them into the class
for convenience. For instance, if we have got a class for symbolic mathematical
expressions and a function to compute the derivative with respect to a symbol,
we could do

.. code:: python

    def diff_expr(expr, symb):
        """Compute the derivative w.r.t. a symbol"""
        ... ...

    class Expr(metaclass=ImmutableClass):
        ... ...
        diff = diff_expr
        ... ...

In this way, to differentiate an expression ``e`` with respect to a symbol
``x``, we could do both ``e.diff(x)`` and ``diff_expr(e, x)``. It only needs to
be noted that for functions that is intended to be used as a method as well,
the argument to be used as ``self`` needs to be put in the first slot. Of
course methods can be kept in the class only as normal if it is desirable.

Non-destructive update
^^^^^^^^^^^^^^^^^^^^^^

Frequently we need values of user-defined class that is different from an
existing value by relatively small amount. With mutable class, frequently this
is achieved by mutating the instance. However, here the instances are no longer
mutable. So methods to update instances non-destructively are provided. Note
that these methods will return new instances with the field updated and leave
the original value intact.

Basically two methods are provided for this purpose, ``_update`` and
``_replace``. Both of them takes keyword arguments with the keys being the name
of the field to be updated and values being the new value. But for the
``_update`` method, only defining fields are able to be updated, and more
importantly, a new instance will be created **by using the updated defining
fields through the initializer**. At the same time, the ``_replace`` method
will just perform a plain replacement of a particular field without going
through the initializer again, and it works for both defining and data fields.

Both of these two methods are named with an initial underscore, this is not
only an attempt to be consistent with the named tuple in the standard library,
but an encourage to use them only in methods as well. Then then wrapping
methods could carry the actual semantics of the update operation.

Inheritance
^^^^^^^^^^^

Immutable classes can inherit from other immutable classes. And this
inheritance has been made to be as similar to the plain mutable classes as
possible. Instances of subclass are instances of the corresponding superclass
and has access to all the methods of the superclass. There is just one notable
difference, in the initializer, the built-in ``super`` function is not working
as before. To call the initializer of superclass, we can either use
``self.super().__init__`` instead, or we can name the superclass explicitly,
like ``SuperClass.__init__(self, args)``.

Miscellaneous
^^^^^^^^^^^^^

Instances of an immutable class with all the defining fields hashable are
hashable. The default hashing function is the default hashing of the tuple
formed by the class identity and the defining fields.

Instances are all picklable.

As the named tuple, classes of this metaclass will carry an ``_asdict`` method
to convert the instance to dictionary. The method comes with two keyword
arguments, ``full`` can be used to make the dictionary contain the data fields
as well, and ``ordered`` can be used to return an ordered dictionary instead.
Both of the two default to false.

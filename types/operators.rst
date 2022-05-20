.. index:: ! operator

Operators
=========

Arithmetic and bit operators can be applied even if the two operands do not have the same type.
For example, you can compute ``y = x + z``, where ``x`` is a ``uint8`` and ``z`` has
the type ``int32``. In these cases, the following mechanism will be used to determine
the type in which the operation is computed (this is important in case of overflow)
and the type of the operator's result:

1. If the type of the right operand can be implicitly converted to the type of the left
   operand, use the type of the left operand,
2. if the type of the left operand can be implicitly converted to the type of the right
   operand, use the type of the right operand,
3. otherwise, the operation is not allowed.

In case one of the operands is a :ref:`literal number <rational_literals>` it is first converted to its
"mobile type", which is the smallest type that can hold the value
(unsigned types of the same bit-width are considered "smaller" than the signed types).
If both are literal numbers, the operation is computed with arbitrary precision.

The operator's result type is the same as the type the operation is performed in,
except for comparison operators where the result is always ``bool``.

The operators ``**`` (exponentiation), ``<<``  and ``>>`` use the type of the
left operand for the operation and the result.

Ternary Operator
----------------
The ternary operator is used in expressions of the form ``<expression> ? <trueExpression> : <falseExpression>``.
It evaluates one of the latter two given expressions depending upon the result of the evaluation of the main ``<expression>``.
If ``<expression>`` evaluates to ``true``, then ``<trueExpression>`` will be evaluated, otherwise ``<falseExpression>`` is evaluated.

The result of the ternary operator does not have a rational number type, even if all of its operands are rational number literals.
The result type is determined from the types of the two operands in the same way as above, converting to their mobile type first if required.

As a consequence, ``255 + (true ? 1 : 0)`` will revert due to arithmetic overflow.
The reason is that ``(true ? 1 : 0)`` is of ``uint8`` type, which forces the addition to be performed in ``uint8`` as well,
and 256 exceeds the range allowed for this type.

Another consequence is that an expression like ``1.5 + 1.5`` is valid but ``1.5 + (true ? 1.5 : 2.5)`` is not.
This is because the former is a rational expression evaluated in unlimited precision and only its final value matters.
The latter involves a conversion of a fractional rational number to an integer, which is currently disallowed.


.. index:: assignment, lvalue, ! compound operators

Compound and Increment/Decrement Operators

如果 ``a`` 是一个 LValue（即一个变量或者其它可以被赋值的东西），以下运算符都可以使用简写：

``a += e`` 等同于 ``a = a + e``。其它运算符如 ``-=``， ``*=``， ``/=``， ``%=``， ``|=``， ``&=`` ， ``^=`` ， ``<<=`` 和 ``>>=``  都是如此定义的。
``a++`` 和 ``a--`` 分别等同于 ``a += 1`` 和 ``a -= 1``，但表达式本身的值等于 ``a`` 在计算之前的值。
与之相反， ``--a`` 和 ``++a`` 虽然最终 ``a`` 的结果与之前的表达式相同，但表达式的返回值是计算之后的值。

.. index:: !delete
.. _delete:

delete
----------

``delete a`` 的结果是将 ``a`` 类型初始值赋值给 ``a``。即对于整型变量来说，相当于 ``a = 0``，delete 也适用于数组，对于动态数组来说，是将重置为数组长度为0的数组，而对于静态数组来说，是将数组中的所有元素重置为初始值。对数组而言，``delete a[x]`` 仅删除数组索引 ``x`` 处的元素，其他的元素和长度不变，这以为着数组中留出了一个空位。如果打算删除项，映射可能是更好的选择。

如果对象  ``a``  是结构体，则将结构体中的所有属性(成员)重置。 

换句话说，在 ``delete a`` 之后 ``a`` 的值与在没有赋值的情况下声明 ``a`` 的情况相同，
但需要注意以下几点：

``delete`` 对整个映射是无效的（因为映射的键可以是任意的，通常也是未知的）。
因此在你删除一个结构体时，结果将重置所有的非映射属性（成员），这个过程是递归进行的，除非它们是映射。
然而，单个的键及其映射的值是可以被删除的。

理解 ``delete a`` 的效果就像是给 ``a`` 赋值很重要，换句话说，这相当于在 ``a`` 中存储了一个新的对象。

当 ``a`` 是应用变量时，我们可以看到这个区别，``delete a`` 它只会重置 ``a`` 本身，而不是更改它之前引用的值。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract DeleteLBC {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // 将 x 设为 0，并不影响数据
            delete data; // 将 data 设为 0，并不影响 x，因为它仍然有个副本
            uint[] storage y = dataArray;
            delete dataArray; 
            // 将 dataArray.length 设为 0，但由于 uint[] 是一个复杂的对象，y 也将受到影响，
            // 因为它是一个存储位置是 storage 的对象的别名。
            // 另一方面："delete y" 是非法的，引用了 storage 对象的局部变量只能由已有的 storage 对象赋值。
            assert(y.length == 0);
        }
    }
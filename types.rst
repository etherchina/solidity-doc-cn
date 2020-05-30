.. include:: glossaries.rst
.. index:: type

.. _types:

*****
类型
*****

Solidity 是一种静态类型语言，这意味着每个变量（状态变量和局部变量）都需要在编译时指定变量的类型。

Solidity 提供了几种基本类型，并且基本类型可以用来组合出复杂类型。

除此之外，类型之间可以在包含运算符号的表达式中进行交互。
关于各种运算符号，可以参考 :ref:`order` 。


“undefined”或“null”值的概念在Solidity中不存在，但是新声明的变量总是有一个 :ref:`默认值 <default-value>` ，具体的默认值跟类型相关。
要处理任何意外的值，应该使用 :ref:`错误处理 <assert-and-require>` 来恢复整个交易，或者返回一个带有第二个 ``bool`` 值的元组表示成功。


.. include:: types/value-types.rst

.. include:: types/reference-types.rst

.. include:: types/mapping-types.rst

.. include:: types/operators.rst

.. include:: types/conversion.rst


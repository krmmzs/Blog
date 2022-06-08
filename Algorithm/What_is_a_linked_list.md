# What is a linked list

## Why write this...

自从这个学期开始被张老师叫起来回答链表的具体实现提到了长度，张老师告知，链表需要有**长度**我觉很想写点什么。

## Main

我在大一的时候被leetcode上对递归反转链表的操作感到吃惊。

我脑中没有任何关于为什么可以进行recursion的知识结点，自然也无法进行演绎推理。

### 什么是链表

其实现在已经有很多的知识结点了。
我想从OO的角度去描述链表, 并用python代码来描述其具体实现。
具体实现只是帮助理解，因为具体实现会让你失去抽象带来的一般性。

链表是由第一个元素和列表的其余部分组成的。链表的其余部分本身就是一个链表--一个递归定义。空列表是链接列表的一个特例，它没有第一元素或其余部分。一个链表是一个序列：它有一个有限的长度（这里的长度不要想当然是一个变量），并支持通过索引选择元素。

我们现在可以实现一个具有相同行为的类。我将使用特殊的方法名来定义它的行为，使该类能够与 Python 中的内置 len 函数和元素选择操作符 (方括号或 operator.getitem) 一起工作。这些内置函数调用了一个类的特殊方法名：长度由 __len__ 计算，元素选择由 __getitem__ 计算。空链表由一个空元组表示，它的长度为 0，没有元素。

```python
class Link:
    """A linked list with a first element and the rest."""
    empty = ()
    def __init__(self, first, rest=empty):
        assert rest is Link.empty or isinstance(rest, Link)
        self.first = first
        self.rest = rest


    def __getitem__(self, i):
        if i == 0:
            return self.first
        else:
            return self.rest[i-1]


    def __len__(self):
        return 1 + len(self.rest)
```

\>>>s = Link(3, Link(4, Link(5)))
\>>>len(s)
3
\>>>s[1]
4

__len__ 和 __getitem__ 的定义实际上是递归的。当应用于一个用户定义的对象参数时，内置的 Python 函数 len 会调用一个叫做 __len__ 的方法。同样地，元素选择操作符调用了一个叫做 __getitem__ 的方法。因此，这两个方法的主体将间接地调用自己。对于 __len__，当 self.rest 评估为空元组，Link.empty，其长度为 0 时，会达到基本情况。

内置的isinstance函数返回第一个参数的类型是否是或继承于第二个参数。如果rest是一个Link实例或Link的某个子类的实例，isinstance(rest, Link)为真。

实现是完整的，但Link类的实例目前很难检查。我们还可以定义一个函数，将Link转换为一个字符串表达式。

```python
def link_expression(s):
        """Return a string that would evaluate to s."""
        if s.rest is Link.empty:
            rest = ''
        else:
            rest = ', ' + link_expression(s.rest)
        return 'Link({0}{1})'.format(s.first, rest)
```

---

递归函数特别适合于操作链表(因为链表本身的定义是递归的)。
接下来给个例子来展现如何递归处理链表

### 实现一个类似于列表推导式的system

```python
class Link:
    """A linked list.

    >>> s = Link(3, Link(4, Link(5)))
    >>> s
    Link(3, Link(4, Link(5)))
    >>> print(s)
    <3 4 5>
    >>> s.first
    3
    >>> s.rest
    Link(4, Link(5))
    >>> s.rest.first
    4
    >>> s.rest.first = 7
    >>> s
    Link(3, Link(7, Link(5)))
    >>> s.first = 6
    >>> s.rest.rest = Link.empty
    >>> s
    Link(6, Link(7))
    >>> print(s)
    <6 7>
    >>> print(s.rest)
    <7>
    >>> t = Link(1, Link(Link(2, Link(3)), Link(4)))
    >>> t
    Link(1, Link(Link(2, Link(3)), Link(4)))
    >>> print(t)
    <1 <2 3> 4>
    """

    empty = ()

    def __init__(self, first, rest=empty):
        assert rest is Link.empty or isinstance(rest, Link)
        self.first = first
        self.rest = rest

    def __repr__(self):
        if self.rest:
            rest_repr = ", " + repr(self.rest)
        else:
            rest_repr = ""
        return "Link(" + repr(self.first) + rest_repr + ")"

    def __str__(self):
        string = "<"
        while self.rest is not Link.empty:
            string += str(self.first) + " "
            self = self.rest
        return string + str(self.first) + ">"

square, odd = lambda x: x * x, lambda x: x % 2 == 1
list(map(square, filter(odd, range(1, 6))))  # [1, 9, 25]

```

```python
def range_link(start, end):
    """Return a Link containing consecutive integers from start to end.

    >>> range_link(3, 6)
    Link(3, Link(4, Link(5)))
    """
    if start >= end:
        return Link.empty
    else:
        return Link(start, range_link(start + 1, end))
```

这是实现 Range(如果对python不是很了解可以不管)

与其说是列表推导式，不如说是使用两个高阶函数：map_link 和 filter_link 从另一个链表生成。下面定义的 map_link 函数将一个函数 f 应用于一个链表 s 的每个元素，并构造一个包含结果的链表。

```python
def map_link(f, s):
    """Return a Link that contains f(x) for each x in Link s.

    >>> map_link(square, range_link(3, 6))
    Link(9, Link(16, Link(25)))
    """
    if s is Link.empty:
        return s
    else:
        return Link(f(s.first), map_link(f, s.rest))

```

filter_link 函数返回一个包含链表 s 中 f 返回真值的所有元素的链表。map_link 和 filter_link 的组合可以表达与列表推导式相同的逻辑。

```python

def filter_link(f, s):
    """Return a Link that contains only the elements x of Link s for which f(x)
    is a true value.

    >>> filter_link(odd, range_link(3, 6))
    Link(3, Link(5))
    """
    if s is Link.empty:
        return s
    filtered_rest = filter_link(f, s.rest)
    if f(s.first):
        return Link(s.first, filtered_rest)
    else:
        return filtered_rest


map_link(square, filter_link(odd, range_link(1, 6)))  # Link(1, Link(9, Link(25)))

```

---

### 启示

- 可以发现，其实基础的东西不一定简单。

- 一个算法或者其数据结构的定义是可能有多个的，往往其中一个是更加本质的。

- 有些难以想到的方法，很可能是在提醒你有更加本质的东西你可以了解（下面会给出一题）

---

### Leetcode

现在给出我当初第一次写的递归处理链表的题和我的解答
[206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list)

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution
{
public:
    ListNode* reverseList(ListNode* head)
    {
        if (!head || !head->next) return head; // Base case: Linked List is empty or only has one node

        auto reversed = reverseList(head->next);
        head->next->next = head;
        head->next = nullptr;

        return reversed;
    }
};
```

---

如果你还不能很自然地理解为什么可以这样写， 可以做下面的事情
1. 评论我的频道, 我会做出解答
2. 重新回到这篇文的你绝对有100%信心理解的地方重新阅读

---

回到开头。
很显然在递归定义下，链表根本不需要有变量可以O(1)地得到长度。
有人可能会想，那我硬要开一个变量存可不可以呢，答案是可以，但是有什么意义呢。即使你有链表的长度，你仍然无法O(1)地让链表变异。

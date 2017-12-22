[Team]
running at top spped
produce at the optimal velocity 

[problem]
That, of course, **eventually** result in problems

**problematic** adj.
: Even obvious two argument functions like assertEquals(expected, actual) are problematic.  How many times have you put the actual where the expected should be?

You should **be aware that** they **come at a cost** and should **take advantage of** ...



[program language]
"语句" : switch statement, 
: By their nature, switch statement always do N things


[可读性]
descriptive (adj.) : Use descriptive names
举个例子, assertEquals(actual, expected)不好, 应该用assertActualEqualsExpected(actual, expected)

consistent  (adj.) : be consistent in your names

capture the intent of the function

make a function communicate its intent?


[函数]
output arguments不可用
Using an output argument instead of a return value is confusing.

Command/Query Separation : 一个函数要么是命令, 要么是查询.不要both. 
一个典型的例子就是: boolean set(key, value)
结果用时要这样 : if( set(key,value) {....}
办法就是分离出两个函数： isKeyExist(), set()

Perfer Unchecked Exception to Returning Error Code

Don't repeat yourself

violate the OCP
By their nature, switch statement always do N things

The ideal number of argument for a function is zero. Next comes one, followed closely by two.  Three arguments should be avoided where possible.

Arguments are even harder **from a testing point of view**. Imagine the difficulty of writting all the test cases to test every combination of values.

Flag argument are ugly. It loudly proclaims that this function does more than one thing.


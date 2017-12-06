[Team]
running at top spped
produce at the optimal velocity 


[program language]
"语句" : switch statement, 


[可读性]
descriptive (adj.) : Use descriptive names
举个例子, assertEquals(actual, expected)不好, 应该用assertActualEqualsExpected(actual, expected)

consistent  (adj.) : be consistent in your names


[函数]
output arguments不可用

Command/Query Separation : 一个函数要么是命令, 要么是查询.不要both. 
一个典型的例子就是: boolean set(key, value)
结果用时要这样 : if( set(key,value) {....}
办法就是分离出两个函数： isKeyExist(), set()


We all know it is good to have test cases to guard our source code. But sometimes it is just hard, very hard to start your first step. I will introduce a real story about how to import test to my project, how to write test cases to cover the Android project, and why writting test cases makes our project better (with example, of course).

### 1. Start to write tests. And Oh, obstacle!

### 2. Design better : time to use MVP

MVP is not always the best option, sometimes MVVM may be a better option for you. I will not explain that too much (maybe I will write a post about MVVM and how to use it later).  Anyway, writting tests makes you refactor your code and build your project with a better architecture. 

This is obvious a example, now your view logic and business logic is separated. Why is it good? 

(1). Code is separated and decoupled. Now you can change presenter and do not affect the View. Or you can change the static ScrollView to a RecyclerView, but you do not need to change the Presenter. That is saying, modifing your code is much easier and less risky now!

(2). reuse.


### 3. Test your view


### 4. Test your presenter


### 5. Conclusion


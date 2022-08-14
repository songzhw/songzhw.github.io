## Requirement
We have a requirement to ask users to take some pictures and upload them to our server.
The whole flow is :
```
Take Photo --> review the photo --> (if happy) upload it
                                --> (if unsatisifed) retake the picture
```

So this flow can be divided into 4 activities: `TakePhotoActivity`,  `ReviewActivity`, `RetakePhotoActivity`, and `UploadActivity`.
 p.s. The "taking photo" activity are quite different from the "retaking photo" actiivty, that's why we have two separate two activities rather than just one activity. However, this two activities do have something in common : `taking photo`. Could we reuse the taking photo logic in both activities?


## Inheritance
The impulse is make RetakePhotoActivity extend TakePhotoActivity. So the taking photo logic could exist in the TakePhotoActivity. And this could be reused in its child activity, RetakePhotoActivity.

However, I feel uncomfortable to do so. Actually I've seen this kind of code many many times before. For example, I saw some Activity hierarchy such as : `BaseAcitivity - NetworkActivity - BaseOrderActiivty - PlaceOrderActivity - OrderDetailActivity`. 
The shortcoming of this approach are: 

1). The PlaceOrderActivity has its own layout xml, and the OrderDetailActivity has its own layout xml as well. Which means the OrderDetailActivity#onCreate() will load several super class' xml as well when you call `super.onCreate()`. 

2). Another point is it really is error-prone. Once you change any parent class's method, you may inflect all it's children or grandchildren classes. 
I have come across this kind of bug many times, most of times it just one change of parent class, making the child class behavir different. 

I want to avoid this coupling, which means I can not use inheritance. I would make the Activity independent from each other, they can be siblings, but they are not parent-child.



## Composition


## Conclusion


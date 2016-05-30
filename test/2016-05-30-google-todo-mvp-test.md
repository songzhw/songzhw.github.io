# Google's android-architecture

Google released the "andorid-architecture" in the early 2016. Obvious, google noticed the complex architecture in Android. Activity is usually a "God Object", and tests are usually missing, and so many architectures exist, like MVP, MVVC, Clean, ... 

Now google offers a same TODO application implementd using different architecture concepts and tools, including : MVP, Loader, ContentProvider, DataBinding, Clean, Dagger, and RxJava. 

I think the "todo-mvp" is the most important project, because some other projects are build on it. So I spend some time to learn the "todo-mvp" project and try to imitate it.  I have to say I learned a lot from that project. This post is one of them. 

# The test framework in the "todo-MVP"
I will only talk about the elegant architecture in the test framework. 
Just like the last two posts about Android Test, I came out a clean test framework by my self, because I realized that how hard is the test in real life. If we depends on the network, or the hardwork, which may fail sometime, or may take a while to responde, we may fail our tests some time, and that's not I expected. I really want all my tests pass every single time. 







External Link:
[Leveraging product Flavors in Android Studio for hermetic testing](http://android-developers.blogspot.ca/2015/12/leveraging-product-flavors-in-android.html)

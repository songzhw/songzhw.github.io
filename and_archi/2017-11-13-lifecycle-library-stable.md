Google had released a stable version of the lifecycle library. Here is something you need to know if you are still using the alpha or beta version. 

### 1. No More LifecycleActivity
LifecycleActivity is a subclass of FragmentActivity, and therefore the lifecycle library does not support AppCompativity and Fragment back then. 

But LifecycleActivity is deprecated in the stable 1.0.0 version.  Now you should use AppCompatActivity. Because the AppCompatActivity implements the LifeOwner interface now, and it can replace the old LifecycleActivity.





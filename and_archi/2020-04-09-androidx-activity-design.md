
The androidx.Activity library that Google released is kind of new to many developers. This is the post to introduce how to use it, and also try to get the bottom of this library. 

## How to use androidx.Activity?

### 1. import androidx.Activity
In your build.gradle, add the dependencies:

```groovy
dependencies {
    def activity_version = "1.1.0"

    // Java language implementation
    implementation "androidx.activity:activity:$activity_version"
    // Kotlin
    implementation "androidx.activity:activity-ktx:$activity_version"
}
```

### 2. what is inside androidx.Activity

#### v1.0
* **ComponentActivity**:  Now the `ComponentActivity` is the new base class for `FragmentActivity`. This said, it is also the base class for `AppcompatActivity`. This enpowers you to have a constructor with an `@layoutResid int layoutId` argument and other useful functionalities. 
  
* **OnBackPressedDispatcher** : this is an alternative for overriding onBackPressed. You can write you onBackPressed logic to a `OnBackPressedCallback` class.


#### v1.1

* **SavedState** : same as OnBackPressedDispatcher, this would enpower to save data instead of overriding onSaveInstanceState(bundle) method.

* **by viewModels()** : this is a syntax sugar to save you troubles to write long initialization of one ViewModel instance. And also, this could use `SavedStateViewModelFactory` as the default factory for generating a ViewModel instance.


#### v1.2.0-alpha 





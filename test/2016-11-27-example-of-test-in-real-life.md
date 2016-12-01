We all know it is good to have test cases to guard our source code. But sometimes it is just hard, very hard to start your first step. I will introduce a real story about how to import test to my project, how to write test cases to cover the Android project, and why writting test cases makes our project better (with example, of course).

### 1. Start to write tests. And Oh, obstacle!
Here is the source code:

```java
class PaymentFragment extends BasePaymentFragment {
    public void onCreate(Bundle b){
        super.onCreate(b);
        ...
       httpRequest.sendRequst(id, callback);
    }
    
    // callback's method
    public void onHttpResponseSuccessful() {
        refreshView();
    }
}
```

Oh, this is not hard to test, you just test the http callback and UI. But is it? 

Robolectric can help us test Fragment, just like this:

```java
    PaymentFragment fragment = new PaymentFragment(arg1, arg2); 
    FragmentTestUtil.startFragment(fragment);
    // Then the fragment will call "onCreateView()", "onResume()" ...
```

But when I do it, I found it's very difficult to get an instance of Fragment in the test.  Because the arg1, arg2 is very huge. It's not a problem in the source, because we can get the data from the server. But in the test, we have to mock it. 

So this is how I did it:

```java
arg1 = mock(ComplexOrder.class);
arg2 = mock(ComplexAccount.class);
fragment = new PaymentFragment(arg1, arg2);
```

Then I still failed the test. The reason is `BasePaymentFragment`. This `BasePaymentFragment` parent class does its own something in its `onCreateView()`.  So I have to mock that logic too. 

After that, I failed again. Because `BasePaymentFragment` has a parent called `BaseMoneyFragment`, which, of course, does something in its own `onCreateView()`. That means I have to mock that logic too.

Bad news is the inheritance hierarchy is very deep. `BaseFragment -> BaseNetworkFragment -> BaseRefreshFragment -> BaseMoneyFragment -> BasePaymentFragment -> BasePaymentFragment`.  OMG, this is a hell of inheritance, and it's so hard to mock them all.

This is the obstacle I have met. Since I cannot mock a PaymentFragment, so I can not test it. But how about I separate the business logic out from PaymentFragment. Then at least I can test the business logic. So it is time to import MVP to our project.

### 2. Design better : time to use MVP

To test the business logic, I import MVP. The basic difference between MVP and MVC is that the Presenter in MVP does not contain the view logic, and View does not know the existance of Model. And here is [an example of MVP](https://github.com/Macarse/50AH-code/tree/master/hack020)

In one work, MVP helps you separate view logic and business logic.  (songzhw : MVP is not always the best option in some occasion, instead MVVM may be a better option then. I will not explain that too much. Maybe I will write a post about MVVM and how to use it later).  Anyway, writting tests makes you refactor your code and build your project with a better architecture. 

Here is how we refactor the original code. 

**Step 1 : create interface for View**

```java
public interface IPaymentView {
    void refreshPayView(PayResponse resp);
}
```
**Step 2: refactor our fragment**

```java
// PaymentFragment (as a View)
class PaymentFragment extends BasePaymentFragment implements IPaymentView {
    
    // ▼        
    private PaymentPresent presenter;
    
    public void onCreate(Bundle b){
        super.onCreate(b);
        ...
        // ▼
       presenter = new PaymentPresent(this);
       presenter.getPaymentInfo();
    }
    
    // ▼ called by presenter
    public void refreshPayView(PayResponse resp) {
        boolean isSucc = resp.isSucc();
        if(isSucc) {
            // show UI for successful situation
        }
    }
}
```

**step 3: create Presenter**
In the same time, we extract the business logic to Present class. So our view is only in charge of showing, refreshing UI views. 

```java
// ▼ move business logic here
public class PaymentPresent {
    public PaymentPresent(IPaymentView view) {...}
    
    public void getPaymentInfo(){
        httpRequest.sendRequst(id, callback);        
    }

    // ▼ callback's method
    public void onHttpResponseSuccessful(PayResponse resp) {
        view.refreshPayView(resp);
    }    
    
}
```

This is a simple but clear example, now your view logic and business logic is separated. Why is it good? 

(1). Code is separated and decoupled. Now you can change presenter and do not affect the View. Or you can change the static ScrollView to a RecyclerView, but you do not need to change the Presenter. That is saying, modifing your code is much easier and less risky now!

(2). reuse. Let's say, you have to deposite money from the bank, the UI is same UI as the payment. You also have to input the password and show your account info. This time, you can reuse the View, just create a new Presenter(DepositePresenter).


### 3. Test your presenter


### 4. Conclusion


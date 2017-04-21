
## Chapter 6. I don't have much time and I have to change it
Suppose you are very busy and now your boss need you to add a new feature. What should you do with the legacy code?

### 6.1 Sprout Method
Here is your legacy code that our entries need to post date and be added to the transaction bundle.

```java
public class TranscationGate {
    public void postEntries(List<Entry> entries) {
        for(Entry entry : entries){
            entry.postData();
        }
        mTransactionBundle.getListManager().add(entries);
    }
}
```
Now we need to verify that the transaction bundle only add the new entries.  We could change the code to this:
```java
// Not a good example
public class TranscationGate {
    public void postEntries(List<Entry> entries) {
        List<Entry> tmp = new LinkedList<>(); //▼
        for(Entry entry : entries){
            if(!mTransactionBundle.getListManager().hasEntry(entry)) { //▼
                entry.postData();
                tmp.add(entry); //▼
            }
        }
        mTransactionBundle.getListManager().add(tmp); //▼
    }
}
```
This seems like a simple change, but it was pretty invasive. There isn't any separation between the new code we've added and the old code. 

What we need to do is separate the new code and old code, so the test new features is easier. A better version of code is like this:
```java
public class TranscationGate {

    //TODO add a @Test method to test the new method/feature
    List<Entry> uniqueEntries(List<Entry> entries){
        List<Entry> tmp = new LinkedList<>();
        for(Entry entry : entries){
            if(!mTransactionBundle.getListManager().hasEntry(entry)) {
                tmp.add(entry);
            }
        }
        return tmp;
    }

    public void postEntries(List<Entry> entries) {
        List<Entry> tmp = uniqueEntries(entries) //▼
        for(Entry entry : entries){
            entry.postData();
        }
        mTransactionBundle.getListManager().add(tmp);//▼
    }
}
```

```
songzhw:
    Actually I am not 100% agreeing with this section. The book think `How do we know we got it right?`. I think that should not be worried. Because that's JUnit's job. 
    We should add test code to cover the old code. Then we are not afraid of changing the code, because we know after we made the change our test case are in the background to guard our code. 
```

### 6.2 Wrap Method
Adding behavior to existing method is quite easy, but maybe not the right thing to do. Let's see an example.

Old code
```java
public class Employee {
    public void pay(){
        for(TimeCard time : timeCards){
            .... // time.getHours() * payRates
        }
        payDispatcher.pay(amout);
    }
}
```

Let's suppose that a new requirement comes along. Every time that we pay an employee, we have to update a file with the employee's name so that that it can be sent off to some reporting software. 

A good way to do that is to add a wrap method, whose name is still "pay()", but the implementation is changing. 
```java
public class Employee {
    private void dispatchPay(){ //▼ name: pay() --> dispatchPay()
        for(TimeCard time : timeCards){
            .... // time.getHours() * payRates
        }
        payDispatcher.pay(amout);
    }
   
    public void pay(){ //▼ a wrapper
        logPayment();
        dispatchPay();
    }
}
```
Clients who used to call pay() don't have to know or care about the change. They just make their call, and everything works out okay.


Another form of *Wrap Method* is having two parallel methods. You can call pay(), and you can call payWithLog() too. It's your decision to call which method.
```java
public class Employee {
    public void payWithLog(){
        logPayment();
        pay();
    }

    public void pay(){  ...  }
   
    private void logPayment() { ... }
}
```

### 6.3 Wrap Class



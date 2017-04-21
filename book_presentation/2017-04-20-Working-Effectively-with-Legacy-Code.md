
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


### 6.3 Wrap Class
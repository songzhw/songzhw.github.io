
## Question
Recently, I've been asked a question about the request queue in different pages. Consider the following scenario, I launch Page A which send a couple of requests, then I navigate to Page B immediately. Page B also need to send some requests. And I quickly switch back to Page A again.  

Let's say Page A sends two requests: A1, A2, and Page B sends one reqeust B1.  Thus the whole request queue would be like this:
A1, A2, B1, A1, A2

Here is the requirement:
* we don't want to send duplicated requst
* the request in the later pager is much important, so we need to do B1 before A1 and A2 when we navigate to Page B from A. (Becuase this is the current page our users see, so we would like to load this page's data first.)

I was asked to implement such a request queue. 


## Thoughts
It's like saying "the latter requst is more important". By "important", we mean priority. Thus we may need the help of `PriorityQueue`.

I tried some solution try to using `PriorityQueue`, and make sure every Request class has a priority field. But the priority is dynamic. Once you switch to another page, the existing request's priority should be decreased. Thus, it's quite difficult to do it this way. 

Somehow one idea came to me. "The latter request is more important" means the latter request should be exectued first. Ooooh, isn't it just describing a Stack?!

So what we need to do is just to implement a Stack without duplication.

## Solution 01:
We all know there is a `java.util.Stack` class for this structure. But we don't have the mechanism to remove duplications.

To remove duplications, we may need to iterate through the whole stack to see if we have duplication. And `java.util.Stack` class does not have such a method for us to iterate.

That's the problem we need to solve.

I then accidently saw the solution when I browse the [API document of Stack](https://developer.android.com/reference/java/util/Stack.html):

`A more complete and consistent set of LIFO stack operations is provided by the Deque interface and its implementations, which should be used in preference to this class. For example:`
```java
Deque<Integer> stack = new ArrayDeque<Integer>();
```
   
`

Oh, so we could just use Deque to mock a Stack. After all, Deque supports element insertion and removal at both ends.

The following code is what I got at last. And it works.

```java
public class UniqueStack<E> {
    private Deque<E> deque = new ArrayDeque<>(); 

    public void push(E item) {
        boolean isContaining = deque.contains(item);
        if (isContaining) {
            deque.remove(item);
        }
        deque.addFirst(item);
    }

    public E pop() {
        return deque.getFirst();
    }

    public E peek() {
        return deque.peekFirst();
    }

    public boolean isEmpty() {
        return deque.isEmpty();
    }

}
```








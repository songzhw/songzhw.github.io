
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
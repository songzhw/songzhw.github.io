#Respect user Attention : *Notification Best Practices*

Content
> Notification Best Practices
>> Don't annoy the user
>> Respect the user
>> Empower them
>> Delight them
>> Connect them to the people they care about


## Don't annoy the user
One of the things we do in Lollipop and on is we have a mode called "Do Not Disturb".

You don't know if you're going to be interrupting me and something I care about. But I can give the system signals. I can say, listen, I don't want to be bothered by unimportant things for the next hour.

The right way to do this:
```java
new NotificationCompat.Builder(this)
    ...
    .setCategory(NotificationCompat.CATEGORY_MESSAGE)
    .build();
```

I will try to use NotificiationCompa everywhere, because this will at least compile and run all the way back to four(Android  1.6).

You have to pick one [category](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html) for you notification. Here is some categories:

Category  |  Description
:-------------------------:|:-------------------------:
CATEGORY_ALARM | alarm or timer
CATEGORY_REMINDER | user-scheduled reminder
CATEGORY_EVENT | calendar event
CATEGORY_CALL | in coming call(voice or video)
CATEGORY_MESSAGE | incoming direct message(SMS, instant message, etc.)
CATEGORY_EMAIL | asynchronous message, such as email
CATEGORY_SOCIAL | social network
CATEGORY_RECOMMENDATION | a specific, timely recommendation for a single thing

## Respect the user
1: **set priority**
```java
new NotificationCompat.Builder(this)
    ...
    .setPriority(NotificationCompat.PRIORITY_LOW)
    .build();
```


Priority  |  Description
:-------------------------:|:-------------------------:
PRIORITY_MAX | time-critial tasks <br/> (incoming call, turn-by-turn directions)
PRIORITY_HIGH | important communications<br/> (chats, texts, important emails)
PRIORITY_LOW | not time sensitive<br/>(social broadcast)
PRIORITY_MIN | contextual or background information<br/>(recommendation, weather)

2: **setOnlyAlertOnce**
Another way to get the user the information they need is to use this thing called "setOnlyAlertOnce()"

```java
new NotificationCompat.Builder(this)
    ...
    .setOnlyAlertOnce(true)
    .setprogress(100, 50, false)
    .build();
```
It's not an instant message notification. It's not the one that I hear the beep, and I pull my phone out, and look at it right away.  It might be something like, what's the weather, or what's the latest stock quote, or how far are you away from some place.    Those are the kinds of things where you're not going to alert these. You don't have any timely information to give them, but you have information they might be able to take advantage of if they happen to see it.

And this is the way if you did want to alert them that you were posting it, but you didn't want to keep bothering them while you're updating it. So **setOnlyAlertOnce()** says, if you've already made noise, don't do it again.

3: **cancle out of date notification**


## Empower them


## Delight them


## Connect them to the people they care about
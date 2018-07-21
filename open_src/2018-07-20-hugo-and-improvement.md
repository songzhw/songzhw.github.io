[Hugo](https://github.com/JakeWharton/hugo) is a library made by Jake Wharton. I have studied the Hugo library in June 2016. But two years later, Junly 2018, I investigated the library again, and found out it could be improved. This post is about Hugo and how to improve it.

## I. Hugo

### 1.  What 
Hugo is a library that helps you log the method call and the time it took to execute.  So it would be convenient for the developers to know which method takes more time to execute. 

Simply add @DebugLog to your methods and you will automatically get all of the things listed above logged for free.
```java
@DebugLog
public String getName(String first, String last) {
  SystemClock.sleep(15); // Don't ever really do this!
  return first + " " + last;
}
```

``` java
V/Example: ⇢ getName(first="Jake", last="Wharton")
V/Example: ⇠ getName [16ms] = "Jake Wharton"
```

### 2. How
How does Hugo implement this functionality? 

### 2-1). Outline
Base what we saw before, Hugo uses an annotation, so Hugo must use APT(Annotation-Processing Tool) to know which method is needed to track.

Also, this log sort of thing seamlessly record a log, which seems like a AOP(Aspect-Orientated Programming). 

### 2-2). Annotation Processing


### 2-3). AspectJ


## II. Gradle Plugin

## III. Improvement
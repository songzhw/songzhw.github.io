# Lambda in Java 8 : Part I

Android N is about to come out. The biggest feature for android developer is a custom compiler that can use the features of Java 8. 

Java 8 has some fantacitic feature, like lambda, collection stream API.  Today, I will talk about the lambda firstly. 

## I. samples of lambda in Java 8

Python, Ruby, Groovy also have lambda. So maybe a lot of people are familiar with the concept of lambda. Besides, there are a lot of introduction ot lambda in java 8. So I will not descript too much concept of lambda. "Talk is cheap, show me the code". Here is some samples:

### 1. Runable

#### before Java 8

```java
new Thread( new Runnable(){
	@Override
	public void run(){
		System.out.println("run a thread");
	}
}).start();
```


#### Java 8

```java
new Thread(()-> System.out.println("run a thread"))
	.start();
```

### 2. Click event

#### <= java 7

```java
button.setOnClickListener(new OnClickListener(){
		@Override
		public void onClick(View v){
			System.out.println(v); // println the view you click
		}
	});
```

#### java8

```java
button.setOnClickListener( v -> System.out.println(v) );
```

Actually, there is a functional refrence in Java 8. So the codes can become better:

```java
button.setOnClickListener(System.out::println)
```

```System.out::println``` is an expression, which equals ```arg -> System.out.println(arg)```

```::``` is a operator that tell the compiler this is a functional interface. 

Then, what is functional interface? This question leads us to the next part. 

### 3. lambda in Stream API
This will be posted in the next post. So now I will not talk too much about it.


## II. What is lambda?
Lambda in Java reprensents a **functional interface**. Functional interface is an interface which has only one non-default method. 

However, in fact , lambda is implemented by the anonymous nested class. 

```java
@FunctionalInterface
interface Print<T> {
    public void print(T x);
}

public class WhatsLambda {
    public static void PrintString(String s, Print<String> print) {
        print.print(s);
    }
    public static void main(String[] args) {
        PrintString("test", (x) -> System.out.println(x));
    }
}

```

When we run "javap -p WhatsLambda.class" in the build directory, we can find out a another function called "lambda$main$0(String str)".


![](/imgs/20160318_01.png)

In fact, you can think of lambda in the way which the previous codes actually equals this:

```java
@FunctionalInterface
interface Print<T> {
    public void print(T x);
}

public class WhatsLambda {
    public static void PrintString(String s, Print<String> print) {
        print.print(s);
    }

    final class $WhatsLambda$1 implements Print{
    	@Override
    	public void print(Object x){
    		lambda$main$0(x);
    	}
    }

    private static void lambda$main$0(String str){
    	System.out.println(str);
    }

    public static void main(String[] args) {
        PrintString("test", new $WhatsLambda$1().print("test"));
    }
}
```



## III. Closure

```java
    public void foo(int count){
        Runnable r = () -> {
            System.out.println("count = "+count);
        };
    }

```
In the previous codes, we find out that the lambda use the count variable. However the count variable is not defined in the lambda? Is this okay?

Yes, lambda is like a nested class, which can use member variables of a class or a final local variables. The count variable is local variable, which can be accessed by the lambda.

Since the count variable is out of the definition of the lambda, so we call the lambda is a **Closure**.  

So, the count variable can be called **Free Variables**.

Closure is a block of codes that can use the variables out of its own definition block.

Or, closure is a block of codes that can use free variables.

Java have its own closure. Yes, the nested class is a closure. But this old-style closure is not so handy. You have to definition a new Runnable object to use the run function. 

So, lambda is here to rescue us. 



## Reference 

http://www.cnblogs.com/WJ5888/p/4667086.html 




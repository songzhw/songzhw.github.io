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



## III. 

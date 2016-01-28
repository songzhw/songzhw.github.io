One of my ways learning English is watching some episodes. When I take notes about one good english sentence, I have to pause the tv, and write these english words down. 

And one day, I get tired. Can I have a smarter way to get the english scripts?

Take One of my favorite tv show for example, the scripts in the Internet is like this

```
1
00:00:01,140 --> 00:00:03,600
我看起来怎么样
Okay, how do I look?

2
00:00:03,740 --> 00:00:06,200
棒极了！
Great

```

My goal is to abstract these english words out, like this

```
Okay, how do I look?
Great
```

Since the script files are so regual. The english script lies in the fourth line every five lines. So it's easy.

I am a fun of Groovy and Ruby, so I choose Groovy to finish this job:

```groovy
def srt = new File('one_episode.srt')
def out = new File('out.txt')

int index = 1
srt.withReader{ reader ->
	reader.transformLine( out.newWriter() ) { line ->
		index++
		if(index % 5 == 0){
			line  // the last line is actually the returnd value
		}
	}
}
``` 


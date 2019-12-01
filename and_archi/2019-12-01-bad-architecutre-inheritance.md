Last week, I just watched one technical video presented by two Airbnb employees. I have to say I am a little disappointment at their architecture choice. And I'm afraid many people might do the same bad choice, so this post is trying to explain what it is, and why it is bad.

## 1. The technical video to introduce Airbnb's architecture
First of all, the video I watched is a React video that Airbnb is explaining they roadmap of architecture change. However, the ideas are actually common, so it applies to Android as well. 

## Airbnb v1.0
The first approach Airbnb was using is the plain js. Just like this, `<button className="alpaca/cat">Click me </button>`.
The problem of course is the style info are piled up in the .css files, which make the css become more and more fat.


## Airbnb v2.0
So Airbnb managed to change this style to one type-specific design. For the previous example, they remove the `className` attribute, and add some more meaningful attributes. Let's say, they change the button to `<button isAlpaca={true/false}>Click me</button>`.

I've done something like that before. The problem of this design is the flexibility. What if you have more types, like `isCat`, `isDog`, `isWhale`... , the possibility is endless. So this kind of design is hard to expand in the future. 

## Airbnb v3.0


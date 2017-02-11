### Background
I've developed a payment SDK at the previous company. Our payment SDK is easy to import, but I think there is still one issue about our SDK: our SDK only supports loading the local images. 

The main reason about it is becuase I want our SDK easy to import, and enough lightweight. If my SDK add a ImageLoader library, let's say, picasso, and all the apps that import my SDK need to import picasso, too. And this hardset rule is a little unfair for our client. All the apps that import my SDK are my clients. I want my cients happy and feel comfortable to use my SDK.



## Problems We Had
I managed one project before. The sprint is short, and we need to release our app every month, which leaves us only 1week to get the requirement, 1.5 week to devleop, and 1.5 week to test. Even worse, our Android/iOS developer always need to wait the complete of bck-end API development. It this takes the back-end developer four days, then our client may only have three days to develop our features. 

As the mobile leader, I talked to the back-end developer, and aksed him to output an API document first. By using this document, the mobile developers knows what kind of protocol we need to make, and could pretend that we were talking to the server by using some debug tricks. This is crucial for the project we were doing,  at least the mobile developers could start  their job as long as we  finailized the requirement. 

## Improvement
The approach mentioned before is good. But every mobile needed to mock some json files to pretend they were talking to the server. Is there a better way to help them improve the efficiency?

Yes, we have. We could use Node.js to make a fake server. Every mobile developer could just connect to this fake server to mock the successful/failed request. In this post, I will show you how to do that step by step. It is really easy. I know Node.js is using JavaScript, and if you are not familiar with JS, not freak out. It's very easy to understand. 

### 1. config your node.js environment

### 2. put your json files in the node.js server

### 3. customize different failed jsons

## Conclusion
 






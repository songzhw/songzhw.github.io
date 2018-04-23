
## Problems We Had
I managed one project before. The sprint is short, and we need to release our app every month, which leaves us only 1week to get the requirement, 1.5 week to devleop, and 1.5 week to test. Even worse, our Android/iOS developer always need to wait the complete of bck-end API development. It this takes the back-end developer four days, then our client may only have three days to develop our features. 

As the mobile leader, I talked to the back-end developer, and aksed him to output an API document first. By using this document, the mobile developers knows what kind of protocol we need to make, and could pretend that we were talking to the server by using some debug tricks. This is crucial for the project we were doing,  at least the mobile developers could start  their job as long as we  finailized the requirement. 

## Improvement
The approach mentioned before is good. But every mobile needed to mock some json files to pretend they were talking to the server. Is there a better way to help them improve the efficiency?

Yes, we have. We could use Node.js to make a fake server. Every mobile developer could just connect to this fake server to mock the successful/failed request. In this post, I will show you how to do that step by step. It is really easy. I know Node.js is using JavaScript, and if you are not familiar with JS, not freak out. It's very easy to understand. 

### 1. config your node.js environment
1. download Node.js from [https://nodejs.org/en/download/](https://nodejs.org/en/download/)
2. install it
3. if you could see version in terminate after you type "node --version", then you are good to go.
```
MacBookPro:~ songzhw$ node --version
v8.11.1
```

### 2. add back-end server files
we just need a server to receive request, and to response with a json file. Here is the simplest server-side code

```javascript

var http = require('http')  // to get the http request. Just like the Serverlet
var url = require('url')    // to get the url and arguments in the request 
var queryString = require('querystring')// to get the url and arguments in the request 
var fs = require('fs')  // file system

// start a server, which listen on port 8899
http.createServer(onRequest).listen(8899, function () {
      console.log("server listening on port 8899 ...")
})

function onRequest(req, resp) {
      var reqUrl = req.url  //=> something like "dashboard?id=23&name=44"
      var dstFileName = url.parse(reqUrl).pathname //=> "dashboard" 
      var path = "./public" + dstFileName + ".json" 

      fs.exists(path, function (isExist) {
            if (!isExist) {
                  send404(resp, " " + path + " does not exist")
                  return
            }
            fs.readFile(path, function (err, data) {
                  if (err) {
                        send404(resp, err)
                        return
                  }
                  sendFile(resp, path, data)
            })
      })
}

function send404(response, err) {
      response.writeHead(404, { 'Content-Type': 'text/plain' });
      response.write('Error 404: resource not found : ' + err);
      response.end();
}

function sendFile(response, filePath, fileContents) {
      response.writeHead(200, { 'Content-Type': 'text/plain' });
      response.end(fileContents);
}
```

Simple, right? If you are asking "http://localhost:8899/dashboard", we will give you a "dashboard.json". And of course, you need some json files in the server, just like this:

```
[root]
  -- server.js
  -- public
      + dashboard.json
      + splash.json
      + dashboard_20.json
      + dashboard_30.json
```

### 3. customize different failed jsons
As a developer, you can not just think of the successful scenario. You need to take the failed scenario into consideration too. It is the failed scenario that crash your app, like out of boundary, zero or negative number for a ID, unacceptable input for the server.... 

Hence, we need to provide an option for failure scenarioes. What you need to do is adding a failure code to your url. Take "http://localhost:8899:/dashboard" for example, requesting this url will get you the dashboard.json. Now you can call "http://localhost:8899:/dashboard?failed=20", then the fake server I made will provide a json for this failedId = 20 scenario, such as :

```json
{
      "errorCode":20,
      "errorMsg":"No such an ID"
}
```

To do that, we need to change a little to the server.js.

```javascript
...

function onRequest(req, resp) {
      var reqUrl = req.url  //=> something like "dashboard?id=23&name=44"
      var dstFileName = url.parse(reqUrl).pathname //=> "dashboard" 
      var path = "./public" + dstFileName + ".json" 

      // =============== the new content ===============
      var failedId = parsedArgs["failed"] //没有就是undefined
      console.log("szw f = "+failedId)
      if(typeof failedId != 'undefined') {
            dstFileName = dstFileName + "_" + failedId 
      }
      // =============== end ===============
      
      var path = "./public" + dstFileName + ".json" 
      ...
```


## Conclusion
Node.js is easier to deploy and use than Tomcat. And the code is quite easy to understand. With the help of Node.js, mobile developers now could speed up their development even without the real server API. 

By the way, this could also be good for you if you want to use Espresso and real http connection. to test all the different test cases. Of course, to make sure the bad code is the only reason to fail the test case, I don't recommend to use real http connection to do the test. But it's still good to know there's one option there, just in case you may need it somehow.






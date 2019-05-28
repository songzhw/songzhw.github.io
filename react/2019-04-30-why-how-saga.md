// saga(适用人群, reducer处理异步, redux改, saga出场, 主要聊思路(方便测试与异步), 以及测试)

# I. how Redux handles async event?


## 1. one async scenario

Let's assume we have a requirement to fetch user info from the back-end, and we could write such a reducer, as follows: 

```javascript
// [snippet 1]
export const productsReducer: Reducer= (state = initState, action) => {
  if(action.type === "REQUEST_USER") {
    const response = fetch(action.url);   // 1. issue 1 !
    dispatch({type: "RESPONSE_USER", payload: response}); // 2. issue 2 !
  }
}
```

This is pseudocode, which actually have a couple of issues. Now let's go dig it.



### 1.1 issue 1: doesn't handle Promise
`fetch(url: string)` is a function that handles HTTP request and returns a Promise. So we now could know the first issue of snippet 1: it does not handle the async Promise. The response is supposed to be a string, but now it's a Promise.




### 1.2 issue 2: async/await is not a good option
 However, JavaScript does have a `async/await` keyword, which seems a perfect fit for this scenario. The result would be : 
 
 
```javascript
// [snippet 2]
export const productsReducer: Reducer= async (state = initState, action) => {
  if(action.type === "REQUEST_USER") {
    const response = await fetch(action.url);   // 1. issue 1 !
    dispatch({type: "RESPONSE_USER", payload: response}); // 2. issue 2 !
  }
}
```
 
The preceding code seems okay. But it's actually anti-pattern. Redux has three important principles, and one of it is "reducer should be a pure function".

Being a pure function makes sure the function is easy to use, test, and find bugs. Once we use await some Promise, then this promise could success, and also could fail as well. Aka, our reducer could have two result: success or failure. Then, reducer is not a pure function anymore. 

In short, we would like to move this async process code out, to make our reducer a pure fucntion.
 
### 1.3 issue 3: could not find `dispatch` 
`dispatch` is actually a method of `Store`. This means reducers can not just use `dispatch` like that.

There are one work around though. In your action, you put the dispatch in, as follows:

```javascript
// [screen]
this.props.dispatch( {
  type: "..", 
  payload: {}, 
  dispatch: this.props.dispatch}
)

// reducer
export const productsReducer: Reducer= (state = initState, action) => {
  if(action.type === "REQUEST_USER") {
    const response = fetch(action.url);  // leave it here, we know it's a issue
    action.dispatch({type: "RESPONSE_USER", payload: response}); // now we use `action.dispatch` instead
  }
}

```

But this is not a good way, it makes our actions more complex. We woule prefer our action is just a normal plain object.


## 2. how to dispatch in a reducer?
Reducer is not a part of store, so it does not have a dispatch method, as we talk about before. But we could work around it. And the approaches are not only one. 

### 2.1 pass in the dispatch to the action
What we got from reducer parameters is `(state, action)`. What if pass `dispatch` to the reducer through the `action`?

```javascript
[screen]
const newAction = {
  type: "ADD",
  payload: {
  	dispatch: this.props.dispatch.
  	...
  }
}

[reducer]
if(action.type === "ADD") {
  const {dispatch} = action.payload;
  dispatch(action2);
}
```

p.s. `import { bindActionCreators } from 'redux';` the bindActionCreators method can combine action and dispatch together too, and it is from redux library.

### 2.2 create one middleware
Redux middlewares are able to send differnt actions. So we could create a middleware to handle all this dispatching logic. 



## 3. how to test such a reducer?

When it comes to the test, one principle standard is "the only reason the unit test fails should be your test failed your expectation". That means, if your unit tests fails because of outside reasons (like the network connection is failed to set up), then your unit test is not a good unit test. 

This standard actually instruct us to mock the http connnection, to mock database connection, to mock some outside dependencies. 

But the previous async reducer actually makes the test hard. Because when the async reducer's tests fail, we don't know it is the source code is not working as we expected, or it is just the back-end fails to response us. 

In conclusion, if we want to unit test reducers, we need to extract the async code out of reducer. 




# II. Saga coming to rescue



# III. how to test Saga


### how to test async saga code?
As we mentioned before, we want to test the async code. Meanwhile, async code is fragile to the unit test, as it might fail just because of some outside reason. 

# IV. how saga handle async code? 




```javascript

```

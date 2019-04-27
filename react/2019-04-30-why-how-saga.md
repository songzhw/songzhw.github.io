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







## 2. 


## 3. how to dispatch in a reducer?



## 4. how to test such a reducer?


# II. Saga coming to rescue



# III. how to test Saga



# IV. how saga handle async code? 




```javascript

```

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
We've mentioned some disadvantage of "async reducer":
* it is not a pure function
* it is hard to test
* it has to wait time-consuming function to finish, then can start to work.

Saga actually fixes all these issues. 
* saga is not a reducer, it does has to be a pure function
* although saga has async code, saga is super easy to test
* since saga functions are generators, so we could *pause* the function if we don't call `generator.next()`.

## 1. A Simple Sample of Saga

```javascript
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}


function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

This is an easiest example for saga. We call the Api.fetchUser() method, and dispatch a new action with response to Redux. And this example could be an excellent example for us to understand why Saga is better for async code.

## 2. Explanation

I. There are no rule that requires Saga as a pure function. But saga function must be an generator. 
p.s. Why generator? Because it can be paused when we are doing some time-consuming work. After we get the response, then we can call `generator.next(response)`, and the generator would resume as we want.

II. Why saga, an async function, is easy to test?



# III. how to test Saga


### how to test async saga code?
As we mentioned before, we want to test the async code. Meanwhile, async code is fragile to the unit test, as it might fail just because of some outside reason. 

Now instead of testing the `Api.fetchUser` is working, we now just need to make sure the Api.testUser is called with the argument `userID`. Based on this assumption, Saga made an effect called `call` to help us test the async code.

For the code `const user = yield call(Api.fetchUser, action.paload.userId)`, all we need to test, is just the expected result is like this:

```javascript
const result = sagaGenerator.next().value;
expect(result).toEquals(call(Api.fetchUsers, userId))
```

### how to test saga code?
Since the saga is just a generator, so we can test it using generator syntax. We could take advantage of `generate.next().value` and `generator.next(mockedValue)` to help us test saga code.

Of course, there are some other saga test library out there, such as [redux-saga-test-plan](https://github.com/jfairbank/redux-saga-test-plan). These kinds of library will make our unit test for saga even easier. -- but sometimes they will have some odd errors that you don't know how to fix.

Anyway, we still have plan B: use generator to test saga. So Saga really is friendly to unit test, and we appriciate it.

# IV. how saga handle async code? 

The previous sections are talking about why saga is so good: easy to extend, easy to test, and meant for the async code. But why Saga is so good for what it's doing and how Saga handles all thes async/wait code?

This is an interesting topic and deserves at least quick a series of posts. But I still want to help you go through the saga, in deep. 

## 1. how saga detect action dispatch?
We all know, saga could know some specific action has been dispatched just now, but how did Saga do it?

The dev who is familiar with redux must know: "Oh, listen to the action dispatch -- that's what middleware is doing!" Yes, Saga actually is using a redux middleware to detech and , also, change the dispatched action. That's why we could use `put` effect to send out another action. It's no big deal, we just use `next(newAction)` in the middleware. Plain and simple.

## 2. `call` effect
a `call(Api.fetchUser, userId)` is just doing the `Api.fetchUser.call(null, userId)` under the hood.

## 3. `takeEvery` effect
From [the official documentation](https://redux-saga.js.org/docs/api/), we know the `takeEvery` is actually using low-level saga effects, as follows:


```javascript
const takeEvery = (patternOrChannel, saga, ...args) => fork(function*() {
  while (true) {
    const action = yield take(patternOrChannel)
    yield fork(saga, ...args.concat(action))
  }
})
```


But we do wonder, would this `while(true)` block the JavaScript event loop, since JavaScript only have one thread?

Good question. That means you are really thinking. And the answer is no, this `while(true)` do not block the event loop, in fact. 

You have to know that, this code is in the generator (please see the `fork( function*(){..})`), which means you can control the steps. If you don't call `generator.next()`, the `while(true)` loop actually does not go on. In this case, we know `while(true)` loop will not block the JavaScript event loop.

==============================================================
[中文版本]


有关TypeScript的文章好像在掘金上越来越多了, 我公司的React/React Native项目也在使用TypeScript, 我也因此用了将近一年了, 感受略有一些, 所以今天在这和大家分享, 一起提升. 但因为网上介绍TypeScript, 或是TypeScript如何和React结合的文章已经很多了, 所以我就不多讲一些基础的知识点, 可以侧重于工作中碰到的问题来讲解TypeScript的一些特殊(于java, python, ...)的点, 也可以做为handbook用来查询.


# I. 导论


## 1. 初接触
用惯了js的同学, 刚接触ts时, 可能觉得比较烦, 一个小小的点, 也要报错或警报. 你要是用any吧, 那不就和js一样了, 那还用个什么ts啊. 但你要不用any吧, 经常有些司空见惯的地方都会报错, 让人很沮丧. 我们公司在德国还有一个子公司, 德国的团队就是用了多年的js, 很抵制使用ts. 结果让我们的纯ts代码(除了测试), 结果变得又有ts又有js了.


我个人本身是做Android, 也做React, ReactNative, 所以以前用惯了java之后, 用js就很累. 特别是看别人的代码时, 经常看到一个方法, 就不知道这个方法参数到底要传什么.
有些同事比较好, 还加个注释, 但问题注释是容易"腐烂"的. 也就是说当你在修改了代码之后, 有极大概率是只更新代码, 而不更新注释. 注释不像单元测试, 你要是改了源代码, 单元测试可能会fail掉, 但注释不会fail你, 所以你很可能会忘记更新注释.


所以我个人的想法是: 仍推荐使用TypeScript. 转TS的初期可能会有些沮丧, 但稍坚持一下, 我个人用了3星期, 就能习惯TS了.


## 2. TypeScript是强类型, 但和传统的java/python/ruby/kotlin仍是很多地方不一样
TypeScript是一门强类型语言, 你传入的类型不一样, TS可是会报错.
但有两点, 我们是要注意的


1). 因为TS编译后仍是个js, 所以TS中的一些特性其实仍是JS中的特性. 比如TS中的class, 在底层仍是JS中的function


2). TS对类型的检查更灵活. 我们可以使用 TypeA & TypeB, 也有Partial<TypeC>. 这一点在日常中有很多应用


3). TS会分成~.ts, ~.tsx两种文件. 有JSX的话就应该是使用~.tsx文件.
但区别可不是这么简单哦, 我在后面讲泛型时还会回到这两个文件的区别.


## 3. 说明
初接触TS的一个问题, 就是很多类库不知道某一个参数到底是什么类型. 所以我在本文中也会专门就一些常用类库做一些TypeScript的说明


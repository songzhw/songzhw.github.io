## Introduction

This is actually one of a series posts which mostly focus on several critical issue of "react-like" framework. This post is mostly focus on the Jetpack Compose UI. All the posts are:
* [Jetpack Compose version](https://github.com/songzhw/songzhw.github.io/blob/master/and_archi/2021-08-31-compose-issues.md)
* [React Native version](https://github.com/songzhw/songzhw.github.io/blob/master/react/2021-09-11-rn-issues.md)
* [Flutter version](https://github.com/songzhw/songzhw.github.io/blob/master/flutter/2021-10-01-flutter-issues.md)


## I. Re-render issue

### 1.1 Issues on React Native
One of the biggest performance issue in the React Native is the re-render issue. Let's say we have a parent and child:

```javascript
function Parent() {
  // it has two props/states: A , B
  return (
    <Text text={A}/>
    <Child prop={B}>
  )
}
```

Every time the prop A changes, it will refresh the Parent component/View, which means it will generate a new instance of `Child` and `Text`. 

But it's really unnecessary, as `Child` only depends on prop B. The parent ask child to re-render based on a irrelevant prop(A), that's kind of awful. 

### 1.2 What about Flutter?


## II. Long list performance


## III. Call tranditional view
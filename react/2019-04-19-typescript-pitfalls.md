
# Pitfalls of TypeScrit you may get in React project
For those who are familiar with modern language, JavaScript is a language that may let you feel familiar, but also would be the one to slap you right on your face somewhere. The lack of types, of course, would be a downside of JavaScript. That is also why TypeScript can rescue you.

However, TypeScript is not a silver bullet. Importing TypeScript would make sure your code is more readable, but also may have more issues you don't have in JavaScript. This post is trying to help you know some pitfalls in TypeScript, in order to make your TypeScript journey more comfortable.

## I. Define props

The following code works fine. You will not have an error by it. 

```typescript
const Item = () => {
	return (<div/>)
}
```

But reality is more complex than that: we normally would like to pass in some props.

In Javascript, we could use prop-type to define the type of props, as shown in the following code snippet:

```javascript
Item.propTypes = {
	name: PropTypes.string,
	book: PropType.object,
	values: Propypes.array,
	onTextChange: PropTypes.func
}
```

This code is helpful, but not that helpful. You don't know `value` is an array of what type, you don't know what's the types of parameter and returned value of `onTextChange`, of course, you still don't know what is this `book` means. 

A much more readable type definition would be something like the following snippet(pseudocode):


```typescript
type Props {
	name: string,
	book: TxtBook,
	values: string[],
	onTextChanged: (string)=>void
}
```

TypeScript is born for that, and it just help us make the prop type clear. Here is what TypeScript code that fulfill the same requirement:

```typescript
class TxtBook {
  public id: number = 0;
  public content: string = "";
}

interface IProps {
  name: string,
  book: TxtBook,
  values: string[],
  onTextChanged: (url: string) => void
}

const Item: React.FC<IProps> = (props: IProps) => {
  return (<div/>);
}
```

Now, let's take a close look at this preceding code. 

1. React.FC is a short of React.FunctionComponent, which takes in a generics P as the type of its props. (Oh, yes, React.SFC is deprecated now, so please don't use React.SFC anymore.)


2. The one thing that you may get wrong is the definition of a function. You may accidently write it down as `onTextChanged: (string) => void`. But you need to define the name of parameter when you trying to define a function/labmda type in TypeScript. So remember to add this parameter name, or you will get a compilation error.



## II. 



```typescript

```
# Advanced TypeScript Tips in React-Native

TypeScript imports types to the JavaScript world, like a sweet sunshine into a dark room. However, types being compulsory also means you have to know the exact type even when you don't know it. So this is the pain of TypeScript. Most of these situations come from the 3rd-part library, and some of them are hard to solve. But lucky for us, TypeScript has some advanced types to help us.

## I. React
We could use `PropTypes` to define type in JavaScript. But you can not define types of state in JavaScript, while TypeScript can.

Since react 16.8, React imports functional components. So I would like to introduce types into class components and functional components.

### 1. class component

```TypeScript
interface IProps {
  name: string;
}

interface IState {
  offset: number;
}

class SomeScreen extends React.Component<IProps, IState> {
  state = { offset: 0 };

  constructor(props: IProps) {
    super(props);
    console.log(props.name);
  }

}
```

### 2. functional component

```TypeScript
interface IProps {
  name: string;
}

interface IState {
  offset: number;
}

const SomeScreen = (props: IProps) => {
  const [offset, setOffset] = useState(0);
  console.log(props.name);
};
```


## II. React-Navigation
As we know , we need to use `props.navigation.navigate(...)`, so how is this react-navigation compatible with TypeScript? I list a few items that you may come accross in your development.

### 1. Props
We use `this.props.navigation.navigate("DetailScreen")` to navigate from one screen to another screen. But here is the problem : TypeScript does not know where this `props.navigation` comes form. You will get an error like the picture below:

![](images/2019-07-28 at 22.04.59.png)
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

const SomeScreen = (props: IProps) => {
  const [offset, setOffset] = useState<number>(0);
  console.log(props.name);
};
```

Of course, if you have multiple state to change, you may want to use `useReducer`. And to define the type you receive, you may want to add a little more detail, like `useReducer<MyReducer>`


### 3. flexible children view
Here is a scenario. Your children view are flexible; it is generated from some data. Then your children's type could be `JSX.Element`

```TypeScript
  render() {
    const children : JSX.Element[] = this.props.data.map((item, index) => {
      return <Image source={{ uri: item.url }} style={styles.item} key={`item${index}`}/>;
    });

    return (
      <View style={[this.props.style, styles.container]}>
        {children}
      </View>
    );
  }
```

p.s. This kind of flexible children view would be nice to have a key. Otherwise you may got a yellow warning.

  
### 4. ref  



### 5. HoC



## II. React-Navigation
As we know , we need to use `props.navigation.navigate(...)`, so how is this react-navigation compatible with TypeScript? I list a few items that you may come accross in your development.

### 1. Props
We use `this.props.navigation.navigate("DetailScreen")` to navigate from one screen to another screen. But here is the problem : TypeScript does not know where this `props.navigation` comes form. You will get an error like the picture below:

![](images/2019-07-28 at 22.04.59.png)

So the approach to fix it is add a `NavigationScreenProps ` type: 

```TypeScript
type IProps = NavigationScreenProps

export class SplashScreen extends Component<IProps> {
  componentDidMount() {
    setTimeout(() => this.props.navigation.navigate("app"), 10);
  }
}

```

### 2. Override shared navigationOptions
In your app, you probably would like to have a common Header (or Toolbar, ActionBar, as the Android developer calls it). However, you might want to have  the flexibility to change some info, let's say, like title. After all, each screen has different title. So what should we do to override some Header details across screens. 

Just like the component Props, it varies from class components to function components.

#### 1). Class Component

```TypeScript
class DetailScreen extends React.Component<IProps> {
  static navigationOptions = {
    title: "Detail"
  }
  ...
}
```

#### 2). Function Component


```TypeScript
const DetailScreen = (props: IProps) => {
  ...
}

DetailScreen.navigationOptions = {
  title: "Detail"
}
```


## III. react-redux

React-Redux is a popular library that can help us build a clean project. Hence, learning how to combine React-Redux with TypeScript is something we need to handle as wll. Lucky for us, it is quite simple for different steps.

### 1. actions

```TypeScript
export interface IAddAction{
  type: "Add"
}

export interface IRemoveAction{
  type: "Remove",
  paylaod: {
    id: number
  }
}

export type MyAction = IAddAction | IRemoveAction
```



### 2. state

```TypeScript
export interface MyState {
  readonly products: IProduct | null;
}
```


### 3. reducer

```TypeScript
export const MyReducer : Reducer<MyState, MyAction> = (
  state = new MyState(),
  action: MyAction
) => {
  switch(action.type){
    ...
  }
  return state;
}
```

### 4. store


#### 1).

```TypeScript
export interface IAppState {
  products: MyState,
  books: AnotherState
}

const rootReducer = combineReducer<IAppState>({
  products: MyReducer,
  books: ANotherReducer
})

export const store = createStore(rootReducer, undefined, applyMiddleware(...));
```

#### 2). 

Note that, there is another approach to write `IAppState`, which also works:

```TypeScript
export type IAppState = ReturnType<typeof RootReducer>
```


### 5. async action
I'm using Redux-Saga, which handles the async action for me. So I did not write a Thunx action, but it should be easy, especially if you are using `async/await` syntax sugar.

```TypeScript
export const fetches = async (): Promise<IProduct[]> => {
  await wait(1000);
  return products;
}
```


## ?. Hooks

### 1. useState


## ?. Test

`work.mockReturnThis()` : why js, not ts?


```TypeScript

```
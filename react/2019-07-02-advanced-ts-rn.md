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

  
### 4. defaultProps
```TypeScript
interface IProps {
  id: number;
  text?: string;
}

const MyView = (props: IProps) => {
  return (
    <>
      <p style={{ margin: "20px", fontSize: "50px" }}>{props.id} -- {props.text}</p>
    </>
  );
};

MyView.defaultProps = {
  text: "default"
};
```

This could make a defaultProps, but you can also make a code like this:
`MyView.defaultProps = { balabala: 'default'}`, which means there are no type check for (default) props.

About this type check issue, we could [do something else](https://medium.com/@martin_hotell/react-typescript-and-defaultprops-dilemma-ca7f81c661c7), but it would be then long or tedious. So I personally like the previous code.



### 5. ref 

#### 1). React #1
```TypeScript
// React (Approach 1)
const MyView = () => {
  let viewRef : HTMLDivElement | null;
  
  return (
    <div ref={v => viewRef = v} />
  );
};
```

#### 2). React #2
```TypeScript
// React (Approach 2)
const MyView = () => {
  const viewRef = createRef<HTMLDivElement | null>();
  return (
    <div ref={viewRef}/>
  );
};
```

#### 3). React Native
```TypeScript
const MyView = ()=>{
  let ref: View|null = null ;
  let imageRef = createRef<Image>();

  return (
    <View ref={ref}>
      <Image ref={imageRef} source={require("../a.png")} />
    </View>
  )
}
```

### 6. HoC
Let's say I need to add a loading component for any component, so, a HoC would be a good choice.

When it comes to TypeScript, then we must need to clarify the types in the HoC. Here is what I got from practice.

```TypeScript
interface IProps {
  loading: boolean;
}

const withLoader = <P extends object>(InputComponent: React.ComponentType<P>): React.FC<P & IProps> => {
  props.loading ? (... ) : (...)
  ...
;
```

Note that `ComponentType` actually includes function components and class component, as its source code is `type ComponentTYpe<P = {}> = ComponentClass<P> | FunctionComponent<P>;`

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


## III. Redux

Redux is a popular library that can help us build a clean project. Hence, learning how to combine Redux with TypeScript is something we need to handle as wll. Lucky for us, it is quite simple for different steps.

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


#### 1). create store

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

#### 2). ReturnType

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

### 6. `AnyAction`
`AnyAction` is an action type that represents all actions. Let's see how it is defined:

```TypeScript
export interface AnyAction extends Action {
  // Allows any extra properties to be defined in an action.
  [extraProps: string]: any
}
```

You may need it in the Redux-Saga, like :

```TypeScript
export function* handleAddBookmark(action: AnyAction) {
  ...
}
```

### 7. Redux-Persist
If you are using Redux-Persist, then the `IAppState` would have issues because the extra type added by the Redux-Persist. 

Before we adding the Redux-Persist, the state would be like `{book: {id: 22, name: "Harry" } }`. 
Now the stae is something like this: `{book: {id: 22, name: "Harray", _persist: {....} } }`.

We have to do this:

```TypeScript
interface IAppState {
  // book: IBookState  // this would cause an error, since we got a more extra field : '_persist: {...}'
  book: IBookState & PersistPartial;
}
```

### 8. React-Redux
It's obvious that some props are coming from `mapStateToProps` and `mapDispatchToProps`. But the below snippet is hard to write.

```TypeScript
function mapStateToProps(state: IAppState){
  return {
    id: state.book.id,
    name: state.book.name,
    msn: string
  }
}

function mapDispatchToProps(dispatch: Dispatch) {
  return {
    getProducts: () => dispatch({type: "", payload: ""})
  };
}

export interface IProps {
  id: number;
  name: stirng;
  msn: string;
  others: string[]
}
```

A more clever way to deal with this requirement is using `ReturnType`. Here is a better solution :

```TypeScript
type IProps = ReturnType<typeof mapStateToProps> 
					& ReturnType<typeof mapDispatchToProps>
					& ViewProps
```


### 9. Middleware
I saw many books all said this is how to define a middleware: `const middleware = store => next => action => {...}`. But please don't get fooled. The first argument `store` is actually not a Redux Store type. It's a `MiddlewareApi` type. 

THis is the definition of this type:
`type MiddlewareAPI = {dispatch: Dispatch, getState: ()=> State}`. Yeah, it does seems similar with redux store, but it is not.

Now go back to our issue: how to custom a middleware in TypeScript. The difficulty obvious lies on the types: we don't know the types. Here is a code snippet that can help us.

```TypeScript
const myMiddleware = (store: MiddlewareAPI) => (next: Dispatch<AnyAction>) => (action: AnyAction) => {...}
```

Note that we are using generics, otherwise you will get a complaint from TypeScript compiler. 


### 10. Dispatch
We, of course, need to dispatch actions in our screens. Here is some code snippet that might help you.

```TypeScript
export interface DispatchProps {
  dispatch: Dispatch
}

// or define a Dispatch in the 'mapDispatchToProps'
const mapDispatchToProps = (dispatch: Dispatch<AnyAction>) => {
  return {
    setTheme: (theme: string) => {
      const action = createSetThemeAction(theme);
      dispatch(action);
    }

  };
};
```


## IV. Test
When it comes to mock some function or field, TypeScript does have some limits about dynamic extension. And dynamic extension sometimes would be very important. 

The code below is an obvious example. `jest.mock()` just inject some mock methods to the `Worker` file, but TypeScript would never get to know this kinds of injection, and then an error would be generated.

```TypeScript
import { work } from "../Worker"

jest.mock("../Worker")

test("some...", ()=>{
  work.mockReturnThis(); // ERROR!!!, as TypeScript does not know this method exist
  ...
})
```

A temporary work around would be :

```TypeScript
  // @ts-ignore
  work.mockReturnThis()
```


`work.mockReturnThis()` : why js, not ts?

## V. Others

### 1. Lazy Init
When we need to define a constant, and also this constant will be initilized later, what could we do?   (p.s. If you are a Kotlin user, then here we want to make a `lateinit var` effect. )

Here an `as` operator might help you, just like the below code:

```TypeScript
interface People {
  id: number,
  name: string
}

...
// const p = {}  // ERROR! `{}` and `People` are not compatilbe
const p = {} as People
p.id = 100
```

By doing this, we told TypeScript to leave it alone. "This is a People type, and you don't need to check it again". After all, after the compilation, the generated JavaScript code has no problem to handle such code at all.

Also, `as` or `any` are super powerful in TypeScript, they will force the code to do what you want to do. But they are potential very damage to your code, `any` everywhere actually just turn your code into JavaScript, which is hard to know the types of function, arguments, and other all things. 


### 2. Generics
If you define something using generics, you may find out TypeScript yells at you that something is wrong, just like this one:

![](./images/2019-08-06-001.png)

If you've read the TypeScript handbook, you might find it odd, as the code in the previous picture is absolutely right. Yes, you are right, and the code is right as well.

But, if this code lies in a `tsx` file (, rather than a `ts` file), then TypeScript would have a hard time to understand this generics `<T>`. Aka, since it is a `tsx` file, so TypeScript would assume this `<T>` is a JSX element, but it makes no sense to place a JSX element in the lambda declaration. That's why we got an error at the first place.

The solution to fix it is to add `<T extends object>` to tell TypeScript, "hey, man, this is a generics, not a JSX element. Please don't mess it up".  

Here is the correct way: 

```TypeScript
// ***.tsx
const example = <T extends object>(url: T) : number => {
  return 20;
};

```


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

# II. React, React Native


## 1. 类组件
可以声明Props, State的类型, 这样TypeScript会帮你检查props/state是否类型有误


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


## 2. 函数组件
函数组件在以前是没有State的, 所以以前的函数组件的类型叫SFC(Stateless Function Component). 但自从有了React Hooks (React>=16.8, RN>=0.59), 函数组件也可以有state. 所以SFC类型就被废弃了, 而函数组件的类型也改为React.FC.


```TypeScript
interface IProps {

name: string;

}


const SomeScreen = (props: IProps) => {

const [offset, setOffset] = useState<number>(0);

console.log(props.name);
};
```


## 3. 根据数据而动态生成的子组件
你可以声明为`JSX.Element[]`. 例子如下:


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


## 4. defaultProps
这个其实和js很像, class组件中仍是声明一个`static defaultProps = {一个对象}`. 函数组件就是使用 `MyFC.defaultProps = {一个对象}`



例子就是:


```TypeScript
interface IProps {

id: number;

text?: string;

}


const MyView = (props: IProps) => {

return (

<>
<p style={{ margin: "20px", fontSize: "50px" }}>{props.id} -- {props.text}</p>

</>
);
};


MyView.defaultProps = {

text: "default"

};
```
但这样的写法有一个问题, 就是你甚至可以声明 `MyView.defaultProps = {balabala: 'whatever'}`. 明明balabala不在IProps中, 但TS仍不报错. [这篇文章](https://medium.com/@martin_hotell/react-typescript-and-defaultprops-dilemma-ca7f81c661c7)就这一问题展开了详细的论述. 不过我觉得太过复杂, 我仍是喜欢用上面的defaultProps.



## 5. ref
就类组件, 函数组件有三种不同的方案


### 1). React #1
```TypeScript
// React (Approach 1)
const MyView = () => {

let viewRef : HTMLDivElement | null;

return (

<div ref={v => viewRef = v} />

);
};
```


### 2). React #2
```TypeScript
// React (Approach 2)
const MyView = () => {

const viewRef = createRef<HTMLDivElement | null>();

return (

<div ref={viewRef}/>
);
};
```


### 3). React Native
```TypeScript
const MyView = ()=>{

let ref: View|null = null ;

let imageRef = createRef<Image>();



return (

<View ref={ref}>
<Image ref={imageRef} source={require("../a.png")} />

</View>
)
}
```


## 6. HoC
高阶组件, 名字是叫组件, 但其实它本身就是一个函数. 只不过是一个接受组件为参数, 返回另一个组件的函数而已. 在TS里, 自然要给明类型. 关键是TS中还有props这个泛型, 所以要多花一点心思. 下面就是我写过的一个加loading组件的HoC, 表示加载中就显示小菊花, 加载完就展示作为参数的组件.


```TypeScript
interface IProps {

loading: boolean;

}


const withLoader = <P extends object>(InputComponent: React.ComponentType<P>): React.FC<P & IProps> => {

props.loading ? (... ) : (...)

...
;
```


1) 这里就看到了吧, TS的类型比较灵活, 我们可以使用`P & IProps`. 这个在java中就得再新建个接口, 来包一起了.


2). `<P extends object>` 是个什么鬼? 其实它就是个泛型, 我在后面会讲为何不像java一样, 就用个`<P>`, 而要加一个`extends object`.



3). 组件是可以是类组件, 也可以是函数组件. 所以我们就用了React.ComponentType.
ComponentType的源码其实就已经很清晰了: `type ComponentTYpe<P = {}> = ComponentClass<P> | FunctionComponent<P>;`



# III. React-Navigation
React-Navigation是一个React Native的库, 用来跳转. 但平时在js中, 我们在跳转时, 我们是这样用`props.navigation.navigate(...)`. 现在就有问题了, 这个`props.navigation`是哪来的呢? 你要是不声明这个navigation, 那TypeScript就会向你大声叫唤: "我不知道这个navigation哪来的!" 这样的场景其实是有不少的, 所以这一大节专门讲React-Navigation与TypeScript的兼容.


## 1. props


```TypeScript
interface Iprops {

count: number;

}
const MyView = (props: IProps) => {

return (

<Button onPress={()=> props.navigation.navigation("secondPage")} title="go"/>
) // ERROR

}
```


(image)


我们可以加一个navigation的成员到IProps里, 不过要是以后React-Navigation改navigation, 改为nav, 那我们就要每个页面的IProps去修改名字. 所以不如我们使用react-navigation自带的`NavigationScreenProps`. 所以更好的做法应该是:


```TypeScript
interface ISplashScreen {

duration: number;

}


type IProps = ISplashScreen & NavigationScreenProps;



export class SplashScreen extends Component<IProps> {

componentDidMount() {
setTimeout(() => this.props.navigation.navigate("app"), 2000);

}
}
```


## 2. override shared navigationOptions
基本上每个页面都是共享一个状态栏(即在top的一栏, 有页面名字, 有后退键的那一栏. Android中叫ToolBar).


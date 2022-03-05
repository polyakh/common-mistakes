#### Prop Drilling

[A better way of solving prop drilling in React apps](https://blog.logrocket.com/solving-prop-drilling-react-apps/)
If you only want to avoid passing some props through many levels, component composition is often a simpler solution than
context.

```typescript jsx
// ❌ Problem:
const PropDrilling = () => {
    const [count] = useState(0);
    return <Child count={count}/>
}
const Child = ({count}) => <GrandChild/>

const GrandChild = ({count}) => <div>{count}</div>

// ✅ Solution:
// 1. CONTEXT API
<Provider><Child/></Provider>

// 2. Component composition
// Container components (components can contain different varieties of properties, including `other components`):
// 1. By explicitly passing one or more component(s) to another component as that component’s prop
const AppE1 = () => {
    const [data, setData] = useState("some state");
    return <ComponentOne ComponentTwo={<ComponentTwo data={data}/>}/>;
}

const ComponentOneE1 = ({ComponentTwo}) => <>
    <p>This is Component1, it receives component2 as a prop and renders it</p>
    {ComponentTwo}
</>

const ComponentTwoE1 = ({data}) => <h3>This is Component two with the received state {data}</h3>;
// 2. By wrapping a parent component around one or more children component(s)
const AppE2 = () => {
    const [data, setData] = useState("some state");
    return (
        <ParentComponent>
            <ComponentOneE2>
                <ComponentTwoE2 data={data}/>
            </ComponentOneE2>
        </ParentComponent>
    );
}
const ParentComponentE2 = ({children}) => <div>{children}</div>;

const ComponentOneE2 = ({children}) => <>
    <p>This is Component1, it receives component2 as a child and renders it</p>
    {children}</>


const ComponentTwoE2 = ({data}) => <h3>This is Component two with the received {data}</h3>

// Specialized components:
// A specialized component is a generic component that is conditionally created to render specialized variants of itself 
// by passing in props that match the conditions for a specific variant.
const App = () => <PopupModal title="Welcome" message="A popup modal">
    <UniqueContent/>
</PopupModal>

const PopupModal = ({title, message, children}) => <>
    <h1 className="title">{title}</h1>
    <p className="message">{message}</p>
    {children && children}
</>

const UniqueContent = () => <div>Unique Markup</div>

```

[🧹 Erase non-html attributes when spreading props](https://www.freecodecamp.org/news/best-practices-for-react/#-erase-non-html-attributes-when-spreading-props)
!! The rule of thumb: Generally avoid passing props using the spread operator. One time when it’s justified is when
writing a container component or HOC that renders and enhances its children.

```typescript jsx
// ❌ Problem:
const IndexPage = () => <MainTitle isBold hasPadding>
    Welcome to our new site!
</MainTitle>


// MainTitle Component
const MainTitle = ({isBold, children, ...restProps}) => <h1 style={{
    fontWeight: isBold ? 600 : 400,
    padding: restProps.hasPadding ? 16 : 0
}}
                                                            {...restProps}
>
    {children}
</h1> // when you open your browser you'll receive a warning that hasPadding is a non-HTML attribute you're trying to apply on the h1 tag. 


// ✅ Solution:
const MainTitleEX2 = ({isBold, children, hasPadding, ...restProps}) => {...
};
// To avoid this, always extract all non-HTML attributes from the props first, 
// to make sure that there are only valid HTML attributes in restProps that you're spreading onto a JSX element.
```

[#### Passing too much information to components](https://itnext.io/react-antipatterns-to-avoid-350929bdebf0#cef4)
Try to keep the separation between smart and presentational components(output HTML, not hold state and are not handling
any behavioral logic) in mind when deciding how much data to pass.
!! You could instead pass the user’s(object) first name, last name, and date => This makes it easier to reduce the
number of re-renders using React.memo.

[#### Be discriminating about what your components get to decide](https://itnext.io/react-performance-how-to-avoid-redundant-re-renders-6a33618d92a3#35cf)
Try to avoid passing any of the state variables of the parent to the child. Passing parent’s state variables will cause
ALL of the list items to re-render each time that variable changes.

```typescript jsx
// ❌ Problem:
const List = (data) => {
    const [selectedId, setSelectedId] = useState(-1);
    useEffect(() => setSelectedId(2), []); //  Doing so will trigger another re-render of the parent component after the initial render.
    return map(item => {
        <ListItem id={item.id}/>
    })
}
const ListItem = memo(({id, selectedId, title}) => {
    if (selectedId === id) return <p>Selected: {title}</p>
    return <p>{title}</p>
})
// ✅ Solution:
const ListEX2 = (data) => {
    const [selectedId, setSelectedId] = useState(-1);
    useEffect(() => setSelectedId(2), []);
    return map(item => {
        <ListItem isSelected={item.id === selectedId}/>
    }) // the items that weren't selected didn't re-render since none of their props changed.
}
const ListItemEX2 = memo(({isSelected, title}) => {
    if (isSelected) return <p>Selected: {title}</p>
    return <p>{title}</p>
})
```
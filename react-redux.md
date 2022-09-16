#### Ways to Avoid React Component(children) Re-Renderings

Children will get re-render every time when we change something in the state.

```typescript jsx
// ❌ Problem:
const Children = () => <div>Children</div>;
const Parent = () => <StateContext.Provider value={{state}}>
    <ChildComponent/> { /* we are basically creating new React.Node every time, so all the children are re-rendered */}
</StateContext.Provider>
// ✅ Solution:

const MChildren = () => React.memo(() => <div>MChildren</div>);
const MParent = ({ children }) => <StateContext.Provider value={{state}}>
    {children} { /*  */}
</StateContext.Provider>
```

Articles:
[React doesn't need state management tool, I said](https://dev.to/tolgee_i18n/react-doesnt-need-state-management-tool-i-said-31l4)
---
slug: react-redux
title: Between React & Redux
tags: [State Management, Performance]
---

## Redux

1. 狀態機/狀態管理工具, 以 flux 架構設計
2. 和框架無關, react, vue, 任何框架都可以拿來解決狀態問題
3. 全域 **"單一實例(Single Instance)"**, 設計來管理 **"整個 app"** 都用到的狀態
4. 透過 **Context 從頂層傳遞 store 給元件**

### Store

> 一個有 getState, dispatch, subscribe 方法可以用的物件

```jsx
function createStore(reducer) {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach((listener) => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter((l) => l !== listener);
    };
  };

  dispatch({}); // 初始化狀態

  return { getState, dispatch, subscribe };
}
```

### Provider

> 透過 React Context API 傳遞 store

```jsx
const store = createStore(counterReducer);
const ReactReduxContext = createContext();

function Provider({ store, children }) {
  return (
    <ReactReduxContext.Provider value={store}>
      {children}
    </ReactReduxContext.Provider>
  );
}

<Provider store={store}>
  <App />
</Provider>;
```

### useSelector

```js
function useSelector(selector) {
  const [, forceRender] = useReducer((counter) => counter + 1, 0);
  const store = useContext(ReactReduxContext);

  const selectedValueRef = useRef(selector(store.getState()));

  // before browser paint, store.subscribe(syncView)
  useLayoutEffect(() => {
    function syncView() {
      const storeState = store.getState();
      const latestSelectedValue = selector(storeState);

      if (latestSelectedValue !== selectedValueRef.current) {
        selectedValueRef.current = latestSelectedValue;
        forceRender();
      }
    }

    const unsubscribe = store.subscribe(syncView);

    return unsubscribe;
  }, [store]);

  return selectedValueRef.current;
}
```

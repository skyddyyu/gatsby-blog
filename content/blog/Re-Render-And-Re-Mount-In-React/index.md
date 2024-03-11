---
title: React中重新渲染和重新挂载
date: "2023-10-29"
description: "Hello World"
---

1.状态下放

```react
const Child = () => {
  console.log("Child render");
  return (
    <div>
      <span>Child</span>
    </div>
  );
};

const Parent = () => {
  const [count, setCount] = useState(0);
  console.log("Parent render");
  return (
    <div>
      <span>count: {count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
      <Child />
    </div>
  );
};
```

> 上面例子中，Parent组件的state发生变化后Child子组件也会跟着渲染，因为React的渲染机制，父组件re-render的时候，子组件未使用React.memo包裹的时候是一定会发生重新渲染的，该渲染是不必要的。

由于count状态在Parent组件中只被span和button所引用，而Child组件未使用。因此可以将Parent中的count状态下放，并抽取一个新组件Counter。**组件的状态应该尽可能的少，尽可能的下放，这样可以避免不必要的重新渲染。**

下放之后的效果如下：

```React
const Child = () => {
  console.log("Child render");
  return (
    <div>
      <span>Child</span>
    </div>
  );
};

const Couter = () => {
  console.log("Counter render");
  return (
    <span>count: {count}</span>
    <button onClick={() => setCount(count + 1)}>+</button>
  );
};

const Parent = () => {
  console.log("Parent render");
  return (
    <div>
      <Counter/>
      <Child />
    </div>
  );
};
```

此时当Counter组件的状态发生变化的时候，原有的Child组件并不会发生重新渲染。

2.组合或memo

①使用组合

当未使用组合时，下面代码中的Child组件会随着Parent组件状态的改变而re-render

```react
// 未使用组合时
const Child = () => {
    console.log("Child Render");
    return <div>It is Child</div>;
}

const Parent = () => {
    const [count, setCount] = useState(0);
    console.log("Parent Render");
    return (<div>
        <div>
            <p>count: <span style={{color: 'red', fontSize: 24}}>{count}</span></p>
            <button
                type="button"
                onClick={() => {
                    setCount(count + 1)
                }}
                style={{fontSize: 22}}>+1
            </button>
        </div>
        <Child/>
    </div>);
}

const App = () => {
    console.log("APP Render");
    return (
        <div>
            <Parent/>
        </div>
    );
};
```

> 而当我们使用组合时，**将Child组件以Props的children形式进行传递给Parent，此时Child组件相当于提了一个层级**，也就是在App组件中。此时Parent组件状态发生改变，Child组件不会发生re-render。

```react
// 使用组合之后的代码如下
const Child = () => {
    console.log("Child Render");
    return <div>It is Child</div>;
}

const Parent: FC<{ children: React.ReactNode }> = ({children}) => {
    const [count, setCount] = useState(0);
    console.log("Parent Render");
    return (<div>
        <div>
            <p>count: <span style={{color: 'red', fontSize: 24}}>{count}</span></p>
            <button
                type="button"
                onClick={() => {
                    setCount(count + 1)
                }}
                style={{fontSize: 22}}>+1
            </button>
        </div>
        {children}
    </div>);
}

const App = () => {
    console.log("APP Render");
    return (
        <div>
            <Parent>
                <Child/>
            </Parent>
        </div>
    );
};
```

②使用memo

```react
// 使用memo的代码如下
const Child = React.memo(() => {
    console.log("Child Render");
    return <div>It is Child</div>;
});

const Parent = () => {
    const [count, setCount] = useState(0);
    console.log("Parent Render");
    return (<div>
        <div>
            <p>count: <span style={{color: 'red', fontSize: 24}}>{count}</span></p>
            <button
                type="button"
                onClick={() => {
                    setCount(count + 1)
                }}
                style={{fontSize: 22}}>+1
            </button>
        </div>
        <Child/>
    </div>);
}
```

*什么时候使用组合，什么时候使用memo*

```react
const Child = React.memo(({style, count, onSuccess}) => {

    console.log("Child Render")

    return (<div>
        <div style={{...style, fontSize: 17}}>
            {count}
        </div>
        <button type={'button'} onClick={() => {
            onSuccess();
        }}>点击我提交成功
        </button>
    </div>);
});

const Parent = () => {

    console.log("Parent Render")

    const [justP, setJustP] = useState('');
    const [count, setCount] = useState(0);

    const style = useMemo(() => {
        return {
            color: count % 2 === 0 ? 'red' : 'green',
        }
    }, [count])

    const onSuccess = useCallback(() => {
        console.log("表单提交成功的回调");
    }, [])

    return <div>
        <div>
            <input value={justP} onChange={(e) => setJustP(e.target.value)}/>
            <div style={{fontSize: 24, color: 'skyblue'}}>{count}</div>
            <button type='button' onClick={() => {
                setCount(count + 1)
            }}>+1
            </button>
        </div>
        <Child count={count} style={style} onSuccess={onSuccess}/>
    </div>
}
```



当我们使用组合的方式将Child以props的形式传递给Parent。此时Child无法获取Parent的props，也就无法直接通过props进行通信，此时就应该使用memo。使用memo的好处就是只有传递进来的相关props的值发生了改变才会触发组件的re-render。上面例子使用memo包裹后，只有Child直接使用的count状态发生改变时，Child才会re-render，而justP发生改变的时候，child不会发生re-render。**另外需要注意的是，当传递进来的props中的属性中有引用类型和函数类型时，需要使用useMemo和useCallback进行包裹保证引用一致。**

组合它还有许多其他的用途：用来拆分巨型组件、提高组件的可复用性、提高组件的可测试性，解决属性透传（Props drilling）等。

3.列表渲染优化

```react
const Child = () => {
    console.log("Child render");

    useEffect(() => {
        console.log("Child mount");
    }, []);

    return (
        <div>
            <span>Child</span>
        </div>
    );
};
const list = [1, 2, 3, 4, 5];
const Parent = () => {
    const [count, setCount] = useState(0);

    return (
        <div>
            <button onClick={() => setCount(count + 1)}>+</button>
            {list.map((item) => (
                <Child key={item}/>
            ))}
        </div>
    );
};

```



如上代码中，list的值是固定的，那么每次map的时候<Child/>的key值都都不会发生改变。但是当我们点击button触发Parent状态更新的时候Child却打印了5次“Child render”。那么key的作用到底是什么呢？

**Key是用来优化DOM的，而不是用来优化re-render,如果key不变，组件的type也不变，那么React会复用DOM节点，提高性能。**因此上面例子中没有打印“Child mount”二十打印了5次“Child render”。

> 在React中的源码中的reconcile 阶段，React会根据key和type的值来直接复用Fiber Node,Fiber Node中的stateNode就是DOM节点。所以key的作用是优化DOM节点的复用，与re-render无关。如果我们要优化re-render我们应该使用React.memo来包裹Child组件。

假如是 React 新手，可能会写出这样的代码：

```react
const Parent = () => {
  const [count, setCount] = useState(0);

  const Child = () => {
    console.log("Child render");

    useEffect(() => {
      console.log("Child mount");
    }, []);

    return (
      <div>
        <span>Child</span>
      </div>
    );
  };

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>+</button>
      {list.map((item) => (
        <Child key={item} />
      ))}
    </div>
  );
};
```

将Child组件的定义放在Parent组件里面，此时点击button更新状态，“Child render”和“Child mount”都会被打印。但是此时key并没有变化，为什么会re-mount呢。

这是因为，Child组件被定义在Parent里面；Parent状态发生改变，Parent重新执行每次都会重新创建Child函数，也就每次都会生成新的React Element创建新的Fiber Node。所以re-mount和re-render都会发生。

*我们应该尽量避免组件re-render,组件re-render时内部的state状态会丢失。由于DOM节点的重新创建和插入，也会导致一些表单元素的光标丢失，页面闪烁，useEfeect重新执行等问题。*
总结：**type和key变化会触发re-mount，而props变化会触发re-render。**

4.Context优化相关

```react
const ThemeContext = React.createContext({
  theme: "light",
  setTheme: () => {},
});

const Toolbar = React.memo(() => {
  const { theme, setTheme } = useContext(ThemeContext);
  console.log("Toolbar render");
  return (
    <div>
      <button onClick={() => setTheme("dark")}>dark</button>
      <button onClick={() => setTheme("light")}>light</button>
    </div>
  );
});

const Btn = (props) => {
  console.log("Btn render");
  return <button onClick={props.onClick}>{props.count}</button>;
};
const App = () => {
  // App 内部的 state
  const [count, setCount] = useState(0);
  // context 相关state
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Btn count={count} onClick={() => setCount(count + 1)} />
      <Toolbar />
    </ThemeContext.Provider>
  );
};
```

当我们点击Btn的时候"Toolbar render"会打印，但是Toolbar并没有依赖count。且Toolbar也被memo优化了，还是re-render了。当我们切换Toolbar里面的theme的时候，发现"Btn render"也会被打印，但是btn并没有使用Context。

下面使用**组合**和**useMemo**来解决该问题，代码如下：

```react
const ThemeContext = React.createContext({
    theme: "light",
    setTheme: () => {
    },
});

const Toolbar = React.memo(() => {
    const {theme, setTheme} = useContext(ThemeContext);
    console.log("Toolbar render");
    return (
        <div>
            <button onClick={() => setTheme("dark")}>dark</button>
            <button onClick={() => setTheme("light")}>light</button>
        </div>
    );
});

const Btn = (props) => {
    console.log("Btn render");
    return <button onClick={props.onClick}>{props.count}</button>;
};

const ThemeProvider: FC<{ children: React.ReactNode }> = ({children}) => {
    // context 相关state
    const [theme, setTheme] = useState("light");

    const value = useMemo(() => {
        return {theme, setTheme};
    }, [theme]);

    return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
}

const App = () => {
    // App 内部的 state
    const [count, setCount] = useState(0);

    return (
        <ThemeProvider>
            <Btn count={count} onClick={() => setCount(count + 1)}/>
            <Toolbar/>
        </ThemeProvider>
    );
};
```



[本文章原文出处：掘金【viewer_w】](https://juejin.cn/post/7254443448562974775)
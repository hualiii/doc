## react 核心概念

### 虚拟dom 真实dom

#### 虚拟dom的两种创建方式

1. jsx创建虚拟dom

```jsx
const VDOM = (
    <div>
        <h2 className="title" id={myId.toLowerCase()}>
            <span style={{color:'white',fontSize:'29px'}}>{myData.toLowerCase()}</span>
        </h2>
        <h2 className="title" id={myId.toUpperCase()}>
            <span style={{color:'white',fontSize:'29px'}}>{myData.toLowerCase()}</span>
        </h2>
        <input type="text"/>
    </div>
)
```

2. React.createElement()创建虚拟dom

```jsx
React.createElement('h1',{id:'title'},React.createElement('span',{},'Hello,React'))
```

#### jsx语法规则

1. 定义虚拟DOM时，不要写引号。
1. 标签中混入JS表达式时要用{}。
1. 样式的类名指定不要用class，要用className。
1. 内联样式，要用style={{key:value}}的形式去写。
1. 只有一个根标签
1. 标签必须闭合
1. 标签首字母 
   1. 若小写字母开头，则将该标签转为html中同名元素，若html中无该标签对应的同名元素，则报错。
   1. 若大写字母开头，react就去渲染对应的组件，若组件没有定义，则报错。

#### 虚拟dom和真实dom的区别

1. 本质是Object类型的对象（一般对象）
1. 虚拟DOM比较“轻”，真实DOM比较“重”，因为虚拟DOM是React内部在用，无需真实DOM上那么多的属性。
1. 虚拟DOM最终会被React转化为真实DOM，呈现在页面上。

### 组件的定义

#### 函数式组件

```jsx
//1.创建函数式组件
function MyComponent(){
    console.log(this); //此处的this是undefined，因为babel编译后开启了严格模式
    return <h2>我是用函数定义的组件(适用于【简单组件】的定义)</h2>
}
//2.渲染组件到页面
ReactDOM.render(<MyComponent/>,document.getElementById('test'))
/* 
    执行了ReactDOM.render(<MyComponent/>.......之后，发生了什么？
            1.React解析组件标签，找到了MyComponent组件。
            2.发现组件是使用函数定义的，随后调用该函数，将返回的虚拟DOM转为真实DOM，随后呈现在页面中。
*/
```

#### 类式组件

```jsx
//1.创建类式组件
class MyComponent extends React.Component {
    render(){
        //render是放在哪里的？—— MyComponent的原型对象上，供实例使用。
        //render中的this是谁？—— MyComponent的实例对象 <=> MyComponent组件实例对象。
        console.log('render中的this:',this);
        return <h2>我是用类定义的组件(适用于【复杂组件】的定义)</h2>
    }
}
//2.渲染组件到页面
ReactDOM.render(<MyComponent/>,document.getElementById('test'))
/* 
    执行了ReactDOM.render(<MyComponent/>.......之后，发生了什么？
            1.React解析组件标签，找到了MyComponent组件。
            2.发现组件是使用类定义的，随后new出来该类的实例，并通过该实例调用到原型上的render方法。
            3.将render返回的虚拟DOM转为真实DOM，随后呈现在页面中。
*/
```

### 组件的三大属性

#### state

> state 是react的状态管理，更改state用this.setState({key:newValue})方法。


```jsx
//1.创建组件
class Weather extends React.Component{

    //构造器调用几次？ ———— 1次
    constructor(props){
        console.log('constructor');
        super(props)
        //初始化状态
        this.state = {isHot:false,wind:'微风'}
        //解决changeWeather中this指向问题
        this.changeWeather = this.changeWeather.bind(this)
    }

    //render调用几次？ ———— 1+n次 1是初始化的那次 n是状态更新的次数
    render(){
        console.log('render');
        //读取状态
        const {isHot,wind} = this.state
        return <h1 onClick={this.changeWeather}>今天天气很{isHot ? '炎热' : '凉爽'}，{wind}</h1>
    }

    //changeWeather调用几次？ ———— 点几次调几次
    changeWeather(){
        //changeWeather放在哪里？ ———— Weather的原型对象上，供实例使用
        //由于changeWeather是作为onClick的回调，所以不是通过实例调用的，是直接调用
        //类中的方法默认开启了局部的严格模式，所以changeWeather中的this为undefined

        console.log('changeWeather');
        //获取原来的isHot值
        const isHot = this.state.isHot
        //严重注意：状态必须通过setState进行更新,且更新是一种合并，不是替换。
        this.setState({isHot:!isHot})
        console.log(this);

        //严重注意：状态(state)不可直接更改，下面这行就是直接更改！！！
        //this.state.isHot = !isHot //这是错误的写法
    }
}
//2.渲染组件到页面
ReactDOM.render(<Weather/>,document.getElementById('test'))
```

#### props

基本使用：

```jsx
//创建组件
class Person extends React.Component{
   render(){
      // console.log(this);
      const {name,age,sex} = this.props
      return (
          <ul>
             <li>姓名：{name}</li>
             <li>性别：{sex}</li>
             <li>年龄：{age+1}</li>
          </ul>
      )
   }
}
//渲染组件到页面
ReactDOM.render(<Person name="jerry" age={19}  sex="男"/>,document.getElementById('test1'))
ReactDOM.render(<Person name="tom" age={18} sex="女"/>,document.getElementById('test2'))

const p = {name:'老刘',age:18,sex:'女'}
// console.log('@',...p);
// ReactDOM.render(<Person name={p.name} age={p.age} sex={p.sex}/>,document.getElementById('test3'))
ReactDOM.render(<Person {...p}/>,document.getElementById('test3'))
```

对props进行限制：

```jsx
//创建组件
class Person extends React.Component{
   render(){
      // console.log(this);
      const {name,age,sex} = this.props
      //props是只读的
      //this.props.name = 'jack' //此行代码会报错，因为props是只读的
      return (
          <ul>
             <li>姓名：{name}</li>
             <li>性别：{sex}</li>
             <li>年龄：{age+1}</li>
          </ul>
      )
   }
}
//对标签属性进行类型、必要性的限制
Person.propTypes = {
   name:PropTypes.string.isRequired, //限制name必传，且为字符串
   sex:PropTypes.string,//限制sex为字符串
   age:PropTypes.number,//限制age为数值
   speak:PropTypes.func,//限制speak为函数
}
//指定默认标签属性值
Person.defaultProps = {
   sex:'男',//sex默认值为男
   age:18 //age默认值为18
}
//渲染组件到页面
ReactDOM.render(<Person name={100} speak={speak}/>,document.getElementById('test1'))
ReactDOM.render(<Person name="tom" age={18} sex="女"/>,document.getElementById('test2'))

const p = {name:'老刘',age:18,sex:'女'}
// console.log('@',...p);
// ReactDOM.render(<Person name={p.name} age={p.age} sex={p.sex}/>,document.getElementById('test3'))
ReactDOM.render(<Person {...p}/>,document.getElementById('test3'))

function speak(){
   console.log('我说话了');
}
```

函数式组件使用props

```jsx
function Person (props){
    const {name,age,sex} = props
        return (
            <ul>
               <li>姓名：{name}</li>
               <li>性别：{sex}</li>
               <li>年龄：{age}</li>
            </ul>
        )
}
Person.propTypes = {
   name:PropTypes.string.isRequired, //限制name必传，且为字符串
   sex:PropTypes.string,//限制sex为字符串
   age:PropTypes.number,//限制age为数值
}
//指定默认标签属性值
Person.defaultProps = {
   sex:'男',//sex默认值为男
   age:18 //age默认值为18
}
//渲染组件到页面
ReactDOM.render(<Person name="jerry"/>,document.getElementById('test1'))
```

#### ref

字符串使用ref

```jsx
class Demo extends React.Component{
   //展示左侧输入框的数据
   showData = ()=>{
      const {input1} = this.refs
      alert(input1.value)
   }
   //展示右侧输入框的数据
   showData2 = ()=>{
      const {input2} = this.refs
      alert(input2.value)
   }
   render(){
      return(
          <div>
             <input ref="input1" type="text" placeholder="点击按钮提示数据"/>&nbsp;
             <button onClick={this.showData}>点我提示左侧的数据</button>&nbsp;
             <input ref="input2" onBlur={this.showData2} type="text" placeholder="失去焦点提示数据"/>
          </div>            
      )
   }
}			
//渲染组件到页面
ReactDOM.render(<Demo a="1" b="2"/>,document.getElementById('test'))
```

回调函数使用ref

```jsx
class Demo extends React.Component{
//展示左侧输入框的数据
   showData = ()=>{
      const {input1} = this
      alert(input1.value)
   }
   //展示右侧输入框的数据
   showData2 = ()=>{
      const {input2} = this
      alert(input2.value)
   }
   render(){
      return(
          <div>
             <input ref={c => this.input1 = c } type="text" placeholder="点击按钮提示数据"/>&nbsp;
             <button onClick={this.showData}>点我提示左侧的数据</button>&nbsp;
             <input onBlur={this.showData2} ref={c => this.input2 = c } type="text" placeholder="失去焦点提示数据"/>&nbsp;
          </div>
      )
   }
}
//渲染组件到页面
ReactDOM.render(<Demo a="1" b="2"/>,document.getElementById('test'))
```

createRef

```jsx
class Demo extends React.Component{
    /*
        React.createRef调用后可以返回一个容器，该容器可以存储被ref所标识的节点,该容器是“专人专用”的
     */
    myRef = React.createRef()
    myRef2 = React.createRef()
    //展示左侧输入框的数据
    showData = ()=>{
        alert(this.myRef.current.value);
    }
    //展示右侧输入框的数据
    showData2 = ()=>{
        alert(this.myRef2.current.value);
    }
    render(){
        return(
            <div>
                <input ref={this.myRef} type="text" placeholder="点击按钮提示数据"/>&nbsp;
                <button onClick={this.showData}>点我提示左侧的数据</button>&nbsp;
                <input onBlur={this.showData2} ref={this.myRef2} type="text" placeholder="失去焦点提示数据"/>&nbsp;
            </div>
        )
    }
}
//渲染组件到页面
ReactDOM.render(<Demo a="1" b="2"/>,document.getElementById('test'))
```

### 事件处理

1. 通过onXxx属性指定事件处理函数(注意大小写) 
   1. React使用的是自定义(合成)事件, 而不是使用的原生DOM事件 —————— 为了更好的兼容性
   1. React中的事件是通过事件委托方式处理的(委托给组件最外层的元素) ————————为了的高效
2. 通过event.target得到发生事件的DOM元素对象 ——————————不要过度使用ref

```jsx
//创建组件
class Demo extends React.Component{
    /* 
        (1).通过onXxx属性指定事件处理函数(注意大小写)
                a.React使用的是自定义(合成)事件, 而不是使用的原生DOM事件 —————— 为了更好的兼容性
                b.React中的事件是通过事件委托方式处理的(委托给组件最外层的元素) ————————为了的高效
        (2).通过event.target得到发生事件的DOM元素对象 ——————————不要过度使用ref
     */
    //创建ref容器
    myRef = React.createRef()
    myRef2 = React.createRef()

    //展示左侧输入框的数据
    showData = (event)=>{
        console.log(event.target);
        alert(this.myRef.current.value);
    }

    //展示右侧输入框的数据
    showData2 = (event)=>{
        alert(event.target.value);
    }

    render(){
        return(
            <div>
                <input ref={this.myRef} type="text" placeholder="点击按钮提示数据"/>&nbsp;
                <button onClick={this.showData}>点我提示左侧的数据</button>&nbsp;
                <input onBlur={this.showData2} type="text" placeholder="失去焦点提示数据"/>&nbsp;
            </div>
        )
    }
}
//渲染组件到页面
ReactDOM.render(<Demo a="1" b="2"/>,document.getElementById('test'))
```

### 表单处理

#### 非受控组件

```jsx
//创建组件
class Login extends React.Component{
    handleSubmit = (event)=>{
        event.preventDefault() //阻止表单提交
        const {username,password} = this
        alert(`你输入的用户名是：${username.value},你输入的密码是：${password.value}`)
    }
    render(){
        return(
            <form onSubmit={this.handleSubmit}>
                用户名：<input ref={c => this.username = c} type="text" name="username"/>
                密码：<input ref={c => this.password = c} type="password" name="password"/>
                <button>登录</button>
            </form>
        )
    }
}
//渲染组件
ReactDOM.render(<Login/>,document.getElementById('test'))
```

#### 受控组件

```jsx
//创建组件
class Login extends React.Component{

    //初始化状态
    state = {
        username:'', //用户名
        password:'' //密码
    }

    //保存用户名到状态中
    saveUsername = (event)=>{
        this.setState({username:event.target.value})
    }

    //保存密码到状态中
    savePassword = (event)=>{
        this.setState({password:event.target.value})
    }

    //表单提交的回调
    handleSubmit = (event)=>{
        event.preventDefault() //阻止表单提交
        const {username,password} = this.state
        alert(`你输入的用户名是：${username},你输入的密码是：${password}`)
    }

    render(){
        return(
            <form onSubmit={this.handleSubmit}>
                用户名：<input onChange={this.saveUsername} type="text" name="username"/>
                密码：<input onChange={this.savePassword} type="password" name="password"/>
                <button>登录</button>
            </form>
        )
    }
}
//渲染组件
ReactDOM.render(<Login/>,document.getElementById('test'))
```

### 高阶函数_函数柯里化

高阶函数：如果一个函数符合下面2个规范中的任何一个，那该函数就是高阶函数.

1. 若A函数，接收的参数是一个函数，那么A就可以称之为高阶函数
1. 若A函数，调用的返回值依然是一个函数，那么A就可以称之为高阶函数
1. 常见的高阶函数有：Promise、setTimeout、arr.map()等等

函数的柯里化：通过函数调用继续返回函数的方式，实现多次接收参数最后统一处理的函数编码形式。

```javascript
function sum(a){
    return(b)=>{
        return (c)=>{
            return a+b+c
        }
    }
}
```

案例：

```jsx
// 创建虚拟DOM
class Login extends React.Component{
    constructor(props) {
        super(props);
        this.state = {
            username:'',
            password:''
        }
    }
    handleSubmit = (e)=>{
        e.preventDefault()
        const {username,password} = this.state
        alert(`username:${username};password:${password}`)
    }
    usernameHandle = (e)=>{
        console.log(e.target.value)
        this.setState({
            username:e.target.value
        })
    }
    passwordHandle = (e)=>{
        this.setState({
            password:e.target.value
        })
    }
    render(){
        return (
            <form onSubmit={this.handleSubmit}>
                <input type="text" value={this.state.username} onChange={this.usernameHandle}/>
                <br/>
                <input type="password" value={this.state.password} onChange={this.passwordHandle}/>
                <br/>
                <input type="submit" value="login"/>
            </form>
        )
    }
}
// 挂载虚拟DOM
ReactDOM.render(<Login/>,document.getElementById("app"))
```
### 组件的生命周期


#### 生命周期(旧)
![2_react生命周期(旧).png](https://cdn.nlark.com/yuque/0/2022/png/1525169/1652362620076-9cbd85f3-a6de-481d-a442-b4cb59f2c8d3.png#clientId=u30e988b7-6e16-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=536&id=u7f80904b&margin=%5Bobject%20Object%5D&name=2_react%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%28%E6%97%A7%29.png&originHeight=670&originWidth=841&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44022&status=done&style=none&taskId=u726f4047-d85a-4d48-8df5-7bafdf32498&title=&width=672.8)
#### 生命周期(新)![3_react生命周期(新).png](https://cdn.nlark.com/yuque/0/2022/png/1525169/1652362630662-cac25cf0-cc72-4959-8cf4-16bce0a943cf.png#clientId=u30e988b7-6e16-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=630&id=u85dd6f3d&margin=%5Bobject%20Object%5D&name=3_react%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%28%E6%96%B0%29.png&originHeight=788&originWidth=1133&originalType=binary&ratio=1&rotation=0&showTitle=false&size=66531&status=done&style=none&taskId=u92cddf2b-fd0b-4e91-ac43-9b2f54df3e2&title=&width=906.4)

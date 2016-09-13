#React学习笔记

##JSX语法

>HTML语言直接写在JavaScript语言之中，不加任何引号，它允许HTML与JS的混写.
>写法好别扭，每行结束不能写分号;
>使用的标签是<script text="text/babel"></script> 上线之前需要经过babel的编译

```
document.getElementById('example')
//不能写分号 否则就会报错
```
>基本语法规则就是：遇到HTML标签(以<开头)，就用HTML规则解析；遇到代码块(以{开头)，就用js规则解析。
>JSX允许直接在模板插入js变量。如果这个变量是个数组，则会展开这个数组的所有成员。

```
var arr = [
        <h1>Hello world!</h1>,
        <h2>React is awesome</h2>,
      ];
      ReactDOM.render(
        <div>{arr}</div>,
        document.getElementById('example')
      );
//在数组中也能直接用html语言作为数组成员 ReactDOM.render();中渲染所有的东西     
```

##组件

>React允许将代码封装成组件(component)，然后像插入普通HTML标签一样，在网页中插入这个组件。React.createClass方法就用于生成一个组件类
>模板插入<HelloMessage />时，会自动生成HelloMessage的一个实例。所有组件类都必须有自己的render方法，用于输出组件

```
//HelloMessage 就是一个组件类
var HelloMessage = React.createClass({
        render: function() {
          //this.props --- 也就是dom中的属性值 组件的属性可以在组件类的 this.props 对象上获取，比如 name 属性就可以通过 this.props.name 读取。
          return <h1>Hello {this.props.name}</h1>;
        }
});
//实际使用方式
ReactDOM.render(
    <HelloMessage name="JV" />
    document.getElementById('example')
);
```
>注意，组件类第一个字母必须大写，否则就会报错,比如HelloMessage不能写成helloMessage。另外，组件类智能包含一个顶层标签，否则也会报错。
>添加属性class时需要写成className，for属性需要写成htmlFor因为class和for是js关键字.

##this.props.children

>this.props对象的属性与组件的属性一一对应。特殊的就是this.props.children属性---表示组件的所有子节点

```
var NotesList = React.createClass({
        render: function() {
          return (
            <ol>
              {
                React.Children.map(this.props.children, function (child) {
                  return <li>{child}</li>;
                })
              }
            </ol>
          );
        }
});
//实际使用的时候是根据组件中的子节点
ReactDOM.render(
        <NotesList>
          <span>hello</span>
          <span>world</span>
        </NotesList>,
        document.getElementById('example')
);      
```

>注意: this.props.children的值存在三种可能。如果当前组件没有子节点，它就是undefined；如果有一个子节点，数据类型就是object;如果有多个子节点，数据类型就是Array。所以处理this.props.Children需要注意
>React提供一个工具方法 React.Children来处理this.props.children.使用React.Children.map来遍历子节点，不去管子节点数据类型。

##PropTypes
>组件的属性可以接受任意值，字符串、对象、函数等等都可以。有时，我们需要一种机制，验证别人使用组件时，提供的参数是否符合要求。
>组件类的PropTypes属性，就是用来验证组件实例的属性是否符合要求.

```
var MyTitle = React.createClass({
        //验证title属性是否是字符串并且不能为空
        propTypes: {
          title: React.PropTypes.string.isRequired,
        },

        render: function() {
          return <h1> {this.props.title} </h1>;
        }
});
```
>getDefaultProps方法可以用来设置组件属性的默认值。

```
var MyTitle = React.createClass({
  //设置默认值    
  getDefaultProps : function () {
    return {
      title : 'Hello World'
    };
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});

```

##获取真实的DOM节点

>组件并不是真实的DOM节点，而是内存之中的一种数据结构,叫做虚拟DOM。只有当它插入文档以后，才会变成真实的DOM。
>在React设计上，所有DOM变动，都现在虚拟DOM上发生，然后再将实际发生变动的部分，反映在真实DOM上，这种算法叫做DOM diff。

>获取真实DOM的节点，需要用到ref属性。

```
var MyComponent = React.createClass({
  handleClick: function() {
    this.refs.myTextInput.focus();
  },
  render: function() {
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input type="button" value="Focus the text input" onClick={this.handleClick} />
      </div>
    );
  }
});
```
>必须是真实的DOM节点才能获取用户输入的。文本框需要有一个 ref属性，然后this.refs.[refName]就会返回这个真实的DOM节点。
>this.refs.[refName]属性获取的是真实的DOM，必须等到虚拟DOM插入文档以后才能使用属性
>React组件支持很多事件,除了Click事件外，还有KeyDown、Copy、Scroll等。前面增加一个handle +事件方式

##this.state

>组件免不了要与用户互动，React的一大创新，就是将组件看成是一个状态机，一开始有一个初始状态，然后用户互动，导致状态变化，从而触发重新渲染UI

```
var LikeButton = React.createClass({
  //初始化状态
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    //状态变化 取反
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});
```
>getInitialState方法定义初始状态，也就是一个对象，这个对象可以通过this.state属性读取。
>当用户点击组件，导致状态变化,this.setState方法就修改状态值，每次修改以后，自动调用this.render方法，再次渲染。

>由于this.props和this.state都用于描述组件的特性，可能会产生混淆。
>但是this.props表示那些一旦定义就不在改变的特性，而this.state是会随着用户互动而产生变化的特性。

##表单

>用户在表单填入的内容，属于用户跟组件的互动,所以不能用this.props读取
>可以采用this.state去获取属性值

```
var Input = React.createClass({
    getInitialState:function(){
        return {value:'Hello!'};
    },
    handleChange:function(event){
        //获取用户输入的值 event.target.value
        this.setState({value:event.target.value});
    },
    render:function(){
        var val = this.state.value;
        return (
            <div>
            <input type='text' value={value} onChange={this.handleChange}/>
            <p>{value}</p>
            </div>
        );
    }
});
```
>定义onChange事件作为回调函数,textarea元素、select元素、radio元素都属于这种情况 

##组件的生命周期

>组件的生命周期分为三个状态:

* Mounting:已插入真实DOM
* Updating:正在被重新渲染
* Unmounting:已移除真实DOM

>React为每个状态都提供两种处理函数,will函数在进入状态之前调用
>did函数在进入状态之后调用 

* componentWillMount()
* componentDidMount()
* componentWillUpdate(object nextProps,object nextState)
* componentDidUpdate(object preProps,object preState)
* componentWillUnmount();

>还有两种特殊状态的处理函数

* componentWillReceiveProps(object nextProps) :已加载组件收到新的参数时调用
* shouldComponentUpdate(object nextProps,object nextState) : 组件判断是否重新渲染时调用
 
```
var Hello = React.createClass({
    getInitialState:function(){
        return {
            opacity:1.0
        };
    },
    componentDidMount:function(){
        //定时器控制 重新设置style 引发重新渲染
        this.timer = setInterval(function(){
            var opacity = this.state.opacity;
            opacity -= 0.05;
            if(opacity<0.1){
                opacity = 1.0;
            }
            this.setState({
                opacity:opacity
            });
        }.bind(this),100);
    },
    render:function(){
        return {
            //不能写成 opacity:{this.state.opacity}  因为React组件样式是一个对象
            //第一重大括号表示这是 JavaScript 语法，第二重大括号表示样式对象
            <div style={{opacity:this.state.opacity}}>
                Hello {this.props.name}
            </div>
        }
    }
});
ReactDOM.render(
    <Hello name='world' />
    ddocument.body
);
```

##使用Ajax

>组件的数据来源，通常是通过ajax请求从服务器获取,可以使用componentDidMount方法设置Ajax请求，等到请求成功，
>再用this.setState方法重新渲染UI

```
var UserGist = React.createClass({
    getInitialState:function(){
        return {
            username:'',
            lastGistUrl:''
        };
    },
    componentDidMount:function(){
        $.get(this.props.source,function(result){
            var lastGist = result[0];
            if(this.isMounted()){
                this.setState({
                    username:lastGist.owner.login,
                    lastGistUrl:lastGist.html_url
                });
            }
        }.bind(this);
    },
    render:function(){
        return {
            <div>
                {this.state.username}'s last gist is
                <a href={this.state.lastGistUrl}>here</a>
            </div>
        };
    }
});
ReactDOM.render(
  <UserGist source="https://api.github.com/users/octocat/gists" />,
  document.body
);
```

>React本身是没有任何依赖的
>我们可以传入一个Promise对象到组件

```
var RepoList = React.createClass({
  getInitialState: function() {
    return {
      loading: true,
      error: null,
      data: null
    };
  },

  componentDidMount() {
      //promise对象处理
    this.props.promise.then(
      value => this.setState({loading: false, data: value}),
      error => this.setState({loading: false, error: error}));
  },

  render: function() {
    if (this.state.loading) {
      return <span>Loading...</span>;
    }
    else if (this.state.error !== null) {
      return <span>Error: {this.state.error.message}</span>;
    }
    else {
      var repos = this.state.data.items;
      var repoList = repos.map(function (repo, index) {
        return (
          <li key={index}><a href={repo.html_url}>{repo.name}</a> ({repo.stargazers_count} stars) <br/> {repo.description}</li>
        );
      });
      return (
        <main>
          <h1>Most Popular JavaScript Projects in Github</h1>
          <ol>{repoList}</ol>
        </main>
      );
    }
  }
});
ReactDOM.render(
  <RepoList promise={$.getJSON('https://api.github.com/search/repositories?q=javascript&sort=stars')} />,
  document.getElementById('example')
);
```
>从github上的API抓取数据,然后将Promise对象作为属性传给组件

##添加Markdown

>Markdown是一种简化的内联格式化你的文字的方法。使用第三方库remarkable，它接收Markdown文本并且转换为原始的HTML
>调用remarkable库,从this.props.children转换成remarkable能理解的原始字符串
>

```
var md = new Remarkable();
{md.render(this.props.children.toString())}
```
>为了避免受XSS攻击

```
rawMarkup:function(){
    var md = new Remarkable();
    var rawMarkup = md.render(this.props.children.toString());
    return {__html:rawMarkup};
}
<span dangerouslySetInnerHTML={this.rawMarkup()} />
```
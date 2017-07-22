# Es7 decorator

## 一，准备工作

> 1 安装

    npm i babel-plugin-transform-decorators-legacy babel-register --save-dev
    
    安装：
    babel-plugin-transform-decorators-legacy 
    babel-register
    
    transform-decorators-legacy：
    是第三方插件，用于支持decorators
    
    babel-register：
    用于接入node api
    
> 2 使用

方法一：require hook
    
创建 app.ts：
    
    @hairColor
    class Person { }
    
    function hairColor(target) {
       target.hair = "white";
    }
    
创建 complie.js

    require('babel-register')({
        plugins: ['transform-decorators-legacy']
    });
    require("./app.js")

执行 node complie.js

    其中：
    
    app：使用了修饰器的js
    complie：用来编译app
    node：执行编译器complie
    
    原理：
    1，node执行complie.js文件；
    2，complie文件改写了node的require方法；
    3，complie在引用app.js，使用了新的require方法；
    4，app.js在加载过程中被编译，并执行。
    
    好处：
    1，命令行无多余操作，与node集成了。
    2，不用显式的创建.babelrc。
    
    缺点：
    1.同步编译，生产环境会有性能问题
    因为不是真正输出，所以如果要使用具有修饰器的.js文件，每次都需要编译。
    
方法二：命令行操作

    babel --plugins transform-decorators-legacy 
    app.js -o app_decorator.js

    其中：
    babel：
        编译命令
        
    --plugins: 
        babel可选参数
        
    transform-decorators-legacy:
        --plugins指定的插件
        
    app.js:
        需要编译的文件
        
    -o:
        output缩写，编译后输出的命令
    
    app_decorator.js:
        编译后输出的文件名
    
    好处：
    不用显式的创建.babelrc文件
    
    缺点：
    命令行中内容太多，不易维护

方法三：创建.babelrc文件

创建.babelrc
    
    {
      "plugins": ["transform-decorators-legacy"]
    }
命令行执行：
    
    babel app.js -o app_decorator.js
    
    上面当执行babel时，会读取配置文件.babelrc，这样命令行中就不用再写插件名了。

方法四：ts(最简单)

    全局安装
    sudo npm i -g typescript
    
    tsc app.js即可
    
    上述命令，会生成新的app.js
    
    就是这么简单
    
    但是会报错：
    /*Experimental support for decorators is a feature
    that is subject to change in a future release. 
    Set the 'experimental
    Decorators' option to remove this warning.*/
    
    报错的意思是没有配置experimental 用来兼容decorator
    
    注意，此方式：命令行中输入了需要编译的文件。
    
解决报错一：命令行配置

    命令行：
    tsc app.ts --target ES5 --experimentalDecorators

    🐽：注意，此方式命令行中不可输入需要编译的文件，会直接编译当前目录。
    
解决报错二：创建tsconfig.json文件

    {
        "noEmitOnError": true,
        "compilerOptions": {
            "experimentalDecorators": true
        }
    }
    noEmitOnError：如果编译不过，不输出
    compilerOptions：编译选项
    experimentalDecorators:是否编译修饰器
    
    🐽：命令行中执行tsc即可
    🐽：不可在命令行中输入需要编译的文件，否则将忽略tsconfig文件，千万注意。

解决报错三：-p指定tsconfig.ts

    tsc -p tsconfig.json
    
    当项目中有多种配置需求时，可以通过上面这种形式，来指定引用的tsconfig文件
    
ts拓展：
    
    当命令行中不指定文件时，执行tsc：
    
        一，在当前目录下，搜索tsconfig.json文件，如果没找到，则去父文件夹中寻找，直到找不到。
        二，通过--project 或- p来指定需要引用的tsconfig.json

    当命令行中指定文件时，执行tsc file.ts：
    
        会忽略tsconfig.json文件
    
## 二，修饰器

> 1，坎坷起步

    费了这么大力气，我们终于可以使用了。
    
    但是究竟选择哪个方案来编译？
    
    目的：
        快速的学习decorator
        
    如果使用ts||babel：
        第一步，编译ts位js；
        第二步，执行node helloworld.js
    
    不难看出，这很麻烦，想要测试一个功能，需要重复两个操作，整合为一个才是我们的目标。
    
    所以我们选择babel的node api，这样就能够同步测试了
    
    先安装：
    npm i babel-plugin-transform-decorators-legacy babel-register --save-dev
    
    创建complie.js
    require('babel-register')({
        plugins: ['transform-decorators-legacy']
    });
    require("./app.ts")
    
    创建app.ts：
    @addSkill
    class Person {}
    function addSkill(target) {
      target.say = "hello world";
    }
    console.log(Person.say)
    
    最后，执行node complie.js
    🐽：hello world终于出来了。
    
    题外话：太不容易了......
    
> 2 初探（对类新增属性）
    
    @addSkill
    class Person {}
    function addSkill(target) {
      target.say = "hello world";
    }
    
    回顾一下上面的例子：
    
    @addSkill 是修饰器
    Person是类
    
    上面代码的意思：
        执行addSkill方法；
        给Person类，添加了一个静态属性say；
        say的值是”hello world“
        
    上面的行为，相当于
    
    class Person {}
    Person = addSkill(Person) || Person;
    
    可以看出
    addSkill的参数target
    就是要被修改的类
    
> 3 传递多个参数

    上面我们写的修饰器，无法配置，是固定的
    如果想要通过参数配置该如何处理？
    
    解决方案：修饰器外面再封装一层函数：
    
    例：
    @wrap(1,2)
    class Person {}
    function wrap(num1,num2){
        return function addSkill(target) {
          target.say = "hello world";
          target.num = num1+num2;
        }
    }
    
    wrap:是修饰器包裹函数
    1,2：是参数
    Person：是类
    addSkill才是真的修饰器
    
> 4 函数执行顺序

    @wrap(1,2)
    class Person {}
    function wrap(num1,num2){
        console.log("wrap");
        return function addSkill(target) {
            console.log("inner");
          target.say = "hello world";
          target.num = num1+num2;
        }
    }
    console.log(Person.num)
    
    上面的代码输出：
    wrap
    inner
    3
    
    可以得出
    
    首先：
    执行wrap方法；
    
    然后：
    执行addSkill方法；
    
    🐽：上面只有addSkill是对类进行的操作。

> 5 添加实例属性

    function wrap(target){
        target.prototype.say = "hello world";
    }
    @wrap
    class Person {}
    var man = new Person();
    
    上面的代码执行console.log(man.say)
    输出：hello world
    
> 6 模块化

    前面有了修饰器，肯定是用来处理特定问题的，
    所以封装出去
    
创建 decorator.ts文件
    
    export default function(target){
        target.prototype.say = "hello world";
    }

修改 app.ts文件

    import decorator from "./decorator.ts";
    @decorator
    class Person {}
    var man = new Person();
    console.log(man.say);
    
执行 node complie.js
    
    报错：SyntaxError: Unexpected token import
    
    是因为import无法使用，这是es6的语法，还需要兼容

解决问题：

    安装：es6的babel
    
    npm install --save-dev babel-preset-es2015
    
    修改：complie.js
    
    require('babel-register')({
        "presets":["es2015"],
        "plugins": ['transform-decorators-legacy']
    });
    require("./app.ts")
    
    可以看到，编译文件中，添加了一个预设，就是es2015
    es2015，就是用来编译es6的
    
    🐽：以下是版本对照
    
    es1999 = es3
    es2009 = es5
    es2015 = es6
    es2016 = es7
    es2017 = es8
    
> 7 模块化加深
    
    如果我们想要创建一个alien类
    通过修饰器给alien类添加一个说人话的方法,该怎么实现？
    
首先：修改 app.ts

    import decorator from "./decorator.ts";
    let Person = {
        say(){
            console.log("this is Person")
        }
    }
    @decorator(Person)
    class Alien {}
    var monster = new Alien();
    monster.say();
    
    我们想要做到的事情：
    Person是个对象，具备say方法；
    decorator修饰器传入（Person）对象
    对Alien对象进行了操作
    Alien的实例monster就也具备了say方法
    
然后：修改decorator.ts

    export default function(...list){
        return function(target){
            Object.assign(target.prototype,...list)
        }
    }
    
    ok,这样就实现了
    
    过程是：
    首先：
        decorator修饰器接受参数（Person）
    然后：
        修饰器对target，也就是Alien类进行操作,
        把Person类merge进了Alien.prototype.
    
拓展：rest参数

    上面decorator.ts中的...list
    是es6的语法
    
    ...rest的意思是：
    剩余所有的参数的集合
    
    🐷注意：
    
    ...rest和rest并不相同
    
    例：
    function add(a,...rest){
        console.log(...rest)
        console.log(rest)
    }
    add(1,2,3,4)
    
    上面的代码中：
    rest是数组，
    ...rest是剩余的参数2 3 4；
    
    注意...rest有风险：
    如果只有一个元素，那可以直接使用，类似obj；
    如果有多个元素，不能直接使用,类似1 2；
    
> 8 在react中使用

传统情况下：

    class MyReactComponent extends React.Component {}

    export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);
    
    以往，使用react和redux时
    
    需要创建一个组件类，继承自react
    并通过redux提供的connect，返回一个高阶组件；
    
使用修饰器的情况

    @connect(mapStateToProps, mapDispatchToProps)
    export default class MyReactComponent extends React.Component {}
    
    好吧，好似确实容易理解一点了
    
    这样去看的话：
    可以发现，connect方法使用来处理react组件类的
    同是，mapStateToProps和mapDispatchToProps都是传给修饰器外层函数的方法
    
    最后，拿到的是对react组件修饰过后的新组件。

## 三，对属性的修饰
    
前面我们已经初步掌握了对类的修饰，继续深入。

    假设：
    我们有一个alien类，
    它已经有了一个say方法，是说一句人话
    
    现在，我们希望alien这个类，永远只能说一句话，彻底限制住它的学习能力。
    避免对人类造成危险。
    
修改app.ts
    
    import decorator from "./decorator.ts";
    class Alien {
        static say(){
            console.log(1)
        }
    }
    
    Alien.say();
    
    1，定义了一个Alien类
    2，定义了一个static静态方法say
    3，执行Alien.say();
    4，控制台输出1；
    
然后修改decorator.ts

    export default function(target,name,descriptor){
        descriptor.writable = false;
        return descriptor
    }
    
    当对属性修饰时
    
    接受3个参数:
    1，target是要被修饰的对象；
    2，name是属性名；
    3，descriptor是对属性的描述；
    
    这里很类似，es5的
    Object.defineProperty(Person.prototype, 'name', descriptor);
    
再回来修改app.ts
    
    import decorator from "./decorator.ts";
    class Alien {
        @decorator
        static say(){
            console.log(1)
        }
    }
    
    Alien.say();
    
    1，添加了一句@decorator
    2，目的是对属性进行修饰
    3，执行Alien.say()
    4，控制台还是输出1；
    
    我们的目的是让它只能说1。

所以再次修改app.ts

    import decorator from "./decorator.ts";
    class Alien {
        @decorator
        static say(){
            console.log(1)
        }
    }
    Alien.say();
    Alien.say = function(){
        console.log(2);
    }
    Alien.say();
    
    1，修改了say方法
    2，并执行新的say方法
    3，控制台报错！！！！！！！！！
    
    //TypeError: Cannot assign to read only property 'say' of function 'function Alien()
    
    提示我们：say属性read only不可修改
    
    🐷注意：
    read only不是不允许赋值，
    而是赋值一次之后，不允许修改！切记！
    
验证一下：再次修改decorator
    
    export default function(target,name,descriptor){
        descriptor.writable = true;
        return descriptor
    }
    
    1，修改属性的可写属性为true
    2，再次执行app.ts
    3，命令行输出1，2
    
    说明我们的修饰正确!
    

拓展:static和state

    1，static是类的静态方法，不是实例的方法
    
    例：
    class Foo {
      static classMethod() {
        return 'hello';
      }
    }
    
    Foo.classMethod() // 'hello'
    
    var foo = new Foo();
    foo.classMethod()
    // TypeError: foo.classMethod is not a function
    
    2，static能够被子类继承
    
    class Foo {
      static classMethod() {
        return 'hello';
      }
    }
    
    class Bar extends Foo {
    }
    
    Bar.classMethod() // 'hello'
    
    3，state是用来定义类的静态属性的
    
    例：
    class ReactCounter extends React.Component {
      state = {
        count: 0
      };
    }
    
## 四，实现一个log装饰器

修改app.ts

    例：
    import decorator from "./decorator.ts";
    class Math {
      @decorator
      add(a, b) {
        return a + b;
      }
    }
    const math = new Math();
    math.add(2, 4);
    
    1，定义了一个Math类
    2，通过decorator修饰了add属性
    3，目的在调用add方法的时候，记录log
    
修改decorator.ts

    export default function(target,name,descriptor){
        var oldValue = descriptor.value;
        descriptor.value = function() {
          console.log(`Calling "${name}" with`, arguments);
          return oldValue.apply(null, arguments);
        };
        return descriptor;
    }
    
    1,暴露decorator方法
    2,descriptor.value其实就是add(a,b){return a+b}这个函数
    3，存储add函数为oldValue
    4，修改add属性对应的方法，在其中添加console.
    
## 五，修饰器的执行顺序

    例：
    function dec(id){
        console.log('evaluated', id);
        return (target, property, descriptor) => console.log('executed', id);
    }
    
    class Example {
        @dec(1)
        @dec(2)
        method(){}
    }
    
    上面的例子，输出
    1
    2
    2
    1
    
    原理类似：
    function test(num){
        console.log(num);
        return function(){
            console.log(num);
        }
    }
    undefined
    var fn1 = test(1);
    var fn2 = test(2);
    fn1(fn2());
    
    
    
    
    

    
    

    
    
    
    

    
    
    
    



    
    
    

    

    
    
    
    
    
    
    
    
    



    
    
    
        
    
    
    
    
    

    
    



    
    
    
    
    

    
    
    
    
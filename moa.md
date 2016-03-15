# 从0开始写Node.js框架


## 生成项目

```
npm i -g express-generator
express mvc
cd mvc
nrm use taobao
npm i
```

至此准备完事儿

## 使用supervisor

livereolad

- nodemon
- supervisor

```
npm i -S supervisor
```

修改package.json

```
  "scripts": {
    "start": "./node_modules/.bin/supervisor bin/www"
  },
```

然后

```
npm start
```

改一下routes里的代码试试，你再也不要重启了


## 自动挂载路由


```
npm i -S mount-routes
```

app.js里修改如下

修改1

```
var routes = require('./routes/index');
var users = require('./routes/users');
```

改为
```
var mount = require('mount-routes');
```

修改2

```
app.use('/', routes);
app.use('/users', users);
```

改为

```
mount_routes(app,  __dirname + '/routes', true);
```

然后

```
npm start
```

在路由里复制users.js为test.js

测试

http://127.0.0.1:3000/test

妈妈再也不用担心我写路由了

这样做是有好处也有坏处的。但均衡来讲，好处大于坏处，感谢express的路由机制，让我们可以足够定制的里面的逻辑

如果你觉得自动路由不够用，你自己手动挂载也是可以的，只要你不嫌麻烦


## 添加测试

```

npm install --save-dev mocha
npm install --save-dev chai
npm install --save-dev sinon
npm install --save-dev supertest
npm install --save-dev zombie
```

```
mkdir test
touch test/test.js
```

将下面代码copy进去

```
var request = require('supertest');
var assert  = require('chai').assert;
var expect  = require('chai').expect;
require('chai').should();

var app = require('../app');

describe('GET /users', function(){
  it('respond with text', function(done){
    request(app)
      .get('/users/')
      .set('Accept', 'application/text')
      .expect('Content-Type', /text/)
      .expect(200, done);
  })
})
```


执行测试

```
➜  mvc git:(master) mocha -u bdd

******************************************************
		MoaJS Apis Dump
******************************************************

┌─────────────────────────────────────────────────────┬────────┬────────┐
│ File                                                │ Method │ Path   │
├─────────────────────────────────────────────────────┼────────┼────────┤
│ /Users/sang/workspace/stuq/aaaa/mvc/routes/index.js │ get    │ /      │
├─────────────────────────────────────────────────────┼────────┼────────┤
│ /Users/sang/workspace/stuq/aaaa/mvc/routes/test.js  │ get    │ /test  │
├─────────────────────────────────────────────────────┼────────┼────────┤
│ /Users/sang/workspace/stuq/aaaa/mvc/routes/users.js │ get    │ /users │
└─────────────────────────────────────────────────────┴────────┴────────┘


  GET /users
GET /users/ 200 10.206 ms - 23
    ✓ respond with text (50ms)


  1 passing (55ms)
```

就这样完成了简单的测试


测试生命周期是非常重要的，给出模板test2.js

```
var request = require('supertest');
var assert  = require('chai').assert;
var expect  = require('chai').expect;
require('chai').should();

// 测试代码基本结构
describe('UserModel', function(){
	before(function() {
    // runs before all tests in this block
  })
  after(function(){
    // runs after all tests in this block
  })
  beforeEach(function(){
    // runs before each test in this block
  })
  afterEach(function(){
    // runs after each test in this block
  })
	
  describe('#save()', function(){
    it('should return sang_test2 when user save', function(done){
      done()
    })
  })
})
```

至此已有功能

- livereload
- mount-routes
- test



## 添加mongoose

```
mkdir models
npm i -S mongoose
npm i -S mongoosedao
```

在路由里增加创建代码

### 配置

配置mongodb链接信息

- config/mongodb.example.js
- db.js

```
cp config/mongodb.example.js config/mongodb.js
```

### 创建models/user.js

```
var mongoose    = require('mongoose');
var Schema      = mongoose.Schema;
var MongooseDao = require('mongoosedao');

var userSchema = new Schema(
    {
      "name":"String",
      "password":"String"
    }
);

var User = mongoose.model('User', userSchema);
var UserDao = new MongooseDao(User);
 
module.exports = UserDao;
```

### 测试user.js

```
var request = require('supertest');
var assert  = require('chai').assert;
var expect  = require('chai').expect;
require('chai').should();

require('../db')

var User = require('../models/user')

// 测试代码基本结构
describe('UserModel', function(){
	before(function() {
    // runs before all tests in this block
  })
  after(function(){
    // runs after all tests in this block
  })
  beforeEach(function(){
    // runs before each test in this block
  })
  afterEach(function(){
    // runs after each test in this block
  })
	
  describe('#save()', function(){
    it('should return stuq when user save', function(done){
      User.create({"name":"stuq","password":"password"},function(err, user){
        if(err){
            expect(err).to.be.not.null;
            done();
        }

        expect(user.name).to.be.a('string');
        expect(user.name).to.equal('stuq');
        done();
      });
    })
  })
})
```

在测试完成后需要在after里删除测试数据，保证测试完整性，自己写吧

### 在路由里增加user创建和api测试

routes/user.js

```
var User = require('../models/user')

router.post('/register', function(req, res, next) {
  var name = req.body.name;
  var password = req.body.password;
  User.create({
    "name":name,
    "password":password
  },function(err, user){
    if(err){
      res.json('register failed with err');
    }
      
    res.json('register sucess');
  });
});
```

test/user_api.js

```
var request = require('supertest');
var assert  = require('chai').assert;
var expect  = require('chai').expect;
require('chai').should();

var app = require('../app');

require('../db')

describe('POST /users/register', function(){
  it('respond register with json', function(done){
    request(app)
      .post('/users/register')
      .field('name', 'stuq')
      .field('password', '123456')
      .set('Accept', 'application/json')
      .expect('Content-Type', /json/)
      .expect(200, done);
  })
})
```

测试

```
mocha -u bdd test/user_api.js
```

## 添加mvc目录

很多人是这样的做法：

```
$ tree . -L 1 -d
.
├── actions
├── config
├── cron_later
├── doc
├── middleware
├── migrate
├── models
├── node_modules
├── public
├── queues
├── routes
├── test
├── tmp
├── uploads
└── views
```
说明

- 继承express-generator的结构（routes和views）
- actions即controller控制器层
- models即模型层
- config是配置项目录
- middleware中间件层
- migrate是我们处理数据的，跟rails的migrate不完全一样
- test是测试目录
- queues是队列
- cron_later是调度
- doc文档
- uploads是上传自动创建的目录

这阶段仅仅是用，算是基于express抽象了一点业务、配置相关的东西而已，目录多了依然蛋疼

所以还是rails的更清晰一些

```
rails new blog --skip-bundle


git:(master) ✗ tree blog -L 2 -d
blog
├── app
│   ├── assets
│   ├── controllers
│   ├── helpers
│   ├── mailers
│   ├── models
│   └── views
├── bin
├── config
│   ├── environments
│   ├── initializers
│   └── locales
├── db
├── lib
│   ├── assets
│   └── tasks
├── log
├── public
├── test
│   ├── controllers
│   ├── fixtures
│   ├── helpers
│   ├── integration
│   ├── mailers
│   └── models
├── tmp
│   └── cache
└── vendor
    └── assets

29 directories
```

so，我们也来建一个吧


这些就是基本的mvc目录，但是express有其自己的特点，比如路由和中间件等，想想还是放app项目更好一些

```
mkdir -p app/controllers
mkdir -p app/models
mkdir -p app/middlewares
```

而views和routes是已有目录，只需要移动到app下即可

```
mv views app/views
mv routes app/routes
```

我们记得，mount-routes里修改了app里的路由路径，改一下就好

```
mount_routes(app,  __dirname + '/routes', true);
```

改为

```
mount_routes(app,  __dirname + '/app/routes', true);
```

```
app.set('views', path.join(__dirname, 'views'));
```

改为

```
app.set('views', path.join(__dirname, 'app/views'));
```

至此我们的基本目录是有了的下面

下面我们要考虑职责的事儿了

SOLID（藏头命名）

- 单一职责原则 (Single Responsibility Principle，SRP)
- 开放/封闭原则(Open/Close Principle，OCP)
- 里氏替换原则(Liskov Substitution Principle，LSP)
- 接口分离原则(Interface Segregation Principle，ISP)
- 依赖倒置原则(Dependency Inversion Principle，DIP)

此时的路由里完成各种逻辑，实际上是不太合理，没有遵循mvc原则

理想的做法是我们创建一个model叫Order，它有

- routes/order.js
- controllers/OrderController.js
- models/Order.js
- views/order/*.jade

这是正确的做法

最佳目录结构应该如下

```
 git:(master) ✗ tree app -L 3
app
├── controllers
│   └── orders_controller.js
├── models
│   └── order.js
├── routes
│   ├── api
│   │   └── orders.js
│   └── orders.js
└── views
    └── orders
        ├── edit.jade
        ├── index.jade
        ├── new.jade
        ├── order.jade
        └── show.jade

6 directories, 9 files
```

调用级别是

- 我们在路由里不写任何逻辑代表，只是让他转到controller
- 然后controller调用mode里方法完成业务逻辑，并根据返回时，处理视图渲染
- 视图只根据controller给的内容进行渲染即可

## 加载某个目录到一个对象上

如果我想加载某个目录到一个对象上，需要如何做呢？

### 准备

```
mkdir require-dictory
cd require-dictory
ls
npm init
npm i -S require-directory
touch index.js
```

### 核心代码

```
var requireDirectory = require('require-directory');
module.exports = requireDirectory(module);
```

### 测试

touch test.js

```
var obj = require('./index')

console.log(obj);
```

然后

```
➜  require-dictory git:(master) ✗ node test.js 
{ node_modules: { 'require-directory': { index: [Object], package: [Object] } },
  package: 
   { name: 'require-dictory',
     version: '1.0.0',
     description: '',
     main: 'index.js',
     scripts: { test: 'echo "Error: no test specified" && exit 1' },
     author: '',
     license: 'ISC',
     dependencies: { 'require-directory': '^2.1.1' } },
  test: {} }
```

此时index对象就返回了index和test和package.json里的内容

测试,创建third.js

```
➜  require-dictory git:(master) ✗ touch third.js
➜  require-dictory git:(master) ✗ node test.js
{ node_modules: { 'require-directory': { index: [Object], package: [Object] } },
  package: 
   { name: 'require-dictory',
     version: '1.0.0',
     description: '',
     main: 'index.js',
     scripts: { test: 'echo "Error: no test specified" && exit 1' },
     author: '',
     license: 'ISC',
     dependencies: { 'require-directory': '^2.1.1' } },
  test: {},
  third: {} }
```

此时，这个模块就写完啦，打印出当前模块下的挂载的所有对象

### 如何发布npm模块

那么我们来发布一个吧，顺便学习一下如何发布npm模块

- 登陆npm
- 发布

```
➜  require-dictory git:(master) ✗ nrm use npm

   Registry has been set to: https://registry.npmjs.org/

➜  require-dictory git:(master) ✗ npm login
Username: i5ting
Password: 
Email: (this IS public) shiren1118@126.com
```

一定要注意`nrm use npm`，源一定要是npmjs，不然是无法登陆成功的。

然后在package.json所在目录里

```
npm publish .
```

即可

可以在package.json里增加start，每次发布模块的时候能简单点

## 继续优化mvc结构

好吧，上一小节，给出了如何简单的把目录下的文件挂载到某个对象上，并发布npm上

照着这个思路，[我们](https://github.com/moajs)造了几个简单的库，用于挂载某个目录里的文件

- mount-controllers
- mount-models
- mount-middlewares


### 1)mount-controllers

```
var $ = require('mount-controllers')(__dirname).orders_controller;

router.get('/new', $.new); 
```


### 2)mount-models

```
var $models = require('mount-models')(__dirname);

var Order = $models.order;

exports.list = function (req, res, next) {
  console.log(req.method + ' /orders => list, query: ' + JSON.stringify(req.query));
  
  Order.getAll(function(err, orders){
    console.log(orders);
    res.render('orders/index', {
      orders : orders
    })
  });
};
```

### 3)mount-middlewares

一般会在路由或路由下面的api里使用

```
var $ = require('mount-controllers')(__dirname).orders_controller;

var $middlewares  = require('mount-middlewares')(__dirname);

// route define
router.get('/', $middlewares.check_api_token, $.api.list);
```

做这些的目的是为了让大家更好的去分离代码，其实还有一个目的，结构化的代码才能用于写生成器的。下面会讲

### 集成

简单的集成一下

```
npm i -S mount-controllers
npm i -S mount-models
npm i -S mount-middlewares
```


```
moag order name:string password:string
```

然后npm start


发现缺少中间件

- jsonwebtoken
- msgpack5

```
npm i -S jsonwebtoken
npm i -S msgpack5
```

此时访问http://127.0.0.1:3000/orders/

正常是应该有数据的

可是没有，这是为什么呢？找代码

把views下面的error和layout放进去就好了

发现没有连接数据，于是在app.js里

```
require('./db')
```

经测试delete不好使，原因是少东西

## 庖丁解views

我们使用

```
rails g scaffold user name:string password:string

tree blog/app/views 
blog/app/views
├── layouts
│   └── application.html.erb
└── users
    ├── _form.html.erb
    ├── edit.html.erb
    ├── index.html.erb
    ├── index.json.jbuilder
    ├── new.html.erb
    ├── show.html.erb
    └── show.json.jbuilder

2 directories, 8 files
```

去掉jbuilder，api文件，和视图无关

```
tree blog/app/views 
blog/app/views
├── layouts
│   └── application.html.erb
└── users
    ├── _form.html.erb
    ├── edit.html.erb
    ├── index.html.erb
    ├── new.html.erb
    ├── show.html.erb

2 directories, 8 files
```

总结一下里面的特点

- index是显示列表
- new和edit都复用_form
- show显示出所有即可

我们想一下jade是否也可以呢？

jade特性

- block和extend是布局
- include是包含，可以实现引用_form类似的
- foreach类的也支持

所以jade实现erb的功能毫无问题

### 实现列表

index.html.erb

```
<h1>Listing users</h1>

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Password</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= user.name %></td>
        <td><%= user.password %></td>
        <td><%= link_to 'Show', user %></td>
        <td><%= link_to 'Edit', edit_user_path(user) %></td>
        <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New User', new_user_path %>

```

转成jade就无比简单了

```
extends ../layouts/layout

block content
  h1 Listing orders

  table
    thead
      tr
        each n in ['name','password']
          th #{n}
        th(colspan="3")
    tbody
      each order in orders
        tr
          each n in ['order.name','order.password']
            td #{ eval(n) }
          td 
            a(href='/orders/#{ order._id}') Show
          td 
            a(href='/orders/#{ order._id}/edit') Edit
          td 
            a(onclick="click_del('/orders/#{ order._id}/')") Delete

  br

  p 
    a(href='/orders/new') New Order

```

- erb里使用 @users
- 而jade里使用['name','password']

效果是一样的。

### 实现创建和编辑

那么创建呢？

new.jade

```
extends ../layouts/layout

block content
  h1 New order
  
  include order

  a(href='/orders') Back

```

其核心在`include order`的order.jade里

这里的order用_form也不是不可以，不过没啥大劲

jade里还有一个特性，include文件，可以把上下文里的文件同名对象也传进去。也就是我include 的order，如果上下文里有的order，的order.jade里。

即减少代码转换，又精简代码，何乐而不为呢？

创建和更新都是类似，我们先看一下更新的edit.jade

```
extends ../layouts/layout

block content
  h1 Editing order

  include order

  a(href='/orders/#{ order._id}') Show
  span |
  a(href='/orders') Back

```

和创建基本无大差别，文字和url稍有变化而已。

看一下order.jade

```
- var _action = order._action == 'edit' ? '#' : '/orders/'
- var _method = order._action == 'edit' ? ""  : "post"
- var _type   = order._action == 'edit' ? "button"  : "submit"
- var onClick  = order._action == 'edit' ?  "click_edit('order-" + order._action + "-form','/orders/" + order._id + "/')" : ""
form(id='order-#{ order._action}-form',action="#{_action}", method="#{_method}",role='form')
  each n in ['order.name','order.password']
    - m = eval(n);
    div(class="field")
      label #{n.split('.')[1]} #{m}
      br
      input(type='text',name="#{n.split('.')[1]}" ,value="#{ m == undefined ? '' : m }")
        
  div(class="actions") 
    input(type='#{_type}',value='Submit',onClick='#{onClick}')
```

这个代码有点丑陋，为了兼容创建和更新，所以定义了几个变量的。

说明

- form表单处理请求，form的action约定了
- 提交分2种
  - 创建，就直接post提交
  - 更新交给了onClick变量对应的click_edit方法（位于public/javascripts/app.js）

看一下这个前端js吧

```
function click_edit(id, url){
  // $('#' + id).attr('action','#');
  console.log(url);
  
  if (!confirm("确认要更新？")) {
    return window.event.returnValue = false;
  }
  
  var form = document.querySelector('form')
  var data = form2obj(form);
  console.log(data);
  
  // return false;
  $.ajax({
    type: "PATCH",
    url: url,
    data : data
  })
  .done(function( res ) {
    if(res.status.code == 0){
      // alert( "Data delete: success " + res.status.msg );
      window.location.href= res.data.redirect;
    }else{
      alert( "Data delete fail: " + res.status.msg );
    }
  });
  
  return false;
}

```

简单的更新，使用PATCH方法而已。这里有一个知识点

```
var data = form2obj(form);
```

把表单里的内容转换成ajax里的传值对象，算是个tips吧（具体见public/javascrips/form2obj.js）。


### 实现删除

首先index.jade里有删除功能

```
  td 
    a(onclick="click_del('/orders/#{ order._id}/')") Delete
```

根据rest路由，我们一般的做法

```
 DELETE /users/:id(.:format)      users#destroy
```

也就是我delete请求到/users/:id即可

即，对应的是click_del方法（位于public/javascripts/app.js）

```

function click_del(url){
  if (!confirm("确认要删除？")) {
    return window.event.returnValue = false;
  }
  
  console.log(url);
  
  $.ajax({
    type: "DELETE",
    url: url
  })
  .done(function( res ) {
    if(res.status.code == 0){
      // alert( "Data delete: success " + res.status.msg );
      window.location.href= location.href;
    }else{
      alert( "Data delete fail: " + res.status.msg );
    } 
  });
}
```

### jade版的crud

详情如下

```
➜  mvc git:(master) ✗ tree app/views/orders 
app/views/orders
├── edit.jade
├── index.jade
├── new.jade
├── order.jade
└── show.jade

0 directories, 5 files
```

### jade版的依赖js

- app.js
  - click_del删除方法
  - click_edit编辑方法
- form2obj.js

```
➜  mvc git:(master) ✗ tree public           
public
├── 404.html
├── 422.html
├── 500.html
├── favicon.ico
├── images
├── javascripts
│   ├── app.js
│   └── form2obj.js
├── robots.txt
├── stylesheets
│   └── style.css
└── uploads

4 directories, 8 files
```

### layout

最后再简单的介绍一下layout.jade

上面已知依赖

- app.js
  - click_del删除方法
  - click_edit编辑方法
- form2obj.js

如果我让生成的代码看不到app和form2obj的存在，生成的代码就可以复用了，是不是？

其实丢在布局文件里就可以

```
doctype html
html
  head
    title= title
    link(rel='stylesheet', href='/stylesheets/style.css')
    link(href="http://libs.baidu.com/bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet")
  body
    block content
script(src="http://libs.baidu.com/jquery/1.9.0/jquery.js")
script(src="http://libs.baidu.com/bootstrap/3.0.3/js/bootstrap.min.js")
script(src='/javascripts/form2obj.js')
script(src='/javascripts/app.js')
```

说明

- 引了jq和bootstrap
- 引了app.js
- 引了form2obj.js

简单吧？

## 没rest怎么行？

曾经rest被吹的玄之又玄，唯恐不谈rest就没有x格，其实rest理解起来还是蛮简单的

最简单的入门是理解crud方法，参考rails的scaffold即可

```
rails g scaffold user name:string password:string
```

打印路由

```
blog git:(master) ✗ rake routes
   Prefix Verb   URI Pattern               Controller#Action
    users GET    /users(.:format)          users#index
          POST   /users(.:format)          users#create
 new_user GET    /users/new(.:format)      users#new
edit_user GET    /users/:id/edit(.:format) users#edit
     user GET    /users/:id(.:format)      users#show
          PATCH  /users/:id(.:format)      users#update
          PUT    /users/:id(.:format)      users#update
          DELETE /users/:id(.:format)      users#destroy
```

是不是代码非常清晰？

既然它是最佳实践，我们也没必要再自己搞一套,只需要因express制宜即可

*  GET    /orders[/]        => order.list()
*  GET    /orders/new       => order.new()
*  GET    /orders/:id       => order.show()
*  GET    /orders/:id/edit  => order.edit()
*  POST   /orders[/]        => order.create()
*  PATCH  /orders/:id       => order.update()
*  DELETE /orders/:id       => order.destroy()


在app/routes/order.js里

```
var express = require('express');
var router = express.Router();

// mount all middlewares in app/middlewares, examples:
// 
// router.route('/')
//  .get($middlewares.check_session_is_expired, $.list)
//  .post($.create);
// 
var $middlewares  = require('mount-middlewares')(__dirname);

// core controller
var $ = require('mount-controllers')(__dirname).orders_controller;


/**
 * Auto generate RESTful url routes.
 *
 * URL routes:
 *
 *  GET    /orders[/]        => order.list()
 *  GET    /orders/new       => order.new()
 *  GET    /orders/:id       => order.show()
 *  GET    /orders/:id/edit  => order.edit()
 *  POST   /orders[/]        => order.create()
 *  PATCH  /orders/:id       => order.update()
 *  DELETE /orders/:id       => order.destroy()
 *
 */

router.get('/new', $.new);  
router.get('/:id/edit', $.edit);

router.route('/')
  .get($.list)
  .post($.create);

router.route('/:id')
  .patch($.update)
  .get($.show)
  .delete($.destroy);

module.exports = router;
```

于是所有东西就都丢到controller里去。

整个世界顿时就清净了

有了Rest，格调也有了。。。。


## 脚手架scaffold

rails在2005年横空出世，靠的就是10分钟完成一个blog的当时的创举，其实scaffold居功甚伟

scaffold说白了就是生成器

### 模板引擎的原理
我们来回想模板引擎的原理

- 数据
- 模板
- 然后模板+数据编译，生成html页面

那么，如果我要生成文件呢？比如controller.js文件呢？

思路其实也是一样的

举个例子

http://handlebarsjs.com/

步骤1：定义模板

```
<div class="entry">
  <h1>{{title}}</h1>
  <div class="body">
    {{body}}
  </div>
</div>
```

步骤2：编译模板

```
var source   = $("#entry-template").html();
var template = Handlebars.compile(source);

var context = {title: "My New Post", body: "This is my first post!"};
var html    = template(context);
```

步骤3：然后写入到文件

```
fs.writeFile(dest_file_path, content , function (err) {
  if (err) throw err;
  log('It\'s saved!');
});
```

其实也模板引擎就差最后一步，写入文件。

### tpl_apply

但写每次都这样文件读写，还是挺累的，索性就写一个npm吧

```
var tpl = require('tpl_apply');


var source = process.cwd() + '/tpl.js'
var dest = process.cwd() + '/test/tpl.generate.js'


tpl.tpl_apply(source, {
    title: "My New Post", body: "This is my first post!"
}, dest);
```

用法很简单，3个参数

- source是模板
- 中间的对象是数据，要和模板编译的数据
- dest是生成的文件名称

是不是很清晰呢？

### 实现一个scaffold

解决了基本的文件生成问题，下面我们就要分析一下scaffold的实现可能

参考rails的scaffold即可

```
rails g scaffold user name:string password:string
```

说明

- user是模型名称
- name和password是字段
- string是自定对应的类型

是不是很简单？

ok，我们来看一下model

```
var mongoose    = require('mongoose');
var Schema      = mongoose.Schema;
var MongooseDao = require('mongoosedao');

var orderSchema = new Schema(
    {"name":"String","password":"String"}
);

var Order = mongoose.model('Order', orderSchema);
var OrderDao = new MongooseDao(Order);
 
module.exports = OrderDao;
```

这里面可变的就模型名称的字段和类型，是吧？

于是模型模板定义如下

```
/**
 * Created by alfred on {{created_at}}.
 */

var mongoose    = require('mongoose');
var Schema      = mongoose.Schema;
var MongooseDao = require('mongoosedao');

var {{model}}Schema = new Schema(
    {{{mongoose_attrs}}}
);

var {{entity}} = mongoose.model('{{entity}}', {{model}}Schema);
var {{entity}}Dao = new MongooseDao({{entity}});
 
module.exports = {{entity}}Dao;
```

剩下的就是从cli里获得数据，然后编译就好了。

cli里获取参数就是字符串分割，没啥难度

其他依此类推

- controllers
- routes
- views

https://github.com/moajs/moa/tree/dev/tpl


我们来回顾一下，为什么能如此简单的完整生成器呢？

- 我们的代码结构和目录结构已经固定死了
- 模板引擎：数据+模板->生成模板

简单吧？

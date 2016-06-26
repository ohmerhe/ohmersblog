title: LoopBack操作记录
date: 2016-06-26 19:02:04
tags: 
- mysql
- loopback
  
thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-153854891.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-153854891.jpg?imageView2/1/w/1024/h/460 

---


tags: loopback, mysql

## 什么是LoopBack

请看[官方文档](https://loopback.io/)

<!-- more -->


## Get Started

继续查看[Getting started](http://loopback.io/getting-started/)

## 自定义MySql连接

### 自定义table名字

在模型的json文件中`common/model/category.json`添加Mysql的配置

```
{
  "name": "Category",
  ...
  "mysql": {
    "table": "custom_category" // 自定义的表名
  }
  ...
}

```

更多Model的Json定义参考[Model definition JSON file](https://docs.strongloop.com/display/public/LB/Model+definition+JSON+file)

### 自定义table列名

同样在模型定义的json文件中，`common/model/category.json`找到`properties`模块的配置，为需要自定义表列名的属性下添加`msql`配置信息

```
{
  "name": "category",
  ...
  "properties": {
    ...
    "title": {
      "type": "String",
      "required": true,
      "mysql":{
        "columnName":"question_content",
        "dataType":"VARCHAR"
      }
    },
    ...
  },
  ...
}
```

还有其他类型类型的对应关系，可以看[MySQL connector](https://docs.strongloop.com/display/public/LB/MySQL+connector)

## 自定义请求方法(Remote Method)

loopback默认提供了许多方法，可以方便的访问到服务端的资源，这里列出一些常用的，更多的方法可以参考[文档](http://apidocs.strongloop.com/loopback/#persistedmodel)。

method			| path				| verb
-------------	| -------------	| -------------
find 			| /					| GET
findById 		| /:id				| GET
findOne		| /findOne		| GET
deleteById	| /:id				| DELETE
count			| /count			| GET
exists			| /:id/exists		| GET
exists			| /:id				| HEAD
create			| /					| POST
upsert			| /					| PUT
exists			| /exists			| GET
exists			| /exists			| GET

这些方法的定义和注册都可以在`/node_modules/loopback/lib/persisted-model.js`文件中看到。如`find`
 
```

PersistedModel.find = function find(filter, cb) {
  throwNotAttached(this.modelName, 'find');
};

 setRemoting(PersistedModel, 'find', {
  description: 'Find all instances of the model matched by filter from the data source.',
  accessType: 'READ',
  accepts: {arg: 'filter', type: 'object', description: 'Filter defining fields, where, include, order, offset, and limit'},
  returns: {arg: 'data', type: [typeName], root: true},
  http: {verb: 'get', path: '/'}
});
 
```

参照上面的方法我们可以自定义一些需要的方法

```
module.exports = function(Person){
     
    Person.greet = function(msg, cb) {
      cb(null, 'Greetings... ' + msg);
    }
     
    Person.remoteMethod(
        'greet', 
        {
          accepts: {arg: 'msg', type: 'string'},
          http: {verb: 'get', path: '/greet'},
          returns: {arg: 'greeting', type: 'string'}
        }
    );
};
```

我们可以通过`GET /api/persons/greet?msg=John`获得如下的返回

```
Greetings... John
```

关于自定义方法的参数说明可以参考[详细描述](https://docs.strongloop.com/display/public/LB/Remote+methods)


## 自定义返回数据

loopback提供了一系列方法可以改变资源返回，比如我需要将请求到的资源列表包装在一个对象的`list`下返回。


```
  Category.afterRemote('find', function(ctx, remoteMethodOutput, next) {
    if (ctx.result && Array.isArray(remoteMethodOutput)) {
      ctx.result = {
        "list": remoteMethodOutput
      }
    }
    next();
  });
};
```

```
自定义返回结果前
[
	{...},
	{...},
	{...},
	{...}
]

自定义返回结果后
{
	"list" : [
		{...},
		{...},
		{...},
		{...}
	]
}
```

更多关于请求资源的逻辑控制，请参考[Adding logic to models](https://docs.strongloop.com/display/public/LB/Adding+logic+to+models)

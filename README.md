# ali-mns (ali-mqs)
[![npm version](https://badge.fury.io/js/ali-mns.svg)](http://badge.fury.io/js/ali-mns)
[![npm version](https://badge.fury.io/js/ali-mqs.svg)](http://badge.fury.io/js/ali-mqs)

The nodejs sdk for aliyun mns service

[阿里云消息服务-简体中文-帮助手册](http://armclr.azurewebsites.net/Links/AliMNS?lang=zh-Hans)

Ali MNS service is a MQ(message queue) service provided by AliYun.
The world largest online sales website www.taobao.com is heavily relying on it.

You can visit [http://www.aliyun.com/product/mns](http://www.aliyun.com/product/mns) for more details.

The original Ali-MQS service has been upgraded and changed it's name to Ali-MNS since June, 2015.
Go to  [Migrate](#migrate) part for the old version informations.

# QuickStart
Use 'npm install ali-mns' to install the package.

```javascript
    var AliMNS = require("ali-mns");
    var account = new AliMNS.Account("<your-account-id>", "<your-key-id>", "<your-key-secret>");
    var mq = new AliMNS.MQ("<your-mq-name>", account, "hangzhou");
    // send message
    mq.sendP("Hello ali-mns").then(console.log, console.error);
```
More sample codes can be found in [GitHub](https://github.com/InCar/ali-mns/tree/master/test).

# Promised
The ali-mns use the [promise](https://www.npmjs.org/package/promise) pattern.
Any functions suffix with 'P' indicate a promise object will be returned from it.

# Typescript
If you only want to use it, forget this.

Most source files are written in typescript instead of javascript.
Visit [http://www.typescriptlang.org/](http://www.typescriptlang.org/) for more information about typescript.

If you interest in source file, visit GitHub [https://github.com/InCar/ali-mns](https://github.com/InCar/ali-mns)

Please use 'gulp' to compile ts files into a single index.js file after downloading source files. 

# ali-mns(源)文档

* npm

https://www.npmjs.com/package/ali-mns

* github

https://github.com/InCar/ali-mns

# 改造说明

* 源库中mq.notifyRecv和batch.notifyRecv方法无法实现异步处理

源库中notifyRecv的原型如下:

```javascript
mq.notifyRecv(function(err, message){
    console.log(message);
    if(err && err.message === "NetworkBroken"){
        // Best to restart the process when this occurs 
        throw err;
    }
    return true; // this will cause message to be deleted automatically 
});
```

   这段代码的问题在于, callback中用**return true**来表示是否删除消息, 如果对消息的处理过程是异步的, 我们将无法确保处理完成后再删除消息.
<br>
修改后的notifyRecv原型如下:

```javascript
mq.notifyRecv(function(err, message, doneP){
    console.log(message);
    if(err && err.message === "NetworkBroken"){
        // Best to restart the process when this occurs 
        throw err;
    }
    
    setImmediate(function (){
    	// this will cause message to be deleted automatically 
    	doneP(true).then(function (){
    		// do something
    		return null;
    	}).catch(function (err){
    		// handle delete message errors!
    	})
    });
    
});
```

另一个问题是, 采用notifyRecv(callback)这种函数原型,在异步的情况下, 会导致callback重入的问题. <br>
如果需要确保callback被多次调用的期间, 函数域之间是相互独立的, 需要使用闭包(closure)处理, 如下:

```javascript
mq.notifyRecv(function(err, message, doneP){
    (function (err, message, doneP){
        console.log(message);
        if(err && err.message === "NetworkBroken"){
            // Best to restart the process when this occurs 
            throw err;
        }
        
        setImmediate(function (){
            // this will cause message to be deleted automatically 
            doneP(true).then(function (){
                // do something
                return null;
            }).catch(function (err){
                // handle delete message errors!
            })
        });    
    })(err, message, doneP);   
});
```

# License
MIT

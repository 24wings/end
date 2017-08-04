# express

express快速入门


express应用


原有的 express 应用

app.ts
```ts

import express = require('express');
import path = require('path');
var favicon = require('serve-favicon');
import logger = require('morgan');
import cookieParser = require('cookie-parser');
import session = require('express-session');
import bodyParser = require('body-parser');
import middle = require('./middle');
import nunjucks = require('nunjucks');
import service = require('./services');
import { Route } from './route';
import { Middleware } from './middleware';

import moment = require('moment');

import { ShareRoute, CommonMiddle, WechatRoute, ApiRoute } from './middle';



var app = express();

var njk = nunjucks.configure(path.resolve(__dirname, '../views'), { // 设置模板文件的目录，为views
    autoescape: true,
    express: app,
    noCache: true,
})
//
njk.addFilter('time', function (obj: Date) {
    return moment(obj).format('YYYY-MM-DD');
})
njk.addFilter('json', function (obj) {
    return JSON.stringify(obj)
})
njk.addFilter('money', function (money: number) {
    return money.toFixed(2);
})
njk.addFilter('boolean', function (ok) {
    return !!ok;
});

// app.set('trust proxy', 1) // trust first proxy 

app.use(Middleware.MiddlewareBuilder.buildMiddleware(CommonMiddle))
     .use(express.static(path.resolve(__dirname, '../public')))
    .set('view engine', 'html')
    app.use(favicon(path.resolve(__dirname, '../public', 'favicon.ico')));
      .use(logger('dev')) 
     .use(bodyParser.json({ limit: '50mb' }))
     .use(bodyParser.urlencoded({ extended: false }))
     .use(cookieParser())
     .use(session({
     secret: 'keyboard cat',
     resave: false,
 saveUninitialized: true,
     cookie: { maxAge: 60 * 60 * 24 * 1000 }
     }))
    // 下面是路由
     .use('/wechat/oauth', middle.common.wechatOauth)
     .use('/payment', (req, res, next) => { })
     .get('/', (req, res) => res.redirect('/share'))
     .get('/share', middle.share.index)
    .all('/', service.wechat.wechat(service.CONFIG.wechat, (req, res, next) => {
        var parent = req.query.parent;
        console.log(req.query)
        var url = service.wechat.client.getAuthorizeURL(`${service.CONFIG.domain}/wechat/oauth` + (parent ? '?parent=' + parent : ''), '', 'snsapi_userinfo');
        res.reply({ content: url, type: 'text' });
    }))
    .use('/wechat/:action', Route.RouteBuilder.buildRoute(WechatRoute))
    .use('/share/:action', Route.RouteBuilder.buildRoute(ShareRoute))
    .use('/api/:action', Route.RouteBuilder.buildRoute(ApiRoute))

    .use((err, req, res, next) => {
        // set locals, only providing error in development
        res.locals.message = err.message;
        res.locals.error = req.app.get('env') === 'development' ? err : {};
        console.log(err);
        // render the error page
        res.status(err.status || 500);
        res.render('error');

    });
export =app;  

```


model.ts

```ts
import mongoose = require('mongoose');
import { taskModel } from './Task';
import { userModel } from './User';
import { taskTagModel } from './TaskTag';
import { taskRecordModel } from './TaskRecord';
import { wxRechargeRecordModel } from './WXRechargeRecord';
import { wxGetMoneyRecordModel } from './WXGetMoneyRecord';
import { getMoneyRequestModel } from './GetMoneyRequest';
import { boardModel } from './Board';
import { boardRecordModel } from './BoardRecord';
import { bannerModel } from './Banner';
import { taskTemplateModel } from './TaskTemplate';
import { advertModel } from './advert';
mongoose.connect('mongodb://localhost:27017/test');

export var db = {
    userModel,
    taskModel,
    taskTagModel,
    taskRecordModel,
    wxGetMoneyRecordModel,
    wxRechargeRecordModel,
    getMoneyRequestModel,
    bannerModel,
    taskTemplateModel,
    advertModel
}

```

middle.common.ts
```ts
import {db} from '../model';
import express =  require('express');
export = {
wechatOauth: (req:express.Request,res:express.Response){
    res.redirect('/a/')
}

}


```



但是这样管理路由,当路由出现过多的时候非常难以查找,并且难以清晰的了解应用的布局,以及其他的特色

而一般的对象作为路由,并不利于查找路由,如下

```ts
export class AdminRoute{
        index(){}

        detail(){

        }

}


```

希望有一个doAction作为所有请求的入口,方便查找路由,通过 F12定位到函数入口


```ts
export class AdminRoute{
        doAction(action:string,method:string,next){
            switch(action){
                case 'index': return this.index;//F12可以定位到路由去
                case 'about': return this.about;
            }

        }

        index(){}

        about(){}


}


```


如何实现?

在第一个小步骤中创建控制器实体类
```ts
interface IController{
    doAction:(action:string,method:string,next)=>express.RequestHandle;
}

export class ShareController implements IController{
    doAction(action:string,method:string,next){
        return this.index
    }

    index(){}


}



// lib/index.ts

export function ContollerBuilder(controllerClass:IController):Express.RequestHanlder{
    let ctrl =new controllerClass();

    return  (req,res,next)=>{
        let runtimeThis =  {
            db:db,
            service:service
        }
            ctrl.doAction.bind(runtimeThis)(req.params.action)
    }    

}
export class BaseController{
    service=service;
    db=db;
}


// app.ts
app.use('/share/:action',ControllerBuilder(ShareController))


```


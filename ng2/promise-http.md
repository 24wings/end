通常rest风格的http, 我们希望能通过http直接获取服务器的数据而不是一大堆的回调

例如jquery的ajax 就不是很优美

```javascript
$.ajax({
    url:'',
    data:'',
    sucess:function(rtn){
        if(rtn.ok){
            let dataList =  rtn.data;
            //业务代码
        }else{
            //当服务器出现错误,或者用户权限等,非法操作,参数错误,data是错误的原因
            alert(rtn.data)
        }
    }
})
```



所以我们希望能够这样写代码


```javascript
/**
* 已经将错误在底层进行处理,如果服务器返回 
{
    ok:true,
    data:[]  
}
则data是对象
若
{
    ok:false,
    data:'用户名或密码错误'
}
则alert('用户名或密码错误')
*/
let data =await  http.get('xxx?_id=');
```



```javascript
class MyHttp{

 Get()   {
    return this.http.get(`${this.serverIp}/share-server/${action}`, options)
 .toPromise()
 .then(rtn => { let result = rtn.json(); return result.ok ? result.data : alert(result.data); });
 }

}
 ```



然而这样并不是最完美的,我们希望能将服务器ip和对应Controller抽离出来,这样可以便于修改 ip 和Controller,例如线上环境的ip和本地ip就是不同的
 ```javascript
export class MyHttp{
    serverIp='http://192.168.0.1'
    onlineServerIp= 'http://47.92.87.28'
    Get(url:string,options?:RequestOptions){
        return this.http.get(`${this.serverIp}/admin`,options).toPromise().then(rtn=>{let result = rtn.json(); return result.ok ? result.data: alert(result.data);});
    }
    GetManage(url:string,options?:RequestOptions){
        return this.http.get(`${this.serverIp}/manage`,options).toPromise().then(rtn=>{let result = rtn.json(); return result.ok ? result.data: alert(result.data);});

    }
    GetServer(url:string,options?:RequestOptions){
return this.http
.get(`${this.onlineServerIp}/server`,options)
.toPromise()
.then(rtn=>{
    let result = rtn.json();
 return result.ok ?  result.data: alert(result.data);
 });
}

}

```

这样就可以写出不同的请求传向不同的服务器

app.component.ts
```javascript
export class MyServer{
GetUsers(){
this.users =await    this.tool.Get(`user-list`)
}
GetTasks(){
await this.tools.GetManage('task-list);
}

}
```
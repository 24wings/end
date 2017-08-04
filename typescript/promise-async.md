# Promise 的基础用法

异步编程的模型,
nodejs有许多api有两个版本,
同步和异步回调
如 fs(file system)文件系统


* writeFile(filepath,data,(err,rtn)=>{})  异步方法
* writeFileSync(filepath,data,(err,rtn)=>{}) 同步方法


当调用同步方法的时候,整个程序都会直接执行这个方法,代码等待同步方法执行完成后才会执行下一行代码,如下

```javascript
import fs = require('fs');


console.log('开始写完文件);
fs.writeFileSync('temp.txt','这是一句话');
console.log('文件写完了');


/**
输出如下:

开始写完文件
文件写完了
*/

```
而一般调用同步方法,会影响整个程序的功能的执行,比如一个超大的文件的读写,需要20秒钟,当调用同步方法,web服务器将有20秒完全陷入读写文件中,不能处理用户的请求

所以引出了异步方法,异步方法不会阻塞应用程序的执行,而当应用程序读写文件的时候,它

```javascript

console.log(`开始写完文件`)
fs.writeFile('temp.txt','这是一句话',(err,rtn)=>{
    if(err) console.error(err);
    console.log('文件写完了)
})
console.log('下一行程序');


/**
* 当文件较大时  输出如下:
*
*  开始写完成
*  下一行程序
*  文件写完了
*
**/

```



 现在出现一个问题,如果回调太多,代码就不优雅了,如下,希望能够把temp.txt读写到log.txt,
----

```javascript

fs.readFile('temp.txt',(err,data)=>{
    if(err) console.error(err);
    fs.writeFile('log.txt',function(err,data){
        if(err)console.error(err);
            console.log('文件写完了') ;
    })
})
```

然而异步出现的问题就是回调太多,所以为了解决异步问题引出了Promise,
Promise一旦创建就会指向两个状态
* resolve 成功
* reject  失败  现在nodejs已经弃用所以大部分都推荐使用 resolve
```javascript

// 用Promise 封装好的WriteFile
 function WriteFile(filename:string,content:string){
        return  new Promise((resolve,rejct)=>{
         fs.writeFile(filename,(err,data){
             if(err) throw new Error(err);//或者console.error(err)//resolve(err)
             resolve(data)
         }) 
        });
 }

// 用Promise封装好的 readFile
 function ReadFile(filename:string){
    return  new Promise((resolve,reject)=>{
        fs.readFile(filename',(err,data){
            if(err) throw new Error(err);
            resolve(data);
        })
    })
 }


//完美而简洁的方案;
async function  writeTemp2Log(){
   let conent =  await ReadFile('temp.txt');
    let result =  await WriteFile('log.txt');
}
```


扩展阅读

[es6 Promise 阮一峰](http://es6.ruanyifeng.com/#docs/promise)

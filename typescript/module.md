# module 模块化开发


本文倾向于typescript模块化开发的技巧



一般在nodejs 中,或commonjs 规范中, 
`require`是加载模块方法,  

* `require('http')` 是从环境变量里加载 nodejs内置库
* `require('express')`   是从当前项目的node_modules下加载 `expresss`文件夹
*  `require('./lib')` 
    * 若 lib是文件夹 则加载 当前文件夹下的lib/index.js 或者lib/package.json 下的main属性的文件
    * 若 lib是文件直接加载文件

```javascript
var http   = require('http');
var exprees=  require('express');

```

但是用javascript写nodejs代码在typescript也是可以的,然而一般我们会希望引入http的同时,引入http的类型系统,这样才能在写代码的时候能够有智能语法提示和类型检测系统.

所以对于文件的解析方式,我们依然选择node

tsconfig.json
```json
{
"compilerOptions":{
    "target":"es5",
 "moduleResolution": "node"
}

}


```


```typescript

// 将类型定义文件同时引入
import   http  = require('http');
http.createServer((req,res)=>{res.end('hello world')}).listen(80)
```



上面讲解了模块的引入,现在讲解模块的导出


```typescript

//在 a.ts
export  var a =1;


//在 b.ts

import {a} from './a';

// 1
console.log(a)

```


而在es6 中指定 了 export default,本质上是导出一个变量名为default的属性


```typescript
// a.ts
export default = 1;
//
export var default =1

// b.ts    变量名可以自定义,这就是导出default
import  variable  from './a';
// 
```





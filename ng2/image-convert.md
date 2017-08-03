# ng2-图片压缩



一般我们会在上传头像,预览图等功能在 ng2的组件html里这样写

app.component.html
```html
<input type="file" (change)="uploadAvatar()" #avatarSelect>
```



对于选取的图片,我们希望有两步操作
* 第一步是:将文件转换成base64上传
* 第二步是:压缩到一定大小一下

```javascript

class ToolService{
    
  /**
   * 
   * @param url   file
  * 
   * @param outputFormat string
   * 
   * 将文件转成base64
   */
  convertImageToBase64(base64: string, maxSize: number = 40000, outputFormat = "image/png"): Promise<string> {

    return new Promise((resolve, reject) => {
      var canvas = document.createElement('canvas');
      var ctx = canvas.getContext('2d');
      var img = new Image;
      img.crossOrigin = 'Anonymous';
      img.onload = function () {
        var width = img.width;
        var height = img.height;
        let compress = 1;
        let rate =1
        if (width * height > maxSize) {  rate= Math.ceil(width * height / 40000); }
        compress = 1 / rate;
        canvas.width = width * compress;
        canvas.height = height * compress;
        ctx.drawImage(img, 0, 0, width, height, 0, 0, width * compress, height * compress);
        let compressData = canvas.toDataURL(outputFormat)
        resolve(compressData);
      };
      img.src = base64;
    });
  }

  convertFileToBase64(file: File): Promise<string> {
    let reader = new FileReader();
    return new Promise((resolve, reject) => {
      reader.onload = (e) => {
        let base64: string = <string>e.target['result']
        resolve(base64);
      };
      reader.readAsDataURL(file);
    });

  }

}

```

调用示例   app.component.ts
```javascript
export class AppComponent{
    constructor(public tool:ToolService){}

    uploadImage(file:File){
        // 文件转化成base64,大部分文件可以直接用base64上传
        let base64 =  this.tool.convertFileToBase64(file);
        // 图片文件还可以进一步压缩
        let compressData = this.tool

    }

}
```
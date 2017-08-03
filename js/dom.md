



```javascript
 function convertImgToBase64(url, callback, outputFormat) {
        var canvas = document.createElement('CANVAS');
        var ctx = canvas.getContext('2d');
        var img = new Image;
        img.crossOrigin = 'Anonymous';
        img.onload = function() {
            var width = img.width;
            var height = img.height;
            let compress = 1;
            // 按比例压缩4倍
            console.log(width * height);
            if (width * height > 40000) {
                var rate = Math.ceil(width * height / 40000);


            }
            // var rate = (width < height ? width / height : height / width) / 4;
            compress = 1 / rate;
            // var ratio;
            // if ((ratio = width * height / 40000) > 1) {
            // ratio = Math.sqrt(ratio);
            // width /= ratio;
            // height /= ratio;
            // } else {
            // ratio = 1;
            // }

            canvas.width = width * compress;
            canvas.height = height * compress;
            ctx.drawImage(img, 0, 0, width, height, 0, 0, width * compress, height * compress);
            // ctx.drawImage(img, 0, 0, width, height, 0, 0, width * ratio, height * ratio);

            var dataURL = canvas.toDataURL(outputFormat || 'image/png');
            callback.call(this, dataURL);
            canvas = null;
        };
        img.src = url;
    }
    ```
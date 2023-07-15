
# easyJBIG2show
an easy JBIG2 file web show

# 一、背景
  
  最近无意中接触到了一个二维码图片，该图片格式是jb2格式。翻阅资料发现JBIG标准最初在1993年发布，在当时被广泛应用于传真机和文档扫描仪等设备中。JBIG采用了一种自适应二进制编码算法，能够有效地压缩包含大量文本和线条等的二值图像。  
  随着技术的进步和需求的增加，JBIG2标准于2000年发布，对于从传真到高分辨率彩色扫描文档等各种类型的图像进行无损压缩提供了更好的性能和灵活性。JBIG2引入了更先进的编码技术，如上下文自适应算法和模板匹配，能够进一步减小文件大小，提高压缩率。
  JBIG2标准的发布促使了许多设备和应用程序的支持。它被广泛用于电子归档、数字图书馆、OCR（光学字符识别）和文档管理等领域，以实现高效的文档扫描和存储。
  总的来说，JBIG2作为一种专门针对二值图像的无损压缩技术，为高质量的文档图像处理提供了重要的工具和方法，并在数字化文档管理和传输中发挥了重要的作用。  
  由于web端的图片不能对jb2文件格式正确展示，由此想到需要转换。

# 二、前人经验
  网络资源很少，这也是为什么在这里分享的原因。虽然很少，但还是找了些蛛丝马迹，指向了mozilla/pdf.js。  
  在这里我把需要使用的所有js都整合到eaysjbig2.js文件内主要包括了：  
  （1） "../shared/util.js" 文件内的 shadow;  
  （2）"./core_utils.js" 文件内的log2, readInt8, readUint16, readUint32;  
  （3）"./arithmetic_decoder.js";  
  （4）jbig2.js

# 三、使用方法
   在index.html中，尝试打开一个jb2文件格式并把他展示到canvas上。  
```html   
 <!DOCTYPE html>
<html>
<head>
    <title>打开本地文件</title>
    <script>
        function handleFileSelect(event) {
            var files = event.target.files; // 获取选择的文件数组
            var file = files[0]; // 获取第一个文件
            var reader = new FileReader();
            reader.onload = function (e) {
                var arrayBuffer = e.target.result;
                // 创建一个Uint8Array来存储文件内容
                var uintArray = new Uint8Array(arrayBuffer);
                var jbig2 = new Jbig2Image();
                var data = jbig2.parse(uintArray);
                var canvas = document.createElement('canvas');
                canvas.width = jbig2.width;
                canvas.height = jbig2.height;
                var ctx = canvas.getContext('2d');
                
                var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                var data1 = imageData.data;
                for (var i = 0,d=0; i < data1.length; i += 4,d++) {
                    let sdata = data[d]
                    if (sdata == 255) {
                        // 修改像素的颜色或透明度等
                        data1[i] = 255; // 红色通道（0-255）
                        data1[i + 1] = 255;   // 绿色通道（0-255）
                        data1[i + 2] = 255;   // 蓝色通道（0-255）
                    } else {  // 修改像素的颜色或透明度等
                        data1[i] = 0; // 红色通道（0-255）
                        data1[i + 1] = 0;   // 绿色通道（0-255）
                        data1[i + 2] = 0;   // 蓝色通道（0-255）
                    }
                    data1[i + 3] = 255; // Alpha 通道（0-255）
                }
                ctx.putImageData(imageData, 0, 0);
                document.body.appendChild(canvas);
            }
            reader.readAsArrayBuffer(file);
        }
    </script>
</head>
<body>
    <input type="file" onchange="handleFileSelect(event)">
</body>
<script src="jbig2.js"></script>
</html>
```

   
   当然你可以把处理的文件直接存为BMP图片格式文件，利用image元素来进行展示，伪代码如下：  
   ```html

  <script>
// 获取二进制数据，假设为Uint8Array类型的data
var data = new Uint8Array([...]);

// 定义BMP文件头
var fileHeaderSize = 14;
var infoHeaderSize = 40;
var fileSize = fileHeaderSize + infoHeaderSize + data.length;
var width = 100;  // BMP图像宽度
var height = 100;  // BMP图像高度

// 创建一个Uint8Array来保存BMP文件内容
var bmpData = new Uint8Array(fileSize);

// 设置文件类型标识 "BM"
bmpData.set([0x42, 0x4D], 0);

// 设置文件大小
bmpData.set(new Uint8Array([fileSize & 0xFF, (fileSize >> 8) & 0xFF, (fileSize >> 16) & 0xFF, (fileSize >> 24) & 0xFF]), 2);

// 设置保留字段（设置为0）
bmpData.set([0, 0, 0, 0], 6);

// 设置文件内容的偏移量（从文件头开始）
var offset = fileHeaderSize + infoHeaderSize;
bmpData.set(new Uint8Array([offset & 0xFF, (offset >> 8) & 0xFF, (offset >> 16) & 0xFF, (offset >> 24) & 0xFF]), 10);

// 设置信息头大小
bmpData.set(new Uint8Array([infoHeaderSize & 0xFF, (infoHeaderSize >> 8) & 0xFF, (infoHeaderSize >> 16) & 0xFF, (infoHeaderSize >> 24) & 0xFF]), 14);

// 设置图像宽度和高度
bmpData.set(new Uint8Array([width & 0xFF, (width >> 8) & 0xFF, (width >> 16) & 0xFF, (width >> 24) & 0xFF]), 18);
bmpData.set(new Uint8Array([height & 0xFF, (height >> 8) & 0xFF, (height >> 16) & 0xFF, (height >> 24) & 0xFF]), 22);

// 设置图像颜色面数（设置为1）
bmpData.set(new Uint8Array([1, 0]), 28);

// 设置每个像素的比特数（设置为8，灰度图像）
bmpData.set(new Uint8Array([8, 0]), 30);

// 将二进制图像数据追加到BMP数据中
bmpData.set(data, offset);

// 创建Blob对象
var blob = new Blob([bmpData], { type: 'image/bmp' });

 // 创建Image对象
var img = new Image();

// 为Image对象设置src为blob URL
img.src = URL.createObjectURL(blob);

// 将Image对象添加到网页中的某个元素
document.getElementById("image-container").appendChild(img);
 </script>
   ```
# 四、结果展示
  ![image](https://github.com/11627685/easyJBIG2show/blob/main/easyJBIG2.png)


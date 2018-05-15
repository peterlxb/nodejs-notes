![](/assets/屏幕快照 2018-05-12 下午8.47.51.png)

## 示例代码

```
const crypto = require('crypto');
const fs = require('fs');
const https = require('https');

const start = Date.now();

function doRequest() {
  https.request('https://jsonplaceholder.typicode.com/posts/1', res => {
      res.on('data',() => {});
      res.on('end',() => {
        console.log(Date.now() - start);
      });
    })
    .end();
}

doRequest();

function doHash() {
  crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
    console.log("Hash:", Date.now() - start);
  });
}

fs.readFile('multitask.js','utf8',() => {
  console.log("FS:", Date.now() - start);
});

doHash();
doHash();
doHash();
doHash();
```

打印出来的结果

```
987
Hash: 2736
FS: 2738
Hash: 2752
Hash: 2765
Hash: 2771
```

读取文件操作

![](/assets/屏幕快照 2018-05-15 下午4.56.23.png)

HTTP操作默认由操作系统底层代理。与thread pool无关。

![](/assets/屏幕快照 2018-05-15 下午4.55.54.png)






# Get Started with PearPlayer

**PearPlayer** is a multi-source and multi-protocol P2P streaming player that works in the **browser**. It's easy
to get started!

## Import
### Script
Simply include the
([`pear-player.min.js`](dest/pear-player.min.js))
script on your page and use `require()`:
```html
<script src="pear-player.min.js"></script>
```

### Browserify
To install PearPlayer for use in the browser with `require('PearPlayer')`, run:
```bash
npm install pearplayer --save
```
Then you can require PearPlayer like this:
```js
var PearPlayer = require('PearPlayer');
```

## Quick Examples

### Hook the video and play

```js
var PearPlayer = require('PearPlayer');
var xhr = new XMLHttpRequest();
//CP需要先登录来获取token
xhr.open("POST", 'https://api.webrtc.win:6601/v1/customer/login');
var data = JSON.stringify({
    user:'test',
    password:'123456'
});
xhr.onload = function () {
    if (this.status >= 200 && this.status < 300) {

        var res = JSON.parse(this.response);
        if (!!res.token){
            console.log('token:' +res.token);

            var player = new PearPlayer('#pearvideo', res.token, {//第一个参数为video标签的id或class
                type: 'mp4',                           //播放视频的类型，目前只能是mp4
                algorithm: 'firstaid',                 //核心算法,默认firstaid
                autoplay: true,                        //是否自动播发视频，默认true
                chunkSize: 1*1024*1024,                //每个chunk的大小，必须是32K的整数倍,默认1M
                interval: 5000,                        //滑动窗口的时间间隔,单位毫秒，默认10s
                auto: false,                           //true为连续下载buffer，false则是只有当前播放时间与已缓冲时间小于slideInterval时下载buffer，如果是fmp4建议设为true，默认false
                slideInterval: 10,                     //当前播放时间与已缓冲时间小于这个数值时触发窗口滑动,单位秒,默认20s
                useDataChannel: true,                  //是否开启Datachannel,默认true
                dataChannels: 1,                       //创建DataChannel的最大数量,默认3
                useMonitor: true                       //是否开启monitor，会稍微影响性能，默认true
            });
        }
    } else {
        alert('请求出错!');
    }
};
xhr.send(data);
```

There is a complete example in [test/test.html](test/test.html)。

### Listen to PearPlayer events

```js
var player = new PearPlayer('#pearvideo', {      //第一个参数为video标签的id或class
                    type: 'mp4',                 //播放视频的类型,目前只能是mp4
                    src: 'https://qq.webrtc.win/tv/pear001.mp4',  //视频播放的src
                    token: res.token,            //与信令服务器连接的token,必须
                    useMonitor: true             //是否开启monitor,会稍微影响性能,默认true
                });

                player.on('exception', onException);
                player.on('begin', onBegin);
                player.on('progress', onProgress);
                player.on('cloudspeed', onCloudSpeed);
                player.on('fogspeed', onFogSpeed);
                player.on('fograte', onWebRTCRate);
                player.on('buffersources', onBufferSources);               //s: server   n: node  d: data channel  b: browser
                player.on('done', onDone);
                
function onBegin(fileLength, chunks) {
    console.log('start downloading buffer by first aid, file length is:' + fileLength + ' total chunks:' + chunks);
}

function onProgress(downloaded) {
    console.log('Progress: ' + (downloaded * 100).toFixed(1) + '%');
}

function onCloudSpeed(speed) {
    console.log('Cloud download speed: ' + speed + 'KBps');
}

function onFogSpeed(speed) {
    console.log('Fog download speed: ' + speed + 'KBps');
}

function onWebRTCRate(p2pRate) {
    console.log('Fog Rate: ' + (p2pRate * 100).toFixed(1) + '%');
}

function onDone() {
    console.log('finished downloading buffer by first aid');
}

function onException(exception) {
    var errCode = exception.errCode;
    switch (errCode) {
        case 1:                   //当前浏览器不支持WebRTC
        console.log(exception.errMsg);
            break
    }
}
function onBufferSources(bufferSources) {    //s: server   n: node  d: data channel  b: browser
    console.log('Current Buffer Sources:' + bufferSources);
}
```

### Complete HTML page example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pear Player</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
<div id="main">
    <video id="pearvideo" src="https://qq.webrtc.win/tv/pear001.mp4" controls>
    </video>
</div>
<script src="../pear-player.js"></script>
<script>
    var xhr = new XMLHttpRequest();
    xhr.open("POST", 'https://api.webrtc.win:6601/v1/customer/login');
    var data = JSON.stringify({
        user:'test',
        password:'123456'
    });
    xhr.onload = function () {
        if (this.status >= 200 && this.status < 300) {

            var res = JSON.parse(this.response);
            if (!!res.token){
                console.log('token:' +res.token);
                //第一个参数为video标签的id或class
                var player = new PearPlayer('#pearvideo', res.token, {
                    algorithm: 'firstaid',      //核心算法,默认firstaid
                    autoplay: true,            //是否自动播发视频,默认true
                    chunkSize: 1*1024*1024,        //每个chunk的大小,必须是32K的整数倍,默认1M
                    interval: 5000,             //滑动窗口的时间间隔,单位毫秒,默认10s
                    auto: false,                //true为连续下载buffer,false则是只有当前播放时间与已缓冲时间小于slideInterval时下载buffer,如果是fmp4建议设为true,默认false
                    slideInterval: 10,          //当前播放时间与已缓冲时间小于这个数值时触发窗口滑动,单位秒,默认20s
                    useDataChannel: true,       //是否开启data channel,默认true
                    dataChannels: 1,            //创建data channel的最大数量,默认3
                    useMonitor: true             //是否开启monitor,会稍微影响性能,默认true
                });

                player.on('exception', onException);               //播放器出现异常时的回调函数
                player.on('begin', onBegin);　　　　　　　　　　　　　 //开始下载时触发
                player.on('progress', onProgress);                 //回调目前的下载进度
                player.on('cloudspeed', onCloudSpeed);             //来自server的HTTP的平均下载速度
                player.on('fogspeed', onFogSpeed);                 //来自fog的节点（包括HTTP和WebRTC）的平均下载速度
                player.on('fograte', onWebRTCRate);　　　　　　　　　 //fog节点的下载比率（下载的字节数除以总的字节数）
                player.on('buffersources', onBufferSources);       //buffer map，记录每个buffer的下载源类型，其中s: server   n: node  d: data channel  b: browser
                player.on('done', onDone);　　　　　　　　           //结束下载时触发

            }
        } else {
            alert('请求出错!');
        }
    };
    xhr.send(data);



    function onBegin(fileLength, chunks) {
        console.log('start downloading buffer by first aid, file length is:' + fileLength + ' total chunks:' + chunks);
    }

    function onProgress(downloaded) {
        console.log('Progress: ' + (downloaded * 100).toFixed(1) + '%');
    }

    function onCloudSpeed(speed) {
        console.log('Cloud download speed: ' + speed + 'KBps');
    }

    function onFogSpeed(speed) {
        console.log('Fog download speed: ' + speed + 'KBps');
    }

    function onWebRTCRate(p2pRate) {
        console.log('Fog Rate: ' + (p2pRate * 100).toFixed(1) + '%');
    }

    function onDone() {
        console.log('finished downloading buffer by first aid');
    }

    function onException(exception) {
        var errCode = exception.errCode;
        switch (errCode) {
            case 1:
                console.log(exception.errMsg);
                break
        }
    }
    function onBufferSources(bufferSources) {            //s: server   n: node  d: data channel  b: browser
        console.log('Current Buffer Sources:' + bufferSources);
    }
</script>
</body>
</html>
```
## Build

PearPlayer works great with [browserify](http://browserify.org/), which lets
you use [node.js](http://nodejs.org/) style `require()` to organize your browser
code, and load packages installed by [npm](https://npmjs.org/).

```bash
npm install -g browserify
```
Install dependencies:
```bash
npm install
```
To get a normal size bundle,use:
```bash
npm run build
```
To get a compressed bundle,use:
```bash
npm run build
npm run gulp
```

## More Documentation

Check out the [API Documentation](https://github.com/PearInc/PearPlayer.js/blob/master/docs/api.md)
and [FAQ](https://github.com/PearInc/PearPlayer.js/blob/master/docs/faq.md) for more details.

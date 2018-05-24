---
title: WebAPP升级方案(iOS版)
date: 2018-04-19 11:33:51
tags: 
- WebAPP
- DCloud
---

![webapp](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1525968985934&di=159aac25c342f55b8ad94fb91c81d2d9&imgtype=jpg&src=http%3A%2F%2Fimg2.imgtn.bdimg.com%2Fit%2Fu%3D616152967%2C869593181%26fm%3D214%26gp%3D0.jpg)

<!-- more -->

### 服务器部分

服务端提供:

- 版本检测接口
  e.g. `http://192.168.11.1:3000/version`
- H5+包下载地址
  e.g. `http://192.168.11.1:3000/download/HelloH5.wgtu`

**H5+包命名**： `Supervisor.wgtu`
格式： package_name.wgtu

> H5+包使用zip压缩

### JS部分

执行步骤:

- 版本检测
- 下载wgtu包
- 安装wgtu包
- 重启

#### 版本检测

```js
function checkVersion() {
    var wgtVer = null;
    plus.runtime.getProperty( plus.runtime.appid, function ( wgtinfo ) {
        //version属性
        wgtVer = wgtinfo.version;
        outSet( wgtStr );
    } );

    var url = "http://192.168.11.1:3000/version";
    var xhr = new plus.net.XMLHttpRequest();
    xhr.onreadystatechange = function () {
        switch ( xhr.readyState ) {
            case 4:
                if ( xhr.status == 200 ) {
                    outSet("检测更新 版本：" + xhr.responseText);
                    var newVer = xhr.responseText;
                    if (wgtVer && newVer && (wgtVer != newVer)) {
                        outSet("有新版本:，可更新！！, 旧版本:");
                        alert("有新版本:" + newVer + "，可更新！！, 旧版本:" + wgtVer);
                    } else {
                        outSet("无新版本可更新！");
                        alert("无新版本可更新！");
                    }
                } else {
                    outSet("检测更新失败！");
                    alert("检测更新失败！");
                }
                break;
            default :
                break;
        }
	}

	xhr.open( "GET", url );
	xhr.send();
}
```

#### H5包升级

```js
function updateApp(){
	var url='http://192.168.11.1:3000/download/HelloH5.wgtu';
	plus.nativeUI.showWaiting("升级中...");
	var dtask = plus.downloader.createDownload( url, {method:"GET"}, function(d,status){
		if ( status == 200 ) { 
			plus.console.log( "Download wgtu success: " + d.filename );
            outSet( "Download wgtu success: " + d.filename );
			plus.runtime.install(d.filename,{},function(){
				plus.nativeUI.closeWaiting();
				plus.nativeUI.alert("Update wgtu success, restart now!",function(){
					plus.runtime.restart();
				});
			},function(e){
				plus.nativeUI.closeWaiting();
				alert("Update wgtu failed: "+e.message);
			});
		} else {
			plus.nativeUI.closeWaiting();
			 alert( "Download wgtu failed: " + status ); 
		} 
	} );
	dtask.addEventListener('statechanged',function(d,status){
		console.log("statechanged: "+d.state);
	});
	dtask.start();
}
```

### iOS部分

- H5包从Build位置向沙盒位置迁移
  依赖`plus.runtime.install`寻找迁移位置
- 动态加载H5+包

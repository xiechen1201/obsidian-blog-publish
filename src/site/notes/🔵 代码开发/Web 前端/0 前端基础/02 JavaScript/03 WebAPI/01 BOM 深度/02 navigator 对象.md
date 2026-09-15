---
{"dg-publish":true,"permalink":"/🔵 代码开发/Web 前端/0 前端基础/02 JavaScript/03 WebAPI/01 BOM 深度/02 navigator 对象/","dg-note-properties":{}}
---

---
---
`window.navigator`表示浏览器信息

```js
window.navigator;
// window 也可以省略
navigator;
```

## 🔢 属性
`navigator`常用的属性

| 属性 | 说明 |
| --- | --- |
| appCodeName | 返回浏览器的代码名 |
| appName | 返回浏览器的名称 |
| appVersion | 返回浏览器的平台和版本信息 |
| cookieEnabled | 返回指明浏览器中是否启用 cookie 的布尔值 |
| platform | 返回运行浏览器的操作系统平台 |
| userAgent | 回浏览器的用户代理字符串 |
| connection | 返回浏览器的网络信息（适用于移动端） |
| onLine | 返回浏览器是否联网 |
| language | 返回设备的使用语言 |
| plugins | 返回浏览器的插件 |
| geolocation | 返回设备的地理位置 |
| watchPosition | 和`geolocation`一样获取设备地位位置，不同的是在设备的地理位置发生改变的时候自动被调用。 |

## 🔢 userAgent
```js
console.log(navigator);
console.log(navigator.userAgent);
// 返回浏览器信息
// 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36'

// 'Mozilla/5.0' 浏览器的代码
// Macintosh 允许平台
// AppleWebKit 浏览器内核
// Chrome 浏览器及版本
```

来个超详细的获取浏览器信息的工具函数：

```js
function checkBrowser(){
  var nVer = navigator.appVersion,
      nAgt = navigator.userAgent,
      browser = navigator.appName,
      version = '' + parseFloat(navigator.appVersion),
      majorVersion,
      nameOffset,
      verOffset,
      ix,
      network = 'unknown';

  // Opera浏览器（老版本）
  if ((verOffset = nAgt.indexOf('Opera')) != -1) {
    browser = 'Opera';
    version = nAgt.substring(verOffset + 6);
    if ((verOffset = nAgt.indexOf('Version')) != -1) {
      version = nAgt.substring(verOffset + 8);
    }
  }
  // Opera浏览器（新版本）
  if ((verOffset = nAgt.indexOf('OPR')) != -1) {
    browser = 'Opera';
    version = nAgt.substring(verOffset + 4);
  }
  // IE浏览器
  else if ((verOffset = nAgt.indexOf('MSIE')) != -1) {
    browser = 'Microsoft Internet Explorer';
    version = nAgt.substring(verOffset + 5);
  }
  // Chrome浏览器
  else if ((verOffset = nAgt.indexOf('Chrome')) != -1) {
    browser = 'Chrome';
    version = nAgt.substring(verOffset + 7);
  }
  // Safari浏览器
  else if ((verOffset = nAgt.indexOf('Safari')) != -1) {
    browser = 'Safari';
    version = nAgt.substring(verOffset + 7);
    if ((verOffset = nAgt.indexOf('Version')) != -1) {
      version = nAgt.substring(verOffset + 8);
    }
  }
  // Firefox浏览器
  else if ((verOffset = nAgt.indexOf('Firefox')) != -1) {
    browser = 'Firefox';
    version = nAgt.substring(verOffset + 8);
  }
  // IE11+浏览器
  else if (nAgt.indexOf('Trident/') != -1) {
    browser = 'Microsoft Internet Explorer';
    version = nAgt.substring(nAgt.indexOf('rv:') + 3);
  }
  // 微信浏览器
  else if (nAgt.indexOf('NetType/') != -1) {
    browser = 'WeiXin';
    if (nAgt.indexOf('NetType/WIFI') != -1) {
      network = 'WIFI';
    }else if(nAgt.indexOf('NetType/2G') != -1) {
      network = '2G';
    }else if(nAgt.indexOf('NetType/3G+') != -1) {
      network = '3G+';
    }
    verOffset = nAgt.lastIndexOf('/')
    version = nAgt.substring(verOffset + 1);
    if (browser.toLowerCase() == browser.toUpperCase()) {
      browser = navigator.appName;
    }
  }
  //其他浏览器
  else if ((nameOffset = nAgt.lastIndexOf(' ') + 1) < (verOffset = nAgt.lastIndexOf('/'))) {
    browser = nAgt.substring(nameOffset, verOffset);
    version = nAgt.substring(verOffset + 1);
    if (browser.toLowerCase() == browser.toUpperCase()) {
      browser = navigator.appName;
    }
  }

  //版本字符串整理
  if ((ix = version.indexOf(';')) != -1) version = version.substring(0, ix);
  if ((ix = version.indexOf(' ')) != -1) version = version.substring(0, ix);
  if ((ix = version.indexOf(')')) != -1) version = version.substring(0, ix);
  majorVersion = parseInt('' + version, 10);
  if (isNaN(majorVersion)) {
    version = '' + parseFloat(navigator.appVersion);
    majorVersion = parseInt(navigator.appVersion, 10);
  }

  //移动版本
  var mobile = /Mobile|mini|Fennec|Android|iP(ad|od|hone)/.test(nVer);

  //系统探测
  var os = '';
  var clientStrings = [
    {s: 'Windows 10', r: /(Windows 10.0|Windows NT 10.0)/},
    {s: 'Windows 8.1', r: /(Windows 8.1|Windows NT 6.3)/},
    {s: 'Windows 8', r: /(Windows 8|Windows NT 6.2)/},
    {s: 'Windows 7', r: /(Windows 7|Windows NT 6.1)/},
    {s: 'Windows Vista', r: /Windows NT 6.0/},
    {s: 'Windows Server 2003', r: /Windows NT 5.2/},
    {s: 'Windows XP', r: /(Windows NT 5.1|Windows XP)/},
    {s: 'Windows 2000', r: /(Windows NT 5.0|Windows 2000)/},
    {s: 'Windows ME', r: /(Win 9x 4.90|Windows ME)/},
    {s: 'Windows 98', r: /(Windows 98|Win98)/},
    {s: 'Windows 95', r: /(Windows 95|Win95|Windows_95)/},
    {s: 'Windows NT 4.0', r: /(Windows NT 4.0|WinNT4.0|WinNT|Windows NT)/},
    {s: 'Windows CE', r: /Windows CE/},
    {s: 'Windows 3.11', r: /Win16/},
    {s: 'Android', r: /Android/},
    {s: 'Open BSD', r: /OpenBSD/},
    {s: 'Sun OS', r: /SunOS/},
    {s: 'Linux', r: /(Linux|X11)/},
    {s: 'iOS', r: /(iPhone|iPad|iPod)/},
    {s: 'Mac OS X', r: /Mac OS X/},
    {s: 'Mac OS', r: /(MacPPC|MacIntel|Mac_PowerPC|Macintosh)/},
    {s: 'QNX', r: /QNX/},
    {s: 'UNIX', r: /UNIX/},
    {s: 'BeOS', r: /BeOS/},
    {s: 'OS/2', r: /OS\/2/},
    {s: 'Search Bot', r: /(nuhk|Googlebot|Yammybot|Openbot|Slurp|MSNBot|Ask Jeeves\/Teoma|ia_archiver)/}
  ];
  for (var id in clientStrings) {
    var cs = clientStrings[id];
    if (cs.r.test(nAgt)) {
      os = cs.s;
      break;
    }
  }
  var osVersion = '';
  if (/Windows/.test(os)) {
    osVersion = /Windows (.*)/.exec(os)[1];
    os = 'Windows';
  }
  switch (os) {
    case 'Mac OS X':
      osVersion = /Mac OS X (10[\.\_\d]+)/.exec(nAgt)[1];
      break;
    case 'Android':
      osVersion = /Android ([\.\_\d]+)/.exec(nAgt)[1];
      break;
    case 'iOS':
      osVersion = /OS (\d+)_(\d+)_?(\d+)?/.exec(nVer);
      osVersion = osVersion[1] + '.' + osVersion[2] + '.' + (osVersion[3] | 0);
      break;
  }

  //返回数据集合
  return {
    //操作系统
    os: os,
    //操作系统版本
    osVersion: osVersion ? osVersion : 'unknown',
    //是否移动端访问
    mobile: mobile,
    //浏览器类型
    browser: browser,
    //浏览器版本
    browserVersion: version,
    //浏览器major版本
    browserMajorVersion: majorVersion
  };

}
```

## 🔢 geolocation
`geolocation.getCurrentPosition()`方法可以获取设备的地理位置，在电脑浏览器中，因为电脑没有`GPS`的装置，所以会去`Google`地图查询你的定位，所以需要科学上网！

```js
window.navigator.geolocation.getCurrentPosition((e) => {
  /**
	 * e.coords 坐标
   * e.latitude 经度
   * e.longitude 纬度
   */
  console.log("获取成功：", e);
},(error) => {
  console.log("获取失败：", error);
},{
  // 配置项
  timeout: 3000 // 超时
});
```

## 🔢 方法
常用的方法：

| 方法 | 说明 |
| --- | --- |
| vibrate() | 触发手机震动 |

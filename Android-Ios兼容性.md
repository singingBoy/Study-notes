# 最近一些 [Android\Ios](#) 兼容性问题记录

> android端，刷新页面location.reload()、window.location.href去跳转页面无效

缓存原因？

`location.reload(location.href+'?time='+((new Date()).getTime())); // 也无效！`

<B style="color:red"> ➡原因：</b><br/>
href是location对象的一个属性，reload()则是location对象的方法
所以对于href，可以为该属性设置新的 URL，使浏览器读取并显示新的 URL 的内容。
对于reload()则是重新加载当前文档，如果该方法没有规定参数，或者参数是 false，它就会用 HTTP 头 If-Modified-Since 来检测服务器上的文档是否已改变。如果文档已改变，reload() 会再次下载该文档。如果文档未改变，则该方法将从缓存中装载文档。这与用户单击浏览器的刷新按钮的效果是完全一样的。如果把该方法的参数设置为 true，那么无论文档的最后修改日期是什么，它都会绕过缓存，从服务器上重新下载该文档。这与用户在单击浏览器的刷新按钮时按住 Shift 健的效果是完全一样。
把该方法的参数设置为 true，那么无论文档的最后修改日期是什么，它都会绕过缓存，从服务器上重新下载该文档。

<b>但对于安卓手机微信中的浏览器，reload只是从缓存中装载文档，所以当你使用该方法，是失效的！</b>

<B style="color:red"> ➡解决：</b><br/>
`location.href = location.href+'?time='+((new Date()).getTime());`

> video 在androi、ios微信端表现形式不一样
表现：
1、android默认播放是全屏的
2、播放完毕会植入广告
3、视频播放层级最高，无法调试
4、控制条改变，不可控
5、微信开发者工具表现普通web，与真机表现不一

<B style="color:red"> ➡原因：</b><br/>
企鹅的工程师会劫持所有video标签，生成一个与其等大的native播放框。

<B style="color:red"> ➡补充：</b><br/>
```
<video
   id="videoID"
   src="video.mp4"
   poster="loadbg.jpg"　视频封面
   preload="auto"
   x-webkit-airplay="allow"
   x5-video-player-type="h5"　启用H5播放器,是wechat安卓版特性
   x5-video-player-fullscreen="true"　全屏设置，设置为 true 是防止横屏
   x5-video-orientation="portraint"　播放器支付的方向，landscape横屏，portraint竖屏，默认值为竖屏
   webkit-playsinline="true"　这个属性是ios 10中设置可以让视频在小窗内播放，也就是不是全屏播放
   playsinline="true"　IOS微信浏览器支持小窗内播放
   style="object-fit:fill">
</video>
```
[video 一些属性：](https://blog.csdn.net/qq_16494241/article/details/62046891)

※ preload="auto" ：属性规定在页面加载后载入视频。如果设置了 autoplay 属性，则忽略该属性。
· auto - 当页面加载后载入整个视频
· meta - 当页面加载后只载入元数据
· none - 当页面加载后不载入视频

※ muted：当设置该属性后，它规定视频的音频输出应该被静音

※ webkit-playsinline="true"：视频播放时局域播放，不脱离文档流 。
但是这个属性比较特别， 需要嵌入网页的APP比如WeChat中UIwebview 的allowsInlineMediaPlayback = YES webview.allowsInlineMediaPlayback = YES，才能生效。
换句话说，如果APP不设置，你页面中加了这标签也无效，这也就是为什么安卓手机WeChat 播放视频总是全屏，因为APP不支持playsinline，而ISO的WeChat却支持。

> 这里就要补充下，如果是想做全屏直播或者全屏H5体验的用户，ISO需要设置删除 webkit-playsinline 标签，因为你设置 false 是不支持的 ，
安卓则不需要，因为默认全屏。但这时候全屏是有播放控件的，无论你有没有设置control。 
做直播的可能用得着播放控件，但是全屏H5是不需要的，那么去除全屏播放时候的控件，需要以下设置：同层播放。

※ x5-video-player-type="h5"：
启用同层H5播放器，就是在视频全屏的时候，div可以呈现在视频层上，<b>也是WeChat安卓版特有的属性</b>。
同层播放别名也叫做沉浸式播放，播放的时候看似全屏，但是已经除去了control和微信的导航栏，只留下"X"和"<"两键。
目前的同层播放器只在Android（包括微信）上生效，暂时不支持iOS。
笔者想过为什么同层播放只对安卓开放，因为安卓不能像ISO一样局域播放，默认的全屏会使得一些界面操作被阻拦，如果是全屏H5还好，但是做直播的话，
诸如弹幕那样的功能就无法实现了，所以这时候同层播放的概念就解决了这个问题。不过笔者在测试的过程中发现，不同版本的ISO和安卓效果略有不同。

※ x5-video-orientation：声明播放器支持的方向，可选值landscape 横屏,portraint竖屏。默认值portraint。
无论是直播还是全屏H5一般都是竖屏播放，但是这个属性需要x5-video-player-type开启H5模式

※ x5-video-player-fullscreen="true"：全屏设置。ture和false的设置会导致布局上的不一样

<B style="color:red"> ➡我的解决办法：</b><br/>
封装视频组件，判断浏览器类型处理，达到比较好的交互效果。

[video组件](https://github.com/lever-Up/blog/blob/master/src/components/video.js)

> 

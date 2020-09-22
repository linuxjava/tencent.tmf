[TOC]



## 1.更新说明

| 内容             | 版本 | 日期       | 作者       |
| ---------------- | ---- | ---------- | ---------- |
| 优化文档结构内容 | v2.0 | 2019-11-27 | robincxiao |
|                  |      |            |            |

## 2.组件简介  

​        H5容器组件提供了良好的外部扩展功能，拥有功能插件化、事件机制、JSAPI 定制和 H5App 推送更新管理能力，主要包含以下功能：
​        加载 H5 页面及按照会话概念管理各个页面，内置丰富的JSAPI，实现页面 push、标题设置等功能，扩展业务需求。接入 H5App 后台，方便管理离线或者在线 H5App。支持自定义网络库、网络通道、键盘等各模块。
​       通过使用H5 容器组件，您将获得优秀的稳定性、强大的离线包能力和完善的客户端能力：
​       H5容器组件的稳定性经过亿级用户考验，崩溃率、应用程序无响应率远低于同类产品，稳定性有保障。通过强大的离线包管理发布平台，可实现多种客户端发布策略，充分满足不同业务方需求，完善的客户端能力可自定义网络库可帮助客户端预防网络劫持，同时有效过滤各类小广告；通过自定义插件，可轻松实现JSAPI。

## 3.开发指南 

### 3.1 Android平台接入

#### 3.1.1 添加SDK

![](..\images\TMF Android组件架构.png)

- **SDK依赖关系**  

H5容器SDK需要用到如下几个包（xxx为包的版本号）：

| **包名或组件**                  | 必选                        | **说明**           |
| ------------------------------- | --------------------------- | ------------------ |
| 移动分析组件                    | N（是否需要上报H5容器事件） | 参考“移动分析”手册 |
| 离线包组件                      | N（是否需要使用离线资源）   | 参考“离线包”手册   |
| TMF-gson-xxx.jar                | Y                           |                    |
| TMF-tbs_static_core_sdk_xxx.aar | Y                           | X5内核             |
| TMF-webview-xxx.aar             | Y                           | webview库          |
| TMF-X5Support-xxx.aar           | Y                           | X5扩展库           |

- **添加SDK并配置工程**  

​	将上面SDK复制到工程的libs目录下， 对工程的build.gradle文件中加入如下代码，确保工程能正确加载aar文件：  

```
repositories {
    flatDir {
        dirs 'libs' //this way we can find the .aar file in libs folder
    }
}

android{
    packagingOptions {
//        不允许AS打包时优化so库，因为X5内核的so库做了MD5的校验，否则会出现加载成功X5内核后，会被删掉
//        这里统一不优化so库，若要优化其他模块so库，也可单独对某个so库配置
        doNotStrip "**/*.so"
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
}
```

#### 3.1.2 使用SDK

##### 3.1.2.1 初始化

- 步骤一

初始化X5内核

```
TMFX5Core.getInstance().init(context);
```

- 步骤二

自定义基于X5的webview组件，参考代码如下：

```
/**
 * 业务侧拓展的WebView，可自行拓展能力
 */
public class X5SampleWebView extends WebView {
    public X5SampleWebView(Context context) {
        super(context);
    }
}
```

注：WebView为X5内核的com.tencent.smtt.sdk.WebView

- 步骤三

创建X5WebViewClient，参考代码如下：

```
/**
 * 业务侧需要的WebViewClient例子，需要可以拓展更多能力
 */
public class X5WebClientDemo extends DefaultTMFX5WebViewClient {
}
```

- 步骤四

创建X5WebChromeClient，参考代码如下：

```
/**
 * 业务需要的WebChromeClient
 */
public class X5WebChromeClientDemo extends DefaultTMFX5WebChromeClient {
    @Override
    public void onProgressChanged(WebView view, int newProgress) {
        
    }
}
```

- 步骤五（可选）

创建H5统计点上报器（需要移动分析模块可用），参考代码如下：

```
public class WebViewReporter implements IWebContainerReporter {

    @Override
    public int trackCustomKVTimeIntervalEvent(Context context, int i, String s, Properties properties) {
        return TMFStatService.trackCustomKVTimeIntervalEvent(context,i,s,properties);
    }

    @Override
    public void trackCustomKVEvent(Context context, String s, Properties properties) {
        TMFStatService.trackCustomKVEvent(context, s, properties);
    }

    @Override
    public void trackBeginPage(String pageName) {
        TMFStatService.trackBeginPage(ContextHolder.sContext, pageName);
    }

    @Override
    public void setCustomUserId(String customUserId) {
        TMFStatService.setCustomUserId(ContextHolder.sContext, customUserId);
    }
}
```

- 步骤六

初始化Webview组件

```
/**
* 获取WebView实例
* @return
*/
private static WebView createX5WebView(Context context){
    X5SampleWebView webView = new X5SampleWebView(context);
    //config webview settings
    WebSettings webSettings = webView.getSettings();
    webSettings.setJavaScriptEnabled(true);
    webView.removeJavascriptInterface("searchBoxJavaBridge_");
    webSettings.setAllowContentAccess(true);
    webSettings.setDatabaseEnabled(true);
    webSettings.setDomStorageEnabled(true);
    webSettings.setAppCacheEnabled(true);
    webSettings.setSavePassword(false);
    webSettings.setSaveFormData(false);
    webSettings.setLoadWithOverviewMode(true);

    return webView;
}
```

- 步骤七

  创建H5容器，参考代码如下

```
ITMFWeb mWebContainer = TMFWeb.withX5(activity)
.setWebView(new DefaultTMFX5WebView(createX5WebView(activity)))//必须设置
.setWebViewClient(new X5WebClientDemo())//必须设置，须是DefaultTMFX5WebViewClient的子类
.setWebChromeClient(new X5WebChromeClientDemo())//必须设置，须是DefaultTMFX5WebChromeClient的子类
.setWebContainerReporter(new WebViewReporter())//设置H5容器上报器（可选）
.build();
```

- 步骤八

将webview添加到页面视图中，参考代码如下：

```
LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(
     LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.MATCH_PARENT);
mContainer.addView((ViewGroup) mWebContainer.getWebViewHolder().getWebView(), params);
```

mContainer为xml页面中的一个布局，如LinearLayout、FraneLayout等。

- 步骤九

在Activity和Fragment的生命周期中管理H5容器，参考代码如下：

```
 @Override
    public void onResume() {
        super.onResume();
        if (mWebContainer != null) {
            mWebContainer.onResume();
        }
    }

    @Override
    public void onStop() {
        super.onStop();
        if (mWebContainer != null) {
            mWebContainer.onStop();
        }
    }

    @Override
    public void onPause() {
        super.onPause();
        if (mWebContainer != null) {
            mWebContainer.onPause();
        }
    }

    @Override
    public void onDestroy() {
    	if (mWebContainer != null) {
    		mWebContainer.onDestroy();
    	}
        super.onDestroy();    
    }
```

- 步骤十

测试是否可以正常加载网页

```
mWebContainer.getWebViewHolder().loadUrl("https:www.baidu.com");
```

##### 3.1.2.2 内置JSAPI

​		H5容器内置了一些JSAPI能力，H5页面监听到容器初始化完成的事件后，就可以通过TMFJSBridge.invoke方法调用这些JSAPI，H5示例代码如下：

```
TMFJSBridge.invoke(apiName, {
    param0 : param0,            // any，参数 0
    param1 : param1,            // any，参数 1
    // ...
    paramN : paramN,            // any，参数 n
}, function (res) {
    console.log({
        ret     : res.ret,      // integer，接入层错误码，有效值：0 表示成功，1 表示接入层失败，2 表示业务层失败，-1 表示取消（部分接口有取消操作）
        errMsg  : res.errMsg,   // string，接入层错误详细信息
    });
});
```

注：这部分代码属于H5代码，内置JSAPI能力参考“**腾讯云移动金融开发平台 用户手册 JSAPI H5**”文档。

##### 3.1.2.3 自定义JSAPI

​		H5容器支持用户自定义JSAPI。

- 创建自定义JSAPI

```
public class JSApiCloseSysKeyboard extends JsApi {
    @Override
    public String method() {
        return "JSApiCloseSysKeyboard";
    }

    @Override
    public void handle(BaseTMFWeb baseTMFWeb, JsCallParam jsCallParam) {
        //实现业务自己的逻辑以及是否返回数据
        KeyboardUtil.closeSysKeyboard(baseTMFWeb.getContext(), baseTMFWeb.getWebViewHolder());
        jsCallParam.mCallback.callback(baseTMFWeb, null);
    }
}
```

（1）自定义JSAPI需要继承JsApi；

（2）实现method，返回JSAPI的名字，在H5调用时需要使用；如果采用一下方式，需要设置自定义JSAPI不被混淆；

```
@Override
public String method() {
	return JSApiCloseSysKeyboard.class.getSimpleName();
}
```

（3）实现handle，H5通过TMFJSBridge.invoke调用时会调用到自定义JSAPI的handle方法；

**注：handle方法中不应该做耗时操作，如果有耗时操作需要在子线程中执行**。

- 添加JSAPI

```
mWebContainer.addJsApi(new JSApiCloseSysKeyboard());
```

##### 3.1.2.4 Native调用H5

在JS中添加事件监听

```
function tmf_sample_onNavigationItemClick() {
    document.addEventListener('onNavigationItemClick', function (e) {
        alert(e.tmf);
    }, false);
}
```

native中触发调用JS中的onNavigationItemClick方法

```
JsonObject refreshObject = new JsonObject();
refreshObject.addProperty("index", 0);
refreshObject.addProperty("tips", "点击了关闭网页");
mWebContainer.nativeCallJs("onNavigationItemClick", refreshObject);
```

#### 3.1.3 支持H5页面打开APP

​		H5页面中如果要打开指定APP或指定APP中的页面，需要在webview中拦截url，参考示例如下：

```
public class SpecialHandle {
    private static final String LOGTAG = "SpecialHandle";

    public static boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (!TextUtils.isEmpty(url) && (url.startsWith("mailto:") || url.startsWith("tel:") || url.startsWith("smsto:"))) {
//            Toast.makeText(view.getContext(), "拦截特殊intent:"+url, Toast.LENGTH_LONG).show();
            return true;
        }
        try {
            Uri uri= Uri.parse(url);
            String scheme = uri.getScheme();
            if ("http".equals(scheme) || "https".equals(scheme)) {
                // 只放开http和https类型请求
            } else {
//                Log.e(LOGTAG, "donot support intent:"+url);
                boolean hasApp = schemeValid(view, url);
                if (hasApp) {
                    Intent action = new Intent(Intent.ACTION_VIEW);
                    action.setData(Uri.parse(url));
                    view.getContext().startActivity(action);

                    return true;
                }
            }
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return false;
    }

    private static boolean schemeValid(WebView view, String url) {
        PackageManager manager = view.getContext().getPackageManager();
        Intent action = new Intent(Intent.ACTION_VIEW);
        action.setData(Uri.parse(url));
        List list = manager.queryIntentActivities(action, PackageManager.GET_RESOLVED_FILTER);
        return list != null && list.size() > 0;
    }
}
```

webview拦截url

```
public class X5WebClientDemo extends DefaultTMFX5WebViewClient{
 	@Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if(SpecialHandle.shouldOverrideUrlLoading(view,url)){
            return true;
        }
        
        return super.shouldOverrideUrlLoading(view, url);
    }
}
```

#### 3.1.3 H5容器与移动分析

​		为了让H5页面也能实现自定义的数据点上报统计，需要添加如下设置：

- 实现ITMFWebSettings接口

```
@Override
public String getUserAgentString() {
        return ((WebSettings) mWebContainer.getWebViewHolder().getWebSettings()).getUserAgentString();
}

@Override
public void setUserAgentString(String userAgent) {
        ((WebSettings) mWebContainer.getWebViewHolder().getWebSettings()).setUserAgentString(userAgent);
}
```

- 移动分析初始化WebSettings

```
//注意设置UserAgent必须在loadUrl之前,否则会导致url被加载两次
TMFStatService.initWebSettings(this);
```

- 拦截url

```
public class X5WebClientDemo extends DefaultTMFX5WebViewClient{
 	@Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (TMFStatService.handleWebViewUrl(view.getContext(), url)) {
            return true;
        }
        
        return super.shouldOverrideUrlLoading(view, url);
    }
}
```

H5容器的webview完成上面的初始化后，H5就可以通过“移动分析”提供的JS方法进行数据上报了。

#### 3.1.4 H5容器与离线包

​		当H5容器加载url时，如果本地离线包中有对应的资源，就可以直接使用本地离线资源加速H5页面的加载。webview需要完成如下初始化才会检查本地离线包中是否有对应资源。

```
public class X5WebClientDemo extends DefaultTMFX5WebViewClient {
	private OfflineManager mOfflineManager;
	
	public void setOfflineManager(OfflineManager manager){
        this.mOfflineManager = manager;
    }
    
    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
        android.webkit.WebResourceResponse response = null;
        if(mOfflineManager != null) { // 用离线包拦截，拦截到了直接返回
            response = mOfflineManager.shouldInterceptRequest(view.getContext(), url);
        }
        
        if (response != null) {
            ToastUtil.showToast("拦截并使用了离线资源: " + url);
            WebResourceResponse x5Response = new WebResourceResponse();
            x5Response.setMimeType(response.getMimeType());
            x5Response.setEncoding(response.getEncoding());
            x5Response.setData(response.getData());
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                x5Response.setResponseHeaders(response.getResponseHeaders());
                x5Response.setStatusCodeAndReasonPhrase(response.getStatusCode(),                                                                     response.getReasonPhrase());
            }
            return x5Response;
        }
        
        // 没有匹配的资源，走线上
        return null;
	}
}
```

### 3.2 API描述

#### 3.2.1 Native调用H5

Native可以通过下面接口调用H5中实现的JS函数

```
public void nativeCallJs(String functionName, JsonObject jsonObject)
```

| 参数         | 类型       | 描述             | 必选 |
| ------------ | ---------- | ---------------- | ---- |
| functionName | String     | JS端定义的事件名 | Y    |
| jsonObject   | JsonObject | 传递的json数据   | N    |

### 3.3 H5容器传递图片

​		H5容器在Native与H5页面进行通信时不支持>2M的数据传输传输，且Native与H5页面传递大量数据效率低下；针对于Native与H5之间图片传输的场景给出如下解决方案：

- 场景一：H5需要通过相机或从本地图库中获取图片，然后上传到服务端；

​		解决方案：

（1）H5通过自定义JSAPI调起相机或本地图库，用户在Native端拍照或选择照片后获取本地图片路径，然后将本地图片路径转换为对应的本地url，示例如下

- 场景二：H5调用native，native通过网络请求获取了图片数据， 图片数据需要传递到H5页面展示；

​		解决方案：

（1）native通过网络请求获取了图片数据后，将其存储在本地，并获取本地图片路径；

（2）将本地图片路径转换为对应的本地url，并将该url返回给H5，H5通过加载本地url展示数据，示例如下；

```
"file:///storage/emulated/0/test/1.jpg"
```

（3）在适当时机，需要清除保存在本地的图片；

### 3.4 调试日志

- 日志开关

```
TMFWebConfig.DEBUG = true;
```

- 日志查看

使用 “TMFJSBridge”关键字查看日志；






















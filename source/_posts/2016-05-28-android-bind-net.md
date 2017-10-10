title: Android多网络机制浅析
date: 2016-05-28 14:42:58
categories:
- Android
tags:
- Multi net
---

Android从4.2版本开始，逐步支持了多网络功能。相关的api能够让开发者选择想要的网络设备访问，并且各个设备之间的切换和绑定也越来越方便。
<!--more-->

## 判断网络连通性机制

从Android4.2.2开始，引入了一个叫“captive portal” detection的机制，用来判断当前网络是否连接上互联网，是不是需要身份验证的公共网络。以5.0版本源码中的代码为例：

https://android.googlesource.com/platform/frameworks/base/+/android-5.0.0_r7/services/core/java/com/android/server/connectivity/NetworkMonitor.java

在NetworkMonitor类中的isCaptivePortal方法：

	/**
	     * Do a URL fetch on a known server to see if we get the data we expect.
	     * Returns HTTP response code.
	     */
	    private int isCaptivePortal() {
	        if (!mIsCaptivePortalCheckEnabled) return 204;
	        HttpURLConnection urlConnection = null;
	        int httpResponseCode = 599;
	        try {
	            URL url = new URL("http", mServer, "/generate_204");
	            if (DBG) {
	                log("Checking " + url.toString() + " on " +
	                        mNetworkAgentInfo.networkInfo.getExtraInfo());
	            }
	            urlConnection = (HttpURLConnection) mNetworkAgentInfo.network.openConnection(url);
	            urlConnection.setInstanceFollowRedirects(false);
	            urlConnection.setConnectTimeout(SOCKET_TIMEOUT_MS);
	            urlConnection.setReadTimeout(SOCKET_TIMEOUT_MS);
	            urlConnection.setUseCaches(false);
	            // Time how long it takes to get a response to our request
	            long requestTimestamp = SystemClock.elapsedRealtime();
	            urlConnection.getInputStream();
	            // Time how long it takes to get a response to our request
	            long responseTimestamp = SystemClock.elapsedRealtime();
	            httpResponseCode = urlConnection.getResponseCode();
	            if (DBG) {
	                log("isCaptivePortal: ret=" + httpResponseCode +
	                        " headers=" + urlConnection.getHeaderFields());
	            }
	            // NOTE: We may want to consider an "HTTP/1.0 204" response to be a captive
	            // portal.  The only example of this seen so far was a captive portal.  For
	            // the time being go with prior behavior of assuming it's not a captive
	            // portal.  If it is considered a captive portal, a different sign-in URL
	            // is needed (i.e. can't browse a 204).  This could be the result of an HTTP
	            // proxy server.
	            // Consider 200 response with "Content-length=0" to not be a captive portal.
	            // There's no point in considering this a captive portal as the user cannot
	            // sign-in to an empty page.  Probably the result of a broken transparent proxy.
	            // See http://b/9972012.
	            if (httpResponseCode == 200 && urlConnection.getContentLength() == 0) {
	                if (DBG) log("Empty 200 response interpreted as 204 response.");
	                httpResponseCode = 204;
	            }
	            sendNetworkConditionsBroadcast(true /* response received */, httpResponseCode == 204,
	                    requestTimestamp, responseTimestamp);
	        } catch (IOException e) {
	            if (DBG) log("Probably not a portal: exception " + e);
	            if (httpResponseCode == 599) {
	                // TODO: Ping gateway and DNS server and log results.
	            }
	        } finally {
	            if (urlConnection != null) {
	                urlConnection.disconnect();
	            }
	        }
	        return httpResponseCode;
	    }


简单的说，原理就是访问google的clients3.google.com/generate_204地址，当返回204代码，或者是200并且内容ContentLength是0时，判断网络已连通。否则就是未连上互联网，或者需要身份验证的公共网络。为什么返回200并且ContentLength为0时也认为是连上互联网了呢？因为需要身份验证的系统返回长度不可能是0，可能是因为代理的缘故导致状态码错误。

使用Android 5.0版本以上原生系统的同学会发现，状态栏上wifi或者cell的图标上会有个叹号。这是因为谷歌被墙，http的response code自然就不会是204了。

我们看到isCaptivePortal 方法是传入一个参数InetAddress server的。要想在国内使用这个feature，可以将谷歌的地址替换为国内网友架的服务或者自己建一个服务例如：https://github.com/HorseLuke/drafts/blob/master/sinaapp_generate_204/README.md。

或者使用现成的v2ex搭建的，只要一个命令即可：

>adb shell "settings put global captive_portal_https_url https://captive.v2ex.co/generate_204"

也可以完全禁止掉这个检测：
>adb shell "settings put global captive_portal_detection_enabled 0"

但这样会有一个问题，就是如果连接上一个需要网页验证的wifi，就没有办法自动跳到登陆界面了。

## 多网络连接机制

一直以来Android系统的访问网络的类型都是不可选的，连接WiFi走WiFi，否则走Cellular。但从5.0版本开始引入了多网络连接机制，引用：

>Android 5.0 provides new multi-networking APIs that let your app dynamically scan for available networks with specific capabilities, and establish a connection to them. This functionality is useful when your app requires a specialized network, such as an SUPL, MMS, or carrier-billing network, or if you want to send data using a particular type of transport protocol.

在api21中新加入了方法

>ConnectivityManager.setProcessDefaultNetwork

可以将进程绑定到特定的网络，这样即使是wifi打开的情况下也可以使用Cellular访问网络了。但是当WiFi连接时，不管WiFi是否联网，系统默认依然是会选择走WiFi，只有手动绑定app进程才能切到Cellular网络。

https://developer.android.com/about/versions/android-5.0.html#Wireless

## 自动切换网络机制

到了Android6.0，在网络方面又有如下变化。如change note中所说，在之前版本的系统中，当连接到WiFi时，其他类型的网络就会断开；而在6.0中，其他网络不会断开，虽然会优先从WiFi访问，但当检测到连接的WiFi没有联网而其他网络（例如Cellular）是联网的情况下，所有数据的访问会走到Cellular网络。引用：

>This release introduces the following behavior changes to the Wi-Fi and networking APIs.

>- Your apps can now change the state of WifiConfiguration objects only if you created these objects. You are not permitted to modify or deleteWifiConfiguration objects created by the user or by other apps.
- Previously, if an app forced the device to connect to a specific Wi-Fi network by using enableNetwork() with the disableAllOthers=true setting, the device disconnected from other networks such as cellular data. In This release, the device no longer disconnects from such other networks. If your app’s targetSdkVersion is “20” or lower, it is pinned to the selected Wi-Fi network. If your app’s targetSdkVersion is “21” or higher, use the multinetwork APIs (such as openConnection(), bindSocket(), and the new bindProcessToNetwork() method) to ensure that its network traffic is sent on the selected network.

https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-network

对大部分开发者来说，这个特性似乎没什么用。在大部分情况下，app不需要关心系统网络如何切换，只要能够成功访问并且得到当前访问的类型就够了。

但是对某些特殊的应用场景，有了这个特性就惨了。例如：你的app需要访问到本地的网络服务（例如一台没有接入互联网的路由器），如果cellular是关闭的，那可以正常访问；而如果cellular是打开的，所有的数据都默认走到cellular，就无法访问本地的服务了。

从这个例子看6.0的系统是有些傻，明明可以做到两个网络同时连通，但却很“智能”地选择了一个连接到互联网的网络去访问。当然解决这个问题并不难，使用系统提供的bindsocket和bindprocesstonetwork就够了。

## 多网络同时访问

正如前一节所说，要同时访问多个网络设备，需要拿到网络对应的Network，并且跟访问的socket做绑定。以网络框架Okhttp和主流图片加载框架picasso与glide为例，用以下几个步骤便可实现网络绑定：

### 获取Network

参照如下的代码片段，首先构造一个NetworkRequest.Builder，包含wifi但不包含网络访问；其次为ConnectivityManager注册监听；当onAvailable回调获取到Network时，记录下当前连接的wifi。


	final ConnectivityManager connManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
	NetworkRequest.Builder request = new NetworkRequest.Builder()
                    .addTransportType(NetworkCapabilities.TRANSPORT_WIFI)
                    .removeCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET);
   	connManager.registerNetworkCallback(request.build(), new ConnectivityManager.NetworkCallback() {
                @Override
                public void onAvailable(Network network) {
                    NetworkUtil.setNetwork(BindNetActivity.this, network);
                    L.d("bind network " + network.toString());
                }

                @Override
                public void onLost(Network network) {
                    NetworkUtil.setNetwork(BindNetActivity.this, null);
                    try {
                        connManager.unregisterNetworkCallback(this);
                    } catch (SecurityException e) {
                        L.d("Failed to unregister network callback");
                    }
                }
            });


### 绑定网络

在setNetwork方法里，做了一下几件事。首先记录下Network，然后更新相关httpclient并绑定网络，然后更新图片加载并添加绑定网络的注册。

	public static void setNetwork(Context context, Network network) {
        L.d("init network" + network);
        NetworkUtil.network = network;
        BindedHttpClient.getInstance().updateClient(network);
        ImageLoader.initGlide(context);
    }            

BindHttpClient 里的关键代码如下，可以看到，在updateClient方法中，使用OkHttpClient.Builder的socketFactory方法将创建连接的socketFactory 指定为Network的socketFactory。这样所有通过OkHttpClient的访问都会走到指定的Network了。

	public OkHttpClient client;
	public void updateClient(Network network) {
        OkHttpClient.Builder builder = new OkHttpClient().newBuilder();
        builder.connectTimeout(10, TimeUnit.SECONDS)
                .writeTimeout(30, TimeUnit.SECONDS)
                .readTimeout(30, TimeUnit.SECONDS)
                .connectionPool(new ConnectionPool(0, 5, TimeUnit.MINUTES));
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M && network != null) {
            builder.socketFactory(network.getSocketFactory());
        }
        client = builder.build();
    }    

ImageLoader里的关键代码如下，glide支持图片加载的自定义注册。简单的说，就是可以将加载的地址包装成modelClass，指定加载后的数据为resourceClass，以及加载工厂factory。

	public static void initGlide(Context context) {
        Glide.get(context).register(GlideCameraUrl.class, InputStream.class, new LocalLoaderFactory());
    }    

GlideCustomUrl里面没什么，就是继承了GlideUrl，方便区分是普通的图片请求，还是特定网络的图片请求。

    public class GlideCustomUrl extends GlideUrl {

	    public GlideCustomUrl(URL url) {
	        super(url);
	    }

	    public GlideCustomUrl(String url) {
	        super(url);
	    }

	    public GlideCustomUrl(URL url, Headers headers) {
	        super(url, headers);
	    }

	    public GlideCustomUrl(String url, Headers headers) {
	        super(url, headers);
	    }
	}

LocalLoaderFactory里的build方法返回了自定义的ModelLoader OkHttpUrlLoader，传入之前绑定过网络的httpclient作为网络请求，这样就可以绑定到特定网络了。


	public class LocalLoaderFactory implements ModelLoaderFactory<GlideCustomUrl, InputStream> {

    	@Override
    	public ModelLoader<GlideCustomUrl, InputStream> build(Context context, GenericLoaderFactory factories) {
        	return new OkHttpUrlLoader(BindedHttpClient.getInstance().client);
    	}

    	@Override
    	public void teardown() {
    	}
	}

在OkHttpUrlLoader里又需要自定义一个DataFetcher，在OkHttpStreamFetcher这里面才是真正的请求网络数据的部分。

	@Override
    public DataFetcher<InputStream> getResourceFetcher(GlideCustomUrl model, int width, int height) {
        return new OkHttpStreamFetcher(client, model);
    }

### 对c层socket的绑定

以上写的都是在java层创建http的绑定，对于有些应用场景需要在C层访问网络，又如何绑定呢？

参照如下代码，linux系统中创建socket会分配对应的fileDescriptor即文件描述符，这个是一个IO的唯一标识。只要创建完socket之后通过jni调用此java方法，将fileDescriptor绑定到相关network中就可以了。

	@TargetApi(Build.VERSION_CODES.M)
    public static void bindSocketToNetwork(int socketfd) {
        L.d("start bindSocketToNetwork");
        if (network != null && Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            FileDescriptor fileDescriptor = new FileDescriptor();
            try {
                Field field = FileDescriptor.class.getDeclaredField("descriptor");
                field.setAccessible(true);
                field.setInt(fileDescriptor, socketfd);
                // fileDescriptor.sync();

                network.bindSocket(fileDescriptor);
	//                bindSocket(socketfd, netId);
                L.d("bindSocketToNetwork success: network" + network + "+socketfd" + socketfd);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

## 相关源码
本文涉及到的相关代码可以参考[BindSocketDemo](https://github.com/mqstack/BindSocketDemo)  

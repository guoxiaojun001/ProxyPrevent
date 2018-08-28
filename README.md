# ProxyPrevent
如何防止charles 以及Fiddler  抓包

金融类的App，因为有些交易金额特别大，对应用的安全性也非常高，如何防止被第三方抓包，就需要对App内部做一些设置。

具体应用到以下步鄹就可以

1.判断当前系统是否挂代理

获取当前系统是否设置代理，可以根据不同的 Api Level，分别通过 System.getProperty() 和 android.net.proxy.getXxx() 方法获取到。

  private fun checkWifiProxy(): Boolean {
        val IS_ICS_OR_LATER = Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH
        val proxyAddress: String?
        val proxyPort: Int?
        if (IS_ICS_OR_LATER) {
            proxyAddress = System.getProperty("http.proxyHost")
            val portStr = System.getProperty("http.proxyPort")
            proxyPort = Integer.parseInt(portStr ?: "-1")
        } else {
            proxyAddress = android.net.Proxy.getHost(this)
            proxyPort = android.net.Proxy.getPort(this)
        }
        Log.i("cxmyDev","proxyAddress : ${proxyAddress}, prot : ${proxyPort}")
        return !TextUtils.isEmpty(proxyAddress) && proxyPort != -1
    }
    
    通过 Log，就可以看到当前设备，在 WiFi 中，挂的代理的 IP 和 Port 了。

2，拒绝发请求。

既然知道代理的IP和Port的就可以做出判断，直接进行拦截，同时进行一定的提示。

3，设置不使用代理

Fiddler 和 Charles 这类抓包工具，本质上就是利用中间人攻击的方式，通过这个中间人获取到通信的数据。

而利用这些工具抓包的前提，都是在设备上，设置代理，通常我们直接在 WiFi 连接页面，设置代理即可。

而对于一些常用的网络库，其实是提供了我们设置的代理的接口，我们只需要将其设置成无代理的模式，它就不会去应用系统默认的代理了。

就拿比较常用的 OkHttp 来举例，在初始化的时候，就可以通过 proxy() 方法，为 OkHttp 设置一个代理。


    var httpBuilder = OkHttpClient.Builder()
                .addInterceptor(defaultInterceptor())
                .connectTimeout(DEFAULT_TIMEOUT, TimeUnit.MILLISECONDS)
                .writeTimeout(DEFAULT_TIMEOUT, TimeUnit.MILLISECONDS)
                .readTimeout(DEFAULT_TIMEOUT, TimeUnit.MILLISECONDS)
                .proxy(Proxy.NO_PROXY)
                
例如这里，我们对其设置为不使用代理的模式，它就不会从系统中，读取代理信息，进行网络请求。而是会忽略掉它，直接发送网络请求。以这样的方式，就可以阻止第三方使用 Fiddler 或 Charles 进行抓包。

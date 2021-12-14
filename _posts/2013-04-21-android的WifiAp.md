---
title: "android的WifiAp"
layout: post
comments: true
---
````
看了WifiManager的源码。 关于AP这些 都是隐藏的。
 /**
     * Broadcast intent action indicating that Wi-Fi AP has been enabled, disabled,
     * enabling, disabling, or failed.
     *这是Ap状态改变的广播
     * @hide
     */
    public static final String WIFI_AP_STATE_CHANGED_ACTION ="android.net.wifi.WIFI_AP_STATE_CHANGED";
 /***********************************************这是源码一些状态的定义************************************/
    /**
     * Wi-Fi AP is currently being disabled. The state will change to
     * {@link #WIFI_AP_STATE_DISABLED} if it finishes successfully.
     *
     * @see #WIFI_AP_STATE_CHANGED_ACTION
     * @see #getWifiApState()
     *
     * @hide
     */
    public static final int WIFI_AP_STATE_DISABLING = 10;
    /**
     * Wi-Fi AP is disabled.
     *
     * @see #WIFI_AP_STATE_CHANGED_ACTION
     * @see #getWifiState()
     *
     * @hide
     */
    public static final int WIFI_AP_STATE_DISABLED = 11;
    /**
     * Wi-Fi AP is currently being enabled. The state will change to
     * {@link #WIFI_AP_STATE_ENABLED} if it finishes successfully.
     *
     * @see #WIFI_AP_STATE_CHANGED_ACTION
     * @see #getWifiApState()
     *
     * @hide
     */
    public static final int WIFI_AP_STATE_ENABLING = 12;
    /**
     * Wi-Fi AP is enabled.
     *
     * @see #WIFI_AP_STATE_CHANGED_ACTION
     * @see #getWifiApState()
     *
     * @hide
     */
    public static final int WIFI_AP_STATE_ENABLED = 13;
    /**
     * Wi-Fi AP is in a failed state. This state will occur when an error occurs during
     * enabling or disabling
     *
     * @see #WIFI_AP_STATE_CHANGED_ACTION
     * @see #getWifiApState()
     *
     * @hide
     */
    public static final int WIFI_AP_STATE_FAILED = 14;


然后咱们来实际一下。 注册广播 监听Ap的变化。
04-19 11:12:34.297: D/wifiAP(22701): 1
04-19 11:14:29.837: D/wifiAP(22701): 2
04-19 11:14:34.437: D/wifiAP(22701): 3
04-19 11:15:00.307: D/wifiAP(22701): 0
04-19 11:15:01.587: D/wifiAP(22701): 1
我擦！ 获取Ap状态 竟然不是 上面定义的值。
好吧 咱们猜测一下。
1：表示 ap关闭
2：表示 ap正在打开
3：表示 ap打开了
0：表示 ap正在关闭


如果要调用函数，用反射
````


---
layout: post
title:  "android动态配置Wifi信息"
date:   2013-02-27 20:22:14 +0800
categories: jekyll update
---
#### 多人在开发应用的时候会代码连接Wifi的功能。 
很多人认为 输入SSID和Password调用API 就可以去连接了啊，但是没有这种API，这种连接方式思路也是错的。
#####这才是正确的方法

打开到wifi连接页面，我们会发现最后一行有一个  add NewNetwork 的功能。 如果你输入已经覆盖Wifi的 SSID，加密方式，和Password。 你会发现竟然连接到这个wifi了。 代码连接wifi 用的就是这种方式。废话不说 直接上源码.

````
我们找到 android.net.wifi.WifiConfiguration.java  看看如何配置Wifi信息。
/**************************************SSID**********************/
 /**
     * The network's SSID. Can either be an ASCII string,
     * which must be enclosed in double quotation marks
     * (e.g., {@code "MyNetwork"}, or a string of
     * hex digits,which are not enclosed in quotes
     * (e.g., {@code 01a243f405}).
     */
    public String SSID;  //请看注释： 如果是 ASCII 码 SSID必须要 " " 包裹起来

/**************************************BSSID**********************/
/**
     * When set, this network configuration entry should only be used when
     * associating with the AP having the specified BSSID. The value is
     * a string in the format of an Ethernet MAC address, e.g.,
     * <code>XX:XX:XX:XX:XX:XX</code> where each <code>X</code> is a hex digit.
     */
    public String BSSID; //但是我发现 在配置信息的时候，BSSID 是不能赋值的

 /******************************************* preSharedKey*********************/
/**
     * Pre-shared key for use with WPA-PSK.
     * <p/>
     * When the value of this key is read, the actual key is
     * not returned, just a "*" if the key has a value, or the null
     * string otherwise.
     */
    public String preSharedKey; //这是WPA模式 加密的，密码赋值的地方 

/*************************************wepKeys******************/
/**
     * Up to four WEP keys. Either an ASCII string enclosed in double
     * quotation marks (e.g., {@code "abcdef"} or a string
     * of hex digits (e.g., {@code 0102030405}).
     * <p/>
     * When the value of one of these keys is read, the actual key is
     * not returned, just a "*" if the key has a value, or the null
     * string otherwise.
     */
    public String[] wepKeys;// 这是WEP加密 密码赋值的地方，必须考虑ASCII

//***************基于属性太多 我只是把一些重要的属性列出来，其他属性 自己去看

 下面开始配置wifi信息。
/**************************web 配置信息****************************/
   config.SSID = "\"".concat(SSID).concat("\"");
   config.status = WifiConfiguration.Status.DISABLED;
   config.priority = 40;
   config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
   config.allowedProtocols.set(WifiConfiguration.Protocol.RSN);
   config.allowedProtocols.set(WifiConfiguration.Protocol.WPA);
   config.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.OPEN);
   config.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.SHARED);
   config.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.CCMP);
   config.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.TKIP);
   config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.WEP40);
   config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.WEP104); 
   if (isHexWepKey(Password)) 
    config.wepKeys[0] = Password;
   else 
     config.wepKeys[0] = "\"".concat(Password).concat("\"");
     config.wepTxKeyIndex = 0;

/***************************wpa配置信息*****************************/ 
   config.SSID = "\"" + SSID + "\"";
   config.hiddenSSID = false;
   config.status = WifiConfiguration.Status.ENABLED;
   config.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.OPEN);
   config.allowedProtocols.set(WifiConfiguration.Protocol.WPA);
   config.allowedProtocols.set(WifiConfiguration.Protocol.RSN); // For WPA2
   config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.WPA_PSK);
   config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.WPA_EAP);
   config.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.TKIP);
   config.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.CCMP);
   config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.CCMP);
   config.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.TKIP);
   config.preSharedKey = "\"" + Password + "\"";
   config.priority =1; 

/****************************无密码的配置***********************/
   config.wepKeys[0] = "";
   config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
   config.wepTxKeyIndex = 0; 

  wifiManager.addNetwork(wifiConfig);  //添加配置信息,返回NetworkId， 如果是-1 则配置失败
 wifiManager.saveConfiguration();//保存信息
wifiManager.enableNetwork（NetWorkId,true）；//连接返回的NetWordID，并且断开其他的Wifi连接。 
````


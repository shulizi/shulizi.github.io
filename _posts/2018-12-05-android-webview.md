---
layout: post
title:  "android开发篇：内置浏览器上传文件"
date:   2016-08-12
categories: android
tags: android baidumap
---

* content
{:toc}

android内置浏览器虽然使用起来很方便，但一些功能还是要设置之后才能使用。比如上传文件在手机自带浏览器中就能轻松解决，但是在android内置浏览器中却不是这样。





## MainActivity.java
这里除了加载网页外，还阻止了打开手机自带浏览器，允许读取手机文件夹等等。
``` java
package com.example.qrcodemaster;

import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.webkit.ValueCallback;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;

public class MainActivity extends Activity {
private WebView webView;
private ValueCallback<Uri> mUploadMessage;
private final static int FILECHOOSER_RESULTCODE = 1;
private String WEB_URL = "http://121.42.159.177/Submit/";

@SuppressLint({ "SetJavaScriptEnabled", "JavascriptInterface" })
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_browse_news);
webView = (WebView) findViewById(R.id.wv_news);
String url = WEB_URL;
webView.setWebViewClient(new MyWebViewClient());

webView.getSettings().setJavaScriptEnabled(true);
webView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
webView.setWebChromeClient(new MyWebChromeClient());
webView.loadUrl(url);

}

@Override
protected void onActivityResult(int requestCode, int resultCode,
Intent intent) {
if (requestCode == FILECHOOSER_RESULTCODE) {
if (null == mUploadMessage)
return;
Uri result = intent == null || resultCode != RESULT_OK ? null
: intent.getData();
mUploadMessage.onReceiveValue(result);
mUploadMessage = null;
}
}

public class MyWebViewClient extends WebViewClient {
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
view.loadUrl(url);
return true;
}
}

public class MyWebChromeClient extends WebChromeClient {
// For Android 3.0-
public void openFileChooser(ValueCallback<Uri> uploadMsg) {
mUploadMessage = uploadMsg;
openFileChooser(uploadMsg, "");
}

// For Android 3.0+
public void openFileChooser(ValueCallback<Uri> uploadMsg,
String acceptType) {
mUploadMessage = uploadMsg;
Intent i = new Intent(Intent.ACTION_GET_CONTENT);
i.addCategory(Intent.CATEGORY_OPENABLE);
i.setType("*/*");
startActivityForResult(Intent.createChooser(i, "File Browser"),
FILECHOOSER_RESULTCODE);
}

// For Android 4.1
public void openFileChooser(ValueCallback<Uri> uploadMsg,
String acceptType, String capture) {
mUploadMessage = uploadMsg;

openFileChooser(uploadMsg, acceptType);
}
}

}
```

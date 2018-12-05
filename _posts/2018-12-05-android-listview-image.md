---
layout: post
title:  "android开发篇：新闻客户端listview图片加载乱序的处理"
date:   2016-08-11
categories: android
tags: android baidumap
---

* content
{:toc}

此前我们做了一个android开发篇：新闻客户端的实现，虽然效果不错，但是有一个致命的问题，图片加载会乱序，为了解决这个问题，我们会接触到android的缓存处理机制。





## android缓存处理
android加载网页或是下载数据或是打开本地图片或是打开Activity时都会将数据先添加到缓存里，这样下一次加载速度就会加快好几倍。而缓存的处理是通过键值对的查找进行的。我们只要调用响应接口，就可以很简单地实现android的缓存处理。

httpUtil.java
``` java
package com.lizi.shanghaisandtmuseums.utils;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.ref.SoftReference;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.HashMap;
import java.util.Map;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.AsyncTask;
import android.os.Handler;
import android.os.Message;
import android.widget.ImageView;

public class HttpUtil {
private static Map<String, SoftReference<Bitmap>> caches = new HashMap<String, SoftReference<Bitmap>>();;

public static void getNewsJSON(final String url, final Handler handler) {
new Thread(new Runnable() {

@Override
public void run() {
HttpURLConnection conn;
InputStream is;

try {
conn = (HttpURLConnection) new URL(url).openConnection();
conn.setRequestMethod("GET");
is = conn.getInputStream();
BufferedReader br = new BufferedReader(
new InputStreamReader(is));
String line = "";

StringBuilder sb = new StringBuilder();
while ((line = br.readLine()) != null) {
sb.append(line);
}
Message msg = new Message();
msg.obj = sb.toString();
handler.sendMessage(msg);
is.close();
} catch (MalformedURLException e) {
e.printStackTrace();
} catch (IOException e) {
e.printStackTrace();
}
}
}).start();
}

public static void setPicBitmap(final ImageView ivPic, final String pic_url) {

// 判断是否有缓存
if (caches.containsKey(pic_url)) {
SoftReference<Bitmap> rf = caches.get(pic_url);
Bitmap bm = rf.get();
// 如果缓存里面图片已经释放，则去掉软链接
if (bm == null) {
caches.remove(pic_url);
} else {
ivPic.setImageBitmap(bm);
return;
}

} else {
new AsyncTask<String, Void, Bitmap>() {

@Override
protected Bitmap doInBackground(String... pic_url) {
HttpURLConnection conn;
InputStream is;
Bitmap bitmap = null;
try {
conn = (HttpURLConnection) new URL(pic_url[0])
.openConnection();
conn.connect();
is = conn.getInputStream();
bitmap = BitmapFactory.decodeStream(is);
is.close();
} catch (Exception e) {
e.printStackTrace();
}
return bitmap;
}

@Override
protected void onPostExecute(Bitmap result) {
// System.out.println(pic_url+"  "+ivPic.getTag());
if (ivPic.getTag().equals(pic_url)) {

// 添加图片到缓存
caches.put(pic_url, new SoftReference<Bitmap>(result));
ivPic.setImageBitmap(result);
super.onPostExecute(result);
}

}
}.execute(pic_url);
}
}
}
```

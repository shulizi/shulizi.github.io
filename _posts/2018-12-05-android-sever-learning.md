---
layout: post
title:  "android开发篇：客户端连接服务器学习笔记"
date:   2016-6-25 
categories: android
tags: android load_and_register
---

* content
{:toc}

学习 极客学院 的《Android项目开发实战-秘密APP》所做笔记，关于POST,GET方法，接口方法，服务器测试编写等。




## 1.运用接口实现回调函数

### 1）定义方法和接口。
``` java
public class NetConnection{
public NetConnection(SuccessCallback successCallback,FailCallback failCallback){......}
}
public static interface SuccessCallback successCallback{
void onSuccess();
}
public static interface FailCallback failCallback{
void onFail();
}
```
### 2)访问方法，并实现接口。
``` java
new NetConnection(new NetConnection.SuccessCallback(){
@Override
public void onSuccess(){......}
},new NetConnection.FailCallback(){
@Override
public void onFail(){......}
});
```
## 2.线程类AsyncTask的使用

### 1）传递参数若为空，可以是Void 
### 2）覆写doInBackground方法和 onPostExecute方法，替代Thread的run和handler。

## 3.post和get方式访问服务器

### 1）post方式访问
``` java
URLConnection uc=new URL(url).openConnection();
uc.setDoOutput(true);
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(uc.getOutputStream(),"utf-8");
bw.write(paramsStr);
bw.flush();
```
### 2)get方式访问
``` java
URLConnection uc=new URL(url+"?"+paramsStr).openConnection();
```
### 3)post和get方式读
``` java
BufferedReader br = new BufferedReader(new InputStreamReader(uc.getInputStream(),"utf-8");
String line=null;
StringBuffer sb = new StringBuffer();
while((line=br.readline())!=null){
sb.append(line);
}
```
### 4)URLConnection和HttpURLConnection区别

就像handler和AsyncTack的区别，URLConnection只是实现了简单的连接功能，HttpURLConnection封装了更多功能，比如下载图片文件等。

## 4.进度条（圈）
``` java
ProgressDialog pd = ProgressDialog.show(this,getResources().getString(R.string.title),getResources().getString(R.string.content))
pd.dismiss();
```
## 5.创建服务器测试编写

### 1）新建.jsp文档
``` javascript
<%
out.clear();
String action=request.getParameter("action");
if(action!=null){
if(action.equals("send_pass")){
out.print("Success");
}
}else{
out.print("No action");
}

%>
```

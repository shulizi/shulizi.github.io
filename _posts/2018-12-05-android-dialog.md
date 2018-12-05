---
layout: post
title:  "android开发篇：dialog全屏，宽度全屏和动画"
date:   2016-08-11
categories: android
tags: android dialog
---

* content
{:toc}

 dialog 是一个弹出框，一般用作提示，当然也可以做一个弹出的界面，现在app开发中更多的是做一个从底部弹出的选择列表。在开发过程中有几个难题，我们需要一一解决。






## dialog如何全屏
dialog全屏在java代码里，xml修改都不顶用，不管是setPadding=0还是LayoutParams.FillParent都会在屏幕边缘出现一点空隙。网上的方法都是千篇一律。最后找到两种方法，两种方法都要用到style。
### 方法一：style里面设置全屏
java代码
``` java
Dialog dialog=new Dialog(this,R.style.dialog_fullscreen);
dialog.setContentView(R.layout.main);
dialog.show();

```
### style.xml
``` xml
<style name="dialog_fullscreen">
<item name="android:windowFullscreen">true</item>
<!--
<item name="android:windowBackground">0xff0000ff</item> 
<item name="android:windowNoTitle">true</item>
-->
</style>
```
### 方法二：代码里面修改全屏
### java代码
``` java
<Dialog dialog=new Dialog(this,R.style.dialog);
dialog.setContentView(R.layout.main);
dialog.show();
Window window = dialog.getWindow();//必须在show之后设置dialog的宽高
WindowManager.LayoutParams layoutParams = window
.getAttributes();
layoutParams.width = LayoutParams.MATCH_PARENT;
layoutParams.height = LayoutParams.MATCH_PARENT;
window.setAttributes(layoutParams);


```
### style.xml


``` xml
<style name="dialog">
<!--
什么都不需要写
<item name="android:windowBackground">0x00000000</item> 
<item name="android:windowNoTitle">true</item>
-->
</style>
```
## dialog如何宽度全屏
dialog宽度全屏也得用到style，然后在代码里面将高度设置为wrap_content就可以了。同时我们可以设置dialog出现的位置。

### java代码
``` java
Dialog dialog = new Dialog(context, R.style.dialog);
dialog.setContentView(R.layout.main);
WindowManager.LayoutParams layoutParams = window.getAttributes();
layoutParams.width = LayoutParams.MATCH_PARENT;
layoutParams.height = LayoutParams.WRAP_CONTENT;
window.setGravity(Gravity.BOTTOM);//设置为dialog出现在屏幕底部
window.setAttributes(layoutParams);


```
### style.xml


``` xml
<style name="dialog">
<!--
什么都不需要写
<item name="android:windowBackground">0x00000000</item> 
<item name="android:windowNoTitle">true</item>
-->
</style>
```
## dialog弹出动画效果
dialog的出现和退出可以做一个动画，同样需要用到style，通过style调用drawable动画实现。
``` java
Dialog dialog = new Dialog(context, R.style.dialog);
Window window = dialog.getWindow();
window.setWindowAnimations(R.style.dialog_animation); // 添加动画
dialog.setContentView(R.layout.main);
//dialog.setCanceledOnTouchOutside(true);
//dialog.setTitle("标题");
dialog.show();
window.setGravity(Gravity.BOTTOM);
/*
WindowManager.LayoutParams layoutParams = window
.getAttributes();
layoutParams.width = LayoutParams.MATCH_PARENT;
layoutParams.height = LayoutParams.WRAP_CONTENT;
window.setAttributes(layoutParams);
*/


```
### style.xml


``` xml
<style name="dialog" >
<!-- <item name="android:windowBackground">0x00000000</item> -->
</style>

<style name="dialog_animation" parent="android:Animation">
<item name="@android:windowEnterAnimation">@drawable/dialog_enter</item>
<item name="@android:windowExitAnimation">@drawable/dialog_exit</item>
</style>
```
### dialog_enter.xml
``` xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
<translate
android:duration="600"
android:fromYDelta="100%p" />
</set>
```
### bdialog_exit.xml
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
<translate
android:duration="600"
android:toYDelta="100%p" />
</set>
```

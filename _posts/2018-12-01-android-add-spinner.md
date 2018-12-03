---
layout: post
title:  "android开发篇：ActionBar添加按钮，修改背景，设置标题和返回，添加spinner"
date:   2018-12-01 22:14:54
categories: android
tags: android actionbar spinner
---

* content
{:toc}

ActionBar的开发是App开发中比较重要的一环，虽然自定义actionbar既灵活又简单，但是不符合规范，适配性便不是最强的，所以了解好ActionBar依然是相当重要。





## ActionBar添加按钮

ActionBar修改按钮只需要修改menu就可以了，而不用考虑点击效果，图片尺寸，可以说这是它相较于自定义ActionBar的优势之一。然后在代码里面添加点击事件就可以了。

### main.xml
``` xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
<item   
android:id="@+id/action_search"   
android:icon="@drawable/search"  
android:title="@string/action_search"  
android:showAsAction="always|withText" />  
<item
android:id="@+id/action_location"
android:icon="@drawable/location"
android:title="@string/action_delete"
android:showAsAction="always|withText"/>

</menu>
```


### java代码

``` java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
getMenuInflater().inflate(R.menu.main, menu);
return true;
}
@Override
public boolean onOptionsItemSelected(MenuItem item) {
switch (item.getItemId()) {
case R.id.action_search:
startActivity(new Intent(this,SecondActivity.class));
break;
case R.id.action_locatino:
startActivity(new Intent(this,ThridActivity.class));
break;
default:
break;
}
return super.onOptionsItemSelected(item);
}
```

## ActionBar修改背景，标题大小，返回按钮

### styles.xml

``` xml
<style name="AppBaseTheme" parent="Theme.AppCompat.Light.DarkActionBar">
<item name="android:actionBarStyle">@style/CustomActionBarStyle</item><!--ActionBar样式-->
<item name="android:homeAsUpIndicator">@drawable/back</item> <!--返回icon-->  
</style>

<style name="CustomActionBarStyle" parent="@android:style/Widget.ActionBar">
<item name="android:background">@color/skyblue</item><!-- ActionBar背景颜色 -->
<item name="android:titleTextStyle">@style/ActionBarTitleStyle</item><!-- ActionBar文字样式 -->

</style>    

<style name="ActionBarTitleStyle" parent="@android:style/Widget.Holo.Light">
<item name="android:textColor">@color/white</item>
<item name="android:textSize">20sp</item>
</style>
```

### java代码
然后在代码里面可以静止ActionBar显示icon，显示返回键。

``` java
ActionBar actionBar = getActionBar();
actionBar.setDisplayHomeAsUpEnabled(true);//显示返回
actionBar.setDisplayShowHomeEnabled(false);//不显示icon
```
## ActionBar添加spinner下拉菜单
添加下拉菜单通过代码修改导航模式，添加导航，添加响应事件，然后在style里面设置样式。

### arrays.xml
这是下拉菜单的数据源，放到values里面就可以了。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
<string-array name="city">
<item>厦门市</item>
</string-array> 
</resources>   
```
### java代码
``` java
SpinnerAdapter mSpinnerAdapter = ArrayAdapter.createFromResource(this,R.array.city,android.R.layout.simple_spinner_dropdown_item);
ActionBar actionBar = getActionBar();
//actionBar.setDisplayHomeAsUpEnabled(true);
//actionBar.setDisplayShowHomeEnabled(false);
//actionBar.setTitle(ConfigUtil.APP_NAME);
actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_LIST);// 导航模式必须设为NAVIGATION_MODE_LIST
actionBar.setListNavigationCallbacks(mSpinnerAdapter, null);
```
### styles.xml

``` xml
<style name="AppBaseTheme" parent="Theme.AppCompat.Light.DarkActionBar">
<item name="android:spinnerDropDownItemStyle">@style/CustomDropDownItemStyle</item>
</style>

<style name="CustomDropDownItemStyle" parent="@android:style/Widget.Holo.DropDownItem.Spinner"><!--改变了spinner样式-->
<item name="android:textAppearance">@style/CustomDropDownItemTextStyle</item>
</style>

<style name="CustomDropDownItemTextStyle" parent="@android:style/TextAppearance.DeviceDefault.Large"><!--改变了文字样式-->
<item name="android:textColor">@color/white</item><!--改变了文字颜色-->
</style>
```

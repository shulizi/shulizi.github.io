---
layout: post
title:  "android开发篇：自己做一个json文件工具类"
date:   2016-07-03
categories: android
tags: android jsonutil
---

* content
{:toc}

自己做了一个json文件的工具类，功能有：

根据键更新json中的值
根据单层对象下的多个键获取多个值
根据多层对象下的键获取一个值
根据键值生成json





## 工具类代码

代码中进行了重构，考虑了输入对象，数组，字符串的各个JSON形式的不同解决方案。

JsonUtil.java
``` java
package com.lizi.jsonmaster.Util;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class JsonUtil {
/**
* 判断是否是JSON字符串
* @param strJson 字符串类型
* @return
*/
public static boolean IsJsonArray(String strJson) {
if (strJson.startsWith("["))
return true;
else
return false;
};
/**
* 判断是否是JSON字符串
* @param json JSON对象
* @return
*/
public static boolean IsJsonArray(JSONObject json) {
String strJson = json.toString();
if (strJson.startsWith("["))
return true;
else
return false;
};
/**
* 更新JSON
* @param strJson JSON字符串
* @param strkeys 多层目录的键
* @param value 要更新成什么值
* @return
*/
public static String updateJson(String strJson,String strKeys, String value
) throws JSONException {
String[] keys=strKeys.split(",");
String strValue;
JSONObject object = new JSONObject(strJson);

if (IsJsonArray(strJson)) {
JSONArray jsonArray = new JSONArray(strJson);
strValue = (String) jsonArray.getString(Integer.parseInt(keys[0]));
} else {
strValue = object.getString(keys[0]);
}
for (int i = 1; i < keys.length; i++) {

if (IsJsonArray(strValue)) {
JSONArray jsonArray = new JSONArray(strValue);
strValue = jsonArray.getJSONObject(Integer.parseInt(keys[i++]))
.getString(keys[i]);
continue;
}
JSONObject object2 = new JSONObject(strValue);
strValue = object2.getString(keys[i]);
}
String strJson2 = null;
strJson2 = strJson.substring(0, strJson.indexOf(strValue))
+ value
+ strJson.substring(
strJson.indexOf(strValue) + strValue.length(),
strJson.length());
return strJson2;

}
/**
* 更新JSON
* @param json JSON对象
* @param strkeys 多层目录的键
* @param value 要更新成什么值
* @return
*/
public static String updateJson(JSONObject json,String strKeys, String value
) throws JSONException {
String[] keys=strKeys.split(",");
String strValue;
String strJson = json.toString();

strValue = json.getString(keys[0]);
for (int i = 1; i < keys.length; i++) {

if (IsJsonArray(strValue)) {
JSONArray jsonArray = new JSONArray(strValue);
strValue = jsonArray.getJSONObject(Integer.parseInt(keys[i++]))
.getString(keys[i]);
continue;
}
JSONObject object2 = new JSONObject(strValue);
strValue = object2.getString(keys[i]);
}
String strJson2 = null;
strJson2 = strJson.substring(0, strJson.indexOf(strValue))
+ value
+ strJson.substring(
strJson.indexOf(strValue) + strValue.length(),
strJson.length());
return strJson2;

}
/**
* 更新JSON
* @param json JSON数组
* @param strkeys 多层目录的键
* @param value 要更新成什么值
* @return
*/
public static String updateJson(JSONArray json,String strKeys, String value
) throws JSONException {
String[] keys=strKeys.split(",");

String strJson = json.toString();

String strValue;
strValue = (String) json.getString(Integer.parseInt(keys[0]));

for (int i = 1; i < keys.length; i++) {

if (IsJsonArray(strValue)) {
JSONArray jsonArray = new JSONArray(strValue);
strValue = jsonArray.getJSONObject(Integer.parseInt(keys[i++]))
.getString(keys[i]);
continue;
}
JSONObject object2 = new JSONObject(strValue);
strValue = object2.getString(keys[i]);
}
String strJson2 = null;
strJson2 = strJson.substring(0, strJson.indexOf(strValue))
+ value
+ strJson.substring(
strJson.indexOf(strValue) + strValue.length(),
strJson.length());
return strJson2;

}

/**
* 获得某个键的值
* @param strJson JSON字符串
* @param strkeys 多层目录下的键
* @return
*/
public static String getValueFromJson(String strJson, String strKeys)
throws JSONException {
String[] keys=strKeys.split(",");

JSONObject object = new JSONObject(strJson);
String strValue;
if (IsJsonArray(strJson)) {
JSONArray jsonArray = new JSONArray(strJson);
strValue = (String) jsonArray.getString(Integer.parseInt(keys[0]));
} else
strValue = object.getString(keys[0]);
for (int i = 1; i < keys.length; i++) {
if (IsJsonArray(strValue)) {
JSONArray jsonArray = new JSONArray(strValue);
strValue = jsonArray.getJSONObject(Integer.parseInt(keys[i++]))
.getString(keys[i]);
continue;
}
JSONObject object2 = new JSONObject(strValue);
strValue = object2.getString(keys[i]);

}
return strValue;
}
/**
* 获得某个键的值
* @param json JSON数组
* @param strkeys 多层目录下的键
* @return
*/
public static String getValueFromJson(JSONArray json, String strKeys)
throws JSONException {
String[] keys=strKeys.split(",");
String strValue;
strValue = (String) json.getString(Integer.parseInt(keys[0]));
for (int i = 1; i < keys.length; i++) {
if (IsJsonArray(strValue)) {
JSONArray jsonArray = new JSONArray(strValue);
strValue = jsonArray.getJSONObject(Integer.parseInt(keys[i++]))
.getString(keys[i]);
continue;
}
JSONObject object2 = new JSONObject(strValue);
strValue = object2.getString(keys[i]);

}
return strValue;
}
/**
* 获得某个键的值
* @param json JSON对象
* @param strkeys 多层目录下的键
* @return
*/
public static String getValueFromJson(JSONObject json, String strKeys)
throws JSONException {
String[] keys=strKeys.split(",");

String strValue;
strValue = json.getString(keys[0]);
for (int i = 1; i < keys.length; i++) {
if (IsJsonArray(strValue)) {
JSONArray jsonArray = new JSONArray(strValue);
strValue = jsonArray.getJSONObject(Integer.parseInt(keys[i++]))
.getString(keys[i]);
continue;
}
JSONObject object2 = new JSONObject(strValue);
strValue = object2.getString(keys[i]);

}
return strValue;
}
/**
* 获得多个键的值
* @param strJson JSON字符串
* @param strkeys 单层目录下的多个键
* @return
*/
public static String[] getValuesFromJson(String strJson, String... keys)
throws JSONException {
JSONObject object = new JSONObject(strJson);
String[] values = new String[keys.length];
for (int i = 0; i < keys.length; i++) {
values[i] = object.getString(keys[i]);
}
return values;
}
/**
* 获得多个键的值
* @param json JSON对象
* @param strkeys 单层目录下的多个键
* @return
*/
public static String[] getValuesFromJson(JSONObject json, String... keys)
throws JSONException {
String[] values = new String[keys.length];
for (int i = 0; i < keys.length; i++) {
values[i] = json.getString(keys[i]);
}
return values;
}
/**
* 得到一个json字符串
* @param key
* @param value
* @return
*/
public static String getJsonString(String[] keys, Object[] values)
throws JSONException {
JSONObject jsonObject = new JSONObject();
for (int i=0;i<keys.length;i++){
if(values[i]==null)
values[i]="";
jsonObject.put(keys[i], values[i]);
}

return jsonObject.toString();
}
/**
* 得到一个json对象
* @param key
* @param value
* @return
*/
public static JSONObject getJsonObject(String[] keys, Object[] values)
throws JSONException {
JSONObject jsonObject = new JSONObject();
for (int i=0;i<keys.length;i++){
if(values[i]==null)
values[i]="";
jsonObject.put(keys[i], values[i]);
}
return jsonObject;
}
}

```
## 测试主函数
MainActivity.java
``` java
package com.lizi.jsonmaster;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import android.os.Bundle;
import android.support.v7.app.ActionBarActivity;
import android.util.Log;

import com.lizi.jsonmaster.Util.JsonUtil;
public class MainActivity extends ActionBarActivity {

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
String strJson = "{\"index\":[{\"name\":duli,\"id\":0},{\"name\":duli2,\"id\":1}],\"age\":21}";

String strJsonArray = "[{\"index\":[{\"name\":duli,\"id\":0},{\"name\":duli2,\"id\":1}],\"age\":21},{\"index\":[{\"name\":duli3,\"id\":2},{\"name\":duli4,\"id\":3}],\"age\":21}]";

String result1 = null;
String result2 = null;
String result3 = null;
String result4 = null;
String result5 = null;
String result6 = null;
String result7 = null;
String result8 = null;

try {
result1 = JsonUtil.getValueFromJson(strJson, "index,0,name");
JSONObject jsonObject2 = new JSONObject(strJson);
result2 = JsonUtil.getValueFromJson(jsonObject2, "index,1,id");
JSONArray jsonArray3 = new JSONArray(strJsonArray);
result3 = JsonUtil.getValueFromJson(jsonArray3, "0,age");

result4 = JsonUtil.updateJson(strJson, "index,1,id", "3");
JSONObject jsonObject5 = new JSONObject(strJson);
result5 = JsonUtil.updateJson(jsonObject5, "age", "20");
JSONArray jsonArray6 = new JSONArray(strJsonArray);
result6 = JsonUtil.updateJson(jsonArray6, "1,index,1,name", "lizi");

result7=JsonUtil.getJsonString(new String[]{"name","sex"}, new String[]{"duli","boy"});
JSONObject jsonObject8 = new JSONObject(result7);
result8=JsonUtil.getJsonObject(new String[]{"person","class"}, new Object[]{jsonObject8,null}).toString();
} catch (JSONException e) {
e.printStackTrace();
}
Log.d("test", "result1: " + result1);
Log.d("test", "result2: " + result2);
Log.d("test", "result3: " + result3);
Log.d("test", "result4: " + result4);
Log.d("test", "result5: " + result5);
Log.d("test", "result6: " + result6);
Log.d("test", "result7: " + result7);
Log.d("test", "result8: " + result8);

}

```
## 测试输出
```
07-03 16:50:56.654: D/test(20790): result1: duli 
07-03 16:50:56.654: D/test(20790): result2: 1 
07-03 16:50:56.654: D/test(20790): result3: 21 
07-03 16:50:56.654: D/test(20790): result4: {“index”:[{“name”:duli,”id”:0},{“name”:duli2,”id”:3}],”age”:21} 
07-03 16:50:56.654: D/test(20790): result5: {“index”:[{“id”:0,”name”:”duli”},{“id”:1,”name”:”duli2”}],”age”:20} 
07-03 16:50:56.654: D/test(20790): result6: [{“index”:[{“id”:0,”name”:”duli”},{“id”:1,”name”:”duli2”}],”age”:21},{“index”:[{“id”:2,”name”:”duli3”},{“id”:3,”name”:”lizi”}],”age”:21}] 
07-03 16:50:56.654: D/test(20790): result7: {“sex”:”boy”,”name”:”duli”} 
07-03 16:50:56.654: D/test(20790): result8: {“class”:”“,”person”:{“sex”:”boy”,”name”:”duli”}}

```
## 感谢
“Android系列—JSON数据解析” xiaoluo501395377 [http://www.cnblogs.com/xiaoluo501395377/p/3446605.html](http://www.cnblogs.com/xiaoluo501395377/p/3446605.html) 2016-7-3 
“android json解析及简单例子” 深度开源 [http://www.open-open.com/lib/view/open1326376799874.html](http://www.open-open.com/lib/view/open1326376799874.html) 2016-7-3

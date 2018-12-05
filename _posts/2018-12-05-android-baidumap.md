---
layout: post
title:  "android开发篇：百度地图的定位，搜索"
date:   2016-08-11
categories: android
tags: android baidumap
---

* content
{:toc}

百度地图sdk的官方教程里是我见过写得最好的教程之一了，但对于其开发者来说百度地图的开发还是颇有难度，因为除了这个教程以外能拿得出手的就只有一个论坛了。自己摸索真是困难重重。





## 百度地图定位
定位需下载定位sdk，官方文档说得很清楚，这里我就不再多言，当你做好一切准备工作：导入百度地图lib，定位lib，初始化百度地图之后，我们就可以开始了。

MainActivity.java
``` java


private LocationClient mLocationClient = null;
private BaiduMap mBaiduMap = null;
private MapView mMapView = null;

......
mLocationClient = new LocationClient(getApplicationContext());
mLocationClient.registerLocationListener(myListener);
mLocationClient.start();
mMapView = (MapView) mapLayout.findViewById(R.id.bmapView);
//mMapView.setLogoPosition(LogoPosition.logoPostionleftTop);
//mMapView.showZoomControls(false);

mBaiduMap = mMapView.getMap();
mBaiduMap.setMyLocationEnabled(true);


private BDLocationListener myListener = new MLocationListener( new Handler(new Handler.Callback() {
@Override
public boolean handleMessage(Message msg) {
BDLocation location = (BDLocation) msg.obj;
MyLocationData locData = new MyLocationData.Builder()
.accuracy(location.getRadius())
// 此处设置开发者获取到的方向信息，顺时针0-360
.direction(100).latitude(location.getLatitude())
.longitude(location.getLongitude()).build();
mBaiduMap.setMyLocationData(locData);
LatLng currentLocationLatLng = new LatLng(location.getLatitude(), location
.getLongitude());
MapStatusUpdate mapStatusUpdate = MapStatusUpdateFactory
.newLatLng(currentLocationLatLng);
try {
mBaiduMap.animateMapStatus(mapStatusUpdate);
} catch (Exception e) {
}

return false;
}
}));
```
MLocationListener.java

这个是官方文档里面给的，直接拿来用就可以了。
``` java
package com.lizi.shanghaisandtmuseums.mview;

import java.util.List;

import android.os.Handler;
import android.os.Message;
import android.util.Log;

import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.Poi;

public class MLocationListener implements BDLocationListener {
private Handler handler;
public MLocationListener(Handler whenFinished){
this.handler=whenFinished;
}
@Override
public void onReceiveLocation(BDLocation location) {
// Receive Location
StringBuffer sb = new StringBuffer(256);
sb.append("time : ");
sb.append(location.getTime());
sb.append("\nerror code : ");
sb.append(location.getLocType());
sb.append("\nlatitude : ");
sb.append(location.getLatitude());
sb.append("\nlontitude : ");
sb.append(location.getLongitude());
sb.append("\nradius : ");
sb.append(location.getRadius());
if (location.getLocType() == BDLocation.TypeGpsLocation) {// GPS定位结果
sb.append("\nspeed : ");
sb.append(location.getSpeed());// 单位：公里每小时
sb.append("\nsatellite : ");
sb.append(location.getSatelliteNumber());
sb.append("\nheight : ");
sb.append(location.getAltitude());// 单位：米
sb.append("\ndirection : ");
sb.append(location.getDirection());// 单位度
sb.append("\naddr : ");
sb.append(location.getAddrStr());
sb.append("\ndescribe : ");
sb.append("gps定位成功");

} else if (location.getLocType() == BDLocation.TypeNetWorkLocation) {// 网络定位结果
sb.append("\naddr : ");
sb.append(location.getAddrStr());
// 运营商信息
sb.append("\noperationers : ");
sb.append(location.getOperators());
sb.append("\ndescribe : ");
sb.append("网络定位成功");
} else if (location.getLocType() == BDLocation.TypeOffLineLocation) {// 离线定位结果
sb.append("\ndescribe : ");
sb.append("离线定位成功，离线定位结果也是有效的");
} else if (location.getLocType() == BDLocation.TypeServerError) {
sb.append("\ndescribe : ");
sb.append("服务端网络定位失败，可以反馈IMEI号和大体定位时间到loc-bugs@baidu.com，会有人追查原因");
} else if (location.getLocType() == BDLocation.TypeNetWorkException) {
sb.append("\ndescribe : ");
sb.append("网络不同导致定位失败，请检查网络是否通畅");
} else if (location.getLocType() == BDLocation.TypeCriteriaException) {
sb.append("\ndescribe : ");
sb.append("无法获取有效定位依据导致定位失败，一般是由于手机的原因，处于飞行模式下一般会造成这种结果，可以试着重启手机");
}
sb.append("\nlocationdescribe : ");
sb.append(location.getLocationDescribe());// 位置语义化信息
List<Poi> list = location.getPoiList();// POI数据
if (list != null) {
sb.append("\npoilist size = : ");
sb.append(list.size());
for (Poi p : list) {
sb.append("\npoi= : ");
sb.append(p.getId() + " " + p.getName() + " " + p.getRank());
}
}
Log.i("BaiduLocationApiDem", sb.toString());
Message msg = new Message();
msg.obj=location;
handler.sendMessage(msg);
}
}
```
## 百度地图sdk搜索

百度地图sdk搜索有三种方式：城市搜索，附近搜索和范围搜索，这里只谈前两种，再加一个城市内辖区边界搜索。搜索要设定结果返回listener，这是常识，然后则涉及到热点的添加。

### 城市搜索

``` java
PoiSearch mPoiSearch = PoiSearch.newInstance();
mPoiSearch.setOnGetPoiSearchResultListener(poiListener);
mPoiSearch.searchInCity((new PoiCitySearchOption()).city(ConfigUtil.SHANGHAI).keyword(skeyword).pageCapacity(50).pageNum(pageNumber));
private OnGetPoiSearchResultListener poiListener = new OnGetPoiSearchResultListener() {
public void onGetPoiResult(final PoiResult result) {
// 获取POI检索结果
if (result == null
|| result.error == SearchResult.ERRORNO.RESULT_NOT_FOUND) {
return;
} else if (result.error == SearchResult.ERRORNO.NO_ERROR) {
mBaiduMap.clear();
PoiOverlay overlay = new PoiOverlay(mBaiduMap,
LocationPagerActivity.this, currentSelectNum);
// 设置overlay可以处理标注点击事件
mBaiduMap.setOnMarkerClickListener(overlay);
// 设置PoiOverlay数据
overlay.setData(result);
// 添加PoiOverlay到地图中
overlay.addToMap();
// overlay.zoomToSpan();
for (int i = 0; i < asynResult.length; i++) {
double currentLatitude = result.getAllPoi().get(i).location.latitude;
double currentLongitude = result.getAllPoi().get(i).location.longitude;
LatLng llText = new LatLng(currentLatitude,
currentLongitude);
// 构建文字Option对象，用于在地图上添加文字
OverlayOptions textOption = new TextOptions()
.fontSize(24).fontColor(Color.BLACK)
.text(result.getAllPoi().get(i).name)
.position(llText);
// 在地图上添加该文字对象并显示
mBaiduMap.addOverlay(textOption);

}
};
}
```
上面代码中，PoiOverplay是百度地图已经开源了的添加热点的代码，可以在其sdk附赠的demo中找到。

### 附近搜索
``` java
PoiSearch mPoiSearch = PoiSearch.newInstance();
mPoiSearch.setOnGetPoiSearchResultListener(poiListener);
PoiNearbySearchOption nearbySearchOption = new PoiNearbySearchOption();
nearbySearchOption.location(location);
nearbySearchOption.keyword(skeyword);
nearbySearchOption.radius(radius);// 检索半径，单位是米
mPoiSearch.searchNearby(nearbySearchOption);// 发起附近检索请求
行政区域检索

行政区域检索的代码在demo里面也可以找到，所以多看看demo，什么问题都会很轻松地解决掉的。

DistrictSearch mDistrictSearch;
mDistrictSearch = DistrictSearch.newInstance();
mDistrictSearch.setOnDistrictSearchListener(districSearchResultListener);
mDistrictSearch.searchDistrict(new DistrictSearchOption().cityName("上海").districtName("普陀区"));
private OnGetDistricSearchResultListener districSearchResultListener = new OnGetDistricSearchResultListener() {

@Override
public void onGetDistrictResult(DistrictResult districtResult) {
mBaiduMap.clear();
if (districtResult == null) {
return;
}
if (districtResult.error == SearchResult.ERRORNO.NO_ERROR) {
List<List<LatLng>> polyLines = districtResult.getPolylines();
if (polyLines == null) {
return;
}
LatLngBounds.Builder builder = new LatLngBounds.Builder();
for (List<LatLng> polyline : polyLines) {
OverlayOptions ooPolyline11 = new PolylineOptions()
.width(10).points(polyline).dottedLine(true)
.color(Color.BLUE);
mBaiduMap.addOverlay(ooPolyline11);
OverlayOptions ooPolygon = new PolygonOptions()
.points(polyline).stroke(new Stroke(5, 0xAA00FF88))
.fillColor(0x22FFFF00);
mBaiduMap.addOverlay(ooPolygon);
for (LatLng latLng : polyline) {
builder.include(latLng);
}
}


builder.build());
}
}
};
```

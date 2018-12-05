---
layout: post
title:  "android开发篇：viewpager与listview的合作"
date:   2016-06-11
categories: android
tags: android viewpager listview
---

* content
{:toc}

viewpager和listview的合作据我观察是目前手机app比较流行的一种页面开发方式，这里简单探讨一下。





## viewpager

首先准备五个page页面和一个主页面，主页面里面有个ViewPager（需要android.support.v4包）。
``` xml
<android.support.v4.view.ViewPager
android:id="@+id/vp_main"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
></android.support.v4.view.ViewPager>
```
然后是一个MViewPagerAdapter类，继承PagerAdapter，重载实现方法。
``` java
public class MViewPagerAdapter extends PagerAdapter{
private List<View> mListViews;
public MViewPagerAdapter(List<View> mListViews){ 
this.mListViews=mListViews;
}
@Override
public int getCount() {
return mListViews.size();
}
@Override
public void destroyItem(View v,int index,Object o){
((ViewPager) v).removeView(mListViews.get(index));
}
@Override
public Object instantiateItem(View v,int index){
((ViewPager)v).addView(mListViews.get(index),0);
return mListViews.get(index);
}
@Override
public boolean isViewFromObject(View v, Object o) {
return v==o;
}
}

```
最后在主函数里面进行初始化。
``` java
LayoutInflater lInflater = getLayoutInflater();
mPager=(ViewPager) findViewById(R.id.vp_main);
listViews=new ArrayList<View>();
listViews.add(lInflater.inflate(R.layout.activity_campus, null));
listViews.add(lInflater.inflate(R.layout.activity_competetion, null));
listViews.add(lInflater.inflate(R.layout.activity_disscussion, null));
listViews.add(lInflater.inflate(R.layout.activity_question, null));
mPager.setAdapter(new MViewPagerAdapter(listViews));

mPager.setCurrentItem(0);


```
## ListView
首先准备一个list_item，是listView里满每行的样式。

然后是一个继承自BaseAdapter的Adapter类。
``` java
public class MListAdapter extends BaseAdapter{
private Activity a;
private String[][]mData;
private int layout;
public MListAdapter(Activity a,String[][]mData,int layout){
super();
this.a=a;
this.layout=layout;
this.mData=new String[mData.length][];
for(int i=0;i<mData.length;i++){
this.mData[i]=new String[mData[0].length];
for(int j=0;j<mData[0].length;j++){
this.mData[i][j]=mData[i][j];
}
}
}
......
@Override
public View getView(int position, View view, ViewGroup parent   ) {
LayoutInflater inflater = a.getLayoutInflater();
ViewGroup v=(ViewGroup) inflater.inflate(layout, null);
ImageView imageView = (ImageView) v.findViewById(R.id.im_icon);
TextView title = (TextView) v.findViewById(R.id.tx_title);
TextView content = (TextView) v.findViewById(R.id.tx_content);
imageView.setImageResource(R.drawable.ic_launcher);
title.setText(mData[position][0]);
content.setText(mData[position][1]);
return v;
}
}
```
这里是一个图标，一个标题，一个内容的样式。最后在主函数实现。
``` java
ViewGroup accountGroup = (ViewGroup) lInflater.inflate(R.layout.activity_account, null);
ListView listView = (ListView) accountGroup.findViewById(R.id.lv_main);
ListAdapter listAdapter = new MListAdapter(MainActivity.this, Contents.mData, R.layout.listview_item);
listView.setAdapter(listAdapter);

mPager=(ViewPager) findViewById(R.id.vp_main);
listViews=new ArrayList<View>();

listViews.add(accountGroup);
```
## 感谢
[Android ViewPager多页面滑动切换以及动画效果 越冬越酷 2016-6-10](http://www.cnblogs.com/dwinter/archive/2012/02/27/AndroidViewPager%E5%A4%9A%E9%A1%B5%E9%9D%A2%E6%BB%91%E5%8A%A8%E5%88%87%E6%8D%A2%E4%BB%A5%E5%8F%8A%E5%8A%A8%E7%94%BB%E6%95%88%E6%9E%9C.html)



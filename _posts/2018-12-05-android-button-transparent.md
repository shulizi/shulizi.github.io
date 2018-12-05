---
layout: post
title:  "android开发篇：自定义一个button点击效果辅助类"
date:   2016-06-05
categories: android
tags: android buttonutil
---

* content
{:toc}

原始button点击后有变色效果，如果不喜欢这个效果有很多种修改方法，这里用自定义辅助类的方案实现





## 知识点

### button设置背景透明度
直接用button.setAlpha(100);是个不准确的方法，虽然也可以编译通过，需要指定设定的是drawble还是background的透明度。
``` java
button.getDrawable().setAlpha(100);
```

### 长按效果
长按方法setOnLongClickListener，返回值为true时则点击效果也会被同时监听。
### 透明度恢复
在点击之后，button背景透明有轻微的停顿，展示动画效果，实现方法是通过新建一个线程完成的，过后需要恢复原状，在handler里面进行处理。
``` java
private Handler handler = new Handler(new Handler.Callback() {

@Override
public boolean handleMessage(Message msg) {
// TODO Auto-generated method stub

Button button = (Button) msg.obj;
button.getBackground().setAlpha(255);

return false;
}
});
```
## 代码
``` java
public class ButtonClickUtil {

private Context context;
private Handler handler = new Handler(new Handler.Callback() {

@Override
public boolean handleMessage(Message msg) {
// TODO Auto-generated method stub

Button button = (Button) msg.obj;
button.getBackground().setAlpha(255);

return false;
}
});
public ButtonClickUtil(Context context){
this.context = context;
}
public void setAlphaClickListener(final Button button,final Class toClass){
button.setOnLongClickListener(new OnLongClickListener() {

@Override
public boolean onLongClick(View arg0) {
// TODO Auto-generated method stub
button.getBackground().setAlpha(100);
new Thread(new Runnable(){

@Override
public void run() {
// TODO Auto-generated method stub
try {
Thread.sleep(1000);
} catch (InterruptedException e) {
// TODO Auto-generated catch block
e.printStackTrace();
}
}}).start();
return true;
}

});
button.setOnClickListener(new OnClickListener() {

@Override
public void onClick(View arg0) {
// TODO Auto-generated method stub
button.getBackground().setAlpha(100);
new Thread(new Runnable(){

@Override
public void run() {
// TODO Auto-generated method stub
try {
Thread.sleep(50);
} catch (InterruptedException e) {
// TODO Auto-generated catch block
e.printStackTrace();
}
Message msg = new Message();
msg.obj = button;
handler.sendMessage(msg);

}}).start();
if(toClass!=null){
Intent intent = new Intent();
intent.setClass(context,toClass );
context.startActivity(intent);
}
}
});
}
}
```

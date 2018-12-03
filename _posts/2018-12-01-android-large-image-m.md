---
layout: post
title:  "android开发篇：不压缩加载大图片的自定义view实现下"
date:   2018-12-02 23:20:54
categories: android
tags: android largeimage
---

* content
{:toc}

上一篇提到如何加载高清大图的方法，这一篇进行效果优化。主要有两点：预加载的实现和fling的实现。





## 预加载

预加载的实现比较容易，新添一个bitmap_pre，宽高和大图设成一样，大小却小了很多，可以实现完全加载。然后增添一个preM的matrix即可。
``` java
private Handler preHandler = new Handler(new Handler.Callback() {

@Override
public boolean handleMessage(Message msg) {
bitmap_pre = (Bitmap) msg.obj;
invalidate();
return false;
}
});
private void initPreBitmap() {
new Thread(new Runnable() {
@Override
public void run() {
Bitmap bitmap = BitmapFactory.decodeResource(
context.getResources(), R.drawable.map_pre);
Message msg = new Message();
msg.obj = bitmap;
preHandler.sendMessage(msg);
}
}).start();
}
@Override
protected void onDraw(Canvas canvas) {
super.onDraw(canvas);
if (bitmap != null) {

if (isDrawPre)
canvas.drawBitmap(bitmap_pre, preM, null);
else
canvas.drawBitmap(bitmap, matrix, null);
}

}
```
## fling的实现

fling是快速滑动的意思，这里只是简单实现， 复杂的功能相信你可以自己优化。 
``` java
gestureDetector.onTouchEvent(event); 
```
首先我们需要一个姿态手势的监听继承自SimpleOnGestureListener来获得快速滑动的信息。
``` java
private class MGestureListener extends SimpleOnGestureListener {
@Override
public boolean onDoubleTap(MotionEvent e) {
return super.onDoubleTap(e);
}

@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
float velocityY) {
//          Log.i("test", "onFling:velocityX = " + velocityX + " velocityY"
//                  + velocityY);
startAnimation(e1,e2,velocityX,velocityY);
return super.onFling(e1, e2, velocityX, velocityY);
}
}
};
```
其次需要在ontouchevent里面添加gestureDetector.onTouchEvent(event);这句调用。velocityX和velocityY分别是水平垂直的滑动速度。

动画的实现是新建一个线程，改动matrix实现的，根据速度的递减实现fling。
``` java
private void setMatrix(){
fingerDefaultP.x = fingerScaleP.x + fingerTranslateP.x;
fingerDefaultP.y = fingerScaleP.y + fingerTranslateP.y;
initBitmap();
}
private void move(float x,float y) {
preM.postTranslate(x, y);
matrix.postTranslate(x, y);
fingerTranslateP.x = fingerTranslateP.x - x/scaleRate;
fingerTranslateP.y = fingerTranslateP.y - y/scaleRate;
invalidate();
}
private Handler flingH = new Handler(new Handler.Callback() {
@Override
public boolean handleMessage(Message msg) {
float x=msg.arg1;
float y=msg.arg2;
move(x,y);
return false;
}
});
private void startAnimation(MotionEvent e1, MotionEvent e2,
final float velocityX, final float velocityY) {

new Thread(new Runnable() {
float speedX=velocityX;
float speedY=velocityY;
@Override
public void run() {
while(Math.abs(speedX)>10){
speedX=(float) (speedX/1.1);
speedY=(float) (speedY/1.1);
try {
Thread.sleep(10);
} catch (InterruptedException e) {
e.printStackTrace();
}

Message msg=new Message();
msg.arg1=(int) (speedX/100);
msg.arg2=(int) (speedY/100);
flingH.sendMessage(msg);
}
setMatrix();
}
}).start();

}
```

### 感谢
[Android GestureDetector手势识别类](aben_2005 http://blog.csdn.net/aben_2005/article/details/6417423) 2015-6-17 
[Android 屏幕手势滑动中onFling()函数的技巧分析 泡在网上的日子](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1022/452.html) 2015-6-17 
[Android 关于手指拖动onScroll、onFling…[geoway] 张峰_r](http://www.cnblogs.com/zfrr/archive/2012/07/19/2599591.html) 2015-6-17

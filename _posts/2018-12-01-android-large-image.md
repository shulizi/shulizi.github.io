---
layout: post
title:  "android开发篇：不压缩加载大图片的自定义view实现上"
date:   2018-12-01 23:14:54
categories: android
tags: android largeimage
---

* content
{:toc}

ActionBar的开发是App开发中比较重要的一环，虽然自定义actionbar既灵活又简单，但是不符合规范，适配性便不是最强的，所以了解好ActionBar依然是相当重要。





## 自定义view的基本实现

初始化和覆写onDraw方法就完成了自定义view，能够实现加载一张图片。
``` java
public MapView(Context context) {
super(context);
this.context=context;
init();
}
public MapView(Context context,AttributeSet a) {
super(context,a);
this.context=context;
init();
}
private void init(){

bitmap = BitmapFactory.decodeResource(context.getResources(),R.drawable.map);
}
@Override
protected void onDraw(Canvas canvas) {
super.onDraw(canvas);
if(bitmap!=null)
canvas.drawBitmap(bitmap, matrix, null);
}
```

## 图片滑动

但这还远远不够，起码要实现基本的图片滑动吧，所以要覆写onTouchEvent方法。
``` java
@Override
public boolean onTouchEvent(MotionEvent event) {
switch( event.getAction()){
case MotionEvent.ACTION_DOWN:
fingerOriginP.x = event.getX();
fingerOriginP.y = event.getY();
break;          
case MotionEvent.ACTION_MOVE:
if(fingerOriginDistanceF==-1){

float currentX=event.getX();
float currentY=event.getY();

matrix.postTranslate(currentX- fingerOriginP.x, currentY-fingerOriginP.y);

fingerOriginP.x = currentX;
fingerOriginP.y = currentY;
}

invalidate();
break;

}           
return true;
};
```


## 图片放大和缩小

图片只能移动不能缩小也是不允许的，所以贪心的我继续探究将放大和缩小写进onTouchEvent中。
``` java
@Override
public boolean onTouchEvent(MotionEvent event) {
switch( event.getAction()){
case MotionEvent.ACTION_DOWN:
fingerOriginP.x = event.getX();
fingerOriginP.y = event.getY();
break;          
case MotionEvent.ACTION_MOVE:
if(event.getPointerCount()>=2){
if(fingerOriginDistanceF!=-1){
float currentDistanceF=(float) Math.sqrt((Math.pow(event.getX(0)-event.getX(1), 2)+Math.pow(event.getY(0)-event.getY(1), 2)));
scaleRate*=currentDistanceF/fingerOriginDistanceF;
matrix.postScale(currentDistanceF/fingerOriginDistanceF, currentDistanceF/fingerOriginDistanceF,screenP.x/2,screenP.y/2);
fingerOriginDistanceF=currentDistanceF;
}
else{
fingerOriginDistanceF=(float) Math.sqrt((Math.pow(event.getX(0)-event.getX(1), 2)+Math.pow(event.getY(0)-event.getY(1), 2)));
}
}
else if(fingerOriginDistanceF==-1){

float currentX=event.getX();
float currentY=event.getY();

matrix.postTranslate(currentX- fingerOriginP.x, currentY-fingerOriginP.y);

fingerOriginP.x = currentX;
fingerOriginP.y = currentY;
}

invalidate();
break;
case MotionEvent.ACTION_UP:
if(fingerOriginDistanceF!=-1){
fingerOriginDistanceF=-1;
}

initBitmap();
}           
return true;
};
```

## 大图片的加载
重点到了，虽然大图片我们通常可以进行压缩处理，这是最简单直接的做法，但是我就是要像地图一样显示超大图片，且保证清晰度怎么做，则只能裁剪了。图片的加载由于比较耗时间我放到了线程里，滑动的代码也要相应改动。
``` java
private Handler handler=new Handler(new Handler.Callback() {

@Override
public boolean handleMessage(Message msg) {
bitmap=(Bitmap) msg.obj;
float[] values=getMatrixValues();
values[2]=0;
values[5]=0;
matrix.setValues(values);
invalidate();
return false;
}
});
private void init(){
fingerOriginP=new PointF();
screenP=new PointF();
fingerDefaultP=new PointF();
DisplayMetrics dm=context.getResources().getDisplayMetrics();
screenP.x=dm.widthPixels;
screenP.y=dm.heightPixels;
matrix=new Matrix();
initBitmap();
}
@SuppressLint("NewApi") private void initBitmap(){
new Thread(new Runnable() {
@Override
public void run() {
Bitmap bitmap=null;
try {
InputStream inputStream = context.getAssets().open("map.jpg");
BitmapFactory.Options options = new BitmapFactory.Options();
//                  BitmapFactory.decodeStream(inputStream,null,options);
BitmapRegionDecoder bitmapRegionDecoder = BitmapRegionDecoder.newInstance(inputStream, false);
options.inPreferredConfig = Bitmap.Config.RGB_565;
bitmap=bitmapRegionDecoder.decodeRegion(new Rect((int) (fingerDefaultP.x),(int) ( fingerDefaultP.y), (int) ( fingerDefaultP.x+screenP.x/scaleRate),(int) ( fingerDefaultP.y+screenP.y/scaleRate)), options);

} catch (IOException e) {
Log.d("test","Error");
e.printStackTrace();
}
Message msg=new Message();
msg.obj=bitmap;
handler.sendMessage(msg);
}
}).start();
}
@Override
public boolean onTouchEvent(MotionEvent event) {
switch( event.getAction()){
case MotionEvent.ACTION_DOWN:
fingerOriginP.x = event.getX();
fingerOriginP.y = event.getY();
break;          
case MotionEvent.ACTION_MOVE:
if(event.getPointerCount()>=2){
if(fingerOriginDistanceF!=-1){
float currentDistanceF=(float) Math.sqrt((Math.pow(event.getX(0)-event.getX(1), 2)+Math.pow(event.getY(0)-event.getY(1), 2)));
scaleRate*=currentDistanceF/fingerOriginDistanceF;
matrix.postScale(currentDistanceF/fingerOriginDistanceF, currentDistanceF/fingerOriginDistanceF,screenP.x/2,screenP.y/2);
scaleM.postScale(currentDistanceF/fingerOriginDistanceF, currentDistanceF/fingerOriginDistanceF,screenP.x/2,screenP.y/2);

fingerOriginDistanceF=currentDistanceF;


}
else{
fingerOriginDistanceF=(float) Math.sqrt((Math.pow(event.getX(0)-event.getX(1), 2)+Math.pow(event.getY(0)-event.getY(1), 2)));
}
}
else if(fingerOriginDistanceF==-1){

float currentX=event.getX();
float currentY=event.getY();
fingerTranslateP.x-=(currentX- fingerOriginP.x)/scaleRate;
fingerTranslateP.y-=(currentY- fingerOriginP.y)/scaleRate;
matrix.postTranslate(currentX- fingerOriginP.x, currentY-fingerOriginP.y);

fingerOriginP.x = currentX;
fingerOriginP.y = currentY;
}

invalidate();
break;
case MotionEvent.ACTION_UP:
if(fingerOriginDistanceF!=-1){
fingerOriginDistanceF=-1;
float []scaleValues=getMatrixValues(scaleM);
fingerScaleP.x=-scaleValues[2]/scaleRate;
fingerScaleP.y=-scaleValues[5]/scaleRate;
}
fingerDefaultP.x=(fingerScaleP.x+fingerTranslateP.x);
fingerDefaultP.y=(fingerScaleP.y+fingerTranslateP.y);
initBitmap();
}           
return true;
};
```
如你所见，这里用了两个Matrix，因为一个matrix每次更新视图前都要把values[2]和values[5]，即平移位置为0，使得每次视图都显示剪切部分的左上角。另一个scaleM是用来记录放大信息的，因为前一个Matrix每次将values[2]和values[5]置为0后第二次放大会受影响，并不能得到实际的放大偏移量。另外，有两个记录偏移量的分别是fingerScaleP和fingerTranslateP对应记录因为放大产生的偏移量和因为滑动产生的偏移量，最后加起来便是总的偏移量。

### 感谢
Android 高清加载巨图方案 拒绝压缩图片 [Hongyang 2016-6-11 http://blog.csdn.net/lmj623565791/article/details/49300989](]Hongyang 2016-6-11 http://blog.csdn.net/lmj623565791/article/details/49300989)

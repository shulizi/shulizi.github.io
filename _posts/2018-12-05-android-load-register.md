---
layout: post
title:  "android开发篇：客户端与服务器端交互实现注册与登录"
date:   2016-06-18
categories: android
tags: android load_and_register
---

* content
{:toc}
之前注册与登录在服务器端的实现已经写成了博客，详见javaweb实现android注册与登录，现在主要讲客户端的实现。




## 登录
### 页面布局
#### titlebar的设计
``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:background="@color/white" >

<RelativeLayout
android:layout_width="match_parent"
android:layout_height="50dp"
android:layout_alignParentLeft="true"
android:layout_alignParentTop="true" >

<ImageView
android:id="@+id/imageView1"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_centerVertical="true"
android:paddingLeft="10dp"
android:src="@android:drawable/ic_menu_revert" />

<TextView
android:id="@+id/textView1"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_centerInParent="true"
android:text="@string/login_title"
android:textColor="@android:color/black"
android:textSize="15sp" />

<Button
android:id="@+id/bt_register"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_alignParentRight="true"
android:layout_centerVertical="true"
android:background="@color/white"
android:paddingRight="20dp"
android:text="@string/register"
android:textColor="@color/light_blue" />
</RelativeLayout>

</RelativeLayout>
```
#### 登录界面的设计
登录界面主要是titlebar+ScrollView的组合方式，而登录界面是通过多个LinearLayout嵌套完成的，当然这只是登录设计的一种，大家可以有自己的选择。对于应用来说，排版也是很重要的一环，所以设计这么个登录界面也千万不要马虎，不仅要美观，且要同时考虑到适配性。
``` xml

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="wrap_content" >

<include layout="@layout/titlebar" />

<ScrollView
android:id="@+id/scrollView1"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:layout_alignParentLeft="true"
android:layout_alignParentTop="true"
android:layout_marginTop="50dp"
android:background="@color/gray" >

<LinearLayout
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:paddingTop="150dp"
android:orientation="vertical" >

<LinearLayout
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:orientation="horizontal"
android:background="@color/white"
android:padding="10dp" >

<ImageView
android:id="@+id/ic_account"
android:layout_width="wrap_content"
android:layout_height="match_parent"
android:src="@android:drawable/ic_menu_call" 
android:contentDescription="TODO"/>

<EditText
android:id="@+id/ed_username"
style="@style/my_edittext_style"
android:layout_width="0dp"
android:layout_height="wrap_content"
android:layout_weight="1"
android:ems="10"
android:inputType="phone"
android:paddingLeft="10dp" />
</LinearLayout>

<View
android:layout_width="match_parent"
android:layout_height="1dp"
android:background="@color/gray" />

<LinearLayout
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:orientation="horizontal"
android:background="@color/white"
android:padding="10dp" >

<ImageView
android:id="@+id/ig_password"
android:layout_width="wrap_content"
android:layout_height="match_parent"
android:src="@android:drawable/ic_lock_lock" 
android:contentDescription="TODO"/>

<EditText
android:id="@+id/ed_password"
style="@style/my_edittext_style"
android:layout_width="0dp"
android:layout_height="wrap_content"
android:layout_weight="1"
android:ems="10"
android:inputType="textPassword"
android:paddingLeft="10dp" />
</LinearLayout>

<Button
android:id="@+id/bt_login"
android:layout_margin="10dp"
android:layout_width="match_parent"
android:layout_height="50dp"
android:textColor="@color/white"
android:background="@color/light_blue"
android:text="@string/login" />

<TextView
android:padding="10dp"
android:id="@+id/textView1"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="@string/forgot_pass" />

</LinearLayout>
</ScrollView>

</RelativeLayout>
```

大家可能会注意到这里的editview有个style是自定义的，没错，自定义方法如下。 
style.xml
``` xml
<style name="my_edittext_style" parent="@android:style/Widget.EditText">
<item name="android:background">@drawable/my_edittext</item>
</style>
drawble-*dpi/my_edittext.xml

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android" >
<item android:state_focused="true" android:drawable="@color/white" />      
<item android:drawable="@color/white" />
</selector>
```
浅显易懂不是吗？ 
另外在layout里面有个自定义的color类，大家也可以在res/values下建一个colors.xml。我的colors.xml：
``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
<color name="green">#00ff00</color>
<color name="white">#ffffff</color>
<color name="gray">#eeeeee</color>
<color name="light_blue">#4a64d5</color>
<color name="black">#000000</color>
</resources>
```
### 逻辑实现
访问网页的方法用的是HttpURLConnection，读取数据流用的方法BufferedReader，也有用二进制直接读取inputstream的，但是那多数用于读取图片等信息，自己只是一个登录反馈的文本信息，所以可以直接读取文本。当然如何从网页下载图片我也写过一篇博客android开发篇：如何实现下载一张图片。
``` java
public class LoginActivity extends Activity {
private EditText username;
private EditText password;
private Handler handler = new Handler(new Handler.Callback() {

@Override
public boolean handleMessage(Message msg) {
String returnS=(String) msg.obj;
if (returnS.equals("SUCCESS!WELCOME YOU :"
+ username.getText().toString()+"\n"))
Toast.makeText(LoginActivity.this, "正在登录", Toast.LENGTH_LONG)
.show();
else
Toast.makeText(LoginActivity.this, "用户名或密码错误", Toast.LENGTH_LONG)
.show();
return false;
}
});

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_login);
SMSSDK.initSDK(this, "13be****",
"b01ed3*******");

username = (EditText) findViewById(R.id.ed_username);
password = (EditText) findViewById(R.id.ed_password);

Button login = (Button) findViewById(R.id.bt_login);
login.setOnClickListener(new OnClickListener() {

@Override
public void onClick(View arg0) {
new Thread(new Runnable() {
public void run() {
String urlS = "http://121.42.159.177:8080/login/?username="
+ username.getText().toString()
+ "&password="
+ password.getText().toString();
String returnS = visitNetWork(urlS);
Message msg = new Message();
msg.obj = returnS;
handler.sendMessage(msg);
}
}).start();

}
});



private String visitNetWork(String urlS) {
StringBuilder sb = null;
try {
URL url = new URL(urlS);
HttpURLConnection httpURLConnection = (HttpURLConnection) url
.openConnection();
InputStream is = httpURLConnection.getInputStream();
BufferedReader br = new BufferedReader(new InputStreamReader(is));
sb = new StringBuilder();
String line = null;
while ((line = br.readLine()) != null) {
sb.append(line + "\n");
}
is.close();

} catch (MalformedURLException e) {
e.printStackTrace();
} catch (IOException e) {
e.printStackTrace();
}
return sb.toString();
}

}

```
## 注册
服务器的注册并不是和登录一样那么简单，特别是我要实现验证码登录的时候。要用服务器向手机发验证码需要向运营商注册，而我不想搞那么麻烦，所以这里用了第三方验证码sdk：mob。到mob官网注册个帐号，下载sdk，里面有相当详尽的导入说明，这里就不赘述了。

### 页面布局
#### activity_register
activity_register和totalbar_register我都是采用和登录同样的布局，略做修改，所以这里就不贴代码了。

`注意`在注册界面是手机验证码验证通过后跳转的，所以该界面的手机号码需要设置不可更改，具体可以通过设定不可获得焦点，不可点击，不可长按点击实现。
``` xml
<EditText
android:id="@+id/ed_username"
style="@style/my_edittext_style"
android:layout_width="0dp"
android:layout_height="wrap_content"
android:layout_weight="1"
android:focusable="false"
android:clickable="false"
android:longClickable="false"
android:ems="10"
android:inputType="phone"
android:paddingLeft="10dp" />
```
### 逻辑实现

#### mob部分

mob官方教程里面写得很清楚，导入项目后，添加liberary，修改权限，主界面初始化，新建注册页面就ok了。官方教程传送站。
``` java
Button register = (Button) findViewById(R.id.bt_register);
register.setOnClickListener(new OnClickListener() {

@Override
public void onClick(View arg0) {
// 打开注册页面
RegisterPage registerPage = new RegisterPage();
registerPage.setRegisterCallback(new EventHandler() {
public void afterEvent(int event, int result, Object data) {
// 解析注册结果
if (result == SMSSDK.RESULT_COMPLETE) {
@SuppressWarnings("unchecked")
HashMap<String, Object> phoneMap = (HashMap<String, Object>) data;
String country = (String) phoneMap.get("country");
String phone = (String) phoneMap.get("phone");

// 提交用户信息
registerUser(country, phone);
}
}

private void registerUser(String country, String phone) {
Contents.username=phone;
Intent intent = new Intent();
intent.setClass(LoginActivity.this, RegisterActivity.class);
LoginActivity.this.startActivity(intent);
}
});
registerPage.show(LoginActivity.this);
}
});
}
```
#### RegisterActivity部分

register部分和login部分一样，get方式访问网页。只是新添了密码一致验证，和如果已经注册则跳转登录界面的判定。这里就不贴代码了。注意在短信验证的时候将验证号码保存到contents常量中，在跳转注册界面或者回跳登录界面的时候设置edittext。


## 感谢
android的EditText设置边框颜色 [mrhouzhibin 2016-6-12 http://mrhouzhibin.blog.163.com/blog/static/1945962412011814103148764/](mrhouzhibin 2016-6-12 http://mrhouzhibin.blog.163.com/blog/static/1945962412011814103148764/)

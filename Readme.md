### 参考资料
https://blog.csdn.net/ahuyangdong/article/details/76405557
https://blog.csdn.net/ITermeng/article/details/69403918

### 前言
最近一个项目需要用到扫描二维码的功能，在网上查了一下，都是使用google的ZXing开源库来实现的
### 第一步：导入依赖
导入依赖包,目前最新的是3.3.2，可以通过[这里](http://mvnrepository.com/artifact/com.google.zxing/core)来查看最新版本
```
  implementation 'com.google.zxing:core:3.3.2'
 
```
添加权限
```
<uses-permission android:name="android.permission.INTERNET" /> <!-- 网络权限 -->
<uses-permission android:name="android.permission.VIBRATE" /> <!-- 震动权限 -->
<uses-permission android:name="android.permission.CAMERA" /> <!-- 摄像头权限 -->
<uses-feature android:name="android.hardware.camera.autofocus" /> <!-- 自动聚焦权限 -->
```
### 第二步：添加个性化代码
添加这里面的代码[github](https://github.com/Dream97/ZXingDemo/tree/master/zxing)
![我已经上传到github上面了](https://img-blog.csdn.net/20180524005430283?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjYxMjE0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)那么问题来了，为什么我添加里依赖还是需要用这里面的代码？
因为我们在使用ZXing的时候扫描二维码的页面是用zxing里面的activity的，那么如果我们对扫码页面有其他个性化的需求呢，所以我们需要把这里面我们可能会更改到的代码拷贝到我们的代码中，以实现自己个性化的需求
### 第三步：导入资源文件
![这里写图片描述](https://img-blog.csdn.net/20180524010748551?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjYxMjE0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
raw里面是扫码成功的声音，和其它颜色等等，在mipmap有返回icon,这里不贴图了
### 第四步： 导入XML文件
![这里写图片描述](https://img-blog.csdn.net/20180524011536861?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjYxMjE0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这里就是二维码扫描activity的页面了，另一个是toolbar
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    >

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <SurfaceView
            android:id="@+id/scanner_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center" />

        <com.nongyan.teastore.zxing.view.ViewfinderView
            android:id="@+id/viewfinder_content"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:corner_color="@color/corner_color"
            app:frame_color="@color/viewfinder_frame"
            app:label_text="二维码/条形码扫描"
            app:label_text_color="@color/colorAccent"
            app:laser_color="@color/laser_color"
            app:mask_color="@color/viewfinder_mask"
            app:result_color="@color/result_view"
            app:result_point_color="@color/result_point_color" />

    </FrameLayout>
    <include layout="@layout/toolbar_scanner" />
</RelativeLayout>
```
注意ViewfinderView指的就是我们拷贝的代码中的activity,所以要把路径给改正确

要导入的东西就这么多，接下来要看如何调用的代码了
### 第五步：在myActivity中启动扫描二维码页面
```java
//    @Override
    public void init() {
        setToolbar();
        bt_scan.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.fragment_bt_scan:
            startQrCode();
            break;
        }
    }

    private void startQrCode() {
        if (ContextCompat.checkSelfPermission(getActivity(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            // android 6.0以上需要动态申请权限
            ActivityCompat.requestPermissions(getActivity(), new String[]{Manifest.permission.CAMERA}, Constants.REQ_PERM_CAMERA);
            return;
        }
        // 二维码扫码
        Intent intent = new Intent(getActivity(), CaptureActivity.class);
        startActivityForResult(intent, Constants.REQ_QR_CODE);
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //扫描结果回调
        Toast.makeText(getActivity(),"扫描成功",Toast.LENGTH_SHORT).show();
        if (requestCode == Constants.REQ_QR_CODE && resultCode == RESULT_OK) {
            Bundle bundle = data.getExtras();
            String scanResult = bundle.getString(Constants.INTENT_EXTRA_KEY_QR_SCAN);
            //将扫描出的信息显示出来
            textView.setText(scanResult);
        }
    }
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case Constants.REQ_PERM_CAMERA:
                // 摄像头权限申请
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    // 获得授权
                    startQrCode();
                } else {
                    // 被禁止授权
                    Toast.makeText(getActivity(), "请至权限中心打开本应用的相机访问权限", Toast.LENGTH_LONG).show();
                }
                break;
        }
    }
}

```
### 另外值得一看的
浏览第二篇博客，发现大佬连相册和散光灯都实现了，那就直接把代码贴上来，以后直接用岂不是美滋滋

- 在原有的扫二维码页面中加上打开相册和散光灯按钮 
```XML
 <LinearLayout
        android:orientation="vertical"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginLeft="15dp"
        android:gravity="center">

        <Button
            android:id="@+id/photo_btn"
            android:layout_width="65dp"
            android:layout_height="67dp"
            android:layout_centerVertical="true"
            android:background="@drawable/photo_drawable_btn" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_below="@id/photo_btn"
            android:layout_centerHorizontal="true"
            android:layout_marginTop="10dp"
            android:text="相册"
            android:textColor="#ffffff"
            android:textSize="18sp" />
    </LinearLayout>

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:gravity="center">


        <Button
            android:id="@+id/flash_btn"
            android:layout_width="65dp"
            android:layout_height="67dp"
            android:layout_centerInParent="true"
            android:background="@drawable/flash_drawable_btn" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_below="@id/flash_btn"
            android:layout_centerHorizontal="true"
            android:layout_marginTop="10dp"
            android:text="开灯"
            android:textColor="#ffffff"
            android:textSize="18sp" />
    </LinearLayout>
```
#### 首先是打开相册的
在CaptureActivity中，监听按钮调用选择图片
```java
/**
     * 相册选择图片
     */
    private void selectPhoto() {
        Intent innerIntent = new Intent(Intent.ACTION_GET_CONTENT); // "android.intent.action.GET_CONTENT"

        innerIntent.setType("image/*");

        Intent wrapperIntent = Intent.createChooser(innerIntent, "选择二维码图片");

        startActivityForResult(wrapperIntent,
                REQUEST_CODE);
    }
```
并在Constants中加上    
```java
public static final int REQUEST_CODE = 234;// 相册选择code
```
#### 闪光灯
首先在点击时判断是关灯还是开灯
```java
                if (isTorchOn) {
                    isTorchOn = false;
                    flash_btn.setText("关灯");
                    CameraManager.start();
                } else {
                    isTorchOn = true;
                    flash_btn.setText("开灯");
                    CameraManager.stop();
                }
```
然后调用CameraManager类来控制灯光，start()和stop()方法如下
```java
  private static Camera camera;
 private static Camera.Parameters parameter;
	public static void start() {

		if (camera != null) {
			parameter = camera.getParameters();
			parameter.setFlashMode(Parameters.FLASH_MODE_TORCH);
			camera.setParameters(parameter);
		}

	}

	public static void stop() {
		if (camera != null) {
			parameter = camera.getParameters();
			parameter.setFlashMode(Parameters.FLASH_MODE_OFF);
			camera.setParameters(parameter);
		}

	}

```
静态方法，所以camera也用上静态修饰。调用，开灯，搞定收工。

### 最后说两句
用了ZXing库，是不是发现原来我们也可以对库里面的源码魔改一下的。所以说阅读一下源码，还是很棒的嘛，所以要不要一起阅读一下其他库的源码呢（滑稽脸）


# 下面讲解如何导入到你的项目中


1、在你的root gradle(也就是项目最外层的gradle)添加如下代码

```

allprojects {
    repositories {
        google()
        mavenCentral()
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}


```

2、然后在module gradle dependencies 添加以下依赖，如果添加失败，请打开vpn后重试

```
    implementation "androidx.appcompat:appcompat:1.2.0"
    implementation "androidx.recyclerview:recyclerview:1.1.0"
    implementation "androidx.legacy:legacy-support-v4:1.0.0"
    implementation "com.facebook.fresco:fresco:2.3.0"
    // 支持 GIF 动图，需要添加
    implementation "com.facebook.fresco:animated-gif:2.3.0"
    // 支持 WebP （静态图+动图），需要添加
    implementation "com.facebook.fresco:animated-webp:2.3.0"
    implementation "com.facebook.fresco:webpsupport:2.3.0"
    //跟随viewpager的点
    implementation 'me.relex:circleindicator:1.1.8@aar'
    //上滑控制面板,项目中的potopick中有使用案例
    implementation 'com.sothree.slidinguppanel:library:3.3.0'
    //android6.0权限工具类
    implementation 'com.lovedise:permissiongen:0.1.1'
    //加载超长图必备库
    implementation 'com.davemorrissey.labs:subsampling-scale-image-view:3.10.0'
    //图库
    implementation 'com.github.Awent:PhotoPick-Master:v3.3'
```

3、然后在你的Application的onCreate()方法里初始化即可使用

```
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        FrescoImageLoader.init(this);
        //下面是配置toolbar颜色和存储图片地址的
//        FrescoImageLoader.init(this,android.R.color.holo_blue_light);
//        FrescoImageLoader.init(this,android.R.color.holo_blue_light,"/storage/xxxx/xxx");
    }
}

```

## 下面讲解如何使用
- Hao to use(可进行动态设置参数，不用的可以不设置)

- 图库

```
new PhotoPickConfig.Builder(this)
    .pickModeMulti()                            //多选
    .pickModeSingle()                           //单选
    .onlyImage()                                //只显示图片,不包含视频，默认是图片跟视频都展示,如果showCamera(true)，只会启动系统拍照，并返回图片路径
    .onlyVideo()                                //只显示视频,不包含图片，默认是图片跟视频都展示,如果showCamera(true)，只会启动系统视频录制，并返回视频路径
    .maxPickSize(15)                            //最多可选15张
    .showCamera(false)                          //是否展示拍照icon,默认展示
    .clipPhoto(true)                            //是否选完图片进行图片裁剪，默认是false,如果这里设置成true,就算设置了是多选也会被配置成单选
    .spanCount(4)                               //图库的列数，默认3列，这个数建议不要太大
    .showGif(true)//default true                //是否展示gif
    .setOnPhotoResultCallback(OnPhotoResultCallback onPhotoResultCallback) //设置数据回调，如果不想在Activity通过onActivityResult()获取回传的数据，可实现此接口
    .build();

```

- 获取图库选择了(或裁剪好)的图片地址

```
方法一：
new PhotoPickConfig.Builder(this)
    .pickModeMulti()
    .maxPickSize(15)
    .showCamera(true)
    .setOriginalPicture(true)//让用户可以选择原图
    .setOnPhotoResultCallback(new PhotoPickConfig.Builder.OnPhotoResultCallback() {
          @Override
          public void onResult(PhotoResultBean result) {
                 Log.e("MainActivity", "result = " + result.getPhotoLists().size());
          }
    })
    .build();
    
    
方法二：
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (resultCode != Activity.RESULT_OK || data == null) {
        return;
    }
    switch (requestCode) {
        case PhotoPickConfig.PICK_REQUEST_CODE:
            //返回的图片地址集合
            ArrayList<String> photoLists = data.getStringArrayListExtra(PhotoPickConfig.EXTRA_STRING_ARRAYLIST);
            if (photoLists != null && !photoLists.isEmpty()) {
                File file = new File(photoLists.get(0));//获取第一张图片
                if (file.exists()) {
                    //you can do something

                } else {
                    //toast error
                }
            }
            break;
    }
}

```

- 查看大图

```
new PhotoPagerConfig.Builder<T>(this)
    .fromList(Iterable<? extends T> iterable, OnItemCallBack<T> listener)//接收集合数据并提供内循环返回所有item
    .fromMap(Map<?, T> map, OnItemCallBack<T> listener)                  //接收集合数据并提供内循环返回所有item
    .setBigImageUrls(ImageProvider.getImageUrls())      //大图片url,可以是sd卡res，asset，网络图片.
    .setSmallImageUrls(ArrayList<String> smallImgUrls)  //小图图片的url,用于大图展示前展示的
    .addSingleBigImageUrl(String bigImageUrl)           //一张一张大图add进ArrayList
    .addSingleSmallImageUrl(String smallImageUrl)       //一张一张小图add进ArrayList
    .setSavaImage(true)                                 //开启保存图片，默认false
    .setPosition(2)                                     //默认展示第2张图片
    .setSaveImageLocalPath("Android/SD/xxx/xxx")        //这里是你想保存大图片到手机的地址,可在手机图库看到，不传会有默认地址
    .setBundle(bundle)                                  //传递自己的数据，如果数据中包含java bean，必须实现Parcelable接口
    .setOpenDownAnimate(false)                          //是否开启下滑关闭activity，默认开启。类似微信的图片浏览，可下滑关闭一样
    .setOnPhotoSaveCallback(new OnPhotoSaveCallback()   //保存网络图片到本地图库的回调,保存成功则返回本地图片路径，失败返回null
    .build();
    
自定义界面,详细自定义用法参考demo：

new PhotoPagerConfig.Builder(this,Class<?> clazz)       //这里传入你自定义的Activity class,自定义的activity必须继承PhotoPagerActivity
    ...
    ...
    ...
    .build();
```

- 动态改变toolbar颜色

```
FrescoImageLoader.setToolbarBackGround(R.color.black);

```

- 混淆

```
参考simple中的proguard-rules文件

```

- 开发中常用的查看网络大图`fromList`和`fromMap`用法介绍

    1、使用场景：
    图片`url`存在于集合实体类里，例如：`Map<Integer, UserBean.User>` 或 `List<UserBean.User> userList`，
    这时候不需要自己循环这些集合取出图片url了，本方法会提供内部循环，你只需关注你的图片`url`字段就行

    2、使用示例：用户头像`avatar`存在于`User`实体类里面，是通过服务器返回来的

```
List<UserBean.User> list = getUserInfo();           //使用fromList
or
Map<Integer, UserBean.User> map = new HashMap<>();  //使用fromMap

java写法：
            new PhotoPagerConfig.Builder<UserBean.User>(this)
                        .fromList(list, new PhotoPagerConfig.Builder.OnItemCallBack<UserBean.User>() {
                            @Override
                            public void nextItem(UserBean.User item, PhotoPagerConfig.Builder<UserBean.User> builder) {
                                builder.addSingleBigImageUrl(item.getAvatar());
                            }
                        })
                        .setOnPhotoSaveCallback(new PhotoPagerConfig.Builder.OnPhotoSaveCallback() {
                            @Override
                            public void onSaveImageResult(String localFilePath) {
                                Toast(localFilePath != null ? "保存成功" : "保存失败");
                            }
                        })
                        .build();
                                  
                                  
kotlin写法：
              list?.let {
                PhotoPagerConfig.Builder<UserBean.User>(this)
                        .fromList(it) { item, builder ->
                            builder.addSingleBigImageUrl(item.avatar)//这个avatar就是你需要关注的字段，在这里设置进去即可
                        }.build()
              }
                                  
```

### v3.3
2020-09-06
implementation 'com.github.Awent:PhotoPick-Master:v3.3'

新增本地视频预览和选择，默认是图片跟视频都展示

1、onlyImage():只显示图片,不包含视频

2、onlyVideo():只显示视频,不包含图片

注意:

1、如果设置onlyVideo()和showCamera(true)，只会启动系统视频录制，并返回视频路径

2、如果onlyImage()和showCamera(true)，只会启动系统拍照，并返回图片路径

3、如果图片跟视频都显示，并且showCamera(true)，只是启动拍照，而不是录制视频


### v3.2
2020-09-01
implementation 'com.github.Awent:PhotoPick-Master:v3.2'

新增查看网络大图时开发常用写法和使用kotlin实现自定义界面教程
查看网络大图新增方法：

1、`fromList(Iterable<? extends T> iterable, OnItemCallBack<T> listener)`//接收集合数据并提供内循环返回所有item

2、`fromMap(Map<?, T> map, OnItemCallBack<T> listener)`                  //接收集合数据并提供内循环返回所有item

### v3.1
2020-08-28
implementation 'com.github.Awent:PhotoPick-Master:v3.1'

解决由于BaseBitmapDataSubscriber导致的内存泄漏问题，提交的问题跟官方给出的解决方案是不行的,只能自己去解决，有用到查看网络大图功能的都建议更新此版本
https://github.com/facebook/fresco/issues/2530



### v3.0
2020-07-01
implementation 'com.github.Awent:PhotoPick-Master:v3.0'

适配androidX，如果自己项目不适配androidX的请使用v2.9版本





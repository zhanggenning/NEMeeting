## 概述

网易会议Android SDK提供了一套简单易用的接口，允许开发者通过调用NEMeeting SDK(以下简称SDK)提供的API，快速地集成音视频会议功能至现有Android应用中。

## 变更记录

| 日期 | 版本 | 变更内容 |
| :------: | :------: | :------: | 
| 2020-07-10  | 1.0.0 | 首次正式发布 |

## 快速接入

#### 开发环境准备

| 名称 | 要求 | 
| :------ | :------ |
| JDK版本  | >1.8.0 |
| 最小Android API 版本 | API 21, Android 5.0 |
| CPU架构支持 | ARM64、ARMV7 |
| IDE | Android Studio |
| 其他 | 依赖androidx，不支持support库 |

#### SDK快速接入

1. 新建Android工程

    a. 运行Android Sudio，顶部菜单依次选择“File -> New -> New Project...”新建工程，选择'Phone and Tablet' -> 'Empty Activity' 单击Next。
    ![new android project](images/new_project.png)
    
    b. 配置工程相关信息，请注意Minimum API Level为API 21。
    ![configure project](images/configure_project.png)
    
    c. 单击'Finish'完成工程创建。

2. 添加SDK编译依赖

    修改工程目录下的'app/build.gradle'文件，添加网易会议SDK的依赖。
    ```groovy
    dependencies {
      //声明SDK依赖，版本可根据实际需要修改
      implementation 'com.netease.yunxin:meetinglib:1.0.0'
    }
    ```
    之后通过顶部菜单'Build -> Make Project'构建工程，触发依赖下载，完成后即可在代码中引入SDK中的类和方法。

3. 权限处理

    网易会议SDK正常工作需要应用获取以下权限
    ```xml
    <!-- 网络相关 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />

    <!-- 读写外部存储 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <!-- 多媒体 -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    ```
    以上权限已经在SDK内部进行声明，开发者可以不用在```AndroidManifest.xml```文件中重新声明这些权限，但运行时的权限申请需要应用开发者自己编码实现，可在应用首页中统一申请，详情可参考[Android运行时权限申请示例](https://developer.android.google.cn/guide/topics/permissions/overview)。如果运行时对应权限缺失，SDK可能无法正常工作，如会议时无图像、对方听不到己方声音等。

4. SDK初始化

    在使用SDK其他功能之前首先需要完成SDK初始化，初始化操作需要保证在**Application**的**onCreate**方法中执行。代码示例如下：
    ```java
    public class MyApplication extends Application {

        private static final String TAG = "MyApplication";
    
        @Override
        public void onCreate() {
            super.onCreate();
            NEMeetingSDKConfig config = new NEMeetingSDKConfig();
            config.appKey = Constants.APPKEY;
            NEMeetingSDK.getInstance().initialize(this, config, new NECallback<Void>() {
                @Override
                public void onResult(int resultCode, String resultMsg, Void resultData) {
                    if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
                        //TODO when initialize success
                    } else {
                        //TODO when initialize fail
                    }
                }
            });
        }
    }
    ```

5. 调用相关接口完成特定功能，详情请参考API文档。

- 登录鉴权
    ```java
    NEMeetingSDK.getInstance().login(String account, String token, NECallback<Void> callback);
    ```
- 创建会议
    ```java
    NEMeetingService meetingService = NEMeetingSDK.getInstance().getMeetingService();
    meetingService.startMeeting(Context context, NEStartMeetingParams param, NEStartMeetingOptions opts, NECallback<Void> callback);
    ```
- 加入会议
    ```java
    NEMeetingService meetingService = NEMeetingSDK.getInstance().getMeetingService();
    meetingService.joinMeeting(Context context, NEJoinMeetingParams param, NEJoinMeetingOptions opts, NECallback<Void> callback);
    ```
- 添加/移除会议状态监听
    ```java
    NEMeetingService meetingService = NEMeetingSDK.getInstance().getMeetingService();
    meetingService.addMeetingStatusListener(NEMeetingStatusListener listener);    //添加监听
    meetingService.removeMeetingStatusListener(NEMeetingStatusListener listener); //移除监听
    ```
- 添加/移除登录状态监听
    ```java
    NEMeetingSDK.getInstance().addAuthListener(NEAuthListener authListener);    //添加监听
    NEMeetingSDK.getInstance().removeAuthListener(NEAuthListener authListener); //移除监听
    ```
- 注销登录
    ```java
    NEMeetingSDK.getInstance().logout(NECallback<Void> callback);
    ```

## 业务开发

### 初始化

#### 描述

在使用SDK其他接口之前，首先需要完成初始化操作。

#### 业务流程

1. 配置初始化相关参数

```java
NEMeetingSDKConfig config = new NEMeetingSDKConfig();
config.appKey = Constants.APPKEY; //应用AppKey
```

2. 调用接口并进行回调处理，该接口无额外回调结果数据

```java
NEMeetingSDK.getInstance().initialize(getApplication(), config, new NECallback<Void>() {
    @Override
    public void onResult(int resultCode, String resultMsg, Void result) {
        if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
            //初始化成功
        } else {
            //初始化失败
        }
    }
});
```

#### 注意事项

- 初始化操作需要保证在**Application**类的**onCreate**方法中执行

--------------------

### 登录鉴权

#### 描述

请求SDK进行登录鉴权，只有完成SDK登录鉴权才允许创建会议。

#### 业务流程

1. 获取登录用账号ID和Token。Token由网易会议应用服务器下发，但SDK不提供对应接口获取该信息，需要开发者自己实现。

```java
String accountId = "accountId";
String accountToken = "accountToken";
```

2. 登录并进行回调处理，该接口无额外回调结果数据

```java
NEMeetingSDK.getInstance().login(accountId, accountToken, new NECallback<Void>() {
    @Override
    public void onResult(int resultCode, String resultMsg, Void result) {
        if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
            //登录成功
        } else {
            //登录失败
        }
    }
});
```

#### 注意事项

- SDK不提供账号注册机制，第三方应用集成SDK时需要为第三方应用的用户帐号绑定网易会议系统中企业管理员开通的员工帐号，第三方应用的用户帐号和企业员工帐号是1:1映射的。

--------------------

### 创建会议

#### 描述

在已经完成SDK登录鉴权的状态下，创建并开始一个新的会议。

#### 业务流程

1. 配置创建会议用的相关参数

```java
NEStartMeetingParams params = new NEStartMeetingParams();   //会议参数
params.meetingId = "123456789";                             //会议号
params.displayName = "我的会议昵称";                          //会议昵称

NEStartMeetingOptions options = new NEStartMeetingOptions(); //会议选项
options.noVideo = true;                                      //入会时关闭视频，默认为true
options.noAudio = true;                                      //入会时关闭音频，默认为true
```

2. 调用接口并进行回调处理。该接口无额外回调结果数据，可根据错误码判断是否成功

```java
NEMeetingSDK.getInstance().getMeetingService().startMeeting(getActivity(), params, options, new NECallback<Void>() {
    @Override
    public void onResult(int resultCode, String resultMsg, Void result) {
        if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
            //创建会议成功
        } else {
            //创建会议失败
        }
    }
});
```

3. 创建会议成功后，SDK会拉起会议界面并接管会议逻辑，开发者无需做其他处理。创会人会自动成为该会议的主持人，可进行相关的会议控制操作。其他参会者可通过该会议号加入到该会议中来。

#### 注意事项

- 创建会议时，会议号可以配置为个人会议号(登录后可通过**AccountService**获取)，或者置空(此时由服务器随机分配会议号)。
- 该接口仅支持**在登录鉴权成功后调用**，其他状态下调用不会成功

--------------------

### 加入会议

#### 描述

在已登录或未登录的状态下，加入一个当前正在进行中的会议。

#### 业务流程

1. 配置加入会议用的相关参数

```java
NEJoinMeetingParams params = new NEJoinMeetingParams();     //会议参数
params.meetingId = "123456789";                             //会议号
params.displayName = "我的会议昵称";                          //会议昵称

NEJoinMeetingOptions options = new NEJoinMeetingOptions();   //会议选项
options.noVideo = true;                                      //入会时关闭视频，默认为true
options.noAudio = true;                                      //入会时关闭音频，默认为true
```

2. 调用接口并进行回调处理。该接口无额外回调结果数据，可根据错误码判断是否成功

```java
NEMeetingSDK.getInstance().getMeetingService().joinMeeting(getActivity(), params, options, new NECallback<Void>() {
    @Override
    public void onResult(int resultCode, String resultMsg, Void result) {
        if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
            //加入会议成功
        } else {
            //加入会议失败
        }
    }
});
```

3. 加入会议成功后，SDK会拉起会议界面并接管会议逻辑，开发者无需做其他处理。

#### 注意事项

- 会议号不能为空，需要配置为真实进行中的会议ID
- 该接口支持登录和未登录状态调用

--------------------

### 监听会议状态

#### 描述

通过注册会议状态回调接口，可获取到会议状态变更的通知。

#### 业务流程

1. 注册回调接口开始监听，并在回调方法中处理感兴趣的事件

```java
NEMeetingStatusListener listener = new NEMeetingStatusListener() {
    @Override
    public void onMeetingStatusChanged(Event event) {
       //处理会议状态变更事件          
    }
};
NEMeetingSDK.getInstance().getMeetingService().addMeetingStatusListener(listener);
```

2. 反注册回调接口停止监听

```java
NEMeetingSDK.getInstance().getMeetingService().removeMeetingStatusListener(listener);
```

#### 注意事项

- 在SDK初始化成功后可以调用

--------------------

### 获取个人会议号

#### 描述

在完成登录授权状态下，获取当前账号关联的个人会议ID。

#### 业务流程

1. 确认已完成SDK登录鉴权

2. 调用接口并进行回调处理。该接口的回调结果数据类型为字符串类型，可根据错误码判断是否成功

```java
NEMeetingSDK.getInstance().getAccountService().getPersonalMeetingId(new NECallback<String>() {
    @Override
    public void onResult(int resultCode, String resultMsg, String result) {
        if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
            //获取个人会议号成功，result即为个人会议号
        } else {
            //获取个人会议号失败
        }
    }
});
```

3. 获取个人会议号后，可用于创建会议

#### 注意事项

- 仅在已登录状态下才会成功返回数据

--------------------

### 监听登录状态

#### 描述

通过注册登录状态回调接口，可获取到登录状态变更的通知。

#### 业务流程

1. 注册回调接口开始监听，并在回调接口中处理感兴趣的事件

```java
NEAuthListener authListener = new NEAuthListener() {
    @Override
    public void onKickOut() {
        //当前账号已在其他设备上登录
    }
};
NEMeetingSDK.getInstance().addAuthListener(authListener);    //添加监听
```

2. 反注册回调接口停止监听

```java
NEMeetingSDK.getInstance().removeAuthListener(authListener); //移除监听
```

#### 注意事项

- 无

--------------------

### 注销

#### 描述

请求SDK注销当前已登录账号，返回未登录状态。

#### 业务流程

1. 调用接口并进行回调处理。该接口无额外回调结果数据，可根据错误码判断是否成功

```java
NEMeetingSDK.getInstance().logout(new NECallback<Void>() {
    @Override
    public void onResult(int resultCode, String resultMsg, Void result) {
        if (resultCode == NEMeetingError.ERROR_CODE_SUCCESS) {
            //注销登录成功
        } else {
            //注销登录失败
        }
    }
});
```

#### 注意事项

- 账号注销后，登录状态被清空，不再允许创建会议



## Android Permission库

在项目添加仓库地址：
```
maven {
    url 'https://raw.githubusercontent.com/Fly-Felix/android-maven/master/repository'
}
```

在APP级build.gradle添加：
```
dependencies {
    /**Request Permission**/
    implementation 'fly:android.permission:1.0.0'
}
```

## 基本使用

支持在Activity和Fragment中使用，使用方法如下：
```
PermissionManager.builder(this)
             .add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
             .setCallback(new PermissionManager.Callback() {
                 @Override
                 public void onPermissionGrant(String[] permissions) {
                     Log.d(TAG, "onPermissionGrant: " + Arrays.toString(permissions));
                 }
             })
             .request();
```

## 个性化扩展
在权限被拒绝或禁用时，发起request之前会显示一个对话框，这个对话框可由自己精心定制，库提供了两种默认配置

* 强制授权配置：PermissionRequestConfigs.mustAuthorize()
* 非强制获权配置：PermissionRequestConfigs.ignorableAuthorize()
* 自定义请求配置：实现PermissionRequestConfig抽象类

自定义配置创建请求权限Dialog示例：
```
 PermissionManager.builder(this)
                .add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
                .setCallback(new PermissionManager.Callback() {
                    @Override
                    public void onPermissionGrant(String[] permissions) {
                        Log.d(TAG, "onPermissionGrant: " + Arrays.toString(permissions));
                    }
                })
                .setPermissionRequestConfig(new PermissionRequestConfig() {
                    @Override
                    public AlertDialog.Builder onCreateRequestPermissionDialog(Context context, IPermissionManager manager, String action, String permission) {
                        String msg = permission;
                        if (permission != null) {
                            msg = PermissionUtils.getPermissionDescription(context, permission);
                        } else {
                            PackageManager pm = context.getPackageManager();
                            PermissionInfo info = PermissionUtils.getPermissionInfoByAction(context, action);
                            if (info != null) {
                                msg = info.loadDescription(pm).toString();
                            }
                        }
                        return new AlertDialog.Builder(context)
                                .setTitle(msg)
                                .setCancelable(false)
                                .setPositiveButton("好的", new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialog, int which) {
                                        manager.request();
                                    }
                                })
                                .setNegativeButton("取消", new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialog, int which) {
                                        manager.cancelRequest();
                                    }
                                });
                    }
                })
                .request();
```

## API

### PermissionManager


主要使用类,所有开放方法支持链式调用

<br/>
获取实例

`PermissionManager.budiler(this) 或 new PermissionManager(this)`

<br/>
添加权限或Settings的Action

`add(String value)`

<br/>
设置权限请求结果回调

`setCallback(PermissionManager.Callback callback)`

<br/>
设置自定义配置

`setPermissionRequestConfig(PermissionRequestConfig config)`

<br/>
请求权限

`request()`


<br/>

### PermissionRequestConfig

提供三个抽象方法，**checkPermissionBySettingAction**和**onCreateAppSettingIntent**两个方法主要就是如果库更新不及时出现权限申请逻辑问题可由自己实现，再提交issue有时间我会修改。**onCreateRequestPermissionDialog**这个大部分人应该会需要自定义Dialog毕竟原生的太丑了。

<br/>
通过Settings参数检查是否拥有对应权限

`checkPermissionBySettingAction(Context context, String action)`

<br/>
实现自定义请求权限提示Dialog

`onCreateRequestPermissionDialog(Context context, final IPermissionManager manager, String action, String permission)`

<br/>
通过Settings和权限创建跳转系统的Intent

`onCreateAppSettingIntent(Context context, String action, String permission)`

<br/>

## PermissionManager.Callback

获取权限结果回调类，这也是是一个抽象类，默认抽象方法实现了继续请求和显示对话框逻辑

<br/>
授权拒绝

`onPermissionDenied(IPermissionDenied denied)`

<br/>
权限禁用

`onPermissionDisabled(IPermissionDisabled disabled)`

<br/>
授予权限

`onPermissionGrant(String[] permissions)`


<br/>

## IPermissionDenied和IPermissionDisabled

继承于IPermissionManager，这两个类分别实现了拒绝和禁用权限时逻辑，提供三个方法，方便权限被拒绝之后继续请求或者其它的逻辑

<br/>
请求权限

`request()`

<br/>
取消请求

`cancelRequest()`

<br/>
是否已取消请求授权

`isCancelRequest()`
























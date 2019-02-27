# 前言

>Android在碎片化的体现已不再是API层面上的，近两年受异形屏市场冲击，最终就是开发者的一场灾难，并没有一个万全之策能做到通用的适配。

>真的有必要每位开发者都花大量时间聚焦在适配问题上吗？适配不是一个多高深的技术，无非是`某个点`我们是否知道。正是为避面这种`某个点`不知道而折腾很久而建立本仓。这里不提供所谓的适配库，只是把简短的差异化代码公开，防止脏引用。


## 厂商资料
- [谷歌](https://developer.android.com/guide/practices/screens_support)
- [小米](https://dev.mi.com/console/doc/)
- [华为](https://developer.huawei.com/consumer/cn/devservice/doc/50111)
- [Vivo](https://dev.vivo.com.cn/documentCenter/doc/145)
- [Oppo](https://open.oppomobile.com/wiki/doc#id=10159)
- [三星](http://support-cn.samsung.com/App/DeveloperChina/notice)
- [魅族](http://open-wiki.flyme.cn/doc-wiki/index#id?76)
- [联想](http://open.lenovo.com/sdk/)
- [锤子](https://resource.smartisan.com/resource/61263ed9599961d1191cc4381943b47a.pdf)
>讲个笑话，当用户运行普通app在手机上没能正常运行时用户会喷垃圾App,微信没能在手机上正常运行时，用户会喷垃圾手机....所以结果就是手机厂商在发新机之前会找大厂提前安排适配，并提供文档。而小公司就无能为力了，甚至硬件资源的条件也没用。你说气人不？


### 差异化API

### 虚拟按键 or 全屏手势

- Vivo：`0: 手势 1: 虚拟按键`
```kotlin
Settings.Secure.getInt(context.contentResolver, "navigation_gesture_on", 0)
```

- 小米：`0: 手势 1: 虚拟按键`
```kotlin
Settings.Global.getInt(context.contentResolver, "force_fsg_nav_bar", 0)
```

- 其它：`是否存在导航条`
```kotlin
        private fun hasNavigationBar(context: Context): Boolean {
            var hasNavigationBar = false
            try {
                val systemPropertiesClass = Class.forName("android.os.SystemProperties")
                val m = systemPropertiesClass.getMethod("get", String::class.java)
                val navBarOverride = m.invoke(systemPropertiesClass, "qemu.hw.mainkeys") as String
                if ("1" == navBarOverride) {
                    hasNavigationBar = false
                } else if ("0" == navBarOverride) {
                    hasNavigationBar = true
                } else {
                    val rs = context.resources
                    val id = rs.getIdentifier("config_showNavigationBar", "bool", "android")
                    if (id > 0) {
                        hasNavigationBar = rs.getBoolean(id)
                    }
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }

            return hasNavigationBar
        }
```

### 异形屏
- AndroidP：`是否异型屏`
```
        WindowInsets windowInsets = window.getDecorView().getRootWindowInsets();
        if(null == windowInsets){
            return false;
        }
        DisplayCutout displayCutout = windowInsets.getDisplayCutout();
        return null != displayCutout;

```

- 小米：`0:非异型，1:异型`
```kotlin
SystemProperties.get("ro.miui.notch", null)
```
- 魅族：`0:非异型，1:异型`
```kotlin
Settings.Global.getInt(window.getContext().getContentResolver(),
                    "mz_fringe_hide", 0)
```
- 华为：`是否异型屏`
```java
        ClassLoader cl = context.getClassLoader();
        Class HwNotchSizeUtil = cl.loadClass("com.huawei.android.util.HwNotchSizeUtil");
        Method get = HwNotchSizeUtil.getMethod("hasNotchInScreen");
         (boolean) get.invoke(HwNotchSizeUtil);

```
- Oppo：`是否异型屏`

```kotlin
getContext().hasSystemFeature("com.oppo.feature.screen.heteromorphism");
```

- Vivo：`是否异型屏  圆角：0x00000008 ，凹凸：0x00000020` 
```java
            ClassLoader cl = window.getContext().getClassLoader();
            vivoFtFeature = cl.loadClass("android.util.FtFeature");
            Method get = vivoFtFeature.getMethod("isFeatureSupport", int.class);
            (boolean) get.invoke(vivoFtFeature, 0x00000020);
            
```

- 锤子（Smartisan）：`是否异型屏`
```java
           Class<?> DisplayUtilsSmt = Class.forName("smartisanos.api.DisplayUtilsSmt");
           Method isFeatureSupport = DisplayUtilsSmt.getMethod("isFeatureSupport", int.class);
           (boolean) isFeatureSupport.invoke(DisplayUtilsSmt, 0x00000001);
```
- 锤子（Smartisan）：`获取锤子手机导航栏模式  0导航栏  1手势操作`
```java
Settings.Global.getInt(context.getContentResolver(), "navigationbar_trigger_mode", 0);
```

- 三星（挖孔屏）

  [三星手机钻孔屏适配指导](http://support-cn.samsung.com/App/DeveloperChina/notice/detail?noticeid=86)


### 展示刘海

- 华为: `0:展示刘海，1:隐藏刘海`
```kotlin
Settings.Secure.getInt(context.contentResolver, "display_notch_status", 0)
```

- 小米: `0:展示刘海，1:隐藏刘海`
```kotlin
Settings.Global.getInt(context.contentResolver, "force_black", 0)
```




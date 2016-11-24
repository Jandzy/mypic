## android沉浸式标题栏

> ![img](https://raw.githubusercontent.com/Jandzy/mypic/master/android%E6%B2%89%E6%B5%B8%E5%BC%8F%E7%8A%B6%E6%80%81%E6%A0%8F/1.jpg)

<!--more-->

## 状态栏透明

一般状态栏（colorPrimaryDark）是黑色的，变透明有两种方式

### 在style文件中修改

```xml
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
  <!-- Customize your theme here. -->
  <item name="colorPrimary">@color/colorPrimary</item>
  <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
  <item name="colorAccent">@color/colorAccent</item>
  <item name="android:windowTranslucentStatus">true</item>
  <item name="android:windowTranslucentNavigation">true</item>
</style>
```

主要是android:windowTranslucentStatus设置为true是拉升到顶部状态栏，并且定义顶部状态栏透明。（其中colorPrimaryDark就是默认状态栏的颜色）

### 在代码中修改
在setContentView之前调用
```java
if(VERSION.SDK_INT >= VERSION_CODES.KITKAT) {
                                //透明状态栏
  getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
                                //透明导航栏
  getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
                        }
```

设置完之后效果如下图，
![状态栏透明拉伸](https://raw.githubusercontent.com/Jandzy/mypic/master/android%E6%B2%89%E6%B5%B8%E5%BC%8F%E7%8A%B6%E6%80%81%E6%A0%8F/2.png)
会发现状态栏和toolbar重叠到了一起，网上说的有很多种方法，比如加padding值等，发现一个比较方便的方法，在toolbar中添加fitSystemwindows属性即可
```xml
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true"
        android:background="#ff0000">
```
关于fitSystwindows属性：
>官方描述:
 Boolean internal attribute to adjust view layout based on system windows such as the status bar. If true, adjusts the padding of this view to leave space for the system windows. Will only take effect if this view is in a non-embedded activity.

简单描述：

这个一个boolean值的内部属性，让view可以根据系统窗口(如status bar)来调整自己的布局，如果值为true,就会调整view的paingding属性来给system windows留出空间....

实际效果：

当status bar为透明或半透明时(4.4以上),系统会设置view的paddingTop值为一个适合的值(status bar的高度)让view的内容不被上拉到状态栏，当在不占据status bar的情况下(4.4以下)会设置paddingTop值为0(因为没有占据status bar所以不用留出空间)。

添加fitSystemWindows属性后效果如图：
![fitSystemWindows效果](https://raw.githubusercontent.com/Jandzy/mypic/master/android%E6%B2%89%E6%B5%B8%E5%BC%8F%E7%8A%B6%E6%80%81%E6%A0%8F/3.png)
现在状态栏的颜色就和toolbar颜色一致啦。

>另外，除了使用fitSystemWindows属性外还可以给toolbar添加`pddingTop=25`来达到同样的效果。使用fitSystemWindows的时候可能会有坑，比如我的仿今日头条的一个小项目———["今日小头条"](https://github.com/Jandzy/News)，底部用的TabWidgt，此时整个布局相当于在FrameLayout之中，如果在用fitSystemWindows的话会发现只有一个可以，这时候就需要用paddingTop来实现。这也算是一个小坑。


## 通过Palette动态改变颜色

```java
 Palette.from(bitmap).generate(new Palette.PaletteAsyncListener() {
            @Override
            public void onGenerated(Palette palette) {
                Palette.Swatch swatch = palette.getVibrantSwatch();
                if(swatch!=null){
                    toolbar.setBackgroundColor(swatch.getRgb());
                }else{
                    if(palette.getMutedSwatch()!=null) toolbar.setBackgroundColor(palette.getMutedSwatch().getRgb());
                }
            }
        });
```

Palette是android 5.0后添加的新特性，主要用途是通过一个从bitmap中获得你想要的色调，然后用来适配app达到色调一致的效果。

### 使用Palette 

1. 先添加gradle依赖

	dependencies {
    
	    compile 'com.android.support:+'
	    compile 'com.android.support:+'
 
	}



2. 使用Palette

```java
/**
 * A helper class to extract prominent colors from an image.
 * <p>
 * A number of colors with different profiles are extracted from the image:
 * <ul>
 *     <li>Vibrant</li>//充满活力的色调
 *     <li>Vibrant Dark</li>//充满活力的黑
 *     <li>Vibrant Light</li>//充满活力的亮
 *     <li>Muted</li>//柔和的色调
 *     <li>Muted Dark</li>//柔和的黑
 *     <li>Muted Light</li>//柔和的亮
 * </ul>
 * These can be retrieved from the appropriate getter method.
 *
 * <p>
 * Instances are created with a {@link Builder} which supports several options to tweak the
 * generated Palette. See that class' documentation for more information.
 * <p>
 * Generation should always be completed on a background thread, ideally the one in
 * which you load your image on. {@link Builder} supports both synchronous and asynchronous
 * generation:
 *
 * <pre>
 * // Synchronous
 * Palette p = Palette.from(bitmap).generate();
 *
 * // Asynchronous
 * Palette.from(bitmap).generate(new PaletteAsyncListener() {
 *     public void onGenerated(Palette p) {
 *         // Use generated instance
 *     }
 * });
 * </pre>
 */
```

直接看Palette类的注释，获取Palette有两种方法，同步和异步。其中Palette对象包含六种色调，获得palette对象后，通过`palette.getVibrantSwatch()`等不同的色调获取Palette.Swatch内部类，通过swatch对象可以获取到具体的rgb、hsl等信息。然后对相应的组件设置颜色即可。


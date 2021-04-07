---
layout: post
title: "Android 动画深入分析"
subtitle: '全面了解 Android 动画'
author: "Song"
header-style: text
tags:
  - android
  - animation
---

## View 动画
### View 动画种类
目前 Android 应用框架支持的补间动画效果有以下5种。具体实现在 android.view.animation 类库中。
1. AlphaAnimation：透明度动画，对应 `<alpha>` 标签。
2. TranslateAnimation：平移动画，需要指定移动点的开始和结束坐标，对应 `<translate>` 标签。
3. ScaleAnimation：缩放动画，可以指定缩放的参考点，对应 `<scale>` 标签。
4. RotateAnimation：旋转动画，可以指定旋转的参考点，对应 `<rotate>` 标签。
5. AnimationSet：组合渐变，支持组合多种渐变效果，对应 `<set>` 标签。
> 他们即可以用代码来动态创建也可以用 XML 来定义，推荐使用可读性更好的 XML 来定义。若用 xml 资源文件实现，资源文件全部放入 Android 项目的 `res/anim/` 目录下。

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@android:anim/decelerate_interpolator"  
    android:shareInterpolator="true" >
    <!--
    android:interpolator 动画的渲染器  
  1、accelerate_interpolator(动画加速器) 使动画在开始的时候 最慢,然后逐渐加速  
  2、decelerate_interpolator(动画减速器)使动画在开始的时候 最快,然后逐渐减速  
  3、accelerate_decelerate_interpolator（动画加速减速器）中间位置分层:  使动画在开始的时候 最慢,然后逐渐加速。使动画在开始的时候 最快,然后逐渐减速  结束的位置最慢  
    -->
    
    <scale  
        android:duration="2000"  
        android:fromXScale="0.2"  
        android:fromYScale="0.2"  
        android:pivotX="50%"  
        android:pivotY="50%"  
        android:toXScale="1.5"  
        android:toYScale="1.5" />
        <!--   
        fromXScale:表示沿着x轴缩放的起始比例  
        toXScale:表示沿着x轴缩放的结束比例  
        fromYScale:表示沿着y轴缩放的起始比例  
        toYScale:表示沿着y轴缩放的结束比例  
        图片中心点：  
        android:pivotX="50%"   
        android:pivotY="50%"  
        --> 
  
    <rotate  
        android:duration="1000"  
        android:fromDegrees="0"  
        android:repeatCount="1"  
        android:repeatMode="reverse"  
        android:toDegrees="360" />
        <!--   
        fromDegrees:表示旋转的起始角度  
        toDegrees:表示旋转的结束角度  
        repeatCount:旋转的次数  默认值是0 代表旋转1次  如果值是repeatCount=4 旋转5次，值为-1或者infinite时，表示补间动画永不停止  
        repeatMode 设置重复的模式。默认是restart。当repeatCount的值大于0或者为infinite时才有效。  
        repeatCount=-1 或者infinite 循环了  
        还可以设成reverse，表示偶数次显示动画时会做与动画文件定义的方向相反的方向动行。  
        -->
  
    <translate  
        android:duration="2000"  
        android:fromXDelta="0"  
        android:fromYDelta="0"  
        android:toXDelta="320"  
        android:toYDelta="0" /> 
        <!--   
        fromXDelta  动画起始位置的横坐标  
        toXDelta    动画起结束位置的横坐标  
        fromYDelta  动画起始位置的纵坐标  
        toYDelta   动画结束位置的纵坐标  
        duration 动画的持续时间  
        -->  
  
    <alpha  
        android:duration="2000"  
        android:fromAlpha="1.0"  
        android:toAlpha="0.1" />  
        <!--   
           fromAlpha :起始透明度  
           toAlpha:结束透明度  
           1.0表示完全不透明  
           0.0表示完全透明  
  			--> 
</set>  
```

**设置加载属性动画**
```java
public void translateImpl(View v) {
    Animation animation = AnimationUtils.loadAnimation(this, R.anim.translate_demo);
    animation.setRepeatCount(Animation.INFINITE);//循环显示
    v.startAnimation(animation);
}
```
> **但是要注意的一点是，例如平移开始点相对的锚点默认是view的左上角**

**使用代码应用动画**
- setDuration	设置持续时间
- startNow	立刻启动动画
- start	启动动画
- cancel	取消动画
- setRepeatCount	设置重复次数
- setFillEnabled	使能填充效果
- setFillBefore	设置起始填充
- setFillAfter	设置终止填充
- setRepeatMode	设置重复模式
- setStartOffset	设置启动时间

#### TranslateAnimation
TranslateAnimation类是Android系统中的位置变化动画类，用于控制View对象的位置变化，该类继承于Animation类。TranslateAnimation类中的很多方法都与Animation类一致，该类中最常用的方法便是TranslateAnimation构造方法。

【基本语法】public TranslateAnimation (float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)

【参数说明】
fromXDelta：位置变化的起始点X坐标。
toXDelta：位置变化的结束点X坐标。
fromYDelta：位置变化的起始点Y坐标。
toYDelta：位置变化的结束点Y坐标。

#### RotateAnimation
RotateAnimation类是Android系统中的旋转变化动画类，用于控制View对象的旋转动作，该类继承于Animation类。RotateAnimation类中的很多方法都与Animation类一致，该类中最常用的方法便是RotateAnimation构造方法。

【基本语法】public RotateAnimation (float fromDegrees, float toDegrees, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)

【参数说明】
fromDegrees：旋转的开始角度。
toDegrees：旋转的结束角度。
pivotXType：X轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT。
pivotXValue：X坐标的伸缩值。
pivotYType：Y轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT。
pivotYValue：Y坐标的伸缩值。

#### ScaleAnimation
 ScaleAnimation类是Android系统中的尺寸变化动画类，用于控制View对象的尺寸变化，该类继承于Animation类。ScaleAnimation类中的很多方法都与Animation类一致，该类中最常用的方法便是ScaleAnimation构造方法。

【基本语法】public ScaleAnimation (float fromX, float toX, float fromY, float toY, int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)

【参数说明】
fromX：起始X坐标上的伸缩尺寸。
toX：结束X坐标上的伸缩尺寸。
fromY：起始Y坐标上的伸缩尺寸。
toY：结束Y坐标上的伸缩尺寸。
pivotXType：X轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT。
pivotXValue：X坐标的伸缩值。
pivotYType：Y轴的伸缩模式，可以取值为ABSOLUTE、RELATIVE_TO_SELF、RELATIVE_TO_PARENT。
pivotYValue：Y坐标的伸缩值。

#### AlphaAnimation
AlphaAnimation类是Android系统中的透明度变化动画类，用于控制View对象的透明度变化，该类继承于Animation类。AlphaAnimation类中的很多方法都与Animation类一致，该类中最常用的方法便是AlphaAnimation构造方法。

【基本语法】public AlphaAnimation (float fromAlpha, float toAlpha)

【参数说明】
fromAlpha：开始时刻的透明度，取值范围0~1。
toAlpha：结束时刻的透明度，取值范围0~1。

#### AnimationSet
AnimationSet类是Android系统中的动画集合类，用于控制View对象进行多个动作的组合，该类继承于Animation类。AnimationSet类中的很多方法都与Animation类一致，该类中最常用的方法便是addAnimation方法，该方法用于为动画集合对象添加动画对象。

```java
public void test(View view) {
		Animation translateAnimation = new TranslateAnimation(0, 300, 0, 300);//移动动画效果 
		translateAnimation.setDuration(3000); //设置动画持续时间 
		view.setAnimation(translateAnimation); //设置动画效果 
		translateAnimation.startNow(); //启动动画 
}
```

### 自定义 View 动画
自定义View动画只需要继承 Animation 这个抽象类，并重写 initialize 和 applyTransformation 方法，在 initialize 方法中做一些初始化工作，在 applyTransformation 中进行相应的矩形变换，很多时候需要采用 Camera 来简化矩形变换过程

### 帧动画
帧动画是顺序播放一组预先定义好的图片，类似电影播放；使用简单但容易引发OOM,尽量避免使用过多尺寸较大的图片。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    根标签为animation-list
    XML文件要放在/res/drawable目录下
    其中oneshot代表着是否只展示一遍，设置为false会不停的循环播放动画根标签下，通过item标签对动画中的每一个图片进行声明
    android:duration 表示展示所用的该图片的时间长度
 -->
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="true">
    <item
        android:drawable="@mipmap/ic_launcher1"
        android:duration="150"></item>
    <item
        android:drawable="@mipmap/ic_launcher2"
        android:duration="150"></item>
    <item
        android:drawable="@mipmap/ic_launcher3"
        android:duration="150"></item>
    <item
        android:drawable="@mipmap/ic_launcher4"
        android:duration="150"></item>
</animation-list>
```
android中的逐帧动画的原理很简单，跟电影的播放一样，一张张类似的图片不停的切换，当切换速度达到一定值时，我们的视觉就会出现残影，残影的出现保证了视觉上变化的连续性，这时候图片的切换看在我们眼中就跟真实的一样了

```java
public void test(ImageView targetButton) {
    // 获取AnimationDrawable对象
    targetButton.setBackgroundResource(R.drawable.test);
    AnimationDrawable animationDrawable = (AnimationDrawable) targetButton.getBackground();
    // 动画是否正在运行
    if (animationDrawable.isRunning()) {
        //停止动画播放
        animationDrawable.stop();
    } else {
        //开始或者继续动画播放
        animationDrawable.start();
    }
}
```
> 需要在res/drawable文件夹下新建一个xml文件

## View 动画特殊使用场景
### LayoutAnimation
LayoutAnimation 作用于 ViewGroup，为 ViewGroup 指定一个动画，当他的子元素出场的时候都会具有这种动画，ListView 上用的多，LayoutAnimation也是一个View动画。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:animationOrder="normal"
    android:delay="0.3" android:animation="@anim/anim_item"/>
//--- delay 表示动画开始的时间延迟，比如子元素入场动画的时间周期为300ms,
//那么0.5表示每个子元素都需要延迟150ms才开始播放入场动画。
//--- animationOrder 表示子元素的动画的顺序，有三种选项：
//normal(顺序显示）、reverse（逆序显示）和random（随机显示）。
//---1. android:animation 为子元素指定具体的入场动画
//----------------------
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:shareInterpolator="true">
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />
    <translate
        android:fromXDelta="300"
        android:toXDelta="0" />
</set>
```

```java
//--- 第一种方法、为需要的ViewGroup指定android:layoutAnimation属性
//---这样ViewGroup的子View就有出场动画了
   <ListView
       android:id="@+id/lv"
       android:layout_width="match_parent"
       android:layout_height="0dp"
       android:layout_weight="1"
       android:layoutAnimation="@anim/anim_layout"/>
       
//--- 第二种方法、通过LayoutAnimationController来实现
Animation animation = AnimationUtils.loadAnimation(this,R.anim.anim_item);
LayoutAnimationController controller = new LayoutAnimationController(animation);
controller.setDelay(0.5f);
controller.setOrder(LayoutAnimationController.ORDER_NORMAL);
listview.setLayoutAnimation(controller);
```

### Activity 切换效果
- 在startActivity(Intent) 或 finish() 之后调用 overridePendingTransition(int enterAnim,int exitAnim)方法。
- Fragment 也可以添加切换动画，通过 FragmentTransaction 中的 setCustomAnimations() 方法来添加

## 属性动画
- 概念：属性动画（property animation），这个是在Android 3.0中才引进的，一定要注意兼容的最低版本。Property Animation改变view的真实属性，比喻坐标、透明度、放大比例等等，在View Animation和Tween Animation中，其改变的是View的绘制效果，真正的View的属性保持不。
- Property Animation 不止可以应用于 View，还可以应用于任何对象。Property Animation只是表示一个值在一段时间内的改变，当值改变时，可以随意对控制对象的操作。**在这里重点声明一下动画无论什么动画只能在父控件里面动，超过了父控件都会被遮盖**

### 常用属性：
- Duration动画的持续时间，默认300ms。
- Time interpolation：时间插值。LinearInterpolator、AccelerateDecelerateInterpolator，定义动画的变化率。
- Repeat count and behavior：重复次数、以及重复模式；可以定义重复多少次；重复时从头开始，还是反向。
- Animator sets: 动画集合，你可以定义一组动画，一起执行或者顺序执行。
- Frame refresh delay：帧刷新延迟，对于你的动画，多久刷新一次帧；默认为10ms，但最终依赖系统的当前状态；基本不用管。

### 相关的类:
- ObjectAnimator 动画的执行类
- ValueAnimator 动画的执行类
- AnimatorSet 用于控制一组动画的执行：线性，一起，每个动画的先后执行等。
- AnimatorInflater 用户加载属性动画的xml文件
- TypeEvaluator 类型估值，主要用于设置动画操作属性的值。
- TimeInterpolator 时间插值

总的来说，属性动画就是，动画的执行类来设置动画操作的对象的属性、持续时间，开始和结束的属性值，时间差值等，然后系统会根据设置的参数动态的变化对象的属性。

先来一最简单的示例
```java
private void test(View view) {
    //最简单的动画
    ObjectAnimator//
            .ofFloat(view, "rotationX", 0.0F, 360.0F)//
            .setDuration(500)//
            .start();
}
```

#### ObjectAnimator 用法
ObjectAnimator 类的静态方法 ofFloat，发现源码里面实际上是 new 出一个 ObjectAnimator 对象，然后返回这个对象的赋值，由此可见其他的方法（ofInt，ofArgb）都是通过类似的方式构建的

```java
public static ObjectAnimator ofFloat(Object target, String propertyName, float... values) {
    ObjectAnimator anim = new ObjectAnimator(target, propertyName);
    anim.setFloatValues(values);
    return anim;
}
```
target：动画的目标对象
value：就是初始值和结束值
propertyName：属性名，详解如下
（借用大神[http://blog.csdn.net/eclipsexys/article/details/38401641](http://blog.csdn.net/eclipsexys/article/details/38401641)）
PS：可操纵的属性参数：x/y；scaleX/scaleY；rotationX/ rotationY；transitionX/ transitionY等等。
PS：X是View最终的位置、translationX为最终位置与布局时初始位置的差。所以若就用translationX即为在原来基础上移动多少，X为最终多少。getX()的值为getLeft()与getTranslationX()的和。

1. translationX和translationY：这两个属性作为一种增量来控制着View对象从它布局容器的左上角坐标开始的位置。
2. rotation、rotationX和rotationY：这三个属性控制View对象围绕支点进行2D和3D旋转。
3. scaleX和scaleY：这两个属性控制着View对象围绕它的支点进行2D缩放。
4. pivotX和pivotY：这两个属性控制着View对象的支点位置，围绕这个支点进行旋转和缩放变换处理。默认情况下，该支点的位置就是View对象的中心点。
5. x和y：这是两个简单实用的属性，它描述了View对象在它的容器中的最终位置，它是最初的左上角坐标和translationX和translationY值的累计和。
6. alpha：它表示View对象的alpha透明度。默认值是1（不透明），0代表完全透明（不可见）。
7. backgroundColor，color等其他属性名

然后设置延迟时间，看源码知道返回值是对象本身

```java
@Override
@NonNull
public ObjectAnimator setDuration(long duration) {
    super.setDuration(duration);
    return this;
}
```
最后直接start动画，如果你要做的操作一般的平移，缩放什么的的无法满足，那就可以自定义自己的动作了，如下。propertyName 可以写任意值，只需要监听（addUpdateListener）在1.0f和0.0f之间变化的数值，然后做自己任意的操作
```java
public void rotateyAnimRun(final View view) {
        ObjectAnimator anim = ObjectAnimator//
                .ofFloat(view, "any", 1.0F, 0.0F)//
                .setDuration(500);//
        anim.start();
        anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float cVal = (Float) animation.getAnimatedValue();
                view.setAlpha(cVal);
                view.setScaleX(cVal);
                view.setScaleY(cVal);
            }
        });
    }
```
ofint用法类似，其实还有另外一个用法ofPropertyValuesHolder，代码贴上，一看就明白，不用多说的吧

```java
public void propertyValuesHolder(View view) {
        PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("alpha", 1f, 0f, 1f);
        PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("scaleX", 1f, 0, 1f);
        PropertyValuesHolder pvhZ = PropertyValuesHolder.ofFloat("scaleY", 1f, 0, 1f);
        ObjectAnimator.ofPropertyValuesHolder(view, pvhX, pvhY, pvhZ).setDuration(1000).start();
    }
```

#### ValueAnimator 用法
上面这部分基本把 ObjectAnimator 的用法分析清楚了，其实ValueAnimator用法和ObjectAnimator比较类似，ObjectAnimator是继承ValueAnimator，拥有了一个propertyName属性，对特有的平移，缩放，透明度，旋转进行处理，下面简单贴一个例子，大家应该就明白了

```java
public void verticalRun( final View view) {
        ValueAnimator animator = ValueAnimator.ofFloat(0, 1000 - view.getHeight());
        animator.setTarget(view);
        animator.setDuration(1000).start();
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                view.setTranslationY((Float) animation.getAnimatedValue());
            }
        });
    }
```
这和 ObjectAnimator 添加 addUpdateListener 监听，设置任意属性用法非常类似

#### TypeEvaluator 用法
还有一个TypeEvaluator可能用到，类型估值，控制属性具体的变化，例子贴出来，直接设置对象的变化

```java
public void verticalRun( final View view) {
        ValueAnimator animator = new ValueAnimator();
        animator.setObjectValues(new PointF(0, 0));
        animator.setTarget(view);
        animator.setDuration(1000).start();
        animator.setEvaluator(new TypeEvaluator<PointF>() {
            @Override
            public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
                PointF point = new PointF();
                point.x = 200 * fraction * 3;
                point.y = 0.5f * 200 * (fraction * 3) * (fraction * 3);
                return point;
            }
        });
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                PointF point = (PointF) animation.getAnimatedValue();
                view.setX(point.x);
                view.setY(point.y);
//                view.setTranslationY((Float) animation.getAnimatedValue());
            }
        });
    }
```

#### 监听

```java
animator.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            super.onAnimationEnd(animation);
        }
    });
```
> 这是系统实现 AnimatorListener 的适配器类，也可以自己实现AnimatorListener接口。
> **animator 还有 cancel() 和 end() 方法：cancel 动画立即停止，停在当前的位置；end动画直接到最终状态。**

#### AnimatorSet 用法
Animatorset用法很简单，将其他的动画类添加进来就可以，一种是playTogether一起执行，一种是playSequentially依次执行，还有一种就是有先后顺序，主要有after，before，with。

下面是playTogether用法
```java
public void togetherRun(View view) {
    ObjectAnimator anim1 = ObjectAnimator.ofFloat(view, "scaleX", 1.0f, 2f);
    ObjectAnimator anim2 = ObjectAnimator.ofFloat(view, "scaleY", 1.0f, 2f);
    AnimatorSet animSet = new AnimatorSet();
    animSet.setDuration(2000);
    animSet.setInterpolator(new LinearInterpolator());
    //两个动画同时执行
    animSet.playTogether(anim1, anim2);
    animSet.start();
}
```
我们可以看到playTogether的源码，参数对象只要是Animator的子类就可以，而本质上还是用的builder.with（）方法，下面重点分析after，before，with这三个方法

```java
public void playTogether(Animator... items) {
        if (items != null) {
            mNeedsSort = true;
            Builder builder = play(items[0]);
            for (int i = 1; i < items.length; ++i) {
                builder.with(items[i]);
            }
        }
    }
```

看看playSequentially依次执行的源码，本质上是before方法，主要是看
这句话
```
play(items[i]).before(items[i+1])
```
第i个是在第i+1个之前，参数里面的动画按照顺序依次执行
```java
public void playSequentially(Animator... items) {
        if (items != null) {
            mNeedsSort = true;
            if (items.length == 1) {
                play(items[0]);
            } else {
                mReversible = false;
                for (int i = 0; i < items.length - 1; ++i) {
                    play(items[i]).before(items[i+1]);
                }
            }
        }
    }
```

after，before，with的分析，直接代码贴上

```java
public void playWithAfter(View view) {
        float cx = view.getX();
        ObjectAnimator anim1 = ObjectAnimator.ofFloat(view, "scaleX", 1.0f, 2f);
        ObjectAnimator anim2 = ObjectAnimator.ofFloat(view, "scaleY", 1.0f, 2f);
        ObjectAnimator anim3 = ObjectAnimator.ofFloat(view, "x", cx, 0f);
        ObjectAnimator anim4 = ObjectAnimator.ofFloat(view, "x", cx);
        /**
         * anim1，anim2,anim3同时执行
         * anim4接着执行
         */
        AnimatorSet animSet = new AnimatorSet();
        animSet.play(anim1).with(anim2);
        animSet.play(anim2).with(anim3);
        animSet.play(anim4).after(anim3);
        animSet.setDuration(1000);
        animSet.start();
    }
```
XML 示例
```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="sequentially">
    <objectAnimator
        android:duration="4000"
        android:propertyName="x"
        android:valueTo="300"
        android:valueType="intType" />
    <objectAnimator
        android:duration="4000"
        android:propertyName="y"
        android:valueTo="400"
        android:valueType="intType" />

    <objectAnimator
        android:duration="4000"
        android:propertyName="x"
        android:valueTo="0"
        android:valueType="intType" />
    <objectAnimator
        android:duration="4000"
        android:propertyName="y"
        android:valueTo="0"
        android:valueType="intType" />
</set>
```
【备注：】
android:ordering说明一系列动画动作的执行顺序，有两个选择: sequentially 和together，顺序执行还是一起执行； 
objectAnimator 是设定动画实施的对象；
duration是该动画动作执行从开始到结束所用的时间；
android:repeatCount="infinite"   可以是整数或者infinite
android:repeatMode="restart"    可以是restart 或者 reverse
android:valueFrom=" "     整数|浮点数|颜色

```java
AnimatorSet set =(AnimatorSet)AnimatorInflater.loadAnimator(this,R.animator.property_anim);
// 设置要控制的对象
set.setTarget(move);
// 开始动画
set.start();
```

## 使用动画注意事项
- 使用帧动画时，当图片数量较多且图片分辨率较大的时候容易出现 OOM，需注意，尽量避免使用帧动画。
- 使用无限循环的属性动画时，在 Activity 退出时及时停止，否则将导致 Activity 无法释放从而造成内存泄露。
- View 动画是对 View 的影像做动画，并不是真正的改变了 View 的状态，因此有时候会出现动画完成后 View 无法隐藏（setVisibility(View.GONE）失效），这时候调用 view.clearAnimation() 清理 View 动画即可解决。
- 不要使用 px，使用 px 会导致不同设备上有不同的效果。
- V iew 动画是对 View 的影像做动画，View 的真实位置没有变动，也就导致点击 View 动画后的位置触摸事件不会响应，属性动画不存在这个问题。
- 使用动画的过程中，使用硬件加速可以提高动画的流畅度。
- 动画在 3.0 以下的系统存在兼容性问题，特殊场景可能无法正常工作，需做好适配工作

title: android-animate-review
tags:
---

# View动画

视图动画，也叫Tween（补间）动画可以在一个视图容器内执行一系列简单变换（位置、大小、旋转、透明度）。譬如，如果你有一个TextView对象，您可以移动、旋转、缩放、透明度设置其文本，当然，如果它有一个背景图像，背景图像会随着文本变化。

ImageView spaceshipImage = (ImageView) findViewById(R.id.spaceshipImage);
Animation hyperspaceJumpAnimation = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
spaceshipImage.startAnimation(hyperspaceJumpAnimation);

# 属性动画

顾名思义，属性动画就是动态地修改对象的属性而实现动画。

## ObjectAnimator

```java
ObjectAnimator
                .ofFloat(mImageView, "rotationY", 0f, 360f)
                .setDuration(2000)
                .start();
```
## ValueAnimator
通过addUpdateListener，获取值自定义改变属性。

## TypeEvaluator

通过实现接口，完成复杂动画的属性变化。

ObjectAnimator 

ObjectAnimator ofObject(Object target, String propertyName,
            TypeEvaluator evaluator, Object... values)


# 帧动画

``` xml
	<?xml version="1.0" encoding="utf-8"?>
	<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
			android:oneshot="false">
	<item android:drawable="@drawable/dw1" android:duration="100"/>
	<item android:drawable="@drawable/dw2" android:duration="100"/>
	</animation-list>
```
```java
	mImageView.setImageResource(R.drawable.frame_anim);
	AnimationDrawable animationDrawable = (AnimationDrawable) mImageView.getDrawable();
	animationDrawable.start();
```

# 矢量图片

http://www.jianshu.com/p/afc7b3a2e3e9

http://www.jianshu.com/p/a3cb1e23c2c4

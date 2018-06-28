---
title: android-tips
date: 2016-11-21 16:56:55
tags: [Android,tips]
categories: "Android"
---

这里收集了大家常用的一些Android小技巧代码,内容来自自己和自己所遇到的坑。

- Resources.getSystem().getDisplayMetrics().density 可以不用 Context 也能**获取屏幕密度**哦
- 通过重载 ViewGroup 的 dispatchDraw 可以实现一个简单的**蒙版效果**。 例如下拉刷新时，可以在 contentView 上加一层遮罩。 canvas.drawRect(0, mContentView.getTranslationY(), getWidth(), getHeight(), mMaskPaint);
- 使用 GridView时 android:padding 和 **android:clipToPadding="false" **配合使用效果更好哦。
- **DateUtils**系统提供了关于时间操作的类
-  TypedValue.applyDimension() 系统提供的**单位转换**Api
-  使用LocalBroadcastManager发送**广播**替代全局广播

---
title: android学习笔记(3)
date: 2015-11-19 14:29:05
tags: android
---
## 五大布局
1. ### LinearLayout 线性布局

  * android:orientation="vertical/horizontal" 子类控件排列方式
    <code>vertical</code> 垂直  
    <code>horizontal</code> 水平
  * android:gravity="center/right/left..."子类控件位置
     <code>center_horizontal</code> 水平居中
     <code>center_vertical</code> 垂直居中
     可以同时设置多个属性，如<code>bottom|center_horizontal</code>，设置子类控件位于底部的中央
  * 子控件可以通过设置<code>android:layout_orientation</code>或<code>android:layout_gravity</code>来控制位置
  * 子控件通过<code>android:layout_weight</code>来设置控件的占比。  
       <code>android:layout_weight</code>真实含义是:一旦View设置了该属性(假设有效的情况下)，那么该 View的宽度等于原有宽度(<code>android:layout_width</code>)加上剩余空间的占比！
> Google官方推荐，当使用weight属性时，将width设为0dip即可，效果跟设成<code>wrap_content</code>是一样的。这样weight就可以理解为占比了！

2. ### RelativeLayout 相对布局
  #### 全局属性
  ***
 * <code>android:layout_alignParentLeft="true"</code> 子类控件相对当前父类容器靠左边
 * <code>android:layout_alignParentTop="true"</code> 子类控件相对当前父类容器靠上边
  * <code>android:layout_marginLeft="41dp"</code> 子类控件距离当前父类容器左边的距离
  * <code>android:layout_marginTop="41dp"</code> 子类控件距离当前父类容器上边的距离
  * <code>android:layout_centerInParent="true"</code> 子类控件相对当前父类容器水平垂直居中
   * <code>android:layout_centerHorizontal="true"</code> 子类控件相对当前父类容器水平居中
   * <code>android:layout_centerVertical="true"</code> 子类控件相对当前父类窗口垂直居中

  #### 子控件属性
 ***
  * <code>android:layout_below="@+id/"</code> 该控件位于给定id控件的底部
  * <code>android:layout_toRightOf="@+id/"</code> 该控件位于给定id控件的右边
  * <code>android:layout_above="@+id/"</code> 该控件位于给定id的上面
  * <code>android:layout_toLeftOf="@+id/"</code> 该控件位于给定id的左边
  * <code>android:layout_alignBaseline="@+id/"</code> 该控件的内容与给定id控件的内容在一条线上
  * <code>android:layout_alignBottom/android:layout_alignLeft/android:layout_alignRight/android:layout_alignTop</code> 该控件的底部与给定id控件的[底部/左边/右边/顶部]边缘对齐

3. ### FrameLayout 帧布局

4. ### AbsoluteLayout绝对布局

  * 又叫坐标布局，可以直接指定子控件的绝对位置(xy)
  * <code>android:layout_x/android:layout_y</code>

5. ### TableLayout 表格布局

  #### 全局属性
  ***
  * <code>TableLayout</code>以行列的形式管理子控件，每一行为一个TableRow对象，也可以是一个View对象
  * <code>android:collapseColumns="1,2"</code>隐藏从0开始的索引列，多列用逗号分隔
  * <code>android:shrinkColumns="1,2"</code>收缩从0开始的索引列
  * <code>android:stretchColumns="1,2"</code>拉伸从0开始的索引列，*表示所有列

 #### 子控件属性
  ***
  * <code>android:layout_column</code> 第n列
  * <code>android:layout_span</code> 占据列宽
---
title: android学习笔记(1)
date: 2015-11-19 10:56:28
tags: android
---
1. ### 基础环境  
    - JDK(1.6+)  
    - Android SDK  
    - Eclipse(ADT)  
2. ### 目录结构
   - <code>src</code> 源码目录
   - <code>gen/</code>中存放系统自动生成的配置文件
   - <code>assets/</code>存放资源文件，不会自动生成id且不会自动占用空间(不允许下级文件夹)
   - <code>bin/</code>存放应用被编译后生成的class文件、apk文件
   - <code>res/</code>存放应用用到的所胡资源，如图片布局等等  
        * <code>res/drawable/</code>存放不同分辨率屏幕下的图片资源
        * <code>res/layout/</code>存放布局文件
        * <code>res/values/</code>存放字符串、主题、颜色、样式等资源文件
        * <code>res/raw</code>

    - <code>libs/</code>存放jar包
    - <code>AndroidManifest.xml</code>清单文件，配置应用相关信息，包括包名，权限，程序组件等等

3. ### <code>TextView</code>文本显示控件
   - <code>android:id</code> id名称
   - <code>android:layout_width</code> 控件宽度
     * <code>wrap_content</code> 以包含的内容宽度作为控件宽度
     * <code>fill_parent</code>(2.3以前)/<code>match_parent</code>(2.3以后使用) 当前控件宽度铺满父类容器宽度  
   - <code>android:text</code> 控件文本内容
   - <code>android:textSize</code> 文本大小
   - <code>android:textColor</code> 文本颜色

4. ### <code>EditText</code> 文本输入控件
   - <code>android:hint</code> 提示文本
   - <code>android:inputType</code> 文本输入类型

5. ### Error:Error retrieving parent for item: No resource found that matches the given name 'Theme.AppCompat.Light'.原因：未安装appcompat_v7支持sdk库

6. ### <code>ImageView</code> 显示图片控件
    - <code>android:src = "@drawable/PATH"</code>  图片路径
    - <code>android:background = "@drawable/PATH"</code> 背景图片路径

7. ### <code>Button/ImageButton</code>
    - Button可以通过<code>android:text</code>设置按钮要显示的文本
    - <code>ImageButton</code>可以通过<code>android:src</code>和<code>android:background</code>设置图片和背景图片
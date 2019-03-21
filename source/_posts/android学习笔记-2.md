---
title: android学习笔记(2)
date: 2015-11-19 10:57:52
tags: android
---
1. ### 按钮点击事件监听方式
  - 匿名内部类监听
    ```java
    btn.setOnclickListener(new OnClickListener() {
    });```

 - 外部类监听

    ```java
    btn.setOnclickListener(myListener);  

    class myListener implements OnClickListener {
    }
    ```

  - 实现接口监听

     ```java
       public class MainActivity extends Activity
          implements OnclickListener {
            @Override
            public void Onclick(View v) {
            }
        }
        btn.setOnClickListener(this);
    ```

2. ### dp/dip/sp/px区别  

3. ### AutoCompleteTextView/MultiAutoCompleteTextView  

4. ### ToggleButton  

5. ### CheckBox  

6. ### RadioButton/RadioGroup
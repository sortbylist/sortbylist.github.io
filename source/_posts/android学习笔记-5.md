---
title: android学习笔记(5)
date: 2015-11-19 14:33:27
tags: android
---
1. ### 获取SharedPreference
  - [getSharedPreferences(String name, int mode)][1] — 通过名称，和访问mode来新建一个shared preference文件。mode值可以是：
    ```
    MODE_PRIVATE
    MODE_WORLD_READABLE
    MODE_WORLD_WRITEABLE
    MODE_MULTI_PROCESS
    ```
  - [getPreferences()][2] — 创建默认的shared preference文件,app只需要一个preference文件时使用。

  - 例子：
```
Context context = getActivity();
SharedPreferences sharedPref = context.getSharedPreferences(
        getString(R.string.preference_file_key), Context.MODE_PRIVATE);
```

2. ### 写Shared Preference
  ``` Java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
SharedPreferences.Editor editor = sharedPref.edit();
editor.putInt(getString(R.string.saved_high_score), newHighScore);
editor.commit();
```

3. ### 读Shared Preference  
  ```
    SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
    long default = getResources().getInteger(R.string.saved_high_score_default));
    long highScore = sharedPref.getInt(getString(R.string.saved_high_score), default);

```

  [1]: http://developer.android.com/reference/android/content/Context.html#getSharedPreferences(java.lang.String,int)
  
  [2]: http://developer.android.com/reference/android/app/Activity.html#getPreferences(int)
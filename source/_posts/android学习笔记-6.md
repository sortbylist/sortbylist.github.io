---
title: android学习笔记(6)
date: 2015-11-19 16:19:46
tags: android
---
1. ### 保存内容到文件
 * 保存到内部存储(Internal Storage)
 * 保存到外部存储(External Storage)

2. ### 内部与外部存储比较
  #### 内部存储
    * 总是可用
    * 保存的文件只能被当前应用访问
    * 用户卸载应用后，内部存储保存的文件会被全部删除

  #### 外部存储
    * 当用户挂载外部存储(SD卡)时可用，卸载后无法使用
    * 其他应用可以访问
    * 用户卸载应用后，系统将会删除通过[getExternalFilesDir()][1]创建的目录

3. ### 保存文件到内部存储
 * getFilesDir()返回应用内部存储根目录
 * getCacheDir()返回应用内部存储缓存目录
 * 创建文件
 ```
   File file = new File(context.getFilesDir(), filename);
 ```
 * 通过openFileOutput或者FileOutputStream  
    ```
    String filename = "myfile";
    String string = "Hello world!";
    FileOutputStream outputStream;
    try {
      outputStream = openFileOutput(filename, Context.MODE_PRIVATE);
      outputStream.write(string.getBytes());
      outputStream.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
    ```
 * 创建缓存文件
    ```
    public File getTempFile(Context context, String url) {
        File file;
        try {
            String fileName = Uri.parse(url).getLastPathSegment();
            file = File.createTempFile(fileName, null, context.getCacheDir());
        catch (IOException e) {
            // Error while creating file
        }
        return file;
    }  
    ```
4. ### 保存文件到外部存储
 * 获取外部存储状态
    ```
    /* Checks if external storage is available for read and write */
    public boolean isExternalStorageWritable() {
        String state = Environment.getExternalStorageState();
        if (Environment.MEDIA_MOUNTED.equals(state)) {
      return true;
        }
        return false;
    }
    /* Checks if external storage is available to at least read */
    public boolean isExternalStorageReadable() {
        String state = Environment.getExternalStorageState();
        if (Environment.MEDIA_MOUNTED.equals(state) ||
      Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
      return true;
        }
        return false;
    }
    ```
 * 尽管external storage对于用户与其他app是可以修改的，但有2类文件可以保存在external storage中：
     *  <strong>Public files</strong>这些文件对与用户与其他app来说是public的，当用户卸载你的app时，这些文件应该保留。例如，那些被你的app拍摄的图片或者下载的文件。
     *  <strong>Private files</strong>对其他app无意义的，当用户卸载你的app时，系统会删除你的app的private目录。例如，那些被你的app下载的缓存文件。<br>  
 * 如果你想要保存文件为public形式的，用[getExternalStoragePublicDirectory()][2]方法来获取一个 File 对象来表示存储在external storage的目录。
    ``` public File getAlbumStorageDir(String albumName) {
        // Get the directory for the user's public pictures directory.
        File file = new File(Environment.getExternalStoragePublicDirectory(
        Environment.DIRECTORY_PICTURES), albumName);
        if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
        }
        return file;
    } 
    ```
 * 如果你想要保存文件为私有的方式，你可以通过执行[getExternalFilesDir()][3]来获取相应的目录，并且传递一个指示文件类型的参数。
    ```
    public File getAlbumStorageDir(Context context, String albumName) {
        // Get the directory for the app's private pictures directory.
        File file = new File(context.getExternalFilesDir(
                Environment.DIRECTORY_PICTURES), albumName);
        if (!file.mkdirs()) {
            Log.e(LOG_TAG, "Directory not created");
        }
        return file;
    }
    ```
5. ### 查询剩余空间
 * 通过执行[getFreeSpace()][4] or [getTotalSpace()][5] 来判断是否有足够的空间来保存文件，从而避免发生[IOException](http://developer.android.com/reference/java/io/IOException.html)。

6. ### 删除文件
 * 删除文件<code>myFile.delete()</code>
 * 如果文件是保存在internal storage，你可以通过Context来访问并通过执行deleteFile()进行删除```myContext.deleteFile(fileName);```
 > 当用户卸载你的app时，android系统会删除以下文件：
所有保存到internal storage的文件。
所有使用getExternalFilesDir()方式保存在external storage的文件。
然而，通常来说，你应该手动删除所有通过 getCacheDir() 方式创建的缓存文件，以及那些不会再用到的文件。

  [1]: http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)
  [2]: http://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)
  [3]: http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)
  [4]: http://developer.android.com/reference/java/io/File.html#getFreeSpace()
  [5]: http://developer.android.com/reference/java/io/File.html#getTotalSpace()
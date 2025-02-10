## sortbylist.github.io

1. `git clone https://github.com/sortbylist/sortbylist.github.io.git sortbylist.github.io`
2. `cd sortbylist.github.io`
3. `git clone https://github.com/theme-next/hexo-theme-next themes/next`
4. `cp themes/_config.yml themes/next/_config.yml`
5. edit hexo-renderer-marked插件源码，增加**号之间的代码，位置：sortbylist.github.io\node_modules\hexo-renderer-marked\lib\renderer.js
   ```js
   if (!/^(#|\/\/|http(s)?:)/.test(href) && !relative_link && prependRoot) {
      if (!href.startsWith('/') && !href.startsWith('\\') && postPath) {
        const PostAsset = hexo.model('PostAsset');
        // findById requires forward slash
   
        // ***************** Add the following code *******************
        const fixPostPath = join(postPath, '../');
        const asset = PostAsset.findById(join(fixPostPath, href.replace(/\\/g, '/')));
        // const asset = PostAsset.findById(join(postPath, href.replace(/\\/g, '/')));
        // ************************** End *****************************
   
        // asset.path is backward slash in Windows
        if (asset) href = asset.path.replace(/\\/g, '/');
      }
      href = url_for.call(hexo, href);
    }
   ```
6. install nvs
7. install node.js
8. `npm install`  

## 启动

1. `npm install hexo-cli -g`
2. hexo命令
   - `hexo g`[generate]  
   - `hexo s`[run server]  
   - `hexo new [layout]<title>`[create new post]  
   - `hexo deploy`[deploy,-g generate first]  

## enjoy! :)